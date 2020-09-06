1.2 Rabbitmq 入门(集群安装篇)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

简介:

RabbitMQ是采用Erlang语言实现AMQP（Advanced Message Queuing
Protocol，高级消息队列协议）的消息中间件，它最初起源于金融系统，用于在分布式系统中存储转发消息。
MQ全称为Message Queue,
消息队列（MQ）是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信，直接调用通常是用于诸如远程过程调用的技术。排队指的是应用程序通过
队列来通信。队列的使用除去了接收和发送应用程序同时执行的要求。其中较为成熟的MQ产品有IBM
WEBSPHERE MQ等等。
RabbitMQ是目前非常热门的一款消息中间件，很多行业都在使用这个消息中间件，RabbitMQ凭借其高可靠、易扩展、高可用及丰富的功能特性收到很多人的青睐。

1.2.1 软件下载：
''''''''''''''''

下载\ `Erlang <https://pan.baidu.com/s/1QLR4OelHXfMGLYywD13ydw>`__
百度云提取码: 864m

下载\ `Rabbitmq
源码包 <https://pan.baidu.com/s/15xSC2qo6-0Jh6FYnXkA4Yw>`__
百度云提取码: qwih

`点击此处跳转到官方网站下载最新版本 <https://www.rabbitmq.com/>`__

1.2.2 安装环境:
'''''''''''''''

node1 172.31.8.8 system Centos 7 —- node3 172.31.8.107 system Centos 7

node2 172.31.8.13 system Centos 7 —- node4 172.31.8.75 system Centos 7

软件版本：

Erlang 21.3

rabbitmq 3.7.7

单点安装(以下操作需要在其他四个节点重复执行)

分别编辑四台机器的/etc/hosts 文件，增加以下内容

172.31.8.8 node1

172.31.8.13 node2

172.31.8.107 node3

172.31.8.75 node4

安装依赖包

::

   yum install -y *epel* gcc-c++ unixODBC unixODBC-devel openssl-devel ncurses-devel

编译安装 Erlang

::

   [root@node1 ~]# tar xf otp_src_21.3.tar.gz
   [root@node1 ~]# cd otp_src_21.3
   [root@node1 otp_src_21.3]# ./configure --prefix=/usr/local/bin/erlang --without-javac
   [root@node1 otp_src_21.3]# make && make install
   [root@node1 otp_src_21.3]# echo "export PATH=$PATH:/usr/local/bin/erlang/bin:/usr/local/bin/rabbitmq_server-3.6.5/sbin" >> /etc/profile
   [root@node1 otp_src_21.3]# source /etc/profile

查看erlang 是否安装成功，出现以下输出即证明erlang已经安装成功
|erlang版本.png|

安装Rabbitmq

::

   [root@node1 ~]# tar xf rabbitmq-server-generic-unix-3.7.7.tar
   [root@node2 ~]# mv rabbitmq_server-3.7.7 /usr/local/rabbitmq-3.7.7
   [root@node2 ~]# cd /usr/local/rabbitmq-3.7.7/
   [root@node1 ~]# echo "export PATH=$PATH:/usr/local/rabbitmq-3.7.7/sbin" >> /etc/profile
   [root@node1 ~]# source /etc/profile

   [root@node1 ~]# rabbitmq-plugins enable rabbitmq_management   ---打开管理页面插件
   [root@node1 ~]# rabbitmq-server -detached  --后台启动服务
   [root@node1 ~]# rabbitmqctl add_user  admin 123456  --增加用户名admin，密码123456
   [root@node1 ~]# rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*" 
   [root@node1 ~]# rabbitmqctl set_user_tags admin administrator --设置用户admin为管理员

打开web页面，出现以下页面即安装成功

node1 |rabbitmqnode1.png| node2 |rabbitmqnode2.png|

