---
title: jdk8时间日期
tags:
  - jdk8
  - date
  - time
categories:
  - JDK
  - Date
date: 2018-12-24 08:23:57
---

JDK8之前的日期api有很多不便的地方, 如: 
- `java.util.Date` 和 `java.util.Calendar`类易用性差, 不支持时区, 而且他们都不是线程安全的
- 用于格式化日期的类DateFormat被放在java.text包中, 它是一个抽象类, 在处理日期格式化时我们会实例化一个`SimpleDateFormat`对象, 但DateFormat也是非线程安全
- 对日期的计算方式繁琐, 而且容易出错, 因为月份是从0开始的, 从Calendar中获取的月份需要加一才能表示当前月份

由于以上这些问题, 出现了一些三方的日期处理框架, 例如Joda-Time, date4j等开源项目. 其中Joda-Time框架的作者正是JSR-310的规范的倡导者, 而Java 8中引入了新的日期API是JSR-310规范的实现, 所以能从Java 8的日期API中看到很多Joda-Time的特性.
`java.time`包下有5个包组成, 大部分情况下只用基础包和format就够了
- `java.time`: 包含值对象的基础包
- `java.time.chrono`: 提供对不同的日历系统的访问
- `java.time.format`: 格式化和解析时间和日期
- `java.time.temporal`: 包括底层框架和扩展特性
- `java.time.zone`: 包含时区支持的类

**所有类都是不可变的, 线程安全的**


## 日期/时间类
这些类的方法具有统一的前缀, 其含义如下:

| 前缀 | 含义 | 示例 |
| :-: | :-: | :-: |
| now | 静态工厂方法, 用当前时间创建实例 | `LocalDate.now();` |
| of | 静态工厂方法 | `LocalDate.of(2018, 12, 20);` |
| parse | 静态工厂方法, 关注于解析 | `LocalDate.parse("2018-12-20");` |
| get | 获取某个字段的值 | `localDate.getYear();` |
| is | 比对判断 | `localDate.isAfter(LocalDate.now());` |
| with | 基于当前实例创建新的实例, 但部分字段被更新 | `localDate.withMonth(3);` |
| plus | 在当前实例基础上增加(值可负), 返回新实例 | `localDate.plusDays(1);` |
| minus | 在当前实例基础上减小(值可负), 返回新实例 | `localDate.minusDays(1);` |
| to | 基于当前实例转换出另一个类型的实例 | `localDateTime.toLocalDate();` |
| at | 把当前对象和另一个对象结合, 生成新的类型的实例 | `localDate.atTime(21, 30, 50)` |
| format | 格式化 | `localDate.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));` |

### LocalDate和LocalTime
`LocalDate`表示日期(年月日), 但不包含时间(时分秒), 也**不包时区信息**, 内部使用3个属性`(int: year, short: month, short: day)`记录时间信息. 

