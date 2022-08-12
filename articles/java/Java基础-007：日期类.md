## Java基础-007：日期类

本篇我们来回顾一下日期类的使用。

| 参数          | 解释                                                              |
| ----------- | --------------------------------------------------------------- |
| 无参数         | 使用分配时间（精确到毫秒）创建一个新的 `Date` 对象                                   |
| `long date` | 使用参数为“纪元”（`GMT 1970-01-01 00:00:00`）<br/>以来的毫秒数创建一个新的 `Date` 对象 |

### 1、将 `java.util.Date` 转换为 `java.sql.Date`

`java.util.Date` 到 `java.sql.Date` 的转换通常是很必要的，当一个 `Date` 对象需要保存到数据库中。

`java.sql.Date` 是毫秒值的包装类，`JDBC` 使用它来标识 SQL `DATE` 类型（即格式为 “yyyy-MM-dd”，不含时间部分）。

在下面的例子里，我们使用 `java.util.Date` 的构造器创建了一个对象，并且初始化为当前最近的毫秒时间。这个日期会传入 `convert(java.util.Date)` 方法，然后返回 `java.sql.Date` 对象。

```java
    @Test
    public void utilDateToSqlDateTest() {
        java.util.Date utilDate = new java.util.Date();
        System.out.println("java.util.Date is : " + utilDate);
        java.sql.Date sqlDate = convert(utilDate);
        System.out.println("java.sql.Date is : " + sqlDate);
        DateFormat df = new SimpleDateFormat("dd/MM/yyyy - HH:mm:ss");
        System.out.println("dateFormated date is : " + df.format(utilDate));
    }

    private static java.sql.Date convert(java.util.Date uDate) {
        java.sql.Date sDate = new java.sql.Date(uDate.getTime());
        return sDate;
    }
```

输出如下：

```
java.util.Date is : Wed Aug 10 15:31:55 CST 2022
java.sql.Date is : 2022-08-10
dateFormated date is : 10/08/2022 - 15:31:55
```

`java.util.Date` 包含日期和时间所有信息，而 `java.sql.Date` 值包含日期信息。

### 2、一个基本日期输出

使用下面的代码以及 `yyyy/MM/dd hh:mm.ss` 的格式字符串，我们会得到下面的输出：

```
2016/04/19 11:45.36
```

```java
// 定义要使用的格式
String formatString = "yyyy/MM/dd hh:mm.ss";

// 获取当前的日期对象
Date date = Calendar.getInstance().getTime();

// 创建一个格式化工具
SimpleDateFormat simpleDateFormat = new SimpleDateFormat(formatString);

// 格式日期
String formattedDate = simpleDateFormat.format(date);

// 打印
System.out.println(formattedDate);

// 一行代码流
System.out.println(new SimpleDateFormat("yyyy/MM/dd hh:mm.ss").format(Calendar.getInstance().getTime()));
```

### 3、Java 8的 `LocalDate` 和 `LocalDateTime` 对象

`Date` 和 `LocalDate` 对象不能彼此精确转换，因为 `Date` 对象同时代表一个
特定的日期和时间，而 `LocalDate` 对象不包含时间或时区信息。 然而，它可以
如果您只关心实际日期信息而不关心时间信息，则在两者之间进行转换很有用。

**创建一个 `LocalDate`**

```java
// 创建一个默认对象，当前日期
LocalDate lDate = LocalDate.now();

// 由一些参数来创建
lDate = LocalDate.of(2022, 8, 1);

// 由一个字符串创建
lDate = LocalDate.parse("2022-08-01");

// 由时区创建
LocalDate.now(ZoneId.systemDefault());
```

**创建一个 `LocalDateTime`**

```java
// 创建一个默认对象，当前日期时间
LocalDateTime lDateTime = LocalDateTime.now();

// 由一些参数来创建
lDateTime = LocalDateTime.of(2017, 12, 15, 11, 30);

// 由一个字符串创建
lDateTime = LocalDateTime.parse("2017-12-05T11:30:30");

// 由时区创建
LocalDateTime.now(ZoneId.systemDefault());
```

