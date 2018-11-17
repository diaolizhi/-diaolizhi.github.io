---
title: Spring Boot 整合 Mybatis 和阿里巴巴数据源
date: 2018-11-17
categories: Spring Boot
---

如何在 Spring Boot 中使用 Mybatis 查询数据。
<!--more-->

# 添加依赖

一个很方便的方式是在创建项目的时候就添加 Mybatis 和 Mysql-connector 依赖。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/%E5%88%9B%E5%BB%BA%20Spring%20Boot%20%E9%A1%B9%E7%9B%AE%E5%B9%B6%E6%B7%BB%E5%8A%A0%E4%BE%9D%E8%B5%96.png)

但是现在阿里巴巴数据源的依赖还没有添加，[druid 的项目地址](https://mvnrepository.com/artifact/com.alibaba/druid)。

当前用的最多的版本是：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
```

**千万别忽略了 Mysql 驱动的依赖。**



# 添加配置

在 application.properties 中添加：

```properties
#=================================== 数据库相关 ============================================
#可以自动识别
#spring.datasource.driver-class-name =com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/movie?useUnicode=true&characterEncoding=utf-8
spring.datasource.username =root
spring.datasource.password =password
#如果不使用默认的数据源 （com.zaxxer.hikari.HikariDataSource）
spring.datasource.type =com.alibaba.druid.pool.DruidDataSource
```



# 添加注解

在启动类添加一行注解：

```java
@MapperScan("com.diaolizhi.mybatisdemo.mapper")
```

注解中的包名对应自己项目中 mapper 包的路径。



# 创建 Mapper 类进行测试



## 准备工作

- 创建数据库
- 创建数据表
- 添加数据
- 在 IDEA 连接数据库，并生成实体类



## 创建 TestMapper 接口

- 注意这是一个接口而不是类
- 注意数据库字段与实体类的字段是否一致

```java
import com.diaolizhi.mybatisdemo.domain.Video;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface TestMapper {

    @Select("SELECT * FROM Video")
    //加下面这个注解是因为：数据库字段和实体类变量名不一致
    @Results({
            @Result(column = "video_id",property = "videoId"),  //javaType = java.util.Date.class
            @Result(column = "video_name",property = "videoName"),
            @Result(column = "video_author",property = "videoAuthor")
    })
    List<Video> findAll();
}
```



## 创建测试接口

在 IDEA 中注入 TestMapper 会有错误提示，虽然不影响使用，但看不顺眼的时候可以在设置中关闭提示。

```java
@RestController
public class TestController {

    @Autowired
    private TestMapper mapper;

    @GetMapping("test_db")
    public Object testDB() {
        return mapper.findAll();
    }
}
```



# 总结

温故而知自己有多蠢。