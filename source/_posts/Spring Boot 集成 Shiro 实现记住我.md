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
@Bean
public RememberMeManager rememberMeManager(){
    CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();

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

2019年1月9日 22:18:23 更新