---
title: 在Centos7下安装php的扩展v8js
tags: ["v8js","centos7", "rpm"]
---
**前言**

最近，我们小组在做一个网页可视化的项目——就是通过拖拽组件，设置属性、参数的方式快速的生成 `web页面`。客户在后台设置完成后，会将各个组件的属性和参数存储到数据库中，后端程序从数据库中读取数据并动态的拼接 `css` 和 `html` 代码完成页面的渲染工作。因此，需要根据预先设定的规则做计算。

**后台编辑时呈现的可视化**是前端同事通过编写大量 `js` 的 `function` 完成 css 和 html 的计算和渲染。而后端如果需要渲染这些数据，则需要将他们写的前端代码逻辑在 `php` 中重新编写一遍。（目前，我们是这样做的）

那么，有没有一种解决方案，直接使用前端同事编写好的 js 脚本呢？

如果有可以极大的提高我们的开发效率和提升整个项目的稳定性。至少它可以解决以下三个问题：

1. 保持前端代码的一致性。只需要执行前端人员编写的 js 脚本即可，无需再通过 php 转换一遍；
2. 减少后端人员开发工作量。不需要再通过 php 转换一遍，节省下来的时间是非常可观的；
3. 代码更加稳定。后面如果前端的 function 有任何变动也不用再通知后端人员做代码同步了。

经过调研，[V8JS](https://github.com/phpv8/v8js) 进入了我们的视野。它是 php 的一个扩展，可以在 php 中直接执行 js 代码并获得结果。经过下面一系列的安装和测试，有一些体会记录如下：

V8JS 实际上是一个 php 与 `v8` 之间搭建的桥梁，它最终会将 js 代码提交给 v8 来执行。由于 v8 是谷歌团队开发的浏览器引擎，安装过程中需要访问谷歌的相关地址，所以尽量用一台能够访问外网的机器来安装。如果要用国内的机器，则需要配置科学上网的VPN。

`Centos7` 默认的 `gcc` 版本是 `4.8.5`，不能满足 v8 编译的需求。需要先升级gcc（这个升级过程非常耗时）之后下载谷歌自家的 `depot_tools` 用来拉取 v8 源码（由于谷歌为自家产品推广的考虑，虽然有些繁琐，但这种方式也是现在唯一的获取v8最新代码的方式）。经测试，编译 v8 是非常耗时的。最后，只要 v8 安装成功了，剩下 `v8js` 的安装就比较容易了，它跟编译 php 的其他扩展没有什么不同。

---------------------

下面是整个安装和测试过程

**〇、安装测试环境如下**

```shell
硬件
Linode 1核1.8G的VPS

软件
[root@li1594-191 ~]# cat /proc/version
Linux version 3.10.0-1160.15.2.el7.x86_64 (mockbuild@kbuilder.bsys.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) ) #1 SMP Wed Feb 3 15:06:38 UTC 2021
[root@li1594-191 ~]# cat /etc/redhat-release
CentOS Linux release 7.9.2009 (Core)
```

**一、启动screen**

当想要一个命令或者操作一直运行下去，但是你直接在终端里面执行的话，这个终端退出后命令就无法再去接着执行了，也无法看到这个命令操作的状态，这个时候可以用到screen

强烈建议在开始安装之前先启动屏幕会话，因为后面的一些编译非常耗时。运行以下命令

```shell
#创建utf8编码模式的新会话gcc
screen -U -S gcc
```

**二、查看本机的gcc版本并安装指定版本的gcc**

```shell
#下载压缩包
cd /tmp
wget https://mirrors.ustc.edu.cn/gnu/gcc/gcc-7.5.0/gcc-7.5.0.tar.gz

#解压
tar -xzvf gcc-7.5.0.tar.gz
cd gcc-7.5.0

#下载供编译需求的依赖项
./contrib/download_prerequisites

#建立一个文件夹存放编译文件
mkdir build && cd build

#生成 Makefile 文件
../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j4 && make install
注：这个过程非常耗时

-------------分割线-------------

下面是将gcc最新版本的动态库替换系统中老版本的动态库

查找编译gcc时生成的最新动态库
find / -name "libstdc++.so*"

将找到的动态库libstdc++.so.6.0.21复制到/usr/lib64
cp /tmp/gcc-7.5.0/build/stage1-x86_64-pc-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.24 /usr/lib64

切换工作目录至/usr/lib64，删除原来的软连接， 将默认库的软连接指向最新动态库。
cd /usr/lib64
rm -rf libstdc++.so.6
ln -s libstdc++.so.6.0.24 libstdc++.so.6

-------------分割线-------------

[root@li1594-191 ~]# gcc -v
使用内建 specs。
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/local/libexec/gcc/x86_64-pc-linux-gnu/7.5.0/lto-wrapper
目标：x86_64-pc-linux-gnu
配置为：../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
线程模型：posix
gcc 版本 7.5.0 (GCC)
```

**三、安装v8**

```shell
#如果没有安装git，需要先安装
yum install git

cd /tmp

# depot_tools安装
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH=`pwd`/depot_tools:"$PATH"

# 下载 v8 这一步很慢
fetch v8
cd v8

# 选择你想编译的版本
git checkout 7.5.288.23
gclient sync

# Setup GN
tools/dev/v8gen.py -vv x64.release -- is_component_build=true use_custom_libcxx=false

# 开始编译 这一步很慢
ninja -C out.gn/x64.release/

# Install to /opt/v8/
mkdir -p /opt/v8/{lib,include}
cp out.gn/x64.release/lib*.so out.gn/x64.release/*_blob.bin \
  out.gn/x64.release/icudtl.dat /opt/v8/lib/
cp -R include/* /opt/v8/include/
```

**四、安装php（这里用的是7.2.33)**

