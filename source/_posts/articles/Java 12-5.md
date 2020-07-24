---
title: Java 中的方法
date: 2017-12-5
categories: JavaSE
---

第二篇关于 Java 的笔记.
<!--more-->

# 1. 静态导入
从 JDK 5.0 开始, 支持一种称为 "静态导入" 的方法

```java
import static java.lang.Math.*;
```

从而允许在 Java 代码中省略类名只写静态方法名:

```java
System.out.println(abs(-100));
```

上述方法完全等价于:

```java
System.out.println(Math.abs(-100));
```

(一般的导入方法是导入类, 而就算导入了 Math 类, 想要使用它里面的静态方法还是使用 **类名+方法名** 的方法. 而通过静态导入, 就可以只写静态方法名.)

# 2. Java 生成真正的随机数
//不会

# 3. 参数可变的方法

```java
public class VariableArguments{

    public static double max(double...values){
        double largest = Double.MIN_VALUE;
        for(double v: values){
            if(v > largest){
                largest = v;
            }
        }
        return largest;
    }

    public static void main(String args[]){

        System.out.println("Max: " + max(1, 11, 300, 2, 3));
        System.out.println("Max: " + max(1, 11, 300, 2, 3, 800));
    }
}
```

## A. 特点
- 只能出现在方法参数表列的最后
- "..." 位于变量类型和变量名之间, 前后有无空格均可.
- 调用可变参数方法时, 编译器为该可变参数隐含创建一个数组u在方法体以数组的形式访问可变参数.


# 4. 方法重载( overload )
## A. 要点
要满足以下条件:
- 方法名相同
- 参数类型不同, 参数个数不同, 或者是参数类型的顺序不同
- 注意: **方法的返回值不作为方法重载的判断条件**

# 5. 递归
## A. 递归的模式
- 每个递归函数的开头一定是判断递归是否结束的语句(一般是 if 语句)
- 函数体至少有一句是 "自己调用自己" 的.
- 每个递归函数一定有一个控制递归可以终结的变量(通常作为函数的参数而存在). 每次调用自己时, 此变量会变化, 并传给被调用的函数.

# 6. 处理大数字
## A. 数字的范围
- Java 中 int 类型的数值占 32 位, 是有符号的, 第一位是 0 代表正数, 第一位是 1 代表负数. 剩下 31 位, 最小是 0, 最大是 2,147,483,647. 所以 int 类型的范围是 -2,147,483,647 ~ 2,147,483,647.
- 同理, long 类型的数值占 64 位, 能表示的范围是 2 的 63 次方减 1;
- 计算机使用固定的位数来保存数值, 因此, 能处理的数值大小是有限的, 当超过一定范围, 数值就会被截断, 这将导致错误的产生.

## B. 解决方法

使用 BigInteger.
eg: 求 50! 

```java
import java.math.BigInteger;
import java.util.Scanner;

public class Recursive {

    public static void main(String[] args) {

        Scanner in = new Scanner(System.in);
        int n;
        BigInteger sum;
        
        System.out.println("请输入 N: ");
        
        n = in.nextInt();
        sum = calculateN(n);
        
        System.out.println(sum);
        
        in.close();
        
    }
    
    public static BigInteger calculateN(int n){
        if(n==1 || n==0){
            return BigInteger.valueOf(1);
        }
        return BigInteger.valueOf(n).multiply(calculateN(n-1));
    }

}
```

# 7. 浮点数
下面的代码会出错

```java
double i = 0.0001;
double j = 0.000100000000000000000001;
System.out.println(i == j);  //输出 true
```

- 原因: 计算机不能精确地表达浮点数(特殊形式的除外), 因此, 当需要比较两个浮点数是否相等时, 应该比较其差的绝对值是否在某个允许范围之内即可.

```java
if(Math.Abs(i - j) < 1e-10){
    System.out.println("true");
}
else{
    System.out.println("false");
}
```
(还是输出 true ? 是我写错还是在这个范围内算是相等的.)
