---
title: Spring Boot 增删改查
date: 2018-11-17
categories: Spring Boot
---

使用注解的方式在 Spring Boot 中进行增删改查。

<!--more-->

进行 CRUD 的前提是连接数据库，而连接数据库、执行 SQL 语句都需要 Mybatis 完成，所以必须先引入 Mybatis 的相关依赖，并进行配置。

在上一篇文章中，已经完成 Mybatis 的相关配置，也写了一个查询接口进行测试，现在完成其他操作。



# 数据库字段下划线和 Java 实体类映射

在开始之前，现在 application.properties 中添加配置：

```properties
#mybatis 下划线转驼峰配置，两者任选其一
#mybatis.configuration.mapUnderscoreToCamelCase=true
mybatis.configuration.map-underscore-to-camel-case=true
```

这是一个技巧。因为数据库中的字段习惯上是使用下划线的，而 Java 中使用驼峰命名法，如果不加这个配置，就需要像上一篇文章那样进行转换：

```java
@Results({
    @Result(column = "video_id",property = "videoId"),
    @Result(column = "video_name",property = "videoName"),
    @Result(column = "video_author",property = "videoAuthor")
})
```

加了之后上面这一堆东西就不用写了。



# 控制台打印 SQL 语句

在 application.properties 中添加：

```properties
#控制台打印 SQL 语句
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```

这是另一个技巧，方便调试。



# 改

## 在 Mapper 接口中创建方法

```java
@Update("UPDATE Video SET video_author = #{videoAuthor} WHERE video_id = #{videoId}")
int updateVideo(Video video);
```

**注意：这里是根据 videoId 修改 videoAuthor，但是接收的参数不是这两个参数，而是一个 Video 对象。**

这样写形参看起来比较短，另外刚才写的是一个 String 和一个 int 类型的形参，还出现了错误。



## 创建 service 包

- 这个包属于业务层，在这个包下创建一个 VideoService 接口
- 然后创建一个 impl 包
- 在 impl 包下创建 VideoServiceImpl 类实现 VideoService 接口。

```java
@Service
public interface VideoService {
    int updateVideo(Video video);
}
```

```java
@Service
public class VideoServiceImpl implements VideoService {

    @Autowired
    private VideoMapper videoMapper;

    @Override
    public int updateVideo(Video video) {
        return videoMapper.updateVideo(video);
    }
}
```

### 注意点

- 在 VideoServiceImpl 类中注入 VideoMapper 而不是在 VideoService 接口中
- @Service 注解只需要加在实现类中



## 编写 Controller

- 注入的是 VideoService 而不是它的实现类
- 接收到参数之后，不是直接交给业务层，而是创建一个对象传递过去

```java
@RestController("/api/v1/video")
public class VideoController {

    @Autowired
    private VideoService videoService;

    @PutMapping("update")
    public int update(String videoAuthor, int videoId) {
        Video video = new Video();
        video.setVideoAuthor(videoAuthor);
        video.setVideoId(videoId);
        return videoService.updateVideo(video);
    }
}
```



## 使用 postman 测试

注意使用的是 PUT 方法，返回的是受影响的条数。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-11-17_16-58-02.png)



# 增

先总结一下，新增一个 CRUD 操作的步骤：

- 在 Mapper 类中添加方法（使用注解访问数据库）
- 在业务层添加方法，调用 Mapper 对象的方法
- 在控制层编写接口，将参数传递给业务层

但是对于**增**操作而言，有一些不同。

其他操作返回*受影响的条数*即可，但是增加数据的话，可能需要获取到这条数据插入数据库时的自增主键，比如插入一条订单，需要获取到它的订单号。

下面看代码：

## 数据访问层

```java
@Insert("INSERT INTO Video (video_name, video_author) VALUES (#{videoName}, #{videoAuthor})")
@Options(useGeneratedKeys=true, keyProperty="videoId", keyColumn="video_id")
int addVideo(Video video);
```

使用 @Options 注解就可以获取到自增 id，**注意 keyProperty 才是实体类的字段名，千万不要搞反了**。



## 业务层

```java
@Override
public int addVideo(Video video) {
    int rows = videoMapper.addVideo(video);
    int id = video.getVideoId();
    System.out.println("新添加的视频的 id:" + video.getVideoId());
    return rows;
}
```

通过 getVideoId() 方法就可以取得自增 id，在业务层和控制层都可以获取到，因为这是同一个对象。



## 控制层

```java
@PostMapping("insert")
public int addVideo(String videoName, String videoAuthor) {
    Video video = new Video();
    video.setVideoName(videoName);
    video.setVideoAuthor(videoAuthor);
    return videoService.addVideo(video);;
}
```



# 删

## 数据访问层

```java
@Delete("DELETE FROM Video WHERE video_id = #{id}")
int delete(int id);
```



## 业务层

```java
@Override
public int delete(int id) {
    return videoMapper.delete(id);
}
```



## 控制层

注意需要使用 DELETE 方法提交

```java
@DeleteMapping("delete")
public int delById(int id) {
    return videoService.delete(id);
}
```



# 查

## 数据访问层

```java
@Select("SELECT * FROM Video")
List<Video> findAll();
```



## 业务层

```java
@Override
public List<Video> findAll() {
    return videoMapper.findAll();
}
```



## 控制层

```java
@GetMapping("page")
public Object findAll() {
    return videoService.findAll();
}
```



# 需要注意的细节

在终端执行 SQL 语句时，有时候需要使用单引号：

```sql
UPDATE Video SET video_author = '李四' WHERE video_id = 6
```

但是在数据访问层，使用注解时：

```java
@Update("UPDATE Video SET video_author = #{videoAuthor} WHERE video_id = #{videoId}")
```

是不需要单引号的，如果加了就会出错，一定要注意。



# 总结

这些东西在暑假已经学过，而且也用过一遍了，现在还是不能不看视频写出来，而且还是会犯错。