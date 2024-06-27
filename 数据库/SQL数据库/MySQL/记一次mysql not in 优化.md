## 前提

表结构

```sql
CREATE TABLE `read_info` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `real_name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `age` int DEFAULT NULL,
  `address` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `sex` char(2) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `card` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `phone` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `school` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `role` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `register_time` datetime DEFAULT NULL,
  `register_ip` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `invite_id` int DEFAULT NULL,
  `login_time` date DEFAULT NULL,
  `login_ip` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC;
```

```sql
CREATE TABLE `info_intermediate_task` (
  `id` int NOT NULL AUTO_INCREMENT,
  `name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `real_name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `age` int DEFAULT NULL,
  `address` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `sex` char(2) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `card` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `phone` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `school` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `role` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `register_time` datetime DEFAULT NULL,
  `register_ip` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `invite_id` int DEFAULT NULL,
  `login_time` date DEFAULT NULL,
  `login_ip` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL,
  `sync_identity` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '唯一标识',
  `sync_state` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '同步状态\r\n1:未同步\r\n2:已同步\r\n3:同步失败',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci ROW_FORMAT=DYNAMIC;
```

`read_info` 表中有5000条数据

`info_intermediate_task`表中有30000条数据,`sync_state = 2`只有5条数据

**sql编写要求,从`read_info`表中获取未同步至`info_intermediate_task`的值,通过唯一主键标识`sync_identity`做去重**

## 优化

初版定义sql语句

```sql
SELECT
    id AS sync_identity,
    NAME,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip
FROM
    read_info 
WHERE
    id NOT IN (SELECT sync_identity FROM info_intermediate_task 
    where sync_state = 2)
```

执行时间为50.72s,这也太慢了,不满意,经百度,在数据量大的情况,有以下优化方式

- not in 优化为left join
- not exist
- 增加分页查询
- 增加索引

### 添加索引

- 针对`info_intermediate_task` 表添加唯一索引`sync_identity`

### 更改为left join

```sql
SELECT
    r.id,
    r.id AS sync_identity,
    r.NAME,
    r.real_name,
    r.age,
    r.address,
    r.sex,
    r.card,
    r.phone,
    r.school,
    r.role,
    r.register_time,
    r.register_ip,
    r.invite_id,
    r.login_time,
    r.login_ip
FROM
    read_info AS r
LEFT JOIN info_intermediate_task AS i ON r.id = i.sync_identity and i.sync_state = 2
WHERE
    i.sync_identity IS NULL;
```

执行时间为46.62s,很好,有效,但还不够

### 基于left join ,增加limit

```sql
select * from (
SELECT
    id,
    id AS sync_identity,
    NAME,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip,
    "2" AS sync_state,
    id AS auto_num,
    "10" AS page_num 
FROM
    read_info) r 
LEFT JOIN info_intermediate_task AS i ON r.id = i.sync_identity and i.syncstate = 2
WHERE
    i.sync_identity IS NULL limit 1000;
```

执行时间为42.12s,马大大,还是不太够

### 修改为not exist

```sql
SELECT
    id AS sync_identity,
    NAME,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip
FROM
    read_info AS r
WHERE
    NOT EXISTS (
        SELECT 1
        FROM info_intermediate_task AS i
        WHERE i.sync_identity = r.id and i.sync_state = 2
    )
```

执行时间为45.32s,添加limit 限制呢?

```sql
SELECT
    id AS sync_identity,
    NAME,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip
FROM
    read_info AS r
WHERE
    NOT EXISTS (
        SELECT 1
        FROM info_intermediate_task AS i
        WHERE i.sync_identity = r.id and i.sync_state = 2
    ) limit 1000;
```

执行时间为42.12s,不够,还不够

### 临时表

```sql
CREATE TEMPORARY TABLE temp_sync_identity AS
SELECT sync_identity
FROM info_intermediate_task;

SELECT
    r.id,
    r.id AS sync_identity,
    r.NAME,
    r.real_name,
    r.age,
    r.address,
    r.sex,
    r.card,
    r.phone,
    r.school,
    r.role,
    r.register_time,
    r.register_ip,
    r.invite_id,
    r.login_time,
    r.login_ip,
    "1" AS sync_state,
    r.id AS auto_num,
    "10" AS page_num
FROM
    read_info r
LEFT JOIN
    temp_sync_identity i ON r.id = i.sync_identity and i.sync_state = 2
WHERE
    i.sync_identity IS NULL
ORDER BY
    r.id;
```

执行结果49s,不太行哦

---

傻了傻了,怎么回事呢,优化完,怎么还有100s左右呢,就相当于5000条数据对比5条数据,怎么会辣么辣么慢,不行,在想想

---

### 试试常量怎么样?!!!

```sql
select * from (
SELECT
    id,
    id AS sync_identity,
    NAME,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip
FROM
    read_info) r where sync_identity not in (1,2,3,4,5) limit 1000;
```

执行速度0.179s,我丢,不是吧阿 sir,虽然1,2,3,4,5 是我瞎诌的,但这是不是太夸张

通过explain 查看一哈

![](%E8%AE%B0%E4%B8%80%E6%AC%A1mysql%20not%20in%20%E4%BC%98%E5%8C%96.assets/202305272131903.png)

通过子查询查询,type为all,走的是全表扫描

![](%E8%AE%B0%E4%B8%80%E6%AC%A1mysql%20not%20in%20%E4%BC%98%E5%8C%96.assets/202305272131975.png)

所以来吧,先写成个常量在查询

```sql
SELECT
    @myConstant := GROUP_CONCAT( sync_identity SEPARATOR ',' ) AS sync_identity 
FROM
    info_intermediate_task 
WHERE
    sync_state = 2;

select * from (
SELECT
    id AS sync_identity,
    name,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip,
    "2" AS sync_state,
    id AS auto_num,
    "10" AS page_num
FROM
    read_info
) r where r.sync_identity not in (@myConstant) LIMIT 1000;
```

执行时间0.418s,偶吼吼,那可真的是相当不错呢

## 痛苦才刚刚开始

由于业务的问题,info_intermediate_task 表中sync_state = 2 值是不确定的,

### 痛苦1: 查询数据为空

**当** `select * from user where id not in ()`**中not in 结果集为空时,没有任何值与之匹配，因此所有行都会被排除，结果为空**, 并不是以为的全部查出!!!!

所以一次修改,**基于msql提供的case语句进行改写**

```sql
SELECT * FROM employees 
WHERE 
    CASE 
        WHEN <your_constant_list> IS NOT NULL THEN department NOT IN (<your_constant_list>)
        ELSE 1 -- 返回所有数据
    END;
```

在这个查询中，你需要将`<your_constant_list>`替换为你的常量值列表。如果常量值列表不为空，那么查询将使用NOT IN操作符，并排除与列表中的值匹配的行。如果常量值列表为空，那么查询将忽略NOT IN操作符，并返回所有数据。

改写结果:

```sql
SELECT
    @myConstant := GROUP_CONCAT( sync_identity SEPARATOR ',' ) AS sync_identity 
FROM
    info_intermediate_task 
WHERE
    sync_state = 2;

select * from (
SELECT
    id AS sync_identity,
    name,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip
FROM
    read_info
) r 
WHERE 
    CASE 
        WHEN @myConstant IS NOT NULL THEN r.sync_identity NOT IN (@myConstant)
        ELSE 1 -- 返回所有数据
    END 
    limit 1000
```

### 痛苦2:查询去重数据不全,只有部分

```sql
SELECT
    @myConstant := GROUP_CONCAT( sync_identity SEPARATOR ',' ) AS sync_identity 
FROM
    info_intermediate_task 
WHERE
    sync_state = 2;
```

**当`@myConstant`过大时,常量值会被截取,因为 `GROUP_CONCAT `函数在默认情况下有一个最大长度限制。这个限制可以通过系统变量 `group_concat_max_len` 来设置。**

**如果连接后的字符串长度超过了 group_concat_max_len 的值，那么结果将被截断，并且只显示部分值。**

不行不行,不太行,常量的方法不太行,必须要换,不然业务乱套了

## 继续学习

### 索引优化

- 为read_info表增加覆盖索引 ,包含查询中涉及到的所有列,以减少表的访问次数

```sql
CREATE INDEX idx_read_info_id ON read_info (id, name, real_name, age, address, sex, card, phone, school, role, register_time, register_ip, invite_id, login_time, login_ip);

SELECT
    id,
    id AS sync_identity,
    NAME,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip,
    "1" AS sync_state,
    id AS auto_num,
    "10" AS page_num 
FROM
    read_info 
WHERE
    id NOT IN ( SELECT sync_identity FROM info_intermediate_task 
where sync_state = 2 ) ;
```

![](https://pbad-1305768026.cos.ap-nanjing.myqcloud.com/202305272131081.png)

增加limit限制数据

read_info内含3000条待同步数据

info_intermediate_task内含27000条待传输数据

- limit 100 : 10s
- limit 200 : 12s
- limit 500 : 15s
- limit 600 : 16s
- limit 1000 : 18s
- limit 3000 : 30s

```sql
SELECT
    * 
FROM
    (
SELECT
    id AS sync_identity,
    name,
    real_name,
    age,
    address,
    sex,
    card,
    phone,
    school,
    role,
    register_time,
    register_ip,
    invite_id,
    login_time,
    login_ip,
    "1" AS sync_state,
    id AS auto_num,
    "10" AS page_num 
FROM
    read_info 
    ) r 
WHERE
    r.sync_identity NOT IN ( SELECT sync_identity FROM info_intermediate_task )
 limit 1000;
```

束手无策

## 峰会路转

记: mysql 突然无法连接成功,由于我是远程调用服务器,而为了不影响公司数据,使用了自己服务器的mysql,重启mysql后,继续研究,mysql又又又断了,就纳闷了

mysql突然断开常见的可能原因:

- **网络问题**：网络故障是最常见的导致MySQL连接断开的原因之一。这可能包括网络中断、防火墙配置、路由器问题等。
- **MySQL服务器问题**：MySQL服务器本身可能出现问题，例如崩溃、重启、资源耗尽等。这可能导致连接中断或无法建立新的连接。
- **配置问题**：错误的配置可能导致连接断开。例如，连接超时设置过短、最大连接数限制、并发连接数限制等。
- **客户端问题**：客户端应用程序可能存在问题，例如连接池设置不当、代码逻辑错误等。这可能导致连接异常或连接无法保持。
- **资源限制**：服务器资源限制，如内存不足、磁盘空间不足、CPU负载过高等，可能导致MySQL连接中断。
- **日志错误**：MySQL日志文件错误或满了，可能导致连接问题。

净检查,网络,配置,日志存储,客户端都好好的,当打开内存时

![](%E8%AE%B0%E4%B8%80%E6%AC%A1mysql%20not%20in%20%E4%BC%98%E5%8C%96.assets/202305272131902.png)

只有266M可用,难道是因为这个问题????

数据备份,切到小伙伴4核8G的服务器,重新执行

- 1核2G的服务器,可用内存300M,执行速度30s
- 4核8G的服务器,可用内存5100M,执行速度2.5s

**原来不是我菜,是我穷!!!**
