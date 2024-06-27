# Mysql快速生成千万数据

为了创建一个包含大约20个字段的MySQL表，并编写一个存储过程插入大约2亿条数据，我们可以先定义表结构，然后编写存储过程插入数据。以下是一个示例：

> 请注意，插入2亿条数据可能会非常耗时，并且可能会占用大量的数据库资源。建议在执行此操作之前确保有足够的资源，并在可能的情况下分批插入数据，以避免锁表或其他性能问题。
>
> 此外，由于插入2亿条数据是一个非常耗时的操作，建议在实际操作中使用并发插入或批量插入来提高效率。例如，可以编写多个存储过程，每个过程插入一部分数据，或者在应用程序中使用多线程并发插入。

## 创建数据库和表

```sql
CREATE DATABASE IF NOT EXISTS mycat_test;
USE mycat_test;

CREATE TABLE IF NOT EXISTS test_table (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    col1 VARCHAR(255),
    col2 VARCHAR(255),
    col3 INT,
    col4 DECIMAL(10,2),
    col5 DATE,
    col6 DATETIME,
    col7 TIMESTAMP,
    col8 BOOLEAN,
    col9 CHAR(10),
    col10 TEXT,
    col11 BLOB,
    col12 MEDIUMTEXT,
    col13 ENUM('val1', 'val2', 'val3'),
    col14 SET('val1', 'val2', 'val3'),
    col15 SMALLINT,
    col16 BIGINT,
    col17 FLOAT,
    col18 DOUBLE,
    col19 YEAR,
    col20 TIME
);

```

## (不推荐)改造一

### 创建批量插入数据的存储过程

```sql
DELIMITER //

CREATE PROCEDURE InsertBatchData(IN start_id INT, IN end_id INT)
BEGIN
    DECLARE i INT DEFAULT start_id;

    WHILE i <= end_id DO
        INSERT INTO test_table (col1, col2, col3, col4, col5, col6, col7, col8, col9, col10, col11, col12, col13, col14, col15, col16, col17, col18, col19, col20) 
        VALUES (
            CONCAT('value_', i), 
            CONCAT('data_', i), 
            i, 
            i * 0.01, 
            CURDATE(), 
            NOW(), 
            CURRENT_TIMESTAMP, 
            i % 2, 
            LEFT(UUID(), 10), 
            REPEAT('text', 10), 
            RANDOM_BYTES(100), 
            REPEAT('mediumtext', 10), 
            'val1', 
            'val2', 
            MOD(i, 32767), 
            i * 1000, 
            i * 0.1, 
            i * 0.001, 
            YEAR(CURDATE()), 
            CURTIME()
        );
        SET i = i + 1;
    END WHILE;
END //

DELIMITER ;

```

### 创建控制存储过程

```sql
DELIMITER //

CREATE PROCEDURE InsertAllData()
BEGIN
    DECLARE start_id INT DEFAULT 1;
    DECLARE end_id INT;
    DECLARE batch_size INT DEFAULT 100000;
    DECLARE total_records INT DEFAULT 200000000;

    WHILE start_id <= total_records DO
        SET end_id = start_id + batch_size - 1;
        CALL InsertBatchData(start_id, end_id);
        SET start_id = end_id + 1;
    END WHILE;
END //

DELIMITER ;

```

### 调用控制存储过程

```sql
CALL InsertAllData();
```

通过创建控制存储过程 `InsertAllData`，它会循环调用批量插入数据的存储过程 `InsertBatchData`，直到插入了所需的全部数据。这样可以避免一次性插入大量数据导致的性能问题，并且可以直接在MySQL中完成所有操作。

等待数据插入完毕,实测,2c4g的5分钟才插入了12w条数据,不太行

## (推荐)改造二

+ **关闭自动提交**：在大批量插入数据时，关闭自动提交可以显著提高性能。

+ **批量插入**：使用单个`INSERT`语句插入多行数据。

+ **禁用索引**：在插入数据前禁用索引，插入完成后再重新启用索引。

+ **调整MySQL配置**：优化MySQL的配置参数，如`innodb_buffer_pool_size`、`bulk_insert_buffer_size`等。

### 关闭自动提交，使用批量插入

```sql
DELIMITER //

CREATE PROCEDURE InsertAllData()
BEGIN
    DECLARE start_id INT DEFAULT 1;
    DECLARE end_id INT;
    DECLARE batch_size INT DEFAULT 1000; -- 每批次插入的行数
    DECLARE total_records INT DEFAULT 20000000; -- 2千万条数据

    WHILE start_id <= total_records DO
        SET end_id = start_id + batch_size * 1000 - 1;
        IF end_id > total_records THEN
            SET end_id = total_records;
        END IF;
        CALL InsertBatchData(start_id, end_id, batch_size);
        SET start_id = end_id + 1;
    END WHILE;
END //

DELIMITER ;

```

### 创建控制存储过程

```sql
DELIMITER //

CREATE PROCEDURE InsertBatchData(IN start_id INT, IN end_id INT)
BEGIN
    DECLARE i INT DEFAULT start_id;

    SET autocommit = 0;
    
    WHILE i <= end_id DO
        INSERT INTO test_table (col1, col2, col3, col4, col5, col6, col7, col8, col9, col10, col11, col12, col13, col14, col15, col16, col17, col18, col19, col20) 
        VALUES 
            (CONCAT('value_', i), CONCAT('data_', i), i, i * 0.01, CURDATE(), NOW(), CURRENT_TIMESTAMP, i % 2, LEFT(UUID(), 10), REPEAT('text', 10), RANDOM_BYTES(100), REPEAT('mediumtext', 10), 'val1', 'val2', MOD(i, 32767), i * 1000, i * 0.1, i * 0.001, YEAR(CURDATE()), CURTIME());
        SET i = i + 1;
    END WHILE;

    COMMIT;
    SET autocommit = 1;
END //

DELIMITER ;
```

### **调用控制存储过程**

```
CALL InsertAllData();
```

通过减小每个批次的大小来避免构建的 SQL 语句过长，同时通过关闭自动提交来减少事务开销，从而提高插入速度。在 `InsertBatchData` 存储过程中，使用 `WHILE` 循环逐条插入数据，并在完成后提交事务。

等待执行完毕,目前5分钟插入200w条数据,waiting!!!