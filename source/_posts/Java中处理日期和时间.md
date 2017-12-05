---
title: Java 中处理日期和时间
date: 2017-12-5
categories: JavaSE
---

第三篇关于 Java 的笔记.
<!--more-->

# 1. Instant 和 Duration
- Instant 类的实例代表时间轴中的一个时刻, 它有静态方法 now() 可以取出当前时间.
- Duration 则代表一个时间段 (比如 "10 秒长的一段时间")

```java
Instant start = Instant.now();
System.out.println(start);

Instant end = Instant.now();
System.out.println(end);

Duration elapse = Duration.between(start, end);
System.out.println(elapse.toMillis());
```

# 2. 本地时间和日期

```java
LocalDate currentDate = LocalDate.now();
System.out.println(currentDate);  //2017-12-5

//现在月份是基于 1 的了
LocalDate specificDate = LocalDate.of(2000, 1, 1);
System.out.println(specificDate);

LocalTime currentTime = LocalTime.now();
System.out.println(currentTime);

LocalTime specificTime = LocalTime.of(14, 10, 12);
System.out.println(specificTime);

LocalDateTime currentDT = LocalDateTime.now();
//输出 ISO 格式的日期时间: 2014-08-31T18:52:20.321
System.out.println(currentDT);

//日期时间的合成
LocalDateTime specificDt = LocalDateTime.of(specificDate, specificTime);
//2000-01-01T14:10:12
System.out.println(specificDT);
```

# 3. 日期格式化-1
使用 DateTimeFormatter 格式化日期
```java
LocalDate currentDate1 = LocalDate.now();
DateTimeFormatter dt = DateTimeFormatter.ISO_DATE;
//输出样例: 2014-08-31
System.out.println(dt.format(currentDate1));

LocalTime currentTime1 = LocalTime.now();
DateTimeFormatter tf = DateTimeFormatter.ISO_TIME;
//输出样例: 23:01:02.706
System.out.println(tf.format(currentTime1));

LocalDateTime currentDateTime = LocalDateTime.now();
DateTimeFormatter dtf = DateTimeFormatter.ISO_DATE_TIME;
//输出样例: 2014-08-31T21:28:48.147
System.out.println(dtf.format(currentDateTime));

DateTimeFormatter f_long = DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG);
//输出样例: 2014年8月31日
System.out.println(f_long.format(currentDateTime));

DateTimeFormatter f_short = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);
//输出样例: 14-8-31
System.out.println(f_short.format(currentDateTime));
```

# 4. 日期格式化-2

```java
DateTimeFormatterBuilder b = new DateTimeFormatterBuilder()
.appendValue(ChronoField.MONTH_OF_YEAR)
.appendLiteral("||")
.appendValue(ChronoField.DAY_OF_MONTH)
.appendLiteral("||")
.appendValue(ChronoField.YEAR);
//输出: 8||31||2014
DateTimeFormatter f = b.toFormatter();
System.out.println(f.format(currentDateTime));
```

# 5. 处理时区

```java
DateTimeFormatter dtf = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT);
LocalDateTime dt = LocalDateTime.now();
//14-8-31 下午10:21
System.out.println(dtf.format(dt));

ZonedDateTime gmt = ZonedDateTime.now(ZoneId.of("GMT+0"));
//14-8-31 上午10:25

ZonedDateTime ny = ZonedDateTime.now(ZoneId.of("America/New_York"));
//14-8-31 上午10:25
System.out.println(dtf.format(ny));

//输出所有的 ZoneId
set<String> zones = ZoneId.getAvailableZoneIds();
zones.forEach(z->System.out.println(z));
```

