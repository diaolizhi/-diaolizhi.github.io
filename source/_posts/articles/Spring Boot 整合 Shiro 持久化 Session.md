---
title: Spring Boot 集成 Shiro 持久化 Session
date: 2018-12-08
categories: Spring Boot
---

避免每次重启服务器，用户都要重新登录。

<!--more-->

# 自定义 SessionDAO

```java
public class MySessionDAO extends EnterpriseCacheSessionDAO {

    @Autowired
    private SessionMapper sessionMapper;

    /** 
    * @Description: 创建 Session 并存储 
    * @Param: [session] 
    * @return: java.io.Serializable 
    * @Author: diaolizhi
    * @Date: 2018/12/8 
    */ 
    @Override
    protected Serializable doCreate(Session session) {

        Serializable sessionId = generateSessionId(session);
        assignSessionId(session, sessionId);
        sessionMapper.addSession(sessionId, SerializableUtils.serialize(session));
        return session.getId();
    }

    /** 
    * @Description: 读取 Session 
    * @Param: [sessionId] 
    * @return: org.apache.shiro.session.Session 
    * @Author: diaolizhi
    * @Date: 2018/12/8 
    */ 
    @Override
    protected Session doReadSession(Serializable sessionId) {

        MySession mySession = sessionMapper.readSession(sessionId);

        if(mySession == null) return null;
        Session session = SerializableUtils.deserialize(mySession.getSession());
        return session;
    }

    /** 
    * @Description: 更新 Session，比如它的最后访问时间。Session 信息有变化时自动调用。
    * @Param: [session] 
    * @return: void 
    * @Author: diaolizhi
    * @Date: 2018/12/8 
    */ 
    @Override
    protected void doUpdate(Session session) {
        if(session instanceof ValidatingSession && !((ValidatingSession)session).isValid()) {
            return; //如果会话过期/停止 没必要再更新了
        }
        sessionMapper.updateSession(session.getId(), SerializableUtils.serialize(session));

        return;
    }

    /** 
    * @Description: 删除 Session。当调用 subject.logout() 时自动调用 
    * @Param: [session] 
    * @return: void 
    * @Author: diaolizhi
    * @Date: 2018/12/8 
    */ 
    @Override
    protected void doDelete(Session session) {
        sessionMapper.deleteSession(session.getId());
    }
}
```



# 创建 Session 实体类

```java
public class MySession {

    private String id;
    private String session;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getSession() {
        return session;
    }

    public void setSession(String session) {
        this.session = session;
    }
}
```






# 序列化工具类

```java
public class SerializableUtils {

    public static String serialize(Session session) {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(bos);
            oos.writeObject(session);
            return Base64.encodeToString(bos.toByteArray());
        } catch (Exception e) {
            throw new RuntimeException("serialize session error", e);
        }
    }
    public static Session deserialize(String sessionStr) {
        try {
            ByteArrayInputStream bis = new ByteArrayInputStream(Base64.decode(sessionStr));
            ObjectInputStream ois = new ObjectInputStream(bis);
            return (Session)ois.readObject();
        } catch (Exception e) {
            throw new RuntimeException("deserialize session error", e);
        }
    }
}
```



# 添加依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.2</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- 阿里巴巴数据源 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>
```



# 添加配置

```properties
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/shiro-login?useUnicode=true&characterEncoding=utf-8
spring.datasource.username =用户名
spring.datasource.password =密码
#如果不使用默认的数据源（com.zaxxer.hikari.HikariDataSource）
spring.datasource.type =com.alibaba.druid.pool.DruidDataSource
```



# Session Mapper

```java
@Mapper
public interface SessionMapper {

    @Insert("INSERT INTO session (id, session) VALUES (#{id}, #{session})")
    int addSession(@Param("id") Serializable id, @Param("session") String session);

    @Select("SELECT * FROM session WHERE id = #{id}")
    MySession readSession(Serializable id);


    @Update("UPDATE session SET session = #{session} WHERE id = #{id}")
    void updateSession(@Param("id") Serializable id, @Param("session") String session);

    @Delete("DELETE FROM session WHERE id = #{id}")
    void deleteSession(Serializable id);
}
```




# Shiro 添加配置

在 Shiro 配置类中添加：

```java
@Bean
public SessionDAO sessionDAO() {
    return new MySessionDAO();
}

@Bean(name="sessionManager")
public SessionManager defaultWebSessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();

    //定时清除无效的session
    sessionManager.setSessionValidationSchedulerEnabled(true);
    //删除无效的session
    sessionManager.setDeleteInvalidSessions(true);

    sessionManager.setSessionDAO(sessionDAO());

    return sessionManager;
}
```



# 创建数据表

```sql
DROP TABLE IF EXISTS `session`;

CREATE TABLE `session` (
  `id` varchar(200) NOT NULL,
  `session` varchar(2000) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



# 测试

给 Session 设置属性，然后重启应用，测试属性是否存在。

```java
@GetMapping("/setA")
public Object setA() {
    Subject subject = SecurityUtils.getSubject();
    Session session = subject.getSession();
    session.setAttribute("label", "aa");
    return "设置成功";
}

@GetMapping("/getA")
public Object getA() {
    return SecurityUtils.getSubject().getSession().getAttribute("label");
}
```

**注意：Shiro 不会自动为浏览器创建 Session。**

只有调用```Session session = subject.getSession();```之后才会创建 Session。

如果 Session 已经存在，那么调用这句就不会再次创建，而是直接获取。



# 总结

核心就是自定义 SessionDAO，然后配置。