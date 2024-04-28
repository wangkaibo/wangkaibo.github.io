---
title: "Centos上从零开始编译安装LNMP环境"
date: 2015-09-15T23:32:45+08:00
tags: ["Linux", "LNMP"]
draft: false
url: '/new-centos-compiled-install-lnmp-environment'
---

前几天买了个 VPS，因为之前接触 Nginx 不是很多，所以折腾 LNMP环境折腾了几个小时，所以我想重新安装一遍并写篇学习笔记，或者说教程，因此本文将从在崭新的 Centos 上搭建 LNMP 环境，为了所有软件都可以选择指定版本和安装配置，因此除了依赖包之外都采用源码包安装的方式来安装。废话不多说，刚刚把 VPS rebulid。

<!--more-->

注：本文属于新手友好型，能遇到的错误基本上都会遇到。😄

## 一、安装 Nginx
这里我选择的是1.8.0
执行命令下载并解压 Nginx 源码包

```
wget http://nginx.org/download/nginx-1.8.0.tar.gz
tar -xzvf nginx-1.8.0.tar.gz
cd nginx-1.8.0
```

<!-- more -->

这里我们./Configure 肯定会报错的，因为我们这是一台崭新的 Centos 系统，缺少好多Nginx依赖包，这里我直接 ./configure 先试一下有什么错误

>./configure: error: C compiler cc is not found

好吧，甚至连 gcc 都没有，先安装 gcc 才能编译源码

```
yum install gcc
```

下面正式编译

```
./configure --prefix=/usr --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --user=nginx --group=nginx --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --http-client-body-temp-path=/var/tmp/nginx/client/ --http-proxy-temp-path=/var/tmp/nginx/proxy/ --http-fastcgi-temp-path=/var/tmp/nginx/fcgi/ --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi --http-scgi-temp-path=/var/tmp/nginx/scgi --with-pcre 
```

又一个错误来了

>./configure: error: the HTTP rewrite module requires the PCRE library.
>You can either disable the module by using --without-http_rewrite_module
>option, or install the PCRE library into the system, or build the PCRE library
>statically from the source with nginx by using --with-pcre=<path> option.

这个错误我们看提示可以看出，Nginx 需要安装 PCRE Library，类似的错误提示还有很多，下面我们一次性将 Nginx 的依赖包都安装上

```
//安装PCRE 库是为使用 Nginx 支持 HTTP Rewrite 模块。
yum -y install pcre-devel
// zlib
yum -y install zlib-devel
// openssl
yum -y install openssl-devel

```

再执行上面的 ./configure 命令，当出现下面提示时，说明我们已经配置好了可以编译了
因为我们配置时定义了 user 和 group，下面新建一个nginx 用户和nginx 用户组才能运行nginx服务进程 

```  
  # groupadd -r nginx
  # useradd -r -g nginx -s /bin/false -M nginx        
  # id nginx
```

>Configuration summary
  + using system PCRE library
  + using system OpenSSL library
  + md5: using OpenSSL library
  + sha1: using OpenSSL library
  + using system zlib library
  ......

```
make
//一般 make test 可省略
make test
make install
```

安装成功后我们还需要做一些配置

1、为Nginx提供sysv风格的服务脚本（ Red Hat Nginx Init Script Should work on RHEL, Fedora, CentOS. Tested on CentOS 5.  Save this file as /etc/init.d/nginx ）

在新建文件 

```
vi /etc/rc.d/init.d/nginx
```
内容如下：

```
#!/bin/sh 
# 
# nginx - this script starts and stops the nginx daemon 
# 
# chkconfig:   - 85 15  
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \ 
#               proxy and IMAP/POP3 proxy server 
# processname: nginx 
# config:      /etc/nginx/nginx.conf 
# config:      /etc/sysconfig/nginx 
# pidfile:     /var/run/nginx.pid 
  
# Source function library. 
. /etc/rc.d/init.d/functions 
  
# Source networking configuration. 
. /etc/sysconfig/network 
  
# Check that networking is up. 
[ "$NETWORKING" = "no" ] && exit 0 
  
nginx="/usr/sbin/nginx" 
prog=$(basename $nginx) 
  
NGINX_CONF_FILE="/etc/nginx/nginx.conf" 
  
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx 
  
lockfile=/var/lock/subsys/nginx 
  
make_dirs() { 
   # make required directories 
   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -` 
   options=`$nginx -V 2>&1 | grep 'configure arguments:'` 
   for opt in $options; do 
       if [ `echo $opt | grep '.*-temp-path'` ]; then 
           value=`echo $opt | cut -d "=" -f 2` 
           if [ ! -d "$value" ]; then 
               # echo "creating" $value 
               mkdir -p $value && chown -R $user $value 
           fi 
       fi 
   done 
} 
  