# 6. 完整代码
```java
package time;

import java.time.Duration;
import java.time.Instant;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.LocalTime;
import java.time.ZoneId;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.time.format.DateTimeFormatterBuilder;
import java.time.format.FormatStyle;
import java.time.temporal.ChronoField;
import java.util.Locale;
import java.util.Set;

public class TestTime {

    public static void main(String[] args) {

//      Instant 类的实例代表时间轴中的一个时刻, 它有静态方法 now() 可以取出当前时间.
        Instant start = Instant.now();
        System.out.println(start);

        Instant end = Instant.now();
        System.out.println(end);

//      Duration 则代表一个时间段 (比如 "10 秒长的一段时间")
        Duration elapse = Duration.between(start, end);
        System.out.println(elapse.toMillis());

        
//------------------------------------------------------------------------------------
        //本地日期(上面那个不知道是哪里的时间)
        LocalDate currentDate = LocalDate.now();
        System.out.println(currentDate);  //2017-12-5
        
        //现在月份是基于 1 的了
        LocalDate specificDate = LocalDate.of(2000, 1, 1);
        System.out.println(specificDate);

        //本地时间
        LocalTime currentTime = LocalTime.now();
        System.out.println(currentTime);

        LocalTime specificTime = LocalTime.of(14, 10, 12);
        System.out.println(specificTime);

        //本地时间 + 日期
        LocalDateTime currentDT = LocalDateTime.now();
        //输出 ISO 格式的日期时间: 2014-08-31T18:52:20.321
        System.out.println(currentDT);

        //日期时间的合成
        LocalDateTime specificDT = LocalDateTime.of(specificDate, specificTime);
        //2000-01-01T14:10:12
        System.out.println(specificDT);
        
        
//---------------------------------------------------------------------------------------------------------------------------------------------
        //时间的输出样式
        
        LocalDate currentDate1 = LocalDate.now();
        DateTimeFormatter dt = DateTimeFormatter.ISO_DATE;
        //输出样例: 2014-08-31
        System.out.println(dt.format(currentDate1));

        LocalTime currentTime1 = LocalTime.now();
        DateTimeFormatter tf = DateTimeFormatter.ISO_TIME;
        //输出样例: 23:01:02.706
        System.out.println(tf.format(currentTime1));

        LocalDateTime currentDateTime = LocalDateTime.now();
        DateTimeFormatter dtf = DateTimeFormatter.ISO_DATE_TIME;
        //输出样例: 2014-08-31T21:28:48.147
        System.out.println(dtf.format(currentDateTime));

        DateTimeFormatter f_long = DateTimeFormatter.ofLocalizedDate(FormatStyle.LONG);
        //输出样例: 2014年8月31日
        System.out.println(f_long.format(currentDateTime));

        DateTimeFormatter f_short = DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);
        //输出样例: 14-8-31
        System.out.println(f_short.format(currentDateTime));
        
        
        String fr_short = f_short.withLocale(Locale.ENGLISH).format(currentDateTime);
        String fr_long = f_long.withLocale(Locale.ENGLISH).format(currentDateTime);
        //输出样例: 8/31/14
        System.out.println(fr_short);
        //输出样例: August 31, 2014
        System.out.println(fr_long);
//------------------------------------------------------------------------------------
        //自定义日期输出格式
        DateTimeFormatterBuilder b = new DateTimeFormatterBuilder()
        .appendValue(ChronoField.MONTH_OF_YEAR)
        .appendLiteral("||")
        .appendValue(ChronoField.DAY_OF_MONTH)
        .appendLiteral("||")
        .appendValue(ChronoField.YEAR);
        //输出: 8||31||2014
        DateTimeFormatter f = b.toFormatter();
        System.out.println(f.format(currentDateTime));

//-----------------------------------------------------------------------------------
        
        //处理时区
        
        DateTimeFormatter dtf1 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.SHORT);
        LocalDateTime dt1 = LocalDateTime.now();
        //14-8-31 下午10:21
        System.out.println(dtf1.format(dt1));

        ZonedDateTime gmt = ZonedDateTime.now(ZoneId.of("GMT+0"));
        //14-8-31 上午10:25

        ZonedDateTime ny = ZonedDateTime.now(ZoneId.of("America/New_York"));
        //14-8-31 上午10:25
        System.out.println(dtf1.format(ny));

        //输出所有的 ZoneId
//      Set<String> zones = ZoneId.getAvailableZoneIds();
//      zones.forEach(z->System.out.println(z));
        
    }

}
```