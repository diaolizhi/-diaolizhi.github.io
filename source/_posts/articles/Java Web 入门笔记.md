---
title: Java Web 入门笔记
date: 2018-05-31
categories: Java Web
---

关于 Servlet 和 Tomcat 的一些理解。

<!--more-->

以前我告诉自己，不要写诸如“某某软件是如何使用”的博客，因为我觉得这种文章没什么技术含量，一步一步的来，每一步都截图，不就完了吗？

到后来，发现自己装个软件也要费半天劲，装好了也不知道怎么用。比如这次的 Tomcat，通过 .exe 文件装好之后，也不知道该怎么在 IDE 里面使用...

我清楚了解到自己的智商，为了让自己下次少花一些时间，我决定花一些时间把这个过程记录下来。

# Windows 下安装 Tomcat

Tomcat 的下载就不说了，肯定不是在百度搜索找一个来安装，而是去官网下载安装，毕竟百度搜索出来的指不定是多久之前的版本。

在安装 Tomcat 有两种方法，一种是通过 .exe 文件来安装，这种方法很简单，我一开始也是这么干的。后来在网上看别人的文章的时候，看到别人说：一开始不要这么装，这样做只会徒增烦恼。而且后面在 YouTube 看视频的时候，别人也是通过另一种方法来安装的，**也就是下载压缩包，解压之后通过 .bat 文件来启动 Tomcat**。

Tomcat 也是需要 Java 运行环境的，所以电脑上需要安装 JDK，并且配置好环境变量。

Tomcat 自身也需要配置一个环境变量，但是我还没用到，所以就没管。

Tomcat 启动之后，通过 127.0.0.1:8080 就可以访问到它自带的页面了。

#  Servlet

先说说我对 Servlet 和 Tomcat 的理解。

首先这些都是服务器端的东西，而服务器的功能就是**接收客户端的请求，并返回指定内容给客户端**，比如 JSON 格式的数据，或者一个 HTML 文件。

这里的客户端具体来说，就是手机 App 或者浏览器。

说回服务器，服务器实际上是一台时刻运行的电脑，但是一台运行着的电脑并不一定是一台服务器。一台服务器，那么必须时刻准备着响应客户端的请求。不然一个浏览器向你这台电脑请求数据，你一点表示都没有，这算怎么回事。

想要响应客户端的请求，至少要存在一个进程，这个进程监听一个端口。之所以要监听一个特定的端口，是因为一台计算机可能提供多个服务，比如文件传输服务、收发邮件服务，这些服务必然是由不同的线程提供，但是客户端只知道这台电脑能提供某项服务，并不知道具体由哪个进程来提供。

所以就需要各自指定端口，就像各自守着一个门口，等到有人来了就开始干活。

而 Tomcat 就具备响应客户端的能力，所以启动 Tomcat 之后，电脑就变成了一台服务器。它监听 8080 端口，就像守在 8080 号门口，时刻等待客户端发送请求。

虽然 Tomcat 具备接收和发送请求的能力，但是它没有处理请求的能力。这么说有点不准确，客户端要访问 index.html 文件，Tomcat 的确可以把这个文件发送给客户端。但是如果客户端说：我想得到 9 + 8 的结果，这个请求倒是简单，问题是服务器没有哪个文件有包含了 17 这个结果，就算有 17，那 16 呢？不可能把所有的结果都事先准备好吧？

这个时候就要 Servlet 登场了，它可以了解到客户端的请求是什么，并且处理这个请求，计算出结果。最后由 Tomcat 再发送给客户端。

通过上面这段话，我们可以知道：**Servlet 必须依赖于 Tomcat 才能发挥自己的作用。因为它根本不具备接收和发送请求的能力，只能接收 Tomcat 的传给它的请求，也只能让 Tomcat 把最终结果返回给客户端。**

说得 Servlet 好像很厉害的样子，实际上它也是一个 Java 对象，这个对象主要是跟**请求**打交道，而且这个对象由 Tomcat 创建、管理和销毁。

通过类才可以生成对象，下面就是一个 Servlet 类的代码：

```java
import javax.servlet.ServletException;
import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "/Login", urlPatterns = {"/Login"})
public class Login extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        String username = request.getParameter("username");
        String password = request.getParameter("password");
        if(username.equals("admin") && password.equals("admin")){
            response.sendRedirect("welcome.jsp");
        } else{
            response.sendRedirect("index.jsp");
        }

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 
    }
}
```



通常把 Tomcat 看作一个容器，而 Servlet 对象就存在于这个容器里面。

我们编写好 Servlet 代码，并交给 Tomcat 处理，这些代码就会变成**对象**，并发挥自己的作用。

*（注意类和对象的概念，类这是没有生命的，对象是运行时真实存在于内存的。）*

