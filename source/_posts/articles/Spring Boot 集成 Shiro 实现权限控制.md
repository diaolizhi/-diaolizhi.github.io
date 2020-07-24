---
title: Spring Boot 集成 Shiro 实现权限控制
date: 2018-12-11
categories: Spring Boot
---

拥有指定的权限才可以进行相应的操作。

<!--more-->

有一些操作是所有人都可以执行的，比如：访问首页、访问登录界面。而有一些就需要管理员权限，比如：删除用户。

为了实现这种功能，Shiro 提供了**角色**和**权限**两个概念。

推荐一篇博客：[Spring Boot Shiro权限控制](https://mrbird.cc/Spring-Boot-Shiro%20Authorization.html)

我的理解是：一个用户对应一个或多个角色，一个角色又对应一个或多个权限。这样一来，用户就对应了多个明确的角色或者权限。然后设置接口所需要的角色或权限，实现权限控制。

在 Web 应用中，角色、权限都存放在数据库中，当然了还有用户表，以及*用户-角色表*、*角色-权限表*，总共有五个数据表。

当 Shiro 需要验证权限的时候，就需要从数据库中查找数据。

那么需要查找的是什么呢？

先看看 Shiro 需要的是什么。Realm 除了认证功能之外，还有授权的功能，对应的就是 doGetAuthorizationInfo 方法。在这个方法中返回的是一个 AuthorizationInfo 对象，而这个对象有两个方法：setRoles 和 setStringPermissions。

也就是说，Shiro 需要用户所对应的多个角色，以及多个权限。那么在数据访问层要做的就是查找一个用户所对应的角色和权限。



大概了解授权的流程之后，下面就具体说该怎么做。



# 创建数据表

建议阅读上面推荐的文章，讲得比较详细。

```java
CREATE DATABASE /*!32312 IF NOT EXISTS*/`shiro-login` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `shiro-login`;

/*Table structure for table `t_permission` */

DROP TABLE IF EXISTS `t_permission`;

CREATE TABLE `t_permission` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `url` varchar(50) NOT NULL,
  `name` varchar(100) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

/*Table structure for table `t_role` */

DROP TABLE IF EXISTS `t_role`;

CREATE TABLE `t_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(20) NOT NULL,
  `memo` varchar(50) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

/*Table structure for table `t_role_permission` */

DROP TABLE IF EXISTS `t_role_permission`;

CREATE TABLE `t_role_permission` (
  `rid` int(11) NOT NULL,
  `pid` int(11) NOT NULL,
  PRIMARY KEY (`rid`,`pid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

/*Table structure for table `t_user` */

DROP TABLE IF EXISTS `t_user`;

CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(20) NOT NULL,
  `password` varchar(100) NOT NULL,
  `salt` varchar(20) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;

/*Table structure for table `t_user_role` */

DROP TABLE IF EXISTS `t_user_role`;

CREATE TABLE `t_user_role` (
  `uid` int(11) NOT NULL,
  `rid` int(11) NOT NULL,
  PRIMARY KEY (`uid`,`rid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



# 插入数据

数据仅供参考，根据自己的设计插入数据。

```sql
CREATE DATABASE /*!32312 IF NOT EXISTS*/`shiro-login` /*!40100 DEFAULT CHARACTER SET utf8 */;

USE `shiro-login`;

/*Data for the table `t_permission` */

insert  into `t_permission`(`id`,`url`,`name`) values 
(1,'/user','user:add'),
(2,'/user','user:delete'),
(3,'/user','user:see');

/*Data for the table `t_role` */

insert  into `t_role`(`id`,`name`,`memo`) values 
(1,'admin','超级管理员'),
(2,'test','测试用户');

/*Data for the table `t_role_permission` */

insert  into `t_role_permission`(`rid`,`pid`) values 
(1,1),
(1,2),
(1,3),
(2,3);

/*Data for the table `t_user` */

insert  into `t_user`(`id`,`username`,`password`,`salt`) values 
(1,'zs','6f294b522deba0531722c92b4199c6b95d76b00e231ca2c765aae8c32fd36222','zsreg'),
(2,'root','6a9629cd8b8c68a86c800cdeae59f2b3185bb6ce1c0896a675e91a860c7702ee','rootreg'),
(3,'admin','dddf9fe0979a7db4fbf06a2366c5862e775fd702c56249decb372e876636b9ab','adminreg'),
(8,'aaa','02d66435c563cc9130876fc5d7c1496e6b451345741b46d1472de3d6c82a8afa','aaareg'),
(9,'aaaa','b344150010d7630f0422fdd9f491a450a8299831e28c78b2f5173ed5b1fd164e','aaaareg');

/*Data for the table `t_user_role` */

insert  into `t_user_role`(`uid`,`rid`) values 
(2,1),
(9,2);
```



# 创建实体类

使用 IDEA 自动生成即可。



# UserRoleMapper

根据用户名查找角色集：

```java
@Mapper
public interface UserRoleMapper {

    @Select("SELECT t_role.name FROM t_role, t_user, t_user_role WHERE t_user.`id` = t_user_role.`uid` AND t_user.`username` = #{username} AND t_user_role.`rid` = t_role.`id`")
    List<TRole> findRolesByUsername(String username);

}
```



# UserPermissionMapper

根据用户名查找权限集：

```java
@Mapper
public interface UserPermissionMapper {

    @Select("SELECT p.* FROM t_user u LEFT JOIN t_user_role ur ON u.`id` = ur.`uid` LEFT JOIN t_role r ON ur.`rid` = r.`id` LEFT JOIN t_role_permission rp ON r.`id` = rp.`rid` LEFT JOIN t_permission p ON rp.`pid` = p.`id` WHERE u.`username` = #{username} ")
    List<TPermission> findPermissonsByUsername(String username);

}
```



# Realm 添加方法

```java
@Autowired
private UserRoleMapper userRoleMapper;

@Autowired
private UserPermissionMapper userPermissionMapper;

@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

    String username = (String) SecurityUtils.getSubject().getPrincipal();

    List<TRole> roles = userRoleMapper.findRolesByUsername(username);

    SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
    Set<String> roleSet = new HashSet<>();
    for (TRole t : roles) {
        roleSet.add(t.getName());
        System.out.println(t.getName());
    }
    info.setRoles(roleSet);

    List<TPermission> ps = userPermissionMapper.findPermissonsByUsername(username);

    Set<String> pSet = new HashSet<>();

    for (TPermission p : ps) {

        //            如果在 mapper 没有查到任何数据，ps 也不为 null，但是 ps.get(0) 的值是 null
        if (p != null) {
            pSet.add(p.getName());
            System.out.println(p.getName());
        }

    }

    info.setStringPermissions(pSet);

    return info;
}
```



# 添加权限控制



## 方法一：配置文件中

在 ShiroConfig 类的 shiroFilterChainDefinition 方法中：

```java
@Bean
public ShiroFilterChainDefinition shiroFilterChainDefinition() {
    DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();

    //        authc 是必须通过认证，而不是“记住我”
    chainDefinition.addPathDefinition("/admin/**", "roles[admin]");

    chainDefinition.addPathDefinition("/me/**", "roles[admin]");

    return chainDefinition;
}
```



## 方法二：使用注解

在 Controller 的接口上配置：

```java
// 表示当前Subject已经通过login进行了身份验证；即Subject.isAuthenticated()返回true。
@RequiresAuthentication  
 
// 表示当前Subject已经身份验证或者通过记住我登录的。
@RequiresUser  

// 表示当前Subject没有身份验证或通过记住我登录过，即是游客身份。
@RequiresGuest  

// 表示当前Subject需要角色admin和user。  
@RequiresRoles(value={"admin", "user"}, logical= Logical.AND)  

// 表示当前Subject需要权限user:a或user:b。
@RequiresPermissions (value={"user:a", "user:b"}, logical= Logical.OR)
```



# 定义授权失败跳转页面

在 properties 文件中：

```properties
shiro.unauthorizedUrl=/error123
shiro.loginUrl=/toLogin
```



# 捕获全局异常

## 两种情况 

- 一个接口使用注解指定了所需要的权限，当**没有登录**或者**没有权限**访问这个接口时，它不会跳转到失败页面，也不会跳转到登录页面，而是抛出一个异常。
- 在配置文件中指定一个接口所需的权限，那么它就会跳转到错误页面或者登录界面。

为了解决第一种情况，就需要捕获异常：

```java
@RestControllerAdvice
public class ShiroExceptionHandler { 
    @ExceptionHandler(value = AuthorizationException.class)
    public String handleAuthorizationException() {
        return "没有权限访问";
    }
}
```



# 总结

- 创建角色和权限
- 查询角色和权限
- 重写 doGetAuthorizationInfo
- 给接口配置权限