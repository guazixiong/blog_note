# mysql全文索引实战和原理

## 1：什么是全文索引

在当我们进行模糊查询的时候，一般都是使用 **like** 关键字来实现，在数据量小的情况下效率确实可以。有的人会说了，数据量大的时候我加索引啊。确实可以加索引，而且 **like name%** 是可以使用到索引的，但是当你要 **like %name%** 这样使用就会导致索引无效。在某些情况下就造成了局限性。

关于索引失效的原理可以移步这里：[juejin.cn/post/704479…](https://juejin.cn/post/7044793268780924935)

**在数据量大的情况下，全文索引的效率会比like高，在MySql5.6版本以前，只有MyISAM索引支持全文索引，在这之后Innodb也支持。**

## 2：全文索引的创建和使用

```sql
# 创建全文索引
CREATE FULLTEXT INDEX <index_name> on tableName(字段名)  

ALTER TABLE tableName ADD FULLTEXT[index_name](字段名);

CREATE TABLE tableName([....],FULLTEXT KEY[index_name](字段名))
```

和常用的**like**不同，全文索引有自己的格式，使用**match**和**against**关键字，如下：

```sql
select * from user where match(name) against('aaa');
```

## 3：全文索引实战

创建一张用户表people，主键ID和用户名name

```sql
//插入四条记录
insert into people (`name`) VALUES ('a');
insert into people (`name`) VALUES ('aa');
insert into people (`name`) VALUES ('aaa');
insert into people (`name`) VALUES ('aaaa');


//在字段name上建立全文索引
create fulltext index name_index_ft on people(name);

//开始查询
// 分别执行下面这4条SQL
select * from people where match(name) against('a');  // 没记录
select * from people where match(name) against('aa'); // 没记录
select * from people where match(name) against('aaa'); // 1条记录
select * from people where match(name) against('aaaa'); // 1条记录
复制代码
```

很多人可能会问，为什么我数据库有 name = a 和 name = aa的记录，为什么会找不到，而且全文索引不是跟**like**一样可以模糊查询的吗？？当我执行 **select \* from people where match(name) against('aaa');** 应该有二条记录啊，为什么只有一条??,接下来就要去了解全文的原理了

## 4：全文索引原理

在MySql客户端执行 **show variables like '%ft%';** 查看全文索引的信息

![image-20230402120023495](mysql%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95%E5%AE%9E%E6%88%98%E5%92%8C%E5%8E%9F%E7%90%86.assets/202304021200525.png)

### 为什么在查询 a 和 aa的时候会没有记录

存在`最小内容长度`和`最大内容长度`

先解释下其中几个参数

- ft_min_word_len：**针对MyISAM引擎的**，也就是在你创建的全文索引的字段的内容最大长度
- ft_max_word_len：**针对MyISAM引擎的**，也就是在你创建的全文索引的字段的内容最小长度 也就是说，只有在内容长度为 4 ~ 84（图中数值）的时候，你的全文索引才会有效,但是我们使用的Innodb存储引擎，所以你要看的是这二项
- innodb_ft_min_token_size：**针对Innodb引擎的**，也就是在你创建的全文索引的字段的内容最小长度
- innodb_ft_max_token_size：**针对Innodb引擎的**，也就是在你创建的全文索引的字段的内容最大长度

也就是说你的全文索引字段的内容长度必须在 3 ~ 84之间才会有效，现在能明白为什么在查 a 和 aa的时候找不到记录，但是在查 aaa 的时候就会有记录了吧，因为 aaa 长度刚好等于 3

### 为什么在查询 aaa的时候。name = aaaa的记录找不到

在全文索引底层是有一个**切词**的概念的，比如 **祝中国越来越强大** 全文索引按照一个规则切词，有可能会被切成 **祝** 、 **中国**、 **越来越强大**。那么切词的依据是什么呢？全文索引又是怎么切词的呢？？

![image-20230402120023495](mysql%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95%E5%AE%9E%E6%88%98%E5%92%8C%E5%8E%9F%E7%90%86.assets/202304021200525-1700366122620112.png)

在这里有个很重要的参数：**ft_boolean_syntax**，就是第一项，后面有很多内容，全文索引就是按照这些来切词的。现在表的记录如下

![image.png](mysql%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95%E5%AE%9E%E6%88%98%E5%92%8C%E5%8E%9F%E7%90%86.assets/202304021157810.webp)

将记录更新如下：

![image.png](mysql%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95%E5%AE%9E%E6%88%98%E5%92%8C%E5%8E%9F%E7%90%86.assets/202304021158231.webp)

```sql
select * from people where match(name) against('aaa');
```

![image.png](mysql%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95%E5%AE%9E%E6%88%98%E5%92%8C%E5%8E%9F%E7%90%86.assets/202304021158135.webp)

现在就能把所有的记录都找出来了吧,但是细心的你会发现，id = 6 的这条记录还是没找出来，这是为什么？？

### 全文索引为什么默认不支持模糊查询

为什么是默认呢，因为至少到现在我们发现全文索引都是 **等值查询** 的，这是因为全文索引默认支持的就是等值查询，如果要支持模糊查询需要改写SQL如下：

```sql
select * from people where match(name) against('aaa*' in boolean mode);
```

![image.png](mysql%E5%85%A8%E6%96%87%E7%B4%A2%E5%BC%95%E5%AE%9E%E6%88%98%E5%92%8C%E5%8E%9F%E7%90%86.assets/202304021159872.webp)

现在就可以把所有符合的数据都查出来了，* 表示一个或多个字符

## Mysql如何实现全文检索,关键词跑分

**MySQL 从 5.7.6 版本开始，MySQL就内置了ngram全文解析器**，用来支持中文、日文、韩文分词。**在 MySQL 5.7.6 版本之前，全文索引只支持英文全文索引，不支持中文全文索引**，需要**利用分词器把中文段落预处理拆分成单词，然后存入数据库**。

### ngram解析器

**5.6之后MySQL自带ngram解析器，可以解析中日韩三国文字**，如果不使用ngram解析器，则MySQL默认使用空格与符号作为分隔符（对于英文自然够用了，但对于中日韩文字就不好用了，所以才需要ngram解析器）

### 三种全文搜索模式

#### **自然语言模式（IN NATURAL LANGUAGE MODE，默认情况下为该模式）**

```sql
#如果最小搜索长度为1的话，则查找包含张，或三，或张三的记录；与布尔搜索模式中的‘+张三’结果相同
SELECT * FROM user WHERE MATCH(userName) AGAINST (‘张三’ );
```

#### **布尔搜索模式（IN BOOLEAN MODE）**

- 【+】----------必须包含此字符串
- 【-】----------必须不包含此字符串
- 【“ ”】--------双引号内作为整体不能拆词
- 【> 】--------提高该词的相关性，查询的结果靠前
- 【< 】--------降低该词的相关性，查询的结果靠后
- 【*】---------通配符，只能接在词后面
- 等

```sql
# 查询有‘美女’的又有‘动人’的记录
SELECT * FROM user 
WHERE MATCH(userName) AGAINST (’+“美女” & +“动人”’ IN BOOLEAN MODE);
```

#### **查询扩展搜索**

较少使用，没有细了解

## 参考

> 一文给你讲清楚MySql全文索引实战和原理 - 掘金
> https://juejin.cn/post/7101116456489713700

> 【MySQL】全文索引（FULLTEXT）的使用 - 知乎
> https://zhuanlan.zhihu.com/p/417229576