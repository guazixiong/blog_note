# Java日期和时间处理

在Java中处理日期和时间时，主要涉及三个类：`Date`，`LocalDate`，和`LocalDateTime`。这些类分别位于不同的包中，并且应用于不同的场景。以下是这些类的基本介绍和示例。

## 1. `Date`

`Date`类是`java.util`包中的一部分，代表一个特定的瞬间，精确到毫秒。它是Java早期版本中的日期时间实现，现在大多数方法已被视为过时，不过在处理一些遗留系统时可能仍需使用。

**示例**:

```java
import java.util.Date;

Date now = new Date();
System.out.println(now.toString()); // 输出类似于 "Sun Apr 15 14:58:01 GMT 2024"
```

## 2. `LocalDate`

`LocalDate`类属于`java.time`包，这个包是Java 8中引入的现代日期时间API的一部分。`LocalDate`仅包含日期信息（年、月、日），适用于不需要时间或时区的场景。

**示例**:

```java
import java.time.LocalDate;

LocalDate today = LocalDate.now();
System.out.println(today.toString()); // 输出当前日期，例如 "2024-04-15"
```

## 3. `LocalDateTime`

`LocalDateTime`类也是`java.time`包的一部分。它包含日期和时间，但不含时区信息，适合需要日期和时间但不涉及时区处理的应用场景。

**示例**:

```java
import java.time.LocalDateTime;

LocalDateTime now = LocalDateTime.now();
System.out.println(now.toString()); // 输出当前日期和时间，例如 "2024-04-15T15:00:00"
```

## 推荐使用情况

- **`Date`**: 仅在处理遗留代码或需要与早期Java版本兼容的情况下使用。
- **`LocalDate`** 和 **`LocalDateTime`**: 推荐用于新的开发项目中，它们提供了更丰富的API，例如日期时间的加减、格式化、解析等，并且是线程安全的。

### 进阶功能

`LocalDate`和`LocalDateTime`支持多种操作，例如加减日期、比较日期、格式化输出等，使得日期时间处理更加灵活和强大。

**加日期示例**:

```java
LocalDate tomorrow = LocalDate.now().plusDays(1);
System.out.println(tomorrow.toString()); // 输出明天的日期
```

**格式化输出示例**:

```java
import java.time.format.DateTimeFormatter;

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String formattedDateTime = LocalDateTime.now().format(formatter);
System.out.println(formattedDateTime); // 输出类似于 "2024-04-15 15:02:00"
```

这些示例展示了如何使用Java的日期时间类处理常见的日期和时间问题。

## 日期工具类