```java
/* 创建 */
LocalDate localDate = LocalDate.now();
LocalDate.now(Clock.systemDefaultZone());   // 指定时钟
LocalDate.now(ZoneId.systemDefault());      // 获取某个时区下当前日期
// 统一时刻, 中国时间与夏威夷时间
LocalDate.now(ZoneId.systemDefault()); 		  // 中国: 2020-04-22
LocalDate.now(ZoneId.of("UTC-10"));         // 夏威夷: 2020-04-21

LocalDate.of(2018, 12, 20);                 // 年月日
LocalDate.of(2018, Month.DECEMBER, 20);
LocalDate.ofYearDay(2018, 265);             // 根据年和在本年的天数创建, 2018年第265天
LocalDate.ofEpochDay(17888);                // 自1970-01-01以来的天数

LocalDate.parse("2018-12-20");              // 解析字符串
LocalDate.parse("2018/12/20", DateTimeFormatter.ofPattern("yyyy/MM/dd"));

/* 基于LocalDate创建LocalDateTime */
LocalDateTime localDateTime = localDate.atStartOfDay();
localDateTime = localDate.atTime(21, 30);            // 补充 时分
localDateTime = localDate.atTime(21, 30, 50);        // 补充 时分秒
localDateTime = localDate.atTime(21, 30, 50, 123000123);     // 时分秒纳秒
localDateTime = localDate.atTime(LocalTime.now());
ZonedDateTime zonedDateTime = localDate.atStartOfDay(ZoneId.systemDefault());
OffsetDateTime offsetDateTime = localDate.atTime(OffsetTime.now());

/* 格式化 */
localDate.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));     // 2018-10-23
// 预定义格式 在DateTimeFormatter里
localDate.format(DateTimeFormatter.ISO_DATE);                    // 2018-10-23
localDate.format(DateTimeFormatter.BASIC_ISO_DATE);              // 20181023
// 2018年 12月 23日 星期日
localDate.format(DateTimeFormatter.ofPattern("yyyy年 MM月 dd日 E", Locale.CHINESE));

/* 获取某个时间字段 */
localDate.getYear();
localDate.getMonthValue();
Month month = localDate.getMonth();
localDate.getDayOfMonth();
localDate.getDayOfYear();
DayOfWeek dayOfWeek = localDate.getDayOfWeek();
localDate.getLong(ChronoField.DAY_OF_WEEK);          // 获取指定时间字段

localDate.lengthOfMonth();       // 这个月的总天数
localDate.lengthOfYear();        // 这一年的总天数
localDate.range(ChronoField.MONTH_OF_YEAR);          // 获取某个字段的合法范围

/* 时间比对 */
localDate.isAfter(LocalDate.now());
localDate.isBefore(MinguoDate.now());        // 与民国历法比对, 2018是民国107年
localDate.isLeapYear();                      // 是否是闰年
localDate.isSupported(ChronoUnit.NANOS);     // 是否有指定的时间字段

/* 时间操作, 值可以为负, 返回新的对象 */
localDate.minus(1, ChronoUnit.DAYS);         // 减去不支持的时间字段(时分秒)会抛异常
localDate.minus(Period.of(1, 2, 3));         // 减去1年2个月3天, 基于Period
// 基于Duration会抛异常, 其内部是转化成了秒, LocalDate不支持秒
// date.minus(Duration.ofDays(2));      
localDate.minusDays(1);
localDate.minusMonths(-2);
localDate.minusYears(3);
localDate.minusWeeks(-4);
localDate.plus(Period.of(1, 2, 3));
localDate.plusDays(-1);
localDate.plusMonths(2);
localDate.plusYears(-3);
localDate.plusWeeks(4);

localDate.with(ChronoField.DAY_OF_WEEK, 4);      // 复制新的实例, 指定部分被更新
localDate.with(MinguoDate.now());                // 复制新的实例, 数据被调整
localDate.withDayOfYear(330);                    // 复制新的实例, 天部分被更新
localDate.withDayOfMonth(27);                    // 复制新的实例, 天部分被更新
localDate.withMonth(3);                          // 复制新的实例, 月部分被更新
localDate.withYear(2008);                        // 复制新的实例, 年部分被更新

/* 时间差计算 */
// 计算到指定时间之间的差值, 指定时间在这之前返回负值, 
// 表示已经过去xx时间, 正值表示还有xx时间才到来
Period period = localDate.until(LocalDate.now());  

period.get(ChronoUnit.DAYS) # 同下
period.getDays()    # 返回的是相差的天的部分, 而不是总天数的意思, 可以用下面的方式获取总天数

// 距离民国元年1月1日有多少天
long amount = localDate.until(MinguoDate.of(0, 1, 1), ChronoUnit.DAYS);
// 或者
ChronoUnit.DAYS.between(localDate, localDate2)

/* 自定义访问方法 */
localDate.query(new TemporalQuery<String>() {
    @Override
    public String queryFrom(TemporalAccessor temporal) {
        LocalDate date1 = (LocalDate) temporal;
        return null;
    }
});
```
`LocalTime`与`LocalDate`类似, 大部分接口名都相同, 包含纳秒信息, **不包时区信息**, 内部使用`(byte: hour, byte: minute, byte: second, int: nano)`记录时间信息

