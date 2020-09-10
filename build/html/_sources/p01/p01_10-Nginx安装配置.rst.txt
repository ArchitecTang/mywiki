.. contents::
   :depth: 3
..

1.10 Nginx 安装配置
===================

1.10.1 Nginx 是什么？
---------------------

nginx是俄罗斯人 Igor
Sysoev为俄罗斯访问量第二的Rambler.ru站点开发的一个十分轻量级的HTTP服务器。它是一个高性能的HTTP和反向代理服务器，同时也可以作为IMAP/POP3/SMTP的代理服务器。nginx使用的是BSD许可。

Nginx
以事件驱动的方式编写，所以有非常好的性能，同时也是一个非常高效的反向代理、负载平衡。

Nginx 因为它的稳定性、丰富的模块库、灵活的配置和低系统资源的消耗而闻名。

1.10.2 核心特点：
-----------------

高并发请求的同时保持高效的服务

热部署

低内存消耗

处理响应请求很快

具有很高的可靠性

nginx可以实现高效的反向代理、负载均衡。

1.10.3 安装Nginx
----------------

安装编译工具及依赖

::

   yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel pcre pcre-devel

安装nginx

下载nginx

::

   wget http://nginx.org/download/nginx-1.8.1.tar.gz

解压安装

::

   tar -xf nginx-1.8.1.tar.gz -C /software/
   cd /software/ && mkdir nginx && cd nginx-1.8.1

编译，指定nginx 安装目录：/software/nginx

::

   ./configure --prefix=/software/nginx --with-http_ssl_module --with-http_stub_status_module --with-threads --with-file-aio

安装

::

   make && make install

启动nginx

::

   /software/nginx/sbin/nginx -t
   nginx: the configuration file /software/nginx/conf/nginx.conf syntax is ok
   nginx: configuration file /software/nginx/conf/nginx.conf test is successful
   /software/nginx/sbin/nginx
   ps -ef | grep nginx
   root     22917     1  0 22:01 ?        00:00:00 nginx: master process /software/nginx/sbin/nginx
   nobody   22918 22917  0 22:01 ?        00:00:00 nginx: worker process
   root     22920 21803  0 22:01 pts/1    00:00:00 grep --color=auto nginx

1.10.4 访问测试
---------------

http://IPaddr,出现以下页面nginx就已经安装成功 |安装测试.png|

1.10.5 创建nginx 运行用户
-------------------------

::

   /usr/sbin/groupadd www
   /usr/sbin/useradd -g www www

1.10.6 配置nginx.conf
---------------------

::

   user  www www;    --- 指定nginx启动用户
   worker_processes  1;

   error_log  logs/nginx_error.log crit; ---配置日志存储位置和级别
   pid        logs/nginx.pid;         --- 设置nginx.pid 文件存储位置

   events {
       worker_connections  65535;
   }


   http {
       include       mime.types;
       default_type  application/octet-stream;

       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                       '$status $body_bytes_sent "$http_referer" '
                       '"$http_user_agent" "$http_x_forwarded_for"';

       sendfile        on;
       #keepalive_timeout  0;
       keepalive_timeout  65;


       server {
           listen       80;
           server_name   172.31.8.11;

           #charset koi8-r;

           access_log  logs/172.31.8.11_access.log  main;   --- 设置日志格式
           access_log  logs/172.31.8.11_json_access.log main;  --- 设置json 日志格式
           location / {
               root   html;
               index  index.html index.htm;
           }

           #error_page  404              /404.html;

           # redirect server error pages to the static page /50x.html
           #
           error_page   500 502 503 504  /50x.html;
           location = /50x.html {
               root   html;
           }

           # proxy the PHP scripts to Apache listening on 127.0.0.1:80
           #
           #location ~ \.php$ {
           #    proxy_pass   http://127.0.0.1;
           #}

           # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
           #
           #location ~ \.php$ {
           #    root           html;
           #    fastcgi_pass   127.0.0.1:9000;
           #    fastcgi_index  index.php;
           #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
           #    include        fastcgi_params;
           #}

           # deny access to .htaccess files, if Apache's document root
           # concurs with nginx's one
           #
           #location ~ /\.ht {
           #    deny  all;
           #}
       }


       # another virtual host using mix of IP-, name-, and port-based configuration
       #
       #server {
       #    listen       8000;
       #    listen       somename:8080;
       #    server_name  somename  alias  another.alias;

       #    location / {
       #        root   html;
       #        index  index.html index.htm;
       #    }
       #}


       # HTTPS server
       #
       #server {
       #    listen       443 ssl;
       #    server_name  localhost;

       #    ssl_certificate      cert.pem;
       #    ssl_certificate_key  cert.key;

       #    ssl_session_cache    shared:SSL:1m;
       #    ssl_session_timeout  5m;

       #    ssl_ciphers  HIGH:!aNULL:!MD5;
       #    ssl_prefer_server_ciphers  on;

       #    location / {
       #        root   html;
       #        index  index.html index.htm;
       #    }
       #}

   }

   /software/nginx/sbin/nginx -t
   nginx: the configuration file /software/nginx/conf/nginx.conf syntax is ok
   nginx: configuration file /software/nginx/conf/nginx.conf test is successful
   /software/nginx/sbin/nginx -s reload

   相关命令：
   /usr/local/webserver/nginx/sbin/nginx -s reload            --  重新载入配置文件
   /usr/local/webserver/nginx/sbin/nginx -s reopen            --  重启 Nginx
   /usr/local/webserver/nginx/sbin/nginx -s stop              --  停止 Nginx

以上就是Nginx安装后的简单配置

.. |安装测试.png| image:: https://i.loli.net/2019/07/26/5d3b06de61f2657662.png
