# MySQL中查看正在执行的SQL并停止

## 查看当前正在执行的SQL

```sql
show full processlist;
```

![img](MySQL%E4%B8%AD%E6%9F%A5%E7%9C%8B%E6%AD%A3%E5%9C%A8%E6%89%A7%E8%A1%8C%E7%9A%84SQL%E5%B9%B6%E5%81%9C%E6%AD%A2.assets/202305042330452.png)

在结果中可以看到（id）、User（操作人的账户）、Host（操作人的ip地址和端口号）、db（操作的数据库）、（Command）操作的指令、Time（执行时间）、State（状态）、Info（信息）。

如果是一条查询语句Command中的值就是Query，Info中的值就是执行的SQL语句。

* `id`:操作id
* `User`:操作人账户
* `Host`:操作人ip地址:端口号
* `db`:操作数据库
* `Command`:操作命令
* `Time`:执行时间
* `State`:状态
* `Info`:信息

## 查询当前正在运行的所有事务

```sql
SELECT * FROM  information_schema.innodb_trx;
```

* `trx_state`:事务状态
* `trx_mysql_thread_id`:执行事务的线程id
* `trx_query`:查询事务的SQL
* `trx_isolation_level`:事务隔离级别

## 结束操作

```sql
kill {id}
```

`{id}` 为 操作`processlist` 中`id`或者`information_schema.innodb_trx`中`trx_mysql_thread_id`