.. contents::
   :depth: 3
..

8.2 Mysql 一主多从集群搭建
==========================

8.2.1 主从配置
--------------

Mysql安装可以使用yum或编译安装\ `编译安装可参考 <https://architectang.github.io/2018/06/22/Mysql-编译安装/>`__

新建以下三台mysql服务器，数据库为编译安装。

::

   master：192.168.1.105
   slave1：192.168.1.79
   slave2：192.168.1.111    

（如果是新增从库，需要先在master节点进行锁表，禁止数据写入，然后备份master节点的mysql数据，在slave节点导入备份的mysql数据后，再把master节点的锁表关闭，我这里是新建数据库所以就不做以上操作了）

::

   锁表sql
   flush tables with read lock;
   在从库导入数据，slave状态正常以后，再回到master节点解锁
   unlock tables;

mysql默认是不允许远程连接的，所以需要打开所有服务器的远程访问权限

::

   创建远程访问用户
   grant all on *.* to 'devsync'@'192.168.1.%' identified by '123456' with grant option;
   刷新权限
   flush privileges;

修改master节点的配置文件

::

   vim /etc/my.cnf

增加以下内容：

::

   server-id=1    #server的唯一标识
   auto_increment_offset=1 #自增id起始值
   auto_increment_increment=2 #每次自增数字

   log-bin=master-bin
   max_binlog_size=1024M #binlog单文件最大值
   binlog_format=mixed #指定mysql的binlog日志的格式，mixed是混合模式
   log-bin-index=master-bin.index
   #只同步test数据库(可选配置)
   #binlog-do-db=test
   replicate-ignore-db = mysql #忽略不同步主从的数据库

保存退出后重启master节点的mysql服务

::

   systemctl restart mysqld

查看master的信息

::

   +-------------------+----------+--------------+------------------+-------------------+
   | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
   +-------------------+----------+--------------+------------------+-------------------+
   | master-bin.000001 |  159     |              |                  |                   |
   +-------------------+----------+--------------+------------------+-------------------+
   1 row in set (0.00 sec)

到这里，master节点就已经配置完成了

配置slave节点，修改/etc/my.cnf文件

::

   vim /etc/my.cnf

在[mysqld]中添加以下内容：

::

   server-id=74
   log-bin=master-bin
   log-bin-index=master-bin.index

重启slave 节点的mysql数据库

::

   systemctl restart mysqld

连接slave节点的的数据库，配置master节点的信息，开启slave

::

   change master to 
   master_host='192.168.1.105'
   ,master_user='devsync'
   ,master_password='123456'
   ,master_log_file='master-bin.000001'
   ,master_log_pos=159;

开启slave

::

   start slave；

查看slave状态：

::

   mysql> show slave status \G;
   *************************** 1. row ***************************
                  Slave_IO_State: Waiting for master to send event
                     Master_Host: 192.168.1.105
                     Master_User: devsync
                     Master_Port: 3306
                   Connect_Retry: 60
                 Master_Log_File: master-bin.000003
             Read_Master_Log_Pos: 5584496
                  Relay_Log_File: bogon-relay-bin.000002
                   Relay_Log_Pos: 5584663
           Relay_Master_Log_File: master-bin.000003
                Slave_IO_Running: Yes
               Slave_SQL_Running: Yes
                 Replicate_Do_DB: 
             Replicate_Ignore_DB: 
              Replicate_Do_Table: 
          Replicate_Ignore_Table: 
         Replicate_Wild_Do_Table: 
     Replicate_Wild_Ignore_Table: 
                      Last_Errno: 0
                      Last_Error: 
                    Skip_Counter: 0
             Exec_Master_Log_Pos: 5584496
                 Relay_Log_Space: 5584870
                 Until_Condition: None
                  Until_Log_File: 
                   Until_Log_Pos: 0
              Master_SSL_Allowed: No
              Master_SSL_CA_File: 
              Master_SSL_CA_Path: 
                 Master_SSL_Cert: 
               Master_SSL_Cipher: 
                  Master_SSL_Key: 
           Seconds_Behind_Master: 0
   Master_SSL_Verify_Server_Cert: No
                   Last_IO_Errno: 0
                   Last_IO_Error: 
                  Last_SQL_Errno: 0
                  Last_SQL_Error: 
     Replicate_Ignore_Server_Ids: 
                Master_Server_Id: 1
                     Master_UUID: 5d8b6583-2546-11ea-a678-000c29128652
                Master_Info_File: /usr/local/mysql/data/master.info
                       SQL_Delay: 0
             SQL_Remaining_Delay: NULL
         Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Master_Retry_Count: 86400
                     Master_Bind: 
         Last_IO_Error_Timestamp: 
        Last_SQL_Error_Timestamp: 
                  Master_SSL_Crl: 
              Master_SSL_Crlpath: 
              Retrieved_Gtid_Set: 
               Executed_Gtid_Set: 
                   Auto_Position: 0
            Replicate_Rewrite_DB: 
                    Channel_Name: 
              Master_TLS_Version: 
   1 row in set (0.00 sec)

   ERROR: 
   No query specified

特别需要注意的参数是：

::

   Slave_IO_Running: Yes
   Slave_SQL_Running: Yes
   需要全部都为Yes，同步状态才是正常的

在另一个slave节点做同样的操作，需要注意server-id不能一样

主从同步配置 END

8.2.2 同步状态异常情况处理
--------------------------

Slave_IO_Running:或 Slave_SQL_Running: 为No

查看slave状态

