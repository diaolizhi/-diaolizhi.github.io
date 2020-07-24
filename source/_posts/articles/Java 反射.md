---
title: Java 反射
date: 2018-4-23
categories: JavaSE
---

第十八篇关于 Java 的笔记.
<!--more-->

# Class 类 -- 反射的基础

## Class 类是什么

我们可以自己自定义类，JDK 中也提供了很多类，这些类有自己的名字，可能还有个爹（父类），也有自己的属性，比如：是否是接口？是不是基本数据类型等等。

从这个角度看，类跟现实世界的事物一样，有自己的特点。既然我们可以创建一个类来表示某种事物，那么是否可以设计一个类来表示“类”呢？

答案是可以的，而且已经有了这个类了，这就是 Class 类。

## 获取 Class 类实例的三种方法

Class 类是用来表示一个类的，我们已经知道可以定义一个 Student 类表示学生，然后可以调用 Student 对象的方法（比如 getName() 方法）获取到这个学生的信息。

这里也是一样的，通过调用 Class 对象的方法就可以获取到某个类的信息。比如 getName() 方法就可以获取到类的名称。

所有我们首先应该做的就是创建 Class 类型对象。

### 方法一

通过**已存在的对象**创建，此处使用的是 getClass() 方法

```java
Employee e;
....
Class cl = e.getClass();
```

*注意：Class 是泛型类，明确类型时可以写清楚：*

```java
Class<Employee>
```

不知道的情况下可以写个问号：

```java
Class<?>
```

### 方法二

通过一个字符串来创建

```java
Class cl = Class.forName("java.util.Date");
```

### 方法三

通过 类名.class 来创建

```java
Class c1 = int.class;
Class c2 = Double[].class;
```

# 反射的意义

通过上面的讲解，可以了解到 Class 类型的对象可以获取到某个类的各种信息，但实际上不仅仅可以获取信息，还可以**由 Class 类型的对象创建出某个类的实例**（通过 newInstance() 方法），也可以**执行这个类里面的静态方法或普通方法**，甚至可以**在运行的时候修改这个类的实例的成员变量**（比如通过 Class 对象，我可以修改一个 Student 对象里面的私有变量的值）。

暂时看不懂没关系，下面会用一些例子来说明。

需要**留意**的一点，通过 forName() 方法创建一个 Class 对象，比如

```java
Class<?> cl = Class.forName("反射.Student");
```

并不会创建一个 Student 对象，所以 Student 类的初始化块并不会执行。

**但是 Student 类的静态字段会被执行（用 static 修饰的代码块）。**



# Class 类部分方法介绍

- getName() 获取类名
- isInterface() 是否是接口
- isPrimitive() 是否是基本类型
- isArray() 是否为数组对象
- getSuperclass() 获取父类 Class 对象，可能得到 null
- getPackage() 获取类所在包



# Class 类的实际运用

## 判断对象所属类型

```java
if(e.getClass().getName() == "Employee")
    ...
if(e.getClass() == Employee.class)
    ...
```

不管每个类型创建了多少个实例，也不管你用什么方法获取此类型的 Class 实例，每个类型都只对应一个 Class 实例。

基本数据类型，比如 int，float 等，也有一个对应的 Class 实例，它并不等于包装类的 Class 实例，包装类另提供了一个 TYPE 字段，向外界返回它所包装的基本数据类型的 Class 实例，所以 int.class == Integer.TYPE，其值为 true。

## 运行时动态加载类并进行查询

使用 Class.forName() 方法可以加载指定名字的类型信息到内存中以便查询，所以在可以运行时获取用户输入的类名，然后加载一个类，也就是上面所说的方法二。加载类之后就可以查询其成员的方法。

*提示：指定类所拥有的属性、方法、构造函数分别由 java.lang.reflect 包中的 Field、Method 和 Constructor 三个类来表达。*

