---
title: Homebrew 安装 PHP环境
date: 2015-08-26 22:11:50
tags: [homebrew, php]
---
>homebrew 是 os x 上的包管理控制器类似于 apt-get 、yum 官网是 http://brew.sh

## 安装方式 Homebrew

打开 终端 执行

```bash
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装完成后更新 brew 源

```bash
brew update #更新brew
```

### brew 常用命令

```bash
brew install APPNAME #安装应用

brew search APPNAME # 搜索已有应用

brew tap  GIHUB/user # 安装扩展

brew uncap GITHUB/user #移除扩展

brew remove APPNAME #移除应用

brew upgrade APPNAME #更新应用

brew options APPNAME #查看安装信息

brew info APPNAME #查看应用信息
```

<!-- more -->
 

## 安装 mysql

```bash
brew install homebrew/versions/mysql56
```


### MySQL开机启动：

```bash
ln -sfv /usr/local/opt/mysql56/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql56.plist
```

### 安装完成之后开启MySQL安全机制：

```bash
/usr/local/opt/mysql56/bin/mysql_secure_installation
```
 

## 安装 PHP 5.6

### 添加 PHP 扩展

```bash
brew tap homebrew/dupes
brew tap josegonzalez/homebrew-php
```

>可以使用brew options php56命令来查看安装选项

```bash
brew install php56 --with-fpm --with-debug --with-gmp --with-imap --with-tidy --with-mysql --with-libmysql --with-homebrew-curl --with-homebrew-openssl --without-snmp
```

### 安装 PHP 扩展

```bash
brew install php56-apcu\
 php56-gearman\
 php56-geoip\
 php56-gmagick\
 php56-imagick\
 php56-intl\
 php56-mcrypt\
 php56-redis\
 php56-sphinx\
 php56-mongo\
 php56-mongodb\
 php56-xdebug;
