---
title: Spring Boot 配置文件映射
date: 2018-11-15
categories: Spring Boot
---



如何读取配置文件中的信息。

<!--more-->

# 直接引用

-  在 application.properties 中添加配置信息

```properties
test.name = ZhangSan
```

- 在某个类（比如 TestController）中引用

```java
@Controller
/**
 * 引用配置文件需要添加下面这一句注解
 */
@PropertySource(value = "classpath:my.properties")
public class TestController {

    /**
     * 引用配置信息
     */
    @Value("${test.name}")
    private String name;

    @GetMapping("/test")
    @ResponseBody
    public String test() {
        // 使用配置信息
        return name;
    }
}
```



# 建立配置类

- 添加配置信息

```properties
test.name = ZhangSan
test.age = 18
```

- 新建配置类
  - 添加 @Configuration 注解
  - 添加 @PropertySource 注解
  - 使用 @Value 注解引用配置信息
  - 生成 get、set 方法

```java
/**
 * 使用 @Component 和 @Configuration 都会进行扫描，但是后者意义更准确
 */
//@Component
@Configuration
@PropertySource(value = "classpath:my.properties")
public class TestConfig {

    @Value("${test.name}")
    private String name;

    @Value("${test.age}")
    private String age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }
}
```

- 使用

```java
@Controller
@PropertySource(value = "classpath:my.properties")
public class TestController {

    /**
     * 注入配置类
     */
    @Autowired
    private TestConfig testConfig;

    @GetMapping("/test2")
    @ResponseBody
    public String test2() {
//        使用配置信息
        return testConfig.getName() + testConfig.getAge();
    }
}
```



# 总结

在开发时，一定要避免硬编码，一个显而易见的好处是：要修改某个配置时，不用到处修改代码，只是需要修改配置文件。