```java
/**
   @version 1.1 2004-02-21
   @author Cay Horstmann
*/

import java.util.*;
import java.lang.reflect.*;

public class ReflectionTest
{  
   public static void main(String[] args)
   {  
      // read class name from command line args or user input
      String name;
      if (args.length > 0) 
         name = args[0];
      else 
      {
         Scanner in = new Scanner(System.in);
         System.out.print("Enter class name (e.g. java.util.Date): ");
         name = in.next();
         in.close();
      }

      try
      {  
         // print class name and superclass name (if != Object)
         Class<?> cl = Class.forName(name);
         Class<?> supercl = cl.getSuperclass();
         System.out.print("class " + name);
         if (supercl != null && supercl != Object.class)
            System.out.print(" 继承自 " + supercl.getName());

         System.out.print("\n{\n");
         printConstructors(cl); //输出构造函数
         System.out.println();
         printMethods(cl);    //输出方法
         System.out.println();
         printFields(cl);    //输出字段
         System.out.println("}");
      }
      catch(ClassNotFoundException e) { e.printStackTrace(); }
      System.exit(0);
   }

   /**
      输出所有构造方法
      @param cl a class
    */
   public static void printConstructors(Class<?> cl)
   {  
      Constructor<?>[] constructors = cl.getDeclaredConstructors();

      for (Constructor<?> c : constructors)
      {           
         String name = c.getName();
         System.out.print("   " + Modifier.toString(c.getModifiers()));
         System.out.print(" " + name + "(");

         // print parameter types
         Class[] paramTypes = c.getParameterTypes();
         for (int j = 0; j < paramTypes.length; j++)
         {  
            if (j > 0) System.out.print(", ");
            System.out.print(paramTypes[j].getName());
         }
         System.out.println(");");
      }
   }

   /**
      输出所有方法
      @param cl a class
    */
   public static void printMethods(Class<?> cl)
   {  
      Method[] methods = cl.getDeclaredMethods();

      for (Method m : methods)
      {  
         Class<?> retType = m.getReturnType();
         String name = m.getName();

         // print modifiers, return type and method name
         System.out.print("   " + Modifier.toString(m.getModifiers()));
         System.out.print(" " + retType.getName() + " " + name + "(");

         // print parameter types
         Class<?>[] paramTypes = m.getParameterTypes();
         for (int j = 0; j < paramTypes.length; j++)
         {  
            if (j > 0) System.out.print(", ");
            System.out.print(paramTypes[j].getName());
         }
         System.out.println(");");
      }
   }

   /**
      输出所有字段
      @param cl a class
    */
   public static void printFields(Class<?> cl)
   {  
      Field[] fields = cl.getDeclaredFields();

      for (Field f : fields)
      {  
         Class<?> type = f.getType();
         String name = f.getName();
         System.out.print("   " + Modifier.toString(f.getModifiers()));
         System.out.println(" " + type.getName() + " " + name + ";");
      }
   }
}
```

## 获取某个类的构造方法

获取类的所有构造方法：

```java
获取类的所有构造方法：
Constructor[] constructors = Class.forName("java.lang.String").getConstructors();
获取指定类的构造方法：
Constructor constructor = String.class.getConstructors(StringBuffer.class);
```

有了类的构造方法的引用，就可以用它来创建对象。

## 动态创建对象的典型方法

一种是通过 Class 对象的 newInstance() 方法动态创建，另一种是通过构造方法对象（Constructor 对象）的 newInstance() 方法动态创建。

