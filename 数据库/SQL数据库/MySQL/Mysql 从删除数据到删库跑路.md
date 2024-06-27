# Mysql 删除数据

- MySQL删除数据的方式都有哪些？

- 为什么不建议使用DELETE 删除数据

- 删库后如何跑路

- 总结

## Mysql

MySQL 是一种关系型数据库管理系统，它的数据存储是基于磁盘上的文件系统实现的。MySQL 将数据存储在表中，每个表由一系列的行和列组成。每一行表示一个记录，每一列表示一个字段。表的结构由其列名、数据类型、索引等信息组成。

MySQL 的数据存储采用了多种技术来优化性能和存储效率。以下是 MySQL 数据存储的一些`关键特性`。

### 存储引擎

MySQL 支持多种不同的存储引擎，每种引擎都有不同的性能和存储特性。常见的存储引擎有 InnoDB、MyISAM、Memory 等。不同的存储引擎支持不同的数据存储方式，如 B树索引、哈希索引、全文索引等。

### 数据页

MySQL 使用数据页来管理存储在磁盘上的数据。数据页是 MySQL 存储引擎中最基本的存储单元，通常情况下每个数据页的大小为 16KB。数据页包含多个记录，每个记录对应一行数据。

### 索引

MySQL 使用索引来优化数据的检索效率。索引是一种特殊的数据结构，它能够快速地查找表中的数据。MySQL 支持多种类型的索引，如 B 树索引、哈希索引、全文索引等。B 树索引是 MySQL 中最常用的索引类型，它能够快速地查找表中的数据。

### 事务

MySQL 支持事务，事务可以保证数据的一致性、可靠性和安全性。MySQL 的事务是基于 ACID 模型实现的，它能够确保数据在事务中的操作要么全部成功，要么全部回滚。事务的支持使得 MySQL 在多用户并发访问时能够保证数据的完整性和一致性。

总之，MySQL 的数据存储基于磁盘上的文件系统实现，采用多种技术来优化性能和存储效率，如存储引擎、数据页、索引、事务等。这些特性使得 MySQL 成为一种高性能、可靠和安全的关系型数据库管理系统。

## Mysql删除数据的方式都有哪些？

咱们常用的三种删除方式：**通过 delete、truncate、drop 关键字进行删除**；这三种都可以用来删除数据，但场景不同。

**执行速度 ：drop > truncate >> DELETE**

### 从原理上理解

#### delete

```sql
DELETE from TABLE_NAME where xxx
```

DELETE 属于数据库`DML操作语言`，**只删除数据不删除表的结构，会走事务，执行时会触发trigger**；

**在 InnoDB 中，DELETE其实并不会真的把数据删除，mysql 实际上只是给删除的数据打了个标记为已删除，因此 delete 删除表中的数据时，表文件在磁盘上所占空间不会变小，存储空间不会被释放，只是把删除的数据行设置为不可见。虽然未释放磁盘空间，但是下次插入数据的时候，仍然可以重用这部分空间（重用 → 覆盖）。**

**DELETE执行时，会先将所删除数据缓存到rollback segement中，事务commit之后生效;**

`delete from table_name`删除表的全部数据,对于MyISAM 会立刻释放磁盘空间，InnoDB 不会释放磁盘空间;

对于`delete from table_name where xxx` 带条件的删除, 不管是InnoDB还是MyISAM都不会释放磁盘空间;

delete操作以后使用 `optimize table table_name` 会立刻释放磁盘空间。不管是InnoDB还是MyISAM 。所以要想达到释放磁盘空间的目的，delete以后执行optimize table 操作。

示例：查看表占用硬盘空间大小的SQL语句如下：

（用M做展示单位，数据库名：csjdemo，表名：demo2）

```sql
select concat(round(sum(DATA_LENGTH/1024/1024),2),'M') as table_size 
   from information_schema.tables 
      where table_schema='csjdemo' AND table_name='demo2';
```

然后执行空间优化语句，以及执行后的表Size变化：

```sql
optimize table demo2
```

再看看这张表的大小，就只剩下表结构size了。

delete 操作是一行一行执行删除的，并且同时将该行的的**删除操作日志记录在redo和undo** 表空间中以便进行回滚（rollback)和重做操作，**生成的大量日志也会占用磁盘空间** 。

#### drop

```sql
Drop table Tablename
```

属于数据库`DDL定义语言`，同Truncate，**执行后立即生效，无法找回** 。

