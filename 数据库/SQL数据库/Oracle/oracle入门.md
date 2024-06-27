# Oracle入门

## Oracle用法

```sql
-- 创建表空间
create tablespace waterboss
datafile 'd:\environment\oracle\workspace\waterboss.dbf'
size 100m
autoextend on
next 10m;

-- 创建用户wateruser,密码itcast,表空间waterboss
create user wateruser
IDENTIFIED by itcast
DEFAULT tablespace waterboss;

-- wateruser赋予dba权限
GRANT dba to wateruser;
```

### 批量插入

#### mysql批量插入

```sql
# mysql批量插入
INSERT INTO target_table (column1, column2, column3)
VALUES
(value1_1, value1_2, value1_3),
(value2_1, value2_2, value2_3),
(value3_1, value3_2, value3_3);

# mysql批量插入
INSERT INTO target_table (column1, column2, column3)
SELECT column1,column2,column3 FROM your_table WHERE condition;
```

#### oracle批量插入

```sql
# oracle 批量插入
INSERT ALL
INTO target_table (column1, column2, column3) VALUES (value1_1, value1_2, value1_3)
INTO target_table (column1, column2, column3) VALUES (value2_1, value2_2, value2_3)
INTO target_table (column1, column2, column3) VALUES (value3_1, value3_2, value3_3)
SELECT * FROM dual;

# oracle批量插入
INSERT INTO target_table (col1, col2, col3)
SELECT col1,col2,col3 FROM your_table WHERE condition;
```

在 Oracle 中，`INSERT ALL INTO ... SELECT` 语法中的 `SELECT * FROM dual` 实际上是一个占位符，用于结束 `INSERT ALL INTO ... SELECT` 语句的语法结构。在这种情况下，`SELECT * FROM dual` 是一个虚拟的查询，它不会产生实际的数据结果，仅用于结束 `INSERT ALL INTO ... SELECT` 语句的语法。

#### oracle条件批量插入

```sql
# oracle有条件的批量插入
INSERT [ ALL | FIRST ]
    WHEN condition1 THEN
        INTO table_1 (column_list ) VALUES (value_list)
    WHEN condition2 THEN 
        INTO table_2(column_list ) VALUES (value_list)
    ELSE
        INTO table_3(column_list ) VALUES (value_list)
Subquery;
```

如果指定了`ALL`关键字，则Oracle将在`WHEN`子句中评估每个条件。如果条件评估/计算为`true`，则Oracle执行相应的`INTO`子句。

但是，当指定`FIRST`关键字时，对于由子查询返回的每一行，Oracle都会从`WHEN`子句的上下方向评估每个条件。 如果Oracle发现条件的计算结果为`true`，则执行相应的`INTO`子句并跳过给定行的后续`WHEN`子句。

> 使用`FIRST`时,满足条件后跳出后续判断条件;
>
> 使用`ALL`时,会继续穿透执行,直至结束.

示例以下`CREATE TABLE`语句创建三个表：`small_orders`，`medium_orders`和`big_orders`，它们具有相同的结构：

```sql
CREATE TABLE small_orders (
    order_id NUMBER(12) NOT NULL,
    customer_id NUMBER(6) NOT NULL,
    amount NUMBER(8,2) 
);

CREATE TABLE medium_orders AS
SELECT *
FROM small_orders;

CREATE TABLE big_orders AS
SELECT *
FROM small_orders;
```

以下条件Oracle `INSERT ALL`语句根据订单金额将订单数据插入到三个表：`small_orders`，`medium_orders`和`big_orders`之中：

```sql
# 小于10000的插入small_orders
# 不大于30000的插入medium_orders
# 大于30000的插入big_orders
INSERT ALL
   WHEN amount < 10000 THEN
      INTO small_orders
   WHEN amount <= 30000
      INTO medium_orders
   WHEN amount > 30000 THEN
      INTO big_orders
 SELECT order_id,
        customer_id,
        (quantity * unit_price) as amount
 FROM orders
 INNER JOIN order_items USING(order_id);

```

通过使用`ELSE`子句插入到`big_orders`表中，这样也可以达到相同的结果，如下所示：

```sql
# 小于10000的插入small_orders
# 不大于30000的插入medium_orders
# 大于30000的插入big_orders
INSERT ALL
   WHEN amount < 10000 THEN
      INTO small_orders
   WHEN amount <= 30000
      INTO medium_orders
   ELSE
      INTO big_orders
  SELECT order_id,
         customer_id,
         (quantity * unit_price) amount
  FROM orders
  INNER JOIN order_items USING(order_id);
```

> 以上内容均使用`ALL`来控制实现,当amount价格为9000时,会同时插入数据到`small_orders`和`medium_orders`表中.
>
> 使用`FIRST`,当amount价格为9000时,在满足第一个结果为`TRUE`的条件后,会主动跳出其他条件,仅插入数据到`small_orders`
>
> ```sql
> # 小于10000的插入small_orders
> # 不大于30000的插入medium_orders
> # 大于30000的插入big_orders
> INSERT FIRST
>    WHEN amount < 10000 THEN
>       INTO small_orders
>    WHEN amount <= 30000
>       INTO medium_orders
>    ELSE
>       INTO big_orders
>   SELECT order_id,
>          customer_id,
>          (quantity * unit_price) amount
>   FROM orders
>   INNER JOIN order_items USING(order_id);
> ```

### TO_CHAR

`TO_CHAR()` 是 Oracle 数据库中用于将日期、数字或其他数据类型转换为字符型的函数。它的基本语法如下：

```sql
TO_CHAR(value, [format], [nls_parameter])
```

- `value`：要转换为字符型的值，可以是日期、数字或其他数据类型。
- `format`：可选参数，用于指定转换后的字符型的格式。对于日期值，可以使用不同的格式化模式，比如 `'YYYY-MM-DD HH24:MI:SS'`。
- `nls_parameter`：可选参数，用于指定 NLS（National Language Support）参数，用于定义日期和数字格式。

下面是一些常见用法：

1. 将日期转换为字符型：这会将当前日期时间转换为指定格式的字符串，例如 `2024-05-07 15:30:00`。
   ```sql
   # 年-月-日
   SELECT TO_CHAR(sysdate, 'YYYY-MM-DD') FROM dual;
   
   # 日-月-年
   SELECT TO_CHAR( SYSDATE, 'MM/DD/YYYY' ) FROM dual;
   
   # 年-月-日 12时-分-秒
   SELECT TO_CHAR(sysdate, 'YYYY-MM-DD HH:MI:SS') FROM dual;
   
   # 年-月-日 24时-分-秒
   SELECT TO_CHAR(sysdate, 'YYYY-MM-DD HH24:MI:SS') FROM dual;
   
   ```

2. 格式化数字：这会将数字 `12345.67` 格式化为 `12,345.67`。
   ```sql
   SELECT TO_CHAR(12345.67, '999,999.99') FROM dual;
   ```

3. 指定 NLS 参数,这会将日期转换为西班牙语格式的字符串。
   ```sql
   SELECT TO_CHAR(sysdate, 'YYYY-MM-DD HH24:MI:SS', 'NLS_DATE_LANGUAGE = Spanish') FROM dual;
   ```

   使用默认格式：如果省略格式参数，`TO_CHAR()` 将使用默认的日期或数字格式

   ```sql
   SELECT TO_CHAR(sysdate) FROM dual; -- 默认格式为 'DD-MON-YY'
   ```





## Q&A

### Oracle和Mysql的区别在哪里?