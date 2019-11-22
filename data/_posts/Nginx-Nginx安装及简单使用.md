---
title: Nginx-Nginx安装及简单使用
date: 2018-03-08 11:13:08
tags: [Nginx]
categories: Nginx
---

# 安装

本文安装方法面向系统：

- CentOS 6.x

<!-- more -->

- [openresty官网下载地址](http://openresty.org/cn/download.html)

- [CentOS 6.x源码安装包个人存档](https://pan.baidu.com/s/1PJLD0LG0IBY2Xeq9ZnGonw)

openresty官网中有详尽的安装教程，在此总结CentOS 6.x下源码安装的步骤。安装包不妨以openresty-1.13.6.1.tar.gz为例：

```
tar -xzvf openresty-1.13.6.1.tar.gz
cd openresty-1.13.6.1/
./configure
make
make install
```

其中./configure默认为：

```
./configure --prefix=/usr/local/openresty
```

也可自行指定安装目录，例如：

```
./configure --prefix=/reimu/soft/openresty
```

# 基本组件位置

不妨设根目录为/reimu/soft/openresty，则：

Nginx程序位置：/reimu/soft/openresty/nginx/sbin/nginx

Nginx配置文件位置：/reimu/soft/openresty/nginx/conf/nginx.conf

Nginx默认日志文件夹：/reimu/soft/openresty/nginx/logs

# 常见操作

启动Nginx：

```
/reimu/soft/openresty/nginx/sbin/nginx -c /reimu/soft/openresty/nginx/conf/nginx.conf
```

---

验证Nginx配置文件正否符合规范：

```
/reimu/soft/openresty/nginx/sbin/nginx -t
```

若符合规范，则输出如下：

```
nginx: the configuration file /reimu/soft/openresty/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /reimu/soft/openresty/nginx/conf/nginx.conf test is successful
```

---

刷新配置文件配置，重启程序：

```
/reimu/soft/openresty/nginx/sbin/nginx -s reload
```