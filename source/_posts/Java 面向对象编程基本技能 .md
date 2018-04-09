---
title: Java 面向对象编程基本技能
date: 2018-03-22
categories: JavaSE
---

第十六篇关于 Java 的笔记. 
<!--more-->

主要内容：

- 对象比较（使用 equals 方法比较两个对象内容是否相同，使用 compareTo 方法比较两个对象的大小。）
- 对象组合（描述对象之间的关系：包含或者引用。）
- 对象复制（把一个对象的内容全部复制到另一个对象里面。）

# 对象比较

## Comparable 接口

 Comparable 接口里面定义了一个 compareTo 方法，这个方法接收一个对象。

Comparable 接口定义如下：

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```



通常一个类实现 Comparable 接口，然后在 CompareTo 方法中对两个对象（一个是形参，一个是 this）进行比较。如果 this 比较小，则返回 -1，相等则返回 0，否则返回 1 。



下面定义了一个学生类，它实现了 Comparable 接口，在 compareTo 方法中，通过分数比较两个对象的大小。

```java
class Student implements Comparable<Student>{  // 需要指明泛型接口的类型变量，不能使用 T
	
	private String name;
	private int score;
	
	Student(String name, int score) {	
		this.name = name;
		this.score = score;
	}
	
	public int compareTo(Student o) {  // 不理解为什么，这里的参数类型必须是 Student
		if(this.score == o.score){
			return 0;
		}else if(this.score < o.score){
			return -1;
		}else {
			return 1;
		}
	}

	public String getName() {
		return name;
	}
	
	public int getScore() {
		return score;
	}
}
```



实现了 Comparable 接口，并且写好了 compareTo 方法，所以两个学生对象之间就可以比较大小了。

我们可以利用 compareTo 方法，自定义方法来比较两个对象的大小，也可以利用 JDK 自带的方法对学生数组进行排序。如下：

```java
public class StudentSort {

	public static void main(String[] args) {
		
		Student[] students2 = new Student[3];  // 中括号内指定数组元素的个数
		students2[0] = new Student("张三", 80);
		students2[1] = new Student("李四", 60);
		students2[2] = new Student("王五", 70);
		Arrays.sort(students2);
		for (int i = 0; i < students2.length; i++) {
			System.out.println(students2[i].getName() + students2[i].getScore());
		}
	}

}
```

## equals 方法

- 如果使用 ”==“ 比较两个对象，那么比较的是这两个对象变量是否引用同一个对象。
- 如果需要比较对象的内容，也就是各字段的值，通常是调用对象的 equals 方法。
- equals 方法由 Object 类所定义，默认情况下还是比较两个对象变量是否引用同一个对象。所以通常需要重写 equals 方法。

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

- 重写 Object 的 equals 方法，注意其参数类型必须是 ”Object“。
- 重写 Object 的模板：

```java
//	重写 equals 的模板
	public boolean equals(Object obj) {
		if(obj == null) {
			return false;
		}
		if(this == obj) {
			return true;
		}
		if(obj instanceof xixi) {
			xixi xi = (xixi)obj;
			return this.value == xi.value;
		}else {
			return false;
		}
	}
}
```

## 自定义比较规则

之前说过，实现了 Comparable 接口，就可以调用 Arrays.sort() 进行排序，它默认从小到大进行排序。

假设对学生排序，而学生类的 compareTo 方法是根据分数返回 0、-1 或者 1 。那么默认是根据分数从小到达进行排序。

如果想根据学生的姓名进行排序该怎么办呢？方法是自定义比较规则。

- 自定义一个类，实现 Comparator 接口
- 重写 compare 方法（接收两个对象，返回 0、-1 或 1 ，表示 s1 和 s2 比较的结果）
- 在调用 Arrays.sort() 的时候，将自定义比较规则传入

```java
class NameComparator implements Comparator<Student> {
	
	public int compare(Student s1, Student s2) {
		return s1.getName().compareTo(s2.getName());
	}
}
```

```java
Arrays.sort(students2, new NameComparator());
```

这样一来，学生对象的数组就已经根据名字来排序了。



# 对象组合

一个对象包容另一个对象，称为“对象组合”。

有两种组合方式：

- A 对象完全包容 B 对象，容器对象管理被包容对象的生命期。（B 是 A 的内部类）
- B 对象是独立的，A 对象引用现成的 B 对象。（A 对象里面有一个 B 的引用而已）



# 对象复制

- 浅克隆，只是复制每个字段的值

```java
public class CloneATest {

	public static void main(String[] args) {
		A a = new A();
		a.change();
		A b = CloneA(a);
		System.out.println(a.a + "   " + b.a);
	}
	
//	简单复制对象，只是复制里面的值。
	public static A CloneA(A other) {
		A a = new A();
		a.a = other.a;
		return a;
	}

}

class A {
	public int a = 100;
	
	public void change() {
		a = 66;
	}
}
```

- 如果类中 b 字段引用一个对象，那么浅克隆的方式可能会导致，两个对象的 b 字段指向同一个对象，这显然不是真正的复制，我们需要的是两个对象的 b 各自指向一个对象，同时被指向的对象的内容完全一样。而这就是深克隆。

```java
public class DeepCopy {

	public static void main(String[] args) {
		
		A3 a = new A3();
		A3 b = CloneA3(a);
		System.out.println(b.b.val);
	}

	private static A3 CloneA3(A3 other) {
		A3 a = new A3();
		a.num = other.num;
		a.b = new B();
		a.b.val = other.b.val;
		return a;
	}
}

class A3 {
	public int num = 100;
	public B b = new B();
}

class B {
	public int val = 100;
}

```

- JDK 中提供了一个 Cloneable 接口，需要深克隆的对象可以实现这一接口。（注意强制类型转换和捕获异常）

```java
public class DeepCopy2 {

	public static void main(String[] args) throws CloneNotSupportedException {
		// TODO Auto-generated method stub

		MyCopy myCopy = new MyCopy();
		myCopy.change();
		MyCopy myCopy2 = (MyCopy) myCopy.clone();
		System.out.println(myCopy2.a6.val);
	}

}

class MyCopy implements Cloneable {
	
	private int num;
	public A6 a6;
	
	MyCopy() {
		a6 = new A6();
	}
	
	public Object clone() throws CloneNotSupportedException {
		MyCopy mycopy = new MyCopy();
		mycopy.num = this.num;
		mycopy.a6 = new A6();
		mycopy.a6.val = this.a6.val;
		return mycopy;
	}
	
	public void change() {
		a6.val = 999;
	}
}

class A6 {
	public int val = 100;
}
```





















