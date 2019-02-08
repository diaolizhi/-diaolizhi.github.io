---
title: Maven 仓库
date: 2019-2-8
categories: Maven
---

了解 Maven 中的仓库。

<!--more-->



# 使用 Maven 添加依赖

一般来说，要在 Java 中使用别人的写好的代码，通常是下载别人提供的 jar 包，然后引入当前的项目来使用。

这么做的问题是：

- 需要手动下载 jar 包
- 可能这个 jar 包又依赖于其他的 jar 包
- ...



而在 Maven 中一切都变得很简单，创建一个 Maven 项目之后，假设我们需要使用 okhttp，只需要在谷歌搜索 okhttp，复制下面这段代码到我们的 pom.xml 中即可。

```xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.13.1</version>
</dependency>
```



当然了，实际上还是需要下载 jar 包的，毕竟“存在”才可以“使用”，不过这些工作不需要我们来完成。

在 pom.xml 添加 okhttp 的依赖之后，Maven 会替我们解决依赖的问题，并下载需要的 jar 包。



# Maven 背后的工作

我们只是告诉 Maven 需要 okhttp 这个依赖，然后呢？Maven 会去哪里下载这个依赖呢？

实际上 Maven 中有一个默认的地址，它默认就会去那里寻找 jar 包。

不过，暂时不关心如何查找，先关心在我们电脑中发生的事情。



Maven 下载 jar 包之后，统一保存到 `~/.m2/repository` 目录下，这个目录称为本地仓库。



# 仓库的概念

我认为仓库其实就是一个文件夹而已。仓库中存在文件，实际上就是文件夹中存放了一大堆 jar 包（也存在其他文件，**我总是说 jar 包是因为我对其他类型文件用得比较少，认识不深**）。



# 仓库的规范（布局）

为了更好的管理这些 jar 包，仓库有一套规范。否则，假设所有 jar 包都存在一个文件夹中，当 jar 包文件特别多时，查找就需要花很多的时间。（规范的作用还有很多，我还体会不到）



那么这套规范（或者是说布局）是什么？

简单说来，就是对于每一个 Maven 项目所构建的文件，就好比一个 jar 包，都在仓库中有一个明确且独立的位置。换句话说，根据一个 Maven 项目的坐标，就可以确定这个项目所生成的 jar 包的位置。

比如：

项目坐标：

```xml
<groupId>com.diaolizhi</groupId>
<artifactId>tieba-tool</artifactId>
<version>1.3-SNAPSHOT</version>
```

实际的文件路径：

`C:\Users\xxx\.m2\repository\com\diaolizhi\tieba-tool\1.3-SNAPSHOT`



由此可知，**大致**的对应关系是：

`groupId/artifactId/version/artifactId-version.packaging`



**实际上，仓库的规范（布局）不仅仅是根据上面的 groupId、artifactId 和 version，这里只是举个例子。**



# 仓库的分类

- 仓库
  - 本地仓库
  - 远程仓库
    - 中央仓库
    - 私服
    - 其他公共库



# 本地仓库

就是默认的 `~/.m2/repository` 那么目录而已。



## 修改本地仓库的位置

编辑 `~/.m2/setting.xml` 文件。（先从 Maven 的安装目录将该文件复制过来）

```xml
<settings>
    <localRepository>D:\java\repository</localRepository>
</settings>
```



# 中央仓库

可以看成一台服务器，上面存放了非常多的 Maven 项目。

当本地仓库找不到某个依赖时，就会到中央仓库去寻找。



# 私服

架设在局域网的仓库服务。

特点：

- 当 Maven 需要下载时，会向私服发送请求
- 如果私服不存在该文件，就向外部的远程仓库请求，并且缓存到私服上，以供下一次使用
- 从本地上传文件到私服上供局域网内用户使用



# 配置远程仓库

