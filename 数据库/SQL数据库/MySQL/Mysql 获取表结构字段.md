# Mysql 获取表结构字段

## 基础版:Mysql 获取表结构字段

```sql
select * from table_name where 1 = 0;
```

## Plus1:使用JdbcTemplate获取表字段,并顺序返回

```java
LinkedHashMap<String, String> selectTableField(Sring tableName) {
    String sql = "SELECT * FROM " + tableName + " WHERE 1 = 0";
    LinkedHashMap<String, String> columns = targetJdbcTemplate.query(sql,
       rs -> {
           LinkedHashMap<String, String> columns1 = new LinkedHashMap<>();
           ResultSetMetaData metaData = rs.getMetaData();
           int columnCount = metaData.getColumnCount();
           for (int i = 1; i <= columnCount; i++) {
                String columnName = metaData.getColumnName(i);
                String columnType = metaData.getColumnTypeName(i);
                columns1.put(columnName, columnType);
           }
           return columns1;
       });}
```