```java
LocalTime localTime = LocalTime.now();
LocalTime.now(Clock.systemDefaultZone());           // SystemClock[Asia/Shanghai]
LocalTime.now(ZoneId.systemDefault());              // Asia/Shanghai

LocalTime.of(21, 36);       // 小时(0-23) + 分钟
LocalTime.of(21, 30, 0);     //
LocalTime.of(21, 30, 0, 120);   // 小时 + 分钟 + 秒 + 纳秒(0 - 999,999,999)
LocalTime.ofSecondOfDay(120);
LocalTime.ofNanoOfDay(120);

// 最后一位值纳秒, 最短为: 10:01, 必须两位, 用零补齐
LocalTime.parse("10:15:23.829890898"); 
LocalTime.parse("10.15.23.898989", new DateTimeFormatterBuilder().appendPattern("HH.mm.ss")
        // 可以有9位的纳秒表示(包含了点符号)
        .optionalStart().appendFraction(ChronoField.NANO_OF_SECOND, 9, 9, true).optionalEnd()
        // 可以有6位的纳秒表示(包含了点符号)
        .optionalStart().appendFraction(ChronoField.NANO_OF_SECOND, 6, 6, true).optionalEnd()
        // 可以有3位的纳秒表示(包含了点符号)
        .optionalStart().appendFraction(ChronoField.NANO_OF_SECOND, 3, 3, true).optionalEnd()
        .toFormatter());

LocalDateTime localDateTime = localTime.atDate(LocalDate.now());
localTime.adjustInto(localDateTime.now());    // 调整指定对象的时间和当前对象一致
localTime.getHour();
localTime.getMinute();
localTime.getSecond();
localTime.getNano();
localTime.getLong(ChronoField.MINUTE_OF_HOUR);

// 从某个字段开始截断, 截断分钟则秒, 纳秒会被清零
localTime.truncatedTo(ChronoUnit.MINUTES);        

// offsetTime表示具有相对于UTC的固定区偏移的时间
OffsetTime offsetTime = localTime.atOffset(ZoneOffset.ofHours(8));
```

### LocalDateTime
`LocalDateTime`内部用两个属性: `LocalDate`和`LocalTime`记录时间信息, **不包含时区信息**

```java
/* 创建 */
LocalDateTime localDateTime = LocalDateTime.now();
LocalDateTime.of(LocalDate.now(), LocalTime.now());
LocalDateTime.of(2018, 12, 23, 16, 58, 47);
LocalDateTime.parse("2007-12-03T10:15:30");

LocalDate.now().atTime(LocalTime.now());
LocalTime.now().atDate(LocalDate.now());
```

### Instant
`Instant`表示一个UTC时间戳(从1970-01-01T00:00:00Z开始),**不包含时区信息** , 相比`System.currentTimeMillis()`, 它可以精确到纳秒. 其内部有两个常量, `seconds`表示从1970-01-01 00:00:00开始到现在的秒数, `nanos`表示纳秒部分(不超过999,999,999). **可以替换`Date`使用**

```java
/* 创建 */
Instant now = Instant.now();
Instant.ofEpochMilli(333L);     // 从1970-01-01T00:00:00Z开始到现在的毫秒数
Instant.ofEpochMilli(new Date().getTime());
Instant.ofEpochSecond(22L);     // 从1970-01-01T00:00:00Z开始到现在的秒数
Instant.ofEpochSecond(44L, 6L); // 从1970-01-01T00:00:00Z开始到现在的秒数 + 纳秒

now.getEpochSecond();       // 获取秒部分
now.getNano();              // 获取纳秒部分

// 增加5小时4分钟, 因为是不可变对象, 所以会产生新的对象
Instant after = now.plus(Duration.ofHours(5).plusMinutes(4));
// 等同于此, 但会多产生一个对象
after = now.plus(5, ChronoUnit.HOURS).plus(4, ChronoUnit.MINUTES);

// 时间比对, 可读性更强
now.isBefore(after); // true

// 计算差异(5*60+4=304), 用不同单位表示
after.until(now, ChronoUnit.MINUTES); // -304
ChronoUnit.MINUTES.between(after, now); // -304
// 等同于
now.until(after, ChronoUnit.MINUTES); // 304
ChronoUnit.MINUTES.between(now, after); // 304
```
### Duration
`Duration`用来表示一小段时间(时分秒), 其内部和`Instant`类似也有`seconds`和`nanos`

```java
/* 创建 */
Duration duration = Duration.between(LocalTime.now(), LocalTime.now());
Duration.ofNanos(888L);
Duration.ofSeconds(123L);       
Duration.ofMinutes(5L);
Duration.ofHours(1L);
Duration.ofDays(1);
Duration.of(3, ChronoUnit.YEARS);
Duration.from(Period.ofDays(1));

// PnDTnHnMn.nS  
// PT20.345S: 20.345秒 P2DT3H4M: 2天3小时4分钟 
// P-6H3M: -6小时+3分钟  -P6H3M: -6小时-3分钟
Duration.parse("PT15M");    // 15分钟

// 这段时间转成秒/分钟/小时...的长度
duration.toNanozs();
duration.toMillis();
duration.toMinutes();
duration.toHours();
duration.toDays();

duration.dividedBy(4);      // 时间段均分
duration.negated();         // 转负值
duration.abs();             // 转正值
```

`Period`用来表示较长的时间(年月日)

