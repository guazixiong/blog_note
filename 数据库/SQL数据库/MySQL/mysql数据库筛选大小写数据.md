# Mysql数据库筛选大小写数据

​	

```sql
mysql> select id,create_by from test_demo where create_by = 'admin';
+----------------------------------+-----------+
| id                               | create_by |
+----------------------------------+-----------+
| 1331884149004910593              | admin     |
| 1331901553776869377              | admin     |
| 1533107308342210561              | admin     |
| 4028810c6aed99e1016aed9b31b40002 | Admin     |
| 4028810c6b02cba2016b02cba21f0000 | admin     |
+----------------------------------+-----------+
5 rows in set (0.00 sec)
```

查询条件为`admin`.结果将`Admin`查询出来了.

只想`admin`查询出来,追加注解`binary`

```sql
mysql> select id,create_by from test_demo where binary create_by = 'admin';
+----------------------------------+-----------+
| id                               | create_by |
+----------------------------------+-----------+
| 1331884149004910593              | admin     |
| 1331901553776869377              | admin     |
| 1533107308342210561              | admin     |
| 4028810c6b02cba2016b02cba21f0000 | admin     |
+----------------------------------+-----------+
4 rows in set (0.00 sec)

mysql> select id,create_by from test_demo where binary create_by = 'Admin';
+----------------------------------+-----------+
| id                               | create_by |
+----------------------------------+-----------+
| 4028810c6aed99e1016aed9b31b40002 | Admin     |
+----------------------------------+-----------+
1 row in set (0.00 sec)
```