```shell
wget -c https://www.php.net/distributions/php-7.2.33.tar.gz
tar -zxvf php-7.2.33.tar.gz
./configure --prefix=/usr/local/php7.2.33 --with-mysqli --with-pdo-mysql --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir --enable-simplexml --enable-xml --disable-rpath --enable-bcmath --enable-soap --enable-zip --with-curl --enable-fpm --with-fpm-user=nobody --with-fpm-group=nobody --enable-mbstring --enable-sockets --with-gd --with-openssl --with-mhash --enable-opcache --disable-fileinfo
make
make install
cp php.ini-development /usr/local/php7.2.33/lib/php.ini
cd /usr/local
ln -s /usr/local/php7.2.33 php

vim /etc/profile
export PATH=$PATH:/usr/local/php/bin
source /etc/profile
```

**五、安装v8js**

```shell
cd /tmp
git clone https://github.com/phpv8/v8js.git

cd v8js
phpize

./configure --with-v8js=/opt/v8 LDFLAGS="-lstdc++" --with-php-config=/usr/local/php/bin/php-config

make && make install
```

**六、初步测试一下**

```php
<?php

$v8 = new V8Js();

/* basic.js */
$JS = <<< EOT
len = print('Hello' + ' ' + 'World!' + "\\n");
len;
EOT;

try {
  var_dump($v8->executeString($JS, 'basic.js'));
} catch (V8JsException $e) {
  var_dump($e);
}
```

**七、用我们的项目代码试一下**

zhangdan.js（下面三个 function 是可视化在用的前端代码）
```js
[root@li1594-191 phptest]# cat zhangdan.js
/**
 * image opacity
 * @param {object} prop
 * @param {string} prefix 前缀，默认空
 */
function getOpacity(prop, prefix = '') {
  if (!prop) return null;
  const bt = prop[prefix + 'bt'];
  const bo = prop[prefix + 'bo'];

  return bt === 'image' ? ((bo || 100) / 100) : 1
}
/**
 * image padding
 * @param {object} prop
 * pdt padding type  'custom' | 'default'
 * pt  padding top
 * pb  padding bottom
 * pl  padding left
 * pr  padding right
 */
function getPadding(prop) {
  if (!prop) return null;

  const { pdt, pt = 0, pb = 0, pl = 0, pr = 0 } = prop;
  if (pdt === 'custom') {
    return `${pt}px ${pr}px ${pb}px ${pl}px`;
  }

  return null
}

/**
 * border
 * @param {object} prop
 */
function getBorder(prop) {
  if (!prop) return null;

  const { bdw, bds, bdc, btd } = prop
  const width = bdw || 0;

  if (btd && btd === 'default') return null;

  return `${width}px ${bds} ${bdc}`;
}
//计算透明度
var jsonA = '{"bt":"image", "bo":"123"}';
var jsa=JSON.parse(jsonA);
a =getOpacity(jsa);

//计算边距
var jsonB = '{"pdt":"custom", "pt": 5, "pr": 6, "pb": 7, "pl": 8}';
var jsb=JSON.parse(jsonB);
b = getPadding(jsb);

//计算边框
var jsonC = '{"bdw": 5, "bds": "solid", "bdc":"rgba(147, 42, 42, 1)"}';
var jsc=JSON.parse(jsonC);
c = getBorder(jsc);

'透明度：'+a+'边距：'+b+'边框：'+c
```

