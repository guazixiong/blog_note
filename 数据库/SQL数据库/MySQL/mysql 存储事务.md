# mysql 存储过程使用事务

+ 存储过程外部使用事务失效
+ 存储过程内部调用事务
+ 手动启动事务并控制事务的提交和回滚

## 存储过程外部使用事务失效

存储过程`BatchInsert`,用于批量新增数据

```sql
CREATE DEFINER=`root`@`%` PROCEDURE `BatchInsert`(IN initId INT, IN loop_counts INT)
BEGIN
    DECLARE Var INT;
    DECLARE ID INT;
    SET Var = 0;
    SET ID = initId;
    set autocommit=0; -- 关闭自动提交事务，提高插入效率
    WHILE Var < loop_counts DO
        -- 执行sql语句,以";"结尾
    INSERT INTO `user`(`id`, `user_name`) VALUES (ID, ID);
        SET ID = ID + 1;
        SET Var = Var + 1;
    END WHILE;
    COMMIT;
END
```

外部调用事务

```sql
begin;
call BatchInsert(1,1000);
rollback;
```

存储过程`BatchInsert`照常执行,why?

MySQL中的存储过程（Stored Procedure）在默认情况下是自动提交事务的。这意味着每个存储过程语句都被视为一个独立的事务，并在执行完成后立即提交。

**如果在存储过程中发生了错误，并且你尝试在存储过程内部进行回滚操作，但没有显式地启用事务，那么回滚操作将不会生效。因为每个语句都会自动提交，回滚只会撤销之前的语句，而不会影响已提交的语句。**

解决办法:

+ 存储过程内部使用事务

+ 手动启动事务并控制事务的提交和回滚

### 存储过程内部调用事务

创建`user`表

```sql
CREATE TABLE `user` (
  `id` int NOT NULL AUTO_INCREMENT,
  `user_name` varchar(255) DEFAULT NULL,
  `age` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

准备基础数据

```sql
INSERT INTO `test`.`user`(`id`, `user_name`) VALUES (1, '王五','11');
INSERT INTO `test`.`user`(`id`, `user_name`) VALUES (2, '王五','11');
INSERT INTO `test`.`user`(`id`, `user_name`) VALUES (4, '王五','11');
INSERT INTO `test`.`user`(`id`, `user_name`) VALUES (5, '王五','11');
INSERT INTO `test`.`user`(`id`, `user_name`) VALUES (6, '王五','11');
INSERT INTO `test`.`user`(`id`, `user_name`) VALUES (7, '王五','11');
```

创建存储过程`pre_affair`,入参`in_user_name`,出参`out_rest`(没用到)

操作步骤:

- 删除`id = 5`的数据

- 插入数据,从5开始,由于主键的唯一性,`6`会插入失败,触发`SQLEXCEPTION`,事务回滚

```sql
#将结束符号 改成 ;; 结束
DELIMITER ;;
#创建一个存储过程，名称是pre_affair ，输入变量是 in_user_name 字符串型 ，输出变量是out_rest 整形
CREATE PROCEDURE `pre_affair`(IN in_user_name VARCHAR(255), OUT out_rest INT)
BEGIN
    #定义一个循环变量i，类型是整形，默认是5
    DECLARE i INT DEFAULT 5;
    #定义一个错误的变量,类型是整形，默认是0
    DECLARE t_error INTEGER DEFAULT 0;
    #捕获到sql的错误，就设置t_error为1
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET t_error=1;
    #开启事务
    START TRANSACTION;

    #删除id是5的这条数据
    DELETE FROM `user` WHERE id = 5;
    #循环插入数据，这里是从 id是5插入到id是6为止
    #注意：这里user表的id是5的这条被删除了，因为id是主键，但是id是6的这条本身存在，如果这里插入id是6时，将会发生事务回滚，id是5的这条不会被删除
    WHILE i<7 DO
        #插入一条数据进来
        INSERT INTO `user`(id,user_name)VALUES(i,in_user_name);
        #设置i每次循环加1
        SET i = i + 1;
    END WHILE;

    #如果捕获到错误
    IF t_error=1 THEN
        #回滚
        ROLLBACK;
    ELSE
        #提交
        COMMIT;
    END IF;
    #查看返回的结果,并给到输出的变量中
    SELECT t_error INTO out_rest;
END
;;
#将结束符号 改成 ; 结束
DELIMITER ;
```

执行存储过程,数据还在,没有任何变动

注释事务部分

```sql
CREATE DEFINER=`root`@`%` PROCEDURE `pre_affair`(IN in_user_name VARCHAR(255), OUT out_rest INT)
BEGIN
    #定义一个循环变量i，类型是整形，默认是5
    DECLARE i INT DEFAULT 5;
--     #定义一个错误的变量,类型是整形，默认是0
--     DECLARE t_error INTEGER DEFAULT 0;
--     #捕获到sql的错误，就设置t_error为1
--     DECLARE CONTINUE HANDLER FOR SQLEXCEPTION SET t_error=1;
--     #开启事务
--     START TRANSACTION;

    #删除id是5的这条数据
    DELETE FROM `user` WHERE id = 5;
    #循环插入数据，这里是从 id是5插入到id是6为止
    #注意：这里user表的id是5的这条被删除了，因为id是主键，但是id是6的这条本身存在，如果这里插入id是6时，将会发生事务回滚，id是5的这条不会被删除
    WHILE i<7 DO
        #插入一条数据进来
        INSERT INTO `user`(id,user_name)VALUES(i,in_user_name);
        #设置i每次循环加1
        SET i = i + 1;
    END WHILE;

    #如果捕获到错误
    IF t_error=1 THEN
        #回滚
        ROLLBACK;
    ELSE
        #提交
        COMMIT;
    END IF;
    #查看返回的结果,并给到输出的变量中
    SELECT t_error INTO out_rest;
END
```

```bash
Duplicate entry '6' for key 'user.PRIMARY'
```

> user表数据,`id = 5`数据被删除后从新插入,`age`为空,`id = 6`报错,主键重复,没有继续执行

### 手动启动事务并控制事务的提交和回滚

在调用存储过程的代码中，你可以将外部的事务传递给存储过程，让存储过程在外部事务的

范围内执行。具体的传递方式取决于编程语言和数据库连接库的支持。

以下是一个Java代码的示例，展示如何在使用JDBC调用存储过程时将外部事务传递给它：

```java
Connection connection = null;
try {
    connection = dataSource.getConnection();
    connection.setAutoCommit(false);  // 关闭自动提交

    CallableStatement callableStatement = connection.prepareCall("{CALL your_procedure()}");
    // 设置存储过程的参数...

    callableStatement.execute();

    connection.commit();  // 提交事务
} catch (SQLException e) {
    connection.rollback();  // 回滚事务
    // 处理异常
} finally {
    if (connection != null) {
        connection.close();
    }
}
```
