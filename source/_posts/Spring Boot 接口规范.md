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