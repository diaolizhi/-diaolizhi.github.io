---
title: Java 中的文件操作
date: 2018-4-8
categories: JavaSE
---

第十七篇关于 Java 的笔记.
<!--more-->
# java.nio.file.Path 接口

Path 引用一个文件、目录或者文件链接（快捷方式）。

创建 Path 实例的两种方法：

1. 通过 FileSystem 对象：

```java
FileSystem fileSystem = FileSystem.getDefault();
Path path = fileSystem.getPath("文件路径");
```

2. 通过 Path 类的静态方法

```java
Path path = Paths.get("文件路径")；
```



# 复制、剪切和删除操作

```java
//复制文件
Files.copy(path, toPath);
		
//剪切文件
//可选择：替代已存在的文件、复制属性、定义为原子性操作
Files.move(path, toPath, StandardCopyOption.REPLACE_EXISTING);
		
//删除文件
Files.delete(toPath);

if(Files.deleteIfExists(toPath)) {
    System.out.println("文件存在并成功删除。");
}else {
    System.out.println("文件不存在或删除失败。");
}
```



# 简单的读写操作

```java
//		读取文件的所有内容
byte[] bytes = Files.readAllBytes(path);

//		将文件当做字符串读入
String content = new String(bytes, "utf-8");
System.out.println(content);

//		写出一个字符串到文件中
//		无法追加
String mystring = "这是一个准备写入文件的字符串。";
Files.write(path, mystring.getBytes("utf-8"));

//		将文件当做行序列读入
List<String> lines = Files.readAllLines(path);
for (String string : lines) {
    System.out.println(string);
}
```



# 遍历文件夹的一种方式

这种方法不会进入子目录

```java
package com.file;

import java.nio.file.DirectoryStream;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class GetFiles {

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		Path path = Paths.get("/home/dlz/diaolizhi.github.io");
		try {
			DirectoryStream<Path> children = Files.newDirectoryStream(path);
			for(Path entry : children) {
				System.out.println(entry.getFileName());
			}
			
		} catch (Exception e) {
			// TODO: handle exception
		}
	}

}
```

# 另一种方法

walkFileTree，构造方法如下：

```java
public static Path walkFileTree(Path start, FileVisitor<? super Path> visitor) throws IOException
```

FileVisitor 是一个接口，定义了四个方法：

1. preVisitDirectory：访问**文件夹**前调用
2. postVisitDirectory：访问**文件夹**后调用
3. visitFile：访问文件时调用
4. visitFileFailed：当指定文件不可访问时调用 

JDK 中定义了 SimpleFileVisitor<> 实现了 FileVisitor 接口，我们可以继承 SimpleFileVisitor<Path> 并重写其中的方法。

```java
package com.file;

import java.io.IOException;
import java.nio.file.FileVisitResult;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.SimpleFileVisitor;
import java.nio.file.attribute.BasicFileAttributes;

/*
 * 查找路径下的所有文件
 * 2018年04月01日12:35:43
 */

public class GetFiles2 extends SimpleFileVisitor<Path>{

	public static void main(String[] args) {
		// TODO Auto-generated method stub

		Path path = Paths.get("src/");
		GetFiles2 visitor = new GetFiles2();
		try {
			Files.walkFileTree(path, visitor);
		} catch (Exception e) {
			// TODO: handle exception
		}
	}

	@Override
	public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
//		访问文件夹后的操作
		System.out.println(dir + "访问结束");
		return FileVisitResult.CONTINUE;
	}

	@Override
	public FileVisitResult preVisitDirectory(Path dir, BasicFileAttributes attrs) throws IOException {
//		访问文件夹前的操作
		System.out.println("开始访问：" + dir);
		return FileVisitResult.CONTINUE;
	}

	@Override
	public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
//		if(attrs.isRegularFile()){
//			System.out.println("Regular File:");
//		}
		System.out.println(file);
//		可以在访问文件时输出文件名字
		return FileVisitResult.CONTINUE;
	}

	@Override
	public FileVisitResult visitFileFailed(Path file, IOException exc) throws IOException {
//		访问文件失败时输出错误信息，然后继续
		System.err.println(exc.getMessage());
		return FileVisitResult.CONTINUE;
	}
}
```

# 查找文件

JDK7 中定义了一个 PathMatcher 接口，实现此接口的类可用于确定路径的匹配规范。

匹配规则有两种：一是诸如 "*.java" 之类的通配符，另一类是正则表达式。

JDK 中规定第一种匹配规则以 "glob:" 打头，第二种以 "regex:" 开头。

FileSystem.getDefault().getPathMatcher() 方法接收匹配字符串，返回一个可用的 PathMatcher 对象，程序之后就可以用它来进行文件名的匹配工作。

```java
package com.file;

import java.io.IOException;
import java.nio.file.FileSystems;
import java.nio.file.FileVisitResult;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.PathMatcher;
import java.nio.file.Paths;
import java.nio.file.SimpleFileVisitor;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.ArrayList;

/*
 * 2018年04月01日13:17:58
 * 查找指定名字的文件
 */

public class FileFinder2 {

	public static void main(String[] args) {

		Path path = Paths.get("/home/dlz");
//		MyVisitor 是自定义类，继承了 SimpleFileVisitor<Path>
		MyVisitor visitor = new MyVisitor(".*?Java.*?md");
		try {
//			visitor 的作用是：定义了了遍历文件时的操作
			Files.walkFileTree(path, visitor);
//			找到的文件（应该说是 Path 对象都放在一个集合里面）
			ArrayList<Path> results = visitor.results;
			if(results.size() > 0) {
				for(Path p : results) {
					System.out.println(p);
				}
			}else {
				System.out.println("不存在此类文件。");
			}
		} catch (Exception e) {
			// TODO: handle exception
		}
	}
}

class MyVisitor extends SimpleFileVisitor<Path> {
	
	public PathMatcher matcher;//路径匹配对象
	public ArrayList<Path> results = new ArrayList<Path>();//用来保存 Path 对象
	
	MyVisitor(String regex) {
//		通过构造方法，初始化一个路径匹配对象
		matcher =  FileSystems.getDefault().getPathMatcher("regex:" + regex);
	}
	
	@Override
	public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
		Path name = file.getFileName();
		System.out.println("检查：" + name);
		if (matcher.matches(name)) {//使用路径匹配对象的 matches 方法对比当前文件名是否合适
			System.out.println("合适");
			results.add(name);  //这里是 file 还是 name?
//			如果想保存完整路径就是 file，保存文件名就是 name
		}
		return super.visitFile(file, attrs);
	}
	
	@Override
	public FileVisitResult visitFileFailed(Path file, IOException exc)
			throws IOException {
		// TODO Auto-generated method stub
		System.err.println(file);
		return FileVisitResult.CONTINUE;
	}
}
```

总结思路：

1. 首先要遍历文件夹下面所有文件，所以必须使用 Files.walkFileTree 方法。
2. 因为要对比文件的名字，所以需要重写 visitFile 方法，也就是说要自定义一个类，继承 SimpleFileVisitor<Path> ，然后重写 visiFile 方法。
3. 对比文件名需要用到 PathMatcher，通过 FileSystems.getDefault().getPathMatcher 方法初始化一个 PathMatcher 对象。
4. 对比文件是通过 PathMatcher 对象的 matches 方法。将符合条件的 Path 对象放入集合中，以便进行其他操作。