```java

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;
import java.util.*;

public class NewInstanceDemo {
	public static void main(String[] args) {
		invokeConstructorNoArgu();
		invokeConstructorWithArgu();
	}

	//调用无参构造函数创建对象
	private static void invokeConstructorNoArgu() {
		try {
			Class<?> c = Class.forName("java.util.ArrayList");
			List list = (List) c.newInstance(); // 直接调用Class对象的newInstance方法创建对象

			for (int i = 0; i < 5; i++) {
				list.add("element " + i);
			}

			for (Object o : list.toArray()) {
				System.out.println(o);
			}
		} catch (ClassNotFoundException e) {
			System.out.println("找不到指定的类");
		} catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		}
	}

	//调用有参构造函数创建对象
	private static void invokeConstructorWithArgu() {
		try {
            Class<?> c = Class.forName("Student");
            
            // 取得对应参数列的构造函数 ,仅适用于JDK1.5及以上版本           
            Constructor<?> constructor = 
                             c.getConstructor(String.class,int.class); 
            
            // 指定实参
            Object[] argObjs = new Object[2];
            argObjs[0] = "jxl";
            argObjs[1]=90;
            //创建对象
          
            Object obj=constructor.newInstance("jxl",90);
            // 检查结果
            System.out.println(obj);
            
        } catch (ClassNotFoundException e) {
            System.out.println("找不到类");
        } catch (SecurityException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            System.out.println("没有所指定的方法");
        } catch (IllegalArgumentException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
	}
}

//Student类中定义了一个接收两个参数的构造方法
class Student {
    private String name;
    private int score; 

    public Student() {
        name = "N/A"; 
    } 

    public Student(String name, int score) { 
        this.name = name; 
        this.score = score; 
    } 

    public void setName(String name) {
        this.name = name;
    }
    
    public void setScore(int score) {
        this.score = score;
    }

    public String getName() { 
        return name; 
    } 

    public int getScore() { 
        return score; 
    } 

    public String toString() {
        return name + ":" + score;
    }
}
```

## 动态创建数组

```java
import java.lang.reflect.Array;
 
public class NewArrayDemo {
    public static void main(String[] args) {
        Class<?> c = String.class;
        Object objArr = Array.newInstance(c, 5);
        
        for(int i = 0; i < 5; i++) {
            Array.set(objArr, i, i+"");
        }
        
        for(int i = 0; i < 5; i++) {
            System.out.print(Array.get(objArr, i) + " ");
        }
        System.out.println();

        String[] strs = (String[]) objArr;

        for(String s : strs) {
            System.out.print(s + " ");
        }
    }
}
```



## 强类型的对象工厂

*所谓对象工厂，实际上应该只是一个方法吧。利用对象工厂可以避免重复的编写 newInstance() 的代码。*

通过字符串可以创建一个 Class 对象，通过 Class 对象可以动态的创建出一个对象，但是这种方法需要强制类型转换（newInstance() 方法返回的 Object 类型对象）。

```java
package cn.edu.bit.cs.factory;
  
import java.util.*;
import javax.swing.*;

public class ObjectFactory {
	//传给它一个类名，就能得到此类的对象！
	public static Object getInstance(String clsName) {
		try {
			// 创建指定类对应的Class对象
			Class<?> cls = Class.forName(clsName);
			// 返回使用该Class对象所创建的实例
			return cls.newInstance();
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	public static void main(String[] args) {
		String className="java.util.GregorianCalendar";
		// 获取实例后需要强制类型转换
		GregorianCalendar d = (GregorianCalendar) 
				ObjectFactory.getInstance(className);
		System.out.println(d.getTime());
		// 由于没有类型信息，因此，以下这句也能通过编译，但会引发运行时异常
		JFrame f = (JFrame) ObjectFactory.getInstance(className);
		f.setVisible(true);
	}
}
```



利用泛型技术，可以不用强制类型转换，但是这样不能通过用户输入来创建 Class 实例了。**（重点在于实例化 Class 对象的时候，明确是哪个类的 Class 对象。）**

```java
package 反射;

/**
 * 2018年04月23日21:26:07
 * 利用泛型技术，实现一个强类型化的对象工厂
 * @author dlz
 *
 */

public class StrongTypeObjectFactory {

//	中括号里面的 T 是格式需要，第二个 T 是方法的返回类型
//	也不知道如果形参里面没有 T 会是怎样
	private static <T> T getInstace(Class<T> myClass) {
		try {
			return myClass.newInstance();
		} catch (Exception e) {
			System.out.println("创建对象失败");
			return null;
		}
	}
	
	public static void main(String[] args) throws ClassNotFoundException {
//		Class<?> myClass = Class.forName("反射.ClassDemo");
		
//		下面这句只能通过这种方法实现，否则无法明确 Class<T> 中的 T 是什么，那样的话仍然需要强制类型转换。
		ClassDemo cd = StrongTypeObjectFactory.getInstace(ClassDemo.class);
//		通过上面的语句就创建了 ClassDemo 的对象	
		cd.main(args);
	}

}
```

