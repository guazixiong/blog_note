> 配套资料: [https://pan.baidu.com/s/18au42FIhSNrXY9p7MbmNbg](https://pan.baidu.com/s/18au42FIhSNrXY9p7MbmNbg) 提取码: 29ad
> 感谢 B 站用户 [**冷鸟丨会飞**](https://space.bilibili.com/55263887)分享

**课程目标**
MongoDB的副本集: 操作, 主要概念, 故障转移, 选举规则 MongoDB的分片集群：概念, 优点, 操作, 分片策略, 故障转移 MongoDB的安全认证

- 理解 MongoDB 的业务场景, 熟悉 MongoDB 的简介, 特点和体系结构, 数据类型等.
- 能够在 Windows 和 Linux 下安装和启动 MongoDB, 图形化管理界面 Compass 的安装使用
- 掌握 MongoDB 基本常用命令实现数据的 CRUD
- 掌握 MongoDB 的索引类型, 索引管理, 执行计划
## 1. MongoDB 相关概念
### 1.1 业务场景
传统的关系型数据库 (比如 MySQL), 在数据操作的”三高”需求以及对应的 Web 2.0 网站需求面前, 会有”力不从心”的感觉
所谓的三高需求:
**高并发, 高性能, 高可用**, 简称三高

- High Performance: 对数据库的高并发读写的要求
- High Storage: 对海量数据的高效率存储和访问的需求
- High Scalability && High Available: 对数据的高扩展性和高可用性的需求

**而 MongoDB 可以应对三高需求**
具体的应用场景:

- 社交场景, 使用 MongoDB 存储存储用户信息, 以及用户发表的朋友圈信息, 通过地理位置索引实现附近的人, 地点等功能.
- 游戏场景, 使用 MongoDB 存储游戏用户信息, 用户的装备, 积分等直接以内嵌文档的形式存储, 方便查询, 高效率存储和访问.
- 物流场景, 使用 MongoDB 存储订单信息, 订单状态在运送过程中会不断更新, 以 MongoDB 内嵌数组的形式来存储, 一次查询就能将订单所有的变更读取出来.
- 物联网场景, 使用 MongoDB 存储所有接入的智能设备信息, 以及设备汇报的日志信息, 并对这些信息进行多维度的分析.
- 视频直播, 使用 MongoDB 存储用户信息, 点赞互动信息等.

这些应用场景中, 数据操作方面的共同点有:

1. **数据量大**
2. **写入操作频繁**
3. 价值较低的数据, 对**事务性**要求不高

对于这样的数据, 更适合用 MongoDB 来实现数据存储
那么我们**什么时候选择 MongoDB 呢?**
除了架构选型上, 除了上述三个特点之外, 还要考虑下面这些问题:

- 应用不需要事务及复杂 JOIN 支持
- 新应用, 需求会变, 数据模型无法确定, 想快速迭代开发
- 应用需要 2000 - 3000 以上的读写QPS（更高也可以）
- 应用需要 TB 甚至 PB 级别数据存储
- 应用发展迅速, 需要能快速水平扩展
- 应用要求存储的数据不丢失
- 应用需要 99.999% 高可用
- 应用需要大量的地理位置查询, 文本查询

如果上述有1个符合, 可以考虑 MongoDB, 2个及以上的符合, 选择 MongoDB 绝不会后悔.
> 如果用MySQL呢?
> 相对MySQL, 可以以更低的成本解决问题（包括学习, 开发, 运维等成本）

### 1.2 MongoDB 简介
MongoDB是一个**开源, 高性能, 无模式**的文档型数据库, 当初的设计就是用于简化开发和方便扩展, 是NoSQL数据库产品中的一种.是最 像关系型数据库（MySQL）的非关系型数据库. 它支持的数据结构非常松散, **是一种类似于 JSON 的 格式叫BSON,** 所以它既可以存储比较复杂的数据类型, 又相当的灵活. MongoDB中的记录是**一个文档, 它是一个由字段和值对（ﬁeld:value）**组成的数据结构.MongoDB文档类似于JSON对象, 即一个文档认 为就是一个对象.字段的数据类型是字符型, 它的值除了使用基本的一些类型外, 还可以包括其他文档, 普通数组和文档数组.
**“最像关系型数据库的 NoSQL 数据库”**. MongoDB 中的记录是一个文档, 是一个 key-value pair. 字段的数据类型是字符型, 值除了使用基本的一些类型以外, 还包括其它文档, 普通数组以及文档数组

![image-20240410105116410](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105116410.png)

![image-20240410105129003](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105129003.png)



MongoDB 数据模型是面向文档的, 所谓文档就是一种类似于 JSON 的结构, 简单理解 MongoDB 这个数据库中存在的是各种各样的 JSON（BSON)

- 数据库 (database)
   - 数据库是一个仓库, 存储集合 (collection)
- 集合 (collection)
   - 类似于数组, 在集合中存放文档
- 文档 (document)
   - 文档型数据库的最小单位, 通常情况, 我们存储和操作的内容都是文档

在 MongoDB 中, 数据库和集合都不需要手动创建, 当我们创建文档时, 如果文档所在的集合或者数据库不存在, **则会自动创建数据库或者集合**
### 数据库 (databases) 管理语法
| **操作** | **语法** |
| --- | --- |
| 查看所有数据库 | show dbs; 或 show databases; |
| 查看当前数据库 | db; |
| 切换到某数据库 (**若数据库不存在则创建数据库**) | use <db_name>; |
| 删除当前数据库 | db.dropDatabase(); |

### 集合 (collection) 管理语法
| **操作** | **语法** |
| --- | --- |
| 查看所有集合 | show collections; |
| 创建集合 | db.createCollection("<collection_name>"); |
| 删除集合 | db.<collection_name>.drop() |

### 1.3. 数据模型
![image-20240410105226300](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105226300.png)
### 1.4 MongoDB 的特点
#### 1.4.1 高性能
MongoDB 提供高性能的数据持久化

- 嵌入式数据模型的支持减少了数据库系统上的 I/O 活动
- 索引支持更快的查询, 并且可以包含来自嵌入式文档和数组的键 (文本索引解决搜索的需求, TTL 索引解决历史数据自动过期的需求, 地理位置索引可以用于构件各种 O2O 应用)
- mmapv1, wiredtiger, mongorocks (rocksdb) in-memory 等多引擎支持满足各种场景需求
- Gridfs 解决文件存储需求
#### 1.4.2 高可用
MongoDB 的复制工具称作**副本集** (replica set) 可以提供自动故障转移和数据冗余
#### 1.4.3 高扩展
水平扩展是其核心功能一部分
分片将数据分布在一组集群的机器上 (海量数据存储, 服务能力水平扩展)
MongoDB 支持基于**片键**创建数据区域, 在一个平衡的集群当中, MongoDB 将一个区域所覆盖的读写**只定向**到该区域的那些片
#### 1.4.4 其他
MongoDB支持丰富的查询语言, 支持读和写操作(CRUD), 比如数据聚合, 文本搜索和地理空间查询等. 无模式（动态模式）, 灵活的文档模型
## 2. 基本常用命令
### 2.1 数据库操作
默认保留的数据库

- **admin**: 从权限角度考虑, 这是 root 数据库, 如果将一个用户添加到这个数据库, 这个用户自动继承所有数据库的权限, 一些特定的服务器端命令也只能从这个数据库运行, 比如列出所有的数据库或者关闭服务器
- **local**: 数据永远不会被复制, 可以用来存储限于本地的单台服务器的集合 (部署集群, 分片等)
- **config**: Mongo 用于分片设置时, config 数据库在内部使用, 用来保存分片的相关信息
```
# 查看所有数据库
$ show dbs

# 切换至articledb数据库,不存在创建
$ use articledb

# 查看所有数据库
$ show dbs
```
**当使用 use articledb 的时候. articledb 其实存放在内存之中, 当 articledb 中存在一个 collection 之后, mongo 才会将这个数据库持久化到硬盘之中.**
### 2.2 文档基本 CRUD
官方文档: [https://docs.mongodb.com/manual/crud/](https://docs.mongodb.com/manual/crud/)
#### 2.2.1 创建 Create
Create or insert operations add new [documents](https://docs.mongodb.com/manual/core/document/#bson-document-format) to a [collection](https://docs.mongodb.com/manual/core/databases-and-collections/#collections). If the collection does **not** currently exist, insert operations will create the collection automatically.

- 使用 db.<collection_name>.insert() 向集合中添加_一个文档_, 参数一个 bson 格式的文档
- 使用 db.<collection_name>.insertMany() 向集合中添加_多个文档_, 参数为 bson 文档数组
```
db.collection.insert({
  <document or array of documents>,
  writeConcern: <document>,
  ordered: <boolean>
})


// 向集合中添加一个文档
db.collection.insertOne(
   { item: "canvas", qty: 100, tags: ["cotton"], size: { h: 28, w: 35.5, uom: "cm" } }
)

// 向集合中添加多个文档
db.collection.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], size: { h: 14, w: 21, uom: "cm" } },
   { item: "mat", qty: 85, tags: ["gray"], size: { h: 27.9, w: 35.5, uom: "cm" } },
   { item: "mousepad", qty: 25, tags: ["gel", "blue"], size: { h: 19, w: 22.85, uom: "cm" } }
])
```
注：当我们向 collection 中插入 document 文档时, 如果没有给文档指定** _id **属性, 那么数据库会为文档**自动添加 **`**_id**`** field,** 并且值类型是 `ObjectId(blablabla)`, 就是文档的唯一标识, 类似于 relational database 里的 primary key

- mongo 中的数字, 默认情况下是 double 类型, 如果要存整型, 必须使用函数 NumberInt(整型数字), 否则取出来就有问题了
- 插入当前日期可以使用 new Date()

如果某条数据插入失败, 将会终止插入, 但已经插入成功的数据**不会回滚掉**. 因为批量插入由于数据较多容易出现失败, 因此, 可以使用 try catch 进行异常捕捉处理, 测试的时候可以不处理.如：
```bash
try {
  db.comment.insertMany([
    {"_id":"1","articleid":"100001","content":"我们不应该把清晨浪费在手机上, 健康很重要, 一杯温水幸福你我 他.","userid":"1002","nickname":"相忘于江湖","createdatetime":new Date("2019-0805T22:08:15.522Z"),"likenum":NumberInt(1000),"state":"1"},
    {"_id":"2","articleid":"100001","content":"我夏天空腹喝凉开水, 冬天喝温开水","userid":"1005","nickname":"伊人憔 悴","createdatetime":new Date("2019-08-05T23:58:51.485Z"),"likenum":NumberInt(888),"state":"1"},
    {"_id":"3","articleid":"100001","content":"我一直喝凉开水, 冬天夏天都喝.","userid":"1004","nickname":"杰克船 长","createdatetime":new Date("2019-08-06T01:05:06.321Z"),"likenum":NumberInt(666),"state":"1"},
    {"_id":"4","articleid":"100001","content":"专家说不能空腹吃饭, 影响健康.","userid":"1003","nickname":"凯 撒","createdatetime":new Date("2019-08-06T08:18:35.288Z"),"likenum":NumberInt(2000),"state":"1"},
    {"_id":"5","articleid":"100001","content":"研究表明, 刚烧开的水千万不能喝, 因为烫 嘴.","userid":"1003","nickname":"凯撒","createdatetime":new Date("2019-0806T11:01:02.521Z"),"likenum":NumberInt(3000),"state":"1"}

]);

} catch (e) {
  print (e);
}
```
#### 2.2.2 查询 Read

- 使用 db.<collection_name>.find() 方法对集合进行查询, 接受一个 json 格式的查询条件. 返回的是一个**数组**
- db.<collection_name>.findOne() 查询集合中符合条件的第一个文档, 返回的是一个**对象**

![image-20240410105427063](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105427063.png)

可以使用 $in 操作符表示_范围查询_

```
# mongodb 
db.inventory.find( { status: { $in: [ "A", "D" ] } } )

# mysql
select * from inventory where status in ("A","D");
```
多个查询条件用逗号分隔, 表示 AND 的关系
```bash
# mongodb 
db.inventory.find( { status: "A", qty: { $lt: 30 } } )

# mysql
select * from inventory where status = "A" and qty < 30;
```
使用 `$or` 操作符表示后边数组中的条件是OR的关系
```bash
# mongodb 
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )

# mysql
SELECT * FROM inventory WHERE status = "A" OR qty < 30
```
联合使用 AND 和 OR 的查询语句
```
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )

# item 是正则表达式
select * from inventory where status = "A" and ( qty < 30 or item REGEXP '/^p/');
```
在 terminal 中查看结果可能不是很方便, 所以我们可以用 pretty() 来帮助阅读
```
db.inventory.find().pretty()
```
匹配内容
```
db.posts.find({
  comments: {
    $elemMatch: {
      user: 'Harry Potter'
    }
  }
}).pretty()

// 正则表达式
db.<collection_name>.find({ content : /once/ })
```
创建索引
```
db.posts.createIndex({
  { title : 'text' }
})

// 文本搜索
// will return document with title "Post One"
// if there is no more posts created
db.posts.find({
  $text : {
    $search : "\"Post O\""
  }
}).pretty()
```
#### 2.2.3 更新 Update

- 使用 db.<collection_name>.update(<filter>, <update>, <options>) 方法修改一个匹配 <filter> 条件的文档
- 使用 db.<collection_name>.updateMany(<filter>, <update>, <options>) 方法修改所有匹配 <filter> 条件的文档
- 使用 db.<collection_name>.replaceOne(<filter>, <update>, <options>) 方法**替换**一个匹配 <filter> 条件的文档
- db.<collection_name>.update(查询对象, 新对象) 默认情况下会使用新对象替换旧对象

其中 <filter> 参数与查询方法中的条件参数用法一致.
如果需要修改指定的属性, 而不是替换需要用“修改操作符”来进行修改

- $set 修改文档中的制定属性

其中最常用的修改操作符即为$set和$unset,分别表示**赋值**和**取消赋值**.
```
db.inventory.update(
    { item: "paper" },
    {
        $set: { "size.uom": "cm", status: "P" },
        $currentDate: { lastModified: true }
    }
)

db.inventory.updateMany(
    { qty: { $lt: 50 } },
    {
        $set: { "size.uom": "in", status: "P" },
        $currentDate: { lastModified: true }
    }
)
```

- uses the [$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set) operator to update the value of the size.uom field to "cm" and the value of the status field to "P",
- uses the [$currentDate](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate) operator to update the value of the lastModified field to the current date. If lastModified field does not exist, [$currentDate](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate) will create the field. See [$currentDate](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate) for details.

db.<collection_name>.replace() 方法替换除 _id 属性外的**所有属性**, 其<update>参数应为一个**全新的文档**.
```
db.inventory.replaceOne(
    { item: "paper" },
    { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 40 } ] }
)
```
**批量修改**
```
// 默认会修改第一条
db.document.update({ userid: "30", { $set {username: "guest"} } })

// 修改所有符合条件的数据
db.document.update( { userid: "30", { $set {username: "guest"} } }, {multi: true} )
```
**列值增长的修改**
如果我们想实现对某列值在原有值的基础上进行增加或减少, 可以使用 $inc 运算符来实现
```
db.document.update({ _id: "3", {$inc: {likeNum: NumberInt(1)}} })
```
##### 修改操作符
| **Name** | **Description** |
| --- | --- |
| [$currentDate](https://docs.mongodb.com/manual/reference/operator/update/currentDate/#up._S_currentDate) | Sets the value of a field to current date, either as a Date or a Timestamp.<br />将字段的值设置为当前日期，作为日期或时间戳。 |
| [$inc](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc) | Increments the value of the field by the specified amount.<br />将字段的值增加指定的量。 |
| [$min](https://docs.mongodb.com/manual/reference/operator/update/min/#up._S_min) | Only updates the field if the specified value is less than the existing field value.<br />仅当指定值小于现有字段值时才更新字段。 |
| [$max](https://docs.mongodb.com/manual/reference/operator/update/max/#up._S_max) | Only updates the field if the specified value is greater than the existing field value.<br />仅当指定值大于现有字段值时才更新字段。 |
| [$mul](https://docs.mongodb.com/manual/reference/operator/update/mul/#up._S_mul) | Multiplies the value of the field by the specified amount.<br />将字段的值乘以指定的金额。 |
| [$rename](https://docs.mongodb.com/manual/reference/operator/update/rename/#up._S_rename) | Renames a field.<br />重命名字段。 |
| [$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set) | Sets the value of a field in a document.<br />设置文档中字段的值。 |
| [$setOnInsert](https://docs.mongodb.com/manual/reference/operator/update/setOnInsert/#up._S_setOnInsert) | Sets the value of a field if an update results in an insert of a document. Has no effect on update operations that modify existing documents.<br />如果更新导致插入文档，则设置字段的值。 对修改现有文档的更新操作没有影响。 |
| [$unset](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset) | Removes the specified field from a document.<br />从文档中删除指定字段。 |

#### 2.2.4 删除 Delete

- 使用 db.collection.deleteMany() 方法删除所有匹配的文档.
- 使用 db.collection.deleteOne() 方法删除单个匹配的文档.
- db.collection.drop()
- db.dropDatabase()
```
db.inventory.deleteMany( { qty : { $lt : 50 } } )
```
> Delete operations **do not drop indexes**, even if deleting all documents from a collection.
> 一般数据库中的数据都不会真正意义上的删除, 会添加一个字段, 用来表示这个数据是否被删除

### 2.3 文档排序和投影 (sort & projection)
#### 2.3.1 排序 Sort
在查询文档内容的时候, 默认是按照 _id 进行排序
我们可以用 $sort 更改文档排序规则
```
{ $sort: { <field1>: <sort order>, <field2>: <sort order> ... } }
```
> For the field or fields to sort by, set the sort order to 1 or -1 to specify an _ascending_ or _descending_ sort respectively, as in the following example:
> 对于要排序的一个或多个字段，将排序顺序设置为 1 或 -1 以分别指定升序或降序排序，如下例所示：

```
db.users.aggregate(
   [
     { $sort : { age : -1, posts: 1 } }
     // ascending on posts and descending on age
   ]
)
```
##### $sort Operator and Memory
##### $sort + $limit Memory Optimization
> When a [$sort](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/index.html#pipe._S_sort) precedes a [$limit](https://docs.mongodb.com/manual/reference/operator/aggregation/limit/#pipe._S_limit) and there are no intervening stages that modify the number of documents, the optimizer can coalesce the [$limit](https://docs.mongodb.com/manual/reference/operator/aggregation/limit/#pipe._S_limit) into the [$sort](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/index.html#pipe._S_sort). This allows the [$sort](https://docs.mongodb.com/manual/reference/operator/aggregation/sort/index.html#pipe._S_sort) operation to **only maintain the top n results as it progresses**,
> where n is the specified limit, and ensures that MongoDB only needs to store n items in memory. This optimization still applies when allowDiskUse is true and the n items exceed the [aggregation memory limit](https://docs.mongodb.com/manual/core/aggregation-pipeline-limits/#agg-memory-restrictions).
> Optimizations are subject to change between releases.
> 
当 $sort 在 $limit 之前并且没有修改文档数量的中间阶段时，优化器可以将 $limit 合并到 $sort 中。 这允许 $sort 操作在进行时仅维护前 n 个结果，其中 n 是指定的限制，并确保 MongoDB 仅需要在内存中存储 n 个项目。 当allowDiskUse为true并且n个项目超过聚合内存限制时，此优化仍然适用。
> 优化可能会因版本而异。

有点类似于用 heap 做 topK 这种问题, 只维护 k 个大小的 heap, 会加速 process
举个栗子:
```
db.posts.find().sort({ title : -1 }).limit(2).pretty()
```
#### 2.3.2 投影 Projection
有些情况, 我们对文档进行查询并不是需要所有的字段, 比如只需要 id 或者 用户名, 我们可以对文档进行“投影”

- 1 - display
- 0 - dont display
```
> db.users.find( {}, {username: 1} )

> db.users.find( {}, {age: 1, _id: 0} )
```
### 2.4 forEach()
```
> db.posts.find().forEach(fucntion(doc) { print('Blog Post: ' + doc.title) })
```
### 2.5 其他查询方式
#### 2.5.1 正则表达式
```
$ db.collection.find({field:/正则表达式/})

$ db.collection.find({字段:/正则表达式/})

$ db.collection.find({nickname:/^p/})
```
#### 2.5.2 比较查询
<, <=, >, >= 这些操作符也是很常用的, 格式如下:
```
db.collection.find({ "field" : { $gt: value }}) // 大于: field > value
db.collection.find({ "field" : { $lt: value }}) // 小于: field < value
db.collection.find({ "field" : { $gte: value }}) // 大于等于: field >= value
db.collection.find({ "field" : { $lte: value }}) // 小于等于: field <= value
db.collection.find({ "field" : { $ne: value }}) // 不等于: field != value
```
#### 2.5.3 包含查询
包含使用 $in 操作符. 示例：查询评论的集合中 userid 字段包含 1003 或 1004的文档
```
db.comment.find({userid:{$in:["1003","1004"]}})
```
不包含使用 $nin 操作符. 示例：查询评论集合中 userid 字段不包含 1003 和 1004 的文档
```
db.comment.find({userid:{$nin:["1003","1004"]}})
```
## 2.6 常用命令小结
```
选择切换数据库：use articledb
插入数据：db.comment.insert({bson数据})
查询所有数据：db.comment.find();
条件查询数据：db.comment.find({条件})
查询符合条件的第一条记录：db.comment.findOne({条件})
查询符合条件的前几条记录：db.comment.find({条件}).limit(条数)
查询符合条件的跳过的记录：db.comment.find({条件}).skip(条数)

修改数据：db.comment.update({条件},{修改后的数据})
        或
        db.comment.update({条件},{$set:{要修改部分的字段:数据})

修改数据并自增某字段值：db.comment.update({条件},{$inc:{自增的字段:步进值}})

删除数据：db.comment.remove({条件})
统计查询：db.comment.count({条件})
模糊查询：db.comment.find({字段名:/正则表达式/})
条件比较运算：db.comment.find({字段名:{$gt:值}})
包含查询：db.comment.find({字段名:{$in:[值1, 值2]}})
        或
        db.comment.find({字段名:{$nin:[值1, 值2]}})

条件连接查询：db.comment.find({$and:[{条件1},{条件2}]})
           或
           db.comment.find({$or:[{条件1},{条件2}]})
```
## 3. 文档间的对应关系

- 一对一 (One To One)
- 一对多 (One To Many)
- 多对多 (Many To Many)

举个例子, 比如“用户-订单”这个一对多的关系中, 我们想查询某一个用户的所有或者某个订单, 我们可以
```
db.users.findOne( {username: "username_here"} )._id
db.orders.find( {user_id: user_id} )
```
## 4. MongoDB 的索引
### 4.1 概述
索引支持在 MongoDB 中高效地执行查询.如果没有索引, MongoDB 必须执行全集合扫描, 即扫描集合中的每个文档, 以选择与查询语句 匹配的文档.这种扫描全集合的查询效率是非常低的, 特别在处理大量的数据时, 查询可以要花费几十秒甚至几分钟, 这对网站的性能是非常致命的.
如果查询存在适当的索引, MongoDB 可以使用该索引限制必须检查的文档数.
索引是特殊的数据结构, 它以易于遍历的形式存储集合数据集的一小部分.索引存储特定字段或一组字段的值, 按字段值排序.索引项的排 序支持有效的相等匹配和基于范围的查询操作.此外, MongoDB 还可以使用索引中的排序返回排序结果.
**MongoDB 使用的是 B Tree, MySQL 下的Innodb 引擎使用的是 B+ Tree**
```bash
// create index 创建索引
db.<collection_name>.createIndex({ userid : 1, username : -1 })

// retrieve indexes 展示所有索引
db.<collection_name>.getIndexes()

// remove indexes 移除index索引
db.<collection_name>.dropIndex(index)

// there are 2 ways to remove indexes:
// 1. removed based on the index name
// 2. removed based on the fields

db.<collection_name>.dropIndex( "userid_1_username_-1" )
db.<collection_name>.dropIndex({ userid : 1, username : -1 })

// remove all the indexes, will only remove non_id indexes  移除全部索引
db.<collection_name>.dropIndexes()
```
### 4.2 索引的类型

- -1 : 倒序排列
- 1 : 正序排列
#### 4.2.1 单字段索引
MongoDB 支持在文档的单个字段上创建用户定义的**升序/降序索引**, 称为**单字段索引** Single Field Index
对于单个字段索引和排序操作, 索引键的排序顺序（即升序或降序）并不重要, 因为 MongoDB 可以在任何方向上遍历索引.

![image-20240410105258398](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105258398.png)

按照`score`升序进行排列

#### 4.2.2 复合索引
MongoDB 还支持多个字段的用户定义索引, 即复合索引 Compound Index
复合索引中列出的字段顺序具有重要意义.例如, 如果复合索引由 { userid: 1, score: -1 } 组成, 则索引首先按 userid 正序排序, 然后 在每个 userid 的值内, 再在按 score 倒序排序.

![image-20240410105308464](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105308464.png)

按照`userid`进行升序排列,再按照`score`进行倒序排列

#### 4.2.3 其他索引

- 地理空间索引 Geospatial Index
- 文本索引 Text Indexes
- 哈希索引 Hashed Indexes
##### 地理空间索引（Geospatial Index）
为了支持对地理空间坐标数据的有效查询, MongoDB 提供了两种特殊的索引: 返回结果时使用平面几何的二维索引和返回结果时使用球面几何的二维球面索引.
##### 文本索引（Text Indexes）
MongoDB 提供了一种文本索引类型, 支持在集合中搜索字符串内容.这些文本索引不存储特定于语言的停止词（例如 “the”, “a”, “or”）, 而将集合中的词作为词干, 只存储根词.
> 如果涉及到文本索引,建议使用elk进行搜索.

##### 哈希索引（Hashed Indexes）
为了支持基于散列的分片, MongoDB 提供了散列索引类型, 它对字段值的散列进行索引.这些索引在其范围内的值分布更加随机, 但只支持相等匹配, 不支持基于范围的查询.
### 4.3 索引的管理操作
#### 4.3.1 索引的查看
语法
```
db.collection.getIndexes()
```
默认 _id 索引： MongoDB 在创建集合的过程中, 在 _id 字段上创建一个唯一的索引, 默认名字为 _id , 该索引可防止客户端插入两个具有相同值的文 档, 不能在 _id 字段上删除此索引.
注意：该索引是**唯一索引**, 因此值不能重复, 即 _id 值不能重复的.
在分片集群中, 通常使用 _id 作为**片键**.
#### 4.3.2 索引的创建
语法
```
db.collection.createIndex(keys, options)
```
参数

![image-20240410105324852](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105324852.png)

options（更多选项)列表

![image-20240410105335091](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105335091.png)

> 常用项: 
> `background`: 项目上线后,创建索引时,后台创建索引,不阻塞其他数据库操作.
> `sparse`: 针对文档中不存在的字段数据不启用索引,为true时,在索引字段中不会查出不包含对应字段的文档.

**注意在 3.0.0 版本前创建索引方法为 db.collection.ensureIndex() , 之后的版本使用了 db.collection.createIndex() 方法, ensureIndex() 还能用, 但只是 createIndex() 的别名.**
举个🌰

```
$  db.comment.createIndex({userid:1})
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}

$ db.comment.createIndex({userid:1,nickname:-1})
...
```
#### 4.3.3 索引的删除
语法
```
# 删除某一个索引
$ db.collection.dropIndex(index)

# 删除全部索引
$ db.collection.dropIndexes()
```
提示:
**_id 的字段的索引是无法删除的, 只能删除非 _id 字段的索引(默认索引)**
示例
```
# 删除 comment 集合中 userid 字段上的升序索引
$ db.comment.dropIndex({userid:1})
```
### 4.4 索引使用
#### 4.4.1 执行计划
分析查询性能 (Analyze Query Performance) 通常使用执行计划 (解释计划 - Explain Plan) 来查看查询的情况
```
$ db.<collection_name>.find( query, options ).explain(options)
```
比如: 查看根据 user_id 查询数据的情况
**未添加索引之前**
"stage" : "COLLSCAN", 表示全集合扫描

![image-20240410105350514](MongoDB%20%E4%BD%BF%E7%94%A8%E4%B8%8ECRUD%E6%93%8D%E4%BD%9C.assets/image-20240410105350514.png)

**添加索引之后**
"stage" : "IXSCAN", 基于索引的扫描

#### 4.4.2 涵盖的查询
当查询条件和查询的投影仅包含索引字段是, MongoDB 直接从索引返回结果, 而不扫描任何文档或将文档带入内存, 这些覆盖的查询十分有效
> [https://docs.mongodb.com/manual/core/query-optimization/#covered-query](https://docs.mongodb.com/manual/core/query-optimization/#covered-query)

## 5. Springboot 整合 MongoDB

Spring Boot 与 MongoDB 的整合为开发人员提供了一个灵活高效的方法来处理数据存储和检索。MongoDB 是一种文档数据库，它存储数据在类似 JSON 的格式中，使得数据结构能够非常灵活。Spring Boot 则提供了快速开发的能力，通过简化配置来加速应用程序的开发。整合这两者，可以创建能够高效处理大量数据并且易于扩展的应用程序。
以下是使用 Spring Boot 与 MongoDB 整合的基本步骤及相关配置：
### 1. 添加依赖
首先，需要在你的 Spring Boot 项目的 pom.xml 文件中添加 MongoDB 的依赖。
```
xmlCopy code
<dependencies>
    <!-- Spring Boot 的 MongoDB Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

    <!-- Spring Boot 的 Web Starter，如果你的项目是 Web 应用 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
### 2. 配置 MongoDB
接着，需要在 application.properties 或 application.yml 文件中配置 MongoDB 的连接信息。
application.yml 示例：
```
spring:
  data:
    mongodb:
      # 主机地址
      host: 127.0.0.1
      # 数据库
      database: comment
      # 默认端口
      port: 27017
      # 使用uri连接
      # mongodb://用户名:密码@localhost:27017/数据库名
#      uri: mongodb://127.0.0.1:27017/article
```
### 3. 创建实体类
定义一个实体类（Model）来映射 MongoDB 中的文档。使用 @Document 标记类和 @Id 标记文档的ID字段。
```java
package pbad.com.service.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.index.Indexed;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.mongodb.core.mapping.Field;

import java.io.Serializable;
import java.time.LocalDateTime;
import java.util.Date;

@Data
@AllArgsConstructor
@NoArgsConstructor
// 绑定到mongodb的article集合
@Document(collection = "comment")
// 创建复合索引,user_id正序排列,nickname倒序排列
// @CompoundIndex(def = "{'user_id':1,'nickname':-1}")
public class Comment implements Serializable {

    /**
     * 自增id
     */
    // @Id,主键标识,该属性的值会自动对应mongodb的主键字段("_id"),如果该属性名就叫"id",则该注解可以省略,否则必须填写
    @Id
    private String id;

    /**
     * 评论内容
     */
    // @Field注解映射对应的mongodb字段,如果一致,无需该注解
    @Field("content")
    private String content;

    /**
     * 发布时间
     */
    @Field("publishtime")
    private Date publishTime;

    /**
     * 发布人id
     */
    // 添加了一个单字段的索引
//    @Indexed
    private String userId;

    /**
     * 发布人昵称
     */
    private String nickName;

    /**
     * 创建时间
     */
    @Field("createdatetime")
    private LocalDateTime createDateTime;

    /**
     * 点赞数
     */
    private Integer likeNum;

    /**
     * 回复数
     */
    private Integer replyNum;

    /**
     * 状态
     */
    private String state;

    /**
     * 父评论id
     */
    private String parentId;

    /**
     * 文章id
     */
    private String articleId;

    @Override
    public String toString() {
        return "Comment{" +
                "id='" + id + '\'' +
                ", content='" + content + '\'' +
                ", publishTime=" + publishTime +
                ", userId='" + userId + '\'' +
                ", nickName='" + nickName + '\'' +
                ", createDateTime=" + createDateTime +
                ", likeNum=" + likeNum +
                ", replyNum=" + replyNum +
                ", state='" + state + '\'' +
                ", parentId='" + parentId + '\'' +
                ", articleId='" + articleId + '\'' +
                '}';
    }
}

```
### 4. 创建仓库接口
定义一个接口来扩展 MongoRepository，为你的实体类提供 CRUD 操作。
```java
public interface CommentRepository extends MongoRepository<Comment, String> {

    /**
     * 查询父级评论为parentid的子评论.
     * 方法名要求严格: findByXXX属性名,是class对象内的属性名,不是mongodb中的属性名
     *
     * @param parentid 父级id
     * @param pageable 分页参数
     * @return 评论集合
     */
    Page<Comment> findByParentId(String parentid, Pageable pageable);

    // 可以根据需要添加自定义查询方法
}

```
### 5. 服务层和控制器
在服务层（Service）处理业务逻辑，在控制器（Controller）中定义路由来处理 HTTP 请求。
Service 示例：
```java
@Service
public class CommentService {

    @Autowired
    private CommentRepository commentRepository;

    /**
     * 保存评论
     *
     * @param comment 评论
     */
    public void saveComment(Comment comment) {
        commentRepository.save(comment);
    }

    /**
     * 更新评论
     *
     * @param comment 评论
     */
    public void updateComment(Comment comment) {
        commentRepository.save(comment);
    }

    /**
     * 删除评论.
     *
     * @param id 文档id
     */
    public void deleteComment(String id) {
        commentRepository.deleteById(id);
    }

    /**
     * 查询所有评论
     *
     * @return 评论列表
     */
    public List<Comment> findCommentList() {
        return commentRepository.findAll();
    }

    /**
     * 根据id查询评论.
     *
     * @param id 文档id
     * @return 评论
     */
    public Comment findCommentById(String id) {
        return commentRepository.findById(id).get();
    }

    /**
     * 查询父级评论为parentid的子评论.
     *
     * @param parentId 父级id
     * @param page     分页参数
     * @param size     分页参数
     * @return 评论集合
     */
    public Page<Comment> findCommentListByParentId(String parentId, int page, int size) {
        return commentRepository.findByParentId(parentId, (Pageable) PageRequest.of(page - 1, size));
    }

    /**
     * 评论点赞 - 更新所有对象,效率低
     */
    public void commentLike(String id) {
        Comment comment = commentRepository.findById(id).get();
        comment.setLikeNum(comment.getLikeNum() + 1);
        commentRepository.save(comment);
    }

    @Autowired
    private MongoTemplate mongoTemplate;

    /**
     * 评论点赞 - 局部更新,效率高
     */
    public void commentLike2(String id) {
        // 查询对象
        Query query = Query.query(Criteria.where("_id").is(id));
        // 更新对象
        Update update = new Update();
        // 局部刷新,相当于$set
//        update.set(key,value);

        // 递增
        update.inc("likeNum");

        // 仅更新query条件查出来的第一个值,updateFirst(查询对象,更新对象,集合的名字或实体类的class)
        mongoTemplate.updateFirst(query, update, "comment");
//        mongoTemplate.updateFirst(query,update,Comment.class);

        // 根据query条件批量更新,updateMulti(查询对象,更新对象,集合的名字或实体类的class)
        mongoTemplate.updateMulti(query,update,Comment.class);
    }
}

```
Controller 示例：
```java
@RestController
@RequestMapping("/comment")
public class UserController {
    @Autowired
    private CommentService commentService;

    // 定义处理请求的方法，例如 GET, POST
}
```
### 6. 测试类
```java
/**
 * 评论业务层测试类
 *
 * @author pangdi
 * @version 1.0
 * @date 2024年04月09日 20:35
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class CommentServiceTest {

    @Autowired
    private CommentService commentService;

    /**
     * 查询所有评论
     */
    @Test
    public void testFindCommentList() {
        List<Comment> commentList = commentService.findCommentList();
        commentList.forEach(System.out::println);
    }

    /**
     * 查询单个评论
     */
    @Test
    public void testFindCommentById() {
        Comment comment = commentService.findCommentById("6615379a7c6e4870ef16c9bc");
        System.out.println(comment);
    }

    /**
     * 创建评论
     */
    @Test
    public void testCreateComment() {
        Comment comment = new Comment();
        comment.setContent("测试评论");
        comment.setPublishTime(new Date());
        comment.setUserId("003");
        comment.setNickName("大筒木辉夜");
        comment.setCreateDateTime(LocalDateTime.now());
        comment.setLikeNum(0);
        comment.setReplyNum(0);
        comment.setParentId("6615379a7c6e4870ef16c9bc");
        comment.setArticleId("1");
        commentService.saveComment(comment);

        commentService.findCommentList().forEach(System.out::println);
    }

    /**
     * 分页查询评论
     */
    @Test
    public void testPageCommentList() {
        Page<Comment> page = commentService.findCommentListByParentId("6615379a7c6e4870ef16c9bc", 1, 3);
        System.out.println(page.getTotalPages());
        System.out.println(page.getTotalElements());
    }

    /**
     * 评论点赞
     */
    @Test
    public void testLikeComment() {
        // 测试点赞,全部更新
//        commentService.commentLike("6615379a7c6e4870ef16c9bc");
//         测试点赞,局部更新
        commentService.commentLike2("6615379a7c6e4870ef16c9bc");
        System.out.println("点赞成功!!!");
    }
}

```
