---
title: Maven 的坐标与依赖
date: 2018-06-14
categories: Maven
---

书上还有不少东西我看不懂，暂时放着。

<!--more-->



# 坐标详解

每一个 Maven 项目都有一个 pom.xml，这个文件中就配置了这个 Maven 项目的 groupId、artifactId、name（给用户看的项目名称，非必须）等等。

groupId、artifactId 和 version 就是一个 Maven 项目的基本坐标，坐标在空间中定位到唯一的一点，也就是说，通过坐标可以找到唯一的 Maven 项目。

## groupId

首先要明确“当前 Maven 项目”和“实际项目”两个概念，实际项目可以是一个完整的网上购物系统，而这个“实际项目”可以包含多个 Maven 项目，没有真正开发过，先记住。

groupId 是在 Maven 项目中配置的，所以**它的含义是：当前 Maven 项目隶属的实际项目。**

通常的格式：**公司域名倒序+项目名**

比如：com.diaolizhi.projectA

其中的 projectA 就是项目名。

## artifactId

这个元素配置的就是 **Maven 项目的 id**，推荐的做法是使用实际项目名称作为 artifactId 的前缀，比如 projectA-core。

## version

定义当前 Maven 项目的**版本号**。

## packaging

定义 Maven 项目的打包方式，可以为 jar 或者 war。这个元素是可选的，如果没有定义，默认为 jar。



# 依赖的配置

如果项目中需要用到其他的 jar 包，不需要手动导入，只需要在 pom.xml 文件中配置依赖。

配置依赖的格式：

```xml
<dependencies>
  <dependency>
    <groupId>...</groupId>
    <artifactId>...</artifactId>
    <version>...</version>
    <type>...</type>
    <scope>...</scope>
    <optional>...</optional>
    <exclusions>
      <excluseion>
        ...
      </excluseion>
    </exclusions>
  </dependency>
</dependencies>
```

## groupId、artifactId 和 version

依赖的基本坐标，Maven 根据坐标才能找到需要的依赖。

## type

依赖的类型，对应 packaging 元素。大部分情况下，该元素不必声明，默认值为 jar。

## scope

依赖的范围。

## optional

依赖依赖是否可选。

## exclusions

用来排除传递性依赖。



# 依赖范围

Maven 在编译主项目时使用一套 classpath，编译和测试时使用一套 classpath，实际运行时又使用另一套 classpath。

依赖范围就是用来说明：添加的依赖要添加到哪些 classpath 中。

## compile

默认配置。在三种 classpath 中都生效。

## test

对于测试 classpath 有效。

## provided

对于编译和测试 classpath 中有效，但是运行项目的时候不用。

## runtime

在编译主代码时无效，对于测试和运行 classpath 有效。



# 传递性依赖

当前 Maven 项目依赖 projectA，而 projectA 有依赖于 projectB。

只需要在 pom.xml 中添加 projectA 的依赖，Maven 会自动依赖 projectB。

当前项目对 projectA 的依赖范围可能是 compile、test 等等。projectA 对 projectB 的依赖范围也有多种可能，那么问题来了，当前项目对 projectB 的依赖范围是什么呢？



|          | compile  | test | provided | runtime  |
| :------: | :------: | :--: | :------: | :------: |
| compile  | compile  |  -   |    -     | runtime  |
|   test   |   test   |  -   |    -     |   test   |
| provided | provided |  -   | provided | provided |
| runtime  | runtime  |  -   |    -     | runtime  |

左边第一列表示第一直接依赖范围，最上面一行表示第二依赖直接依赖范围，中间的表示传递性依赖范围。



# 依赖调解

假设有这样的依赖关系：A->B->C->X(1.0)、A->D->X(2.0)。不能把项目 X 引入两次，所以规定：路径最短者优先。1.0 的路径长度为 3，2.0 的路径长度为 2，所以 2.0 会被解析使用。

如果两者路径长度相同，那么哪个依赖先声明就使用哪个。



# 可选依赖

假设项目 projectD 依赖了 projectE 和 projectF，并且这两个依赖是可选依赖，也就是说 option 元素内是 true。

那么就算当前项目依赖了 projectD，projectE 和 projectF 也**不会**添加到当前项目，换句话说：可选依赖不会传递。

如果需要用到 projectE，就必须显式地声明它的依赖。



# 排除依赖

假设添加了 projectB 的依赖，projectB 又依赖于 projectC。

当我想使用另一个版本的 projectC 的时候，就可以使用依赖排除。

排除依赖的方法就是在 exclusion 元素里说明 projectC 的 groupId 和 artifactId，这里只需要这两个元素就足够了。 



# 归类依赖

多个依赖版本相同，如果要升级版本，一个一个地改版本号岂不是很麻烦？

为了解决这个问题，可以定义一个 Maven 属性，然后在 version 中使用这个属性。

要升级的时候，只需要改一次。

在 project 元素内定义：

```xml
<properties>
	<myapp.version>1.2</myapp.version>
</properties>
```

然后这样使用：

```xml
<version>${myapp.version}</version>
```


