---
title: Java 包
date: 2017-12-7
categories: JavaSE
---

第五篇关于 Java 的笔记.
<!--more-->

# 1. 包的概述
## A. 什么是包
- 包由一个个 .java 文件组成, 或者说是一个个类组成了包.
- 一个包里面还可以包含多个包
- 包的具体体现就是文件夹

## B. 包的作用
- 解决类的同名问题
- 便于代码复用与维护

# 2. 包的创建和使用
## A. 如何使用包
- 导入特定的类或接口

```java
import 包名.类型名;
```

- 导入指定包中所有类型

```java
import 包名.*;
```

- Java 编译器会将 import 语句引入的包字符串拼接到标识符前(不懂 2017年12月7日 16:45:37)

## B. 将类放入自定义包中

```java
package top.mp;
```

## C. 编译运行

在 Temp.java 文件目录下执行

```
javac Temp.java
```

会在当前目录产生一个 Temp.class 文件. 但是执行

```
java Temp
```

得到的结果是*错误: 找不到或无法加载主类 top.mp.Temp*

正确的做法是: 在编译时

```
javac -d . Temp.java
```

执行时 

```
java top.mp.Temp
```

## D. Java 编程须知

- Java 从一开始就规定了代码的命名规范, 比如, 所有用户开发的包, 都不能以 java 和 javax 打头, 另外, **声明为 public 的类, 应该放到与其类名一致的文件中(也就是说, 如果 Temp 类声明为 public, 但是这个类所在的文件名字不是 Temp.java,  就会就无法编译)**, 与此同时, **以点号分隔的包名, 其实对应着响应的文件夹结构**, 这样一来, 给定一个完整的类型名, 就可以很快定位到其编译后的 .class 文件
- 当使用 java 命令运行一个 .class 文件时, **此文件必须放在与它所在的包名一致的文件夹层次结构之下**(如果例子中的 Temp.class 不是在 top/mp 下就无法运行, 而且运行的时候必须处于 top 的父级目录中, 使用 java top.mp.Temp 运行)
- 当使用 java 命令运行一个 .class 文件时, JVM 会默默地加载 JRE 中的多个包(通过 java -verbose Temp 命令可以看到)

# 3. jar 包
>[jar 包的创建与使用](http://jinxuliang.com/course2/ppt/show/5412dd3c137e430e98768373)
