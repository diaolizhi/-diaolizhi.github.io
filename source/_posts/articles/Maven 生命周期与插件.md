---
title: Maven 生命周期与插件
date: 2019-2-10
categories: Maven
---



生命周期是对项目进行的一系列操作，插件用来完成这些操作。

<!--more-->

# 概述

构建一个项目包括：清理文件 -> 编译 -> 测试 -> 部署。

因为每一个项目都要进行类似的操作，所以 Maven 抽象出了**生命周期**。

光是存在生命周期并没有什么意义，而且生命周期**只是定义了**一系列操作，所以在实际的使用中，需要对项目进行某个具体的操作时，将由某个插件来完成这项任务。

抽象**生命周期**的好处是：制定了规范，只要遵守规范 Maven 就可以自动帮我们完成很多事情。

**抽象**生命周期的好处是：便于拓展。



# 三套生命周期

三套生命周期包括：clean、default、site，每个生命周期又包括多个阶段。

后一个阶段需要前一个阶段先完成。比如执行第 3 个阶段，Maven 会先执行前面第 1、2 个阶段。



## clean 生命周期

clean 生命周期的目的是清理项目，它包括三个阶段：

- pre-clean 执行一些清理前需要完成的工作
- clean 清理上一次构建生成的文件
- post-clean 执行一些清理后需要完成的工作



## default 生命周期

default 生命周期是真正构建时需要执行的，比如编译就是在这个生命周期里面的。

话说回来，为什么要有三个生命周期呢？我自己的理解是：构建项目时需要执行的任务在 default 生命周期里，但是清理文件这一工作并不是每一次都要执行的，或者是说，清理文件这件事是和编译、打包项目相独立的。所以，他们分为了两个生命周期。site 生命周期同理。逻辑上独立生命周期可以方便理解，实际中使用中，独立生命周期也有好处，比如不是当这一次不需要清理文件时，就不需要加 `clean` 命令。



default 生命周期所包括的阶段：

- validate
- initialize
- generate-sources
- process-sources 处理项目主资源文件。一般来说，是对 src/main/resources 目录的内容进行变量替换等工作后，复制到项目输出的主 classpath 目录中。
- generate-resources
- process-resources
- compile 编译项目的主源码。一般来说，是编译 src/main/java 目录下的 Java 文件至项目输出的主 classpath 目录中。
- process-classes
- generate-test-sources
- process-test-sources
- generate-test-resources
- process-test-resources
- test-compile 编译项目的测试代码。一般来说，是编译 src/test/java 目录下的 Java 文件至项目输出的测试 classpath 目录中
- process-test-classes
- test 使用单元测试框架运行测试，测试代码不会被打包或部署。
- prepare-package
- package 接受编译好的代码，打包成可发布的格式，如 JAR
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install 将包安装到 Maven 本地仓库，供本地其他 Maven 项目使用。
- deploy 将最终的包复制到远程仓库，供其他开发人员和 Maven 使用。



[官方解释](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)



## site 生命周期

site 生命周期的目的是建立和发布项目站点，Maven 能够基于 POM 所包含的信息，自动生成一个友好的站点，方便团队交流和发布项目信息。



site 生命周期所包含的阶段：

- pre-site 执行一些在生成项目站点之前需要完成的工作
- site 生成项目站点文档
- post-site 执行一些在生成项目站点之后需要完成的工作
- site-deploy 将生成的项目站点发布到服务器生



# 命令行与生命周期

- 命令 `mvn clean` 调用 clean 生命周期的 clean 阶段。实际执行的是 clean 生命周期的 pre-clean 和 clean 阶段。也就是说，调用某个阶段，之前的阶段都会被调用。
- 命令 `mvn clean install` 实际执行的是 clean 生命周期的 pre-clean、clean 阶段，以及 default 生命周期的从 validate 至 install 的所有阶段。



# 插件目标

前面说了，生命周期只是定义了规范，具体执行某一项操作是通过插件来完成的。

有些操作会比较相似，这样也就有一些代码可以复用。

现在的问题是：如果每一项任务都对应一个插件，代码就不好复用，所以这样不可取。

所以，一个插件有多个目标，一个目标用来执行生命周期中的一个任务。



# 插件绑定

Maven 抽象出了生命周期，假设现在也有了插件，但是它们两者似乎并没有任何关系。

所以就需要插件绑定，将二者联系起来。



## 内置绑定

Maven 为一些**生命周期阶段**绑定了插件的目标。

比如 default 生命周期的 compile 阶段跟 maven-compiler-plugin 插件的 compile 目标绑定了。



## 自定义绑定

有时候我们需要执行某项任务，但是并没有内置绑定插件目标来实现它。

此时就需要自定义绑定插件来完成。

这里需要注意的是 `executions` 元素，它下面的每一个 `execution` 子元素用来配置一个任务。

其中 `phrase` 指的是生命周期的某一个阶段，`goal` 指的是插件的某一个目标。

