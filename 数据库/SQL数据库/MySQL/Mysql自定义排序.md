---
title: Mysql自定义排序
category: 数据库
tags: Mysql
updatedAt: 2023-02-15T15:12:08.799Z
createdAt: 2023-02-15T15:11:17.157Z
---

# Mysql自定义排序（FIELD、INSTR、LOCATE）

```sql
# FIELD(str,str1,str2,str3,...)
# FIELD 使用时需要把字段放在前面，排序的顺序每个值是一个参数
select * from t_test order by FIELD(id,2,4,1,3)

# INSTR(str,substr)
# 第一个参数是字符串。根据参数写SQL
select * from t_test order by INSTR('2,4,1,3',id)

# LOCATE(substr,str)
# 第一个参数是字段。后面是字符串：
select * from t_test order by LOCATE(id,'2,4,1,3')

# 相比较FIFLD的多参数，INSTR和LOCATE的两个参数使用时更方便
```

<!-- more -->

## FIELD

先来看看参数，field函数的参数

```sql
FIELD(str,str1,str2,str3,...)
```

参数就是N个字符串，排序的字段名也是一个字符串。根据参数写SQL：

```sql
select * from t_test order by FIELD(id,2,4,1,3)
```

运行后结果是正确的，顺序：2,4,1,3



如果把字段名放到后面：

```sql
select * from t_test order by FIELD(2,4,1,3,id)
```

结果顺序变成了：1,3,4,2。这个结果很显然是不对的。

结论：FIELD 使用时需要把字段放在前面，排序的顺序每个值是一个参数。

## INSTR

看一下参数：

```sql
INSTR(str,substr)
```

第一个参数是字符串。根据参数写SQL：

```sql
select * from t_test order by INSTR('2,4,1,3',id)
```

运行后结果是正确的，顺序：2,4,1,3

## LOCATE

看一下参数：

```sql
LOCATE(substr,str)
```

和INSTR一样是两个参数，顺序和INSTR是相反的。根据参数写SQL：

```sql
select * from t_test order by LOCATE(id,'2,4,1,3')
```

运行后结果是正确的，顺序：2,4,1,3

注：相比较FIFLD的多参数，INSTR和LOCATE的两个参数使用时更方便。

其它：自定义排序也是可以使用ASC和DESC的，DESC的结果就是ASC的反向，这很好理解。如果数据中出现了我们没有自定义排序的值，那么这些没有出现的值会排在上面，DESC时就排在下面。

## mybatis plus 自定义排序

```java
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.lambda()
        // 姓名
        .eq(StringUtils.isNotEmpty(entity.getUserName()), User::getUserName, entity.getUserName())
        // 过滤年龄
        .notIn(!CollectionUtils.isEmpty(entity.getAges()),User::getAges,entity.getAges())
        .orderByDesc(TradeBusinessOrder::getHappenDate);
    if (StringUtils.isNotEmpty(entity.getDate())) {
        queryWrapper.apply("DATE(date) = DATE('"+ entity.getDate().toString() +"')");
    }
    /**
    * 排序规则
    * 1.日期倒序
    * 2.自定义年龄排序
    */
    String fieldSql = " Field(age,'" + 1 + "','" + 4 + "','" + 5 + "','"
        + 2 + "','" + 3 + "')";
    queryWrapper.orderByAsc(fieldSql);
    List<User> records = this.selectList(queryWrapper);
```

LambdaQueryWrapper 无法使用自定义排序,需要使用QueryWrapper,

```java
    /**
     * 排序规则
     * 1.日期倒序
     * 2.自定义年龄排序
     */
    queryWrapper.orderByDesc(TradeBusinessOrder::getHappenDate);
    String fieldSql = " Field(age,'" + 1 + "','" + 4 + "','" + 5 + "','"
        + 2 + "','" + 3 + "')";
    queryWrapper.orderByAsc(fieldSql)
```