[官方文档 - Setting up Multiple Repositories](https://maven.apache.org/guides/mini/guide-multiple-repositories.html)

修改 pom.xml：

```xml
<project>
...
  <repositories>
    <repository>
      <id>my-repo1</id>
      <name>your custom repo</name>
      <url>http://jarsm2.dyndns.dk</url>
      <releases>
          <enabled>true</enabled>
          <updatePolicy>daily</updatePolicy>
          <checksumPolicy>ignore</checksumPolicy>
      </releases>
      <snapshots>
          <enabled>false</enabled>
      </snapshots>
      <layout>default</layout>
    </repository>
    <repository>
      <id>my-repo2</id>
      <name>your custom repo</name>
      <url>http://jarsm2.dyndns.dk</url>
    </repository>
  </repositories>
...
</project>
```

- id 用来声明一个远程仓库
  - 如果 id 设置为 central 将会覆盖中央仓库的配置
- name 是远程仓库的名字，用来给人看的
- url 表示远程仓库的地址
- releases 对应发行版本，snapshots 对应快照版本
  - enabled 表示是否对发行版本/快照版本进行下载
    - 根据上面的配置：从远程仓库下载发行版本，但是不会下载快照版本
  - updatePolicy 配置从远程仓库检查更新的频率
    - daily 每天检查一次
    - never 从不
    - always 每次构建时更新
    - interval: X 每个 X 分钟检查更新一次（X 为任意整数）
  - checksumPolicy 配置 Maven 检查校验和文件的策略
    - 默认值 warn 校验失败输出警告信息
    - fail 校验失败就让构建失败
    - ignore 完全忽略校验和错误



# 远程仓库的认证

[参考文档](https://maven.apache.org/guides/mini/guide-encryption.html)

修改 settings.xml：

```xml
<settings>
...
  <servers>
...
    <server>
      <id>my-proj</id>
      <username>repo-user</username>
      <password>repo-pwd</password>
    </server>
...
  </servers>
...
</settings>
```



远程仓库是在 pom.xml 中配置的，而认证是在 settings.xml 中配置的。

这么做也是很好理解的，如果账号密码写在 Maven 项目中，那么这个项目发给别人的时候就可能造成账号的泄漏，所以写在本地的 settings.xml 是相对安全的。



另外需要注意，这里的 `id` 和 pom.xml 中配置的远程仓库的 `id` 需要对应起来。



# 部署至远程仓库

[参考文档](https://maven.apache.org/pom.html#Distribution_Management)

在 settings.xml 中添加：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      https://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <distributionManagement>
    <repository>
      <id>corp1</id>
      <name>Corporate Repository</name>
      <url>scp://repo/maven2</url>
    </repository>
    <snapshotRepository>
      <id>propSnap</id>
      <name>Propellors Snapshots</name>
      <url>sftp://propellers.net/maven</url>
    </snapshotRepository>
    ...
  </distributionManagement>
  ...
</project>
```

`distributionManagement` 包括两个子元素： `repository`、`snapshotRepository`，分别对应发布版本的仓库和快照版本的仓库。



**上面的配置是在 settings.xml 中的，当执行 `mvn clean deploy` 时，会判断当前项目是发布版本还是快照版本，部署到对应的远程仓库中。**



# 快照版本

一个人写项目的时候体现不出快照版本的好处，当两个人合作的时候就体现出来了。

小明开发项目 A，小红开发项目 B。项目 B 依赖于项目 A，并且项目 A 正在开发。

怎么保证小红时刻可以获取到最新的项目 A 呢？

- 每次下载源码进行编译。这显然不合理，不仅浪费时间，而且编译出错也不好解决。
- 项目 A 部署时修改版本号。这可以，但是有些麻烦，每次上传需要修改版本号，下载也需要修改版本号。
- 使用快照。上传和下载时都使用： x.y-SNAPSHOT。这样做的神奇之处在于，上传到远程仓库时，名字不是 x.y-SNAPSHOT 而是 x.y-时间戳。另一方面，需要下载最新版本的项目 A 时，Maven 会检查远程仓库是否有更新，如果有就下载。
  - 什么时候会下载最新版本的项目 A 呢？
    - 根据上面提到的 updatePolicy，默认每天更新一次
    - 或者使用 `mvn clean install-U`



# 解析依赖的机制

当我们在 pom.xml 中添加依赖的时候，Maven 是如何判断从哪里得到这个依赖的呢？

- 如果依赖的范围是 system，直接从本地文件系统解析该文件，找到就结束
- 根据依赖计算仓库路径，尝试在本地仓库查找该文件，找到就结束
- 上面两种方式都找不到时，如果依赖的版本号是确定的，比如 1.2、2.1-beta-1，将遍历所有远程仓库，发现后就下载，然后结束
- 如果版本号指定的是 RELEASE 或者 LATEST 或者 SNAPSHOT，Maven 会根据更新策略，访问远程仓库的元数据：`groupId/artifactId/maven-metadata.xml`，将其与本地仓库的元数据合并后，计算出真实的版本号，然后在从本地仓库或者远程仓库中查找。



注意点：

- 快照版本在远程仓库中带时间戳，但是从远程仓库下载之后时间戳又变成了 SNAPSHOT。
- RELEASE 表示最新的发布版本，LATEST 表示最新版本（包括快照版本）
- 不要使用 RELEASE 和  LATEST 



# 镜像

修改 settings.xml：

```xml
<settings>
  ...
  <mirrors>
    <mirror>
      <id>UK</id>
      <name>UK Central</name>
      <url>http://uk.maven.org/maven2</url>
      <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
  ...
</settings>
```

`mirrorOf` 表示这个是哪个仓库的镜像，这里的 `central` 指的就是中央仓库。

因为私服可以访问外部仓库，所以直接指定私服作为镜像也是可以的，反正找不到的时候它会向外部寻找。



`mirrorOf` 中间可以是：

- `*` 匹配所有远程仓库
- `external: *` 匹配所有不在本机上的远程仓库
- `repo1, repo2` 匹配仓库 repo1 和 repo2
- `*, !repo1` 匹配所有远程仓库，repo1 除外





# 总结

- 仓库用来存放 Maven 生成的文件，且仓库的目录有自己的规范。
- 仓库可以大致分为本地仓库和远程仓库。
- 快照是在开发时使用的。
- 使用镜像可以加快 Maven 的下载速度。

