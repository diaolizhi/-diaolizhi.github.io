---
title: Spring Boot MyBatis 多个参数的坑
date: 2018-11-30
categories: Spring Boot
---

我真傻，真的。
<!--more-->



# 场景

数据库中有一个字段（time）记录这条数据产生的时间，现在需要一个接口，返回某一天产生的所有数据。



# 参数个数的坑



## 接收两个参数并使用

在数据访问层，**可以接收两个参数，但是不能用，一旦用了就出错**：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-11-30_13-50-30.png)



## 接收两个参数但是不用

接收参数，并且 SQL 语句没有错误，是**可以查询到数据的**。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-11-30_13-58-57.png)



两种情况不同之处：前者使用了传递过来的参数，后者直接把字符串写上去。

所以我一直以为是我**传递过来的参数有问题**，也想过是不是**参数类型的问题？**

改了很久....



## 解决方法

```java
@Select("SELECT * FROM find WHERE time BETWEEN #{time1} AND #{time2}")
List<FindLog> findDay(@Param("time1") String time1, @Param("time2") String time2);
```

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-11-30_14-16-01.png)

简单来说，就是在形参前面加了一个注解：@Param。。。



# 是否需要加引号的坑

- 直接在 SQL 语句中写日期：**需要**用单引号包起来
- 如果在 SQL 中 #{time}，那么 time 引用的字符串就**不需要**加单引号



# 其他

- 利用 IDEA 创建实体类时：使用 java.util.Date 类型，方便设置日期

```java
log.setTime(new Date());
```

- 数据库 time 字段的类型为：datetime。如果是 date 就不会精确到几点几分几秒。
- 修改数据库的时区，免得跟本地时间不同



# 使用的接口

- 接收的 num 为 0 表示今天，为 1 就表示前一天。
- 通过 SimpleDateFormat 格式化时间成：*2018-11-30* 这样的格式。

```java
@GetMapping("find_day")
public Object showDay(int num) {
    long DAY_IN_MS = 1000 * 60 * 60 * 24;
    Date date = new Date(System.currentTimeMillis() - (num * DAY_IN_MS));

    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
    String time = format.format(date);

    //        拼接字符串
    String time1 = time + " 00:00:00";
    String time2 = time + " 23:59:59";

    return service.findDay(time1, time2);
}
```

- 拼接字符串是用于 SQL 查询，查询某一天的语句：

```sql
SELECT * FROM find WHERE TIME BETWEEN "2018-11-30 00:00:00" AND "2018-11-30 23:59:59"
```



# 总结

因为一个注解浪费了一个多小时的时间。

另外查询方法也很不优雅。