---
title: Spring Boot 集成 Shiro 并使用 SHA-256 加密密码
date: 2018-12-06
categories: Spring Boot
---

用户密码的加密以及验证。

<!--more-->

我们不能将用户的密码以明文存储，否则如果自己的数据库被入侵，用户在其他网站的帐号也可能被盗。

我对各种加密算法并不了解，就不说什么了，直接开始。



# 注册时生成密文

Shiro 提供了一个 SimpleHash 类可以很方便的对字符串进行加密，只需一行代码就可以对密码进行加密。

```java
String hashPassword = new SimpleHash("SHA-256", password, username+"reg", 1024).toString();
```

它的第一个参数是加密的算法，第二个是要加密的内容，第三个是盐（这个参数随便你定），第四个是期望的次数。

得到加密之后的密码，就可以将整个 User 对象存储到数据库中，以后登录的时候再通过 Realm 查询。

```java
/**
 * 注入用于存储 user 对象
 */
@Autowired
private UserService userService;

@GetMapping("reg")
public Object reg(@RequestParam(name = "username") String username,
                  @RequestParam(name = "password") String password) {

    String hashPassword = new SimpleHash("SHA-256", password, username+"reg", 1024).toString();

    User user = new User();
    user.setUsername(username);
    user.setPassword(hashPassword);
    user.setSalt(username + "reg");

    userService.addUser(user);

    return "注册成功";
}
```



# Realm

上一篇文章说了如何实现登录，但是当时没有对密码进行加密。

加密之后的 Realm 又该怎么写了呢？

Shiro 给人一种感觉：基本的东西我都准备好了，你要啥就自己加。

所以只需要在自定义 Realm 里重写它的 setCredentialsMatcher 接口，在里面指定算法类型、期望的次数就行了。

```java
@Override
public void setCredentialsMatcher(CredentialsMatcher credentialsMatcher) {
    HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
    matcher.setHashAlgorithmName("SHA-256");
    matcher.setHashIterations(1024);
    super.setCredentialsMatcher(matcher);
}
```

这里跟注册时基本一致，不过好像没有指定盐？

加密使用了盐，验证的时候也少不了它，只不过它不是写在这里的，毕竟每个用户的盐可能是不一致的（盐根据你自己的想法去设置）。

可还记得 doGetAuthenticationInfo 方法返回了一个**身份认证信息对象**供 Shiro 验证，使用盐之后，在创建这个对象的时候把盐传进去就行了。

**注意：因为现在需要的不仅仅是密码，还需要盐。所以模拟查询数据库的时候不能仅仅返回一个字符串了，而是应该返回一个对象，然后从对象中取出密码、盐。**

```java
/**
 * 注入用于存储 user 对象
 */
@Autowired
private UserService userService;

@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    String username = token.getPrincipal().toString();

    User user = userService.getUserByUsername(username);

    if (user != null) {

        String password = user.getPassword();

        AuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(
            username,
            password,
            //传入盐
            ByteSource.Util.bytes(user.getSalt()),
            getName());
        return authenticationInfo;
    }
    return null;
}
```



# 模拟存取 User 对象

一切为了简单。

```java
@Service
public class UserService {

    private HashMap<String, User> users = new HashMap<>();

    public void addUser(User user) {
        users.put(user.getUsername(), user);
    }

    public User getUserByUsername(String username) {
        return users.get(username);
    }

}
```



# 测试

## 没有注册时登录

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-06_19-36-54%20%E6%B2%A1%E6%9C%89%E6%B3%A8%E5%86%8C.png)



## 注册成功

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-06_19-37-10%20%E6%B3%A8%E5%86%8C%E6%88%90%E5%8A%9F.png)



## 登录成功

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-06_19-37-24%20%E7%99%BB%E5%BD%95%E6%88%90%E5%8A%9F.png)



## 密码错误

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-06_19-37-37%20%E5%AF%86%E7%A0%81%E9%94%99%E8%AF%AF.png)



# 总结

希望对你有帮助。