**drop table table_name 立刻释放磁盘空间 ，不管是 InnoDB 和 MyISAM; drop 语句将删除表的结构被依赖的约束(constrain)、触发器(trigger)、索引(index);  依赖于该表的存储过程/函数将保留,但是变为 invalid 状态。**

小心使用 drop ，要删表跑路的兄弟，请在订票成功后在执行操作！

可以这么理解，一本书，delete是把目录撕了，truncate是把书的内容撕下来烧了，drop是把书烧了

#### truncate

属于数据库`DDL定义语言`，**不走事务** ，原数据不放到 rollback segment 中，操作不触发 trigger。**执行后立即生效，无法找回** ！

**truncate table table_name 立刻释放磁盘空间 ，不管是 InnoDB和MyISAM**。truncate table其实**有点类似于drop table 然后creat,只不过这个create table 的过程做了优化**，比如表结构文件之前已经有了等等。所以速度上应该是接近drop table的速度;

**truncate 能够快速清空一个表。并且重置auto_increment的值。**

但**对于不同的类型存储引擎需要注意的地方是**：

- 对于`MyISAM`，truncate会重置auto_increment（自增序列）的值为1。而delete后表仍然保持auto_increment。

- 对于`InnoDB`，truncate会重置auto_increment的值为1。delete后表仍然保持auto_increment。但是在做delete整个表之后重启MySQL的话，则重启后的auto_increment会被置为1。

- 也就是说，InnoDB 的表本身是无法持久保存auto_increment。delete表之后 auto_increment仍然保存在内存，但是重启后就丢失了，只能从1开始。实质上重启后的auto_increment会从 SELECT 1+MAX(ai_col) FROM t 开始。

小心使用 truncate，尤其没有备份的时候，如果误删除线上的表，记得及时跑路。

## 为什么不建议使用DELETE 删除数据

了解上面的一些原理之后，可以知道！

- **无法恢复数据**

在使用DELETE操作删除数据时，如果没有**事先备份数据**，一旦误操作就会导致数据无法恢复。因此，在进行任何数据操作之前，最好先备份数据，以便在出现错误时可以轻松地恢复数据。

- **删除数据会导致索引失效，影响查询性能**。

- **删除数据会导致表空间的浪费，因为删除的数据并不会立即释放，而是留在表中，只是被标记为“已删除”**。

- **数据不一致**

如果在删除数据时表中存在外键关联，使用DELETE操作可能会导致其他表中的数据不一致。例如，如果一个表有一个外键，指向另一个表中的一行，如果从主表中删除行而不更新外键引用，则外键引用将成为无效引用。这可能会导致查询时出现错误，或者在更新时导致数据不一致。

## 删库后如何跑路

**一般来说，在生产环境DBA都会在定期生成全量数据的备份，然后开启binlog记录增量数据**。恢复的时候借助数据备份和binlog日志一般情况下是可以很大程度上复原数据，当然一般情况下开发也不会拥有删库的权限，一般都是有删除数据的权限。所以我们在遇到这种紧急情况不能慌，要赶紧去想办法补救。

### 备份

最简单，也是最实用的方式就是在我们接到，清理数据，或者是修改数据的需求时，先将**数据备份**，备份是王道。这样会让我们的数据恢复变得更容易。**一般在企业中，DBA都会有备份脚本，他们会长期定时对数据进行备份，防止发生悲剧。**

### 规范操作

1. 操作前，先备份，不要怕麻烦，出错后就悔不当初了；
2. 删除数据库、表时，不要直接用drop命令，而是重命名到一个专用归档库里；
3. 删除数据时，不要直接用delete或truncate命令，尤其是truncate命令，目前不支持事务，无法回滚；
4. 使用delete命令删除数据时，应当先开启事务，这样误操作时，还是有机会进行回滚；
5. 要大批量删除数据时，可以将这些数据插入到一个新表中，确认无误后再删除。或者把要保留的数据写到新表，然后将表重命名对掉，这样需要注意的是增量数据，不要把新插入的数据丢掉；

### 基本的恢复流程

- 看看是否有办法快速补救（没有可以看下一条）
- 看看是否有定期备份，和binlog日志（没有就凉凉）
- 先备份数据恢复
- 用mysqlbinlog命令将上述的binlog文件导出为sql文件，并剔除其中的drop语句
- 恢复binlog中增量数据的部分

### 补救措施

