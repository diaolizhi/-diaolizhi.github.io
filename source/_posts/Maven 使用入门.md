---
title: Maven 使用入门
date: 2018-06-11
categories: Maven
---

如何手动创建一个 Maven 工程。

<!--more-->

# 编写 POM

- 新建一个 hello-world 文件夹
- 在文件夹下新建一个 pom.xml 文件。这个文件用来定义项目的基本信息。代码如下：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <!-- The Basics -->
  <groupId>com.diaolizhi</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Hello World Project</name>

</project>
```

# 编写主代码

在 hello-world 文件夹下创建 src/main/java 文件夹，在 java 文件夹下创建 com/diaolizhi/HelloWorld.java 文件，代码如下：

```	java
package com.diaolizhi.helloworld;

public class HelloWorld
{
	public String sayHello() {
		return "Hello World";
	}

	public static void main(String[] args) {
		System.out.println(new HelloWorld().sayHello());	
	}
}
```

在主目录 hello-world 下打开命令行，运行：

```
mvn clean compile
```

这句话先清理输出目录 target/，compile 告诉 Maven 编译项目主代码。

# 编写测试代码

首先为项目添加依赖，添加之后的 pom.xml 文件：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
 
  <!-- The Basics -->
  <groupId>com.diaolizhi</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Hello World Project</name>

  <dependencies>
  	<dependency>
  		<groupId>junit</groupId>
  		<artifactId>junit</artifactId>
  		<version>4.7</version>
  		<scope>test</scope>
  	</dependency>
  </dependencies>

</project>
```

**在 根目录/src/test/java 文件夹下创建 HelloWorldTest.java**

```java
package com.diaolizhi.helloworld;

import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class HelloWorldTest
{
	@Test
	public void testSayHello()
	{
		HelloWorld helloWorld = new HelloWorld();
		String result = helloWorld.sayHello();
		assertEquals("Hellod World", result);
	}
}
```

注意：**测试类不用放在固定的包下，直接放在 src/test/java 目录下即可，而且第一行代码 package xx.xx.xxx; 和主类一样。**

## 执行测试

在根目录下运行 

```
mvn clean test
```

执行测试，结果发现有错误。

> 由于历史原因，Maven 的核心插件之一——compiler 插件默认只支持编译 Java 1.3，因此需要配置该插件使其支持 Java 5。

在 pom.xml 中加入如下的代码：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

这样就可以成功执行测试了。

# 打包和运行

- 单纯地打包，得到的 jar 包位于 target/ 目录下

```
mvn clean package
```

- 打包并放入本地仓库，以便其他项目使用

```
mvn clean install
```

## 生成可运行的 jar 包

HelloWorld 类是有一个 main 方法的，默认打包生成的 jar 是不能够直接运行的，因为带有 main 方法的类信息不会添加到 manifest 中。为了生成可执行的 jar 文件，需要借助 maven-shade-plugin，配置该插件如下：

（plugin 元素在 pom.xml 中的 project 下的 build 下的 plugins 标签下）

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>1.2.1</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <mainClass>com.diaolizhi.helloworld.HelloWorld</mainClass>
                    </transformer>
                </transformers>
            </configuration>
        </execution>
    </executions>
</plugin>
```

再次运行

```
mvn clean package
```

就会自动生成可执行 jar 包，位于 target/ 目录下， original 开头的包是原始的包，另一个才是可执行的。

使用下面的命令执行 jar 包：

```
java -jar target\hello-world-1.0-SNAPSHOT.java
```

