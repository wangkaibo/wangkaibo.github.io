---
title: "日常折腾环境😂Nginx + PHP 7.1.3"
date: 2017-10-08T22:59:46+08:00
tags: ["Linux", "LNMP"]
draft: false
url: '/centos-6-8-install-nginx-stable-and-php7'
---

今天看 PHP 官网 7.1.3 的稳定版已经出来了，准备尝尝鲜，安装过程没有什么变化，这里只是简单记录下。

<!--more-->

### 安装 nginx 1.8

##### 添加官方 yum 源

nginx 提供了官方的 yum 源，安装 nginx 的 1.8 可以使用官方的 yum 源，速度还不错。

在 /etc/yum.repo.d/ 下新建文件 nginx.repo
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

ubuntu/debian 源具体参见：[官方文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

##### 安装 nginx

使用 `yum install nginx` 安装，`yum` 会查找软件包和依赖然后提示是否安装，看好版本后，选择 `Y`。

```
/etc/init.d/nginx start
```
可以使用 `curl localhost` 验证 nginx 是否安装成功。

### 安装 PHP7

注意：如果没有安装 MySQL 的话，最好先安装一下 MySQL。

##### 安装依赖

```
yum install -y libxml2 libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers php-mysqlnd libmcrypt-devel libtidy libtidy-devel recode recode-devel libxpm-devel libXpm-devel
```

安装 libmcrypt：

```
wget ftp://mcrypt.hellug.gr/pub/crypto/mcrypt/libmcrypt/libmcrypt-2.5.6.tar.gz
tar -zxvf libmcrypt-2.5.6.tar.gz
cd libmcrypt-2.5.6.tar.
make && make install
```
默认会安装在 `/usr/local` 目录下。

##### 下载 PHP 源码包

我这里是安装的官方最新稳定版 PHP7.1.3，所以将使用源码安装方式安装。

```
wget http://cn.php.net/distributions/php-7.1.3.tar.gz
tar -xzvf php-7.1.3.tar.gz
cd php-7.1.3
```
编译配置（当前应该在 `php-7.1.3` 目录下）：
```
./configure --prefix=/usr/local/php --with-fpm-user=nginx --with-fpm-group=nginx --with-config-file-path=/etc/php --with-mysql=mysqlnd --with-pdo-mysql=mysqlnd --with-mysqli=mysqlnd --with-gd --with-iconv --with-zlib --enable-xml --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-fpm --enable-mbstring --enable-ftp --enable-gd-native-ttf --with-openssl --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --enable-session --with-mcrypt --with-curl --with-jpeg-dir --with-freetype-dir --with-xpm-dir=/usr --with-bz2
```
编译：
```
make
```
这里可能会等挺长时间，安装过程大概就是复制编译后的文件的一个过程。
```
make test & make install
```

这里需要注意一下，前面 `configure` 时我们配置了 `php.ini` 的路径为 `/etc/php`，然而发现并没有这个目录也找不到 `php.ini` 文件。没关系，这里可以手动的复制一个配置文件过去，配置文件可以在源码包 `php-7.1.3` 里找到，它包括了开发环境和生产环境的默认配置。

```
#开发环境
cp ~/php-7.1.3/php.ini-development ./php.ini
#生产环境
cp ~/php-7.1.3/php.ini-production ./php.ini
```

##### 配置 nginx

修改 `/etc/nginx/conf.d/default.conf` 文件
```
...
location / {
    root   /var/www;
    index  index.html index.htmi index.php;
}
...
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
location ~ .php$ {
    # 根目录 root
    root /var/www;     
    fastcgi_pass 127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

##### 配置 php-fpm
```
cp /local/php/etc/php-fpm.conf.default /local/php/etc/php-fpm.conf
cp /local/php/etc/php-fpm.d/www.conf.default /local/php/etc/php-fpm.d/www.conf
```
为了更方便的启动 `php-fpm`，我们为它添加 sysv 风格的初始化服务脚本，nginx 是通过 `yum` 安装的是有 `/etc/init.d/nginx` 这个启动脚本的。

新建文件 `/etc/init.d/php-fpm`：

```
#!/bin/bash
#
# Startup script for the PHP-FPM server.
#
# chkconfig: 345 85 15
# description: PHP is an HTML-embedded scripting language
# processname: php-fpm
# config: /usr/local/php/etc/php.ini
 
# Source function library.
. /etc/rc.d/init.d/functions
 
PHP_PATH=/usr/local
DESC="php-fpm daemon"
NAME=php-fpm
# php-fpm路径
DAEMON=$PHP_PATH/php/sbin/$NAME
# 配置文件路径
CONFIGFILE=$PHP_PATH/php/etc/php-fpm.conf
# PID文件路径不清楚可以查看 /usr/local/php/etc/php-fpm.conf
PIDFILE=$PHP_PATH/php/var/run/$NAME.pid
SCRIPTNAME=/etc/init.d/$NAME
 
# Gracefully exit if the package has been removed.
test -x $DAEMON || exit 0
 
rh_start() {
  $DAEMON -y $CONFIGFILE || echo -n " already running"
}
 
rh_stop() {
  kill -QUIT `cat $PIDFILE` || echo -n " not running"
}
 
rh_reload() {
  kill -HUP `cat $PIDFILE` || echo -n " can't reload"
}
 
case "$1" in
  start)
        echo -n "Starting $DESC: $NAME"
        rh_start
        echo "."
        ;;
  stop)
        echo -n "Stopping $DESC: $NAME"
        rh_stop
        echo "."
        ;;
  reload)
        echo -n "Reloading $DESC configuration..."
        rh_reload
        echo "reloaded."
  ;;
  restart)
        echo -n "Restarting $DESC: $NAME"
        rh_stop
        sleep 1
        rh_start
        echo "."
        ;;
  *)
         echo "Usage: $SCRIPTNAME {start|stop|restart|reload}" >&2
         exit 3
        ;;
esac
exit 0
```

```
# 赋予执行权限
sudo chmod +x /etc/init.d/php-fpm
# 修改启动等级
sudo /sbin/chkconfig php-fpm on
```
下面就可以用以下命令控制 `php-fpm` 了

```
/etc/init.d/php-fpm start
/etc/init.d/php-fpm stop
/etc/init.d/php-fpm restart
/etc/init.d/php-fpm reload
```
启动 `php-fpm` 后就可测试一下 PHP 文件能否执行了。

有问题可留言

###### 参考资料
[nginx 文档](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)

[Nginx和PHP-FPM的启动/重启脚本](http://www.lovelucy.info/nginx-phpfpm-init-script.html)