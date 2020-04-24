---
layout: post
title: Class for Time
tags: Java
---
## 引言
时间类在Java中用法多样，使用中容易陷入误区，本文总结了常用的时间和字符串互转、格式化自定义时间、时间增减和改变时区的推荐用法，并特别提出了在多线程环境下的解决方案，最后根据生产经验总结了一些注意事项。

##  使用场景
### 时间转字符串
```
public static String getString(String pattern) {
    Date date = new Date();
    SimpleDateFormat sf = new SimpleDateFormat(pattern);
    return  sf.format(date);
}
```
此为调用系统时间。其中，pattern为输出格式例如"yyyy-MM-dd HH:mm:ss"，字符含义如下：
- yyyy：年，**必须保留年份字段，不要使用YYYY**，如果省略年份字段，则默认年份为1970年
- MM：月
- dd：日
- HH：小时，24小时制
- mm：分钟
- ss：秒

### 字符串转时间
```
public static Date getDate(String pattern, String str) throws ParseException{
    SimpleDateFormat sf = new SimpleDateFormat(pattern);
    return sf.parse(str);
}
```
注意，传入参数str与pattern格式一致，pattern要求与时间转字符串一致

### 格式化自定义时间
```
public static String getCustomTime(String pattern, int year, int month, int date, int hourOfDay, int minute, int second){
    Calendar calendar = Calendar.getInstance();
    calendar.set(year, month, date, hourOfDay, minute, second);
    Date d = calendar.getTime();
    SimpleDateFormat sf = new SimpleDateFormat(pattern);
    return sf.format(d);
}
```
其中，pattern为输出格式；month为月，取值为【0, 11】，0为一月；hourOfDay为24小时制的小时，取值为【0, 23】；minute为分钟，取值为【0, 59】；second为秒，取值为【0, 59】。

### 时间增减
```
public static String getAdjustedTime(String pattern) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(new Date());
    // 月份加1
    calendar.add(Calendar.MONTH, 1);
    Date date = calendar.getTime();
    SimpleDateFormat sf = new SimpleDateFormat(pattern);
    return sf.format(date);
}
```
此为在系统时间上增减外部输入时间。其中，pattern为输出格式。

### 改变时区
```
public static String getTimezoneTime(String pattern, String timezone) {
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(new Date());
    Date date = calendar.getTime();
    SimpleDateFormat sf = new SimpleDateFormat(pattern);
    sf.setTimeZone(TimeZone.getTimeZone(timezone));
    return sf.format(date);
};
```
此为调用系统时间。其中，pattern为输出格式；timezone为“GMT“代表格林威治时间，"GMT+8:00"为北京时间。

## 多线程实现
由于Date、Calendar、SimpleDateFormat类中的方法线程不安全，解决方法如下
### 创建局部变量

需要的时候创建局部变量，将有线程安全问题的对象由共享变为局部私有，举例如下
```
public static  String formatDate(Date date)throws ParseException {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    return sdf.format(date);
}
public static Date parse(String strDate) throws ParseException {
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    return sdf.parse(strDate);
}
```

### ThreadLocal
使用ThreadLocal为每个线程都创建一个线程独享的变量，举例如下
```
private static ThreadLocal<SimpleDateFormat> sdfThreadLocal =  new InheritableThreadLocal<SimpleDateFormat>() {
    @Override
    public SimpleDateFormat initialValue() {
        return  new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");           
    }
};

public static  String format(Date date) throws ParseException {  
    return sdfThreadLocal.get().format(date);
}
public static Date parse(String strDate) throws ParseException {       
    return sdfThreadLocal.get().parse(strDate);
}
```
### 使用Java8支持的新类(不可变类)

1. 字符串与时间互转
```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime localDateTime = LocalDateTime.now();
// 时间转字符串
System.out.println(formatter.format(localDateTime));
// 字符串转时间
LocalDateTime localDateTime1 = LocalDateTime.parse("2020-02-29 10:12:05",formatter);
```

2. 格式化自定义时间
```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
// 设置自定义时间
LocalDateTime localDateTime = LocalDateTime.of(2020, 2,29,18,36,30);
System.out.println(formatter.format(localDateTime));
```

3. 时间增减
```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime localDateTime = LocalDateTime.now();
// 增加一天
LocalDateTime localDateTime1 = localDateTime.plusDays(1);
// 减去一天
// LocalDateTime localDateTime1 = localDateTime.minusDays(1);
System.out.println(formatter.format(localDateTime1));
```

4. 改变时区
```
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
// 设置时区为格林威治中央区时
LocalDateTime localDateTime = LocalDateTime.now((ZoneId.of("GMT")));
System.out.println(formatter.format(localDateTime));
```