zhangdan.php
```php
[root@li1594-191 phptest]# cat zhangdan.php
<?php
$v8 = new V8Js();
$code = file_get_contents('zhangdan.js');
try {
  $res = $v8->executeString($code);
  echo $res.PHP_EOL;
} catch (V8JsException $e) {
  var_dump($e);
}
```

执行结果（执行结果符合预期）
```shell
[root@li1594-191 phptest]# php zhangdan.php
透明度：1.23边距：5px 6px 7px 8px边框：5px solid rgba(147, 42, 42, 1)
[root@li1594-191 phptest]#
```

**八、rpm方式安装v8**

鉴于上面第1-4步的源码安装难度很大，既需要外网的支持又需要升级gcc，在生产环境部署时可能会有难度。下面就介绍一下 `rpm` 的安装方式。rpm 是下载好编译好的包文件，所以就绕开了上述的两个问题。

1.安装环境如下

```
硬件
2核1G的内网开发机

软件
➜  ~ cat /proc/version
Linux version 3.10.0-327.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.3 20140911 (Red Hat 4.8.3-9) (GCC) ) #1 SMP Thu Nov 19 22:10:57 UTC 2015
➜  ~ cat /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)
```

2.安装v8

```
screen -r v8
yum install -y http://repo.okay.com.mx/centos/7/x86_64/release/okay-release-1-1.noarch.rpm
#--nogpgcheck 禁掉GPG验证检查
yum install -y v8 v8-devel --nogpgcheck
```

3.检查是否安装完成。如果出现版本号说明已经安装成功

```
➜  ~ d8 -v
V8 version 6.2.91
d8>
```

4.复制文件
```
cp /usr/lib64/libv8* /usr/lib/
```

剩下的就是安装 v8js 了，详见上面源码编译安装第五步。

经过测试，v8js 2.1.0（2018-01-07发布）版本可以成功安装

```
➜  ~ php --ri "v8js"

v8js

V8 Javascript Engine => enabled
V8 Engine Compiled Version => 6.2.91
V8 Engine Linked Version => 6.2.91
Version => 2.1.0

Directive => Local Value => Master Value
v8js.flags => no value => no value
v8js.icudtl_dat_path => no value => no value
v8js.use_date => 0 => 0
v8js.use_array_access => 0 => 0
```

注：

v8js 2.1.1及以后版本会提示下面的错误，可能是 v8 的 6.2.91 不能满足最新的 v8js 的编译需求

```
checking for libv8_libplatform... configure: error: could not find libv8_libplatform library
表示安装libv8-dev包的版本太低了。
```

**参考文档**

1. [如何在 Centos7 中安装 gcc](https://www.cnblogs.com/lpbottle/p/install_gcc.html)
1. [Centos7宝塔面板安装PHP V8js扩展](https://blog.csdn.net/zpinsist/article/details/116762793)
1. [Installing on CentOS 7 x64 PHP 7.3](https://github.com/phpv8/v8js/wiki/Installing-on-CentOS-7-x64---PHP-7.3)
1. [How To Install GCC on CentOS 7](https://linuxhostsupport.com/blog/how-to-install-gcc-on-centos-7/)
1. [pkgs.org](https://pkgs.org/search/?q=v8)