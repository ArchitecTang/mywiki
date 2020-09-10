.. contents::
   :depth: 3
..

1.8 zookeeper 集群部署
======================

访问zookeeper官网下载最新、稳定zookeeper版本

将下载二进制文件分别上传至需要安装的服务器上(以下操作需要在所有节点执行)

创建zookeeper目录

::

   mkdir /software/data/

将二进制文件上传至/software/data/目录

解压文件：

::

   tar xf apache-zookeeper-3.5.5-bin.tar.gz

重命名

::

   mv apache-zookeeper-3.5.5-bin zookeeper
   cd zookeeper

创建logs和data目录

::

   mkdir logs data

进入conf目录修改以下配置

::

   cd conf
   cp zoo_sample.cfg zoo.cfg
   vim zoo.cfg


   # The number of milliseconds of each tick
   tickTime=2000
   # The number of ticks that the initial 
   # synchronization phase can take
   initLimit=10
   # The number of ticks that can pass between 
   # sending a request and getting an acknowledgement
   syncLimit=5
   # the directory where the snapshot is stored.
   # do not use /tmp for storage, /tmp here is just 
   # example sakes.
   dataDir=/software/data/zookeeper/data    #修改
   dataLogDir=/software/data/zookeeper/logs #新增
   # the port at which the clients will connect
   clientPort=2181
   # the maximum number of client connections.
   # increase this if you need to handle more clients
   #maxClientCnxns=60
   #
   # Be sure to read the maintenance section of the 
   # administrator guide before turning on autopurge.
   #
   # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
   #
   # The number of snapshots to retain in dataDir
   #autopurge.snapRetainCount=3
   # Purge task interval in hours
   # Set to "0" to disable auto purge feature
   #autopurge.purgeInterval=1
   server.1=192.168.1.102:2881:3881    #新增
   server.2=192.168.1.103:2881:3881    #新增             三台机器的新增内容都需要一致
   server.3=192.168.1.105:2881:3881    #新增

新增myid文件

除了修改 zoo.cfg
配置文件，集群模式下还要新增一个名叫myid的文件，这个文件放在上述dataDir指定的目录下，这个文件里面就只有一个数据，就是上图配置中server.x的这个x（1,2,3）值，zookeeper启动时会读取这个文件，拿到里面的数据与
zoo.cfg 里面的配置信息比较从而判断到底是那个server（节点）。

::

   cd /software/data/zookeeper/data/
   cat myid 
   1     #node1节点的值就为1，node2为2，node3为3

设置zookeeper java环境

::

   vim /software/data/zookeeper/bin/zkEnv.sh
   新增：

   export JAVA_HOME=/software/data/jdk1.8.0_231
   export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin:$PATH
   export CLASSPATH=.$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

启动zookeeper服务

::

    /software/data/zookeeper/bin/zkServer.sh start
   ZooKeeper JMX enabled by default
   Using config: /software/data/zookeeper/bin/../conf/zoo.cfg
   Starting zookeeper ... STARTED

   /software/data/zookeeper/bin/zkServer.sh status
   ZooKeeper JMX enabled by default
   Using config: /software/data/zookeeper/bin/../conf/zoo.cfg
   Client port found: 2181. Client address: localhost.
   Mode: leader