可以把上面这个代码看成一张图纸，Tomcat 用这张图纸造出一个机器人。当请求从 8080 号门进入的时候，请求就交给机器人，机器人就会开始工作，最后的结果由 Tomcat 发送出去。

# 编写代码

说了那么多，还只是解释了 Servlet 和 Tomcat 的概念，仍然不知道代码该怎么写。

下面就来讲：

- 在 Tomcat 的解压目录的 \webapps\ROOT\WEB-INF\classes 目录下，创建一个 Java 文件（假设叫做 kk.java）。
- 在这个文件里定义一个 Servlet，（HttpServlet 类在 servlet-api.jar 包里面，而 jre 中没有这个文件，所以要把这个文件复制到 JAVA_HOME\jre\lib\ext 目录下，这样才能编译成功。）
- 使用 javac 编译这个文件之后得到 .class 文件。
- 然后修改 WEB-INF 下的 web.xml 文件。
- 最后运行 Tomcat，在浏览器访问下面的地址。

http://127.0.0.1:8080/kk?username=admin&password=admin 

```java

import java.io.*;
import javax.servlet.*;
import javax.servlet.http.*;


public class kk extends HttpServlet {
 
  private String message;

  public void init() throws ServletException
  {

      message = "Hello World";
  }

  public void doGet(HttpServletRequest request,
                    HttpServletResponse response)
            throws ServletException, IOException
  {
    String username =  request.getParameter("username");
    String password = request.getParameter("password");

    response.setContentType("text/html;charset=utf-8");
      
      
      PrintWriter out = response.getWriter();
//      out.println("<h1>" + message + "</h1> <h2>"+ username + "<br>" + password +  "</h2>");
      if(username.equals("admin") && password.equals("admin")) {
        out.println("<!DOCTYPE html>\r\n" + 
            "<html xmlns=\"http://www.w3.org/1999/xhtml\">\r\n" + 
            "<head>\r\n" + 
            "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\" />\r\n" + 
            "<title></title>\r\n" + 
            "</head>\r\n" + 
            "<body>\r\n" +
            "<h1>Login Sucess</h1>fuck " +
            "</body>\r\n" + 
            "</html>");
      } else {
        out.println("<!DOCTYPE html>\r\n" + 
            "<html xmlns=\"http://www.w3.org/1999/xhtml\">\r\n" + 
            "<head>\r\n" + 
            "<meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\" />\r\n" + 
            "<title></title>\r\n" + 
            "</head>\r\n" + 
            "<body>\r\n" + 
            "<h1>Login Default</h1>" +
            "</body>\r\n" + 
            "</html>");
      }
  }
  
  public void destroy()
  {

  }
}
```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to Tomcat
  </description>

  <servlet>
        <servlet-name>kk</servlet-name>
        <servlet-class>kk</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>kk</servlet-name>
        <url-pattern>/kk</url-pattern>
    </servlet-mapping>

</web-app>
```

虽然不知道这些文件究竟是怎么发挥作用的，但可以猜测，Tomcat 将这些文件复制到某个指定的目录，或者利用这些文件生成某些文件，最后监听端口，实现服务器的功能。

这么猜测也是有理由的，启动 Tomcat 之后再修改文件，在服务器上并没有及时的更新，而是要重启 Tomcat。

# 使用 IDE（集成开发环境）

用刚才的方法，每次修改代码，都要在命令行编译 .java 文件，而且每次都要重启 Tomcat。

而通过 IDE 可以省掉这些麻烦事，在 IDE 中开发跟平时差不多，只不过当启动 Tomcat 时，它自动地对文件进行编译，并移动到 Tomcat 指定的那个文件夹。

那么问题来了，哪个 IDE 好用？

之前一直使用 eclipse，虽然 eclipse 也有支持 Web 开发的版本，但是既然要装新的软件，为什么不装一个 IntelliJ ？毕竟经常看到有人说它好用。

IntelliJ  有收费版和社区版，但是社区版不支持 Web 开发，所以只能通过激活码的方法使用第一种。

下面记录一下创建一个简单的项目的过程：

- 首先创建一个 Project，这里的 Project 类似于 eclipse 的工作空间。
- 选择 Maven 选项。
- 创建 Module，选择 Java Enterprise，并把 Web Application 和 JSF 勾上。（我猜： 选择 Java Enterprise 之后， IntelliJ 就知道要启动 Tomcat，并且把相关文件移动到指定目录。）
- 设置 Tomcat 所在的目录。
- 点击绿色三角形启动 Tomcat。



# 总结

本文所涉及的内容非常有限，网上还有非常多的知识。

我并没有找到一个期望中的教程，要么不成体系，要么不是我想要的。

想买一本关于 Java Web 的书，又听说那些书不能跟上软件的更新。

但是网上的内容就能跟上了吗？最新的内容就是官网的内容，但是英语是硬伤。

路还很长。





























































