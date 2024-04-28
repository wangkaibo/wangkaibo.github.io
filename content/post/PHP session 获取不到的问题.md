---
title: "PHP session 获取不到之前设置值的问题"
date: 2017-05-05T23:50:45+08:00
tags: ["php"]
draft: false
url: '/php-session-获取不到'
---

### PHP session 获取不到之前设置值的问题

今天帮朋友的服务器部署一个小程序商城时遇到一个 session 无法保存的问题。

<!--more-->

在 web 的一个页面通过 $_SESSION 设置的 session 值：

```php
$_SESSION['user'] = 'user';

var_dump($_SESSION);
```

设置完之后是可以打印出值的。但是跳转到另一页面却取不到 session 的值。

虽然session.auto_start 配置为 Off，但已经排除了 session_start(); 没调用的问题。也没有通过 `session_set_save_handler()` 去自定义 session 的存储过程。

通过 `phpinfo`函数给出的信息，看到了 `session.save_path` 配置是 `/var/lib/php/session`，进去该目录发现并没有任何文件。看到这里定位到应该是 session 目录权限问题。

安装完 php-fpm ，默认运行用户是 apache，我之前改成了 nobody ，session 目录忘了改（这里应该极其容易忘）。😂

这里把session目录权限改下就好了。环境问题有时候真的是防不胜防啊😂