**`LocalDate` 到 `Date`，反之亦然**

```java
Date date = Date.from(Instant.now());
ZoneId defaultZoneId = ZoneId.systemDefault();

// Date 转 LocalDate
LocalDate localDate = date.toInstant().atZone(defaultZoneId).toLocalDate();

// LocalDate 转 Date
date = Date.from(localDate.atStartOfDay(defaultZoneId).toInstant());
```

**`LocalDateTime` 到 `Date`，反之亦然**

```java
Date date = Date.from(Instant.now());
ZoneId defaultZoneId = ZoneId.systemDefault();
// Date 转 LocalDateTime
LocalDateTime localDateTime = date.toInstant().atZone(defaultZoneId).toLocalDateTime();
// LocalDateTime 转 Date
Date out = Date.from(localDateTime.atZone(defaultZoneId).toInstant());
```

### 4、创建一个具体的 `Date`

虽然 Java Date 类有多个构造函数，但您会注意到大多数都已被弃用。 直接创建 Date 实例的唯一可以接受的方式是使用空构造函数或传入 long（自标准基准时间以来的毫秒数）。 除非你正准备获取当前日期或已经有了另一个 Date 实例。

创建一个新的日期，你将会需要一个 `Calendar` 实例。你可以从 `Calendar` 进行设置为你想要的日期。

```java
Calendar c = Calendar.getInstance();
```

这将返回一个设置为当前时间的新的 `Calendar` 实例。`Calendar` 有很多方法可以改变它的日期和时间或直接设置。 在这种情况下，我们会将其设置为具体的日期。

```java
c.set(1974, 6, 2, 8, 0, 0, 0);
Date d = c.getTime();
```

`getTime` 方法便会返回我们需要的 `Date` 实例。请注意，`Calendar` 的 `set` 方法只设置一个或多个字段，便不会全部设置。 也就是说，如果设置年份，其他字段将保持不变。

**陷阱**

在许多情况下，此代码片段可以实现其目的，但请记住，日期/时间的两个重要部分未定义。

- 参数 `(1974, 6, 2, 8, 0, 0)` 将会带着在其他地方设置的默认时区执行。

- 毫秒部分没有设置为零，而是在创建 `Calendar` 实例时从系统时钟填充。

### 5、将 `Date` 转换为一个字符串格式

`format()` 来自于 `SimpleDateFormat` 类，我们可以借助应用 *格式字符串* 将 `Date` 对象转换为一个特定格式的 `String` 对象。

```java
Date today = new Date();
SimpleDateFormat dateFormat = new SimpleDateFormat("dd-MMM-yy"); // 在这里指定格式
System.out.println(dateFormat.format(today)); // 10-八月-22
```

可以使用 `applyPattern()` 再次应用（另一个）格式。

```java
dateFormat.applyPattern("dd-MM-yyyy");
System.out.println(dateFormat.format(today)); // 10-08-2022
dateFormat.applyPattern("dd-MM-yyyy HH:mm:ss E");
System.out.println(dateFormat.format(today)); // 10-08-2022 16:48:08 星期三
```

**注意：** 这里的 `mm` （小写的 m）表示分钟，而 `MM` （大写的 M）表示月份。当格式化年份的时候，要注意：字母 “Y” 表示“一年的星期”，而小写的 “y” 才表示年份。

### 6、`LocalTime`

如果只是用到 `Date` 中的时间部分，就可以使用 `LocalTime`。这有几种方式可以让你用来初始化一个 `LocalTime` 对象：

1. `LocalTime time = LocalTime.now();`

2. `time = LocalTime.MIDNIGHT;`

3. `time = LocalTime.NOON;`

4. `time = LocalTime.of(12, 12, 45);`

`LocalTime` 也有一个内置的 `toString` 方法，可以很好地显示格式。

```java
System.out.println(time);
```

你还可以从 `LocalTime` 对象中获取、添加和减去小时、分钟、秒和纳秒，即

