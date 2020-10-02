.. contents::
   :depth: 3
..

8.1 mariadb数据库安装初始化
===========================

安装mariadb数据库

::

   yum -y install mariadb mariadb-server

启动数据库：

::

   systemctl start mariadb
   systemctl enable mariadb

初始化数据库：

::

   [root@bogon ~]# mysql_secure_installation

   NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
       SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

   In order to log into MariaDB to secure it, we'll need the current
   password for the root user.  If you've just installed MariaDB, and
   you haven't set the root password yet, the password will be blank,
   so you should just press enter here.

   Enter current password for root (enter for none):     #新安装数据库无密码直接回车
   OK, successfully used password, moving on...

   Setting the root password ensures that nobody can log into the MariaDB
   root user without the proper authorisation.

   Set root password? [Y/n] y    #设置密码
   New password:     #输入密码
   Re-enter new password:    #确认密码
   Password updated successfully!
   Reloading privilege tables..
   ... Success!


   By default, a MariaDB installation has an anonymous user, allowing anyone
   to log into MariaDB without having to have a user account created for
   them.  This is intended only for testing, and to make the installation
   go a bit smoother.  You should remove them before moving into a
   production environment.

   Remove anonymous users? [Y/n] y    #删除匿名用户
   ... Success!

   Normally, root should only be allowed to connect from 'localhost'.  This
   ensures that someone cannot guess at the root password from the network.

   Disallow root login remotely? [Y/n] y    #禁止root远程登陆
   ... Success!

   By default, MariaDB comes with a database named 'test' that anyone can
   access.  This is also intended only for testing, and should be removed
   before moving into a production environment.

   Remove test database and access to it? [Y/n] y    #删除test数据库和对此数据库的访问权限
   - Dropping test database...
   ... Success!
   - Removing privileges on test database...
   ... Success!

   Reloading the privilege tables will ensure that all changes made so far
   will take effect immediately.

   Reload privilege tables now? [Y/n] y  #立即刷新权限
   ... Success!

   Cleaning up...

   All done!  If you've completed all of the above steps, your MariaDB
   installation should now be secure.

   Thanks for using MariaDB!
