# 查询数组中的嵌套文档

本文提供了使用 mongo shell 中的[db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find) 方法对数组中嵌套文档进行查询操作的示例。

可以通过下面的语句生成本文使用的**inventory**集合。

```javascript
db.inventory.insertMany( [
   { item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },
   { item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
   { item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
   { item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] },
   { item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
```



### 查询数组中的嵌套文档

下面的案例返回**instock**数组中元素等于指定文档的的所有文档:

```javascript
db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } )
```



当对整个嵌套文档使用等值匹配的时候是要求精确匹配指定文档，包括字段顺序。比如，下面的语句并没有查询到**inventory**集合中的任何文档：

```javascript
db.inventory.find( { "instock": { qty: 5, warehouse: "A" } } )
```



### 指定查询条件在数组嵌套文档的字段上

#### 指定查询条件在数组中嵌套文档的字段上

如果你不知道数组中嵌套文档的下标，使用**(.)**号连接数组字段的名字和数组中嵌套文档中字段的名字。

下面的案例返回**instock**数组中最少有一个嵌套文档包含字段**qty**的值小于等于**20**的所有文档 :

```javascript
db.inventory.find( { 'instock.qty': { $lte: 20 } } )
```



#### 使用数组下标查询数组中嵌套文档中的字段

使用[dot notation](https://docs.mongodb.com/v4.0/reference/glossary/#term-dot-notation)，可以指定查询条件在数组中指定数组下标的嵌套文档的字段上面。数组下标从0开始。

> <font color=Green>Note:</font>
>
> 当查询使用点号的时候，字段和索引必须在引号内。



下面案例返回**instock**数组中的第一个元素是包含字段**qty**小于等于**20**的文档的所有文档：

```javascript
db.inventory.find( { 'instock.0.qty': { $lte: 20 } } )
```



### 指定多个条件检索数组嵌套文档

当对数组中嵌套文档中多个字段指定查询条件的时候，可以在查询语句中指定单个文档满足这些查询条件或者是数组中多个文档联合(单个文档)满足这些查询条件。

#### 单个嵌套文档中的字段满足多个查询条件

使用[$elemMatch](https://docs.mongodb.com/v4.0/reference/operator/query/elemMatch/#op._S_elemMatch)操作符为数组中的嵌套文档指定多个查询条件，最少一个嵌套文档同时满足所有的查询条件。

下面的案例返回**instock**数组中最少有一个嵌套文档包含**qty**等于**5**同时**warhouse**等于**A**的所有文档：

```javascript
db.inventory.find( { "instock": { $elemMatch: { qty: 5, warehouse: "A" } } } )
```



下面的案例返回**instock**数组中最少一个嵌套文档包含字段**qty**大于**10**并且小于**20**的所有文档:

```javascript
db.inventory.find( { "instock": { $elemMatch: { qty: { $gt: 10, $lte: 20 } } } } )
```



#### 多个元素联合满足查询条件

如果数组字段上的联合查询条件没有使用 [$elemMatch](https://docs.mongodb.com/v4.0/reference/operator/query/elemMatch/#op._S_elemMatch)运算符，查询返回数组字段中多个元素联合满足所有的查询条件的所有文档。

下面的案例返回数组字段**instock**中嵌套文档中**qty**字段大于**10**并且数组中其它嵌套文档(不一定是同一个嵌套文档)**qty**字段小于等于**20**的所有文档:

```javascript
db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 } } )
```

下面的案例返回数组字段**instock**中最少一个嵌套文档包含**qty**等于**5**并且最少一个嵌套文档(不一定是同一个嵌套文档)包含**warehouse**字段等于**A**的所有文档:

```javascript
db.inventory.find( { "instock.qty": 5, "instock.warehouse": "A" } )
```



### 其它查询导航

#### 其它查询案例:

* [数组查询](https://docs.mongodb.com/v4.0/tutorial/query-arrays/)
* [文档查询](https://docs.mongodb.com/v4.0/tutorial/query-documents/)
* [嵌套文档查询](https://docs.mongodb.com/v4.0/tutorial/query-embedded-documents/)



原文链接：https://docs.mongodb.com/manual/tutorial/query-array-of-documents/

译者：张芷嘉



