---
title: 奇葩问题：IDEA 中正确运行，命令行运行就出错
date: 2019-07-29
categories: Java
tags: 
- java
- bug
---

同一个世界，同一份代码，不同的结果。
<!--more-->

# 问题描述

简单来说，就是一个项目在 IDEA 中可以正常运行，打成 jar 包之后，通过 `java -jar` 在命令行运行就出错。

具体来说：

这是一个 Spring Boot 项目，在 service 层发出一个 HTTP 请求。

在 IDEA 运行的时候，能得到正确的 json 数据，在命令行中就只得到“未知错误”的提示。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-07-29_17-28-18.png)



# 解决过程

## HTTP request

既然是 HTTP 的 response 不同，首先想到的是比较一下两处的 request 有何不同。



*查看 okhttp 中请求主体的方法：*

```java
private static String bodyToString(final Request request){
    try {
        final Request copy = request.newBuilder().build();
        final Buffer buffer = new Buffer();
        copy.body().writeTo(buffer);
        return buffer.readUtf8();
    } catch (final IOException e) {
        return "did not work";
    }
}
```



然而，两种方式发出的 HTTP request 好像都很正常，除去一些会变化的字符串外，请求主体一模一样。（这里**会变化的字符串**指的是时间戳、MD5 加密得到的字符串等，它们每次运行都是不一样的值）



此时应该将字符串写死，再对比两者的不同。

但是因为改起来太麻烦，加上以前的经验，我选择对请求主体中的字符进行 URL 编码，事实证明这是一个愚蠢的行为，它导致了在 IDEA 中也得不到想要的结果。

第一次尝试宣告失败。



## 为 okhttp 设置代理

使用下面的方式创建 OkHttpClient 就能在 Fidder 中捕获到 okhttp 发送和接收的数据了。

```java
String hostname = "localhost"/*127.0.0.1*/;
int port = 8888;
System.out.println("设置代理");
Proxy proxy = new Proxy(Proxy.Type.HTTP,
new InetSocketAddress(hostname, port));

this.client = new OkHttpClient().newBuilder().proxy(proxy).build();
```



这里的思路是：不单单对比请求主体，而是通过 Fidder 更细致地比较两次 HTTP 请求的区别。

确实可以捕获到程序发出的 HTTP 请求，但是跟刚才一样，除去一些会变化的值，两者几乎是一样的。

第二次尝试也没有发现有用的信息。



## MD5

还是回到最初的疑惑：明明是一样的代码，而且是同一台电脑上，为什么得到不一样的结果？

同样的代码，难道会计算出不一样的 Hash Value ？

**是的。**

可能是直觉，也可能是实在没有思路了，所以猜测是 sign 字段（通过 MD5 算法生成）的问题。

很快验证出的确是 sign 字段有问题，但是怎么解决呢？

因为不知道在命令行运行时怎么 debug，所以一切都变得很麻烦。

经过慢慢的试验，一步步打印出来，最后发现进行摘要的字符串其实是一样的，但是得到的结果却不一样。

很明显是摘要算法有问题。

这个方法是我在网上复制的，难道它在命令行中执行会得到不一样的结果？

我又在网上搜了其他的 MD5 代码，但是还是一样出错。（如果此时运行对了，可能我永远都不会知道问题真正的原因了）。

最终用我的工地英语在谷歌搜索：`java cmd md5 digest difference`，然后看到[这篇回答](<https://stackoverflow.com/questions/6928641/will-java-messagedigest-generated-different-md5-hash-on-different-jdk-version> )，才知道因为 MD5 算法中，要将字符串转换成 byte[]，而 IDEA 和命令行的编码不同，导致得到的 byte[] 不同，进而导致 sign 不同。

*在计算 Hash Value 的时候指定编码。*

```java
byte[] array = md.digest(str.getBytes("UTF-8"));
```



# 总结

虽然解决这个问题只需要改一行代码，但是整个过程却花了我很多时间，做了太多太多无谓的尝试。

不过还好，还是解决了。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/psb.webp)