start() { 
    [ -x $nginx ] || exit 5 
    [ -f $NGINX_CONF_FILE ] || exit 6 
    make_dirs 
    echo -n $"Starting $prog: " 
    daemon $nginx -c $NGINX_CONF_FILE 
    retval=$? 
    echo 
    [ $retval -eq 0 ] && touch $lockfile 
    return $retval 
} 
  
stop() { 
    echo -n $"Stopping $prog: " 
    killproc $prog -QUIT 
    retval=$? 
    echo 
    [ $retval -eq 0 ] && rm -f $lockfile 
    return $retval 
} 
  
restart() { 
    configtest || return $? 
    stop 
    sleep 1 
    start 
} 
  
reload() { 
    configtest || return $? 
    echo -n $"Reloading $prog: " 
    killproc $nginx -HUP 
    RETVAL=$? 
    echo 
} 
  
force_reload() { 
    restart 
} 
  
configtest() { 
  $nginx -t -c $NGINX_CONF_FILE 
} 
  
rh_status() { 
    status $prog 
} 
  
rh_status_q() { 
    rh_status >/dev/null 2>&1 
} 
  
case "$1" in 
    start) 
        rh_status_q && exit 0 
        $1 
        ;; 
    stop) 
        rh_status_q || exit 0 
        $1 
        ;; 
    restart|configtest) 
        $1 
        ;; 
    reload) 
        rh_status_q || exit 7 
        $1 
        ;; 
    force-reload) 
        force_reload 
        ;; 
    status) 
        rh_status 
        ;; 
    condrestart|try-restart) 
        rh_status_q || exit 0 
            ;; 
    *) 
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}" 
        exit 2 
esac 
```

 给予刚才的文件可执行权限

```
 chmod +x /etc/rc.d/init.d/nginx
```

并把他添加到服务列表中

```
chkconfig --add nginx
chkconfig nginx on
```

启动服务，并查看启动状态

```
/etc/init.d/nginx start
netstat -tnlp  | grep nginx
```

现在在可以访问了，Welcome to nginx!

重启服务命令

```
/etc/init.d/nginx restart
/etc/init.d/nginx stop
```

## 二、安装 MySQL
### 下载 MySQL
MySQL 的官方[下载地址](http://dev.mysql.com/downloads/mysql/)，进去选择 Source Code
这里我装的是5.6.28
如果你不想再去官网找的话，可以在你的下载目录直接执行

```
wget http://cdn.mysql.com//Downloads/MySQL-5.6/mysql-5.6.28.tar.gz
tar -xzvf mysql-5.6.28.tar.gz 
cd mysql-5.6.28
```
### 准备工作
安装前的准备工作，MySQL需要用到的依赖包依次安装上

```
yum -y install gcc-c++
yum -y install cmake
yum -y install bison-devel
yum -y install ncurses-devel