```java
package common.util;


import org.apache.commons.lang3.time.DateFormatUtils;

import java.lang.management.ManagementFactory;
import java.text.Format;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.time.*;
import java.time.temporal.TemporalAdjusters;
import java.util.*;

/**
 * ClassName: DateUtils
 * description: 时间工具类.
 *
 * @author: pangdi
 * @date: 2023-8-29 11:36:02
 * @version: 1.1
 */
public class DateUtil extends org.apache.commons.lang3.time.DateUtils {

    /**
     * 日期格式模式，表示年份，格式为 "yyyy"。
     */
    public static final String YYYY = "yyyy";

    /**
     * 日期格式模式，表示年份和月份，格式为 "yyyy-MM"。
     */
    public static final String YYYY_MM = "yyyy-MM";

    /**
     * 日期格式模式，表示年份、月份和日期，格式为 "yyyy-MM-dd"。
     */
    public static final String YYYY_MM_DD = "yyyy-MM-dd";

    /**
     * 日期时间格式模式，格式为 "yyyyMMddHHmmss"。
     */
    public static final String YYYYMMDDHHMMSS = "yyyyMMddHHmmss";

    /**
     * 日期时间格式模式，格式为 "yyyy-MM-dd HH:mm:ss"。
     */
    public static final String YYYY_MM_DD_HH_MM_SS = "yyyy-MM-dd HH:mm:ss";

    /**
     * 日期时间格式模式，格式为 "yyyy-MM-dd HH:mm"。
     */
    public static final String YYYY_MM_DD_HH_MM = "yyyy-MM-dd HH:mm";

    /**
     * 日期格式模式，表示年份、月份和日期，格式为 "yyyyMMdd"。
     */
    public static final String YYYYMMDD = "yyyyMMdd";

    /**
     * 用于解析日期的常用日期格式模式数组。
     */
    private static String[] parsePatterns = {
            "yyyy-MM-dd", "yyyy-MM-dd HH:mm:ss", "yyyy-MM-dd HH:mm", "yyyy-MM",
            "yyyy/MM/dd", "yyyy/MM/dd HH:mm:ss", "yyyy/MM/dd HH:mm", "yyyy/MM",
            "yyyy.MM.dd", "yyyy.MM.dd HH:mm:ss", "yyyy.MM.dd HH:mm", "yyyy.MM",
            "yyyyMMddHHmmss", "yyyyMMdd"
    };

    /**
     * 获取当前LocalDate类型日期
     *
     * @return 当前LocalDateTime对象
     */
    public static LocalDateTime getNowLocalDateTime() {
        return LocalDateTime.now();
    }

    /**
     * 将LocalTime对象转换为Date对象。
     *
     * @param localTime 要转换的LocalTime对象
     * @return 转换后的Date对象
     */
    public static Date LocalTimeToDate(LocalTime localTime) {
        LocalDate localDate = LocalDate.now();
        LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
        ZoneId zone = ZoneId.systemDefault();
        Instant instant = localDateTime.atZone(zone).toInstant();
        return Date.from(instant);
    }

    /**
     * 将日期型字符串转换为Date对象。
     *
     * @param str 要转换的日期型字符串
     * @return 转换后的Date对象，如果转换失败则返回null
     */
    public static Date parseDate(Object str) {
        if (str == null) {
            return null;
        }
        try {
            return parseDate(str.toString(), parsePatterns);
        } catch (ParseException e) {
            return null;
        }
    }

    /**
     * 将Date对象转换为LocalDate对象。
     *
     * @param date 要转换的Date对象
     * @return 转换后的LocalDate对象，如果Date对象为null则返回null
     */
    public static LocalDate date2LocalDate(Date date) {
        if (null == date) {
            return null;
        }
        return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
    }

    /**
     * 将LocalDate对象转换为Date对象。
     *
     * @param localDate 要转换的LocalDate对象
     * @return 转换后的Date对象，如果LocalDate对象为null则返回null
     */
    public static Date localDate2Date(LocalDate localDate) {
        if (null == localDate) {
            return null;
        }
        ZonedDateTime zonedDateTime = localDate.atStartOfDay(ZoneId.systemDefault());
        return Date.from(zonedDateTime.toInstant());
    }

    /**
     * 将LocalDateTime对象转换为Date对象。
     *
     * @param localDateTime 要转换的LocalDateTime对象
     * @return 转换后的Date对象
     */
    public static Date asDate(LocalDateTime localDateTime) {
        return Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
    }

    /**
     * 获取LocalDateTime对象的时间戳（毫秒）。
     *
     * @param localDateTime 要获取时间戳的LocalDateTime对象
     * @return 时间戳（毫秒）
     */
    public static Long localDateTimeMilli(LocalDateTime localDateTime) {
        return localDateTime.toInstant(ZoneOffset.of("+8")).toEpochMilli();
    }

    /**
     * 获取两个日期之间的日期集合，按照从小到大的顺序排列。
     *
     * @param startDate 起始日期
     * @param endDate   结束日期
     * @return 两个日期之间的日期集合（包括起始日期和结束日期），按顺序排列
     */
    public static List<LocalDate> getAscDateList(LocalDate startDate, LocalDate endDate) {
        List<LocalDate> result = new ArrayList<>();
        if (endDate.compareTo(startDate) < 0) {
            return result;
        }
        while (true) {
            result.add(startDate);
            if (startDate.compareTo(endDate) >= 0) {
                break;
            }
            startDate = startDate.plusDays(1);
        }
        return result;
    }

    /**
     * 获取当前Date型日期
     *
     * @return Date() 当前日期
     */
    public static Date getNowDate() {
        return new Date();
    }

    /**
     * 获取当前日期，格式为"yyyy-MM-dd"。
     *
     * @return 当前日期字符串
     */
    public static String getDate() {
        return dateTimeNow(YYYY_MM_DD);
    }

    /**
     * 获取当前时间，格式为"yyyy-MM-dd HH:mm:ss"。
     *
     * @return 当前时间字符串
     */
    public static final String getTime() {
        return dateTimeNow(YYYY_MM_DD_HH_MM_SS);
    }

    /**
     * 获取当前日期时间，格式为"yyyyMMddHHmmss"。
     *
     * @return 当前日期时间字符串
     */
    public static final String dateTimeNow() {
        return dateTimeNow(YYYYMMDDHHMMSS);
    }

    /**
     * 获取当前日期时间，指定格式。
     *
     * @param format 日期时间字符串的格式
     * @return 当前日期时间字符串
     */
    public static final String dateTimeNow(final String format) {
        return parseDateToStr(format, new Date());
    }

    /**
     * 格式化指定的日期为"yyyy-MM-dd"格式。
     *
     * @param date 要格式化的日期对象
     * @return 格式化后的日期字符串
     */
    public static final String dateTime(final Date date) {
        return parseDateToStr(YYYY_MM_DD, date);
    }

    /**
     * 将Date对象格式化为指定格式的日期时间字符串。
     *
     * @param format 格式化的日期时间字符串的格式
     * @param date   要格式化的Date对象
     * @return 格式化后的日期时间字符串
     */
    public static final String parseDateToStr(final String format, final Date date) {
        return new SimpleDateFormat(format).format(date);
    }

    /**
     * 将日期字符串从一种格式转换为另一种格式
     *
     * @param dateStr   日期字符串
     * @param formatStr 日期格式字符串
     * @return 转换后的日期字符串
     */
    public static String parseStringToString(String dateStr, String formatStr) {
        try {
            SimpleDateFormat sdf = new SimpleDateFormat(formatStr);
            Date date = sdf.parse(dateStr);
            return sdf.format(date);
        } catch (ParseException e) {
            e.printStackTrace();
            return null;
        }
    }

    /**
     * 将日期字符串从一种格式转换为另一种格式
     *
     * @param dateString     输入的日期字符串
     * @param originalFormat 日期转换前的格式
     * @param targetFormat   日期转换后的格式
     * @return 转换后的日期字符串
     */
    public static String convertDateFormat(String dateString, String originalFormat, String targetFormat) {
        try {
            SimpleDateFormat originalDateFormat = new SimpleDateFormat(originalFormat);
            Date date = originalDateFormat.parse(dateString);
            SimpleDateFormat targetDateFormat = new SimpleDateFormat(targetFormat);
            return targetDateFormat.format(date);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 将指定格式的日期时间字符串解析为Date对象。
     *
     * @param format 日期时间字符串的格式
     * @param ts     要解析的日期时间字符串
     * @return 解析后的Date对象
     * @throws RuntimeException 如果解析失败，将抛出运行时异常
     */
    public static final Date dateTime(final String format, final String ts) {
        try {
            return new SimpleDateFormat(format).parse(ts);
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * 获取当前日期的路径形式的字符串表示，格式为"yyyy/MM/dd"。
     *
     * @return 当前日期的路径形式的字符串，格式为"yyyy/MM/dd"
     */
    public static final String datePath() {
        Date now = new Date();
        return DateFormatUtils.format(now, "yyyy/MM/dd");
    }

    /**
     * 获取当前日期时间的字符串表示，格式为"yyyyMMdd"。
     *
     * @return 当前日期时间的字符串，格式为"yyyyMMdd"
     */
    public static final String dateTime() {
        Date now = new Date();
        return DateFormatUtils.format(now, YYYYMMDD);
    }

    /**
     * 获取服务器启动的日期时间。
     *
     * @return 服务器启动的日期时间
     */
    public static Date getServerStartDate() {
        long time = ManagementFactory.getRuntimeMXBean().getStartTime();
        return new Date(time);
    }

    /**
     * 计算两个日期之间的时间差，并以"X天X小时X分钟"的格式返回。
     *
     * @param endDate 结束日期
     * @param nowDate 当前日期
     * @return 表示时间差的字符串，格式为"X天X小时X分钟"
     */
    public static String getDatePoor(Date endDate, Date nowDate) {
        long nd = 1000 * 24 * 60 * 60;
        long nh = 1000 * 60 * 60;
        long nm = 1000 * 60;
        // 获得两个时间的毫秒时间差异
        long diff = endDate.getTime() - nowDate.getTime();
        // 计算差多少天
        long day = diff / nd;
        // 计算差多少小时
        long hour = diff % nd / nh;
        // 计算差多少分钟
        long min = diff % nd % nh / nm;
        // 计算差多少秒//输出结果
        return day + "天" + hour + "小时" + min + "分钟";
    }

    /**
     * 计算两个日期之间的分钟差异。
     *
     * @param endDate 结束日期
     * @param nowDate 当前日期
     * @return 两个日期之间的分钟差异
     */
    public static long getMinutePoor(Date endDate, Date nowDate) {
        long nm = 1000 * 60;
        // 获得两个时间的毫秒时间差异
        long diff = endDate.getTime() - nowDate.getTime();
        // 计算差多少分钟
        return diff / nm;
    }

    /**
     * 计算两个日期之间的小时差异。
     *
     * @param endDate 结束日期
     * @param nowDate 当前日期
     * @return 两个日期之间的小时差异
     */
    public static long getHourPoor(Date endDate, Date nowDate) {
        long nh = 1000 * 60 * 60;
        // 获得两个时间的毫秒时间差异
        long diff = endDate.getTime() - nowDate.getTime();
        // 计算差多少小时
        return diff / nh;
    }

    /**
     * 获取N个月前或N个月后的最后一天。
     *
     * @param date              指定时间
     * @param monthsAgoOrFuture 要获取的月数，正数表示未来，负数表示过去
     * @return N个月前或N个月后的最后一天的Date对象
     */
    public static Date getLastDayOfMonthsAgoOrFuture(Date date, int monthsAgoOrFuture) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        // 根据正负数来增加或减少月份
        calendar.add(Calendar.MONTH, monthsAgoOrFuture);
        // 将日期设置为指定月份的最后一天
        calendar.set(Calendar.DAY_OF_MONTH, calendar.getActualMaximum(Calendar.DAY_OF_MONTH));
        return calendar.getTime();
    }

    /**
     * 获取月有多少天
     *
     * @param date 时间
     * @return 天数
     */
    public static int getDaysOfMonth(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        return calendar.getActualMaximum(Calendar.DAY_OF_MONTH);
    }

    /**
     * 获取时间月的第一天
     *
     * @param date 时间
     * @return 时间月的第一天
     */
    public static Date getFirstDayOfMonth(Date date) {
        Calendar firstCal = Calendar.getInstance();
        firstCal.setTime(date);
        firstCal.set(Calendar.DAY_OF_MONTH, 1);
        return firstCal.getTime();
    }

    /**
     * 获取时间月的最后一天
     *
     * @param date 时间
     * @return 时间月的最后一天
     */
    public static Date getLastDayOfMonth(Date date) {
        Calendar lastCal = Calendar.getInstance();
        lastCal.setTime(date);
        lastCal.set(Calendar.DAY_OF_MONTH, lastCal.getActualMaximum(Calendar.DAY_OF_MONTH));
        return lastCal.getTime();
    }

    /**
     * 获取当年的第一天
     *
     * @return 当年的第一天
     */
    public static Date getCurrYearFirst() {
        Calendar currCal = Calendar.getInstance();
        int currentYear = currCal.get(Calendar.YEAR);
        return getYearFirst(currentYear);
    }

    /**
     * 获取当年的最后一天
     *
     * @return 当年的最后一天
     */
    public static Date getCurrYearLast() {
        Calendar currCal = Calendar.getInstance();
        int currentYear = currCal.get(Calendar.YEAR);
        return getYearLast(currentYear);

    }

    /**
     * 获取某年第一天日期
     *
     * @paramyear 年份
     * @returnDate 某年第一天日期
     */
    public static Date getYearFirst(int year) {
        Calendar calendar = Calendar.getInstance();
        calendar.clear();
        calendar.set(Calendar.YEAR, year);
        Date currYearFirst = calendar.getTime();
        return currYearFirst;
    }

    /**
     * 获取某年最后一天日期
     *
     * @paramyear 年份
     * @returnDate 某年最后一天日期
     */
    public static Date getYearLast(int year) {
        Calendar calendar = Calendar.getInstance();
        calendar.clear();
        calendar.set(Calendar.YEAR, year);
        calendar.roll(Calendar.DAY_OF_YEAR, -1);
        return calendar.getTime();
    }

    /**
     * 计算两个日期 相差天数
     *
     * @param startDate 开始时间
     * @param endDate   结束时间
     * @return 天数
     */
    public static Long getDatePoorDay(Date startDate, Date endDate) {
        long nd = 1000 * 24 * 60 * 60;
        // 获得两个时间的毫秒时间差异
        long diff = endDate.getTime() - startDate.getTime();
        // 计算差多少天
        return diff / nd;
    }

    /**
     * Date 转 LocalDate
     *
     * @param date
     * @return
     */
    public static LocalDate dateToLocalDate(Date date) {
        if (date == null) {
            return null;
        }
        try {
            Instant instant = date.toInstant();
            ZoneId zoneId = ZoneId.systemDefault();
            return instant.atZone(zoneId).toLocalDate();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * date 转 localDateTime
     *
     * @param date
     * @return
     */
    public static LocalDateTime dateToLocalDateTime(Date date) {
        if (date == null) {
            return null;
        }
        return LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
    }

    /**
     * localDateTime 转 date
     *
     * @param localDateTime
     * @return
     */
    public static Date localDateTimeToDate(LocalDateTime localDateTime) {
        if (localDateTime == null) {
            return null;
        }
        ZonedDateTime zonedDateTime = localDateTime.atZone(ZoneId.systemDefault());
        return Date.from(zonedDateTime.toInstant());
    }

    /**
     * LocalDate转Date
     *
     * @param localDate
     * @return
     */
    public static Date localDateToDate(LocalDate localDate) {
        if (localDate == null) {
            return null;
        }
        ZonedDateTime zonedDateTime = localDate.atStartOfDay(ZoneId.systemDefault());
        return Date.from(zonedDateTime.toInstant());
    }

    /**
     * 时间加减小时
     *
     * @param startDate 要处理的时间，Null则为当前时间
     * @param hours     加减的小时
     * @return Date 时间
     */
    public static Date dateAddHours(Date startDate, int hours) {
        if (startDate == null) {
            startDate = new Date();
        }
        Calendar c = Calendar.getInstance();
        c.setTime(startDate);
        c.set(Calendar.HOUR, c.get(Calendar.HOUR) + hours);
        return c.getTime();
    }

    /**
     * 时间加减分钟
     *
     * @param startDate 要处理的时间，Null则为当前时间
     * @param minutes   加减的分钟
     * @return Date 时间
     */
    public static Date dateAddMinutes(Date startDate, int minutes) {
        if (startDate == null) {
            startDate = new Date();
        }
        Calendar c = Calendar.getInstance();
        c.setTime(startDate);
        c.set(Calendar.MINUTE, c.get(Calendar.MINUTE) + minutes);
        return c.getTime();
    }

    /**
     * 时间加减年数
     *
     * @param startDate 要处理的时间，Null则为当前时间
     * @param years     加减的月数
     * @return Date 时间
     */
    public static Date dateAddYears(Date startDate, int years) {
        if (startDate == null) {
            startDate = new Date();
        }
        Calendar c = Calendar.getInstance();
        c.setTime(startDate);
        c.set(Calendar.YEAR, c.get(Calendar.YEAR) + years);
        return c.getTime();
    }

    /**
     * 时间加减月数
     *
     * @param startDate 要处理的时间，Null则为当前时间
     * @param months    加减的月数
     * @return Date 时间
     */
    public static Date dateAddMonths(Date startDate, int months) {
        if (startDate == null) {
            startDate = new Date();
        }
        Calendar c = Calendar.getInstance();
        c.setTime(startDate);
        c.set(Calendar.MONTH, c.get(Calendar.MONTH) + months);
        return c.getTime();
    }

    /**
     * 时间加减天数
     *
     * @param startDate 要处理的时间，Null则为当前时间
     * @param days      加减的天数
     * @return Date 时间
     */
    public static Date dateAddDays(Date startDate, int days) {
        if (startDate == null) {
            startDate = new Date();
        }
        Calendar c = Calendar.getInstance();
        c.setTime(startDate);
        c.set(Calendar.DATE, c.get(Calendar.DATE) + days);
        return c.getTime();
    }

    /**
     * 通过添加或减去指定年数来操作日期字符串，然后将结果转换为指定的输出格式字符串。
     *
     * @param inputDate           输入日期字符串。
     * @param inputFormat         输入日期字符串的格式。
     * @param outputFormat        输出日期字符串的期望格式。
     * @param yearsToAddOrSubtract 要添加（正数）或减去（负数）的年数。
     * @return 按指定输出格式的操作后日期字符串。
     */
    public static String manipulateYear(String inputDate, String inputFormat, String outputFormat, int yearsToAddOrSubtract) {
        try {
            SimpleDateFormat inputDateFormat = new SimpleDateFormat(inputFormat);
            Date date = inputDateFormat.parse(inputDate);
            Calendar calendar = Calendar.getInstance();
            calendar.setTime(date);
            calendar.add(Calendar.YEAR, yearsToAddOrSubtract);
            SimpleDateFormat outputDateFormat = new SimpleDateFormat(outputFormat);
            return outputDateFormat.format(calendar.getTime());
        } catch (ParseException e) {
            e.printStackTrace();
        }

        return null;
    }

    /**
     * 通过添加或减去指定月数来操作日期字符串，然后将结果转换为指定的输出格式字符串。
     *
     * @param inputDate           输入日期字符串。
     * @param inputFormat         输入日期字符串的格式。
     * @param outputFormat        输出日期字符串的期望格式。
     * @param monthsToAddOrSubtract 要添加（正数）或减去（负数）的月数。
     * @return 按指定输出格式的操作后日期字符串。
     */
    public static String manipulateMonth(String inputDate, String inputFormat, String outputFormat, int monthsToAddOrSubtract) {
        try {
            SimpleDateFormat inputDateFormat = new SimpleDateFormat(inputFormat);
            Date date = inputDateFormat.parse(inputDate);
            Calendar calendar = Calendar.getInstance();
            calendar.setTime(date);
            calendar.add(Calendar.MONTH, monthsToAddOrSubtract);
            SimpleDateFormat outputDateFormat = new SimpleDateFormat(outputFormat);
            return outputDateFormat.format(calendar.getTime());
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 通过添加或减去指定天数来操作日期字符串，然后将结果转换为指定的输出格式字符串。
     *
     * @param inputDate           输入日期字符串。
     * @param inputFormat         输入日期字符串的格式。
     * @param outputFormat        输出日期字符串的期望格式。
     * @param daysToAddOrSubtract 要添加（正数）或减去（负数）的天数。
     * @return 按指定输出格式的操作后日期字符串。
     */
    public static String manipulateDate(String inputDate, String inputFormat, String outputFormat, int daysToAddOrSubtract) {
        try {
            SimpleDateFormat inputDateFormat = new SimpleDateFormat(inputFormat);
            Date date = inputDateFormat.parse(inputDate);
            Calendar calendar = Calendar.getInstance();
            calendar.setTime(date);
            calendar.add(Calendar.DAY_OF_MONTH, daysToAddOrSubtract);
            SimpleDateFormat outputDateFormat = new SimpleDateFormat(outputFormat);
            return outputDateFormat.format(calendar.getTime());
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 获取当前周的周一的日期
     *
     * @param date 传入当前日期
     * @return 当前周的周一的日期
     */
    public static Date getThisWeekMonday(Date date) {
        Calendar cal = Calendar.getInstance();
        cal.setTime(date);
        // 获得当前日期是一个星期的第几天
        int dayWeek = cal.get(Calendar.DAY_OF_WEEK);
        if (1 == dayWeek) {
            cal.add(Calendar.DAY_OF_MONTH, -1);
        }
        // 设置一个星期的第一天，按中国的习惯一个星期的第一天是星期一
        cal.setFirstDayOfWeek(Calendar.MONDAY);
        // 获得当前日期是一个星期的第几天
        int day = cal.get(Calendar.DAY_OF_WEEK);
        cal.add(Calendar.DATE, cal.getFirstDayOfWeek() - day);
        return cal.getTime();
    }

    /**
     * 获取指定年月的最后一天的日期
     *
     * @param year  年份
     * @param month 月份（1-12）
     * @return 最后一天的日期
     */
    public static int getMonthLastDay(int year, int month) {
        Calendar a = Calendar.getInstance();
        a.set(Calendar.YEAR, year);
        a.set(Calendar.MONTH, month - 1);
        // 把日期设置为当月第一天
        a.set(Calendar.DATE, 1);
        // 日期回滚一天，也就是最后一天
        a.roll(Calendar.DATE, -1);
        int maxDate = a.get(Calendar.DATE);
        return maxDate;
    }

    /**
     * 同步日期的年月日，将时分秒设置为指定日期的时分秒
     *
     * @param date     日期
     * @param dateTime 包含时分秒的日期
     * @return 同步后的日期
     */
    public static Date synchronization(Date date, Date dateTime) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(dateTime);
        int time = calendar.get(Calendar.HOUR_OF_DAY);
        int branch = calendar.get(Calendar.MINUTE);
        if (date == null) {
            date = new Date();
        }
        calendar.setTime(date);
        calendar.set(Calendar.HOUR_OF_DAY, time);
        calendar.set(Calendar.MINUTE, branch);
        Date zero = calendar.getTime();
        return zero;
    }

    /**
     * 获取指定日期的开始时间（时分秒毫秒均为0）
     *
     * @param date 日期
     * @return 开始时间
     */
    public static Date byStart(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        calendar.set(Calendar.HOUR_OF_DAY, 0);
        calendar.set(Calendar.MINUTE, 0);
        calendar.set(Calendar.SECOND, 0);
        calendar.set(Calendar.MILLISECOND, 0);
        Date zero = calendar.getTime();
        return zero;
    }

    /**
     * 获取指定日期的结束时间（时分秒毫秒均为23:59:59.999）
     *
     * @param date 日期
     * @return 结束时间
     */
    public static Date byEnd(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        calendar.set(Calendar.HOUR_OF_DAY, 23);
        calendar.set(Calendar.MINUTE, 59);
        calendar.set(Calendar.SECOND, 59);
        calendar.set(Calendar.MILLISECOND, 999);
        Date zero = calendar.getTime();
        return zero;
    }

    /**
     * 获取两个日期之间的所有日期集合。
     *
     * @param startDate 开始日期
     * @param endDate   结束日期
     * @return 包含日期字符串的List
     */
    public static List<String> getDatesBetween(Date startDate, Date endDate) {
        List<String> dateList = new ArrayList<>();
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(startDate);
        SimpleDateFormat dateFormat = new SimpleDateFormat(YYYY_MM_DD);
        while (!calendar.getTime().after(endDate)) {
            dateList.add(dateFormat.format(calendar.getTime()));
            calendar.add(Calendar.DATE, 1);
        }
        return dateList;
    }

    /**
     * 获取两个日期之间的所有日期集合（包括开始和结束日期）
     *
     * @param start 开始日期
     * @param end   结束日期
     * @return 日期集合
     */
    public static List<Date> getBetweenDates(Date start, Date end) {
        List<Date> result = new ArrayList<>();
        Format f = new SimpleDateFormat(YYYY_MM_DD);
        String date1 = f.format(start);
        String date2 = f.format(end);
        if (end.before(start)) {
            return result;
        }
        Calendar tempStart = Calendar.getInstance();
        if (date1.equals(date2)) {
            tempStart.setTime(end);
            result.add(tempStart.getTime());
            return result;
        }
        tempStart.setTime(start);
        result.add(tempStart.getTime());
        tempStart.add(Calendar.DAY_OF_YEAR, 1);
        Calendar tempEnd = Calendar.getInstance();
        tempEnd.setTime(end);
        while (tempStart.before(tempEnd)) {
            result.add(tempStart.getTime());
            tempStart.add(Calendar.DAY_OF_YEAR, 1);
        }
        tempStart.setTime(end);
        result.add(tempStart.getTime());
        return result;
    }

    /**
     * 获取两个日期之间的所有月份的字符串集合。
     *
     * @param minDate 开始日期
     * @param maxDate 结束日期
     * @return 包含两个日期之间所有月份的字符串列表，格式为"yyyy/MM"
     */
    public static List<String> getMonthBetween(Date minDate, Date maxDate) {
        ArrayList<String> result = new ArrayList<>();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM");
        Calendar min = Calendar.getInstance();
        Calendar max = Calendar.getInstance();
        min.setTime(minDate);
        min.set(min.get(Calendar.YEAR), min.get(Calendar.MONTH), 1);
        max.setTime(maxDate);
        max.set(max.get(Calendar.YEAR), max.get(Calendar.MONTH), 2);
        Calendar curr = min;
        while (curr.before(max)) {
            result.add(sdf.format(curr.getTime()));
            curr.add(Calendar.MONTH, 1);
        }
        return result;
    }

    /**
     * 获取两个日期之间的所有月份的集合。
     *
     * @param minDate 开始日期
     * @param maxDate 结束日期
     * @return 包含两个日期之间所有月份的Date对象列表
     */
    public static List<Date> getMonthBetweenDate(Date minDate, Date maxDate) {
        ArrayList<Date> result = new ArrayList<>();
        Calendar min = Calendar.getInstance();
        Calendar max = Calendar.getInstance();
        min.setTime(minDate);
        min.set(min.get(Calendar.YEAR), min.get(Calendar.MONTH), 1);
        max.setTime(maxDate);
        max.set(max.get(Calendar.YEAR), max.get(Calendar.MONTH), 2);
        Calendar curr = min;
        while (curr.before(max)) {
            result.add(curr.getTime());
            curr.add(Calendar.MONTH, 1);
        }
        return result;
    }

    /**
     * 处理日期，返回上一个月的第一天的日期。
     *
     * @param createTime 要处理的日期
     * @return 上一个月的第一天的日期
     */
    public static Date handleLastMonth(Date createTime) {
        Instant instant = createTime.toInstant();
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDate localDate = LocalDateTime.ofInstant(instant, zoneId).toLocalDate();
        LocalDate with = localDate.minusMonths(1).with(TemporalAdjusters.firstDayOfMonth());
        Instant instant1 = with.atStartOfDay().atZone(zoneId).toInstant();
        return Date.from(instant1);
    }

    /**
     * 处理日期，返回上一个月的最后一天的日期。
     *
     * @param date 要处理的日期
     * @return 上一个月的最后一天的日期
     */
    public static Date handleLastMonthDay(Date date) {
        Instant instant = date.toInstant();
        ZoneId zoneId = ZoneId.systemDefault();
        LocalDate localDate = LocalDateTime.ofInstant(instant, zoneId).toLocalDate();
        LocalDate with = localDate.minusMonths(1).with(TemporalAdjusters.lastDayOfMonth());
        Instant instant1 = with.atStartOfDay().atZone(zoneId).toInstant();
        return Date.from(instant1);
    }

    /**
     * 获取日期对应的星期几
     *
     * @param dt 日期
     * @return 星期几的字符串，如"星期日"
     */
    public static String getWeekOfDate(Date dt) {
        String[] weekDays = {"星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"};
        Calendar cal = Calendar.getInstance();
        cal.setTime(dt);
        int w = cal.get(Calendar.DAY_OF_WEEK) - 1;
        if (w < 0) {
            w = 0;
        }
        return weekDays[w];
    }

    /**
     * 获取日期是当月的第几天
     *
     * @param date 日期
     * @return 当月的第几天
     */
    public static int getDayOfMonth(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        return calendar.get(Calendar.DAY_OF_MONTH);
    }

    /**
     * 将Excel中表示日期的数字转换为日期格式
     *
     * @param number 1900年到当前时间的间隔天数
     * @return 转换后的日期
     */
    public static Date transferExcelNumberToDate(String number) {
        // month=0代表1月份，day=-1道标
        Calendar calendar = new GregorianCalendar(1900, 0, -1);
        Date time = calendar.getTime();
        LocalDate localDate = dateToLocalDate(time).plusDays(Long.valueOf(number));
        return localDateToDate(localDate);
    }

    /**
     * 查询传入时间所在月的第一天
     *
     * @param localDate 传入日期
     * @return 月的第一天日期
     */
    public static LocalDate firstDayOfCurrentMonth(LocalDate localDate) {
        return localDate.with(TemporalAdjusters.firstDayOfMonth());
    }

    /**
     * 查询传入时间所在月的最后一天
     *
     * @param localDate 传入日期
     * @return 月的最后一天日期
     */
    public static LocalDate lastDayOfCurrentMonth(LocalDate localDate) {
        return localDate.with(TemporalAdjusters.lastDayOfMonth());
    }

    /**
     * 查询传入时间所在周的第一天
     *
     * @param localDate 传入日期
     * @return 周的第一天日期
     */
    public static LocalDate firstDayOfCurrentWeek(LocalDate localDate) {
        return localDate.with(DayOfWeek.MONDAY);
    }

    /**
     * 查询传入时间所在周的最后一天
     *
     * @param localDate 传入日期
     * @return 周的最后一天日期
     */
    public static LocalDate lastDayOfCurrentWeek(LocalDate localDate) {
        return localDate.with(DayOfWeek.SUNDAY);
    }

    /**
     * 判断日期是否在指定期间之间
     *
     * @param startDate 开始日期
     * @param endDate   结束日期
     * @param date      待检查日期
     * @return 如果日期在指定期间之间，返回true；否则，返回false
     */
    public static Boolean isBetweenMonth(LocalDate startDate, LocalDate endDate, LocalDate date) {
        if ((startDate != null && endDate != null)
                && (startDate.equals(date) || startDate.isBefore(date))
                && (endDate.equals(date) || endDate.isAfter(date))) {
            return true;
        }
        return false;
    }

    /**
     * 将字符串转换为日期
     *
     * @param str 待转换的字符串
     * @return 转换后的日期，如果转换失败则返回null
     */
    public static Date convertStringToyyyyMMdd(String str) {
        SimpleDateFormat format = new SimpleDateFormat(YYYYMMDD);
        try {
            return format.parse(str);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 将字符串转换为日期时间
     *
     * @param str 待转换的字符串
     * @return 转换后的日期时间，如果转换失败则返回null
     */
    public static Date convertStringToyyyyMMddHHMMddHHmmss(String str) {
        SimpleDateFormat format = new SimpleDateFormat(YYYY_MM_DD_HH_MM_SS);
        try {
            return format.parse(str);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 判断当前时间是否为当前月.
     *
     * @param date 时间
     * @return true/false
     */
    public static Boolean isNowMonth(Date date) {
        // 获取当前时间
        Calendar cal = Calendar.getInstance();
        // 获取当前月份，注意 Calendar.MONTH 是从 0 开始计数的，需要加 1
        int currentMonth = cal.get(Calendar.MONTH) + 1;
        // 将待判断的日期对象设置到 Calendar 中
        cal.setTime(date);
        // 获取日期对象的月份
        int dateMonth = cal.get(Calendar.MONTH) + 1;
        return dateMonth == currentMonth;
    }

    /**
     * 判断当前时间是否为当前年.
     *
     * @param date 时间
     * @return true/false
     */
    public static Boolean isNowYear(Date date) {
        Date nowDate = new Date();
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(nowDate);
        int currentYear = calendar.get(Calendar.YEAR);
        calendar.setTime(date);
        int year = calendar.get(Calendar.YEAR);
        return year == currentYear;
    }

    /**
     * 转时间为yyyyMMdd的字符串.
     *
     * @param date 时间
     * @return 字符串
     */
    public static String convertYYYYMMDD(Date date) {
        return DateFormatUtils.format(date, YYYYMMDD);
    }

    /**
     * 获取当前 时间年的第一天
     *
     * @param date 时间
     * @return 时间年的第一天
     */
    public static Date getFirstDayOfYear(Date date) {
        // 创建一个新的Calendar实例，并将日期设置为给定日期所在年份的第一天
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        int year = calendar.get(Calendar.YEAR);
        calendar.set(year, 0, 1);
        return calendar.getTime();
    }

    /**
     * 获取指定日期的年份。
     *
     * @param date 指定的日期对象。
     * @return 年份。
     */
    public static int getYear(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        return calendar.get(Calendar.YEAR);
    }

    /**
     * 获取指定日期的月份。
     *
     * @param date 指定的日期对象。
     * @return 月份，范围为1-12。
     */
    public static int getMonth(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        return calendar.get(Calendar.MONTH) + 1;
    }

    /**
     * 获取指定日期的日。
     *
     * @param date 指定的日期对象。
     * @return 日，范围为1-31。
     */
    public static int getDay(Date date) {
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(date);
        return calendar.get(Calendar.DAY_OF_MONTH);
    }

    /**
     * 获取两个日期字符串之间的日期范围（包括起始日期和结束日期）。
     *
     * @param startDateStr 起始日期字符串（yyyyMMdd格式）
     * @param endDateStr   结束日期字符串（yyyyMMdd格式）
     * @return 包含日期范围内所有日期的yyyyMMdd格式字符串集合
     * @throws ParseException 如果日期字符串格式不正确，将抛出ParseException异常
     */
    public static List<String> getDateRange(String startDateStr, String endDateStr) throws ParseException {
        List<String> dateList = new ArrayList<>();
        SimpleDateFormat dateFormat = new SimpleDateFormat(YYYYMMDD);
        Date startDate = dateFormat.parse(startDateStr);
        Date endDate = dateFormat.parse(endDateStr);
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(startDate);
        while (!calendar.getTime().after(endDate)) {
            dateList.add(dateFormat.format(calendar.getTime()));
            calendar.add(Calendar.DAY_OF_MONTH, 1);
        }
        return dateList;
    }

    /**
     * 获取从月初开始到指定日期字符串之间的日期范围（包括月初和指定日期）。
     *
     * @param endDateStr 指定日期字符串（yyyyMMdd格式）
     * @return 包含日期范围内所有日期的yyyyMMdd格式字符串集合
     * @throws ParseException 如果日期字符串格式不正确，将抛出ParseException异常
     */
    public static List<String> getMonthToDateRange(String endDateStr) throws ParseException {
        List<String> dateList = new ArrayList<>();
        SimpleDateFormat dateFormat = new SimpleDateFormat(YYYYMMDD);
        Date endDate = dateFormat.parse(endDateStr);
        Calendar calendar = Calendar.getInstance();
        calendar.setTime(endDate);
        calendar.set(Calendar.DAY_OF_MONTH, 1);
        while (!calendar.getTime().after(endDate)) {
            dateList.add(dateFormat.format(calendar.getTime()));
            calendar.add(Calendar.DAY_OF_MONTH, 1);
        }
        return dateList;
    }

    /**
     * 判断两个日期是否在同一年和同一月。
     *
     * @param date1 第一个日期
     * @param date2 第二个日期
     * @return 如果两个日期在同一年和同一月，返回true；否则返回false。
     * @throws IllegalArgumentException 如果其中一个日期为null，将抛出IllegalArgumentException。
     */
    public static Boolean isSameYearAndMonth(Date date1, Date date2) {
        if (date1 == null || date2 == null) {
            throw new IllegalArgumentException("日期不能为空");
        }
        Calendar cal1 = Calendar.getInstance();
        cal1.setTime(date1);
        Calendar cal2 = Calendar.getInstance();
        cal2.setTime(date2);
        int year1 = cal1.get(Calendar.YEAR);
        int month1 = cal1.get(Calendar.MONTH);
        int year2 = cal2.get(Calendar.YEAR);
        int month2 = cal2.get(Calendar.MONTH);
        return year1 == year2 && month1 == month2;
    }

    /**
     * 判断时间字符串是否为该月的月底，考虑闰年情况
     *
     * @param timeStr 时间字符串，格式为指定时间格式
     * @param format  时间格式，如"yyyy-MM-dd"
     * @return 如果时间字符串为该月的月底，则返回true；否则返回false
     */
    public static boolean isEndOfMonth(String timeStr, String format) {
        try {
            Date date = new SimpleDateFormat(format).parse(timeStr);
            Calendar calendar = Calendar.getInstance();
            calendar.setTime(date);
            int dayOfMonth = calendar.getActualMaximum(Calendar.DAY_OF_MONTH);
            int currentDayOfMonth = calendar.get(Calendar.DAY_OF_MONTH);
            return currentDayOfMonth == dayOfMonth;
        } catch (ParseException e) {
            e.printStackTrace();
            return false;
        }
    }
}

```