## 注意事项
### 不能忽略的yyyy
SimpleDateFormat和DateTimeFormatter类不论在什么Java版本中年份**不可省略yyyy**。省略后默认年份为1970年，闰年时会受到影响。测试如下
```
public static void test() throws ParseException{
    String string = "02-29 13:00:21";
    SimpleDateFormat sf = new SimpleDateFormat("MM-dd HH:mm:ss");
    System.out.println(sf.format(sf.parse(string)));
    SimpleDateFormat sf1 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    System.out.println(sf1.format(sf.parse(string)));
}
```
输出为
```
03-01 13:00:21
1970-03-01 13:00:21
```

### 不使用YYYY
SimpleDateFormat和DateTimeFormatter类不论在什么Java版本中年份**不能使用YYYY**。y 是Year,  Y 表示的是Week year，Week year 意思是当天所在的周属于的年份，一周从周日开始，周六结束，只要本周跨年，那么这周就算入下一年。测试如下
```
public static void test() throws ParseException{
    Calendar calendar = Calendar.getInstance();
    calendar.set(2019, Calendar.DECEMBER, 31, 15, 10, 2);
    Date date = calendar.getTime();
    // 错误
    SimpleDateFormat sf1 = new SimpleDateFormat("YYYY-MM-dd HH:mm:ss");
    System.out.println(sf1.format(date));
    // 正确
    SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    System.out.println(sf.format(date));
}
```
输出为
```
2020-12-31 15:10:02
2019-12-31 15:10:02
```

### 注意一天的结束时间
一天的结束时间请勿使用23:59:59，会存在1秒的间隙。由于Date类的精度为毫秒，应该具体到毫秒，举例为获取当天的结束时间
```
public static Date atEndOfDay() {
    Calendar calendar = Calendar.getInstance();
    calendar.setTime(new Date());
    calendar.set(Calendar.HOUR_OF_DAY, 23);
    calendar.set(Calendar.MINUTE, 59);
    calendar.set(Calendar.SECOND, 59);
    calendar.set(Calendar.MILLISECOND, 999);
    return calendar.getTime();
}
```

### 注意时区设置
注意代码和应用的时区设置。
- 如使用**Jackson**进行日期字段反序列化时，Jackson默认情况下会将时区设置为UTC（Coordinated UniversalTime 世界统一时间），JVM中的时区会根据操作系统来获取，由于操作系统的时区为GMT+8 时区，会将 UTC 时区（GMT）的时间转成 GMT+8 时区的时间，导致最终转换的时间会比实际多8个小时。该问题可通过在字段的@JsonFormat 加上属性 timezone = "GMT+8”解决。
```
@NotNull
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss",timezone = "GMT+8")
@JsonProperty("start_time")
private Date startTime;
```
- **Apache(PHP)**的服务器时间时区默认为UTC。需要修改php.ini中的date.timezone配置项，修改为Asia/ShangHai。**Nginx**不涉及时区问题。
- **MySQL**默认时区为UTC；**UPSQL**默认时区为GMT+8时区（东8区）；使用**mysqldump**导出数据时，需配置--skip-tz-utc，以保证导出数据中时间类型（TIMESTAMP）数据与数据库中一致。

### 小心Calendar类的HOUR
Calendar类Calendar.HOUR和Calendar.HOUR_OF_DAY的区别。Calendar.HOUR是按照12小时制处理的；Calendar.HOUR_OF_DAY是按照24小时制处理的。若按照12小时制设置时间，必须添加Calendar.AM_PM字段，否则会产生与预期不符的情况。示例代码如下
```
/**
 * 12小时制和24小时制期望输出为上午9点，输出格式均为24小时制
 */
public static void test() {
    SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    Calendar calendar = Calendar.getInstance();
    calendar.set(2020, Calendar.MARCH, 3);
    calendar.set(Calendar.MINUTE, 0);
    calendar.set(Calendar.SECOND, 0);
    // 12小时制错误设置方法
    calendar.set(Calendar.HOUR, 9);
    Date date = calendar.getTime();
    System.out.println("12小时制错误设置方法输出：" + sf.format(date));
    // 12小时制正确设置方法
    calendar.set(Calendar.HOUR, 9);
    calendar.set(Calendar.AM_PM, Calendar.AM);
    date = calendar.getTime();
    System.out.println("12小时制正确设置方法输出：" + sf.format(date));
    // 24小时制设置方法
    calendar.set(Calendar.HOUR_OF_DAY, 9);
    date = calendar.getTime();
    System.out.println("24小时制设置方法输出：" + sf.format(date));
}
```
输出如下
```
12小时制错误设置方法输出：2020-03-03 21:00:00
12小时制正确设置方法输出：2020-03-03 09:00:00
24小时制设置方法输出：2020-03-03 09:00:00
```
### 涉及日期务必考虑闰年
获取某年天数，不可以默认为365，需要考虑闰年。给出获取某年天数的代码如下
```
 public static int getNumOfDays(int year) {
    Calendar calendar = Calendar.getInstance();
    calendar.set(year, Calendar.MARCH, 1);
    calendar.add(Calendar.DAY_OF_MONTH, -1);
    if(calendar.get(Calendar.DAY_OF_MONTH) == 29) {
        return 366;
    }
    else {
        return 365;
    }
}
```
