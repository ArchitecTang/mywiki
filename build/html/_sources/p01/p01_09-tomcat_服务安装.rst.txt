.. contents::
   :depth: 3
..

1.9 tomcat 服务安装
===================

tomcat简介
----------

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，Tomcat是Apache
软件基金会（Apache Software Foundation）的Jakarta
项目中的一个核心项目，它早期的名称为catalina，后来由Apache、Sun
和其他一些公司及个人共同开发而成，并更名为Tomcat。Tomcat
是一个小型的轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP
程序的首选，因为Tomcat 技术先进、性能稳定，成为目前比较流行的Web
应用服务器。Tomcat是应用（java）服务器，它只是一个servlet容器，是Apache的扩展，但它是独立运行的。目前最新的版本为Tomcat
8.0.24 Released。

Tomcat不是一个完整意义上的Jave
EE服务器，它甚至都没有提供对哪怕是一个主要Java EE
API的实现；但由于遵守apache开源协议，tomcat却又为众多的java应用程序服务器嵌入自己的产品中构建商业的java应用程序服务器，如JBoss和JOnAS。尽管Tomcat对Jave
EE API的实现并不完整，然而很企业也在渐渐抛弃使用传统的Java
EE技术（如EJB）转而采用一些开源组件来构建复杂的应用。这些开源组件如Structs、Spring和Hibernate，而Tomcat能够对这些组件实现完美的支持。

`详细请参考-运维生存时间 <http://www.ttlsa.com/tomcat/tomcat-install-and-configure/>`__

1.9.1 安装环境:
---------------

::

   Centos 7 
   tomcat版本: apache-tomcat-8.5.42.tar 
   java版本: java8

`官方网站 <http://tomcat.apache.org/>`__

`下载tomcat8 <https://tomcat.apache.org/download-80.cgi>`__

1.9.2 安装
----------

上传至Linux 服务器

创建tomcat安装目录

::

   mkdir /tomcat

解压tomcat、jdk

::

   [root@node1 tomcat]# ll
   总用量 178712
   -rw-r--r--. 1 root root   9711748 7月   9 10:55 apache-tomcat-8.5.42.tar.gz
   -rw-r--r--. 1 root root 173281904 7月   9 10:55 jdk-8u51-linux-x64.tar.gz
   [root@node1 tomcat]# tar xf jdk-8u51-linux-x64.tar.gz 
   [root@node1 tomcat]# tar xf apache-tomcat-8.5.42.tar.gz 
   [root@node1 tomcat]# 

.. figure:: https://i.loli.net/2019/07/09/5d2401ef70aba45222.png
   :alt: 软件列表

   软件列表

1.9.3 启动脚本配置
------------------

