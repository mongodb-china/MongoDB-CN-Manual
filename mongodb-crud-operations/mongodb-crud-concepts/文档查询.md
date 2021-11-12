# 文档查询

本文提供了使用mongo shell中[db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find)方法查询的案例。案例中使用的**inventory**集合数据可以通过下面的语句产生。

```javascript
db.inventory.insertMany([
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
```



## 检索集合中的所有文档

如果想检索集合中的**所有文档**，可以在find方法中传一个**空文档**作为查询过滤条件。查询过滤参数确定选择条件：

```javascript
db.inventory.find( {} )
```



上述操作对应如下SQL语句：

```javascript
SELECT * FROM inventory
```

有关该方法语法的更多信息，请参阅 [find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find)。



## 等值查询

在[查询过滤文档](https://docs.mongodb.com/manual/core/document/#std-label-document-query-filter)中使用**<字段>:<值>**表达式实现等值查询：

```javascript
{ <field1>: <value1>, ... }
```



下面的案例返回inventory**集合中**status**等于**"D"**的所有文档:

```javascript
db.inventory.find( { status: "D" } )
```



上述操作对应如下SQL语句：

```javascript
SELECT * FROM inventory WHERE status = "D"
```



## 查询条件中使用查询操作符

[查询过滤文档](https://docs.mongodb.com/manual/core/document/#std-label-document-query-filter)中可以使用[查询操作符](https://docs.mongodb.com/manual/reference/operator/query/)来指定多个条件，格式如下:

```javascript
{ <field1>: { <operator1>: <value1> }, ... }
```



下面的案例返回**inventory**集合中**status**等于**"A"**或**"D"**的所有文档。

```javascript
db.inventory.find( { status: { $in: [ "A", "D" ] } } )
```

><font color=Green>Note:</font>
>
>尽管可以使用[$or](https://docs.mongodb.com/manual/reference/operator/query/or/#mongodb-query-op.-or)操作符来满足上述需求，但是在对相同字段进行等值检索的时候更建议使用[$in](https://docs.mongodb.com/manual/reference/operator/query/in/#mongodb-query-op.-in)。



上述操作对应如下SQL:

```javascript
SELECT * FROM inventory WHERE status in ("A", "D")
```

有关**MongoDB**查询运算符的完整列表，请参考[查询和映射操作符](https://docs.mongodb.com/v4.0/reference/operator/query/)



## AND条件 

可以指定文档中的多个字段作为查询条件。在查询语句中使用AND连接多个查询条件来检索集合中满足所有查询条件的文档。



下面的案例返回**inventory**集合中**status**等于**"A" **并且**qty**小于([$lt](https://docs.mongodb.com/manual/reference/operator/query/lt/#mongodb-query-op.-lt))**30**的所有文档:

```javascript
db.inventory.find( { status: "A", qty: { $lt: 30 } } )
```



上述操作对应如下SQL:

```javascript
SELECT * FROM inventory WHERE status = "A" AND qty < 30
```

关于MongoDB的比较操作符可以参考[比较操作符](https://docs.mongodb.com/v4.0/reference/operator/query-comparison/#query-selectors-comparison)



## OR条件

使用[$or](https://docs.mongodb.com/v4.0/reference/operator/query/or/#op._S_or)运算符，可以指定一个联合查询，该查询将每个子句与逻辑 OR 连接起来，以便查询选择集合中至少匹配一个条件的文档。



下面的案例返回inventory集合中**status**等于**"A" **或者**qty**小于([$lt](https://docs.mongodb.com/manual/reference/operator/query/lt/#mongodb-query-op.-lt))30的所有文档。

```javascript
db.inventory.find( { $or: [ { status: "A" }, { qty: { $lt: 30 } } ] } )
```



上述操作对应如下SQL:

```javascript
SELECT * FROM inventory WHERE status = "A" OR qty < 30
```

><font color=Green>Note:</font>
>
>使用[比较操作符](https://docs.mongodb.com/v4.0/reference/operator/query-comparison/#query-selectors-comparison)的查询受[Type Bracketing](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#type-bracketing)的约束。



## 同时使用AND和OR条件

下面的案例返回inventory集合中**status**等于**"A" **并且**qty**小于 ([$lt](https://docs.mongodb.com/manual/reference/operator/query/lt/#mongodb-query-op.-lt)) **30**或者**item** 是以**p**字符开头的所有文档。

```javascript
db.inventory.find( {
     status: "A",
     $or: [ { qty: { $lt: 30 } }, { item: /^p/ } ]
} )
```



上述操作对应如下SQL:

```javascript
SELECT * FROM inventory WHERE status = "A" AND ( qty < 30 OR item LIKE "p%")
```

><font color=Green>Note:</font>
>
>MongoDB支持正则表达式操作符[$regex](https://docs.mongodb.com/manual/reference/operator/query/regex/#mongodb-query-op.-regex)来做字符串模式匹配。



## 其他查询教程

其他查询案例:

* [嵌套文档查询](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/)
* [数组查询](https://docs.mongodb.com/manual/tutorial/query-arrays/)
* [数组中的嵌套文档查询](https://docs.mongodb.com/manual/tutorial/query-array-of-documents/)
* [查询语句中返回指定字段](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/)
* [查询Null或者不存在的字段](https://docs.mongodb.com/manual/tutorial/query-for-null-fields/)



## 行为

**游标** 

使用 [db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find)方法返回检索到文档的一个[游标](https://docs.mongodb.com/v4.0/tutorial/iterate-a-cursor/)。



**读隔离**

*新增加于MongoDB3.2版本*

对于[副本集](https://docs.mongodb.com/manual/replication/)或者[分片副本集](https://docs.mongodb.com/manual/sharding/)的查询，读关注允许客户端选择读的隔离级别。更多的信息可以查看[Read Concern](https://docs.mongodb.com/v4.0/reference/read-concern/)



### 其它的方法

下面的方法也可以从集合中查询文档:

* [db.collection.findOne](https://docs.mongodb.com/v4.0/reference/method/db.collection.findOne/#db.collection.findOne)
* 在[聚合管道](https://docs.mongodb.com/v4.0/core/aggregation-pipeline/)中，[$match](https://docs.mongodb.com/v4.0/reference/operator/aggregation/match/#pipe._S_match)管道阶段提供了MongoDB的查询过滤。   

><font color=Green>Note:</font>
>
>[db.collection.findOne](https://docs.mongodb.com/v4.0/reference/method/db.collection.findOne/#db.collection.findOne) 方法提供了返回单个文档的读操作。
>
>实际上，[db.collection.findOne](https://docs.mongodb.com/v4.0/reference/method/db.collection.findOne/#db.collection.findOne) 就是[db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find) 方法后面加了个限制条数1。





原文链接：https://docs.mongodb.com/manual/tutorial/query-documents/

译者：张芷嘉
