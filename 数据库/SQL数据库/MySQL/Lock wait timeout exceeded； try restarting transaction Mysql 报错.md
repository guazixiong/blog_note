# Lock wait timeout exceeded； try restarting transaction

[toc]

## 解决方式

### 方式一:重启mysql服务

### 方式二:kill 掉被锁住的线程

```sql
# 查询语句的系统id
show full processlist;
# 执行sql语句，查看事物表
select * from information_schema.innodb_trx;
# kill {trx_mysql_thread_id} 命令将事务杀死
kill 536
```

### 方式三:调整锁等待时间

`my.ini`文件下,

```ini
#innodb_lock_wait_timeout = 50

# 修改为
innodb_lock_wait_timeout = 500
```

## 产生原因分析

**这个错误是由于当前操作的记录存在于**[**数据库**](https://cloud.tencent.com/solution/database?from=10680)**中未结束的事务导致行锁定。**

**简单说，就是现在要对一条记录进行修改，那么sql语句应该是这样的：**

```javascript
update user set uname = 'zhangsan' where uid = 1
```

**如果执行这条sql语句，发现一直处于处理中的状态，然后等一定时间（超时）后报错**

```bash
Lock wait timeout exceeded; try restarting transaction
```

**说明 uid = 1 的这条记录正处于一个未结束的事务中。**