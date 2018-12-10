---
title: Spring Boot 集成 Shiro 实现 Remember me
date: 2018-12-10
categories: Spring Boot
---

当重启浏览器之后，不用重新登录。

<!--more-->

# 添加配置

在 ShiroConfig 类中添加：

```java
//注意这里不用加 @Bean 注解
public SimpleCookie rememberMeCookie(){
    //        构造方法一定要传一个字符串，作为 Cookies 中的字段名
    SimpleCookie simpleCookie = new SimpleCookie("RememberMe");
    //        设置“记住我”多久
    simpleCookie.setMaxAge(259200);

    return simpleCookie;
}

@Bean
public RememberMeManager rememberMeManager(){
    CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
    cookieRememberMeManager.setCookie(rememberMeCookie());

    //        rememberme cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度（128 256 512 位），通过以下代码可以获取
    //        try {
    //            KeyGenerator keygen = KeyGenerator.getInstance("AES");
    //            SecretKey deskey = keygen.generateKey();
    //            System.out.println(Base64.encodeToString(deskey.getEncoded()));
    //        } catch (Exception e) {
    //            e.printStackTrace();
    //        }

    cookieRememberMeManager.setCipherKey(Base64.decode("WuB+y2gcHRnY2Lg9+Aqmqg=="));
    return cookieRememberMeManager;
}
```



# 登录接口

登录接口添加一个参数用来接收*是否记住我*的参数：

```java
@GetMapping("login")
public Object login(@RequestParam(name = "username") String username,
                    @RequestParam(name = "password") String password,
                    @RequestParam(name = "rememberMe") boolean rememberMe) {

    UsernamePasswordToken token = new UsernamePasswordToken(username, password, rememberMe);

    Subject subject = SecurityUtils.getSubject();

    if (subject.isAuthenticated()) {
        return "已经登录直接返回";
    } else if (subject.isRemembered()) {
        return "已经被记住，直接返回";
    }

    try {

        //            token.setRememberMe(false);
        subject.login(token);

        subject.getSession().setTimeout(180000);

        return "登录成功";

    } catch (Exception e) {
        e.getMessage();
        return "登录失败";
    }

}
```



# 总结

虽然是实现**记住我的功能：**重启浏览器之后可以获取到之前登录的 subject 的名字（使用```subject.getPrincipal();```）。

但是如果给一个 subject 的 session 设置了标签：

```java
Subject subject = SecurityUtils.getSubject();
Session session = subject.getSession();
session.setAttribute("label", "aa");
```

重启浏览器之后，就获取不到这个标签了。

猜测是超时的问题。

如果使用调用 sessionManager 的 setSessionIdCookie 方法设置 cookie，并设置超时时间，就可以做到重启浏览器之后仍然可以获取到之前设置的标签。

然而，这样做之后，无论登录时*记住我*是真是假，重启浏览器之后都还是记住的。

所以，看起来通过 setSessionIdCookie 可以设置 Sesssion 的 id（就是在浏览器控制台看到的名字），也可以设置超时时间，但是无法和 RememberMe 一起使用。

ps：这就是找不到教程的痛苦，一切都是自己摸索，或者说是猜。