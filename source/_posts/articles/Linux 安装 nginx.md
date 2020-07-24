---
title: Linux 安装 nginx
date: 2019-5-14
categories: nginx
---

记录在 Linux 安装 nginx 的过程。

<!--more-->

# 下载

[下载地址](<http://nginx.org/en/download.html> )

```shell
wget http://nginx.org/download/nginx-1.16.0.tar.gz
```



# 解压

```shell
tar -zvxf nginx-xxx.tar.gz
```



# 配置

```shell
./configure
```



# 可能需要安装的库

```shell
yum install -y httpd-devel pcre perl pcre-devel zlib zlib-devel GeoIP GeoIP-devel
```



# 安装

```shell
make install
```



# 启动

```shell
cd /usr/local/nginx/sbin
./nginx
```

