
# SQL到MongoDB的映射图表
**在本页面**

*  [术语和概念](#terminology)
*  [可执行性文件](#Executables)
*  [例子](#Examples)
* [进一步阅读](#Reading)

除了下面的图表之外，您可能需要考虑有关MongoDB的常见问题的常见问题部分。

## <span id="terminology">术语和概念</span>

下表介绍了各种SQL术语和概念以及相应的MongoDB术语和概念。

| SQL术语/概念 | MongoDB术语/概念 |
| --- | --- |
| database | [database](https://docs.mongodb.com/manual/reference/glossary/#term-database) |
| table | [collection](https://docs.mongodb.com/manual/reference/glossary/#term-collection) |
| row | [document](https://docs.mongodb.com/manual/reference/glossary/#term-document) or [BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson) document |
| column | [field](https://docs.mongodb.com/manual/reference/glossary/#term-field) |
| index | [index](https://docs.mongodb.com/manual/reference/glossary/#term-index) |
| table joins | [$lookup](https://docs.mongodb.com/manual/reference/operator/aggregation/lookup/#pipe._S_lookup), 嵌入文档 |
| primary key<br />（指定任何唯一的列或列组合作为主键。） | [primary key](https://docs.mongodb.com/manual/reference/glossary/#term-primary-key)<br />（在MongoDB中，主键自动设置为_id字段。） |
| aggregation (e.g. group by) | aggregation pipeline<br />See the [SQL to Aggregation Mapping Chart](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/). |
| SELECT INTO NEW_TABLE | [$out](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out)<br />See the [SQL to Aggregation Mapping Chart](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/). |
| MERGE INTO TABLE | [$merge](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#pipe._S_merge) (Available starting in MongoDB 4.2)<br />See the [SQL to Aggregation Mapping Chart](https://docs.mongodb.com/manual/reference/sql-aggregation-comparison/). |
| Transactions | [transactions](https://docs.mongodb.com/manual/core/transactions/)<br />在许多情况下，非规范化数据模型（嵌入式文档和数组）<br />将继续是您数据和用例的最佳选择，而不是多文档事务。<br />也就是说，在许多情况下，对数据进行适当的建模将最<br />大程度地减少对多文档交易的需求。<br /> |

## <span id="Executables">可执行文件</span>

下表展示了一些数据库可执行文件和相应的MongoDB可执行文件。这个表格并不是详尽无遗的。

|  | MongoDB | MySQL | Oracle | Informix | DB2 |
| --- | --- | --- | --- | --- | --- |
| Database Server | [mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) | mysqld | oracle | IDS | DB2 Server |
| Database Client | [mongo](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) | mysql | sqlplus | DB-Access | DB2 Client |

## <span id="Examples">例子</span>

下表展示了各种SQL语句和相应的MongoDB语句。表中的例子假设以下条件:

* SQL示例假设有一个名为**people**的表。
* MongoDB示例假设一个名为**people**的集合，它包含以下原型的文档:

```shell
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
| --- | --- |
| **CREATE** **TABLE** people (<br />    id MEDIUMINT **NOT** **NULL**<br />        AUTO_INCREMENT,<br />    user_id Varchar(30),<br />    age Number,<br />    status char(1),<br />    **PRIMARY** **KEY** (id)<br />) | 隐式创建的第一个[`insertOne()`](https://docs.mongodb.com/master/reference/method/db.collection.insertOne/#db.collection.insertOne)或[`insertMany()`](https://docs.mongodb.com/master/reference/method/db.collection.insertMany/#db.collection.insertMany)操作。如果没有指定**_id**字段，则会自动添加主键_id。<br/>db.people.insertOne( {<br />    user_id: "abc123",<br />    age: 55,<br />    status: "A"<br /> } )<br />但是，您也可以显式地创建一个集合:<br />db.createCollection("people") |
| **ALTER** **TABLE** people<br />**ADD** join_date DATETIME | 集合不描述或不强制其文件结构； 即在集合级别没有结构上的更改。<br />但是，在文档级别，[updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)操作可以使用[$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set)运算符将字段添加到现有文档中。<br />db.people.updateMany(<br />    { },<br />    { $set: { join_date: **new** Date() } }<br />) |
| **ALTER** **TABLE** people<br />**DROP** **COLUMN** join_date | 集合不描述或不强制其文件结构； 即在集合级别没有结构上的更改。<br />但是，在文档级别，[updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)操作可以使用[$unset](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset)运算符将字段添加到现有文档中。<br />db.people.updateMany(<br />    { },<br />    { $unset: { "join_date": "" } }<br />) |
| **CREATE** **INDEX** idx_user_id_asc<br />**ON** people(user_id) | db.people.createIndex( { user_id: 1 } ) |
| **CREATE** **INDEX**<br />       idx_user_id_asc_age_desc<br />**ON** people(user_id, age **DESC**) | db.people.createIndex( { user_id: 1, age: -1 } ) |
| **DROP** **TABLE** people | db.people.drop() |

有关使用的方法和运算符的更多信息，请参见：

|                                                              |                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [db.collection.insertOne()](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne) | [db.collection.updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany) | [$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set) |
| [db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) | [db.collection.createIndex()](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) | [$unset](https://docs.mongodb.com/manual/reference/operator/update/unset/#up._S_unset) |
| [db.createCollection()](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection) | [db.collection.drop()](https://docs.mongodb.com/manual/reference/method/db.collection.drop/#db.collection.drop) |                                                              |

**另看：**

- [Databases and Collections](https://docs.mongodb.com/manual/core/databases-and-collections/)

- [Documents](https://docs.mongodb.com/manual/core/document/)

- [Indexes](https://docs.mongodb.com/manual/indexes/)

- [Data Modeling Concepts](https://docs.mongodb.com/manual/core/data-models/)

#### 插入

下表显示了与将记录插入表和相应的MongoDB语句有关的各种SQL语句。

| SQL INSERT语句 | **MongoDB insertOne() Statements** |
| --- | --- |
| **INSERT** **INTO** people(user_id,<br />                  age,<br />                  status)<br />**VALUES** ("bcd001",<br />        45,<br />        "A") | db.people.insertOne(<br />   { user_id: "bcd001", age: 45, status: "A" }<br />) |

有关更多信息，请参见[`db.collection.insertOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne)。

也可以看看：

- [`Insert Documents`](https://docs.mongodb.com/manual/tutorial/insert-documents/)
- [`db.collection.insertMany()`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany)
- [`Databases and Collections`](https://docs.mongodb.com/manual/core/databases-and-collections/)
- [`Documents`](https://docs.mongodb.com/manual/core/document/)

#### 选择

下表展示了与从表中读取记录相关的各种SQL语句以及相应的MongoDB语句。

> **注意**
>
> 除非通过投影明确排除，否则[[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法始终在返回的文档中包含**_id**字段。 下面的某些SQL查询可能包含一个**_id**字段来反映这一点，即使该字段未包含在相应的[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)查询中也是如此。

| SQL SELECT 语句 | MongoDB find() 语句 |
| --- | --- |
| **SELECT** *<br />**FROM** people | db.people.find() |
| **SELECT** id,<br />       user_id,<br />       status<br />**FROM** people | db.people.find(<br />     { },<br />     { user_id: 1, status: 1 }<br />) |
| **SELECT** user_id, status<br />**FROM** people | db.people.find(<br />     { },<br />     { user_id: 1, status: 1, _id: 0 }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** status = "A" | db.people.find(<br />     { status: "A" }<br />) |
| **SELECT** user_id, status<br />**FROM** people<br />**WHERE** status = "A" | db.people.find(<br />     { status: "A" },<br />     { user_id: 1, status: 1, _id: 0 }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** status != "A" | db.people.find(<br />     { status: { $ne: "A" } }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** status = "A"<br />**AND** age = 50 | db.people.find(<br />     { status: "A",<br />       age: 50 }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** status = "A"<br />**OR** age = 50 | db.people.find(<br />     { $or: [ { status: "A" } , { age: 50 } ] }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** age > 25 | db.people.find(<br />     { age: { $gt: 25 } }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** age < 25 | db.people.find(<br />    { age: { $lt: 25 } }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** age > 25<br />**AND**   age <= 50 | db.people.find(<br />    { age: { $gt: 25, $lte: 50 } }<br />) |
| **SELECT** *<br />**FROM** people<br />**WHERE** user_id **like** "%bc%" | db.people.find( { user_id: /bc/ } )<br />_or_<br />db.people.find( { user_id: { $regex: /bc/ } } ) |
| **SELECT** *<br />**FROM** people<br />**WHERE** user_id **like** "bc%" | db.people.find( { user_id: /^bc/ } )<br />_or_<br />db.people.find( { user_id: { $regex: /^bc/ } } ) |
| **SELECT** *<br />**FROM** people<br />**WHERE** status = "A"<br />**ORDER** **BY** user_id **ASC** | db.people.find( { status: "A" } ).sort( { user_id: 1 } ) |
| **SELECT** *<br />**FROM** people<br />**WHERE** status = "A"<br />**ORDER** **BY** user_id **DESC** | db.people.find( { status: "A" } ).sort( { user_id: -1 } ) |
| **SELECT** **COUNT**(*)<br />**FROM** people | db.people.count()<br />_or_<br />db.people.find().count() |
| **SELECT** **COUNT**(user_id)<br />**FROM** people | db.people.count( { user_id: { $exists: **true** } } )<br />_or_<br />db.people.find( { user_id: { $exists: **true** } } ).count() |
| **SELECT** **COUNT**(*)<br />**FROM** people<br />**WHERE** age > 30 | db.people.count( { age: { $gt: 30 } } )<br />_or_<br />db.people.find( { age: { $gt: 30 } } ).count() |
| **SELECT** **DISTINCT**(status)<br />**FROM** people | db.people.aggregate( [ { $group : { _id : "$status" } } ] )<br />or, for distinct value sets that do not exceed the [BSON size limit](https://docs.mongodb.com/manual/reference/limits/#limit-bson-document-size)<br />db.people.distinct( "status" ) |
| **SELECT** *<br />**FROM** people<br />**LIMIT** 1 | db.people.findOne()<br />_or_<br />db.people.find().limit(1) |
| **SELECT** *<br />**FROM** people<br />**LIMIT** 5<br />SKIP 10 | db.people.find().limit(5).skip(10) |
| **EXPLAIN** **SELECT** *<br />**FROM** people<br />**WHERE** status = "A" | db.people.find( { status: "A" } ).explain() |

有关使用的方法和运算符的更多信息，请参见：

|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| .[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) | .[`$ne`](https://docs.mongodb.com/manual/reference/operator/query/ne/#op._S_ne) |
| .[`db.collection.distinct()`](https://docs.mongodb.com/manual/reference/method/db.collection.distinct/#db.collection.distinct) | .[`$and`](https://docs.mongodb.com/manual/reference/operator/query/and/#op._S_and) |
| .[`db.collection.findOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.findOne/#db.collection.findOne) | .[`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/#op._S_or) |
| .[`limit()`](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit) | .[`$gt`](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt) |
| .[`skip()`](https://docs.mongodb.com/manual/reference/method/cursor.skip/#cursor.skip) | .[`$lt`](https://docs.mongodb.com/manual/reference/operator/query/lt/#op._S_lt) |
| .[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) | .[`$exists`](https://docs.mongodb.com/manual/reference/operator/query/exists/#op._S_exists) |
| .[`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort) | .[`$lte`](https://docs.mongodb.com/manual/reference/operator/query/lte/#op._S_lte) |
| .[`count()`](https://docs.mongodb.com/manual/reference/method/cursor.count/#cursor.count) | .[`$regex`](https://docs.mongodb.com/manual/reference/operator/query/regex/#op._S_regex) |

另看：

- [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)
- [Query and Projection Operators](https://docs.mongodb.com/manual/reference/operator/query/)
- [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/)

#### 更新记录

下表显示了与更新表中的现有记录和相应的MongoDB语句有关的各种SQL语句。

| **SQL Update Statements** | **MongoDB updateMany() Statements** |
| --- | --- |
| **UPDATE** people<br />**SET** status = "C"<br />**WHERE** age > 25 | db.people.updateMany(<br />   { age: { $gt: 25 } },<br />   { $set: { status: "C" } }<br />) |
| **UPDATE** people<br />**SET** age = age + 3<br />**WHERE** status = "A" | db.people.updateMany(<br />       { status: "A" } ,<br />       { $inc: { age: 3 } }<br />) |

有关示例中使用的方法和运算符的更多信息，请参见：

- [db.collection.updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)
- [$gt](https://docs.mongodb.com/manual/reference/operator/query/gt/#op._S_gt)
- [$set](https://docs.mongodb.com/manual/reference/operator/update/set/#up._S_set)
- [$inc](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)

另看：

- [Update Documents](https://docs.mongodb.com/manual/tutorial/update-documents/)
- [Update Operators](https://docs.mongodb.com/manual/reference/operator/update/)
- [db.collection.updateOne()](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)
- [db.collection.replaceOne()](https://docs.mongodb.com/manual/reference/method/db.collection.replaceOne/#db.collection.replaceOne)

#### 删除记录

下表显示了与从表中删除记录和相应的MongoDB语句有关的各种SQL语句。

| **SQL Delete Statements** | **MongoDB deleteMany() Statements** |
| --- | --- |
| **DELETE** **FROM** people<br />**WHERE** status = "D" | db.people.deleteMany( { status: "D" } ) |
| **DELETE** **FROM** people | db.people.deleteMany({}) |

获得更多信息，请参见：[db.collection.deleteMany()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany).

另看：

- [Delete Documents](https://docs.mongodb.com/manual/tutorial/remove-documents/)
- [db.collection.deleteOne()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne)

#### <span id="Reading">进一步阅读</span>

如果您正在考虑将SQL应用程序迁移到MongoDB，请下载[《 MongoDB应用程序现代化指南》](https://www.mongodb.com/modernize?tck=docs_server)。

下载内容包括以下资源：

* 演示使用MongoDB进行数据建模的方法
* 白皮书涵盖了从RDBMS数据模型迁移到MongoDB的最佳实践和注意事项
* 参考MongoDB模式及其等效RDBMS
* 应用程序现代化记分卡
  





译者：杨帅

校对：杨帅