## 利用反射存取对象的字段

有一个 Field 类，可以和一个类的字符绑定起来。（绑定的时候不需要跟具体的对象关联起来。）通过 Field 对象的 getField() 和 setField() 方法可以获取、修改一个对象的值。（此时需要跟具体的对象绑定起来，这也很好理解，不绑定谁知道你要修改的是那个对象的字段？）

下面是一个简单的例子：

```java
import java.lang.reflect.Field;

class Myclass {
	public String str1;
	public String str2;
	
	Myclass() {
		str1 = "nishuonimane?";
		str2 = "nimamaicaibizhungjia";
	}
}

public class ChangeString {

	public static void main(String[] args) throws Exception {
		Myclass myclass = new Myclass();
//		下面这句话的 f 应该是指向了 Myclass 类的 str1 字段，但是没有指明是那个对象的 str1 字段
//		所以使用 get、set 方法的时候，都需要指定对象
		Field f = myclass.getClass().getField("str1");
		System.out.println(f.get(myclass).toString().replace("n", "*"));
		f.set(myclass, f.get(myclass).toString().replace("n", "*"));
		System.out.println(myclass.str1);
	}

}
```

再来一个修改私有变量的例子。

注意，Field 绑定时使用的方法有所不用，而且需要用一条语句，设置可以访问私有变量。

```java
import java.lang.reflect.Field;

class Myclass {
	private String str1;
	public String str2;
	
	Myclass() {
		str1 = "nishuonimane?";
		str2 = "nimamaicaibizhungjia";
	}
	
	public void getStr1() {
		System.out.println(str1);
	}
}

public class ChangeString {

	public static void main(String[] args) throws Exception {
		Myclass myclass = new Myclass();
//		如果 str1 是私有的，那么应该使用：getDeclaredField
		Field f = myclass.getClass().getDeclaredField("str1");
//		下面这句设置私有变量是可以访问的，否则调用 get() 方法会出错。
		f.setAccessible(true); 
		System.out.println(f.get(myclass).toString().replace("n", "*"));
		f.set(myclass, f.get(myclass).toString().replace("n", "*"));
		myclass.getStr1();
	}

}
```

## 通过反射调用类的方法

通过 Class 对象的 getMethdo() 方法获取到某个类的方法，返回的是一个 Method 对象，然后通过 Method 对象的 invoke() 方法可以调用该方法。（invoke() 方法的两个参数是：对象引用和实际参数。这里的 main() 是静态方法，所以不用指明对象，而且没有形参，所以 invoke() 没有第二个参数。）

```java
import java.lang.reflect.Method;
import java.util.Scanner;

public class ExecuMethod {

	public static void main(String[] args) throws Exception {

		Scanner scanner = new Scanner(System.in);
		Method method = Class.forName(scanner.nextLine()).getMethod("main");
		method.invoke(null);
	}

}

class bbb {
	public static void main() {
		System.out.println("我叫bbb");
	}
}

class aaa {
	public static void main() {
		System.out.println("我叫aaa");
	}
}
```

## 简单的反射框架

如果我们将类的名字，要对应的方法等放到一个外部的配置文件中，而让程序在运行时使用反射去读取它们，进而创建相应的对象并自动调用特定的方法，我们就创建了一个简单的程序框架。

[例子-GitHub](https://github.com/diaolizhi/Java-Code/tree/master/Java%20%E5%9F%BA%E7%A1%80/%E5%8F%8D%E5%B0%84/%E7%AE%80%E5%8D%95%E7%9A%84%E5%8F%8D%E5%B0%84%E6%A1%86%E6%9E%B6)











