---
title: Java 继承
date: 2017-12-13
categories: JavaSE
---

第十一篇关于 Java 的笔记.
<!--more-->

# 一. 接口是什么?

我只能通过 interface 和 implements 分辨出接口. 

接口里面有**方法**和**域**, 这里的方法只需声明**无需实现**, 而域**必须初始化**.

# 二. 接口有什么用?

在设计类的时候, 可能有些复杂对象难以设计. 比如设计一个鸭子类, 鸭子是鸟, 会游泳, 还是一种食物. 开始的想法是: 设计一个 Food 类和 Bird 类以及 Duck 类, 让 Duck 类继承这两个类, 问题似乎 Java 中不允许多继承. 而且 "会游泳" 这个方法应该放在哪个类当中呢? 放在 Bird 中 ? 可有些鸟是不会游泳的. 放在 Duck 中 ? 可会有用的鸟也不止鸭子一种.

这种情况下, 接口就派上用场了. 定义一个接口, 接口里面实现一个 "会游泳" 的方法. 让 Duck 实现这个接口. 如果还有其他需要的方法, 都可以通过这样的方式来解决. 因为一个类是可以实现多个接口的. 

从作用来看, 接口可以让一个类**拥有特定的方法和域**. 

# 三. 接口的特性

- 接口不是类, 不能实例化一个接口
- 可以声明接口的变量, 接口变量必须引用实现了接口的类对象, 如同使用 instanceof **检查一个对象是否属于某个特定类一样**, 也可以使用 instanceof 检查一个对象是否实现某个特定的接口:

```java
if(anObject instanceof Compareable) { ... }
```

- 定义接口时, 方法无需声明为 public, 因为方法自动设为 public. 接口中的域也一样, 自动设为 public static final. 接口本身可以设为 public 或者留空.

# 四. 接口的例子:

- 编写一个类, 实例化一个对象数组, 实现 CompareTo 接口, 调用 Arrays.sort() 进行排序

```java
package my_interface;

import java.util.Arrays;

public class Interface1 {
    public static void main(String[] args) {
        Emlopyee[] emlopyees = new Emlopyee[5];
        
        emlopyees[0] = new Emlopyee("张三", 2000);
        emlopyees[1] = new Emlopyee("李四", 1000);
        emlopyees[2] = new Emlopyee("王五", 9000);
        emlopyees[3] = new Emlopyee("赵六", 4000);
        emlopyees[4] = new Emlopyee("牛七", 2000);
        
        Arrays.sort(emlopyees);
        
        for(Emlopyee e: emlopyees) {
            e.show();
        }
    }
}

class Emlopyee implements Comparable<Emlopyee>{

    private String name;
    private int salary;

    public Emlopyee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }
    
    public void show() {
        System.out.println(name + "的工资是: " + salary);
    }
    
    @Override
    public int compareTo(Emlopyee o) {
        return Double.compare(salary, o.salary);
    }
}
```

输出结果:

>李四的工资是: 1000
张三的工资是: 2000
牛七的工资是: 2000
赵六的工资是: 4000
王五的工资是: 9000
