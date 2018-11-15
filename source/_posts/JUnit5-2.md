---
title: JUnit5 的基本使用
date: 2018-11-10
categories: JUnit5
---

介绍 JUnit5 中常用的注解、断言以及前置条件。

<!--more-->

# 注解

| 常用注解    | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| @Test       | 被注解的方法是一个测试方法。                                 |
| @BeforeAll  | 被注解的（静态）方法将在当前类中的所有 @Test 方法前执行一次。 |
| @BeforeEach | 被注解的方法将在当前类中的每个 @Test 方法前执行。            |
| @AfterEach  | 被注解的方法将在当前类中的每个 @Test 方法后执行。            |
| @AfterAll   | 被注解的（静态）方法将在当前类中的所有 @Test 方法后执行一次。 |
| @Disabled   | 被注解的方法不会执行（将被跳过），但会报告为已执行。         |

**注意：**被`@Test`、`@BeforeAll`、`@AfterAll`、`@BeforeEach` 或 `@AfterEach` 注解标注的方法不可以有返回值。 

## 示例

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.*;

/**
 * @program: JunitTest
 * @description: Test3
 * @author: diaolizhi
 * @create: 2018-11-10 10:57
 **/
public class Test3 {

    @Test
    @DisplayName("测试 1 + 2 是否等于 3")
    void testAdd() {
        assertEquals(1 + 2, 3);
    }

    @Test
    @DisplayName("测试 3 / 0")
    void testDivision() {
        float a = 3 / 0;
    }

    @BeforeAll
    static void first() {
        System.out.println("----测试开始");
    }

    @BeforeEach
    void beforeEach() {
        System.out.println("--将执行新的测试");
    }

    @AfterAll
    static void last() {
        System.out.println("----测试结束");
    }

    @AfterEach
    void afterEach() {
        System.out.println("--结束一个测试");
    }

    @Test
    @Disabled
    @DisplayName("跳过测试但显示执行成功")
    void disabled() {
        float a = 3 / 0;
    }
}
```

运行结果：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/JUnit5-example.png)

通过上面的注解：

- 可以在测试开始时、结束时运行一个指定的方法

- 可以在每个测试方法运行前后执行指定方法
- 可以使用 `@DisplayName`注解一个方法，这样在测试报告中就显示更有意义的信息，而不是方法名。
- 可以使用 `@Disabled`注解一个方法，在测试时跳过这个方法，并在报告中显示执行成功。



# 断言

断言是用来测试一个条件是否满足，如果满足就往下执行，如果不满足就报告测试不通过。

## JUnit Jupiter 中的断言

| 断言方法                       | 说明                                                |
| ------------------------------ | --------------------------------------------------- |
| assertEquals(expected, actual) | 如果 *expected* 不等于 *actual*，则断言失败。       |
| assertFalse(booleanExpression) | 如果 *booleanExpression* 不是 `false`，则断言失败。 |
| assertNull(actual)             | 如果 *actual* 不是 `null`，则断言失败。             |
| assertNotNull(actua            | 如果 *actual* 是 `null`，则断言失败。               |
| assertTrue(booleanExpression)  | 如果 *booleanExpression* 不是 `true`，则断言失败。  |

## 示例

```java
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

/**
 * @program: JunitTest
 * @description: Test4
 * @author: diaolizhi
 * @create: 2018-11-10 11:22
 **/
public class Test4 {

    @Test
    void testAssert() {
        assertEquals(1, 1);
        assertFalse(1 != 1);
        assertNull(null);
        assertNotNull(new Integer(0));
        assertNotNull(1 == 1);
    }
}
```



## message 参数

断言方法都有一个缺省的 message 参数，用于在断言失败时显示。

### 示例

```java
@Test
void testMessage() {
    assertEquals(1, 2, "测试 1 和 2");
}
```

显示结果：

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/assertiMessage.png)



## assertAll 方法

### 示例

```java
@Test
void testAll() {
    assertAll(
        "进行多次断言",
        () -> assertEquals(1, 2),
        () -> assertTrue(1 == 1),
        () -> assertFalse(true)
    );
}
```

使用 assertAll 可以进行多次断言，并且即使某个断言失败，仍然会往下执行。

assertAll 的第一个参数类似于上面说的 message 参数，在断言失败时会显示。



## assertThrows 方法

### 示例

```java
@Test
void testException() {
    assertThrows(ArithmeticException.class, () -> {float a = 3 / 0;});
}
```

用于测试是否抛出指定的异常，如果抛出，测试通过。



# 前置条件

当满足前置条件时才执行真正的测试代码，前置条件是 org.junit.jupiter.api.Assumptions 中的静态方法。

## 示例

```java
@Test
void testOnlyFriday() {
    LocalDateTime time = LocalDateTime.now();
    assumeTrue(time.getDayOfWeek().getValue() == 5);
    float a = 3 / 0;
}
```

上面这个示例表示的是：如果今天是星期五，就往下进行测试。

看起来使用断言也可以完成同样的工作，但实际上断言和前置条件是不同的，从测试结果就可以看出来。

如果使用的是断言：断言不满足，测试结束，报告测试不通过。

如果使用的是前置条件：前置条件不满足，测试结束，报告测试被忽略（ignored）。



## assumingThat 方法

假设有这样一个需求：如果不满足前置条件就不执行某个测试，但是剩下的测试还是要执行。那么就要使用 assumingThat 方法了。

### 示例

```java
@Test
void testOnlyFriday2() {
    LocalDateTime time = LocalDateTime.now();
    assumingThat(time.getDayOfWeek().getValue() == 5, () -> {
        float a = 3 / 0;
    });
    int b = 1;
    System.out.println("b = " + b);
}
```

如果今天不是星期五，那么就不测试 `() -> {float a = 3 / 0;}`，剩下的代码（从 `int b = 1`开始）还是照样执行。

另外，在这个示例中，即便今天不是星期五，最后的报告也是测试通过，而不是被忽略。



# 总结

本文介绍了几个 JUnit5 中的注解，还有断言以及前置条件。

要了解更多特性和方法只能去看[官方文档](https://junit.org/junit5/docs/current/user-guide/)。