---
title: 在Centos7安装nginx1.21.1
date: 2021-08-18 09:33:54
tags: ["nginx", "centos7", "源码安装"]
---

##### 一、安装准备

首先由于nginx的一些模块依赖一些lib库，所以在安装nginx之前，必须先安装这些lib库，这些依赖库主要有g++、gcc、openssl-devel、pcre-devel和zlib-devel 所以执行如下命令安装

```
yum install gcc-c++  pcre pcre-devel zlib zlib-devel openssl openssl--devel
```

##### 二、安装Nginx

安装之前，最好检查一下是否已经安装有nginx

```
find -name nginx 
```

如果系统已经安装了nginx，那么就先卸载

```
yum remove nginx
```

然后开始安装
首先进入/usr/local目录

```
cd /usr/local
```

从官网下载最新版的nginx

```
wget -c https://nginx.org/download/nginx-1.21.1.tar.gz
```

注：版本号可更改，去官网查看最新版本号修改即可

解压nginx压缩包

```
tar -zxvf nginx-1.21.1.tar.gz 
```

会产生一个nginx-1.21.1 目录，这时进入nginx-1.21.1 目录

```
cd nginx-1.12.1
```

接下来安装，使用--prefix参数指定nginx安装的目录,make、make install安装

```
./configure
make && make install
```

注：默认安装在/usr/local/nginx，推荐使用默认设置

如果没有报错，顺利完成后，最好看一下nginx的安装目录

```
whereis nginx 
```

##### 三、启动和停止nginx

```
cd /usr/local/nginx/sbin/
./nginx 
./nginx -s stop
./nginx -s quit
./nginx -s reload
./nginx -s quit: 此方式停止步骤是待nginx进程处理任务完毕进行停止。
./nginx -s stop: 此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。
```

查询nginx进程：

```
ps aux | grep nginx
```

##### 四、重启 nginx

1.先停止再启动（推荐）：
对 nginx 进行重启相当于先停止再启动，即先执行停止命令再执行启动命令。如下：

```
./nginx -s quit
./nginx
```

2.重新加载配置文件：
当 nginx 的配置文件 nginx.conf 修改后，要想让配置生效需要重启 nginx，使用 -s reload 不用先停止 nginx 再启动 nginx 即可将配置信息在 nginx 中生效，如下：

```
./nginx -s reload
```

启动成功后，在浏览器可以看到如下页面：

![nginx](https://www.jiangbianwanghai.com/img/nginx.png "启动成功看到的画面")

##### 五、开机自启动

即在rc.local增加启动代码就可以了。

```
vi /etc/rc.local
```

增加一行

```
/usr/local/nginx/sbin/nginx
```

到这里，nginx安装完毕，启动、停止、重启操作也都完成。