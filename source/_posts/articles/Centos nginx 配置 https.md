---
title: Centos nginx 配置 https
date: 2018-12-13
categories: nginx
---

在腾讯云主机上给 nginx 配置 https。

<!--more-->

# 前提

- 安装了 nginx（在[这里](http://nginx.org/download/)下载 nginx-1.9.9.tar.gz），解压，通过 make 命令安装。
- 准备好证书，这里是腾讯云备案送一年的证书。
- 为了上传文件方便，下载[winscp](https://winscp.net/eng/download.php)。



# 修改 nginx 配置

进入到 /usr/local/nginx/conf 目录下，修改 nginx.conf，添加一个：

```
server {
        listen 443;
        server_name diaolizhi.com; #填写绑定证书的域名
        ssl on;
        ssl_certificate 1_diaolizhi.com_bundle.crt;
        ssl_certificate_key 2_diaolizhi.com.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
        ssl_prefer_server_ciphers on;
        location / {
            root   html/record; #站点目录
            index  index.html index.htm;
        }
}
```

**注意：站点目录的 html 前面没有斜杠。**



## 报错

```shell
nginx: [emerg] the "ssl" parameter requires ngx_http_ssl_module...
```

提示没有 ssl 这个模块，参考博客：[博客园](https://www.cnblogs.com/ghjbk/p/6744131.html)



# 总结

在 Linux 安装 nginx 之后，可以配置不同的端口，每个端口可以对应 html 目录下的不同的文件夹。

只要添加一个 server，比如：

```
server {
        listen       8000;
        server_name  localhost;

        location /{
            root   html/record;
            index  index.html index.htm;
        }
}
```

具体用到再百度吧。