```java
time.plusMinutes(1);
time.getMinute();
time.minusMinutes(1);
```

你也可以使用下面的代码将它转为 `Date` 对象：

```java
LocalTime lTime = LocalTime.now();
Instant instant = lTime.atDate(LocalDate.of(2022, 8, 1)).
        atZone(ZoneId.systemDefault()).toInstant();
Date time = Date.from(instant);
```

此类在计时器类中也非常好地工作以模拟闹钟。

### 7、将日期的格式化的字符串形式转换为 Date 对象

此方法可用于将日期的格式化字符串形式转换为 `Date` 对象。

```java
    /**
     * 使用给定的格式转换日期
     *
     * @param formattedDate 格式化的字符串
     * @param dateFormat    创建string格式使用的日期格式
     * @return 日期对象
     */
    public static Date parseDate(String formattedDate, String dateFormat) {
        Date date = null;
        SimpleDateFormat objDf = new SimpleDateFormat(dateFormat);
        try {
            date = objDf.parse(formattedDate);
        } catch (ParseException e) {
            // Do what ever needs to be done with exception.
        }
        return date;
    }
```

### 8、创建日期对象

```java
Date date = new Date();
System.out.println(date); // Thu Feb 25 05:03:59 IST 2016
```

下面的 `Date` 在创建的时候就包含了指定的日期和时间：

```java
Calendar calendar = Calendar.getInstance();
calendar.set(90, Calendar.DECEMBER, 11);
Date myBirthDate = calendar.getTime();
System.out.println(myBirthDate); // Mon Dec 31 00:00:00 IST 1990
```

`Date` 对象最好通过 `Calendar` 实例创建，并不推荐使用数据构造函数。 为此，我们需要从工厂方法中获取 `Calendar` 类的实例。 然后我们可以通过使用数字或在月份常量的情况下设置年、月和日，通过 `Calendar` 类可以提高可读性并减少错误。

```java
calendar.set(90, Calendar.DECEMBER, 11, 8, 32, 35);
Date myBirthDatenTime = calendar.getTime();
System.out.println(myBirthDatenTime); // Mon Dec 31 08:32:35 IST 1990
```

除了日期，我们还可以按小时、分钟和秒的顺序传递时间。

### 9、比较日期对象

**Calendar, Date, and LocalDate**

版本 < Java SE 8

```java
// Use of Calendar and Date objects
final Date today = new Date();
final Calendar calendar = Calendar.getInstance();
calendar.set(1990, Calendar.NOVEMBER, 1, 0, 0, 0);
Date birthdate = calendar.getTime();
final Calendar calendar2 = Calendar.getInstance();
calendar2.set(1990, Calendar.NOVEMBER, 1, 0, 0, 0);
Date sameBirthdate = calendar2.getTime();

// Before example
System.out.printf("Is %1$tF before %2$tF? %3$b%n", today, birthdate, today.before(birthdate));
System.out.printf("Is %1$tF before %1$tF? %3$b%n", today, today,
        today.before(today));
System.out.printf("Is %2$tF before %1$tF? %3$b%n", today, birthdate, birthdate.before(today));

// After example
System.out.printf("Is %1$tF after %2$tF? %3$b%n", today, birthdate, birthdate.after(today));
System.out.printf("Is %1$tF after %1$tF? %3$b%n", today, birthdate, today.after(today));
System.out.printf("Is %2$tF after %1$tF? %3$b%n", today, birthdate, birthdate.after(today));

// Compare example
System.out.printf("Compare %1$tF to %2$tF: %3$d%n",
        today, birthdate, today.compareTo(birthdate));
System.out.printf("Compare %1$tF to %1$tF: %3$d%n",
        today, birthdate, today.compareTo(today));
System.out.printf("Compare %2$tF to %1$tF: %3$d%n",
        today, birthdate, birthdate.compareTo(today));

// Equal example
System.out.printf("Is %1$tF equal to %2$tF? %3$b%n",
        today, birthdate, today.equals(birthdate));
System.out.printf("Is %1$tF equal to %2$tF? %3$b%n",
        birthdate, sameBirthdate, birthdate.equals(sameBirthdate));
System.out.printf("Because birthdate.getTime() -> %1$d is different from sameBirthdate.getTime() -> %2$d, there are milliseconds! %n",
        birthdate.getTime(), sameBirthdate.getTime());

// Clear ms from calendars
calendar.clear(Calendar.MILLISECOND);
calendar2.clear(Calendar.MILLISECOND);
birthdate = calendar.getTime();
sameBirthdate = calendar2.getTime();
System.out.printf("Is %1$tF equal to %2$tF after clearing ms? %3$b%n",
        birthdate, sameBirthdate, birthdate.equals(sameBirthdate));
```

