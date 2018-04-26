---
title: Java JDBC 基础
date: 2018-4-26
categories: JavaSE
---

第十九篇关于 Java 的笔记.
<!--more-->

# 概念的理解

- 数据库
  - 一个数据库由多个表构成，而表中存放着信息。
  - 数据库是真实存在的，它储存我们所需要的数据。
- SQL
  - SQL 也是一种语言，和其他编程语言一样，有自己语法规则。就当做是和数据库交流所使用的语言。
- MySQL
  - MySQL 是一个*关系型数据库管理系统*，也就是说它可以用 SQL 和数据库进行交流。这里的交流包括创建、修改、删除数据库等。
- MySQL 服务器端和客户端
  - 服务器端，真正执行 SQL 语句，从而操作数据库。
  - 客户端，获取到 SQL 语句，然后发送给服务器端。

# JDBC 的介绍

首先需要了解，数据库管理系统不止 MySQL 一个，它们之间的语法也有所不同。假设没有 JDBC，那么编写连接数据库的代码的时候，就需要根据实际情况，编写特定的代码。而且当更换数据库时，需要重新写一遍代码。

而 JDBC 规定了一系列接口，各个数据库厂商实现了这些接口，并提供了连接数据库的工具包，使得 Java 可以很方便的连接到数据库，并进行增删改查操作。

**实际上的连接更加复杂，以上的介绍仅仅是为了方便看懂接下去的代码。**

## JDBC 中常用的类

> DriverManager：此类管理数据库驱动程序列表。
>
> Driver ：此接口处理与数据库服务器的通信。

- Connection：应用程序与数据库的连接。
- Statement：用于执行 SQL 语句。
- ResultSet：用于接收 SQL 语句得到的结果。
  - .next() 指向下一条结果
  - .previous() 指向上一条结果
  - .absolute() 定位到某行
  - .beforeFirst() 定位到表头
  - .afterLast() 定位到最后一行
  - .getXXX(列名/索引) 获取一行中特定的列
- SQLException：连接过程中可能抛出的异常。

# JDBC 连接 MySQL 的模板

1. 装载驱动程序
2. 建立数据库连接
3. 执行 SQL 语句
4. 获取执行结果
5. 释放连接

```java
package myJDBC;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;


public class Test1 {

	public static void JdbcExample() {

		String JDBC_DRIVER = "com.mysql.jdbc.Driver";
		String DB_URL = "jdbc:mysql://127.0.0.1:3306/Test";
		String USER = "DLZ";
		String PASSWORD = "fuck";
		Connection conn = null;
		Statement stmt = null;
		ResultSet rs = null;

		try {
//			1. 装载驱动程序
			Class.forName(JDBC_DRIVER);
//			2. 建立数据库连接
			conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
//			3. 执行 SQL 语句
			stmt = conn.createStatement();
			rs = stmt.executeQuery("SELECT * FROM products WHERE vend_id = 1003");
//			4. 获取执行结果
			while(rs.next()) {
				System.out.println(rs.getString(2) + "," + rs.getString(5));
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
//			5. 释放连接
			try {
				if(conn != null) {
					conn.close();
				}
				if(stmt != null) {
					stmt.close();
				}
				if(rs != null) {
					rs.close();
				}
			} catch(Exception e) {
				//ignore
			}
		}
		
		
	}

	
	public static void main(String[] args) {
		JdbcExample();
	}

}
```

# 读取记录过多问题

需要读取的记录过多，导致 Java 内存溢出。

解决方法：通过游标，每次从服务器端读取一部分结果集。

- 在 DB_URL 加上 useCursorFetch=true
- 使用 PreparedStatement 替代 Statement
- PreparedStatement 在创建时就要传入 SQL 语句
- 通过 .setFetchSize() 设置每次读取的大小
- 通过 .executeQuery() 执行 SQL 语句

```java
package myJDBC;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;

/**
 * 2018年04月24日21:38:17
 * 使用游标，每次从数据库中读取一部分数据。
 * 避免 JVM 内存溢出
 * @author dlz
 *
 */

public class Test2 {
	
	public static void JdbcExample() {

		String JDBC_DRIVER = "com.mysql.jdbc.Driver";
		String DB_URL = "jdbc:mysql://127.0.0.1:3306/Test";
		String USER = "DLZ";
		String PASSWORD = "fuck";
		Connection conn = null;
		PreparedStatement pstmt = null;
		ResultSet rs = null;
		
		try {
			Class.forName(JDBC_DRIVER);
//			需要在连接数据库时加上这句。
			DB_URL += "?useCursorFetch=true";
			conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
			
//			在创建 prepareStatement 对象时就需要传入 sql 语句。
			pstmt = conn.prepareStatement("SELECT * FROM products WHERE vend_id = ?");
			pstmt.setFetchSize(1);  //设置每次读取的大小
			pstmt.setString(1, "1003");  //将第一个问号改为 1003
			rs = pstmt.executeQuery();  //执行 sql 语句
			while(rs.next()) {
				System.out.println(rs.getString(2) + "," + rs.getString(5));
			}
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if(conn != null) {
					conn.close();
				}
				if(pstmt != null) {
					pstmt.close();
				}
				if(rs != null) {
					rs.close();
				}
			} catch(Exception e) {
				//ignore
			}
		}
		
		
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		JdbcExample();
	}

}
```

