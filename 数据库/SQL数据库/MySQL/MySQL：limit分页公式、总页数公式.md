# MySQL：limit分页公式、总页数公式

## limit分页公式

- curPage: 当前页
- pageSize: 一页多少条记录

```sql
select * from table_name limit (curPage-1)*pageSize,pageSize
```

## 总页数公式

- totalRecord: 总记录数
- pageSize: 一页多少条记录

$总页数 =  (totalRecord +pageSize - 1) / pageSize;$