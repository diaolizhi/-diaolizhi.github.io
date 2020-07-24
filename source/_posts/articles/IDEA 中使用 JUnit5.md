---
title: IDEA 中使用 JUnit5
date: 2018-11-09
categories: JUnit
---

JUnit5 的简单使用。

<!--more-->

JUnit5 是 Java 中单元测试框架。

[JUnit5 用户指南](https://junit.org/junit5/docs/current/user-guide/)

[JUnit5 用户指南 - 中文版](https://sjyuan.cc/junit5/user-guide-cn/)



# 添加 Maven 依赖

学会如何使用一个框架最好的方法应该是看官方文档，然而即便是中文版我还是看得一脸懵逼。

官方文档的第二部分是**安装**，介绍了 JUnit5 中的依赖关系，但是在这里并没有直接指明在使用的时候应该引入哪些依赖。而是给出了多个示例项目：包括 Maven 和 Gradle 等方式创建的项目，通过这些示例可以看出该引入哪些项目。比如[GitHub - pom.xml](https://github.com/junit-team/junit5-samples/blob/r5.3.0/junit5-jupiter-starter-maven/pom.xml)。

可惜当我把依赖复制过去的时候 IDEA 总是报错：*Failed to read artifact descriptor for..*，导致我浪费了很多时间。

后面谷歌了一下：[解决方法](https://stackoverflow.com/questions/6642146/maven-failed-to-read-artifact-descriptor)，也就是执行一下 *mvn -U clean install*。

主要的代码：

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>${maven.compiler.source}</maven.compiler.target>

    <junit.jupiter.version>5.3.1</junit.jupiter.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>${junit.jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-params</artifactId>
        <version>${junit.jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>${junit.jupiter.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

JUnit5 包含很多个注解，有时候要使用某个注解就要添加特定的依赖。比如要使用 @ParameterizedTest 就要添加 junit-jupiter-params。



# 编写测试代码

通过 Maven 来使用 JUnit5，那么在创建项目的使用肯定是创建 Maven 项目，所以在 test 文件夹下的 java 文件夹下创建类，然后在类中编写测试方法来测试功能代码。

一个方法之所以是测试方法，是因为它加了注解，比如加了 @Test 的方法。

就像这样：

```java
public class Test2 {

    @Test
    void test1() {
        assertEquals(1, 1);
    }
    
}
```

test1 方法就是一个测试方法，在 IDEA 中可以运行这个方法，并得到测试结果。

需要注意的是 assertEquals() 是一个静态方法，直接写是没有自动提示的，要么通过 **类型.方法名** 的方法调用，要么这样导入：

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
```



## 测试自定义方法

在 main 文件夹下的 java 文件夹下创建类：

```java
public class JUnitTest {

    public int add(int a, int b) {
        return a + b;
    }

}
```

然后为它编写测试代码：

```java
@Test
void test2() {
    int res = new JUnitTest().add(1, 3);
    assertEquals(res, 4);
}
```



# 总结

JUnit5 还有很多注解，比如为测试方法设置参数、重复测试等，这些留着下次再说，这一次先了解最最最基本的使用。

