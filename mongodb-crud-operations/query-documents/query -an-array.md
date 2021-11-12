# 数组查询

本文提供了使用mongo shell中[db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find) 方法查询数组的操作案例。案例中使用的**inventory**集合数据可以通过下面的语句产生。

```javascript
db.inventory.insertMany([
   { item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
   { item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
   { item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
   { item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
   { item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] }
]);
```



## 数组查询

数组字段做等值查询的时候，使用查询文档**{<field>:<value>}**其中 **<value>**是要精确匹配的数组，包含元素的顺序。

下面的案例返回inventory集合中数组字段**tags**值是**只包含两个元素"red","blank"并且有指定顺序的数组**的所有文档:

```javascript
db.inventory.find( { tags: ["red", "blank"] } )
```



如果想检索数组中包含**"red"**,**"blank"**两个元素并且不在乎元素顺序或者数组中是否有其它元素。可以使用[$all](https://docs.mongodb.com/manual/reference/operator/query/all/#mongodb-query-op.-all)操作符:

```javascript
db.inventory.find( { tags: { $all: ["red", "blank"] } } )
```



## 查询数组中的元素

检索数组字段中至少一个元素等于指定的值，使用**<field>:<value>**的形式，其中**<value>**是一个元素值。

下面的案例返回inventory集合中数组字段**tags**中有一个元素的值是**"red"**的所有文档:

```javascript
db.inventory.find( { tags: "red" } )
```



对数组中的元素进行检索的时候，可以使用[查询操作符](https://docs.mongodb.com/manual/reference/operator/query/#std-label-query-selectors)在[查询过滤文档](https://docs.mongodb.com/manual/core/document/#std-label-document-query-filter)中。

```javascript
{ <array field>: { <operator1>: <value1>, ... } }
```



下面的案例返回inventory集合中数组字段**dim_cm**中最少有一个元素的值大于**25**的所有文档。

```javascript
db.inventory.find( { dim_cm: { $gt: 25 } } )
```



## 多条件查询数组中的元素

使用多条件查询数组中的元素时，可以在查询语句中指定单个数组元素满足所有查询条件还是多个数组中的元素联合满足所有条件。



#### 使用多条件查询数组中的元素

下面的案例返回inventory集合中数组字段**dim_cm**中单个元素同时满足大于15并且小于20，或者一个元素满足大于**15**，另外一个元素小于**20**的所有文档:

```javascript
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )
```



#### 数组中的元素同时满足多个查询条件 

使用[$elemMatch](https://docs.mongodb.com/v4.0/reference/operator/query/elemMatch/#op._S_elemMatch)来指定多个查询条件在数组中的元素上，数组中最少一个元素同时满足所有的查询条件。



下面的案例返回数组字段**dim_cm**中最少一个元素同时满足大于([$gt](https://docs.mongodb.com/v4.0/reference/operator/query/gt/#op._S_gt))**22** 并且 小于([$lt](https://docs.mongodb.com/v4.0/reference/operator/query/lt/#op._S_lt)) **30**:

```javascript
db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )
```



#### 使用数组下标查询数组中的元素

使用[点号](https://docs.mongodb.com/v4.0/reference/glossary/#term-dot-notation)，可以为数组中指定下标的元素指定查询条件，数组下标从0开始。

><font color=Green>Note:</font>
>
>当使用点号的时候，字段和嵌套文档字段必须在引号内



下面的案例返回数组字段**dim_cm**中第二个元素大于**25**的所有文档:

```javascript
db.inventory.find( { "dim_cm.1": { $gt: 25 } } )
```



#### 使用数组长度来检索

使用[$size](https://docs.mongodb.com/v4.0/reference/operator/query/size/#op._S_size)操作符通过数组中的元素个数来进行检索。

下面的查询返回数组字段**tags**中有三个元素的所有文档 :

```javascript
db.inventory.find( { "tags": { $size: 3 } } )
```



### 其它查询导航

#### 其它查询案例:

* [文档查询](https://docs.mongodb.com/v4.0/tutorial/query-documents/)
* [嵌套文档查询](https://docs.mongodb.com/v4.0/tutorial/query-embedded-documents/)
* [数组嵌套文档查询](https://docs.mongodb.com/v4.0/tutorial/query-array-of-documents/)



原文链接：https://docs.mongodb.com/manual/tutorial/query-arrays/

译者：张芷嘉



