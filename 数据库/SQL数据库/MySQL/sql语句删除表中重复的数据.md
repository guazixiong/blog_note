# sql语句删除表中重复的数据

```sql
mysql> select * from user;
+----+--------+-----+
| id | name   | age |
+----+--------+-----+
|  1 | 张三   |  11 |
|  2 | 张三   |  11 |
|  3 | 李四   |  11 |
+----+--------+-----+
3 rows in set (0.00 sec)
```
```sql
mysql> delete from user
    -> where id not in
    -> (select * from
    -> (select min(id) from user group by name,age)as a);
```
