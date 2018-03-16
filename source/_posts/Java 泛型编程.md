---
title: Java 泛型编程
date: 2018-03-16
categories: JavaSE
---

第十四篇关于 Java 的笔记.
<!--more-->

# 一. Java “泛型” 相关术语

- 以 ArrayList&lt;E&gt;、ArrayList&lt;Integer&gt; 为例
  1. ArrayList&lt;E&gt; 定义了一个泛型类型，“E"称为”类型变量“或”参数类型“。
  2. ArrayList&lt;Integer&gt; 称为“参数化的类型”，“Integer”称为“实际类型参数”。
  3. ArrayList 称为泛型类型 ArrayList&lt;E&gt; 的“原始类型（raw type)”。

# 二. 泛型类

- 我们定义了泛型类 Pair&lt;&gt;，但是在使用的时候，必须给 T 指定一个具体的类型（比如：String）：Pair&lt;String&gt;。（实际上在有些简单的程序中，不加尖括号也不会出错。就像集合一样，可以不加，但是会有警告，而且可能在运行时会出错。出错的原因可能是：传入了不恰当类型的参数。）
- “Pair&lt;String&gt;” 就是 “Pair&lt;T&gt;”的实例化（instantiate)。



# 三. 泛型使用”须知“

1. 泛型类型必须是引用类型，不能是基本数据类型。（也就是说我要实例化一个 Pair&lt;T&gt;的对象，不能写 Pair&lt;double&gt;，只能 Pair&lt;Double&gt;。

2. 参数化类型可以应用一个 ”原始类型对象（raw type）”的对象（反过来也可以），但是编译时会产生一个警告。

   ```java
   ArrayList<String> strs = new ArrayList();
   ArrayList strs2 = new ArrayList<String>();
   ```

3. 不能定义泛型化数组

   ```java
   Pair<String>[] table = new Pair<String>[10];//error
   ```

4. 不能直接创建泛型类型的实例

   ```java
   public class Pair<T> {
       ...
           public Pair() {
           first = new T();//error
           second = new T();//error
       }
   }
   ```

5. 泛型类型不能直接或间接继承自 Throwable。

   ```java
   public class Problem<T> extends Exception //error
   ```

   所以也无法抛出或捕获泛型类型的异常对象

   ```java
   catch (T e) {...} //error
   ```

6. 不能定义静态泛型成员，以下代码无法通过编译

   ```java
   class MyClass<T> {
       public static T value;
       public static T f() { }
   }
   ```



# 四. 从泛型类派生子类的要求

1. 不允许基类中有泛型参数

   ```java
   class MyChild extends MyClass<T> { } //error
   ```

   ```java
   class MyChild extends MyClass<String> { } //OK
   ```

2. 如果 MyClass 是泛型类，在定义子类时不指定泛型参数，则 MyClass 的泛型参数默认为 Object。

   ```java
   class MyChild extends MyClass { }
   ```

   ​


# 五. 泛型方法

下面第一个属于泛型方法，剩下两个是为了观察：传入不同类型时，到底会调用哪个方法？得到的结果是：调用最合适的方法。已知编译器的擦除特点之后， 可以直接把第一个方法的参数类型看成是：Object。

```java
class ArrayAlg {
	
    // 泛型方法
	public static <T> T getMiddle(T[] a) {
		return a[a.length / 2];
	}
	
	public static double getMiddle(double[] d) {
		return 66.66;
	}
	
	// 有了这个方法，传入 Integer 数组的时候，就会调用这个方法
	// 在不存在的时候，可能是寻找 Integer 的基类，所以找到了泛型方法
	public static Integer getMiddle(Integer[] i) {
		 return 66;
	}
}
```



# 六. 类型变量的限定

- 下面的代码中如果没有 &lt;T extends Comparable&gt; ，那么 min.compareTo() 就会报错，因为无法保证传入的类型实现了 Comparable 接口。

- 虽说 Comparable 是接口不是类，但还是使用了 extends 关键字，这是 Java 设计者规定的。

- 一个类型变量可以有多个限定：

  ```java
  T extends Comparable & Serializable
  ```

- 限定中可以有多个接口，但只能有一个类。如果采用一个类作为限定，那么它必须是限定表中的第一位。

```java
class MyClass {
	
	public static <T extends Comparable> T minmax(T[] a) { // 加了限定为什么会出现警告？
		
		if (a == null || a.length == 0) {
			return null;
		}
	
		T min = a[0];
		T max = a[0];
		
		for (int i = 1; i < a.length; i++) {
			if (min.compareTo(a[i]) > 0) min = a[i];
			if (max.compareTo(a[i]) < 0) max = a[i];
		}
		
		return max;
	}
}
```



# 七. 泛型类型的继承规则

假设 Employee 是 Manager 的父类，但是 Pair&lt;Employee&gt; 和 Pair&lt;Manager&gt; 是没有关系的。（它们两个顶多算是兄弟，不符合继承关系。）



# 八. 类型通配符

- 在泛型限制中使用 ? 可以匹配任意类型。

- 要求 ? 必须是 T 的子类

  ```java
  List< ? extends T>
  ```

- 要求 ? 必须是 T 的父类

  ```java
  Collection< ? super T>
  ```

  ​

## 引入 ”?“ 的好处

- 考虑一下泛型方法，它以 MyClass&lt;Parent&gt; 作为参数

  ```java
  static void doSomething(MyClass <Parent> obj) { }
  ```

- Parent 类有一个子类 Child，我们可以这样调用它：

  ```java
  doSomething(new MyClass<Parent>());
  ```

- 却不能这样调用它（这就像”泛型类型的继承规则“说的那样，它们没有继承关系）：

  ```java
  doSomething(new MyClass<Child>());
  ```

- 将泛型声明改为下面这样就行了（可以理解成：Myclass&lt; ? extends Parent&gt; 是 MyClass<Parent&gt; 和 MyClass<Child&gt; 的基类。

  ```java
  static void doSomething(Myclass< ? extends Parent> obj) { }
  ```



# 九. 擦除与恢复

- 类型擦除

  - 以前版本的类解释器不知道什么是泛型，所以编译器为了让类解释器可以运行包含泛型的程序，会将类型变量（T）擦除掉，并替换为限定类型（无限定的变量用 Object）。

- 翻译泛型表达式

  - 类型擦除之后，方法的返回类型有可能是 Object，所以在必要时编译器会插入强制类型转换。

- 翻译泛型方法

  - > [java泛型——桥方法](http://blog.csdn.net/pacosonswjtu/article/details/50374131)

  - 泛型方法的问题主要是在继承的时候：子类无法覆盖泛型父类中的方法。为了解决这个问题就有了桥方法，桥方法由编译器自动生成，间接的调用重新定义的方法。

  - 问题 2：如果没有形参，那么桥方法和重新定义的方法只有返回值不同，看似不能共存，但是桥方法是编译器生成的，所以它有能力分辨，不用我们操心。


**对泛型的理解还远远不够，书上很多东西都还看不懂，还需要长时间的学习才能掌握。**