```java
/* 创建 */
Period period = Period.of(1, 3, 28);    //年 月 日
Period.ofDays(3);
Period.ofWeeks(2);
Period.ofMonths(10);
Period.ofYears(4);
Period.between(LocalDate.now(), LocalDate.of(2018, 1, 1));

Period of = Period.of(2, -13, 400);
System.out.println(of.getYears());       // 2
System.out.println(of.getMonths());      // -13
System.out.println(of.getDays());        // 400

Period normalized = of.normalized();    // 归一化, 只处理年和月部分
System.out.println(normalized.getYears());    // 0
System.out.println(normalized.getMonths());   // 11
System.out.println(normalized.getDays());     // 400
```

## 日期调整与格式化
### 加减调整
jdk8中时间日期对象都是不可变的, 因此在调整时, 总是会返回新的实例. 调整方法主要有`plus`, `minus` 和 `with`

```java
LocalDate date = LocalDate.of(2018, 12, 5);          // 2018-12-05

LocalDate date1 = date.withYear(2016);              // 修改为 2016-12-05
LocalDate date2 = date.withMonth(2);                // 修改为 2018-02-05
LocalDate date3 = date.withDayOfMonth(1);           // 修改为 2018-12-01

LocalDate date4 = date.plusYears(1);                // 增加一年 2019-12-05
LocalDate date5 = date.minusMonths(2);              // 减少两个月 2018-10-05
LocalDate date6 = date.plus(5, ChronoUnit.DAYS);    // 增加5天 2018-12-10
```
在进行复杂的操作时, 比如下一个工作日, 下个月的第一天时, 可以使用`with`的重载方法, 接受一个`TemporalAdjuster`参数.

```java
// 返回下一个距离当前时间最近的星期日
LocalDate date7 = date.with(TemporalAdjusters.nextOrSame(DayOfWeek.SUNDAY));
// 返回本月第一个星期六
LocalDate date8 = date.with(TemporalAdjusters.firstInMonth(DayOfWeek.SATURDAY));   
// 返回本月最后一个星期六
LocalDate date9 = date.with(TemporalAdjusters.lastInMonth(DayOfWeek.SATURDAY));   
```

### java.time.temporal.ChronoField枚举类
此枚举类是作为get方法的参数获取时间的某个字段(年/月/日/时/分/秒...)值, 它里面的属性含义有的跟Calendar的成员变量含义差不多

```java
localDate.with(ChronoField.DAY_OF_WEEK, 4); 
localDate.getLong(ChronoField.DAY_OF_WEEK);

localDate.range(ChronoField.MONTH_OF_YEAR);

LocalDateTime now = LocalDateTime.now();
switch (now.get(ChronoField.AMPM_OF_DAY)) {
case 0:
    System.out.println("上午");
case 1:
    System.out.println("下午");
}
//打印 下午
```
### java.time.temporal.ChronoUnit枚举类
此枚举表示时间单位

```java
localDate.minus(1, ChronoUnit.DAYS);
localDate.isSupported(ChronoUnit.NANOS);  

localTime.truncatedTo(ChronoUnit.MINUTES);

Duration.of(3, ChronoUnit.YEARS);

// 今天距离民国元年1月1日有多少天
long amount = LocalDate.now().until(MinguoDate.of(0, 1, 1), ChronoUnit.DAYS);   
```

### java.time.temporal.TemporalAdjusters类
`TemporalAdjusters`主要配合时间日期的`with`方法使用, 用来对时间日期进行调整, 中有很多静态方法可以使用

| 方法名 | 描述 |
| :-: | :-: |
| `dayOfWeekInMonth` | 返回同一个月中每周的第几天 |
| `firstDayOfMonth` | 返回当月的第一天 |
| `firstDayOfNextMonth` | 返回下月的第一天 |
| `firstDayOfNextYear` | 返回下一年的第一天 |
| `firstDayOfYear` | 返回本年的第一天 |
| `firstInMonth` | 返回同一个月中第一个星期几 |
| `lastDayOfMonth` | 返回当月的最后一天 |
| `lastDayOfNextMonth` | 返回下月的最后一天 |
| `lastDayOfNextYear` | 返回下一年的最后一天 |
| `lastDayOfYear` | 返回本年的最后一天 |
| `lastInMonth` | 返回同一个月中最后一个星期几 |
| `next` / `previous` | 返回后一个/前一个给定的星期几 |
| `nextOrSame` / `previousOrSame` | 返回后一个/前一个给定的星期几，如果这个值满足条件，直接返回 |
还可以通过创建自定义的`TemporalAdjuster`实现来实现更复杂的逻辑, `TemporalAdjuster`是函数式接口, 所有可以使用Lambda表达式. 比如给定一个日期，计算该日期的下一个工作日（不包括星期六和星期天）：

