## Java基础-008：日期和时间（`java.time.*`）

本篇我们继续来看位于 `java.time.*` 包下日期和时间类。

### 1、计算两个 `LocalDate` 之间的差值

使用 `LocalDate` 和 `ChronoUnit` ：

```java
LocalDate d1 = LocalDate.of(2017, 5, 1);
LocalDate d2 = LocalDate.of(2017, 5, 18);
```

现在，由于 `ChronoUnit` 枚举器的 `between` 方法有 2 个 `Temporal` 对象作为参数，因此您可以毫无问题地传递 LocalDate 实例：

```java
long days = ChronoUnit.DAYS.between(d1, d2);
System.out.println(days); // 17
```

### 2、日期和时间

不带时区信息的日期和时间

```java
LocalDateTime dateTime = LocalDateTime.of(2016, Month.JULY, 27, 8, 0);
LocalDateTime now = LocalDateTime.now();
LocalDateTime parsed = LocalDateTime.parse("2016-07-27T07:00:00");
```

带时区信息的日期和时间

```java
ZoneId zoneId = ZoneId.of("UTC+2");
ZonedDateTime dateTime = ZonedDateTime.of(2016, Month.JULY, 27, 7, 0, 0, 235, zoneId);
LocalDate localDate = LocalDate.of(2016, 7, 27);
LocalTime localTime = LocalTime.of(7, 0, 0, 235);
ZonedDateTime composition = ZonedDateTime.of(localDate, localTime, zoneId);
ZonedDateTime now = ZonedDateTime.now(); // 默认时区
ZonedDateTime parsed = ZonedDateTime.parse("2016-07-27T07:00:00+01:00[Europe/Stockholm]");
```

带有偏移信息的日期和时间（即不考虑 DST 更改）

```java
ZoneOffset zoneOffset = ZoneOffset.ofHours(2);
OffsetDateTime dateTime = OffsetDateTime.of(2016, 7, 27, 7, 0, 0, 235, zoneOffset);
LocalDate localDate = LocalDate.of(2016, 7, 27);
LocalTime localTime = LocalTime.of(7, 0, 0, 235);
OffsetDateTime composition = OffsetDateTime.of(localDate, localTime, zoneOffset);
OffsetDateTime now = OffsetDateTime.now(); // Offset taken from the default ZoneId
OffsetDateTime parsed = OffsetDateTime.parse("2016-07-27T07:00:00+02:00");
```

### 3、日期和时间的操作

```java
LocalDate tomorrow = LocalDate.now().plusDays(1);
LocalDateTime anHourFromNow = LocalDateTime.now().plusHours(1);
Long daysBetween = java.time.temporal.ChronoUnit.DAYS.between(LocalDate.now(), LocalDate.now().plusDays(3)); // 3
Duration duration = Duration.between(Instant.now(), ZonedDateTime.parse("2016-07-27T07:00:00+01:00[Europe/Stockholm]"));
```

### 4、`Instant` 类

代表即刻的时间。可以被认为是 Unix 时间戳的包装器。

```java
Instant now = Instant.now();
System.out.println(now); // 2022-08-11T08:45:48.734Z
Instant epoch1 = Instant.ofEpochMilli(0);
Instant epoch2 = Instant.parse("1970-01-01T00:00:00Z");
System.out.println(ChronoUnit.MICROS.between(epoch1, epoch2)); // 0
```

### 5、各种日期时间类 API 的用法

以下示例还具有助于理解所需的注释。