```

>由于Mac自带了php和php-fpm，因此需要添加系统环境变量PATH来替代自带PHP版本。

```bash
echo 'export PATH="$(brew --prefix php56)/bin:$PATH"' >> ~/.bash_profile  #for php
echo 'export PATH="$(brew --prefix php56)/sbin:$PATH"' >> ~/.bash_profile  #for php-fpm
echo 'export PATH="/usr/local/bin:/usr/local/sbin:$PATH"' >> ~/.bash_profile #for other brew install soft
source ~/.bash_profile
```
 

### 测试

```bash
php -v    
PHP 5.6.12 (cli) (built: Aug 25 2015 06:24:07) (DEBUG)
Copyright (c) 1997-2015 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2015 Zend Technologies
```

>修改php-fpm配置文件，vim /usr/local/etc/php/5.6/php-fpm.conf，找到pid相关大概在25行，去掉注释 pid = run/php-fpm.pid, 那么php-fpm的pid文件就会自动产生在/usr/local/var/run/php-fpm.pid，下面要安装的Nginx pid文件也放在这里。

### 测试php-fpm配置

```bash
php-fpm -t
php-fpm -c /usr/local/etc/php/5.6/php.ini -y /usr/local/etc/php/5.6/php-fpm.conf -t
```

### 启动php-fpm

```bash
php-fpm -D
php-fpm -c /usr/local/etc/php/5.6/php.ini -y /usr/local/etc/php/5.6/php-fpm.conf -D
```

### 关闭php-fpm

```bash
kill -INT `cat /usr/local/var/run/php-fpm.pid`
```

### 重启php-fpm

```bash
kill -USR2 `cat /usr/local/var/run/php-fpm.pid`
```

### 使用命令来启动php-fpm

```bash
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist
```

>启动php-fpm之后，确保它正常运行监听9000端口：

```bash
lsof -Pni4 | grep LISTEN | grep php
php-fpm   30907 XXX    9u  IPv4 0x9f5ee1b4361405d3      0t0  TCP 127.0.0.1:9000 (LISTEN)
```

### PHP-FPM开机启动：

```bash
ln -sfv /usr/local/opt/php56/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.php56.plist
```
 
## 安装php composer

```bash
brew install composer
```

### 检查一下情况

```bash
composer --version
Composer version 1.0.0-alpha8 2014-01-06 18:39:59
```
 

## 安装Nginx

```bash
brew install nginx --with-http_geoip_module
```


### 测试配置是否有语法错误
```bash
nginx -t
```

### 打开 nginx
```bash
sudo nginx
```

### 重新加载配置|重启|停止|退出 nginx
```bash
nginx -s reload|reopen|stop|quit
```

>也可以使用Mac的launchctl来启动|停止

```bash
launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
```

### Nginx开机启动

```bash
ln -sfv /usr/local/opt/nginx/*.plist ~/Library/LaunchAgents
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
```
  
Nginx监听80端口需要root权限执行，因此：


```bash
sudo chown root:wheel /usr/local/Cellar/nginx/1.6.0_1/bin/nginx
sudo chmod u+s /usr/local/Cellar/nginx/1.6.0_1/bin/nginx
```

## 创建 NGINX 配置
### 配置nginx.conf创建需要用到的目录：

```bash
mkdir -p /usr/local/var/logs/nginx
mkdir -p /usr/local/etc/nginx/sites-available
mkdir -p /usr/local/etc/nginx/sites-enabled
mkdir -p /usr/local/etc/nginx/conf.d
mkdir -p /usr/local/etc/nginx/ssl
sudo mkdir -p /var/www
sudo chown :staff /var/www
sudo chmod 775 /var/www
```

>创建配置文件

```bash
vim /usr/local/etc/nginx/nginx.conf 
```


```bash
worker_processes  1;

error_log   /usr/local/var/logs/nginx/error.log debug;


pid        /usr/local/var/run/nginx.pid;


events {
    worker_connections  256;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /usr/local/var/logs/access.log  main;

    sendfile        on;
    keepalive_timeout  65;
    port_in_redirect off;

    include /usr/local/etc/nginx/sites-enabled/*;
}
```


## 设置 nginx php-fpm 配置文件


>创建 php-fpm 配置

```bash
vim /usr/local/etc/nginx/conf.d/php-fpm
```

```bash
#proxy the php scripts to php-fpm
location ~ \.php$ {
    try_files                   $uri = 404;
    fastcgi_pass                127.0.0.1:9000;
    fastcgi_index               index.php;
    fastcgi_intercept_errors    on;
    include /usr/local/etc/nginx/fastcgi.conf;
}
 nginx虚拟主机准备工作

#创建 info.php index.html 404.html 403.html文件到 /var/www 下面
vi /var/www/info.php
vi /var/www/index.html
vi /var/www/403.html
vi /var/www/404.html
 创建默认虚拟主机defaultvim /usr/local/etc/nginx/sites-available/default输入：

server {
    listen       80;
    server_name  localhost;
    root         /var/www/;

    access_log  /usr/local/var/logs/nginx/default.access.log  main;

    location / {
        index  index.html index.htm index.php;
        autoindex   on;
        include     /usr/local/etc/nginx/conf.d/php-fpm;
    }

    location = /info {
        allow   127.0.0.1;
        deny    all;
        rewrite (.*) /.info.php;
    }

    error_page  404     /404.html;
    error_page  403     /403.html;
}
 创建ssl默认虚拟主机default-sslvim /usr/local/etc/nginx/sites-available/default-ssl输入：

server {
    listen       443;
    server_name  localhost;
    root       /var/www/;

    access_log  /usr/local/var/logs/nginx/default-ssl.access.log  main;

    ssl                  on;
    ssl_certificate      ssl/localhost.crt;
    ssl_certificate_key  ssl/localhost.key;

    ssl_session_timeout  5m;

    ssl_protocols  SSLv2 SSLv3 TLSv1;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers   on;

    location / {
        include   /usr/local/etc/nginx/conf.d/php-fpm;
    }

    location = /info {
        allow   127.0.0.1;
        deny    all;
        rewrite (.*) /.info.php;
    }

    error_page  404     /404.html;
    error_page  403     /403.html;
}
```
 

>设置SSL

```bash
mkdir -p /usr/local/etc/nginx/ssl
openssl req -new -newkey rsa:4096 -days 365 -nodes -x509 -subj "/C=US/ST=State/L=Town/O=Office/CN=localhost" -keyout /usr/local/etc/nginx/ssl/localhost.key -out /usr/local/etc/nginx/ssl/localhost.crt
```

### 创建虚拟主机软连接，开启虚拟主机

```bash
ln -sfv /usr/local/etc/nginx/sites-available/default /usr/local/etc/nginx/sites-enabled/default
ln -sfv /usr/local/etc/nginx/sites-available/default-ssl /usr/local/etc/nginx/sites-enabled/default-ssl
```

## 启动|停止Nginx

```bash
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
```
 

## 快捷服务控制命令
>为了后面管理方便，将命令 alias 下，vim ~/.bash_aliases 输入一下内容：

```bash
alias nginx.start='launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist'
alias nginx.stop='launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist'
alias nginx.restart='nginx.stop && nginx.start'
alias php-fpm.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist"
alias php-fpm.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.php56.plist"
alias php-fpm.restart='php-fpm.stop && php-fpm.start'
alias mysql.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist"
alias mysql.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist"
alias mysql.restart='mysql.stop && mysql.start'
alias redis.start="launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.redis.plist"
alias redis.stop="launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.redis.plist"
alias redis.restart='redis.stop && redis.start'
``` 

>让快捷命令生效
```bash
echo "[[ -f ~/.bash_aliases ]] && . ~/.bash_aliases" >> ~/.bash_profile     
source ~/.bash_profile
```