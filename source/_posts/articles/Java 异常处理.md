---
title: Java 异常处理
date: 2017-12-17
categories: JavaSE
---

第十三篇关于 Java 的笔记.
<!--more-->

# 一. 什么是异常

一个程序是有可能出错的, 可能是语法错误, 也可能是逻辑错误(比如访问数组时, 算错了数组下标), 还有可能是程序运行的环境造成的错误(比如内存不足)...语法错误在运行之前就可以发现, 而有些错误只是在特定的情况下才能发生. 

如果我们知道在某种情况下, 会发生错误, 那么我们就应该完善程序, 让它运行时无论遇到什么情况都能按照我们所设想的运行, 异常处理机制就是用来实现这个功能的.

话说回来, 异常到底是什么 ? 异常也是一个对象, 抛出异常就是 new 一个异常类的对象, 异常类中有一个属性 detailMessage, 这个属性记录了这个异常的信息, 说明了发生的是什么异常.

# 二. 异常的捕获语句

```java
try{
    //可能发生运行错误的代码;
}
catch(异常类型 异常对象引用){
    //用于处理异常的代码
}
finally{
    //用于"善后"的代码
}
```

**Java 中所有可捕获的异常都派生自 Exception 类**

# 三. 如何使用 Java 异常处理机制

- 将可能会出错的代码放入 try　块中
- 用 catch 捕获异常, 并对异常做出处理
- 无论发不发生异常, finally 语句都会执行

# 四. Java 中异常的分类

- Throwable 类有两个直接子类:
    + Exception: 出现的问题是可以捕获的
    + Error:     系统错误, 通常有 JVM 处理
    
- 可捕获的异常又可以分为两类:
    + check 异常: 直接派生自 Exception 的异常类, 必须被捕获或再次声明抛出
    + runtime 异常: 派生自 RuntimeException 的异常类, **不是必须**被捕获或再次声明抛出. 

# 五. AssertionError

JDK1.4 以上提供了 assert 语句, 允许程序在运行期间判断某个条件是否满足, 不满足时, 抛出 AssertionError.

```java
assert s == 7;
```

//在 Eclipse 中 assert 功能是默认关闭的, 需要手动设置

# 六. OOM Error

全称: OutOfMemoryError, 表示系统内存不足

# 七. catch 语句的特点

- 不能在多个 catch 语句块中捕获同一种异常
- 只能捕获 Exception 及其子类
- 如果第一个就捕获 Exceptioin, Java 不会编译这个程序

# 八. finally 语句

//用于处理 "资源泄漏", 看不懂

# 九. AutoCloseable 接口

//自动释放资源, 看不懂

# 十. 多层的异常捕获

- 理解语句执行的顺序.
- 一旦抛出了异常, 必须捕获, 然后再往下运行.
- 如果内层 catch 语句无法捕获, 那么就马上让外层去捕获, 这样一来, 内层接下去的语句就不会运行


# 十一. printStackTrace() 和 getMessage()

- 第一个方法是输出方法调用堆栈: 异常从哪个方法被抛出, 然后又到了哪个方法, 最后到了哪里才被处理.
- 第二个方法是获取异常的详细信息.



# 十二. 受控和不受控的异常

- 受控异常抛出之后必须捕获, 或者向上一层抛出
- 不受控异常就不必如此

# 十三. 抛出多个受控异常的方法

```java
int test() throw MyException1, MyException2{
    //...
}
```

**抛出多个异常, 但是只需要捕获一个就可以通过编译**

## 十四. 子类抛出受控异常的限制

- 不能是父类同名方法抛出异常对象的父类

# 十五. 捕获多个异常的方法

```java
catch (MyException1 | MyException2){
    //...
}
```

# 十六. 例子

输入整数分数, 判断等级

```java
package exception;

import java.util.Scanner;

public class Score {

    public static void main(String[] args) {
        System.out.println("请输入一个整数的分数: ");
        Scanner in = new Scanner(System.in);
        try {
            int score = in.nextInt();
            
            try {
                printGrage(score);
            } catch (MyException e) {
                System.out.println(e.getMessage());
//              e.printStackTrace();
            }
        }
        catch(Exception e){
            System.out.println("输入格式有误");
        }
        
        
        
    }
    
    public static void printGrage(int score) throws MyException{
        if(score>=90 && score <= 100) {
            System.out.println("成绩优秀");
        }
        else if(score>=60 && score<90) {
            System.out.println("成绩及格");
        }
        else if(score>= 0 && score<60) {
            System.out.println("成绩不及格");
        }
        else {
            throw new MyException("分数有误");
        }
    }

}       

class MyException extends Exception{
    public MyException(String Message) {
        super(Message);
    }
}
```


**简单的异常处理还可以理解, 但是复杂一点的就看不懂了. 这篇笔记只是最基础的内容.**