::

   #!/bin/bash
   source /etc/sysconfig/i18n
   export JAVA_HOME=/tomcat/jdk1.8.0_51
   export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH   --配置启动java
   export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar
   export TOMCAT_HOME=/tomcat/apache-tomcat-8.5.42/
   export PATH=$PATH:$TOMCAT_HOME/bin
   #export JAVA_OPTS="$JAVA_OPTS -Xms10000m -Xmx10000m -Xmn4000m -XX:PermSize=256m -XX:MaxPermSize=512m"    -配置程序内存
   PROG="tomcat"
   IP_ADDR=$(/sbin/ifconfig eth0 | grep "inet addr" | awk '{print $2}' | awk -F ":" '{print $2}')
   tomcat_start()  ---启动tomcat
   {
           rm -fr /tomcat/apache-tomcat-8.5.42/work/*     ----删除缓存
           rm -fr /tomcat/apache-tomcat-8.5.42/temp/*     ----删除临时缓存

           echo  $"Starting $IP_ADDR $PROG: "
           cd /tomcat/apache-tomcat-8.5.42/bin/
           ./startup.sh
           echo "tomcat 8 starting......"

   }
   tomcat_log()    ---查看程序日志
   {
           echo -n $"ShowLoging $IP_ADDR $PROG log: "
           tail -n200 -f /tomcat/apache-tomcat-8.5.42/logs/catalina.out
   }
   tomcat_stop()  ---停止tomcat
   {
       echo $"Stopping $IP_ADDR $PROG: "
           kill $(ps -ef | grep java | grep -v "grep" | grep "apache-tomcat" |awk -F ' ' '{print $2}')
           sleep 12
           TOMCAT_STATUS1=$(ps -ef | grep java | grep -v "grep"| grep "apache-tomcat")
           if [ -n "$TOMCAT_STATUS1" ];then
                   echo "Run Kill -9"
                   kill -9 $(ps -ef | grep java | grep -v "grep" | grep "apache-tomcat" |awk -F " " '{print $2}')
                   sleep 2
           fi
           TOMCAT_STATUS2=$(ps -ef | grep java | grep -v "grep"| grep "apache-tomcat" )
           if [ -z "$TOMCAT_STATUS2" ];then
                   echo "Tomcat Stop ok"
           else
                   exit 1
           fi
   }
   tomcat_status()   ---查看tomcat程序状态
   {
           ps -ef | grep java | grep -v "grep"| grep "tomcat"
   }
   case "$1" in
   start)
           tomcat_start;
           ;;
   stop)
           tomcat_stop;
           ;;
   restart)
           tomcat_stop;
           tomcat_start;
           ;;
   status)
                   tomcat_status;
                   ;;
   log)
           tomcat_log;
                   ;;
   *)
           echo $"Usage: $prog {start | stop | restart | log | status}"
           exit 1
   esac

tomcat
需要jdk才能运行，上面解压以后我是在tomcat启动脚本里面配置的jdk目录，也可以定义在系统全局变量里面，但是我不会这么做，因为我们的环境经常是运行多个实例且JDK版本要求不同，如果安装多个jdk会造成全局冲突

全局变量定义方式:

::

   vim /etc/profile
   写入以下内容:
   export JAVA_HOME=/jboss/jdk1.8.0_51
   export CLASSPATH=$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
   export JRE_HOME=$JAVA_HOME/jre
   export PATH=$PATH:$JAVA_HOME/bin
   source /etc/profile -- 生效配置
   验证java
   [root@node1 ~]# java -version
   java version "1.8.0_51"
   Java(TM) SE Runtime Environment (build 1.8.0_51-b16)
   Java HotSpot(TM) 64-Bit Server VM (build 25.51-b03, mixed mode)
   [root@node1 ~]# 

.. figure:: https://i.loli.net/2019/07/09/5d2411d062f9884358.png
   :alt: jdk.png


1.9.4 启动tomcat
----------------

::

   [root@node1 ~]# ./tomcat.sh start   ---启动服务
   Starting  tomcat: 
   Using CATALINA_BASE:   /tomcat/apache-tomcat-8.5.42
   Using CATALINA_HOME:   /tomcat/apache-tomcat-8.5.42
   Using CATALINA_TMPDIR: /tomcat/apache-tomcat-8.5.42/temp
   Using JRE_HOME:        /jboss/jdk1.8.0_51/jre
   Using CLASSPATH:       /tomcat/apache-tomcat-8.5.42/bin/bootstrap.jar:/tomcat/apache-tomcat-8.5.42/bin/tomcat-juli.jar
   Tomcat started.
   tomcat 8 starting......
   [root@node1 ~]# ./tomcat.sh status
   root     24103     1  0 11:39 ?        00:00:04 /jboss/jdk1.8.0_51/jre/bin/java -Djava.util.logging.config.file=/tomcat/apache-tomcat-8.5.42/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Xms1000m -Xmx1000m -Xmn400m -XX:PermSize=256m -XX:MaxPermSize=512m -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /tomcat/apache-tomcat-8.5.42/bin/bootstrap.jar:/tomcat/apache-tomcat-8.5.42/bin/tomcat-juli.jar -Dcatalina.base=/tomcat/apache-tomcat-8.5.42 -Dcatalina.home=/tomcat/apache-tomcat-8.5.42 -Djava.io.tmpdir=/tomcat/apache-tomcat-8.5.42/temp org.apache.catalina.startup.Bootstrap start

       09-Jul-2019 11:39:42.759 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Server version:        Apache Tomcat/8.5.42
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Jun 4 2019 20:29:04 UTC
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Server number:         8.5.42.0
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            3.10.0-327.el7.x86_64
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /jboss/jdk1.8.0_51/jre
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           1.8.0_51-b16
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Oracle Corporation
   09-Jul-2019 11:39:42.763 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /tomcat/apache-tomcat-8.5.42
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /tomcat/apache-tomcat-8.5.42
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/tomcat/apache-tomcat-8.5.42/conf/logging.properties
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Xms1000m
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Xmx1000m
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Xmn400m
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -XX:PermSize=256m
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -XX:MaxPermSize=512m
   09-Jul-2019 11:39:42.764 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djdk.tls.ephemeralDHKeySize=2048
   09-Jul-2019 11:39:42.765 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources
   09-Jul-2019 11:39:42.765 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027
   09-Jul-2019 11:39:42.765 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dignore.endorsed.dirs=
   09-Jul-2019 11:39:42.765 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/tomcat/apache-tomcat-8.5.42
   09-Jul-2019 11:39:42.765 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/tomcat/apache-tomcat-8.5.42
   09-Jul-2019 11:39:42.765 信息 [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/tomcat/apache-tomcat-8.5.42/temp
   09-Jul-2019 11:39:42.765 信息 [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib]
   09-Jul-2019 11:39:43.048 信息 [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
   09-Jul-2019 11:39:43.071 信息 [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
   09-Jul-2019 11:39:43.095 信息 [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["ajp-nio-8009"]
   09-Jul-2019 11:39:43.113 信息 [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
   09-Jul-2019 11:39:43.124 信息 [main] org.apache.catalina.startup.Catalina.load Initialization processed in 1109 ms
   09-Jul-2019 11:39:43.158 信息 [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
   09-Jul-2019 11:39:43.158 信息 [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet Engine: Apache Tomcat/8.5.42
   09-Jul-2019 11:39:43.207 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/tomcat/apache-tomcat-8.5.42/webapps/ROOT]
   09-Jul-2019 11:44:19.656 警告 [localhost-startStop-1] org.apache.catalina.util.SessionIdGeneratorBase.createSecureRandom Creation of SecureRandom instance for session ID generation using [SHA1PRNG] took [276,003] milliseconds.
   09-Jul-2019 11:44:19.697 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/tomcat/apache-tomcat-8.5.42/webapps/ROOT] has finished in [276,491] ms
   09-Jul-2019 11:44:19.697 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/tomcat/apache-tomcat-8.5.42/webapps/docs]
   09-Jul-2019 11:44:19.739 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/tomcat/apache-tomcat-8.5.42/webapps/docs] has finished in [42] ms
   09-Jul-2019 11:44:19.739 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/tomcat/apache-tomcat-8.5.42/webapps/examples]
   09-Jul-2019 11:44:20.212 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/tomcat/apache-tomcat-8.5.42/webapps/examples] has finished in [473] ms
   09-Jul-2019 11:44:20.213 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/tomcat/apache-tomcat-8.5.42/webapps/host-manager]
   09-Jul-2019 11:44:20.266 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/tomcat/apache-tomcat-8.5.42/webapps/host-manager] has finished in [53] ms
   09-Jul-2019 11:44:20.266 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/tomcat/apache-tomcat-8.5.42/webapps/manager]
   09-Jul-2019 11:44:20.293 信息 [localhost-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/tomcat/apache-tomcat-8.5.42/webapps/manager] has finished in [27] ms
   09-Jul-2019 11:44:20.314 信息 [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
   09-Jul-2019 11:44:20.342 信息 [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["ajp-nio-8009"]
   09-Jul-2019 11:44:20.344 信息 [main] org.apache.catalina.startup.Catalina.start Server startup in 277219 ms    ----看到此输出，程序就已经正常启动

通过浏览器访问tomcat服务,tomcat 默认端口号为8080

::

   http://ipaddr:8080

|tomcat.png| tomcat端口可在/conf/server.xml文件中修改

::

   67    Define a non-SSL/TLS HTTP/1.1 Connector on port 8080
   68    -->
   69    <Connector port="8080" protocol="HTTP/1.1"    --将8080修改为80或其他需要的端口
   70               connectionTimeout="20000"
   71               redirectPort="8443" />
   72    <!-- A "Connector" using the shared thread pool-->

|8080.png| tomcat已安装完毕

1.9.5 tomcat目录结构说明
------------------------

.. figure:: https://i.loli.net/2019/07/09/5d240ec19f9ee56826.png
   :alt: 目录结构.png

::

                       bin             -- 程序启动目录
                       conf            -- 配置文件目录
                       lib             -- 库文件目录
                       logs            -- 日志文件目录
                       temp            -- 临时缓存目录
                       webapps         -- web 应用家目录
                       work            -- 工作缓存目录


.. |tomcat.png| image:: https://i.loli.net/2019/07/09/5d240e566144292606.png
.. |8080.png| image:: https://i.loli.net/2019/07/09/5d2413ad1c50b94868.png
