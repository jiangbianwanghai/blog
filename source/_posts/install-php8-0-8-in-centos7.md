---
title: 在Centos7下安装PHP8
date: 2021-07-08 18:09:05
tags: ["php8", "centos7", "源码安装"]
---
> 安装PHP的逻辑很简单。下载压缩包，解压，生成Makefile文件并安装

**系统环境如下：**

```
➜  ~ cat /proc/version
Linux version 3.10.0-327.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.3 20140911 (Red Hat 4.8.3-9) (GCC) ) #1 SMP Thu Nov 19 22:10:57 UTC 2015
➜  ~ cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)
```

安装php-8.0.8

```
wget https://www.php.net/distributions/php-8.0.8.tar.gz
tar -zxvf php-8.0.8.tar.gz
cd php-8.0.8
#编译Makefile并开启一些必要的插件
./configure --prefix=/usr/local/php-8.0.8 --with-mysqli --with-pdo-mysql --with-zlib --enable-simplexml --enable-xml --disable-rpath --enable-bcmath --enable-soap --with-curl --enable-fpm --with-fpm-user=nobody --with-fpm-group=nobody --enable-mbstring --enable-sockets --with-openssl --with-mhash --enable-opcache --disable-fileinfo –-with-readline
make && make install

#创建软连接
cd /usr/local
ln -s /usr/local/php-8.0.8 php

查看版本
php -v

#copy配置文件
cp php.ini-development /usr/local/php-8.0.8/lib/php.ini

#启动fpm前的准备
cp /usr/local/php-8.0.8/etc/php-fpm.conf.default /usr/local/php-8.0.8/etc/php-fpm.conf
cd /usr/local/php-8.0.8/etc/php-fpm.d
cp www.conf.default www.conf

#启动fpm
/usr/local/php/sbin/php-fpm

#查看fpm是否启动
netstat -tunlp |grep 9000
```

nginx配置文件

```
server {
    listen       80;
    server_name  192.168.8.*;
    location / {
        root   html;
        index  index.html index.htm index.php;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
    location ~ \.php$ {
        root html;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

解决No package 'oniguruma' found

```shell
git clone https://github.com/kkos/oniguruma 
cd oniguruma
./autogen.sh
./configure --prefix=/usr --libdir=/lib64
make && make instal
```

安转readline

经常会需要在命令行验证一些代码片段，所以让php支持readline对开发很重要。默认php是不安装的

```
查看readline是否安装
php -m | grep -i readline
如果没有安装
cd /root/php-8.0.8/ext/readline
/usr/local/php/bin/phpize
./configure --with-php-config=/usr/local/php/bin/php-config --with-readline
make && make install
vim /usr/local/php/lib/php.ini
在最后面添加一行
extension=readline.so
```

**参考**

1. [安装PHP7.4找不到包 No package ‘oniguruma‘ found错误](https://blog.csdn.net/shachao888/article/details/108167214)