```
### 编译安装

编译配置

```
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
-DMYSQL_DATADIR=/usr/local/mysql/data \
-DSYSCONFDIR=/etc \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql/mysql.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
```

参数详细配置，参考MySQL官方文档的[MySQL Source-Configuration Options](http://dev.mysql.com/doc/refman/5.5/en/source-configuration-options.html)
我这里出现了以下警告：

> CMake Warning:
>   Manually-specified variables were not used by the project:

 >   WITH_MEMORY_STORAGE_ENGINE

 >   WITH_READLINE


>-- Build files have been written to: /root/lnmp_source/mysql-5.6.28

提示两个配置项没有应用上，我干脆就把这两项配置去掉了重新./configure 了一下，这次没有警告了
我查了下这两项的作用：

-WITH_MEMORY_STORAGE_ENGINE #支持Memory引擎
-DWITH_READLINE=1 \ #快捷键功能

如果用到再做配置，下面安装

好了开始编译

```
make
```
 MySQL 编译时间比较长...我这单核 VPS...30分钟后...

 ```
 make install
 ```

 创建 MySQL 的用户和用户组

```
groupadd mysql
useradd -g mysql mysql
```

修改/usr/local/mysql权限

```
chown -R mysql:mysql /usr/local/mysql
```

### 初始化配置

进入安装路径

cd /usr/local/mysql
进入安装路径，执行初始化配置脚本，创建系统自带的数据库和表

```
scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --user=mysql
```

这里执行会报错，如下：


>./scripts/mysql_install_db: /usr/bin/perl: bad interpreter: No such file or directory
>原因是系统没有安装Perl 和 Perl-devel

解决办法，执行：

```
yum -y install perl perl-devel
```

安装完成后再执行 MySQL 初始化脚本就可以了
现在应该可以开启 MySQL 了。

同样将 MySQL 提供的sysv风格的服务脚本复制到 /etc/init.d/mysql目录

```
cp support-files/mysql.server /etc/init.d/mysql
// 设置开机启动
chkconfig mysql on
// 启动服务
service mysql start  // 启动MySQL
// 或者使用
/etc/init.d/mysql start // 启动 MySQL
```

将 MySQL 加入环境变量，修改/etc/profile文件，在文件末尾添加

```
PATH=/usr/local/mysql/bin:$PATH
export PATH
```

保存并关闭文件，运行下面的命令让配置生效

```
source /etc/profile
```

MySQL 跑起来了，我极低配 512 内存的 VPS 瞬间被占用了 400MB 啊
修改 /usr/local/mysql/my.cnf 中的以下几项配置（没有该项可以添加）：

```
table_definition_cache=400
table_open_cache=256
```

然后

```
/etc/init.d/mysql restart
```

瞬间清爽了，只能牺牲性能了😂。

### 修改 MySQL root 账户密码

```
mysql -uroot  
mysql> SET PASSWORD = PASSWORD('YOUR PASSWORD');
```

### 修改防火墙配置

打开/etc/sysconfig/iptables

在“-A INPUT –m state --state NEW –m tcp –p –dport 22 –j ACCEPT”，下添加：

```
-A INPUT -m state --state NEW -m tcp -p -dport 3306 -j ACCEPT
```

然后保存，并关闭该文件，在终端内运行下面的命令，刷新防火墙配置：

```
service iptables restart
```

MySQL 安装完毕。

## 安装 PHP7

### 下载 PHP7源码包
我这里下载的是 PHP 7，下载别的版本去[php.net](http://php.net/downloads.php)

```
wget http://cn2.php.net/distributions/php-7.0.2.tar.gz
tar -xzvf php-7.0.2.tar.gz
cd php-7.0.2
```
### 编译配置
如果我们直接配置的话应该会报错，我们还需要安装一些 PHP 的依赖包，编译配置一下试试

```
./configure --enable-fpm --with-mysql
```

首先报以下错误：

>configure: error: xml2-config not found. Please check your libxml2 installation.

缺少 libxml2，直接使用 yum 方式安装 libxml2

```
yum install -y libxml2
```

下面一口气把所有的PHP 用到的软件包都安装上

```
yum install -y libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers php-mysqlnd libmcrypt-devel libtidy libtidy-devel recode recode-devel libxpm-devel
```

安装完之后可以编译配置了

```
./configure --prefix=/usr/local/php7 --with-config-file-path=/etc/php7 --with-mysql=mysqlnd --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-gd --with-iconv --with-zlib --enable-xml --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt --with-curl --with-jpeg-dir --with-freetype-dir --with-xpm-dir=/usr --with-bz2
```

注意前两个配置

```
--prefix=/usr/local/php7 //安装位置
--with-config-file-path=/etc/php7 //配置文件位置
```

如果还会报错，可能是缺少依赖包，按照提示安装就可以了。

>configure: error: xpm.h not found.

上面忘了安装 libxpm 和 mcrypt 扩展
安装 libXpm

```
yum install libXpm-devel
```

好多 yum 源中没有 mcrypt，担心第三方源不可靠，我选择使用源码安装 mcrypt，切换到另一个目录

```
wget ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/libmcrypt/libmcrypt-2.5.6.tar.gz
tar -zxvf libmcrypt-2.5.6.tar.gz
cd libmcrypt-2.5.6.tar.
make && make install
```

再配置，编译安装

```
make
sudo make install

