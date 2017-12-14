---
title: Java 多态
date: 2017-12-14
categories: JavaSE
---

第十二篇关于 Java 的笔记.
<!--more-->

# 一. 什么是多态

从用法上看: 

- 当基类变量引用子类对象时, 通过这个基类变量调用本类中与子类的方法, 实际上调用的就是子类中的方法
- 当接口变量引用实现了这个接口的类的对象时, 调用的方法是类中的方法

或者说:

- 调用同一个方法, 有不同的表现. 前提是基类(接口)对象引用子类对象.

# 二. 为什么要有多态

假设有一个网上购物的程序, 有一个操作 (total) 是把购物车里面的价格加起来. 我们都知道商品的种类是非常多的, 怎么定义这个 total 方法的参数, 使得传入任意商品都可以实现算出价格呢 ? 我们总不能通过函数重载把所有情况都列一遍吧 ? 

这个时候就可以用到 "多态" 了, 定义一个商品类(goods), 让所有商品都继承自这个类, 这个类定义一个 getPrice 方法, 返回商品的价格. 然后在定义 total 时, 只需要传入一个 goods 数组, 调用 goods[i].getPrice 就可以获得每个商品的价格.

# 三. 多态的语法特点

**当满足多态的条件时:**

- 如果基类和子类中有同名方法, 那么调用的是子类中的方法
- 如果基类和子类中有**同名字段**时, 调用的是基类中的**字段**. (**建议不要定义同名字段给自己自找麻烦**)
- 可以通过关键字: super 调用基类中的方法, 以及访问基类中的字段. (**子类方法中: super.方法名 可以调用父类中的方法, 如果那个方法中访问了类中的字段, 那么可能访问的是子类中的字段, 也可能访问父类类中的字段(如果父类和子类有同名字段)**)

```java
package polymorphism;

public class ParentChildTest {
    public static void main(String[] args) {
        Parent parent=new Parent();
        parent.printValue();  //输出父类的
        Child child=new Child();
        child.printValue();  //输出子类的
        
        Parent parent2 = new Child();
        System.out.println(parent2.myValue);
        parent2.printValue();
        
//      parent=child;
//      parent.printValue();//输出子类的
//      
//      parent.myValue++;
//      parent.printValue();//还是子类的
//      
//      ((Child)parent).myValue++;//还是子类的吧
//      parent.printValue();
        
    }
}

class Parent{
    public int myValue=100;
    public void printValue() {
        System.out.println("Parent.printValue(),myValue="+myValue);
    }
}
class Child extends Parent{
    public int myValue=200;
//  {
//      myValue = 888;
//  }
    public void printValue() {
        super.printValue();
        System.out.println("Child.printValue(),myValue="+myValue);
    }
}
```

# 四. 子类和父类变量之间的转换

- 子类对象可以直接赋给基类变量
- 基类对象要赋给子类变量, 需要强制类型转换.
- 通过 instanceof 判断一个对象是否可以转换为某个变量 (好像自定义的不行 ???)

# 五. 动物园的例子

```java
package polymorphism;

public class Zoo {

    public static void main(String[] args) {
        
        Feeder feeder = new Feeder("小王");
        Animal[] animals = new Animal[10];
        
        for(int i=0; i<5; i++) {
            animals[i] = new Lion();
        }
        for(int i=5; i<10; i++) {
            animals[i] = new Monkey();
        }
        
        feeder.feed(animals); 
    }

}

abstract class Animal{
    public abstract void eat();
}

class Lion extends Animal{
    public void eat() {
        System.out.println("狮子在吃肉");
    }
}

class SmallLion extends Lion{
    public void eat() {
        System.out.println("小狮子在吃肉");
    }
}

class Monkey extends Animal{
    public void eat() {
        System.out.println("猴子在吃桃子");
    }
}

class Feeder{
    private String name;
    
    public Feeder(String name) {
        this.name = name;
    }
    
    public void feed(Animal[] animals) {
        for(int i=0; i<animals.length; i++) {
            System.out.printf(name + "正在喂动物: ");
            animals[i].eat();
        }
    }
}
```

输出结果:
>小王正在喂动物: 狮子在吃肉
小王正在喂动物: 狮子在吃肉
小王正在喂动物: 狮子在吃肉
小王正在喂动物: 狮子在吃肉
小王正在喂动物: 狮子在吃肉
小王正在喂动物: 猴子在吃桃子
小王正在喂动物: 猴子在吃桃子
小王正在喂动物: 猴子在吃桃子
小王正在喂动物: 猴子在吃桃子
小王正在喂动物: 猴子在吃桃子
