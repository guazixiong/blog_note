# Cron 表达式

## 简介

cron表达式（cronExpression）是一个字符串，字符串由6个或者7个字段组成，字段之间用空格隔开，用来表示执行器触发执行的规则，使用cronExpression的任务会按照表达式指定的时间触发执行。

比如Spring-Task中用到的定时任务配置中的`cron="0 0 7 * * ?"`如下:

```java
    <task:scheduled-tasks>  
        <task:scheduled ref="idMaintain" method="maintain" cron="0 0 7 * * ?"/>
    </task:scheduled-tasks>   
```

## 字段含义

### cron 6位表达式

![](Cron%20%E8%A1%A8%E8%BE%BE%E5%BC%8F.assets/202305272133272.png)

- 从左到右依次
  
  - 第一位代表 `秒` ，只允许写 `0-59` 之间的数字
  
  - 第二位代表 `分` ，只允许写 `0-59` 之间的数字，不允许出现 `60`
  
  - 第三位代表 `时` ,只允许写 `0-23` 之间的数字，凌晨12时不允许写 `24`
  
  - 第四位代表 `日` ，只允许写 `1-31` 之间的数字
  
  - 第五位代表 `月` ，只允许写 `1-12` 之间的数字
  
  - 第六位代表 `星期` ，只允许写 `0-7` 之间的数字， `0` 和 `7` 都代表周日

### cron 5位表达式

![](Cron%20%E8%A1%A8%E8%BE%BE%E5%BC%8F.assets/202305272134950.png)

- 从左到右依次
  
  - 第一位代表 `分` ，只允许写 `0-59` 之间的数字，不允许出现 `60`
  
  - 第二位代表 `时` ,只允许写 `0-23` 之间的数字，凌晨12点时不允许写 `24`
  
  - 第三位代表 `日` ，只允许写 `1-31` 之间的数字
  
  - 第四位代表 `月` ，只允许写 `1-12` 之间的数字
  
  - 第五位代表 `星期` ，只允许写 `0-7` 之间的数字， `0` 和 `7` 都代表周日

### 特殊字符

- `*`：表示所有值；

- `?`：表示未说明的值，即不关心它为何值；

- `-`：表示一个指定的范围；

- `,`：表示附加一个可能值；

- `/`：符号前表示开始时间，符号后表示每次递增的值；

- `L`：("last") "L" 用在`日期`字段意思是 "这个月最后一天"；用在 `星期`字段, 它简单意思是 "7" or "SAT"。 如果在`星期`字段里和数字联合使用，它的意思就是 "这个月的最后一个星期几" – 例如： "6L" means "这个月的最后一个星期五". 当我们用“L”时。

- `W`：只能用在`日期`字段。用来描叙最接近指定天的工作日（周一到周五）。例如：在`日期`字段用“15W”指“最接近这个月第15天的工作日”，即如果这个月第15天是周六，那么触发器将会在这个月第14天即周五触发；如果这个月第15天是周日，那么触发器将会在这个月第 16天即周一触发；如果这个月第15天是周二，那么就在触发器这天触发。注意一点：这个用法只会在当前月计算值，不会越过当前月。“W”字符仅能在 day-of-month指明一天，不能是一个范围或列表。也可以用“LW”来指定这个月的最后一个工作日。

- `#`：只能用在`星期`字段。用来指定这个月的第几个周几。例：在`星期`字段用"6#3"指这个月第3个周五（6指周五，3指第3个）。如果指定的日期不存在，触发器就不会触发。

- `C`：C是calendar意思，本字段只出现在`星期`字段和`日期`字段。代表一周的第几天，或者是一个月的第几天。写法如`3C`，在`星期`字段则代表一周第三天即为星期二（周日算第一天）,在`日期`字段，则代表每月第二天。

**关于`'?'`** 以前我总是困惑，觉得`'?'`表示不考虑该字段，那就表示取任意值都可以呗，而取任意值用`*`表示，总纠结于这两者的区别。从上面`字段含义`列表中我们可以看出`'?'`只在`日期`字段和`星期`字段出现才明白其中道理。日期字段和星期字段本来就有意思上的冲突，比如一个表达式日期字段上是`'*'`，代表每天都执行，星期字段上是`'2'`代表每个周二执行，问题就来了，到底是每天都执行呢，还是仅周二执行呢。所以这两个字段始终会有一个字段忽略不考虑，总结出一个规律，每个表达式中`日期`和`星期`两个字段中必然有一个的取值是`'?'`

### 错误示范

- `60 * * * * *` 理想：每分钟第60秒执行一次。实际：无法执行。原因：`秒` 位不允许写 `60` 。正确表达： `0 * * * * *`

- `* 12-18 * * *` 理想：每日12时-18时第0分执行一次。实际：每日12时-18时每1分钟执行一次。原因：第一位是 `*` 。正确表达： `0 12-18 * * *`
  
  - `* 12-18 * * *` 上榜原因：群友经典示范，集体眼瞎系列

- `0 18-12 * * *` 理想：每日18时-次日12时第0分钟执行一次。实际：无法执行。原因：不可跨时间刻度。正确表达： `0 18-23,0-12 * * *`