```java
LocalDate date = LocalDate.of(2017, 1, 5);
date.with(temporal -> {
    // 当前日期
    DayOfWeek dayOfWeek = DayOfWeek.of(temporal.get(ChronoField.DAY_OF_WEEK));

    // 正常情况下，每次增加一天
    int dayToAdd = 1;

    // 如果是星期五，增加三天
    if (dayOfWeek == DayOfWeek.FRIDAY) {
        dayToAdd = 3;
    }

    // 如果是星期六，增加两天
    if (dayOfWeek == DayOfWeek.SATURDAY) {
        dayToAdd = 2;
    }

    return temporal.plus(dayToAdd, ChronoUnit.DAYS);
});
```

### 格式化
JDK8中提供了一个新的类`java.time.format.DateTimeFormatter`
来处理格式化操作. 日期类中有一个`format`方法, 该方法接收一个`DateTimeFormatter`类型的参数.

```java
LocalDateTime dateTime = LocalDateTime.now();
dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));   // 2018-12-24
dateTime.fomat(DateTimeFormatter.ISO_LOCAL_DATE);    // 2018-12-24

String dateStr = "2018-12-24";
String dateTimeStr = "2018-12-24 12:30:05";
LocalDate date = LocalDate.parse(dateStr, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
LocalDateTime localDateTime = LocalDateTime.parse(dateTimeStr, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```

其中`DateTimeFormatter`中预定义了许多格式

| 变量 | 输出示例 |
| :-: | :-: |
| BASIC_ISO_DATE | 20111203 |
| ISO_LOCAL_DATE | 2011-12-03 |
| ISO_OFFSET_DATE | 2011-12-03+08:00 |
| ISO_DATE | 2011-12-03 或 2011-12-03+08:00 |
| ISO_LOCAL_TIME | 10:15 或 10:15:30 |
| ISO_OFFSET_TIME | 10:15+01:00 或 10:15:30+08:00 |
| ISO_TIME | 10:15, 10:15:30 或 10:15:30+08:00 |
| ISO_LOCAL_DATE_TIME | 2011-12-03T10:15:30 |
| ISO_OFFSET_DATE_TIME | 2011-12-03T10:15:30+08:00 |
| ISO_ZONED_DATE_TIME | 2011-12-03T10:15:30+08:00[Asia/Shanghai] |
| ISO_DATE_TIME | 2011-12-03T10:15:30, 2011-12-03T10:15:30+08:00 或 2011-12-03T10:15:30+08:00[Asia/Shanghai] |
| ISO_ORDINAL_DATE | 2012-337 |
| ISO_WEEK_DATE | 2012-W48-6 |
| ISO_INSTANT | 2011-12-03T10:15:30Z |
| RFC_1123_DATE_TIME | Tue, 3 Jun 2008 11:05:30 GMT |

还可以自定义格式

```java
DateTimeFormatter formatter = new DateTimeFormatterBuilder().appendPattern("HH.mm.ss")
                // 可以有9位的纳秒表示(包含了点符号)
                .optionalStart().appendFraction(ChronoField.NANO_OF_SECOND, 9, 9, true).optionalEnd()
                // 可以有6位的纳秒表示(包含了点符号)
                .optionalStart().appendFraction(ChronoField.NANO_OF_SECOND, 6, 6, true).optionalEnd()
                // 可以有3位的纳秒表示(包含了点符号)
                .optionalStart().appendFraction(ChronoField.NANO_OF_SECOND, 3, 3, true).optionalEnd()
                .toFormatter();
LocalTime.parse("10.15.23", formatter);
LocalTime.parse("10.15.23.899", formatter);
LocalTime.parse("10.15.23.898989", formatter);
LocalTime.parse("10.15.23.889778667", formatter);
```

模板字段含义如下
> G 年代标志符
y 年
M 月
d 日
h 时 (12小时制)
H 时 (24小时制)
m 分
s 秒
S 毫秒
E 星期几
D 一年中的第几天
F 一月中第几个星期(以每个月1号为第一周,8号为第二周为标准计算)
w 一年中第几个星期
W 一月中第几个星期(不同于F的计算标准,是以星期为标准计算星期数,例如1号是星期三,是当月的第一周,那么5号为星期日就已经是当月的第二周了)
a 上午 / 下午 标记符
k 时 (24小时制,其值与H的不同点在于,当数值小于10时,前面不会有0)
K 时 (12小时值,其值与h的不同点在于,当数值小于10时,前面不会有0)
z 时区

