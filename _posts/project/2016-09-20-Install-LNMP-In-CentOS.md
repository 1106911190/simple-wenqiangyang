---
layout: post
title: Cent OS 下安装 LNMP
category: project
description: 真正的开发环境一般都是用 centos ，安装配置一下 Nginx ， MariaDB 和 PHP
---

LNMP 是 Linux、Nginx、MySQL(MariaDB) 和 PHP 的缩写，这个组合是最常见的 WEB 服务器的运行环境之一，接下来都将在 Cent OS 7 上进行。

## Nginx

> 好像 Cent OS 的官方源里是没有 Nginx 的，需要先自行安装一个扩展包 <br>
> `sudo yum install epel-release`

```
sudo yum install nginx
```

安装完成后，配置文件在 `/etc/nginx` ，根目录在 `/usr/share/nginx/html` ,可以使用

```
sudo systemctl start nginx
```

或

```
sudo service nginx stop
```

来开启 Nginx 服务，关闭就将 `start` 改为 `stop` 即可，重启就是 `restart` 。

如果防火墙关闭，则此时查看 Centos 的 80 端口即可看到 Nginx 启动界面，如果是本机则查看 `127.0.0.1` 或者是 `localhost` ，如果是服务器，则查看外网 ip。

![Centos_Nginx](/images/Centos_Nginx.png)

可以通过

```
sudo systemctl status firewalld
```

或

```
sudo service firewalld status
```

查看最后一行是否为 inactive ，如下即为仍未关闭。

```
[root@VM_15_35_centos ~]# sudo service firewalld status
Redirecting to /bin/systemctl status  firewalld.service
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since 二 2016-10-11 08:22:20 UTC; 3 days ago
 Main PID: 447 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─447 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid
```

这样即是已关闭 

```
[root@VM_1_214_centos nginx]# sudo service firewalld status
Redirecting to /bin/systemctl status  firewalld.service
* firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
```
如果显示 active ，则需调整防火墙策略或者关闭防火墙。

在 `/etc/firewalld/zones/public.xml` 中，在 `zone` 一节中加入

```
<zone>
    ...
    <service name="nginx"/>
<zone>
```

然后重新加载 firewalld 服务

```
sudo systemctl reload firewalld
```

或者是关闭防火墙 (并不推荐)

```
sudo service firewalld stop
```

或者是永久关闭防火墙 (并不推荐)

```
sudo chkconfig --level 123456 firewalld off
```

只是打开相关的端口就好了嘛，永久打开 firewalld http 和 HTTPS 相关端口

```
sudo firewall-cmd --permanent --zone=public --add-service=http 
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

设置 Nginx 开机启动

```
sudo systemctl enable nginx.service
```

## MariaDB

MariaDB是MySQL的一个分支，主要由开源社区进行维护和升级，而MySQL被Oracle收购以后，发展较慢。在CentOS 7的软件仓库中，将MySQL更替为了MariaDB。

```
sudo yum install mariadb-server
```

安装完成之后，启动 MariaDB

```
sudo systemctl start mariadb
```

启动之后即可进入 MariaDB ，命令行和其他操作与 MySQL 基本一致，甚至可以用 MySQL 的客户端连接 MariaDB ，MariaDB 默认 root 密码为空。

```
[root@VM_15_35_centos ~]# mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.47-MariaDB MariaDB Server

Copyright (c) 2000, 2015, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit
Bye

```

MariaDB默认root密码为空，我们需要设置一下，执行脚本：

```
sudo /usr/bin/mysql_secure_installation
```

这个脚本会经过一些列的交互问答来进行MariaDB的安全设置。

首先提示输入当前的root密码：

```
Enter current password for root (enter for none):
```

初始root密码为空，我们直接敲回车进行下一步。

```
Set root password? [Y/n]
```

设置root密码，默认选项为Yes，我们直接回车，提示输入密码，在这里设置您的MariaDB的root账户密码。

```
Remove anonymous users? [Y/n]
```

是否移除匿名用户，默认选项为Yes，建议按默认设置，回车继续。

```
Disallow root login remotely? [Y/n]
```

是否禁止root用户远程登录？如果您只在本机内访问MariaDB，建议按默认设置，回车继续。 如果您还有其他云主机需要使用root账号访问该数据库，则需要选择n。

```
Remove test database and access to it? [Y/n]
```

是否删除测试用的数据库和权限？ 建议按照默认设置，回车继续。

```
Reload privilege tables now? [Y/n]
```

是否重新加载权限表？因为我们上面更新了root的密码，这里需要重新加载，回车。

完成后你会看到Success!的提示，MariaDB的安全设置已经完成。我们可以使用以下命令登录MariaDB：

```
mysql -u root -p
```

按提示输入root密码，就会进入MariaDB的交互界面，说明已经安装成功。

最后我们将MariaDB设置为开机启动。

```
sudo systemctl enable mariadb
```

## PHP

```
sudo yum install php-fpm php-mysql
```

安装完成之后，启动 php-fpm。

```
sudo systemctl start php-fpm
```

将 php-fpm 设为开机自启动。

```
sudo systemctl enable php-fpm
```

现在 PHP 已经安装好了，然后再配置一下 Nginx,在 `/etc/nginx` 中，将 `nginx.conf` 重名为为 `nginx.conf.bak` ，并把 `nginx.conf.default` 重命名为 `nginx.conf`，然后打开 `nginx.conf`

```
sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
sudo mv /etc/nginx/nginx.conf.default /etc/nginx/nginx.conf
sudo vim /etc/nginx/nginx.conf
```

然后先在此处加上 `index.php`

```
        ...
        location / {
            root   html;
            index  index.html index.htm index.php;
        }
        ...

```

并将此处的注释去掉，将 `root` 目录改为绝对路径，并将 fastcgi_param 改为相对路径。

```
server {
    ...
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
    ...
}
```

可以使用 `nginx -t` 检查配置文件是否书写正确。

然后重启 Nginx 服务

```
service nginx restart
```

此时如果在根目录 `/usr/share/nginx/html` 下创建一个名为 `info.php` 的文件，写入

```
<?php
    phpinfo();
?>
```

即可在网站看到站点的基本配置信息。

这样的话 PHP 是通过 127.0.0.1:9000 端口进行传递的，显得不太好，一般通用的办法是用 Unix 的管道 sock。

安装 PHP

```
yum install php
```

配置 `/etc/php.ini`

```
cgi.fix_pathinfo=0
```

配置 `/etc/php-fpm.d/www.conf`

```
user = nginx
group = nginx
listen = /var/run/php-fpm/php-fpm.sock
```

配置 `/etc/nginx/nginx.conf`

```
    location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```

最后提供一个参考的 `nginx.conf`

```
user nginx;
worker_processes auto;

error_log /var/log/nginx/error.log;
pid       /var/run/nginx.pid;

include /etc/nginx/conf.d/*.conf;

events {
    worker_connections 1024;
}

http {
    gzip  on;
    gzip_disable "MSIE [1-6].(?!.*SV1)";

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root   /usr/share/nginx/html;

        location / {
            index  index.html index.htm index.php;
        }

        location ~ \.php$ {
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /usr/share/nginx/html$fastcgi_script_name;
            include        fastcgi_params;
        }

        error_page 404 /404.html;

        error_page 400 401 403 405 /40x.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}

```

参考链接

[在CentOS 7上搭建LNMP环境](https://mos.meituan.com/library/18/how-to-install-lnmp-on-centos7/)

[How To Install Linux, Nginx, MySQL, PHP (LEMP) stack On CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-7)