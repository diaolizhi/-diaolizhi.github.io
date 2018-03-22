---
title: Java 内部类
date: 2018-03-19
categories: Java
---

第十五篇关于 Java 的笔记. 
<!--more-->

# 内部类的定义和特点

## 定义

简单来说， 在一个类里面定义的类就叫做内部类。

如下，在 Outer 里面又定义了一个叫做 Inner 的类，这个 Inner 就叫做内部类，而 Outer 就是外部类。

```java
class Outer {

	class Inner {
		
		void printNum() {
			System.out.println("");
		}
	}
}
```

## 特点

- 内部类可以访问外部类的所有成员
- 外部类需要创建内部类的对象才能访问（私有成员也可以访问）

例如：

```java
package com.inner;

public class InnerTest {

	public static void main(String[] args) {
		Outer o = new Outer();
		o.method();
	}

}

class Outer {
	
	private int num = 666;
	
	class Inner {
		
		private void printNum() {
            // 内部类访问外部类的私有成员
			System.out.println("The num is: "+num);
		}
	}
	
	void method() {
		Inner i = new Inner();
		i.printNum();// 在外部类创建内部类的对象，并访问它的私有成员
	}
}

```





# 何时使用内部类

现实世界里面，一个事物还包含了另一个事物，那么为它们设计类的时候，就可以使用到内部类。



# 静态内部类及其使用

## 定义

在内部类加上 static 修饰符就是*静态内部类*。如下：

```java
class Outer2 {
	
	static class Inner {
		
		void show() {
			System.out.println("");
		}
	}
}
```

## 如何直接创建静态内部类对象

1. 在内部类非静态的情况下**（不是直接创建，用来作对比而已）**

```java
package com.inner;

public class StaticInnerTest {

	public static void main(String[] args) {
		
//		创建一个内部类的对象（内部类非静态的情况下）
		Outer2.Inner i = new Outer2().new Inner();
		i.show();
	}

}

class Outer2 {
	
    // 变量和成员都是非静态的。
	private int num = 999;
	
	class Inner {
		
		void show() {
			System.out.println("The num is: " + num);
		}
	}
	
	void method() {
		Inner i = new Inner();
		i.show();
	}
}

```

2. 在内部类是静态的情况下

**静态内部类体现在可以直接创建其对象，而不是先创建一个外部类的对象。**

```java
package com.inner;

public class StaticInnerTest {

	public static void main(String[] args) {
		
//		直接创建内部类对象（内部类是静态的情况下。）
//		注意：如果内部类中访问了外部类中的成员，那么该成员也必须是静态的
//		注意：new Outer2.Inner() 中的 Outer2 后面是没有括号的
		Outer2.Inner i = new Outer2.Inner();
		i.show();

	}

}

class Outer2 {
	
    // 静态内部类要访问到的成员也是静态的
	private static int num = 999;
	
    // 内部类是静态的
	static class Inner {
		
        // 静态内部类中的方法还是非静态的
		void show() {
			System.out.println("The num is: " + num);
		}
	}
	
	void method() {
		Inner i = new Inner();
		i.show();
	}
}

```



## 静态内部类中的静态方法

静态内部类中的静态方法，可以直接访问。

```java
package com.inner;

public class StaticInnerTest {

	public static void main(String[] args) {
        
//		如果内部类是静态的，并且内部类中的方法也是静态的，那么不创建对象就可以访问了
		Outer2.Inner.show();

	}

}

class Outer2 {
	
    // num 被静态方法所访问，所以 num 也要是静态的。
	private static int num = 999;
	
	static class Inner {
		
		static void show() {
			System.out.println("The num is: " + num);
		}
	});
}

```

# 内部类访问重名变量的特点

如果在外部类中有一个叫做 num 的变量，在内部类中也有一个 num 变量，在内部类的方法里面还有一个 num 变量，那么如何访问到这些同名的变量呢？

答案是：

- 访问方法内的变量：直接访问即可
- 访问内部类中的变量：this.变量名
- 访问外部类中的变量：外部类名.this.变量名

例子：