```xml
<build>
   <plugin>
       <groupId>org.apache.maven</groupId>
       <artifactId>maven-source-plugin</artifactId>
       <version>2.1.1</version>
       <executions>
         <execution>
           <id>id</id>
           <phase>verify</phase>
           <goals>
             <goal>java</goal>
           </goals>
         </execution>
       </executions>
 	</plugin>
</build>
```



如果没有 `phase` 标签，也不过出错。这是因为很多插件的目标在编写时已经定义了默认的绑定阶段，所以这里可以不写。



## 查看插件信息

使用下面这条命令：

（如果省略了版本信息，Maven 将自动获取最新版本来进行表述）

```
mvn help:describe -Dplugin=org.apache.maven.plugins:maven-source-plugin:2.1.1 -Ddetail
```

查看默认绑定阶段：`Bound to phase:`

```
...
  Implementation: org.apache.maven.plugin.source.TestSourceJarNoForkMojo
  Language: java
  Bound to phase: package

...
```



### 使用前缀

```
mvn help:describe -Dplugin=compiler
```



### 指定插件目标

```
mvn help:describe -Dplugin=compiler -Dgoal=compile
```



### 输出更详细的信息

加了 `-Ddetail`，将插件目标读取的属性列出。

```
mvn help:describe -Dplugin=compiler -Dgoal=compile -Ddetail
```



# 插件配置



## 命令行插件配置

用户可以在 Maven 命令中使用 -D 参数，并伴随一个参数建=参数值的形式，来配置插件目标的参数。

例如：maven-surefire-plugin 提供了一个 maven.test.skip 参数，当其值为 true 时，将会跳过测试。

```
mvn install -Dmaven.test.skip=true
```



参数 -D 是 Java 自带的，其功能是通过命令行设置一个 Java 系统属性，Maven 会在需要时读取该属性的值，实现插件参数的配置。



## pom 中插件全局配置

例如，为 maven-compiler-plugin 插件配置：

```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.0</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```



## pom 中插件任务配置

例子：

```xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <artifactId>maven-antrun-plugin</artifactId>
        <version>1.8</version>
        <executions>
          <execution>
            <phase> <!-- a lifecycle phase --> </phase>
            <configuration>
            <tasks>
                <echo>xxxxxx</echo>
            </tasks>
            </configuration>
            <goals>
              <goal>run</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```



# 从命令行调用插件

之前执行的命令，比如：`mvn install` 是执行生命周期中的某一个阶段。

但是 Maven 也支持直接执行执行插件的某个目标，比如：

```
mvn dependency:tree
```



# 插件解析机制

刚说了从命令行调用插件的某个目标，但是话说回来，Maven 定义一个插件，应该也是使用坐标的，为什么 Maven 可以识别上面的 `dependency` 呢？

下面慢慢说。



## 插件仓库

如果在本地找不到插件的话，将会到远程仓库查找。

这跟查找依赖的方式很像，但是查找依赖的远程仓库跟查找插件的远程仓库是不一样的，它们的配置方式也是不一样的。

```xml
<project>
  ...
  <pluginRepositories>
    <pluginRepository>
      <id>apache.snapshots</id>
      <url>http://repository.apache.org/snapshots/</url>
      <!-- The releases element here is due to an issue in Maven 2.0 that will be
           fixed in future releases. This should be able to be disabled altogether. -->
      <releases>
        <updatePolicy>daily</updatePolicy>
      </releases>
      <snapshots>
        <updatePolicy>daily</updatePolicy>
      </snapshots>
    </pluginRepository>
  </pluginRepositories>
  ...
</project>
```

或

```xml
<settings>
  ...
  <profiles>
    <profile>
      <id>apache</id>
      <pluginRepositories>
        <pluginRepository>
          <id>apache.snapshots</id>
          <name>Maven Plugin Snapshots</name>
          <url>http://repository.apache.org/snapshots/</url>
          <releases>
            <enabled>false</enabled>
          </releases>
          <snapshots>
            <enabled>true</enabled>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
  </profiles>
  ...
</settings>
```



## 插件的默认 groupId

如果是官方插件（即如果其 groupId 是 org.apache.maven.plugins）就可以省略 groupId 配置，但是不推荐省略，免得别人不理解看不出来。



## 解析插件版本

- 对于核心插件，Maven 的超级 POM 中设置了版本。因此，即使用户不加任何配置，它们的版本就确定了。
- 对于非核心插件，又没有指定版本，Maven 将会检查所有仓库中的可用版本，然后做出选择。

建议指定一个明确的版本。



## 解析插件前缀

`~\.m2\repository\org\apache\maven\plugins\maven-metadata.xml` 和 `~\.m2\repository\org\codehaus\mojo\maven-metadata.xml` 存储了插件和前缀的对应关系。

随便举个例子：

```xml
<plugin>
    <name>Appfuse Maven Plugin</name>
    <prefix>appfuse</prefix>
    <artifactId>appfuse-maven-plugin</artifactId>
</plugin>
```



# 总结

原来生命周期不仅仅是 `clean`、`install` ...，而且生命周期分为三套。

原来插件是生命周期的具体实现，原来插件目标是与生命周期的某个阶段相绑定的。