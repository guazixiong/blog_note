# MySQL查询执行顺序

在开发和优化SQL查询时，了解MySQL查询执行的内部顺序是至关重要的。这不仅有助于编写更高效的查询，还能帮助调试和优化现有的数据库操作。

## 执行顺序

MySQL处理查询的顺序基本如下：

1. **FROM 子句** - 确定数据来源表及其关联。
2. **JOIN** - 执行表间的连接。
3. **WHERE** - 过滤不符合条件的行。
4. **GROUP BY** - 按指定列对结果进行分组。
5. **HAVING** - 过滤分组后的结果。
6. **SELECT** - 指定输出的列和计算列。
7. **DISTINCT** - 去除重复的结果行。
8. **ORDER BY** - 对结果进行排序。
9. **LIMIT / OFFSET** - 限制返回的结果数量或指定返回结果的起始位置。
10. **UNION / UNION ALL** -合并数据

## 示例说明

### 普通SQL语句

假设我们有一个包含员工信息的表`employees`。

```mysql
SELECT name, age FROM employees WHERE age > 30 ORDER BY age DESC LIMIT 10;
```

**执行顺序解析**：

1. **FROM `employees`** - 从`employees`表获取数据。
2. **WHERE `age > 30`** - 过滤出年龄大于30的员工。
3. **SELECT `name, age`** - 选择`name`和`age`列。
4. **ORDER BY `age DESC`** - 按年龄降序排序结果。
5. **LIMIT `10`** - 限制结果最多返回10行

### 关联查询语句

假设除了`employees`表，我们还有一个`departments`表，其中包含部门信息。

```mysql
SELECT e.name, d.name 
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE d.name = 'Engineering'
ORDER BY e.age;
```

**执行顺序解析**：

1. **FROM `employees`** - 确定数据来自`employees`表。
2. **JOIN `departments`** - 通过`department_id`连接`departments`表。
3. **WHERE `d.name = 'Engineering'`** - 过滤出部门为"Engineering"的记录。
4. **SELECT `e.name, d.name`** - 选择员工和部门的名称。
5. **ORDER BY `e.age`** - 按员工年龄排序结果。

### 子查询语句

假设我们需要找出平均工资高于部门平均工资的员工，此时我们的`employees`表中包含员工的工资`salary`和部门ID`department_id`。

```mysql
SELECT name, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
)
ORDER BY salary DESC;
```

**执行顺序解析**：

1. **FROM `employees e`** - 从`employees`表中选取数据。
2. WHERE - 使用子查询计算同一部门的平均工资。
   - **子查询**：首先从`employees`表中为每个员工的部门计算平均工资。
3. **SELECT `name, salary`** - 选择员工的姓名和工资。
4. **ORDER BY `salary DESC`** - 结果按工资降序排序。

## 示例含UNION的SQL语句

### 合并查询语句

假设我们想要从两个部门，例如`Engineering`和`Marketing`，获取员工的姓名列表，并且不想有重复的名字。

```mysql
SELECT name FROM employees WHERE department_id = (
    SELECT id FROM departments WHERE name = 'Engineering'
)
UNION
SELECT name FROM employees WHERE department_id = (
    SELECT id FROM departments WHERE name = 'Marketing'
);
```

1. **执行顺序解析**：
   1. 第一个SELECT查询
      - **FROM `employees`** - 从`employees`表中获取数据。
      - **WHERE** - 使用子查询确定`Engineering`部门的`id`。
      - **子查询**：从`departments`表获取`Engineering`部门的`id`。
      - **SELECT `name`** - 选择员工姓名。
   2. **第二个SELECT查询**（处理方式同上，只是部门名改为`Marketing`）。
   3. **UNION** - 合并两个查询的结果，自动去除重复的行