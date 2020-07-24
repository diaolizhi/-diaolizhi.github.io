---
title: Maven 聚合与继承
date: 2019-2-11
categories: Maven
---



聚合是为了方便的构建多个项目，继承是为了简化 POM 的重复配置。

<!--more-->



# 聚合

一个实际的项目可能由多个“模块”组成，每个模块都是一个 Maven 项目。

这样就可能产生一个需求：同时构建多个 Maven 项目。此时如果进入到每个 Maven 项目的目录中，执行 mvn 命令进行构建就有点麻烦了。所以就有了“聚合”这个概念，聚合实际的作用就是同时构建多个 Maven 项目。



# 聚合的步骤

要聚合模块其实很简单，就是创建一个 Maven 项目，然后在这个项目的 pom.xml 文件中声明要聚合的模块。



## 目录结构

举例，moduleB和 moduleC 是普通的模块，moduleA 是聚合模块，目录结构如下：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-02-12_11-59-31.png)

需要注意的是：

- 聚合模块下没有 src 目录，只有一个 pom.xml 文件。毕竟它只是起聚合模块的作用，不需要实际代码。
- 目录结构不一定是这样的，下面的 pom.xml 还可以配置聚合模块所在的目录。



## moduleA 的 pom.xml

注意点 ：

- packaging 元素必须为 pom
- 建议填写 name 标签的内容，因为执行 mvn 命令时将会输出该标签下的内容。（如果没有写，将输出 artifactId）
- modules 下的每个 module 标签对应一个模块。**注意 module 标签中的是模块的相对路径，也就是对这个 pom.xml 的相对路径。**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.test.modules</groupId>
    <artifactId>moduleC</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>pom</packaging>
    <name>module A</name>
    <modules>
        <module>../moduleA</module>
        <module>../moduleB</module>
    </modules>
    
</project>
```



## 使用

在 moduleA 目录下执行 Maven 命令即可。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-02-12_12-26-23.png)



# 继承

这里的继承指的是 pom.xml 的继承，因为多个 Maven 项目可能存在多个相同的配置，使用继承就可以减少一些配置。



怎么做呢？

简单来说就是创建一个 Maven 项目，它的 pom.xml 作为父POM，然后在其他项目中通过坐标引入这个父项目，这样子POM将会继承父POM的一些配置。



## 举例

聚合与继承不一样，但是不冲突，刚才的 moduleA 是聚合模块，同时 moduleA 的 pom.xml 也可以作为父POM。



使用了继承之后的 moduleB 的 pom.xml：

注意点：

- 使用 parent 标签继承 moduleA
- 使用坐标定位 moduleA
- relativePath 对应 moduleA 的相对目录
  - 默认值是 `../pom.xml`
  - 如果在这里的目录找不到，将会在本地仓库查找
  - 最好写上 relativePath
- 在 moduleB 的 pom.xml 中不用写 `groupId` 和 `artifactId`，因为会从 moduleA 的 pom.xml 中继承。
  - 如果和父POM不一样，也可以写上去。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <artifactId>moduleB</artifactId>

    <parent>
        <groupId>com.test.modules</groupId>
        <artifactId>moduleA</artifactId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../moduleA</relativePath>
    </parent>
    
</project>
```



# 可以继承的 POM 元素

- groupId 项目组 ID
- version 项目版本
- description 项目的描述信息
- organization 项目的组织信息
- inceptionYear 项目的创始年份
- url 项目的 URL 地址
- developers 项目的开发者信息
- contributors 项目的贡献者信息
- distributionManagement 项目的部署配置
- issueManagement 项目的缺陷跟踪系统信息
- ciManagement 项目的持续集成系统信息
- scm 项目的版本控制系统信息
- mailingLists 项目的邮件列表信息
- properties 自定义的 Maven 属性
- dependencies 项目的依赖配置
- dependencyManagement 项目的依赖管理配置
- repositories 项目的仓库配置
- build 项目源码目录配置、输出目录配置、插件配置、插件管理配置等
- reporting 项目的报告输出目录配置、报告插件配置



# 依赖管理

上面说到 dependencies 是可以继承的，也就是说父POM中添加的依赖，子项目中也会引入。

这样一来有一些好处，不过也有一些不好的地方：如果某个子项目继承了这个父POM，但是这个子项目并不需要那么多的依赖，这样岂不是产生了不必要的引用？



dependencyManagement  的作用就体现出来了，在父POM中使用 dependencyManagement ：

```xml
<dependencyManagement>
    <dependencies>
        <!-- 随意添加的一个依赖 -->
        <dependency>
            <groupId>org.sonatype.aether</groupId>
            <artifactId>aether-parent</artifactId>
            <version>1.7</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

现在的效果是：

- 如果子项目中没有添加这个依赖，那么即使继承了父POM，子项目也不会引入这个依赖。

看起来没什么卵用，因为在子项目中还是要手动的添加依赖，但实际上还是有用的。因为在子项目中添加依赖时，可以省略 `version`，在父POM中做统一的版本管理，需要修改版本时只需要修改父POM中的 `version`。



# 使用 import

import 也是依赖范围的一种，只不过在 dependencyManagement  中才有用。

实际作用是：把上面的 moduleA 中的 dependencyManagement  合并到当前的 POM 的 dependencyManagement  元素中。（当然也可以使用复制、继承的方式实现）

注意点：

- type 是 pom
- scope 是 import

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.test.modules</groupId>
            <artifactId>moduleA</artifactId>
            <version>1.0-SNAPSHOT</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```



# 插件管理

插件管理与依赖管理类似，使用 pluginManagement 之后同样是按需引入。

插件的隐含信息：

- 版本。核心插件已经配置了版本
- 内置绑定。插件已经绑定了生命周期的阶段

```xml
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.5</source>
                    <target>1.5</target>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

同样建议在父POM中统一管理插件的版本。



# 超级POM

超级POM类似于 Java 的 Object 类，是所有类的基类。

Maven 项目中的 pom.xml 还可以自定义源码目录，但是不建议这么做，最好是按照约定来开发。

现在只需要知道有超级POM的存在，并且所有 pom.xml 都继承自它，所以有很多配置不用我们自己书写，就可以直接使用。



# 反应堆

反应堆这个翻译有点难以理解，大概可以理解成：多个存在聚合关系的项目在构建时，Maven 需要解读构建的顺序，解读得到的结构顺序称为反应堆。

比如 moduleA 是聚合模块，moduleB、moduleC 是普通模块，同时这两个模块继承自 moduleD。

假设 moduleA 中模块的声明顺序是：

moduleB -> moduleC -> moduleD

在构建过程的，Maven 得到的反应堆是 moduleD -> moduleB -> moduleC。

这是因为从上往下构建，发现某个模块依赖于另一个模块，那么 Maven 将会先构建被依赖的模块，也就是这里的 moduleD。

另外，moduleC 也依赖于 moduleD，但是由于 moduleD 已经被构建，所以直接构建 moduleC。



# 裁剪反应堆

对于一个聚合项目，有时可能只需要构建一部分的模块，而不是完整的反应堆。

Maven 的命令行选项：

- -am，--also-make 同时构建所列模块的依赖模块
- -amd，-also-make-dependents 同时构建依赖于所列模块的模块（谁依赖我，就构建谁）
- -pl， --projects \<arg\> 只构建指定的模块，模块间用逗号隔开
- -rf，-resume-from 从指定模块开始往下构建反应堆

这些选项还可以组合使用，由于理解不深，不举例了。



# 总结

多模块开发必须使用聚合。

使用继承可以简化项目的依赖和插件配置。