1.2.3 部署集群:
'''''''''''''''

修改 .erlang.cookie文件，node1、node4节点内容改为一致

::

   [root@node1 ~]# chmod 400 .erlang.cookie  --设置.erlang.cookie文件权限,为了防止添加集群失败四个节点均需要调整为一致
   [root@node1 ~]# scp .erlang.cookie root@172.31.8.13:/root/
   [root@node1 ~]# scp .erlang.cookie root@172.31.8.107:/root/
   [root@node1 ~]# scp .erlang.cookie root@172.31.8.75:/root/

添加集群失败报错示例: 出现此信息可根据提示进行排查 |报错示例.png|
替换完.erlang.cookie文件，需要重启各个节点的rabbitmq服务

::

   node1
   [root@node1 ~]# kill -9 PID
   [root@node1 ~]# rabbitmq-server -detached
   node2
   [root@node2 ~]# kill -9 PID
   [root@node2 ~]# rabbitmq-server -detached
   node3
   [root@node3 ~]# kill -9 PID
   [root@node3 ~]# rabbitmq-server -detached
   node4
   [root@node4 ~]# kill -9 PID
   [root@node5 ~]# rabbitmq-server -detached

添加节点到集群，将node1节点作为主节点

在 node2 节点执行以下命令：

::

   [root@node2 ~]# rabbitmqctl stop_app
   [root@node4 ~]# rabbitmqctl join_cluster rabbit@node1  --默认为disc节点，如果需要指定节点角色，可以添加--ram/--disc参数
   [root@node4 ~]# rabbitmqctl start_app
   [root@node4 ~]# rabbitmqctl cluster_status

以上命令在node3、4节点重复执行

执行结果: |添加集群.png| 打开web管理页面查看状态 |查看集群.png|

到此Rabbitmq集群已经安装完毕，四个节点均已正常运行

1.2.4 集群管理
''''''''''''''

假设Rabbitmq-node2节点需要退出集群

在node2节点执行：

::

   rabbitmqctl stop_app   --停止rabbitmq服务
   rabbitmqctl reset --将RabbitMQ node还原到最初状态.包括从所在群集中删除此node,从管理数据库中删除所有配置数据，如已配置的用户和虚拟主机，以及删除所有持久化消息.
   rabbitmqctl start_app 

在主节点（node1）执行

::

   rabbitmqctl forget_cluster_node rabbit@node2 --此命令会从集群中删除rabbit@node2节点.

修改node2、node4节点为内存节点，在node2、4节点执行:

::

   [root@node2 ~]# rabbitmqctl stop_app   --停止rabbitmq服务
   [root@node2 ~]# rabbitmqctl change_cluster_node_type ram    --修改节点为内存节点
   [root@node2 ~]# rabbitmqctl start_app    --启动服务
   [root@node2 ~]# rabbitmqctl cluster_status --查看集群状态

执行结果可以看到node2节点已经修改为RAM节点，disc节点为node1、3、4
|node2RAM.png| 再修改node4节点： |node4RAM.png|
node2、node4节点已经成功修改为内存节点，现在集群就是双内存、双硬盘节点，从控制台查看更为直观:
|web控制台.png|

更多管理命令可参考文档:

https://blog.csdn.net/wulex/article/details/64127224

`点击此处查看官方文档 <https://www.rabbitmq.com/clustering.html>`__

.. |erlang版本.png| image:: https://i.loli.net/2019/07/02/5d1b6526a7b9185754.png
.. |rabbitmqnode1.png| image:: https://i.loli.net/2019/07/03/5d1ca62e894fb35867.png
.. |rabbitmqnode2.png| image:: https://i.loli.net/2019/07/03/5d1ca66bb841381637.png
.. |报错示例.png| image:: https://i.loli.net/2019/07/03/5d1cb7afc3b9469683.png
.. |添加集群.png| image:: https://i.loli.net/2019/07/03/5d1cb507de47990688.png
.. |查看集群.png| image:: https://i.loli.net/2019/07/03/5d1cb57e207a876254.png
.. |node2RAM.png| image:: https://i.loli.net/2019/07/04/5d1d75112e7ab28091.png
.. |node4RAM.png| image:: https://i.loli.net/2019/07/04/5d1d7671ed99316232.png
.. |web控制台.png| image:: https://i.loli.net/2019/07/04/5d1d79643889841417.png