::

   mysql> show slave status \G;
   *************************** 1. row ***************************
                  Slave_IO_State: Waiting for master to send event
                     Master_Host: 192.168.1.105
                     Master_User: devsync
                     Master_Port: 3306
                   Connect_Retry: 60
                 Master_Log_File: master-bin.000003
             Read_Master_Log_Pos: 5584496
                  Relay_Log_File: bogon-relay-bin.000002
                   Relay_Log_Pos: 5584663
           Relay_Master_Log_File: master-bin.000003
                Slave_IO_Running: No
               Slave_SQL_Running: Yes
                 Replicate_Do_DB: 
             Replicate_Ignore_DB: 
              Replicate_Do_Table: 
          Replicate_Ignore_Table: 
         Replicate_Wild_Do_Table: 
     Replicate_Wild_Ignore_Table: 
                      Last_Errno: 0
                      Last_Error: 
                    Skip_Counter: 0
             Exec_Master_Log_Pos: 5584496
                 Relay_Log_Space: 5584870
                 Until_Condition: None
                  Until_Log_File: 
                   Until_Log_Pos: 0
              Master_SSL_Allowed: No
              Master_SSL_CA_File: 
              Master_SSL_CA_Path: 
                 Master_SSL_Cert: 
               Master_SSL_Cipher: 
                  Master_SSL_Key: 
           Seconds_Behind_Master: 0
   Master_SSL_Verify_Server_Cert: No
                   Last_IO_Errno: 0
                   Last_IO_Error: 
                  Last_SQL_Errno: 0
                  Last_SQL_Error: 
     Replicate_Ignore_Server_Ids: 
                Master_Server_Id: 1
                     Master_UUID: 5d8b6583-2546-11ea-a678-000c29128652
                Master_Info_File: /usr/local/mysql/data/master.info
                       SQL_Delay: 0
             SQL_Remaining_Delay: NULL
         Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Master_Retry_Count: 86400
                     Master_Bind: 
         Last_IO_Error_Timestamp: 
        Last_SQL_Error_Timestamp: 
                  Master_SSL_Crl: 
              Master_SSL_Crlpath: 
              Retrieved_Gtid_Set: 
               Executed_Gtid_Set: 
                   Auto_Position: 0
            Replicate_Rewrite_DB: 
                    Channel_Name: 
              Master_TLS_Version: 
   1 row in set (0.00 sec)

   ERROR: 
   No query specified

处理过程： 在slave节点操作：

::

   stop slave；

在master节点操作：

::

   mysql> show master status;
   +-------------------+----------+--------------+------------------+-------------------+
   | File              | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
   +-------------------+----------+--------------+------------------+-------------------+
   | master-bin.000003 |  5584496 |              |                  |                   |
   +-------------------+----------+--------------+------------------+-------------------+
   1 row in set (0.01 sec)

切换到slave节点执行以下sql：

::

   change master to
   master_host='192.168.1.105',    #master节点IP
   master_user='devsync',          #同步账号
   master_password='123456',       #密码
   master_port=3306,               #master节点服务端口
   master_log_file='master-bin.000003',  #master节点的File
   master_log_pos=5584496;               # master节点的Position值

启动slave

::

   start slave；

查看同步状态：

::

   mysql> show slave status \G;
   *************************** 1. row ***************************
                  Slave_IO_State: Waiting for master to send event
                     Master_Host: 192.168.1.105
                     Master_User: devsync
                     Master_Port: 3306
                   Connect_Retry: 60
                 Master_Log_File: master-bin.000003
             Read_Master_Log_Pos: 5584496
                  Relay_Log_File: bogon-relay-bin.000002
                   Relay_Log_Pos: 5584663
           Relay_Master_Log_File: master-bin.000003
                Slave_IO_Running: Yes
               Slave_SQL_Running: Yes
                 Replicate_Do_DB: 
             Replicate_Ignore_DB: 
              Replicate_Do_Table: 
          Replicate_Ignore_Table: 
         Replicate_Wild_Do_Table: 
     Replicate_Wild_Ignore_Table: 
                      Last_Errno: 0
                      Last_Error: 
                    Skip_Counter: 0
             Exec_Master_Log_Pos: 5584496
                 Relay_Log_Space: 5584870
                 Until_Condition: None
                  Until_Log_File: 
                   Until_Log_Pos: 0
              Master_SSL_Allowed: No
              Master_SSL_CA_File: 
              Master_SSL_CA_Path: 
                 Master_SSL_Cert: 
               Master_SSL_Cipher: 
                  Master_SSL_Key: 
           Seconds_Behind_Master: 0
   Master_SSL_Verify_Server_Cert: No
                   Last_IO_Errno: 0
                   Last_IO_Error: 
                  Last_SQL_Errno: 0
                  Last_SQL_Error: 
     Replicate_Ignore_Server_Ids: 
                Master_Server_Id: 1
                     Master_UUID: 5d8b6583-2546-11ea-a678-000c29128652
                Master_Info_File: /usr/local/mysql/data/master.info
                       SQL_Delay: 0
             SQL_Remaining_Delay: NULL
         Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Master_Retry_Count: 86400
                     Master_Bind: 
         Last_IO_Error_Timestamp: 
        Last_SQL_Error_Timestamp: 
                  Master_SSL_Crl: 
              Master_SSL_Crlpath: 
              Retrieved_Gtid_Set: 
               Executed_Gtid_Set: 
                   Auto_Position: 0
            Replicate_Rewrite_DB: 
                    Channel_Name: 
              Master_TLS_Version: 
   1 row in set (0.00 sec)

   ERROR: 
   No query specified

主从同步已经正常

Slave_IO_Running 或 Slave_SQL_Running: 为Connecting

可能的原因有：

::

   1.网络不通
   2.账户密码错误
   3.防火墙没有关闭或放行端口
   4.mysql配置文件异常
   5.连接服务器时语法错误（一般为master节点的IP配置有问题，比较少见）
   6.主服务器mysql权限
