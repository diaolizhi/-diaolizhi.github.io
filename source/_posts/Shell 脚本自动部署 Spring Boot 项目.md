---
title: Shell 脚本自动部署 Spring Boot 项目
date: 2019-01-13
categories: Linux
---

同步代码 -> 打包 -> 终止进程 -> 运行项目
<!--more-->



# 准备

## Git

腾讯云 Linux 主机自带 Git，无需再安装。

但是需要执行：

```shell
ssh-keygen -t rsa
```

并将 ~/.ssh/id_rsa.pub 里的内容复制到 GitHub，这样才可以 git clone 私有项目。



## maven

maven 用来打包 Spring Boot 项目，下载方式：

```shell
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.tar.gz
```

解压：

```shell
tar -zvxf apache-xxxx
```

添加到 $PATH 变量：

```shell
vim /etc/profile
```

```
PATH="$PATH":/usr/xxxxx
```



## JDK

maven 需要 jdk 而不是 jre，否则执行 maven install 时会报错，如果装了 jdk 可以跳过这一步。

安装命令：

```shell
yum install java-1.8.0-openjdk-devel
```

安装之后使用 find 命令找到安装目录，并将其 bin 目录添加到 $PATH 变量中。



# 编写脚本

## 关键点

- 如何终止正在运行的进程
- 如何获取到最新生成的 jar 包的文件名



## 解决办法

- 定义 PRONAME （代表的是项目名）变量，然后通过 pkill 命令根据项目名终止进程。

```shell
pkill -f "java -jar $PRONAME"
```

- 通过 ls 和 tail 命令找到最新生成的 jar 包

```shell
ls -tr $PRONAME*.jar | tail -n 1
```



## 注意点

需要指定项目的**绝对路径**，因为不确定在哪个目录下执行此脚本，所以就可能导致无法切换到正确的目录，那就会出错。

另外，如果：

```shell
PROPATH='~/xxxxx/xxxxx/xxxx.sh'
```

也是无法正确的切换目录的，要把 ~ 替换成 /root



## 完整脚本

```shell
# 项目名
PRONAME="demo"
# 项目所在路径
PROPATH='/root/xxxxx/xxxxx/xxxx.sh'

nohup git pull origin master >/dev/null 2>&1

cd $PROPATH

nohup mvn install -DskipTests >/dev/null 2>&1

cd target

FILENAME=`ls -tr $PRONAME*.jar | tail -n 1`

pkill -f "java -jar $PRONAME"

nohup java -jar $FILENAME >/dev/null 2>log.txt &

```



# Java 执行 Shell 脚本

既然可以通过 Java 执行 Shell 脚本，那么就可以创建一个 Get 接口，当它被调用时就执行上面这个脚本。

```java
public class ReloadSystem {

    public static void reloadSystem() {
        ProcessBuilder processBuilder = new ProcessBuilder();

        processBuilder.command("bash", "-c", "bash ~/xxxxx/xxxxx/xxxx.sh");

        try {

            Process process = processBuilder.start();

            StringBuilder output = new StringBuilder();

            BufferedReader reader = new BufferedReader(
                    new InputStreamReader(process.getInputStream()));

            String line;
            while ((line = reader.readLine()) != null) {
                output.append(line + "\n");
            }

            int exitVal = process.waitFor();
            if (exitVal == 0) {
                System.out.println(output);
                return;
            } else {
            }
        } catch (IOException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



# 总结

1. 本地开发，提交到 GitHub
2. 访问指定接口，执行 Shell 脚本自动完成下面的工作：
   1. 使用 git pull 同步代码
   2. 使用 mvn install 打包
   3. 终止进程
   4. 使用 java -jar 运行项目



有更好的部署工具，但是现在已经可以满足我的需求了，所以没有动力去学习了。