## 相互转换

#### 1.LocalDate转Date

```java
LocalDate nowLocalDate = LocalDate.now();
Date date = Date.from(localDate.atStartOfDay(ZoneOffset.ofHours(8)).toInstant());
```

#### 2.LocalDateTime转Date

```java
LocalDateTime localDateTime = LocalDateTime.now();
Date date = Date.from(localDateTime.atZone(ZoneOffset.ofHours(8)).toInstant());
```

#### 3.Date转LocalDateTime(LocalDate)

```java
Date date = new Date();
LocalDateTime localDateTime = date.toInstant().atZone(ZoneOffset.ofHours(8)).toLocalDateTime();
LocalDate localDate = date.toInstant().atZone(ZoneOffset.ofHours(8)).toLocalDate();
```

#### 4.LocalDate转时间戳

```java
LocalDate localDate = LocalDate.now();
long timestamp = localDate.atStartOfDay(ZoneOffset.ofHours(8)).toInstant().toEpochMilli();
```

#### 5.LocalDateTime转时间戳

```java
LocalDateTime localDateTime = LocalDateTime.now();
long timestamp = localDateTime.toInstant(ZoneOffset.ofHours(8)).toEpochMilli();
```

#### 6.时间戳转LocalDateTime(LocalDate)

```java
long timestamp = System.currentTimeMillis();
LocalDate localDate = Instant.ofEpochMilli(timestamp).atZone(ZoneOffset.ofHours(8)).toLocalDate();
LocalDateTime localDateTime = Instant.ofEpochMilli(timestamp).atZone(ZoneOffset.ofHours(8)).toLocalDateTime();
```



## 时区

jdk8中使用新的时区类`java.time.ZoneId`来替代原来的`java.util.TimeZone`, 对应的时间类是`ZonedDateTime`. 使用方式如下:

```java
// 创建
ZoneId shanghaiZoneId = ZoneId.of("Asia/Shanghai");
// ZoneId.of("UTC+8");
ZoneId systemZoneId = ZoneId.systemDefault();
ZoneId oldToNewZoneId = TimeZone.getDefault().toZoneId();

// 获取所有合法的"“区域/城市"字符串
Set<String> zoneIds = ZoneId.getAvailableZoneIds();  

// LocalDate/LocalTime/LocalDateTime -> ZonedDateTime
LocalDateTime localDateTime = LocalDateTime.now();
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, shanghaiZoneId);

// 输出格式 2018-12-24T11:43:37.183+08:00[Asia/Shanghai]
ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, shanghaiZoneId);
```
另一种表示时区的方式是使用`ZoneOffset`, 它是以当前时间和**世界标准时间(UTC)/格林威治时间(GMT)**的偏差来表示, 对应的类是`OffsetDateTime`.

```java
ZoneOffset zoneOffset = ZoneOffset.of("+08:00");
// ZoneOffset.of("+8");
// ZoneOffset.ofHours(-10);
LocalDateTime localDateTime = LocalDateTime.now();
// 2018-12-24T11:47:36.897+08:00
OffsetDateTime offsetDateTime = OffsetDateTime.of(localDateTime, zoneOffset);
```

## 其他历法
Java中使用的历法是ISO 8601日历系统, 它是世界民用历法, 也就是我们所说的公历. 平年有365天, 闰年是366天. 闰年的定义是: 非世纪年, 能被4整除; 世纪年能被400整除. 为了计算的一致性, 公元1年的前一年被当做公元0年, 以此类推.

此外Java 8还提供了4套其他历法(但是没有农历), 每套历法都包含一个日期类, 分别是:
- `ThaiBuddhistDate`：泰国佛教历
- `MinguoDate`：中华民国历
- `JapaneseDate`：日本历
- `HijrahDate`：伊斯兰历

它们都继承了`ChronoLocalDate`类, 但开发中要避免直接使用`ChronoLocalDate`类. 
```java
MinguoDate minguoDate = MinguoDate.from(LocalDate.now());
LocalDate localDate = LocalDate.from(MinguoDate.now());
```


**参考**
[java 8的java.time包(非常值得推荐)](https://www.jianshu.com/p/19bd58b30660)
[Java 8新特性（四）：新的时间和日期API](https://lw900925.github.io/java/java8-newtime-api.html)
