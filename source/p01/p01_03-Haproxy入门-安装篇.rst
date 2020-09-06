1.3 Haproxy入门(安装篇)
^^^^^^^^^^^^^^^^^^^^^^^

1.3.1 下载需要的haproxy版本并上传到服务器上
'''''''''''''''''''''''''''''''''''''''''''

::

   wget http://pkgs.fedoraproject.org/repo/pkgs/haproxy/haproxy-1.7.9.tar.gz

.. figure:: https://i.loli.net/2019/07/21/5d34068e4705318240.png
   :alt: 下载.png

::

   tar -zxvf haproxy-1.7.9.tar.gz
   cd haproxy-1.7.9

查看linux内核版本 |版本.png| 编译

::

   make TARGET=linux26 prefix=/usr/local/haproxy

|编译.png| 安装

::

   make install PREFIX=/usr/local/haproxy

.. figure:: https://i.loli.net/2019/07/21/5d340797a5b5d68902.png
   :alt: 安装.png

::

   参数说明：
   TARGET=linux26
   #使用uname -r查看内核，如：2.6.18-371.el5，此时该参数就为linux26
   #kernel 大于2.6.28的用：TARGET=linux2628
   CPU=x86_64   #使用uname -r查看系统信息，如x86_64 x86_64 x86_64 GNU/Linux，此时该参数就为x86_64
   PREFIX=/usr/local/haprpxy   #/usr/local/haprpxy为haprpxy安装路径

1.3.2 设置HAproxy
'''''''''''''''''

::

   haproxy的配置文件需要自己创建
   mkdir /usr/local/haproxy/conf/
   mkdir /usr/local/haproxy/logs

   vim /usr/local/haproxy/haproxy.cfg

填写以下内容：

::

       global
       log    127.0.0.1 local3
       chroot  /usr/local/haproxy
       pidfile  /usr/local/haproxy/logs/haproxy.pid
       maxconn  4000
       uid 99
       gid 99
       daemon
       nbproc 1
       stats socket /usr/local/haproxy/haproxy.sock level admin
   #---------------------------------------------------------------------
   # common defaults that all the 'listen' and 'backend' sections will 
   # use if not designated in their block
   #---------------------------------------------------------------------
   defaults
           log     global
           option dontlognull
           option redispatch
           mode http
           timeout connect      5000
           timeout client      50000
           timeout server      50000
   listen stats
           mode http
           bind 172.31.8.102:8888
           stats enable
           stats   uri    /haproxy-status
           stats  auth haproxy:haproxy
               stats hide-version
           stats admin if TRUE
   ########################################################################
   backend test-web
           mode http
           acl being_scanned be_conn gt 1500
           http-request deny if being_scanned
           option httplog
           option httpclose
           option forwardfor
           log global

           server mobile-node9  172.31.8.24:80 cookie web1 check inter 500 rise 3 fall 3

1.3.3 启动haproxy
'''''''''''''''''

::

   /usr/local/haproxy/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg

查看haproxy启动进程 |进程.png|
配置文件设置的访问用户名和密码为haproxy，web访问IP为172.31.8.102端口号为8888

1.3.4 使用浏览器进行访问测试
''''''''''''''''''''''''''''

::

   http://172.31.8.102:8888/haproxy-status

1.3.5 测试
''''''''''

|控制台.png| 停掉nginx服务，检测到的状态为DOWN |状态检测.png|

1.3.6 编写haproxy启动脚本
'''''''''''''''''''''''''

::

   vim haproxy.sh

填写入以下内容：

::

   #!/bin/bash
   BASE_DIR="/usr/local/haproxy"
   ARGV="$@"
   pp=`ps -fe |grep haproxy |grep -v "grep" |grep -v "haproxy.sh"|awk '{print $2}'`

   start()
   {
   echo "START HAPoxy SERVERS OK"
   $BASE_DIR/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
   }

   stop()
   {
   kill -9 $pp
   echo "STOP HAPoxy Listen OK"
   }
   case $ARGV in

   start)
   start
   ERROR=$?
   ;;

   stop)
   stop
   ERROR=$?
   ;;

   restart)
   stop
   start
   ERROR=$?
   ;;

   *)
   echo "haproxy.sh [start|restart|stop]"
   esac
   exit $ERROR

   chmod +x haproxy.sh

|验证.png|
安装验证正常，HAproxy安装完毕（以上为haproxy单节点安装，启动脚本、haproxy.cfg配置文件可看下面汇总内容）

1.3.7 haproxy.cfg 配置文件
''''''''''''''''''''''''''

::

       global
       log    127.0.0.1 local3
       chroot  /usr/local/haproxy
       pidfile  /usr/local/haproxy/logs/haproxy.pid
       maxconn  4000
       uid 99
       gid 99
       daemon
       nbproc 1
       stats socket /usr/local/haproxy/haproxy.sock level admin
   #---------------------------------------------------------------------
   # common defaults that all the 'listen' and 'backend' sections will 
   # use if not designated in their block
   #---------------------------------------------------------------------
   defaults 
           log     global
           option dontlognull 
           option redispatch 
           mode http
           timeout connect      5000 
           timeout client      50000 
           timeout server      50000 
   listen stats
           mode http
           bind 172.31.8.102:8888
           stats enable
           stats   uri    /haproxy-status 
           stats  auth haproxy:haproxy
               stats hide-version
           stats admin if TRUE                  
   ########################################################################
   backend test-web
           mode http
           acl being_scanned be_conn gt 1500
           http-request deny if being_scanned
           option httplog
           option httpclose
           option forwardfor
           log global

           server mobile-node9  172.31.8.24:80 cookie web1 check inter 500 rise 3 fall 3
           server mobile-node8  172.31.8.102:80 cookie web1 check inter 500 rise 3 fall 3

1.3.8 haproxy.sh 启动脚本：
'''''''''''''''''''''''''''

::

   #!/bin/bash
   BASE_DIR="/usr/local/haproxy"
   ARGV="$@"
   pp=`ps -fe |grep haproxy |grep -v "grep" |grep -v "haproxy.sh"|awk '{print $2}'`

   start()
   {
   echo "START HAPoxy SERVERS OK"
   $BASE_DIR/sbin/haproxy -f /usr/local/haproxy/haproxy.cfg
   }

   stop()
   {
   kill -9 $pp
   echo "STOP HAPoxy Listen OK"
   }
   case $ARGV in

   start)
   start
   ERROR=$?
   ;;

   stop)
   stop
   ERROR=$?
   ;;

   restart)
   stop
   start
   ERROR=$?
   ;;

   *)
   echo "haproxy.sh [start|restart|stop]"
   esac
   exit $ERROR

.. |版本.png| image:: https://i.loli.net/2019/07/21/5d3407121b56169904.png
.. |编译.png| image:: https://i.loli.net/2019/07/21/5d340776c5c2a40135.png
.. |进程.png| image:: https://i.loli.net/2019/07/21/5d340858bac9086754.png
.. |控制台.png| image:: https://i.loli.net/2019/07/21/5d3408895571b49377.png
.. |状态检测.png| image:: https://i.loli.net/2019/07/21/5d3408b0e8b1382259.png
.. |验证.png| image:: https://i.loli.net/2019/07/21/5d340907c31e758930.png
