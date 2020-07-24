---
title: 对 Java 中流的理解
date: 2019-07-17
categories: Java
---

通过表达整理自己的思路，但不一定正确。
<!--more-->

以前学习 Java 中文件和流的时候，理解不深，当时又觉得很少用 Java 操作文件，所以就没有好好看，导致对于“文件”和“流”这两个概念都分不清。

印象中只记得有好多个类，记不得有多少个，更分不清各自有什么用，结果现在看不懂别人的代码了，所以再学习一下，争取能看懂。



# 文件和流

Java 提供了文件相关的类，和流相关的类，以前感觉都差不多。实际上前者主要是操作文件（比如删除、复制等），而不是进入文件中读取文件的数据。而后者是读写文件中的内容。

假设有一个 a.txt 文件，与文件相关的类，可以将它删除、改名。而与流相关的类，可以读取 a.txt 里面的内容，或者修改 a.txt 里面的内容。

上面是 Java 中类的区别，那么“文件”和“流”这两个名词各自是什么意思呢？

文件就是电脑里存在的文件：文本文件、图片文件、视频文件等等。

至于流，真的不好理解，网上有些说对数据的读、写是流，有的说文件就是流，以前认为流是内存和数据之间的通道。整半天没搞懂，迫于找不到通俗易懂的解释，所以根据网上的理解，自己瞎猜。

从实际应用的角度看，程序可能需要从外部读取数据，比如从硬盘中读取数据、从网络中获取数据。这些操作类似，不如将它们抽象化，用一个类表示，这也刚好符合面向对象的思想。

看到这里，我想说的是，不要单独去理解什么是流。（忘掉流吧，我偷电瓶养你）

直接从 InputStream 开始理解，它是一个抽象类，既然是类就应该可以用来表示某一类事物（就像 Person 类可以表示人一样）。所以 InputStream 就可以看成是**所有可以读取到数据的东西**。不过是硬盘，还是键盘，只要我可以从你那里读取到数据，你就是 InputStream（输入流）。

OutputStream 也是同样的道理，我往硬盘写数据的时候，硬盘就是 OutputStream（输出流）。

理解的前提是忘掉“流”，“流”只不过是类名里的最后一个字，如果设计者喜欢，叫做 OutputBlock 又有何不可呢？其次是牢记在 Java 中类是为了表示一类事物，InputStream 和 OutputStream 也不例外。



# 常见的类及其描述

## Some important Byte stream classes.

| Stream class             | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **BufferedInputStream**  | Used for Buffered Input Stream.                              |
| **BufferedOutputStream** | Used for Buffered Output Stream.                             |
| **DataInputStream**      | Contains method for reading java standard datatype           |
| **DataOutputStream**     | An output stream that contain method for writing java standard data type |
| **FileInputStream**      | Input stream that reads from a file                          |
| **FileOutputStream**     | Output stream that write to a file.                          |
| **InputStream**          | Abstract class that describe stream input.                   |
| **OutputStream**         | Abstract class that describe stream output.                  |
| **PrintStream**          | Output Stream that contain `print()` and `println()` method  |

 

## Some important Charcter stream classes.

| Stream class           | Description                                                  |
| ---------------------- | ------------------------------------------------------------ |
| **BufferedReader**     | Handles buffered input stream.                               |
| **BufferedWriter**     | Handles buffered output stream.                              |
| **FileReader**         | Input stream that reads from file.                           |
| **FileWriter**         | Output stream that writes to file.                           |
| **InputStreamReader**  | Input stream that translate byte to character                |
| **OutputStreamReader** | Output stream that translate character to byte.              |
| **PrintWriter**        | Output Stream that contain `print()` and `println()` method. |
| **Reader**             | Abstract class that define character stream input            |
| **Writer**             | Abstract class that define character stream output           |

具体的类就不研究怎么用了，估计使用得最多的就是 Bufferedxxx 了。

# 示例

下面这段代码的意思是：

1. 获取到字节输入流
2. 将字节输入流转换成字符输入流
3. 再转换成缓冲字符输入流
4. 然后在逐行读取内容

```java
InputStream inputStream =  request.getInputStream();

//BufferedReader是包装设计模式，性能更高
BufferedReader in = new BufferedReader(new InputStreamReader(inputStream,"UTF-8"));
StringBuffer sb = new StringBuffer();
String line ;
while ((line = in.readLine()) != null){
    sb.append(line);
}
```



创建一个 BufferedReader 之前还要先创建一个 InputStreamReader，这看起来很麻烦，为什么不直接创建缓冲字符输入流呢？

首先，因为要从不同的设备中读取数据，所以需要设计多个类，而每个类都可能需要用到缓冲这个功能，如果再为每个类设计一个具有缓冲功能的类，那么类的数目将会翻一倍。

所以 BufferedReader 被设计为：接收一个字符输入流，并具有缓冲的功能，虽然使用上麻烦了一点，但是还是可以接受的，不然更多的类需要去记。
