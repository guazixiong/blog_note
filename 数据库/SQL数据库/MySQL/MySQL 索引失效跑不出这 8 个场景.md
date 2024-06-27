# MySQL 索引失效跑不出这 8 个场景

## **基础数据准备**

准备一个数据表作为 数据演示  这里面一共 创建了三个索引

- 联合索引  `sname`, `s_code`, `address`
- 主键索引  `id`
- 普通索引  `height`

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for student
-- ----------------------------
DROP TABLE IF EXISTS `student`;
CREATE TABLE `student`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `sname` varchar(20) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  `s_code` int(100) NULL DEFAULT NULL,
  `address` varchar(100) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `height` double NULL DEFAULT NULL,
  `classid` int(11) NULL DEFAULT NULL,
  `create_time` datetime(0) NOT NULL ON UPDATE CURRENT_TIMESTAMP(0),
  PRIMARY KEY (`id`) USING BTREE,
  INDEX `普通索引`(`height`) USING BTREE,
  INDEX `联合索引`(`sname`, `s_code`, `address`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 5 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of student
-- ----------------------------
INSERT INTO `student` VALUES (1, '学生1', 1, '上海', 170, 1, '2022-11-02 20:44:14');
INSERT INTO `student` VALUES (2, '学生2', 2, '北京', 180, 2, '2022-11-02 20:44:16');
INSERT INTO `student` VALUES (3, '变成派大星', 3, '京东', 185, 3, '2022-11-02 20:44:19');
INSERT INTO `student` VALUES (4, '学生4', 4, '联通', 190, 4, '2022-11-02 20:44:25');
```

## **正文**

上面的SQL 我们已经创建好基本的数据  在验证之前 先带着几个问题

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261033212.png)

我们先从上往下进行验证

### **最左匹配原则**

写在前面：我很早之前就听说过数据库的最左匹配原则，当时是通过各大博客论坛了解的，但是这些博客的局限性在于它们对最左匹配原则的描述就像一些数学定义一样，往往都是列出123点，满足这123点就能匹配上索引，否则就不能。

**最左匹配原则就是指在联合索引中，如果你的 SQL 语句中用到了联合索引中的最左边的索引，那么这条 SQL 语句就可以利用这个联合索引去进行匹配**，我们上面建立了联合索引 可以用来测试最左匹配原则 `sname`, `s_code`, `address`

请看下面SQL语句 进行思考 是否会走索引

```sql
-- 联合索引 sname,s_code,address
-- 会走索引吗？
1、select create_time from student where sname = "变成派大星"  
-- 会走索引吗？
2、select create_time from student where s_code = 1   
-- 会走索引吗？
3、select create_time from student where address = "上海"  
-- 会走索引吗？
4、select create_time from student where address = "上海" and s_code = 1 
-- 会走索引吗？
5、select create_time from student where address = "上海" and sname = "变成派大星"  
 -- 会走索引吗？
6、select create_time from student where sname = "变成派大星" and address = "上海" 
-- 会走索引吗？
7、select create_time from student where sname = "变成派大星" and s_code = 1 and address = "上海"  
```

凭你的经验 哪些会使用到索引呢 ？可以先思考一下 在心中记下数字

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261036594.png)

**走索引例子**

```sql
EXPLAIN  select create_time from student where sname = "变成派大星" 
```

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261037596.png)

**未走索引例子**

```sql
EXPLAIN select create_time from student where address = "上海" and s_code = 1
```

走的全表扫描 rows = 4

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261037016.png)

最左匹配原则顾名思义：最左优先，以最左边的为起点任何连续的索引都能匹配上。

**同时遇到范围查询(>、<、between、like)就会停止匹配**。

例如：s_code = 2 如果建立(`sname`, `s_code`)顺序的索引，是匹配不到(`sname`, `s_code`)索引的;

但是如果查询条件是`sname = "变成派大星" and s_code = 2或者a=1(又或者是s_code = 2 and sname = "变成派大星" )`就可以，**因为优化器会自动调整****`sname`, `s_code`的顺序**。

再比如sname = "变成派大星" and s_code > 1 and address = "上海"  `address是用不到索引的`，因为s_code字段是一个**范围查询，它之后的字段会停止匹配**。

#### 不带范围查询 索引使用类型

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261038779.png)

#### 带范围使用类型

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261038579.png)

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261039147.png)

#### **思考** 

为什么左链接一定要遵循最左缀原则呢？

#### **验证** 

看过一个比较好玩的回答：

> 你可以认为联合索引是闯关游戏的设计
> 例如你这个联合索引是state/city/zipCode
> 那么state就是第一关 city是第二关， zipCode就是第三关
> 你必须匹配了第一关，才能匹配第二关，匹配了第一关和第二关，才能匹配第三关

这样描述不算完全准确 但是确实是这种思想

要想理解联合索引的最左匹配原则，先来理解下索引的底层原理。

**索引的底层是一颗B+树**，那么联合索引的底层也就是一颗B+树，只不过**联合索引的B+树节点中存储的是键值**。由于构建一棵B+树只能根据一个值来确定索引关系，所以**数据库依赖联合索引最左的字段来构建**.文字比较抽象 我们看一下

加入我们建立 A,B 联合索引 他们在底层储存是什么样子呢？

- 橙色代表字段 A
- 浅绿色 代表字段B

图解：

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261041987.png)

我们可以看出几个特点

- A 是有顺序的  1，1，2，2，3，4
- B 是没有顺序的 1，2，1，4，1，2 这个是散列的
- 如果A是等值的时候 B是有序的  例如 （1，1），（1，2） 这里的B有序的 （2，1）,(2,4) B 也是有序的

这里应该就能看出 如果没有A的支持 B的索引是散列的 不是连续的

再细致一点 我们重新创建一个表

```sql
DROP TABLE IF EXISTS `leftaffix`;

CREATE TABLE `leftaffix`  (
  `a` int(11) NOT NULL AUTO_INCREMENT,
  `b` int(11) NULL DEFAULT NULL,
  `c` int(11) NULL DEFAULT NULL,
  `d` int(11) NULL DEFAULT NULL,
  `e` varchar(11) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`a`) USING BTREE,
  INDEX `联合索引`(`b`, `c`, `d`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 8 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
 
-- ----------------------------
-- Records of leftaffix
-- ----------------------------
INSERT INTO `leftaffix` VALUES (1, 1, 1, 1, '1');
INSERT INTO `leftaffix` VALUES (2, 2, 2, 2, '2');
INSERT INTO `leftaffix` VALUES (3, 3, 2, 2, '3');
INSERT INTO `leftaffix` VALUES (4, 3, 1, 1, '4');
INSERT INTO `leftaffix` VALUES (5, 2, 3, 5, '5');
INSERT INTO `leftaffix` VALUES (6, 6, 4, 4, '6');
INSERT INTO `leftaffix` VALUES (7, 8, 8, 8, '7');
SET FOREIGN_KEY_CHECKS = 1;
```

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261044574.png)



根据最左缀原则,

- 在创建索引树的时候会对数据进行排序.
- 会先通过 B 进行排序,也就是如果出现值相同就根据 C 排序,
- 如果 C相同就根据D 排序 

排好顺序之后就是如下图

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261046146.png)

> 针对b进行索引排序,得到1,2,2,3,3,4,6,8
>
> 再针对c进行索引排序,(同值从小到大排序),得到1,2,3,1,2,4,4,8
>
> 再针对d进行索引排序,(同值从小到大排序),得到1,2,3,1,2,4,4,8

索引的生成就会根据图二的顺序进行生成  我们看一下 生成后的树状数据是什么样子

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261119578.png)

> 解释一些这个树状图  首先根据图二的排序 我们知道顺序 是 1111a  2222b 所以 在第三层 我们可以看到 1111a 在第一层 2222b在第二层  因为 111 < 222 所以 111 进入第二层 然后得出第一层   ????????

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261125035.png)

简化一下就是这个样子

但是这种顺序是相对的。这是因为MySQL创建联合索引的规则是首先会对联合索引的最左边`第一个字段排序`，在第一个字段的排序基础上，然后在对第二个字段进行排序。所以B=2这种查询条件没有办法利用索引。

看到这里还可以明白一个道理 为什么我们建立索引的时候不推荐建立在经常改变的字段 因为这样的话我们的索引结构就要跟着你的改变而改动 所以很消耗性能

#### **补充**

**最左缀原则可以通过跳跃扫描的方式打破** 简单整理一下这方面的知识

这个是在 8.0 进行的优化

`MySQL8.0版本`开始增加了索引跳跃扫描的功能，**当第一列索引的唯一值较少时，即使where条件没有第一列索引，查询的时候也可以用到联合索引**。

比如我们使用的联合索引是 bcd  但是b中字段比较少 我们在使用联合索引的时候没有 使用 b 但是依然可以使用联合索引**MySQL联合索引有时候遵循最左前缀匹配原则，有时候不遵循。**

#### **小总结**

前提 如果创建 b,c,d 联合索引面

- 如果 我where 后面的条件是`c = 1 and d = 1`为什么不能走索引呢 如果没有b的话 你查询的值相当于` *11` 我们都知道`*`是所有的意思也就是我能匹配到所有的数据
- 如果 我 where 后面是` b = 1 and d =1` 为什么会走索引呢？你等于查询的数据是 `1*1 `我可以通过前面 1 进行索引匹配 所以就可以走索引
- 最左缀匹配原则的最重要的就是 第一个字段

我们接着看下一个失效场景

### **select \***

#### **思考**

这里是我之前的一个思维误区 select * 不会导致索引失效 之前测试发现失效是因为where 后面的查询范围过大 导致索引失效 并不是Select * 引起的  但是为什么不推荐使用select *

#### **解释**

- 增加查询分析器解析成本。
- 增减字段容易与 resultMap 配置不一致。
- 无用字段增加网络 消耗，尤其是 text 类型的字段。

在阿里的开发手册中，大面的概括了上面几点。

在使用Select * 索引使用正常

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261129144.png)

虽然走了索引但是 也不推荐这种写法 为什么呢？

首先我们在上一个验证中创建了联合索引 我们使用B=1 会走索引  但是 与直接查询索引字段不同  使用`SELECT*`,获取了不需要的数据，则首先通过辅助索引过滤数据，然后再通过聚集索引获取所有的列，这就多了一次b+树查询，速度必然会慢很多，减少使用select * 就是降低回表带来的损耗。

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261129553.png)

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261129054.png)

也就是 Select * 在一些情况下是会走索引的 如果不走索引就是 where 查询范围过大 导致MySQL 最优选择全表扫描了 并不是Select * 的问题

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261129409.png)

上图就是索引失效的情况

范围查找也不是一定会索引失效 下面情况就会索引生效就是 级别低 生效的原因是因为缩小了范围

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261129691.png)

#### **小总结**

- select * 会走索引
- 范围查找有概率索引失效但是在特定的情况下会生效 范围小就会使用 也可以理解为 返回结果集小就会使用索引
- mysql中连接查询的原理是先对驱动表进行查询操作，然后再用从驱动表得到的数据作为条件，逐条的到被驱动表进行查询。
- 每次驱动表加载一条数据到内存中，然后被驱动表所有的数据都需要往内存中加载一遍进行比较。效率很低，所以mysql中可以指定一个缓冲池的大小，缓冲池大的话可以同时加载多条驱动表的数据进行比较，放的数据条数越多性能io操作就越少，性能也就越好。所以，如果此时使用`select *` 放一些无用的列，只会白白的占用缓冲空间。浪费本可以提高性能的机会。
- 按照评论区老哥的说法 select * 不是造成索引失效的直接原因 大部分原因是 where 后边条件的问题 但是还是尽量少去使用select * 多少还是会有影响的

### **使用函数**

使用在Select 后面使用函数可以使用索引 但是下面这种做法就不能

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261142920.png)

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261142815.png)

**因为索引保存的是索引字段的原始值，而不是经过函数计算后的值，自然就没办法走索引了。**

不过，从 MySQL 8.0 开始，索引特性增加了函数索引，即可以针对函数计算后的值建立一个索引，也就是说该索引的值是函数计算后的值，所以就可以通过扫描索引来查询数据。

这种写法我没使用过 感觉情况比较少 也比较容易注意到这种写法

### **计算操作**

这个情况和上面一样 之所以会导致索引失效是因为改变了索引原来的值 在树中找不到对应的数据只能全表扫描

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261146843.png)

因为索引保存的是索引字段的原始值，而不是 b - 1 表达式计算后的值，所以无法走索引，只能通过把索引字段的取值都取出来，然后依次进行表达式的计算来进行条件判断，因此采用的就是全表扫描的方式。

下面这种计算方式就会使用索引

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261146854.png)

Java比较熟悉的可能会有点疑问，这种对索引进行简单的表达式计算，在代码特殊处理下，应该是可以做到索引扫描的，比方将 b - 1 = 6 变成 b = 6 - 1。是的，是能够实现，但是 MySQL 还是偷了这个懒，没有实现。

#### **小总结**

总而言之 言而总之 只要是影响到索引列的值 索引就是失效

### **Like %**

1.这个真的是难受哦  因为经常使用这个 所以还是要小心点 在看为什么失效之前 我们先看一下 Like % 的解释

- **%百分号通配符:** 表示任何字符出现任意次数(可以是0次).
- **_下划线通配符:** 表示只能匹配单个字符,不能多也不能少,就是一个字符.
- **like操作符:** LIKE作用是指示mysql后面的搜索模式是利用通配符而不是直接相等匹配进行比较.

**注意:** 如果在使用like操作符时,后面的没有使用通用匹配符效果是和`=`一致的,

```sql
SELECT * FROM products WHERE products.prod_name like '1000';
```

2.匹配包含"Li"的记录(包括记录"Li") :

```sql
SELECT* FROM products WHERE products.prod_name like '%Li%';
```

3.匹配以"Li"结尾的记录(包括记录"Li",不包括记录"Li ",也就是Li后面有空格的记录,这里需要注意)

```sql
SELECT * FROM products WHERE products.prod_name like '%Li';
```

在左不走 在右走

`右：`虽然走 但是索引级别比较低主要是模糊查询 范围比较大 所以索引级别就比较低

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261147411.png)

`左：`这个范围非常大 所以没有使用索引的必要了 这个可能不是很好优化 还好不是一直拼接上面的

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261148309.png)

#### **小总结**

索引的时候和查询范围关系也很大 范围过大造成索引没有意义从而失效的情况也不少

### **使用Or导致索引失效**

这个原因就更简单了

在 WHERE 子句中，如果在 OR 前的条件列是索引列，而在 OR 后的条件列不是索引列，那么索引会失效 举个例子，比如下面的查询语句，b 是主键，e 是普通列，从执行计划的结果看，是走了全表扫描。

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261148210.png)

#### **优化**

这个的优化方式就是 **在Or的时候两边都加上索引**

就会使用索引 避免全表扫描

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261148988.png)

### **in使用不当**

首先使用In 不是一定会造成全表扫描的 **IN肯定会走索引，但是当IN的取值范围较大时会导致索引失效，走全表扫描**

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261149758.png)

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261149712.png)

**in 在结果集 大于30%的时候索引失效**

**not in 和 In的失效场景相同**

### **order By**

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261149632.png)

这一个主要是Mysql 自身优化的问题 我们都知道OrderBy 是排序 那就代表我需要对数据进行排序 如果我走索引 索引是排好序的 但是我需要回表 消耗时间 另一种 我直接全表扫描排序 不用回表 也就是

- 走索引 + 回表
- 不走索引 直接全表扫描

Mysql 认为直接全表扫面的速度比 回表的速度快所以就直接走索引了  在Order By 的情况下 走全表扫描反而是更好的选择

#### **子查询会走索引吗**

答案是会 但是使用不好就不会

### **大总结**

![图片](MySQL%20%E7%B4%A2%E5%BC%95%E5%A4%B1%E6%95%88%E8%B7%91%E4%B8%8D%E5%87%BA%E8%BF%99%208%20%E4%B8%AA%E5%9C%BA%E6%99%AF.assets/202303261151977.png)