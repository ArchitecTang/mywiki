.. contents::
   :depth: 3
..

1.5 clamav Linux-杀毒软件
=========================

1.5.1 clamav简介
----------------

ClamAV是一个C语言开发的开源病毒扫描工具用于检测木马/病毒/恶意软件等。可以在线更新病毒库，Linux系统的病毒较少，但是并不意味着病毒免疫，尤其是对于诸如邮件或者归档文件中夹杂的病毒往往更加难以防范，而ClamAV则能起到不少作用。
`官网上关于clamav的解释很简单，就是一款用于木马、病毒、恶意软件以及其他恶意威胁的杀毒软件(点击此处跳转到官网) <http://www.clamav.net>`__

`官网下载地址 <http://www.clamav.net/downloads>`__

更多特性：

::

   1、主要用途    邮件网关的病毒扫描，内建支持多种邮件格式
   2、高性能      提供多线程的扫描进程
   3、命令行      提供密令行扫描方式
   4、扫描对象    可以对要发送的邮件或者文件进行扫描
   5、文件格式    支持多种文件格式
   6、病毒库更新频度   一天多次病毒库的更新
   7、归档文件        支持扫描多种归档文件,比如Zip, RAR, Dmg, Tar, Gzip, Bzip2, OLE2, Cabinet, CHM, BinHex, SIS等
   8、文档           支持流行的文档文件，比如： MS Office文件，MacOffice文件, HTML, Flash, RTF，PDF

1.5.2 安装clamav
----------------

CENTOS/RHEL

::

   yum -y install clamav

Ubuntu/Debian

::

   apt-get install clamav  

安装杀毒软件

::

    yum -y install clamav

更新数据库

::

   freshclam

|更新数据库.png| 病毒库文件

::

   /var/lib/clamav/daily.cvd
   /var/lib/clamav/main.cvd

添加定时任务，设置自动更新:每两个小时自动更新

::

   crontab -e
   0  */2 * * * root /usr/share/clamav/freshclam-sleep

1.5.3 使用
----------

::

   clamscan是病毒扫描命令，以下是一些常用参数：
   clamscan      ---不加参数的使用：扫描当前目录下的文件
   clamscan -V   ---查看clamAV的版本
   clamscan -r   ---递归扫描子文件夹
   clamscan -i   ---仅仅显示被感染的文件
   clamscan -o   ---跳过显示状态ok的文件
   clamscan --remove       ---检测到有病毒时，直接删除
   clamscan --no-summary   ---不显示统计信息
   clamscan -l scan.log    ---将扫描日志写入scan.log文件
                                            ---以上命令都可以在末尾添加文件夹，来扫描指定目录，如
   clamscan -r -i /home  --remove  -l scan.log   ---递归扫描/home/目录下的所有文件，只显示病毒文件，并同时删除

示例：

::

   clamscan -l scan.log

.. figure:: https://i.loli.net/2019/07/20/5d32e3b169d8085122.png
   :alt: scanlog.png

执行完以后会有详细的扫描信息

.. figure:: https://i.loli.net/2019/07/20/5d32e7c7a53a630074.png
   :alt: 扫描结果.png

.. |更新数据库.png| image:: https://i.loli.net/2019/07/20/5d32e1b20615863780.png