# 读取大数据问题

读取数据库表大字段，比如数据库中，有一条大小为 1G 的数据，读入内存中同样可能发生溢出的问题。

解决方法：通过流方式解决。

```java
package myJDBC;

import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.io.OutputStream;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

/**
 * 2018年04月24日21:52:52
 * 流方式处理数据库中的大字段，比如很长的文章，目的仍然是避免 JVM 内存溢出。
 * @author dlz
 *
 */

public class Test3 {
	
	public static void JdbcExample() {

		String JDBC_DRIVER = "com.mysql.jdbc.Driver";
		String DB_URL = "jdbc:mysql://127.0.0.1:3306/Test";
		String USER = "DLZ";
		String PASSWORD = "fuck";
		Connection conn = null;
		Statement stmt = null;
		ResultSet rs = null;
		
		try {
			Class.forName(JDBC_DRIVER);
			conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
			
			stmt = conn.createStatement();
			rs = stmt.executeQuery("SELECT * FROM products WHERE vend_id = 1003");
			String FILE_URL = "temp.txt";
			InputStream in = null;
			File f = new File(FILE_URL);
			OutputStream out = null;
			out = new FileOutputStream(f);
			while(rs.next()) {
				System.out.println(rs.getString(2) + "," + rs.getString(5));
				
//				将大数据文件，用二进制读取，避免内存溢出。
				in = rs.getBinaryStream("vend_id");
				int temp = 0;
				while((temp = in.read()) != -1) {
					out.write(temp);
				}
				out.write("\n".getBytes());
			}
			in.close();
			out.close();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if(conn != null) {
					conn.close();
				}
				if(stmt != null) {
					stmt.close();
				}
				if(rs != null) {
					rs.close();
				}
			} catch(Exception e) {
				//ignore
			}
		}
		
		
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		JdbcExample();
	}

}
```

# 大量插入数据问题

大量的数据插入操作，导致效率低下，因为发送和接收 SQL 语句都需要时间。

解决方法：批处理，一次发送多条 SQL 语句。

- .addBatch() 将 SQL 语句打包
- .executeBatch() 执行打包的 SQL 语句
- clearBatch() 清空以便下次使用

```java
package myJDBC;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class Test4 {
	
	public static void JdbcExample() {

		String JDBC_DRIVER = "com.mysql.jdbc.Driver";
		String DB_URL = "jdbc:mysql://127.0.0.1:3306/Test";
		String USER = "DLZ";
		String PASSWORD = "fuck";
		Connection conn = null;
		Statement stmt = null;
		
		try {
			Class.forName(JDBC_DRIVER);
			DB_URL += "?useUnicode=true&characterEncoding=utf8";
			conn = DriverManager.getConnection(DB_URL, USER, PASSWORD);
			
			stmt = conn.createStatement();
			stmt.addBatch("INSERT INTO customers2 VALUES (NULL, \"1\",\"77\", \"77\", \"77\", \"77\", \"77\", NULL, NULL)"); // 添加一条语句
			stmt.addBatch("INSERT INTO customers2 VALUES (NULL, \"1\",\"77\", \"77\", \"77\", \"77\", \"77\", NULL, NULL)"); // 添加一条语句
			stmt.executeBatch(); // 一次性执行
			stmt.clearBatch(); // 清空，以便下次使用

			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				if(conn != null) {
					conn.close();
				}
				if(stmt != null) {
					stmt.close();
				}
			} catch(Exception e) {
				//ignore
			}
		}
		
		
	}

	public static void main(String[] args) {
		JdbcExample();
	}

}
```

# 中文编码问题

在 MySQL 中输入

```mysql
show variables like "%character%";
```

可以查看到 character_set_database 和 character_set_server 的编码。

在进入某个数据库后，执行

```mysql
show create table customers;
```

可以看出某个数据表、某个字段的编码。

优先级从高到低：

column > Table > Database > Server

对数据库而言，需要将编码设置为 utf8；

```mysql
set character_set_server = utf8;
```

在创建数据表时，也可以设置编码。

对于 Java 程序而言，需要在 DB_URL 后面加上：

```java
"?useUnicode=true&characterEncoding=utf8"
```