```java
import java.time.*;
import java.util.TimeZone;

public class SomeMethodsExamples {

    /**
     * {@link LocalDateTime} 包含的一些方法
     */
    public static void checkLocalDateTime() {
        LocalDateTime localDateTime = LocalDateTime.now();
        System.out.println("Local Date time using static now() method ::: >>> "
                + localDateTime);
        LocalDateTime ldt1 = LocalDateTime.now(ZoneId.of(ZoneId.SHORT_IDS
                .get("AET")));
        System.out
                .println("LOCAL TIME USING now(ZoneId zoneId) method ::: >>>>"
                        + ldt1);
        LocalDateTime ldt2 = LocalDateTime.now(Clock.system(ZoneId
                .of(ZoneId.SHORT_IDS.get("PST"))));
        System.out
                .println("Local TIME USING now(Clock.system(ZoneId.of())) ::: >>>> "
                        + ldt2);
        System.out
                .println("Following is a static map in ZoneId class which has mapping of short timezone names to their Actual timezone names");
        System.out.println(ZoneId.SHORT_IDS);
    }

    /**
     * {@link LocalDate} 包含的一些方法
     */
    public static void checkLocalDate() {
        LocalDate localDate = LocalDate.now();
        System.out.println("Gives date without Time using now() method. >> "
                + localDate);
        LocalDate localDate2 = LocalDate.now(ZoneId.of(ZoneId.SHORT_IDS.get("ECT")));
        System.out
                .println("now() is overridden to take ZoneID as parameter using this we can get the same date under different timezones. >> "
                        + localDate2);
    }

    /**
     * 抽象类 {@link Clock} 包含的一些方法。
     * Clock 可以用于表示带有时区 {@link TimeZone} 的时间
     */
    public static void checkClock() {
        Clock clock = Clock.systemUTC();
        // 根据 ISO 8601 表示时间
        System.out.println("Time user Clock class: " + clock.instant());
    }

    /**
     * {@link Instant} 包含的方法
     */
    public static void checkInstant() {
        Instant instant = Instant.now();
        System.out.println("Instant using now() method :: " + instant);

        Instant ins1 = Instant.now(Clock.systemUTC());
        System.out.println("Instant using now(Clock clock) :: " + ins1);
    }

    /**
     * 检查 {@link Duration} 类的方法
     */
    public static void checkDuration() {
        // toString() 方法会根据 ISO-8601 标准将 duration 转换为 PTnHnMnS 格式
        // 如果某个属性为 0 的话就忽略。

        // P 是放置在期间表达式开始处的 duration 符号（历史上称为“period”）
        // Y 是年数值之后的年份符号
        // M 是月份数之后的月份符号
        // W 是周数值之后的周符号
        // D 是天数值之后的日期符号
        // T 是表示的时间分量之前的时间符号
        // H 是小时数值之后的小时符号
        // M 是分钟数之后的分钟符号
        // S 是秒数值之后的秒符号
        System.out.println(Duration.ofDays(2));
    }

    /**
     * 显示不带日期的当地时间。它不存储或表示日期和时间。
     * 相反，它是时间的代表，就像墙上的时钟。
     */
    public static void checkLocalTime() {
        LocalTime localTime = LocalTime.now();
        System.out.println("LocalTime :: " + localTime);
    }

    /**
     * ISO-8601 标准格式，带有详细时区的日期时间，
     */
    public static void checkZonedDateTime() {
        ZonedDateTime zonedDateTime = ZonedDateTime.now(ZoneId.of(ZoneId.SHORT_IDS.get("CST")));
        System.out.println(zonedDateTime);
    }
}
```

### 6、日期时间格式化

Java 8 之前，`java.text` 包中有 `DateFormat` 和 `SimpleDateFormat` 类，这些遗留代码将继续使用一段时间。

但是，Java 8 提供了一种现代的方法来处理格式化和转化。

在格式化和转化中，首先将 `String` 对象传递给 `DateTimeFormatter`，然后将其用于格式化或转化。

```java
import java.time.*;
import java.time.format.*;

class DateTimeFormat {

    public static void main(String[] args) {
        // 转化解析
        String pattern = "d-MM-yyyy HH:mm";
        DateTimeFormatter dtF1 = DateTimeFormatter.ofPattern(pattern);
        LocalDateTime ldp1 = LocalDateTime.parse("2014-03-25T01:30"), // 默认格式
                ldp2 = LocalDateTime.parse("15-05-2016 13:55", dtF1); // 自定义格式
        System.out.println(ldp1 + "\n" + ldp2); // 会按照默认格式打印

        // 格式化
        DateTimeFormatter dtF2 = DateTimeFormatter.ofPattern("EEE d, MMMM, yyyy HH:mm");
        DateTimeFormatter dtF3 = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
        LocalDateTime ldtf1 = LocalDateTime.now();
        System.out.println(ldtf1.format(dtF2) + "\n" + ldtf1.format(dtF3));
    }
}
```

一个重要的注意，相对于自定义格式，使用预定义的格式化程序是一种很好的做法。 这样你的代码看起来更清晰，从长远来看，`ISO8061` 的使用肯定会对你有所帮助。

### 7、简单的日期操作

```java
// 获取当前日期
LocalDate.now();
// 获取昨天的日期
LocalDate yesterDay = LocalDate.now().minusDays(1);
// 获取明天的日期
LocalDate tomorrowDay = LocalDate.now().plusDays(1);
// 获取具体的日期
LocalDate t = LocalDate.of(1974, 6, 2, 8, 30, 0, 0);
// 除了 plus 和 minus 方法，还有一系列带有 with 的方法，可用于在 LocalDate 实例上设置特定时间
LocalDate.now().withMonth(6);
```

上面的示例会返回一个新实例，其月份设置为 6 月（这与 `java.util.Date` 不同，其中 `setMonth` 的索引起始为 0 ，设置为 5 的时候结果才是 6 月）。

因为 LocalDate 操作返回不可变的 LocalDate 实例，所以这些方法也可以链接在一起。

```java
// 这会返回一个从今天算一年后明天的日期
LocalDate ld = LocalDate.now().plusDay(1).plusYear(1);
```

### 链接

- 测试代码： https://github.com/lq920320/java-tests/tree/master/src/test/java/base