1. 优先考虑是否能只通过binlog恢复，不能的化，再考虑其它
2. 执行 DROP DATABASE / DROP TABLE 命令误删库表时，如果采用的是共享表空间模式，还有恢复的机会。如果不是，直接从备份文件恢复吧；在共享表空间模式下，误删后立刻杀掉（kill -9）mysql相关进程（mysqld_safe、mysqld），然后尝试从ibdataX文件中恢复数据；
3. 误删除正在运行中的MySQL表ibd或ibdataX文件。利用linux系统的proc文件特点，把该ibd文件从内存中拷出来，再进行恢复，因为此时mysqld实例在内存中是保持打开该文件的，切记这时不要把mysqld实例关闭了。此模式恢复，需要停止线上业务对该实例的写入操作，不再写入新数据，防止丢失新数据。把复制出来的ibd 或 ibdataX文件拷贝回datadir后，重启mysqld进入recovery模式，innodb_force_recovery 选项从 0 - 6 逐级测试，直至能备份出整个实例或单表的所有数据后，再重建实例或单表，恢复数据。
4. 未开启事务模式下，执行delete误删数据。发现问题严重性后，立即将mysqld（以及mysqld_safe）进程杀掉（kill -9），然后再用工具将表空间数据读取出来。因为执行delete删除后，实际数据并没有从磁盘清除，只是先打上deleted-mark标签，后续再统一清理，因此快速杀掉进程可以防止数据被物理删除。
5. 执行truncate误清整张表。如果没使用共享表空间模式的话，直接使用备份恢复和binlog恢复。
6. 执行不带where条件的update，或者update错数据。数据规模大没法补救的话，也只能通过走备份恢复和binlog恢复。

### 相关操作

1. 查看是否开启binlog日志
   
   ```sql
   # log_bin是ON，就说明打开了 OFF就是关闭状态。
   show variables like 'log_bin';
   # log_bin相关的内容都能查到
   show variables like '%log_bin%';
   ​
   # 设置开启log_bin 一般情况下都是通过配置进行设置
   SET SQL_LOG_BIN=1
   ```

2. binlog日志位置
   
   ```sql
   show variables like '%datadir%';
   ```

3. 根据binlog日志恢复数据
   
   - cd 到binlog文件目录
   
   - mysql安装目录/mysql/bin/下找到binlog日志解析工具mysqlbinlog
   
   - 通过mysqlbinlog工具命令按照对应时间解析binlog日志内容，输出到新的文件中
     
     该工具也支持过滤指定表的相关操作记录
     
     ```sql
     mysqlbinlog --no-defaults --database=test --start-datetime="2021-11-10 09:00:00" --stop-datetime="2021-11-10 20:00:00" /data/mysql/mysql-bin.000020    > binlog.txt
     ```
   
   - 利用解析出来的sql进行恢复或者根据需要恢复的位置，使用命令进行恢复
     
     ```sql
     mysqlbinlog --start-position=8000 --stop-position=8888 mysql-bin.000020 |mysql -uroot -p123456;
     ```

4. 通过配置文件对binlog 日志进行配置

```bash
# 日志格式
# Statement模式，每一条会修改数据的sql都会记录在binlog中。
# Row模式，5.1.5版本的MySQL才开始支持row,它不记录sql语句上下文相关信息，仅保存哪条记录被修改。
# Mixed模式，一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog，MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种。
# 设置日志格式
binlog_format = mixed
​
# 设置日志路径，需要注意的是该路经需要mysql用户有写权限
log-bin = /data/mysql/logs/mysql-bin.log
​
# 设置binlog清理时间
expire_logs_days = 7
​
# binlog每个日志文件大小
max_binlog_size = 100m
​
# binlog缓存大小
binlog_cache_size = 4m
​
# 最大binlog缓存大小
max_binlog_cache_size = 512m
```

**各位大佬，删库请慎重！！！**

## 总结

在工作当中执行数据库删除的时候一定要慎重再慎重，建议每次进行数据删除的使用最好数据表的备份工作，这样就会大大减少你删除跑路的几率。很多时候不要过于相信自己的动手能力，老虎还有打盹的时候，万一手滑了呢。尽可能养成好的数据库运维习惯，这样会让自己少跌跟头，你的事业才会更加顺利。



## 参考

+ [新来个技术总监：发现谁再用 delete 删数据直接开除！](https://mp.weixin.qq.com/s/t-i8EV_mqu39SMGaZhGsPw)

+ [删库，误清数据怎么办？MySQL数据恢复指南-阿里云开发者社区](https://developer.aliyun.com/article/934972)


