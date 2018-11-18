---
title: Spring Boot 接口规范
date: 2018-11-18
categories: Spring Boot
---



<!--more-->

# 根据权限区分接口

一般的查询接口可以对普通用户开放，但是对于像删除这些操作只能由管理员执行。

所以在 controller 包中添加一个 admin 包，在 admin 包下新建一个 VideoAdminController 用于存放只对管理员开放的接口。

```java
package com.diaolizhi.mybatisdemo.controller.admin;

import com.diaolizhi.mybatisdemo.domain.Video;
import com.diaolizhi.mybatisdemo.service.VideoService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/admin/api/v1/video")
public class VideoAdminController {

    @Autowired
    private VideoService videoService;

    @PutMapping("update")
    public int update(String videoAuthor, int videoId) {
        Video video = new Video();
        video.setVideoAuthor(videoAuthor);
        video.setVideoId(videoId);
        return videoService.updateVideo(video);
    }

    @PostMapping("/insert")
    public int addVideo(String videoName, String videoAuthor) {
        Video video = new Video();
        video.setVideoName(videoName);
        video.setVideoAuthor(videoAuthor);
        return videoService.addVideo(video);
    }

    @DeleteMapping("delete")
    public int delById(int id) {
        return videoService.delete(id);
    }
}
```



注意：

- 除了将敏感操作移动到这个 controller 之外，还需要注意 @RequestMapping("/admin/api/v1/video")，这一句和原来相比多了一个 admin。



# @RequestParam

## 字段映射

HTTP 中请求的字段名最好以下划线的方式命名，又由于 Java 中常使用驼峰命名法，所以这里使用 @RequestParam 注解完成映射。

*value 对应的是 HTTP 请求携带的字段名。*

```java
@PostMapping("/insert")
public int addVideo(@RequestParam(value = "video_author") String videoAuthor,
                    @RequestParam(value = "video_name") String videoName) {
    Video video = new Video();
    video.setVideoName(videoName);
    video.setVideoAuthor(videoAuthor);
    return videoService.addVideo(video);
}
```



## 设置默认值

如果观察一下，会发现有些 HTTP 请求中会携带一个 page_no 字段，这表示请求服务器第几页的内容。如果没有携带这个参数，就默认返回第一页。

@RequestParam 也提供了这种*默认值*的功能。

```java
@GetMapping("page")
public Object findAll(@RequestParam(value = "page_no", defaultValue = "1") int pageNo) {
    System.out.println(pageNo);
    return videoService.findAll();
}
```

如果没有 page_no 参数默认是 1，由于还没有添加分页功能，所以只是输出 pageNo 的值。



# 接受一个对象

假设要在数据库中添加一个视频，而视频又有很多个字段，一个一个地写就很麻烦了。

所以可以接受一个 Video 对象，然后请求的时候直接发送 json 数据。

```java
@PutMapping("update")
public int update(@RequestBody Video video) {
    return videoService.updateVideo(video);
}
```

- 这里需要使用 @RequestBody 注解
- 发送请求的方法：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-11-18_11-59-41.png)

发送 Json 数据不必将 Java 对象的每个属性都写上去，让这些属性值为 null 就行。



# 总结

遵守必要的规范，不然以后更不方便。