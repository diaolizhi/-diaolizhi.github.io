---
title: PageHelper 的基本使用
date: 2018-11-18
categories: Spring Boot
---

使用 PageHelper 实现查询分页功能。

<!--more-->

# 添加依赖

```xml
<!-- 分页插件依赖 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>4.1.0</version>
</dependency>
```



# 添加配置文件

在 config 包下创建 PageHelperConfig

```java
@Configuration
public class PageHelperConfig {
    @Bean
    public PageHelper pageHelper(){
        PageHelper pageHelper = new PageHelper();
        Properties p = new Properties();
        p.setProperty("offsetAsPageNum","true");
        p.setProperty("rowBoundsWithCount","true");
        p.setProperty("reasonable","true");
        pageHelper.setProperties(p);
        return pageHelper;
    }
}
```



# 在接口中测试

这个方法接收了两个参数：请求的页码、每一页显示的数量。

然后调用 `PageHelper.startPage(pageNo, size);` ，结果返回的就不是全部数据，而是第 3、4 条了。

```java
@GetMapping("page")
public Object findAll(@RequestParam(value = "page_no", defaultValue = "1") int pageNo,
                      @RequestParam(value = "size", defaultValue = "2") int size) {
    PageHelper.startPage(pageNo, size);
    return videoService.findAll();
}
```

```json
[
    {
        "videoId": 3,
        "videoName": "1asdfasdfasdsdf",
        "videoAuthor": "dsfasdaf"
    },
    {
        "videoId": 4,
        "videoName": "微信支付",
        "videoAuthor": "赵六"
    }
]
```



# 包装类

上面根据参数返回了指定的信息，但是这些信息还不够。

```java
@GetMapping("page")
public Object findAll(@RequestParam(value = "page_no", defaultValue = "1") int pageNo,
                      @RequestParam(value = "size", defaultValue = "2") int size) {

    PageHelper.startPage(pageNo, size);
    List<Video> list = videoService.findAll();
    PageInfo<Video> pageInfo = new PageInfo<>(list);
    return pageInfo;
}
```

再添加几句之后，就可以返回更多要用的信息了。

```json
{
    "pageNum": 2,
    "pageSize": 2,
    "size": 2,
    "orderBy": null,
    "startRow": 3,
    "endRow": 4,
    "total": 8,
    "pages": 4,
    "list": [//省略这部分
    ],
    "firstPage": 1,
    "prePage": 1,
    "nextPage": 3,
    "lastPage": 4,
    "isFirstPage": false,
    "isLastPage": false,
    "hasPreviousPage": true,
    "hasNextPage": true,
    "navigatePages": 8,
    "navigatepageNums": [
        1,
        2,
        3,
        4
    ]
}
```

这些就包括了当前页码、显示的条数，是否有下一页等等。

如果只想要一部分信息，还可以进一步封装：

```java
@GetMapping("page")
public Object findAll(@RequestParam(value = "page_no", defaultValue = "1") int pageNo,
                      @RequestParam(value = "size", defaultValue = "2") int size) {

    PageHelper.startPage(pageNo, size);
    List<Video> list = videoService.findAll();
    PageInfo<Video> pageInfo = new PageInfo<>(list);
    Map<String, Object> map = new TreeMap<>();
    map.put("page_num", pageInfo.getPageNum());
    map.put("has_next", pageInfo.isHasNextPage());
    map.put("data", pageInfo.getList());
    return map;
}
```

返回的结果就更简洁：

```json
{
    "data": [
        {
            "videoId": 3,
            "videoName": "1asdfasdfasdsdf",
            "videoAuthor": "dsfasdaf"
        },
        {
            "videoId": 4,
            "videoName": "微信支付",
            "videoAuthor": "赵六"
        }
    ],
    "has_next": true,
    "page_num": 2
}
```



# 总结

暑假就用了 PageHelper 进行分页，但是对于原理一无所知，虽然现在也差不多。

