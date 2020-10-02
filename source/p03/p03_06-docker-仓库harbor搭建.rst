.. contents::
   :depth: 3
..

3.6 docker 仓库harbor搭建
=========================

3.6.1 初始化
------------

在正式安装harbor之前，需要对OS环境进行初始化，升级OS内核，具体内核升级步骤可以自己了解下

3.6.1.1 配置安装源
~~~~~~~~~~~~~~~~~~

因为需要安装相关的以来软件，所以需要安装yum源，安装前先将原先的yum备份

::

   [root@harbor yum.repos.d]# mv CentOS-Base.repo CentOS-Base.repo.bak
   [root@harbor yum.repos.d]# vim CentOS-Base.repo
   [base]
   name=CentOS-$releasever – Base – 163.com
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
   baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
   gpgcheck=1
   gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
   #released updates
   [updates]
   name=CentOS-$releasever – Updates – 163.com
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
   baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
   gpgcheck=1
   gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
   #additional packages that may be useful
   [extras]
   name=CentOS-$releasever – Extras – 163.com
   #mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
   baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
   gpgcheck=1
   gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
   #additional packages that extend functionality of existing packages
   [centosplus]
   name=CentOS-$releasever – Plus – 163.com
   baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
   gpgcheck=1
   enabled=0
   gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

|epel.png| 安装阿里云epel源

::

   wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

|安装epel.png| 安装docker

::

   yum -y install docker

启动docker并查看docker版本 |安装docker.png|

3.6.1.2 安装docker-compose
~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   yum -y install certbot libevent-devel gcc libffi-devel python-devel openssl-devel python2-pip

使用pip的方式进行安装，命令如下

::

   pip install -U docker-compose

|dockercompose.png| 查看安装的版本

::

   docker-compose version

.. figure:: https://i.loli.net/2019/07/12/5d2851ed6e9fd60328.png
   :alt: 查看安装.png

3.6.2、下载安装harbor，选择自己需要的版本
-----------------------------------------

官网地址：\ http://harbor.orientsoft.cn/ |Harbor.png|

3.6.2.1 修改Harbor配置文件，修改服务地址
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

   [root@harbor ~]# ls
   anaconda-ks.cfg  harbor-offline-installer-v1.4.0.tgz
   [root@harbor ~]# tar xf harbor-offline-installer-v1.4.0.tgz 
   [root@harbor ~]# ls
   anaconda-ks.cfg  harbor  harbor-offline-installer-v1.4.0.tgz
   [root@harbor ~]# mv harbor /usr/local/
   [root@harbor ~]# vim harbor.cfg

修改hostname为主机的IP（没有域名的情况下） |修改hostsname.png|

3.6.2.2 修改harbor的默认admin密码（默认密码为Harbor12345）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: https://i.loli.net/2019/07/12/5d28535b2963447635.png
   :alt: 修改Harbor默认密码.png

安装harbor |安装.png|
步骤一会下载相关的docker镜像，这个过程根据各自的网络情况不同花费的时间也不同，相关的docker镜像如下：

::

   docker images

|dockerimages.png|
查看容器，可以看到没有Notary与Clair相关服务；也可使用“docker ps”；

“docker-compose ps”需要在“docker-compose.yml”文件所在目录执行相关操作

::

   [root@harbor harbor]# docker-compose ps

.. figure:: https://i.loli.net/2019/07/12/5d2853fe445b964442.png
   :alt: dockerps.png

3.6.2.3 安装后配置
~~~~~~~~~~~~~~~~~~

访问harbor ui
（注意服务器的防火墙和selinux，可以关闭或者放行相关端口）admin/默认密码
|登陆.png| harbor安装完成 |登陆2.png|

harbor 使用docker login报错的问题 |登陆报错.png|
从harbor安装文档中可以看到\ https://github.com/vmware/harbor/blob/master/docs/installation_guide.md
|报错排查.png|
在Harbor主机和客户机都对这个文件进行设置/etc/docker/daemon.json：

::

   { "insecure-registries":["192.168.33.10"] }

.. figure:: https://i.loli.net/2019/07/12/5d28551d6227692895.png
   :alt: 报错解决.png

3.6.3 简单使用
--------------

向harbor上推拉镜象

给docker.io/tomcat这个镜像打上tag

::

   [root@harbor ~]# docker tag docker.io/tomcat 172.31.8.25/library/tomcat2

|镜像推送.png| 推送至harbor

::

   [root@harbor ~]# docker push 172.31.8.25/library/tomcat2

.. figure:: https://i.loli.net/2019/07/12/5d28558b0c6d544922.png
   :alt: 推送.png

.. |epel.png| image:: https://i.loli.net/2019/07/12/5d2850a00566f24895.png
.. |安装epel.png| image:: https://i.loli.net/2019/07/12/5d28510e95a7b40843.png
.. |安装docker.png| image:: https://i.loli.net/2019/07/12/5d2851689228260891.png
.. |dockercompose.png| image:: https://i.loli.net/2019/07/12/5d2851bd5d99095382.png
.. |Harbor.png| image:: https://i.loli.net/2019/07/12/5d2852421d18d60335.png
.. |修改hostsname.png| image:: https://i.loli.net/2019/07/12/5d2852fc2cd6988291.png
.. |安装.png| image:: https://i.loli.net/2019/07/12/5d2853818460889055.png
.. |dockerimages.png| image:: https://i.loli.net/2019/07/12/5d2853b91656a11429.png
.. |登陆.png| image:: https://i.loli.net/2019/07/12/5d28543a3a18690521.png
.. |登陆2.png| image:: https://i.loli.net/2019/07/12/5d285467ed7a348734.png
.. |登陆报错.png| image:: https://i.loli.net/2019/07/12/5d28549b523cf57301.png
.. |报错排查.png| image:: https://i.loli.net/2019/07/12/5d2854e03c93357954.png
.. |镜像推送.png| image:: https://i.loli.net/2019/07/12/5d28555f2525562448.png