```java
package com.inner;

public class InnerTest2 {
	
	public static void main(String[] args) {
		
		new Outer3().method();
		
	}

}

class Outer3 {
	
	private int num = 888;
	
	class Inner {
		
		private int num = 88;
		
		void show() {
			int num = 8;
			System.out.println("The num is: " + num);// 调用的是 show 方法中的 num
			System.out.println("The num is: " + this.num);// 调用的是 Inner 类中的 num
			System.out.println("The num is: " + Outer3.this.num);// 调用的是 Outer3 类中的 num
		}
	}
	
	void method() {
		new Inner().show();
	}
}
```

顺便一提，内部类之所以可以访问外部类，是因为内部类持有外部类的引用。



# 局部内部类

## 定义

局部内部类就是：在类的方法中定义的类。

例如，在 Outer4 的 method 方法中的 Inner 类就是局部内部类。

```java
class Outer4 {
	
	void method() {
		class Inner {
			
			void show() {
				System.out.println("");
			}
		}
	}
}
```

## 特点

在局部内部类中，只能访问外部类中用 final 修饰的变量。

```java
class Outer4 {
    
	void method() {
		
        // 局部内部类中使用的变量要用 final 修饰
		final int num = 6;
		
		class Inner {
			
			void show() {
                // 这里使用了 method 方法中的变量
				System.out.println(num);
			}
		}
	}
}
```



原因是： 如果 method 方法中定义了一个 Inner 内部类，那么这个类可以将这个内部类的对象返回（可以用 Object 接收）。问题是，将这个对象返回了，可是 method 方法也出栈了，method 的局部变量都释放了，Inner 类中使用的变量不存在，就会出错了。如果使用的变量被 final 修饰了，比如 

```java
final int num = 6;
```

那么出现 num 的地方，实际上使用的是 6 。这样就不存在使用的变量被释放的情况了。

## Java8 的特性

下面的代码是可以运行的。

```java
class Outer4 {
    
	void method() {
		
        // 局部内部类中使用到了，但是并没有使用 final 修饰
		int num = 6;
		
		class Inner {
			
			void show() {
				System.out.println(num);
			}
		}
	}
}
```

可以看成在 Java8 中，默认给我们加了 final 。前提是这个变量没有被修改。

如果这个变量被修改了，那么就不会默认地加上 final。

```java
class Outer4 {
    
	void method() {
		
        // 局部内部类中使用到了，但是不用加 final
		int num = 6;
		
		class Inner {
			
			void show() {
                // 这里使用了 method 方法中的变量
				System.out.println(num);
			}
		}
        // 如果加了下面这句，就无法通过编译了
        // num = 7;
	}
}
```

从编译器的角度看，你的方法中有个变量，你只是在局部内部类中访问了一下，那么我就帮你个忙，替你把 final 几个字加上去，反正这个变量不需要更改。但是，如果这个变量你还要改动的，那我就不能给你加上 finnal 了，加了你就不能改了，我只能提醒你，你的代码有问题了。

# 匿名内部类

## 定义

匿名内部类它也是内部类，所以它还是在一个类里面定义的。

其次，它是匿名的，也就是没有名字的。

它没有名字，那怎么创建这个类的对象呢？只能用它爹的名字了。

所以，匿名内部类的一个要求就是，必须继承某个类，或者实现某个接口。（别忘了，任何一个类都是继承自 Object 类的。）

## 使用

1. 通过它爹的名字，来创建一个匿名内部类。这个类与它爹不同，因为这个类可以覆盖它爹的方法，或者定义新的方法。创建一个匿名内部类之后，就可以直接调用里面的方法了。不过这样只能调用一个方法，要调用另一个的话，只能再创建另一个匿名内部类了，所以，可以采用第二种方式。

```java
class Outer5 {
	
	void method() {
		
		new Demo() {
			void show() {
				System.out.println("第一个内部匿名类。");
			}
		}.show();
		
	}
	
}

abstract class Demo {
	
	abstract void show();
}
```

2. 创建了一个匿名内部类之后，将这个对象的保存起来。这样就可以，创建一次对象，调用多次方法。

```java
class Outer5 {
	
	void method() {
		
		Demo d = new Demo() {
			void show() {
				System.out.println("第一个内部匿名类。");
			}
            void show2() {
                System.out.println("匿名内部类的第二个方法");
            }
		};
		
		d.show();
        d.show2(); // 创建一次对象，多次调用对象的方法。
		
	}
	
}

abstract class Demo {
	
	abstract void show();
    abstract void show2();
}
```

**需要注意的是：通过父类引用，只能调用父类中声明的方法。**

























