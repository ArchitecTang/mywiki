.. contents::
   :depth: 3
..

1.4 zabbix监控历史数据清理操作方案
==================================

::

   2018/12/24 14:00:57 

zabbix监控运行一段时间以后，会留下大量的历史监控数据，zabbix数据库一直在增大；可能会造成系统性能下降，查看历史数据室查询速度缓慢。

zabbix里面最大的表就是history和history_uint两个表，而且zabbix里面的时间是使用的时间戳方式记录，所以可以根据时间戳来删除历史数据

1.4.1 关闭zabbix、http服务
--------------------------

::

   pkill -9 zabbix
   service httpd stop

1.4.2 清理zabbix历史数据
------------------------

查看数据库目录文件

::

   [root@zabbix-server zabbix]# cd /var/lib/mysql/zabbix/
   [root@zabbix-server zabbix]# ls -lh | grep G
   total 177G
   -rw-r----- 1 mysql mysql 1.7G Dec 24 13:49 events.ibd
   -rw-r----- 1 mysql mysql  60G Dec 24 13:49 history.ibd
   -rw-r----- 1 mysql mysql 2.4G Dec 24 13:49 history_str.ibd
   -rw-r----- 1 mysql mysql  99G Dec 24 13:49 history_uint.ibd
   -rw-r----- 1 mysql mysql 4.6G Dec 24 13:02 trends.ibd
   -rw-r----- 1 mysql mysql 9.5G Dec 24 13:49 trends_uint.ibd
   [root@zabbix-server zabbix]# 
   生成Unix时间戳。时间定为2018年2月1日（暂定是保存18年2月以后的监控数据）
   [root@zabbix-server zabbix]# date +%s -d "Feb 1, 2018 00:00:00"    #执行此命令以后会生成一个ID
   1517414400            #这是生成的ID

数据备份

::

   [root@zabbix-server zabbix]#mysql -uroot -p zabbix > /root/mysqlback/zabbix.sql     #需要创建mysqlback目录

登录数据库

::

   [root@zabbix-server zabbix]# mysql -uroot -p
   Enter password: 
   Welcome to the MariaDB monitor.  Commands end with ; or \g.
   Your MariaDB connection id is 7
   Server version: 5.5.60-MariaDB MariaDB Server

   Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

   MariaDB [(none)]> 

   use zabbix;    #选择zabbix数据库

   执行sql查看指定日期之前的数据大小：
   SELECT table_schema as `Database`,table_name AS `Table`,round(((data_length + index_length) / 1024 / 1024 / 1024), 2) `Size in MB`FROM information_schema.TABLES where CREATE_TIME < '2018-02-01 00:00:00' and table_name='history.ibd';
   根据需要修改日期和查询的表名称(如果查询出来的结果是0.0，需要将sql中的三个1024删除一个，以G为单位显示)

执行以下命令，清理指定时间之前的数据、对zabbix数据库执行以下sql

::

   delete from history where clock < 1517414400;
   optimize table history;

   delete from history_uint where clock < 1517414400;
   optimize table history_uint;

   delete from trends where clock < 1517414400;
   optimize table trends;

   delete from trends_uint where clock < 1517414400;
   optimize table trends_uint;

   注意：sql中的ID是生成Unix时间戳的ID号,需要改为自己生成的ID号

1.4.3 启动服务
--------------

::

   /usr/sbin/zabbix_server -c /etc/zabbix/zabbix_server.conf    #zabbix server
   /usr/sbin/zabbix_agentd -c /etc/zabbix/zabbix_agentd.conf    #zabbix agent
   service httpd start

===============================分===========隔==========符===================================

1.4.4 使用truncate命令清空zabbix 所有监控数据
---------------------------------------------

::

   -------------------------------------------------------
   truncate table history;
   optimize table history;
   ------------------------------------------------------- 
   truncate table history_str;
   optimize table history_str;
   -------------------------------------------------------
   truncate table history_uint;
   optimize table history_uint;
   -------------------------------------------------------
   truncate table trends;
   optimize table trends;
   -------------------------------------------------------
   truncate table trends_uint; 
   optimize table trends_uint; 
   -------------------------------------------------------
   truncate table events;
   optimize table events;
   -------------------------------------------------------

注意：这些命令会把zabbix所有的监控数据清空，操作前注意备份数据库

truncate是删除了表，然后根据表结构重新建立，delete删除的是记录的数据没有修改表

truncate执行删除比较快，但是在事务处理安全性方面不如delete,如果我们执行truncat的表正在处理事务，这个命令退出并会产生错误信息