版本 ≥ Java SE 8

```java
// Use of LocalDate
final LocalDate now = LocalDate.now();
final LocalDate birthdate2 = LocalDate.of(2012, 6, 30);
final LocalDate birthdate3 = LocalDate.of(2012, 6, 30);
// Hours, minutes, second and nanoOfSecond can also be configured with another class LocalDateTime
// LocalDateTime.of(year, month, dayOfMonth, hour, minute, second, nanoOfSecond);

// isBefore example
System.out.printf("Is %1$tF before %2$tF? %3$b%n", now, birthdate2, now.isBefore(birthdate2));
System.out.printf("Is %1$tF before %1$tF? %3$b%n", now, birthdate2, now.isBefore(now));
System.out.printf("Is %2$tF before %1$tF? %3$b%n", now, birthdate2, birthdate2.isBefore(now));

// isAfter example
System.out.printf("Is %1$tF after %2$tF? %3$b%n", now, birthdate2, now.isAfter(birthdate2));
System.out.printf("Is %1$tF after %1$tF? %3$b%n", now, birthdate2, now.isAfter(now));
System.out.printf("Is %2$tF after %1$tF? %3$b%n", now, birthdate2, birthdate2.isAfter(now));

// compareTo example
System.out.printf("Compare %1$tF to %2$tF %3$d%n", now, birthdate2, now.compareTo(birthdate2));
System.out.printf("Compare %1$tF to %1$tF %3$d%n", now, birthdate2, now.compareTo(now));
System.out.printf("Compare %2$tF to %1$tF %3$d%n", now, birthdate2, birthdate2.compareTo(now));

// equals example
System.out.printf("Is %1$tF equal to %2$tF? %3$b%n", now, birthdate2, now.equals(birthdate2));
System.out.printf("Is %1$tF to %2$tF? %3$b%n", birthdate2, birthdate3, birthdate2.equals(birthdate3));

// isEqual example
System.out.printf("Is %1$tF equal to %2$tF? %3$b%n", now, birthdate2, now.isEqual(birthdate2));
System.out.printf("Is %1$tF to %2$tF? %3$b%n", birthdate2, birthdate3, birthdate2.isEqual(birthdate3));
```

**Java 8以前的日期比较**

在 Java 8 之前，可以使用 `java.util.Calendar` 和 `java.util.Date` 类来进行比较。`Date` 类提供类 4 个方法来比较日期：

