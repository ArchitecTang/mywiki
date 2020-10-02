.. contents::
   :depth: 3
..

6.2 批量ping存活主机
====================

创建一个ip.txt文件，把需要测试IP地址写入文档

.. figure:: https://i.loli.net/2019/05/17/5cde3e1a07bca53225.png
   :alt: 1.png

   1.png

.. raw:: html

   <!-- more -->

创建一个ping.sh 的shell 脚本，并修改ip.txt 文件路径

::

   #!/bin/bash
   for ips in `cat /root/manager/script/ip.txt`
   do
           result=`ping -w 2 -c 3 ${ips} | grep packet | awk -F" " '{print $6}'| awk -F"%" '{print $1}'| awk -F' ' '{print $1}'`
           if [ $result -eq 0 ]; then
                   echo ""${ips}" is ok !"
           else
                   echo ""${ips}" is not connected ....."
           fi

   done

给脚本执行权限 chmod +x ping.sh

执行脚本

::

   [root@k8s-node1 script]# ./ping.sh 
   172.31.8.101 is ok !
   172.31.8.192 is ok !
   172.31.8.42 is ok !
   172.31.8.176 is ok !
   172.31.8.45 is ok !
   [root@k8s-node1 script]# 

