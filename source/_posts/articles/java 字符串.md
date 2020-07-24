---
title: Java 字符串
date: 2017-12-8
categories: JavaSE
---

第六篇关于 Java 的笔记.
<!--more-->


# 1. String 类的 9 中构造方法


```java
char charArray[] = { 'b', 'i', 'r', 't', 'h', ' ',
                   'd', 'a', 'y' };
byte byteArray[] = { (byte) 'n', (byte) 'e', (byte) 'w',
                   (byte) ' ', (byte) 'y', (byte) 'e',
                   (byte) 'a', (byte) 'r' };
StringBuffer buffer;
String s, s1, s2, s3, s4, s5, s6, s7, output;

s = new String( "hello" );
buffer =  new StringBuffer( "Welcome to Java Programming!" );  //分配缓冲区

// use the String constructors
s1 = new String();    //缺省构造函数
s2 = new String( s ); //拷贝构造函数        
s3 = new String( charArray ); //以字符数组初始化字串
s4 = new String( charArray, 6, 3 );   //从字符数组第6个元素起取3个字符
s5 = new String( byteArray, 4, 4 );   //从字节数组第4个元素起取4个字节  
s6 = new String( byteArray );
s7 = new String( buffer );    //从缓冲区中提取字符创建字串,注意,是复制
```

# 2. 代码分析

```java
String s0 = "Hello";
String s1 = "Hello";
String s2 = "He" + "llo";
System.out.println(s0 == s1);  //true
System.out.println(s0 == s2);  //ture
System.out.println(new String("Hello") == new String("Hello"));  //false
```

- 在 Java 中, 内容相同的字符串常量("Hello")只保存一份以节约空间, 所以 s0, s1, s2 实际上引用的是同一个对象. ("Hello" 就是创建了一个对象, 这样理解? 2017年12月8日 16:57:34)
- 编译器在编译 s2 一句时, 会去掉 "+" 号, 直接把两个字符串连接起来得到一个字符串("Hello"). 这种优化工作由 Java 编译器自动完成.
- 当直接使用 new 关键字创建字符串对象时, 虽然值一致(都是"Hello"), 但仍然是两个独立的对象. (== 判断的是地址吧)


```java
String s1 = "a";
String s2 = s1;
System.out.println(s1 == s2);  //true

s1 += "b";
System.out.println(s1 == s2);  //false
System.out.println(s1 == "ab");  //false
System.out.println(s1.equals("ab"));  //true
```

- **String 对象的内容是只读的**, 使用 "+" 修改 s1 变量的值, 实际上 s1 原来指向的内存没有被修改, 只是指向了一块新的内存. 所以 s1 指向新的内存, 而 s2 指向原来的内存, 所以 s1==s2 返回 false.
- 代码中的 "ab" 字符串是一个常量, 它所引用的字符串与 s1 所引用的 "ab" 对象无关.(不理解 2017年12月8日 17:09:57)
- String.equals() 方法可以比较两个字符串的内容

# 3. 字符串的比较
- 使用 equals() 和 equalsIgnoreCase() 方法比较两个字符串**内容**是否相等, 使用 == 比较两个字符串变量引用同一个字符串变量.
- compareTo: 使用字典法(??)进行比较, 返回 0 表示两个字符串相等, 小于返回负值, 大于返回正值.
- regionMatches: 比较两个字符串的某一部分是否相等.

## A. String.equals() 是如何实现的

```java
public boolean equals(Object anObject) {

    //首先比较两个指针是否相等, 如果指向了同一块内存, 那肯定相等
    if (this == anObject) {
        return true;
    }

    //然后看看 anObject 是不是 String 类的实例化(这里可能说错了)
    //总不能拿一个不是字符串的对象来比较吧
    if (anObject instanceof String) {

        //把形参转换成 String, (不知道为什么要这步 2017年12月8日 17:33:45)
        String anotherString = (String)anObject;

        //字符串长度
        int n = value.length;

        //如果长度不同返回 false
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;

            //逐个比较字符
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```


# 4. 字符串查找
- 查询字符串是否以某字符串开头和结尾: startsWith() 和 endWith()
- 在字符串查找字符串或子串, 调用 indexOf() 和 lastIndexOf() 方法

# 5. 字符串的哈希定位

- String 已经内置了 Hash 码的方法

# 6. 字符串常用功能示例

- 提取子串: subString()
- 字符连接: concat()
- 将其他类型的数据转为字符串类型: valueOf()
- 获取字符串长度: length()
- 获取指定位置的字符: charAt()
- 获取从指定位置起的子串, 复制到字符数组: getchars()
- 子串替换: replace()
- 大小写转换: toUpperCase(), toLowerCase()
- 去除头尾空格: trim()
- 将字符串对象转换为字符数组: toCharArray()

# 7. 可以修改的字符串

- StringBuffer 对象创建以后, 它的内容是可以修改的.
- StringBuffer 对象其内容可以修改, 其占用的空间能自动增长
- 可以使用 "+" 和 "+=" 运算符连接字符串
- 可用方法: length, capacity, setLength, ensureCapacity, charAt, setCharAt, getChars, reverse, append, insert, delete.
- 除了 StringBuffer 之外, JDK 1.5 新增了一个 StringBuffer 类, 它的功能与 StringBuffer 一样, 只不过 StringBuffer 是线程安全的. 由于不需要加锁, 所以 StringBuilder 性能更高, 但是在多线程环境下一定要小心使用.

# 8. Character 类
- 这是字符数据类型 char 的包装类, 它提供了一些与字符处理相关的方法

# 9. StringTokenizer 类
- token 是语法分析的基本单位, 以 "定界符" (delimiter) 区分
- 使用 StringTokenizer 类可以使用指定的定界符将一个字符串为多个 token