- [after(Date when)](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html#after(java.util.Date)

- [before(Date when)](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html#before(java.util.Date))

- [compareTo(Date anotherDate)](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html#compareTo(java.util.Date))

- [equals(Object obj)](https://docs.oracle.com/javase/7/docs/api/java/util/Date.html#equals(java.lang.Object)

`after`、`before`、`compareTo` 和 `equals` 方法都是比较每个日期 `getTime` 方法返回的值。

`compareTo` 方法返回整数值：

- 结果比 0 大：日期在待比较日期参数之后

- 结果比 0 小：日期在待比较日期参数之前

- 结果等于 0 ：日期与待比较日期参数相等

如示例中所示，`equals` 结果可能令人惊讶，因为如果未明确给出值，例如毫秒，则不会使用相同的值进行初始化。

**从 Java 8 起**

在 Java 8 中，可以使用 `java.time.LocalDate` 来处理 `Date` 的新对象。 `LocalDate` 实现 `ChronoLocalDate`，这是 `Chronology` 或日历系统可插入的日期的抽象表示。

要获得日期时间精度，必须使用 `Object java.time.LocalDateTime`。 `LocalDate` 和 `LocalDateTime` 使用相同的方法名称进行比较。

使用 LocalDate 比较日期与使用 ChronoLocalDate 不同，因为 `Chronology` 或日历系统不考虑第一个。

因为大多数应用程序应该使用的是 `LocalDate`，`ChronoLocalDate` 不包括在示例中。

> 大多数应用程序应该将方法签名、字段和变量声明为 `LocalDate`，而不是 这个[`ChronoLocalDate`] 接口。

`LocalDate` 有 5 个方法来比较日期：

- `isAfter(ChronoLocalDate other)`

- `isBefore(ChronoLocalDate other)`

- `isEqual(ChronoLocalDate other)`

- `compareTo(ChronoLocalDate other)`

- `equals(Object obj)`

对于 `LocalDate` 参数，`isAfter`、`isBefore`、`isEqual`、`equals` 和 `compareTo` 现在使用此方法：

```java
    int compareTo0(LocalDate otherDate) {
        int cmp = (year - otherDate.year);
        if (cmp == 0) {
            cmp = (month - otherDate.month);
            if (cmp == 0) {
                cmp = (day - otherDate.day);
            }
        }
        return cmp;
    }
```

`equals` 方法首先检查参数引用是否等于日期，而 `isEqual` 直接调用  `compareTo0`。

如果是 `ChronoLocalDate` 的其他类实例，则使用 `Epoch Day` 比较日期。 `Epoch Day` 计数是天数的简单递增计数，其中第 0 天是 1970-01-01 (ISO)。

### 10、将字符串转为 `Date`

`parse` 来自 `SimpleDateFormat` 类，来帮助我们将 `String` 格式转换为 `Date` 对象。

```java
DateFormat dateFormat = DateFormat.getDateInstance(DateFormat.SHORT, Locale.CHINA);
String dateStr = "2022-08-01"; // 输入String
Date date = dateFormat.parse(dateStr);
System.out.println(date); // Mon Aug 01 00:00:00 CST 2022
```

对于文本格式有四种不同的样式，`SHORT`，`MEDIUM`（默认），`LONG` 以及 `FULL`，这些格式都依赖于时区。如果没有指定时区，便会使用系统默认时区。

| 样式     | Locale.CHINA  | Locale.US            |
| ------ | ------------- | -------------------- |
| SHORT  | 22-8-1        | 8/1/22               |
| MEDIUM | 2022-8-1      | Aug 01, 2022         |
| LONG   | 2022年8月1日     | Aug 01, 2022         |
| FULL   | 2022年8月1日 星期一 | Monday, Aug 01, 2022 |

### 11、时区与 `java.util.Date`

一个 `java.util.Date` 对象并没有时区的概念。

- 没有给一个 `Date` **设置**时区的方法

- 没有 **更改** 一个 `Date` 对象时区的方法

- 使用默认构造器 `new Date()` 来创建一个 `Date` 对象，会被初始化为系统默认时区的当前时间

但是，可以使用例如 `java.text.SimpleDateFormat` ，显示在不同时区由 `Date` 对象描述的时间点表示的日期：

```java
Date date = new Date();
// 打印默认时区
System.out.println(TimeZone.getDefault().getDisplayName());
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"); // 注意：时区没有在 format 中设置
// 打印原始时区的日期
System.out.println(sdf.format(date));
// 在伦敦的当前时间
sdf.setTimeZone(TimeZone.getTimeZone("Europe/London"));
System.out.println(sdf.format(date));
```

输出：

```
中国标准时间
2022-08-01 10:50:26
2022-08-01 03:50:26
```

### 

### 链接

- 测试代码： https://github.com/lq920320/java-tests/tree/master/src/test/java/base