- `*/90 * * * * *` 理想：每90秒执行一次。实际：每60秒执行一次。原因：`秒` 只有 `60` ，大于 `60` 时按 `60` 计算。正确表达：目前无法做到，推荐曲线救国

## 在线工具

使用[在线工具](http://cron.qqe2.com/)，帮助我们检测自己的表达式。输入表达式，反解析执行时间，如果没有解析出预期结果，说明表达式有问题

## Cron工具类

```java
package com.sangto.board.task.util;

import com.sangto.board.task.domain.TaskScheduleModel;

import java.util.HashMap;
import java.util.Map;

/**
 * @author: pangdi @Date: 2023/5/19 09:25 @Description: cron表达式生成器
 */
public class CronUtil {

  /**
   * 方法摘要：构建Cron表达式
   *
   * @param taskScheduleModel 定时任务cron表达式生成类
   * @return String cron表达式
   */
  public static String createCronExpression(TaskScheduleModel taskScheduleModel) {
    StringBuffer cronExp = new StringBuffer("");

    if (null == taskScheduleModel.getJobType()) {
      throw new RuntimeException("执行周期未配置");
    }

    switch (taskScheduleModel.getJobType().intValue()) {
      case 0: // 秒
        if (taskScheduleModel.getSecond() != null) {
          cronExp.append("0/").append(taskScheduleModel.getSecond());
          cronExp.append(" * * * * ?");
          return cronExp.toString();
        }
        break;

      case 1: // 分钟
        if (taskScheduleModel.getSecond() != null) {
          cronExp.append("0/").append(taskScheduleModel.getSecond());
        }
        if (taskScheduleModel.getMinute() != null) {
          cronExp.append(" 0/").append(taskScheduleModel.getMinute());
          cronExp.append(" * * * ? *");
          return cronExp.toString();
        }
        break;

      case 2: // 天
        if (taskScheduleModel.getSecond() != null
            && taskScheduleModel.getMinute() != null
            && taskScheduleModel.getHour() != null) {
          cronExp.append(taskScheduleModel.getSecond()).append(" ");
          cronExp.append(taskScheduleModel.getMinute()).append(" ");
          cronExp.append(taskScheduleModel.getHour()).append(" ");
          cronExp.append("* * ?");
          return cronExp.toString();
        }
        throw new RuntimeException("时或分或秒参数未配置");
      case 3: // 周
        if (taskScheduleModel.getSecond() != null
                && taskScheduleModel.getMinute() != null
                && taskScheduleModel.getHour() != null) {
          cronExp.append(taskScheduleModel.getSecond()).append(" ");
          cronExp.append(taskScheduleModel.getMinute()).append(" ");
          cronExp.append(taskScheduleModel.getHour()).append(" ");
          cronExp.append("? * ");
          Integer[] weeks = taskScheduleModel.getDayOfWeeks();
          for (int i = 0; i < weeks.length; i++) {
            if (i == 0) {
              cronExp.append(weeks[i]);
            } else {
              cronExp.append(",").append(weeks[i]);
            }
          }
          return cronExp.toString();
        }
        throw new RuntimeException("时或分或秒参数未配置");
      case 4: // 月
        if (taskScheduleModel.getSecond() != null
                && taskScheduleModel.getMinute() != null
                && taskScheduleModel.getHour() != null) {
          cronExp.append(taskScheduleModel.getSecond()).append(" ");
          cronExp.append(taskScheduleModel.getMinute()).append(" ");
          cronExp.append(taskScheduleModel.getHour()).append(" ");
          Integer[] days = taskScheduleModel.getDayOfMonths();
          for (int i = 0; i < days.length; i++) {
            if (i == 0) {
              cronExp.append(days[i]);
            } else {
              cronExp.append(",").append(days[i]);
            }
          }
          cronExp.append(" * ? *");
          return cronExp.toString();
        }
        throw new RuntimeException("时或分或秒参数未配置");
      default:
        throw new RuntimeException("时或分或秒参数未配置");
    }
    return cronExp.toString();
  }

  /**
   * 方法摘要：生成计划的详细描述
   *
   * @param taskScheduleModel
   * @return String
   */
  public static String createDescription(TaskScheduleModel taskScheduleModel) {
    StringBuffer description = new StringBuffer("");

    switch (taskScheduleModel.getJobType().intValue()) {
      case 0: // 每秒
        description.append("每秒").append(taskScheduleModel.getSecond()).append("秒").append("执行");
        break;
      case 1: // 每分钟
        description.append("每分").append(taskScheduleModel.getMinute()).append("分").append("执行");
        break;
      case 2: // 按每天
        description.append("每天");
        description.append(taskScheduleModel.getHour()).append("时");
        description.append(taskScheduleModel.getMinute()).append("分");
        description.append(taskScheduleModel.getSecond()).append("秒");
        description.append("执行");
        break;
      case 3: // 按每周
        if (taskScheduleModel.getDayOfWeeks() != null
            && taskScheduleModel.getDayOfWeeks().length > 0) {
          String days = "";
          for (int i : taskScheduleModel.getDayOfWeeks()) {
            days += "周" + i;
          }
          description.append("每周的").append(days).append(" ");
        }
        description.append(taskScheduleModel.getHour()).append("时");
        description.append(taskScheduleModel.getMinute()).append("分");
        description.append(taskScheduleModel.getSecond()).append("秒");
        description.append("执行");
        break;
      case 4: // 按每月
        if (taskScheduleModel.getDayOfMonths() != null
            && taskScheduleModel.getDayOfMonths().length > 0) {
          String days = "";
          for (int i : taskScheduleModel.getDayOfMonths()) {
            days += i + "号";
          }
          description.append("每月的").append(days).append(" ");
        }
        description.append(taskScheduleModel.getHour()).append("时");
        description.append(taskScheduleModel.getMinute()).append("分");
        description.append(taskScheduleModel.getSecond()).append("秒");
        description.append("执行");
        break;
      default:
        throw new RuntimeException("不支持的任务类型");
    }
    return description.toString();
  }

  /**
   * 根据触发间隔返回cron表达式.
   *
   * @param frequencyInSeconds 触发间隔秒
   * @return cron表达式.
   */
  public static String createCronExpressionBySeconds(long frequencyInSeconds) {
    Map<String, Integer> stringIntegerMap = convertSeconds((int) frequencyInSeconds);
    int hours = stringIntegerMap.getOrDefault("hours", 0);
    int minutes = stringIntegerMap.getOrDefault("minutes", 0);
    int seconds = stringIntegerMap.getOrDefault("seconds", 0);
    TaskScheduleModel taskScheduleModel = new TaskScheduleModel();
    if (hours > 0) {
      taskScheduleModel.setJobType(2);
      taskScheduleModel.setHour(hours);
      taskScheduleModel.setMinute(minutes);
      taskScheduleModel.setSecond(seconds);
      return createCronExpression(taskScheduleModel);
    }
    if (minutes > 0) {
      taskScheduleModel.setJobType(1);
      taskScheduleModel.setMinute(minutes);
      taskScheduleModel.setSecond(seconds);
      return createCronExpression(taskScheduleModel);
    }
    taskScheduleModel.setJobType(0);
    taskScheduleModel.setSecond(seconds);
    return createCronExpression(taskScheduleModel);
  }

  /**
   * 以秒为单位,返回对应的年月日时分秒.
   *
   * @param seconds 秒
   * @return map集合
   */
  public static Map<String, Integer> convertSeconds(int seconds) {
    int minutes = seconds / 60;
    int hours = minutes / 60;
    int days = hours / 24;
    int months = days / 30;
    int years = months / 12;

    seconds %= 60;
    minutes %= 60;
    hours %= 24;
    days %= 30;
    months %= 12;

    Map<String, Integer> result = new HashMap<>();
    result.put("years", years);
    result.put("months", months);
    result.put("days", days);
    result.put("hours", hours);
    result.put("minutes", minutes);
    result.put("seconds", seconds);
    return result;
  }

  /**
   * demo.
   *
   * @param args
   */
  public static void main(String[] args) {
    TaskScheduleModel taskScheduleModel = new TaskScheduleModel();

    System.out.println(createCronExpressionBySeconds(62));

    // 每30s
    taskScheduleModel.setJobType(0);
    taskScheduleModel.setSecond(30);
    String cropExp = createCronExpression(taskScheduleModel);
    System.out.println(cropExp + ":" + createDescription(taskScheduleModel));

    // 按每分钟
    taskScheduleModel.setJobType(1);
    taskScheduleModel.setMinute(1);
    cropExp = createCronExpression(taskScheduleModel);
    System.out.println(cropExp + ":" + createDescription(taskScheduleModel));

    // 执行时间：每天的12时12分12秒
    taskScheduleModel.setJobType(2);
    Integer hour = 12;
    Integer minute = 12;
    Integer second = 12;
    taskScheduleModel.setHour(hour);
    taskScheduleModel.setMinute(minute);
    taskScheduleModel.setSecond(second);
    cropExp = createCronExpression(taskScheduleModel);
    System.out.println(cropExp + ":" + createDescription(taskScheduleModel));

    // 每周的哪几天执行
    taskScheduleModel.setJobType(3);
    Integer[] dayOfWeeks = new Integer[3];
    dayOfWeeks[0] = 1;
    dayOfWeeks[1] = 2;
    dayOfWeeks[2] = 3;
    taskScheduleModel.setDayOfWeeks(dayOfWeeks);
    cropExp = createCronExpression(taskScheduleModel);
    System.out.println(cropExp + ":" + createDescription(taskScheduleModel));

    // 每月的哪几天执行
    taskScheduleModel.setJobType(4);
    Integer[] dayOfMonths = new Integer[3];
    dayOfMonths[0] = 1;
    dayOfMonths[1] = 21;
    dayOfMonths[2] = 13;
    taskScheduleModel.setDayOfMonths(dayOfMonths);
    cropExp = createCronExpression(taskScheduleModel);
    System.out.println(cropExp + ":" + createDescription(taskScheduleModel));
  }
}
```
