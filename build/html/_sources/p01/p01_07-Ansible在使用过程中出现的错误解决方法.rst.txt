.. contents::
   :depth: 3
..

1.7 Ansible在使用过程中出现的错误解决方法
=========================================

1.7.1 安装完成后允许命令出错
----------------------------

::

   Traceback (most recent call last):
   File "/usr/bin/ansible", line 197, in <module>
           (runner, results) = cli.run(options, args)
   File "/usr/bin/ansible", line 163, in run
           extra_vars=extra_vars,
   File "/usr/lib/python2.6/site-packages/ansible/runner/__init__.py", line 233, in __init__
           cmd = subprocess.Popen(['ssh','-o','ControlPersist'], stdout=subprocess.PIPE, stderr=subprocess.PIPE)
   File "/usr/lib64/python2.6/subprocess.py", line 639, in __init__
           errread, errwrite)
   File "/usr/lib64/python2.6/subprocess.py", line 1228, in _execute_child
           raise child_exception
   OSError: [Errno 2] No such file or directory

解决办法

::

   yum -y install openssh-clients

1.7.2 出现Error: ansible requires a json module, none found!
------------------------------------------------------------

::

   SSH password:
   10.0.1.110 | FAILED >> {
   "failed": true,
   "msg": "Error: ansible requires a json module, nonefound!",
   "parsed": false
   }

解决办法

python版本过低，可以升级python或者python-simplejson

1.7.3 安装完成后链接客户端报错（配图为我在使用ansible推送文件到客户端的时候遇到的，这个客户端是第一次推送）
-----------------------------------------------------------------------------------------------------------

::

   FAILED => Using a SSH password insteadof a key is not possible because Host Key checking is enabled and sshpass doesnot support this.  Please add this host'sfingerprint to your known_hosts file to manage this host.

解决办法：

在ansible 服务器上使用ssh 登陆下/etc/ansible/hosts
里面配置的服务器。然后再次使用ansible 去管理就不会报上面的错误了！

但这样大批量登陆就麻烦来。因为默认ansible是使用key验证的，如果使用密码登陆的服务器，使用ansible的话，要不修改ansible.cfg配置文件

的ask_pass = True给取消注释，要不就在运行命令时候加上-k，这个意思是-k,
–ask-pass ask for SSH password。再修改以下参数即可：

::

   host_key_checking= False

1.7.4 如果客户端不在know_hosts里将会报错
----------------------------------------

::

   paramiko: The authenticity of host '192.168.24.15'can't be established.

   The ssh-rsa key fingerprint is397c139fd4b0d763fcffaee346a4bf6b.

   Are you sure you want to continueconnecting (yes/no)?

解决办法

::

   需要修改 ansible.cfg 配置文件中的  #host_key_checking= False 参数取消注释

1.7.5 出现FAILED => FAILED: not a valid DSA private key file
------------------------------------------------------------

解决办法：

需要你在最后命令内添加参数-k

1.7.6 openssh升级后无法登录报错
-------------------------------

::

   PAM unable todlopen(/lib64/security/pam_stack.so): /lib64/security/pam_stack.so: cannot openshared object

   file: No such file or directory

解决方法：

sshrpm 升级后会修改/etc/pam.d/sshd
文件。需要升级前备份此文件最后还原即可登录。

1.7.7 第一次系统初始化运行生成本机ansible用户key时报错
------------------------------------------------------

::

   failed: [127.0.0.1] =>{"checksum": "f5f2f20fc0774be961fffb951a50023e31abe920","failed": true}

   msg: Aborting, target uses selinux but pythonbindings (libselinux-python) aren't installed!

   FATAL: all hosts have already failed –aborting

解决办法

::

   yum -y install libselinux-python

`参考自：http://blog.csdn.net/longxibendi/article/details/46989735 <http://blog.csdn.net/longxibendi/article/details/46989735>`__
