---
title: Spring Boot 集成 Shiro 实现简单的登录
date: 2018-12-06
categories: Spring Boot
---

如何上手 Shiro ？

<!--more-->

# 概述

在 Spring Boot 2.x 中集成 Shiro 做一个登录功能大体上可以分为三步：

1. 配置 Shiro
2. 自定义 Realm
3. 创建登录接口



# 了解登录流程

我刚开始只知道 Shiro 是一个安全框架，但是完全搞不懂怎么用它实现一个**登录**的功能，更别提集成到 Spring Boot 中了。

经过了几天的摸索，终于了解了**登录**的流程，不过对 Shiro 的工作原理并不深入，而且我没有实际的开发经验，所以这篇文章只能算作我自己的一篇笔记，它很可能对你没有帮助。

概述中提到，第一步是配置 Shiro，在说它之前，想象一下，用户要登录一个网站，最开始要做的是什么？肯定是输入用户名和密码吧。所以作为后端，对于登录这个功能最开始要做的就是接收用户名和密码。

```java
@GetMapping("/login")
public Object login(@RequestParam String username, @RequestParam String password) {

}
```

这就是 Spring Boot 中的一个接口，接收传过来的用户名和密码。（这里为了简单使用了 GetMapping）

收集了用户名和密码，然后就应该验证它们是否正确，具体怎么验证不用我们操心，因为我们只需要把 Shiro 需要的东西交给它，它会替我们判断这一次登录是成功还是失败。

那么 Shiro 需要什么东西呢？第一，它需要这一次输入的用户名和密码。第二，它需要这个用户名所对应的真正的密码。

就这两样东西，这么看来 Shiro 还真是挺容易用的，但是刚开始我真的是不知道从哪里下手。



# 配置 Shiro

[将Apache Shiro集成到Spring-Boot应用程序中](https://shiro.apache.org/spring-boot.html)

谷歌搜索 Spring Boot Shiro 就会出现上面这个链接，根据里面的步骤可以将 Shiro 添加到 Spirng Boot 中。（不过这个教程没有具体说某个功能怎么实现，如果有的话，我这篇笔记就没有存在的意义了。）

下面说一下具体的步骤：



## 添加依赖

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>1.4.0-RC2</version>
</dependency>
```



## 配置 Shiro

在自己的项目中添加一个 config 包，在其中创建一个 Shiro 的配置类，目的是提供一些 Shiro 需要的对象。

第一个 shiroFilterChainDefinition 方法中可以配置哪些接口需要登录才可以访问，哪些接口需要特定的权限才可以访问。因为暂时不考虑这么多，所以里面只返回一个对象。

第二个就很重要了，上面说了 Shiro 在登录的时候需要两样东西，一个是输入的用户名和密码，另一个是这个用户名所对应的真正的密码。前者接收用户的输入就行了，而后者就需要从数据库中获取了。在 Shiro 中，Realm 对象就扮演这么一个角色：在登录的时候它查数据库，并把查到的数据交给 Shiro，Shiro 会有特定的模块去验证这一次登录。至于登录结果：**登录成功就没事，要是失败就抛出异常。** 注意这里的 myShiroRealm 是 Realm 接口的实现类，接下来马上就说到它。

```java
@Configuration
public class ShiroConfig {

    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();

        return chainDefinition;
    }

    @Bean
    public Realm myShiroRealm() {
        MyShiroRealm realm = new MyShiroRealm();
        return realm;
    }
}
```



# 自定义 Realm

第一眼看到这个类可能有些懵逼，这个方法是干嘛的？它的形参是个什么鬼？返回值又是啥玩意？这个类的父类又是谁？

```java
public class MyShiroRealm extends SimpleAccountRealm {

    /**
     * 注入 userService 用于查询用户数据
     */
    @Autowired
    private UserService userService;

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        String username = token.getPrincipal().toString();

        String password = userService.getPasswordByUsername(username);

        if (password != null) {
            AuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(
                    username,
//                    这里的密码是数据库中的密码
                    password,
//                    返回Realm名
                    getName());
            return authenticationInfo;
        }
        return null;
    }

}
```

先说这个父类：在 Shiro 中 Realm 是一个接口，而 SimpleAccountRealm 是它的实现类。具体关系：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-05_19-46-47.png)

实话说这些类是干嘛的，我也不知道，但是知道 Realm 的作用，以及 SimpleAccountRealm 是它的实现类就行了。

至于这个 doGetAuthenticationInfo 方法，它在登录的时候要用到。

在这个方法中：

- 接收这次登录时输入的用户名和密码
- 根据用户名查询真正的密码
- 将用户名和真正的密码一起返回给 Shiro

它的职责还是很明确的，根据功能不难看出：它的形参就是登录时的凭证，里面包含了用户名和密码等信息。返回值就是提供给 Shiro 验证此次登录的对象，这个对象包含了真正的用户信息。

形参可以看作是**认证凭证**，返回值可以看作是**身份认证信息**，它们具体是谁的实现类在这里的就 8 说了。



## 模拟查询数据库

如果真要查数据库，就得引入更多的依赖，这里就模拟一下。

```java
@Service
public class UserService {

    public String getPasswordByUsername(String username){
        switch (username){
            case "root":
                return "root";
            case "admin":
                return "admin123";
            default:
                return null;
        }
    }

}
```



# 创建登录接口

登录接口就是接收用户输入的用户名和密码，然后让 Shiro 去验证是否正确。

```java
@GetMapping("/login")
public Object login(@RequestParam String username, @RequestParam String password) {

    UsernamePasswordToken token = new UsernamePasswordToken(username, password);

    Subject subject = SecurityUtils.getSubject();

    try {
        subject.login(token);
        return "登录成功";
    } catch (Exception e) {
        return "登录失败";
    }
}
```

 接口的参数就不说了，就是传递过来的用户名和密码。

通过用户名和密码创建一个 UsernamePasswordToken，这个类是上面自定义 Realm 类中提过的**认证凭证**的实现类，不信自己去查。这个 token 将用户输入的信息封装起来，之后就会传递给 Realm 对象。

至于这个 Subject 类，官方文档说可以把它看作是与系统交互一个对象，可以是人也可能不是。具体怎么说我忘了，但是现在直接把它看成是一个用户，因为我想不出其他情况。

对于后端，对于某一次登录，先获取到这次登录的用户是谁，然后调用它的 login 方法去登录。

接下去不用判断登录的结果，因为登录失败会抛出异常，捕获异常然后处理就完事了。异常具体有很多种，比如用户名不存在、密码错误等。这里为了简（tou）单（lan）就不一一列出了。



# 测试



## 登录成功

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-05_20-32-07.png)



## 登录失败

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-05_20-33-26.png)



# 总结

写了一个多小时，想方设法说得通俗易懂。

不管怎么样，文档和资料难啃还是要看，不然对一些基本的类都不了解。

既然你看到这里，就希望对你有帮助。