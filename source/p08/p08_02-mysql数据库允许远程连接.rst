.. contents::
   :depth: 3
..

8.2 mysql数据库允许远程连接
===========================

::

   use mysql    /选择mysql库
   select  User,authentication_string,Host from user;       /查看用户信息表
   GRANT ALL PRIVILEGES ON *.* TO 'username'@'IP' IDENTIFIED BY 'password';     /开放权限到指定IP，允许这个IP使用root账户远程连接
   flush privileges;         /刷新权限，立即生效
