---
title: Spring Boot 动态 SQL 语句
date: 2018-11-18
categories: Spring Boot
---

根据实际需要生成 SQL 语句。

<!--more-->

更新视频信息时，每次需要更新的字段可能不一致，这一次可能是视频名称，下一次可能是视频的作者。

对应的，就需要写很多判断、多个 SQL 语句。

幸好 Spring Boot 中可以动态生成 SQL 语句来满足各种情况。



# 创建 VideoSqlProvider

- 先创建一个 provider 包
- 然后创建 VideoSqlProvider 类

```java
public class VideoSqlProvider {
    
    public String updateVideo(final Video video) {
        return new SQL(){{
            UPDATE("Video");
            if(video.getVideoAuthor() != null) {
                SET("video_author = #{videoAuthor}");
            }
            if(video.getVideoName() != null) {
                SET("video_name = #{videoName}");
            }
            WHERE("video_id = #{videoId}");
        }
        }.toString();
    }
    
}
```

它指明了要更新那个表，并且根据 Java 对象的属性是否为 null 生成 SQL 语句，最后设置更新的条件。



# 在数据访问层使用

通过使用 @UpdateProvider 注解指明使用哪个 Provider，以及使用哪个方法动态生成 SQL 语句。

```java
//@Update("UPDATE Video SET video_author = #{videoAuthor} WHERE video_id = #{videoId}")
@UpdateProvider(type = VideoSqlProvider.class, method = "updateVideo")
int updateVideo(Video video);
```

除了 @UpdateProvider 之外，还有：

```java
@DeleteProvider()
@InsertProvider()
@SelectProvider()
```



# 总结

略。