# MySql插入百万级数据的方法

+ 存储过程插入
+ 代码工具
+ mysqlImport工具

## 存储过程插入

```sql
# 操作数据库
use test_db;
# 创建存储过程
DROP PROCEDURE if EXISTS BatchInsert;
delimiter $$
CREATE PROCEDURE BatchInsert(IN initId INT, IN loop_counts INT)
BEGIN
    DECLARE Var INT;
    DECLARE ID INT;
    SET Var = 0;
    SET ID = initId;
    set autocommit=0; -- 关闭自动提交事务，提高插入效率
    WHILE Var < loop_counts DO
    	-- 执行sql语句,以";"结尾
        INSERT INTO `xxx_audit_order` (`product_no`,`xxx_channel_code`,`business_no`,`xxx_product_id`,`xxx_product_name`,`xxx_audit_no`,`xxx_audit_status`,`inspection_report_no`,`audit_report_no`,`re_audit_count`,`inspector_id`,`remark`,`delete_dt`,`create_by`,`create_dt`,`update_by`,`update_dt`,`xxx_link`,`service_standard_no`,`depth_inspection`,`execute_channel`,`seller_type`) 
        VALUES (CONCAT('20220704', 100000000000 + ID),106,'RS20190719143225916727',26958,'荣耀 Play',CONCAT('C0', 512201907191454553491 + ID),FLOOR(RAND()*10) % 4,'R1152109544189558784','R1152216911870734336',2,0,null,0,6532,UNIX_TIMESTAMP() + ID ,0,Now(),FLOOR(RAND()*10) % 3,'',0,1,null);
        SET ID = ID + 1;
        SET Var = Var + 1;
    END WHILE;
    COMMIT;
END$$;

delimiter ;  -- 界定符复原为默认的分号
CALL BatchInsert(1, 1000000);  -- 调用存储过程

```

## 代码插入

在应用代码中（我们自己造数据的测试代码）使用MySQL的batch [insert](https://so.csdn.net/so/search?q=insert&spm=1001.2101.3001.7020)方式插入：

```sql
insert into table(id,col) values(1,'foo'),(2,'bar')…
```

## mysql下mysqlImport工具

mysqlimport 客户端提供了 `LOAD DATA INFILEQL`语句的一个命令行接口。

mysqlimport 的大多数选项直接对应 LOAD DATA INFILE 子句。

### 常用命令

| 选项                         | 功能                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| -d or --delete               | 新数据导入数据表中之前删除数据数据表中的所有信息             |
| -f or --force                | 不管是否遇到错误，mysqlimport将强制继续插入数据              |
| -i or --ignore               | mysqlimport跳过或者忽略那些有相同唯一 关键字的行， 导入文件中的数据将被忽略。 |
| -l or -lock-tables           | 数据被插入之前锁住表，这样就防止了， 你在更新数据库时，用户的查询和更新受到影响。 |
| -r or -replace               | 这个选项与－i选项的作用相反；此选项将替代 表中有相同唯一关键字的记录。 |
| --fields-enclosed- by= char  | 指定文本文件中数据的记录时以什么括起的， 很多情况下 数据以双引号括起。 默认的情况下数据是没有被字符括起的。 |
| --fields-terminated- by=char | 指定**各个数据的值之间的分隔符**，在句号分隔的文件中， 分隔符是句号。您可以用此选项指定数据之间的分隔符。 默认的分隔符是跳格符（Tab） |
| --lines-terminated- by=str   | 此选项指定**文本文件中行与行之间数据的分隔字符串 或者字符**。 默认的情况下mysqlimport以newline为行分隔符。 您可以选择用一个字符串来替代一个单个的字符： 一个新行或者一个回车。 |

### demo

从文件 dump.txt 中将数据导入到 mytbl 数据表中, 可以使用以下命令：

```sql
$ mysqlimport -u root -p --local mytbl dump.txt
password *****
```

mysqlimport 命令可以指定选项来设置指定格式,命令语句格式如下：

```sql
$ mysqlimport -u root -p --local --fields-terminated-by=":" \
   --lines-terminated-by="\r\n"  mytbl dump.txt
password *****
```

mysqlimport 语句中使用 `--columns` 选项来设置列的顺序：

```sql
$ mysqlimport -u root -p --local --columns=b,c,a \
    mytbl dump.txt
password *****
```

## 参考

- [开心档之MySQL 导入数据 - 掘金 (juejin.cn)](https://juejin.cn/post/7229540359272284215)