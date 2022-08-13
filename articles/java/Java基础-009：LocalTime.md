## Java基础-009：LocalTime

本篇我们来看下 `java.time.LocalTime` 的一些方法。

| 方法  | 输出  |
| --- | --- |
| `LocalTime.of(13, 12, 11)` | `13:12:11` |
| `LocalTime.MIDNIGHT` | `00:00` |
| `LocalTime.NOON` | `12:00` |
| `LocalTime.now()` | 系统当前时间 |
| `LocalTime.MAX` | 支持的最大当地时间 `23:59:59.999999999` |
| `LocalTime.MIN` | 支持的最小当地时间 `00:00` |
| `LocalTime.ofSecondOfDay(84399)` | `23:59:59` 从一天的秒数中获取时间 |
| `LocalTime.ofNanoOfDay(2000000000)` | `00:00:02`，从一天的纳秒中获取时间 |

### 1、两个 `LocalTime` 之间时间的差值

有两种等效的方法来计算两个 `LocalTime` 之间的时间单位量：

(1) 通过工具类(`Temporal`, `TemporalUnit`) 方法

(2) `TemporalUnit.between(Temporal, Temporal)`

```java
import java.time.LocalTime;
import java.time.temporal.ChronoUnit;

public class AmountOfTime {

    public static void main(String[] args) {
        LocalTime start = LocalTime.of(1, 0, 0); // hour, minute, second
        LocalTime end = LocalTime.of(2, 10, 20); // hour, minute, second
        long halfDays1 = start.until(end, ChronoUnit.HALF_DAYS); // 0
        long halfDays2 = ChronoUnit.HALF_DAYS.between(start, end); // 0
        long hours1 = start.until(end, ChronoUnit.HOURS);// 1
        long hours2 = ChronoUnit.HOURS.between(start, end); // 1
        long minutes1 = start.until(end, ChronoUnit.MINUTES); // 70
        long minutes2 = ChronoUnit.MINUTES.between(start, end); // 70
        long seconds1 = start.until(end, ChronoUnit.SECONDS); // 4220
        long seconds2 = ChronoUnit.SECONDS.between(start, end); // 4220
        long millisecs1 = start.until(end, ChronoUnit.MILLIS); // 4220000
        long millisecs2 = ChronoUnit.MILLIS.between(start, end); // 4220000
        long microsecs1 = start.until(end, ChronoUnit.MICROS); // 4220000000
        long microsecs2 = ChronoUnit.MICROS.between(start, end); // 4220000000
        long nanosecs1 = start.until(end, ChronoUnit.NANOS); // 4220000000000
        long nanosecs2 = ChronoUnit.NANOS.between(start, end); // 4220000000000
        // 使用其它的单位时间单位会抛出 UnsupportedTemporalTypeException
        // 下面是会抛出异常的例子.
        long days1 = start.until(end, ChronoUnit.DAYS);
        long days2 = ChronoUnit.DAYS.between(start, end);
    }
}
```

### 2、介绍

`LocalTime` 是一个不可变的类和线程安全的，用于表示时间，通常被视为小时-分钟-秒。时间以纳秒精度表示。 例如，值“13:45.30.123456789”可以存储在 `LocalTime` 中。
此类不存储或表示日期或时区。 相反，它是对挂钟上的当地时间的描述。 如果没有诸如偏移或时区之类的附加信息，它就不能代表时间线上的瞬间。 这是一个基于值的类，应该使用 `equals` 方法进行比较。

**属性**

`MAX` - 当地时间支持的最大值，`23:59:59.999999999` 。类似的还有 `MIDNIGHT`、`MIN`、`NOON`。

**重要的静态方法**

`now()`、`now(Clock clock)`、`now(ZoneId zone)`、`parse(CharSequence text)`。

**重要的实例方法**

`isAfter(LocalTime other)`、 `isBefore(LocalTime other)`、 `minus(TemporalAmount amountToSubtract)`、`minus(long amountToSubtract, TemporalUnit unit)`、`plus(TemporalAmount amountToAdd)`、 `plus(long amountToAdd, TemporalUnit unit)`

```java
ZoneId zone = ZoneId.of("Asia/Kolkata");
LocalTime now = LocalTime.now();
LocalTime now1 = LocalTime.now(zone);
LocalTime then = LocalTime.parse("04:16:40");
```

计算两个时间之差可以使用下面的任意一种方法：

```java
long timeDiff = Duration.between(now, now1).toMinutes();
long timeDiff1 = java.time.temporal.ChronoUnit.MINUTES.between(now2, now1);
```

你还可以从 `LocalTime` 的任何对象中添加/减去小时、分钟或秒。

```java
now.plusHours(1L);
now1.minusMinutes(20L);
```

### 3、时间修改

你可以增加小时、分钟、秒以及纳秒：

```java
LocalTime time = LocalTime.now();
LocalTime addHours = time.plusHours(5); // Add 5 hours
LocaLTime addMinutes = time.plusMinutes(15) // Add 15 minutes
LocalTime addSeconds = time.plusSeconds(30) // Add 30 seconds
LocalTime addNanoseconds = time.plusNanos(150_000_000) // Add 150.000.000ns (150ms)
```

### 4、时区以及它们的时差

```java
    @Test
    public void test() {
        ZoneId zone1 = ZoneId.of("Europe/Berlin");
        ZoneId zone2 = ZoneId.of("Brazil/East");
        LocalTime now = LocalTime.now();
        LocalTime now1 = LocalTime.now(zone1);
        LocalTime now2 = LocalTime.now(zone2);
        System.out.println("Current Time : " + now);
        System.out.println("Berlin Time : " + now1);
        System.out.println("Brazil Time : " + now2);
        long minutesBetween = ChronoUnit.MINUTES.between(now2, now1);
        System.out.println("Minutes Between Berlin and Brazil : " + minutesBetween + "mins");
    }
```

### 链接

- 测试代码： https://github.com/lq920320/java-tests/tree/master/src/test/java/base