---
title: Spring Boot 2.x 免费升级 HTTPS
date: 2019-01-02
categories: Spring Boot
---

有个🔒的感觉就是 8 一样嗷。

<!--more-->

本文只说操作步骤，别的不讲，因为不会。



# 必要的前提

- 拥有一个域名
- 拥有一个 Linux 服务器



# 申请证书

我是通过 [Youtube 视频](https://www.youtube.com/watch?v=fL6A9I1-U6M) 了解到如何快速免费的申请一个 HTTPS 证书的，推荐大家可以看一哈。

申请证书的网站是：[FreeSSL](https://freessl.cn/)

## 进入首页

需要注意的是：**证书的类型**以及**证书的有效期**，根据个人需要选择。

我选择的是第二个，因为我需要用到多个二级域名。

如果只有一个网站，建议选择第一个，因为有效期比较长。

**如果证书到期了，就需要到这个网站再申请一次。**

接下来输入自己的域名，我输入的是：*.ddllzz.xyz

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_15-59-34.png)



## 创建证书

直接输入邮箱然后点击创建即可。

**但是为了安全起见，推荐自己生成 CSR，工具自己去网上搜。**

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_16-18-43.png) 



## 验证

完成上面一步之后可以看到：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_16-27-12.png)

此时需要根据上面的提示为自己的域名添加 DNS 解析：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_16-29-44.png)

等待一会点击上面图中的“配置完成，检测一下”。

看到下图说明 OK 了。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_16-31-19.png)

再然后点击上图的“点击验证”，得到证书信息：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_17-44-33.png)

点击下载证书，记得保管好。



# 转换

刚才下载了证书（下载的是一个压缩包），压缩包里的文件是：full_chain.pem 和 private.key，而我们在 Spring Boot 中需要的是 pkcs12 类型的文件，所以需要用 openssl 进行转换。

**推荐在 Linux 下完成转换，因为我在 Windows 下生成的 private.key 一直是 0kb，同样的操作在 Linux 下就没问题。**

```shell
openssl pkcs12 -export -in full_chain.pem -inkey private.key -out keystore.p12（这个是生成的文件名） -name tomacat -CAfile full_chain.pem -caname root
```

现在得到了 keystore.p12，就可以在 Spring Boot 项目中使用它了。



# 配置

将刚刚得到的 kyestore.p12 文件复制到 Spring Boot 项目中的 resources 目录下，然后在 application.properties 文件中添加：

```properties
server.port=8443
server.ssl.trust-store-type=PKCS12
server.ssl.key-store=classpath:kyestore.p12
server.ssl.key-store-password=这里是进行转换时输入的密码
server.ssl.key-alias=tomcat
```



# 本地测试

在本地测试时，如果通过 https://127.0.0.1:8443 访问，会提示不安全：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_17-52-02.png) 

因为刚才的证书是属于 ddllzz.xyz 这个域名的，但是现在并非通过这个域名访问，所以会提示不安全。

现在修改本机 hosts 文件，添加一行：

```
127.0.0.1 ddllzz.xyz
```

再次通过 https://ddllzz.xyz:8443 访问就行了:

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-01-02_17-55-45.png)



# 80 端口重定向到 HTTPS

在 Spring Boot 中使用下面这段代码，就可以实现：

> 当访问 ddllzz.xyz 时跳转到 https://ddllzz.xyz:8443。

它存在的意义是：别人可能并不知道你 HTTPS 设置的端口，如果没有这个重定向就无法通过 HTTPS 的方式访问了。

当然了，设置其他端口重定向到 HTTPS 也是可以的。

```java
@Bean
public ServletWebServerFactory servletContainer() {
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
        @Override
        protected void postProcessContext(Context context) {
            SecurityConstraint securityConstraint = new SecurityConstraint();
            securityConstraint.setUserConstraint("CONFIDENTIAL");
            SecurityCollection collection = new SecurityCollection();
            collection.addPattern("/*");
            securityConstraint.addCollection(collection);
            context.addConstraint(securityConstraint);
        }
    };
    tomcat.addAdditionalTomcatConnectors(redirectConnector());
    return tomcat;
}

private Connector redirectConnector() {
    Connector connector = new Connector(
        TomcatServletWebServerFactory.DEFAULT_PROTOCOL);
    connector.setScheme("http");
    connector.setPort(80);
    connector.setSecure(false);
    connector.setRedirectPort(8443);
    return connector;
}
```



# 参考资料

[一文教你将 SpringBoot 网站升级为 HTTPS](https://juejin.im/post/5b44b4fef265da0f767530f7)

[Spring Boot Application Secured by Let’s Encrypt Certificate](https://www.heydari.be/spring-boot-application-secured-by-a-lets-encrypt-certificate/)

[Spring Boot Secured By Let's Encrypt](openssl pkcs12 -export -in crt1 -inkey private.key -out keystore.p12 -name tomcat -CAfile CAcrt -caname root)

[redirect http to https in Spring boot 2.0.0 M2](https://github.com/spring-projects/spring-boot/issues/9836)



# 总结

终于写完了，写了好几个小时。

比上午第一次弄还要久，不过总算是搞定了，下次再弄应该就不会那么费劲了。