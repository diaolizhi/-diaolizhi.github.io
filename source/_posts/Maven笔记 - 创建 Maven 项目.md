---
title: 创建 Maven 项目
date: 2019-2-8
categories: Maven
---

温故知新。

<!--more-->



# pom 文件

该文件的作用：

- 在茫茫 Java 类库中声明自己的身份
- 并且声明需要依赖的类库
- 说明项目将怎么样进行构建（比如说明入口文件）
- ...



# 简单的 pom.xml 示例

- xml 版本、编码方式声明
- project 是根元素
- 对于 Maven2 和 Maven3 来说 `modelVersion` 只能是 4.0.0
- groupId 域名倒序加项目名
- artifactId 模块名
- version 版本
- name 项目的描述

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.diaolizhi.second</groupId>
  <artifactId>myHelloWorld</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>myHelloWorld</name>
</project>
```



# Maven 项目规定

- 项目代码放在 `/src/main/java/com.xxx.groupId.artifactId` 目录下 
- 测试代码放在 `/src/test/java/com.xxx.groupId.artifactId` 目录下



# Maven 输出 

- 输出内容在 `/target/` 目录下，包括生成的 jar 包
- `classes` 目录下是编译生成的 class 文件



# 指定入口类

- 借助 maven-shade-plugin



# mvn clean

- 删除 target 目录



# mvn compile

- 编译项目主代码



# mvn test

- 执行测试用例



# mvn package

- 打包



# mvn install

- 打包并添加到本地仓库

