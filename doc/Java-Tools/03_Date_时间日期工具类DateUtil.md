# 时间日期工具类DateUtils

[toc]



## 推荐阅读





## 一、常见需求

### 1. LocalDate、LocalDateTime 与 Date、Timestamp互转

#### 1.1 LocalDate、LocalDateTime 与 Date

（1）LocalDate 转 Date

```java
        // LocalDate 转为 ZonedDateTime
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDate localDate = LocalDate.now();
        ZonedDateTime zdt = localDate.atStartOfDay().atZone(zoneId);

        // ZonedDateTime 转为 Instant
        Instant instant = zdt.toInstant();

        // instant 转为 Date
        Date date = Date.from(instant);
```

也即：

```java
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDate localDate = LocalDate.now();
        Date date = Date.from(localDate.atStartOfDay().atZone(zoneId).toInstant());
```



（2）  LocalDateTime 转 Date

```java
        // LocalDateTime 转为 ZonedDateTime
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDateTime localDateTime = LocalDateTime.now();
        ZonedDateTime zdt = localDateTime.atZone(zoneId);

        // ZonedDateTime 转为 Instant
        Instant instant = zdt.toInstant();

        // instant 转为 Date
        Date date = Date.from(instant);
```



（3）Date 转 LocalDate 

```java
  		// Date 转 Instant
        Date date = new Date();
        Instant instant = date.toInstant();

        // Instant 转 ZonedDateTime
        ZoneId zoneId = ZoneId.systemDefault();
        ZonedDateTime zdt = instant.atZone(zoneId);

        // ZonedDateTime 转 LocalDate
        LocalDate localDate = zdt.toLocalDate();
```

也即

```java
        ZoneId zoneId = ZoneId.systemDefault();
        Date date = new Date();
        LocalDate localDate = date.toInstant().atZone(zoneId).toLocalDate();
```



（4） Date 转 LocalDateTime

```java
        // Date 转 Instant
        Date date = new Date();
        Instant instant = date.toInstant();

        // Instant 转 ZonedDateTime
        ZoneId zoneId = ZoneId.systemDefault();
        ZonedDateTime zdt = instant.atZone(zoneId);

        // ZonedDateTime 转 LocalDateTime
        LocalDateTime localDateTime = zdt.toLocalDateTime();
```



#### 1.2  LocalDate、LocalDateTime 转 Timestamp

（1） LocalDate 转 Timestamp

```java
        // LocalDate 转 LocalDateTime
        LocalDateTime localDateTime = localDate.atStartOfDay();
        
        // LocalDateTime 转 Timestamp
        Timestamp timestamp = Timestamp.valueOf(localDateTime);
```



（2） LocalDateTime 转 Timestamp

```java
        // LocalDateTime 转 Timestamp
        Timestamp timestamp = Timestamp.valueOf(localDateTime);
```



（3）Timestamp 转 LocalDate

```java
        Timestamp timestamp = Timestamp.from(Instant.now());
        LocalDateTime localDateTime = timestamp.toLocalDateTime();
        LocalDate localDate = localDateTime.toLocalDate();
```



（3）Timestamp 转 LocalDateTime 

```java
        Timestamp timestamp = Timestamp.from(Instant.now());
        LocalDateTime localDateTime = timestamp.toLocalDateTime();
```



### 2.计算时间间隔

（1）使用 Period，可计算的时间间隔单位为 `day / month / year`

```java
        LocalDate startDateInclusive = LocalDate.now();
        LocalDate endDateExclusive =  LocalDate.now().plusDays(14);
        Period period = Period.between(startDateInclusive, endDateExclusive);

        int durationDays = period.getDays();
        int durationMonths = period.getMonths();
        int durationYears = period.getYears();
```



（2）使用 `ChronoUnit`

```java
        LocalDateTime startTime = LocalDate.now();
        LocalDateTime endTime =  LocalDate.now().plusDays(14);
        double intervalHours = ChronoUnit.HOURS.between(startTime, endTime);
```





## 二、工具类

```java
public class DateUtils {
    private static final ZoneId ZONE_ID = ZoneId.systemDefault();


    public static Date toDate(LocalDate localDate) {
        return Date.from(localDate.atStartOfDay().atZone(ZONE_ID).toInstant());
    }

    public static Date toDate(LocalDateTime localDateTime) {
        return Date.from(localDateTime.atZone(ZONE_ID).toInstant());
    }

    public static Timestamp toTimestamp(LocalDate localDate) {
        return Timestamp.valueOf(localDate.atStartOfDay());
    }

    public static Timestamp toTimestamp(LocalDateTime localDateTime) {
        return Timestamp.valueOf(localDateTime);
    }

    public static LocalDate toLocalDate(Date date) {
        return Instant.ofEpochMilli(date.getTime()).atZone(ZONE_ID).toLocalDate();
    }

    public static LocalDateTime toLocalDateTime(Date date) {
        return Instant.ofEpochMilli(date.getTime()).atZone(ZONE_ID).toLocalDateTime();
    }

}

```













