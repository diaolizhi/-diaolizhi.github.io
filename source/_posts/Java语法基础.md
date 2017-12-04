---
title: Java 语法基础
date: 2017-12-4
categories: JavaSE
---

第一篇关于 Java 的笔记.
<!--more-->

# 1. 标识符: 
- 开头: 字母, _ 和 $
- 不能以数字开头, 不能包含空白字符: Tab, 空格, 回车, 换行
- 不能使用关键字;

# 2. 标识符的规范
- 类名首字母大写
- 方法名首字母小写
- 常量全大写.

# 3. Java 的常用数据类型: 
- 整型 byte, short, int, long
- 浮点型 float, double(123.456 = 1.23456e+2)
- 布尔型 true, false
- 字符型 'a', 'A'
- 字符串: "Hello, China";

# 4. Java 7 新特性
- 允许使用下划线分隔多个数位. 使用当前区域语言特性格式化数字: 
```java
int number1 = 1_000_000;
NumberFormat format = NumberFormat.getInstance(); 
System.out.println(format.format(number1));
```

# 5. final 关键字
- final 用于定义常量. 对于在整个项目中使用的常量, 通常按以下模式声明: 
```java
public static - final int MAX_VALUE = 512; //常量名通常采用大写.
```

# 6. 原始数据类型
- Java 中除了 int, float 等少数几个数据类型, 其余的数据类型都用来引用数据. int float 等这些数据类型称为"原始数据类型(primitive type)";

# 7. 枚举类型
```java
//定义
enum Size{SMALL, MEDIUM, LARGE};
//使用
Size s = Size.SMALL;
//从字符串转换为枚举
Size t = Size.valueof("SMALL");
```
*JDK 5 及以上才可用枚举类型.*
```java
//枚举值的 foreach 迭代
private enum MyEnum{
    ONE, TWO, THREE
}
public static void main(String[] args){
    for(MyEnum value: MyEnum.values()){
        System.out.prinln(value);
    }
}
//成员枚举MyEnum只能在顶级类或接口或静态上下文中定义???
```
*枚举可用于 switch 语句*

- 枚举类型是引用类型. 枚举不属于原始数据类型, 它的每个具体值都引用一个特定的对象. 相同的值则引用一个对象. 可以用 "==" 和 equals() 方法直接对比枚举类型的值, 换句话说, 对于枚举类型的变量, "==" 和 equals() 方法执行的结果是等价的. (2017年12月4日 19:20:38, 看不懂)

# 8. 运算符结合性
- 除了赋值运算符 =, 所有的结合性都是从左到右, 例如: x = y = z 相当于 x = (y = z); 像 x = y =z 这样可能引起误解的代码应该尽量避免, 至少要加上括号, 或者添加注释.

# 9. Java 中的变量
变量可看成某内存单元(区域)的名字. 每个变量都有:
- 名字(对应于内存中的位置)
- 数据类型(决定了它所占用的内存单元数量)
- 值(表示变量所占用的内存单元中所保存的数据.)
- 变量的读写: 当新值被赋给变量时, 老值将被取代, 仅从内存中读数据不会破坏数据.
- (这个变量说的是那块内存还是那个名字? 2017年12月4日 20:46:42)

# 10. 变量的使用准则
- 在实际开发中,  一般使用变量来存储用户在程序运行时输入的数据.
- **变量在使用之前应 保证它有确切的值.**
-  Java 变量遵循 **"同名变量的屏蔽原则"**.

# 11. 在运行时读取用户输入
- 方法一

```java
String firstNumber = JOptionPane.showInputDialog("Enter: ");
```

- 方法二

```java
Scanner in = new Scanners(System.in);
System.out.print("What is your name? ");
String name = in.nextLine();
```

# 12. 变量间的类型转换
- 自动类型转换是安全的(不精确的->更精确的, 不用强制类型转换)

```java
int intValue = 100;
long longValue = intValue;
```

- 强制类型转换可能会引起信息的损失

```java
double doubleValue = 1234567890;
float floatValue = (float)doubleValue;
System.out.println(floatValue);  //1.234567894E9
```

# 13. 另一种数据类型转换的方法
- 通过原始类型的包装类完成类型转换

```java
double doubleValue = 156.5d;
Double doubleObj = new Double(doubleValue);
byte myByteValue = doubleObj.byteValue();
int myIntValue = doubleObj.intValue();
float myFloatValue = doubleObj.floatValue();
String myString = doubleObj.toString();
```
# 14. 精度损失问题

- double 类型

```java
    System.out.println("0.05 + 0.01 = " + (0.05 + 0.01));
    System.out.println("1.0 - 0.42 = " + (1.0 - 0.42));
    System.out.println("4.015 * 100 = " + (4.015 * 100));
    System.out.println("123.3 / 1000 = " + (123.3 / 100));
```

- 使用 BigDecimal 解决该问题

```java
BigDecimal f1 = new BigDecimal("0.05");
BigDecimal f2 = BigDecimal.valueOf(0.01);
BigDecimal f3 = new BigDecimal(0.05);
System.out.println("下面使用String作为BigDecimal构造器参数的计算结果：");
System.out.println("0.05 + 0.01 = " + f1.add(f2));
System.out.println("0.05 - 0.01 = " + f1.subtract(f2));
System.out.println("0.05 * 0.01 = " + f1.multiply(f2));
System.out.println("0.05 / 0.01 = " + f1.divide(f2));
```
(在构建 BigDecimal 对象时应该使用字符串而不是 double 数值, 否则, 仍可能引发计算精度问题. (不知为何 2017年12月4日 21:43:20))

# 15. 有关字符串

## A. 字符串转为数字
 
```java
int number = Integer.parseInt(numberString);
```

类 Integer 属于包 java.lang, 它 "封装" 了一个 int 类型的整数, 因此, 它是原始数据类型 int 的 "包装类". (这个封装是什么意思? 2017年12月4日 21:46:21)

## B. 字符串转为浮点数
```java
number1 = Double.parseDouble(firstNumber);
number2 = Double.parseDouble(secondNumber);
```

Double.parseDouble 是一个 Double 类所定义的静态方法, 作用是将 String 数据转为 double 类型的, 返回值是 double 类型的数值. Double 是原始数据类型 double 的 "包装类", 属于引用类型.(这句话看不懂, 包装类是什么???2017年12月4日 22:03:19)

## C. 字符串联接操作
使用运算符 + 联接字符串. **将 String 和其他数据类型相加, 结果是一个新的 String.**

# 16. 两种类型的变量
- 引用类型的变量, 引用一个对象(变量本身用于存放对象在内存中的位置, 可以看成是一个指针), 故又被称为 "对象变量".
- 原始数据类型的变量.
- 两者的区分: 如果变量的数据类型是一个类的名字, 就是引用类型的变量, 它将引用一个对象. 如果变量的数据类型是一个原始类型(其名称由小写字母组成), 这种类型的变量将直接保存一个原始数据类型的值.