```

这里列出 PHP 编译配置常见错误和解决办法：
>configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution
>
```
yum install libxslt-devel
```
>configure: error: Could not find net-snmp-config binary. Please check your net-snmp installation.
>
```
yum install net-snmp-devel
```
>configure: error: Please reinstall readline - I cannot find readline.h
>
```
yum install readline-devel
```
>configure: error: Cannot find pspell
>
```
yum install aspell-devel
```
>checking for unixODBC support... configure: error: ODBC header file '/usr/include/sqlext.h' not found!
>
```
yum install unixODBC-devel
```
>configure: error: Unable to detect ICU prefix or /usr/bin/icu-config failed. Please verify ICU install prefix and make sure icu-config works.
>
```
yum install libicu-devel
```
>configure: error: utf8mime2text() has new signature, but U8TCANONICAL is missing. This should not happen. Check config.log for additional information.
>
```
yum install libc-client-devel
```

>configure: error: freetype.h not found.
>
```
yum install freetype-devel
```
>configure: error: xpm.h not found.
>
```
yum install libXpm-devel
```
>configure: error: png.h not found.
>
```
yum install libpng-devel
```
>configure: error: vpx_codec.h not found.
>
```
yum install libvpx-devel
```
>configure: error: Cannot find enchant
>
```
yum install enchant-devel
```
>configure: error: Please reinstall the libcurl distribution - easy.h should be in /include/curl/

### 配置 Nginx 

nginx本身不能处理PHP，它只是个web服务器，当接收到请求后，如果是php请求，则发给php解释器处理，并把结果返回给客户端。

nginx一般是把请求发fastcgi管理进程处理，fascgi管理进程选择cgi子进程处理结果并返回被nginx

首先修改 Nginx 的配置，刚才安装时配置的配置文件目录是 /etc/nginx/nginx.conf

```
vi /etc/nginx/nginx.conf
```

```
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
location ~ .php$ {
	root /var/www; //根目录 root	
	fastcgi_pass 127.0.0.1:9000;
	fastcgi_index index.php;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	include fastcgi_params;
}
```

这里需要注意 location ~.php 的配置块的 root 根目录一定要跟 server 块的配置一样，否则 php 文件会不能解析，会报 File not found 错误。

同时也建议在location / 配置下的 index 加上 index.php

配置好 Nginx 可以先尝试启动 php-fpm

```
/usr/local/php7/sbin/php-fpm
```

嗯，是的，应该会报下面的错误😄

>ERROR: failed to open configuration file '/local/php7/etc/php-fpm.conf': No such file or directory (2)
>[20-Jan-2016 10:56:50] ERROR: failed to load configuration file '/local/php7/etc/php-fpm.conf'

将默认的配置文件 cp 为正房

```
cp /local/php7/etc/php-fpm.conf.default /local/php7/etc/php-fpm.conf
```

如果这时候启动 php-fpm 会报下面的错误

>WARNING: Nothing matches the include pattern '/local/php7/etc/php-fpm.d/*.conf' from /local/php7/etc/php-fpm.conf at line 125.
>[20-Jan-2016 11:03:40] ERROR: No pool defined. at least one pool section must be specified in config file

根据错误提示信息，还是初始化配置文件

```
cp /local/php7/etc/php-fpm.d/www.conf.default /local/php7/etc/php-fpm.d/www.conf
```

这时候应该可以启动了。
下面测试一下

```
cd /var/www
echo "<?php phpinfo();" > index.php
```

到此为止，整个安装过程就写完了.

零零散散地看了网上很多的教程，所以决定总（fu）结（zhi）一（yi）下（fen），描述比较傻瓜化，安装过程中基本所有的错误就提到了。希望能对刚踏入 Linux 服务端的同学有所帮助。


参考文章：

* [Nginx 源码编译安装](http://xyuex.blog.51cto.com/5131937/1013414)
* [CentOS 6.4下编译安装MySQL 5.6.14](http://www.cnblogs.com/xiongpq/p/3384681.html)
* [编译安装PHP7 beta1](http://blog.wuxu92.com/compile-and-install-php7-beta1/)
* [PHP configuration error and Solutions in RPM](https://coderwall.com/p/ggmpfa/php-configuration-error-and-solutions-in-rpm)
