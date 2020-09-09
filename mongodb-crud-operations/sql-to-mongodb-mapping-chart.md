# SQL到MongoDB的映射图表

**在本页面**

* [术语和概念](sql-to-mongodb-mapping-chart.md#terminology)
* [可执行性文件](sql-to-mongodb-mapping-chart.md#Executables)
* [例子](sql-to-mongodb-mapping-chart.md#Examples)
* [进一步阅读](sql-to-mongodb-mapping-chart.md#Reading)

除了下面的图表之外，您可能需要考虑有关MongoDB的常见问题的常见问题部分。

## 术语和概念

下表介绍了各种SQL术语和概念以及相应的MongoDB术语和概念。

| SQL术语/概念 | MongoDB术语/概念 |
| :--- | :--- |
| database | [database](https://docs.mongodb.com/manual/reference/glossary/#term-database) |
| table | [collection](https://docs.mongodb.com/manual/reference/glossary/#term-collection) |
| row | [document](https://docs.mongodb.com/manual/reference/glossary/#term-document) or [BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson) document |
| column | [field](https://docs.mongodb.com/manual/reference/glossary/#term-field) |
| index | [index](https://docs.mongodb.com/manual/reference/glossary/#term-index) |
| table joins | [$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup), 嵌入文档 |
| primary key （指定任何唯一的列或列组合作为主键。） | [primary key](https://docs.mongodb.com/manual/reference/glossary/#term-primary-key) （在MongoDB中，主键自动设置为\_id字段。） |
| aggregation \(e.g. group by\) | aggregation pipeline See the [SQL to Aggregation Mapping Chart](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/). |
| SELECT INTO NEW\_TABLE | [$out](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out) See the [SQL to Aggregation Mapping Chart](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/). |
| MERGE INTO TABLE | [$merge](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#pipe._S_merge) \(Available starting in MongoDB 4.2\) See the [SQL to Aggregation Mapping Chart](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/). |
| Transactions | [transactions](https://docs.mongodb.com/manual/core/transactions/) 在许多情况下，非规范化数据模型（嵌入式文档和数组） 将继续是您数据和用例的最佳选择，而不是多文档事务。 也就是说，在许多情况下，对数据进行适当的建模将最 大程度地减少对多文档交易的需求。  |

## 可执行文件

下表展示了一些数据库可执行文件和相应的MongoDB可执行文件。这个表格并不是详尽无遗的。

|  | MongoDB | MySQL | Oracle | Informix | DB2 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Database Server | [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) | mysqld | oracle | IDS | DB2 Server |
| Database Client | [mongo](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) | mysql | sqlplus | DB-Access | DB2 Client |

## 例子

下表展示了各种SQL语句和相应的MongoDB语句。表中的例子假设以下条件:

* SQL示例假设有一个名为**people**的表。
* MongoDB示例假设一个名为**people**的集合，它包含以下原型的文档:

```text
 { 
       _id: ObjectId("509a8fb2f3f4948bd2f983a0"),
       user_id: "abc123",
       age: 55,
       status: 'A'
 }
```

### 创建和修改

下表展示了与表级操作相关的各种SQL语句以及相应的MongoDB语句。

| SQL Schema语句 | MongoDB Schema语句 |
| :--- | :--- |
| **CREATE** **TABLE** people \(     id MEDIUMINT **NOT** **NULL**         AUTO\_INCREMENT,     user\_id Varchar\(30\),     age Number,     status char\(1\),     **PRIMARY** **KEY** \(id\) \) | 隐式创建的第一个[`insertOne()`](https://docs.mongodb.com/master/reference/method/db.collection.insertOne/#db.collection.insertOne)或[`insertMany()`](https://docs.mongodb.com/master/reference/method/db.collection.insertMany/#db.collection.insertMany)操作。如果没有指定**\_id**字段，则会自动添加主键\_id。 db.people.insertOne\( {     user\_id: "abc123",     age: 55,     status: "A"  } \) 但是，您也可以显式地创建一个集合: db.createCollection\("people"\) |
| **ALTER** **TABLE** people **ADD** join\_date DATETIME | 集合不描述或不强制其文件结构； 即在集合级别没有结构上的更改。 但是，在文档级别，[updateMany\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)操作可以使用[$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set)运算符将字段添加到现有文档中。 db.people.updateMany\(     { },     { $set: { join\_date: **new** Date\(\) } } \) |
| **ALTER** **TABLE** people **DROP** **COLUMN** join\_date | 集合不描述或不强制其文件结构； 即在集合级别没有结构上的更改。 但是，在文档级别，[updateMany\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)操作可以使用[$unset](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset)运算符将字段添加到现有文档中。 db.people.updateMany\(     { },     { $unset: { "join\_date": "" } } \) |
| **CREATE** **INDEX** idx\_user\_id\_asc **ON** people\(user\_id\) | db.people.createIndex\( { user\_id: 1 } \) |
| **CREATE** **INDEX**        idx\_user\_id\_asc\_age\_desc **ON** people\(user\_id, age **DESC**\) | db.people.createIndex\( { user\_id: 1, age: -1 } \) |
| **DROP** **TABLE** people | db.people.drop\(\) |

有关使用的方法和运算符的更多信息，请参见：

|  |  |  |
| :--- | :--- | :--- |
| [db.collection.insertOne\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne) | [db.collection.updateMany\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany) | [$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set) |
| [db.collection.insertMany\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) | [db.collection.createIndex\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) | [$unset](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset) |
| [db.createCollection\(\)](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection) | [db.collection.drop\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.drop/#db.collection.drop) |  |

**另看：**

* [Databases and Collections](https://docs.mongodb.com/manual/core/databases-and-collections/)
* [Documents](https://docs.mongodb.com/manual/core/document/)
* [Indexes](https://docs.mongodb.com/manual/indexes/)
* [Data Modeling Concepts](https://docs.mongodb.com/manual/core/data-models/)

#### 插入

下表显示了与将记录插入表和相应的MongoDB语句有关的各种SQL语句。

| SQL INSERT语句 | **MongoDB insertOne\(\) Statements** |
| :--- | :--- |
| **INSERT** **INTO** people\(user\_id,                   age,                   status\) **VALUES** \("bcd001",         45,         "A"\) | db.people.insertOne\(    { user\_id: "bcd001", age: 45, status: "A" } \) |

有关更多信息，请参见[`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne)。

也可以看看：

* [`Insert Documents`](https://docs.mongodb.com/manual/tutorial/insert-documents/)
* [`db.collection.insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany)
* [`Databases and Collections`](https://docs.mongodb.com/manual/core/databases-and-collections/)
* [`Documents`](https://docs.mongodb.com/manual/core/document/)

#### 选择

下表展示了与从表中读取记录相关的各种SQL语句以及相应的MongoDB语句。

> **注意**
>
> 除非通过投影明确排除，否则\[[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法始终在返回的文档中包含**\_id**字段。 下面的某些SQL查询可能包含一个**\_id**字段来反映这一点，即使该字段未包含在相应的[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)查询中也是如此。

| SQL SELECT 语句 | MongoDB find\(\) 语句 |
| :--- | :--- |
| **SELECT**  _\*FROM_ people | db.people.find\(\) |
| **SELECT** id,        user\_id,        status **FROM** people | db.people.find\(      { },      { user\_id: 1, status: 1 } \) |
| **SELECT** user\_id, status **FROM** people | db.people.find\(      { },      { user\_id: 1, status: 1, \_id: 0 } \) |
| **SELECT**  _**FROM** people_ _\*WHERE_ status = "A" | db.people.find\(      { status: "A" } \) |
| **SELECT** user\_id, status **FROM** people **WHERE** status = "A" | db.people.find\(      { status: "A" },      { user\_id: 1, status: 1, \_id: 0 } \) |
| **SELECT**  _**FROM** people_ _\*WHERE_ status != "A" | db.people.find\(      { status: { $ne: "A" } } \) |
| **SELECT**  _**FROM** people_ _**WHERE** status = "A"_ _\*AND_ age = 50 | db.people.find\(      { status: "A",        age: 50 } \) |
| **SELECT**  _**FROM** people_ _**WHERE** status = "A"_ _\*OR_ age = 50 | db.people.find\(      { $or: \[ { status: "A" } , { age: 50 } \] } \) |
| **SELECT**  _**FROM** people_ _\*WHERE_ age &gt; 25 | db.people.find\(      { age: { $gt: 25 } } \) |
| **SELECT**  _**FROM** people_ _\*WHERE_ age &lt; 25 | db.people.find\(     { age: { $lt: 25 } } \) |
| **SELECT**  _**FROM** people_ _**WHERE** age &gt; 25_ _\*AND_   age &lt;= 50 | db.people.find\(     { age: { $gt: 25, $lte: 50 } } \) |
| **SELECT**  _**FROM** people_ _**WHERE** user\_id \*like_ "%bc%" | db.people.find\( { user_id: /bc/ } \)_ _\_or_ db.people.find\( { user\_id: { $regex: /bc/ } } \) |
| **SELECT**  _**FROM** people_ _**WHERE** user\_id \*like_ "bc%" | db.people.find\( { user_id: /^bc/ } \)_ _\_or_ db.people.find\( { user\_id: { $regex: /^bc/ } } \) |
| **SELECT**  _**FROM** people_ _**WHERE** status = "A"_ _**ORDER** **BY** user\_id \*ASC_ | db.people.find\( { status: "A" } \).sort\( { user\_id: 1 } \) |
| **SELECT**  _**FROM** people_ _**WHERE** status = "A"_ _**ORDER** **BY** user\_id \*DESC_ | db.people.find\( { status: "A" } \).sort\( { user\_id: -1 } \) |
| **SELECT** **COUNT**\(_\)_ _\*FROM_ people | db.people.count\(\) _or_ db.people.find\(\).count\(\) |
| **SELECT** **COUNT**\(user\_id\) **FROM** people | db.people.count\( { user_id: { $exists: **true** } } \)_ _\_or_ db.people.find\( { user\_id: { $exists: **true** } } \).count\(\) |
| **SELECT** **COUNT**\(_\)_ _**FROM** people_ _\*WHERE_ age &gt; 30 | db.people.count\( { age: { $gt: 30 } } \) _or_ db.people.find\( { age: { $gt: 30 } } \).count\(\) |
| **SELECT** **DISTINCT**\(status\) **FROM** people | db.people.aggregate\( \[ { $group : { \_id : "$status" } } \] \) or, for distinct value sets that do not exceed the [BSON size limit](https://docs.mongodb.com/manual/reference/limits/#limit-bson-document-size) db.people.distinct\( "status" \) |
| **SELECT**  _**FROM** people_ _\*LIMIT_ 1 | db.people.findOne\(\) _or_ db.people.find\(\).limit\(1\) |
| **SELECT**  _**FROM** people_ _\*LIMIT_ 5 SKIP 10 | db.people.find\(\).limit\(5\).skip\(10\) |
| **EXPLAIN** **SELECT**  _**FROM** people_ _\*WHERE_ status = "A" | db.people.find\( { status: "A" } \).explain\(\) |

有关使用的方法和运算符的更多信息，请参见：

|  |  |
| :--- | :--- |
| .[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) | .[`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne) |
| .[`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) | .[`$and`](https://docs.mongodb.com/manual/reference/operator/query/and/#op._S_and) |
| .[`db.collection.findOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOne/#db.collection.findOne) | .[`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#op._S_or) |
| .[`limit()`](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit) | .[`$gt`](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt) |
| .[`skip()`](https://docs.mongodb.com/manual/reference/method/cursor.skip/#cursor.skip) | .[`$lt`](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt) |
| .[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) | .[`$exists`](https://docs.mongodb.com/manual/reference/operator/query/exists/#op._S_exists) |
| .[`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort) | .[`$lte`](https://docs.mongodb.com/manual/reference/operator/query/lte/#op._S_lte) |
| .[`count()`](https://docs.mongodb.com/manual/reference/method/cursor.count/#cursor.count) | .[`$regex`](https://docs.mongodb.com/manual/reference/operator/query/regex/#op._S_regex) |

另看：

* [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)
* [Query and Projection Operators](https://docs.mongodb.com/manual/reference/operator/query/)
* [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/)

#### 更新记录

下表显示了与更新表中的现有记录和相应的MongoDB语句有关的各种SQL语句。

| **SQL Update Statements** | **MongoDB updateMany\(\) Statements** |
| :--- | :--- |
| **UPDATE** people **SET** status = "C" **WHERE** age &gt; 25 | db.people.updateMany\(    { age: { $gt: 25 } },    { $set: { status: "C" } } \) |
| **UPDATE** people **SET** age = age + 3 **WHERE** status = "A" | db.people.updateMany\(        { status: "A" } ,        { $inc: { age: 3 } } \) |

有关示例中使用的方法和运算符的更多信息，请参见：

* [db.collection.updateMany\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)
* [$gt](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt)
* [$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set)
* [$inc](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)

另看：

* [Update Documents](https://docs.mongodb.com/manual/tutorial/update-documents/)
* [Update Operators](https://docs.mongodb.com/manual/reference/operator/update/)
* [db.collection.updateOne\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)
* [db.collection.replaceOne\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne)

#### 删除记录

下表显示了与从表中删除记录和相应的MongoDB语句有关的各种SQL语句。

| **SQL Delete Statements** | **MongoDB deleteMany\(\) Statements** |
| :--- | :--- |
| **DELETE** **FROM** people **WHERE** status = "D" | db.people.deleteMany\( { status: "D" } \) |
| **DELETE** **FROM** people | db.people.deleteMany\({}\) |

获得更多信息，请参见：[db.collection.deleteMany\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany).

另看：

* [Delete Documents](https://docs.mongodb.com/manual/tutorial/remove-documents/)
* [db.collection.deleteOne\(\)](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne)

#### 进一步阅读

如果您正在考虑将SQL应用程序迁移到MongoDB，请下载[《 MongoDB应用程序现代化指南》](https://www.mongodb.com/modernize?tck=docs_server)。

下载内容包括以下资源：

* 演示使用MongoDB进行数据建模的方法
* 白皮书涵盖了从RDBMS数据模型迁移到MongoDB的最佳实践和注意事项
* 参考MongoDB模式及其等效RDBMS
* 应用程序现代化记分卡

译者：杨帅

校对：杨帅

