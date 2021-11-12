# 嵌套文档查询

本文提供了使用mongo shell中[db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find) 方法查询嵌套文档的操作案例。案例中使用的**inventory**集合数据可以通过下面的语句产生。

```javascript
db.inventory.insertMany( [
   { item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
   { item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
   { item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
   { item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
   { item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }
]);
```



## 嵌套文档查询

对嵌套文档的字段做**等值**查询的时候，使用[query filter document](https://docs.mongodb.com/manual/core/document/#std-label-document-query-filter) **{<field>:<value>}** 其中**<value>**是等值匹配的文档。

下面的案例返回inventory集合中**size**字段的值等于**文档{ h: 14, w: 21, uom: "cm" }**的所有文档。

```javascript
db.inventory.find( { size: { h: 14, w: 21, uom: "cm" } } )
```



对嵌套文档整体做**等值匹配**的时候，要求的是对指定**<value>**文档的精确匹配，包含字段顺序。

下面的案例无法查询到任何文档。

```javascript
db.inventory.find(  { size: { w: 21, h: 14, uom: "cm" } }  )
```



## 嵌套文档中的字段

查询嵌套文档中的字段，使用[dot notation](https://docs.mongodb.com/v4.0/reference/glossary/#term-dot-notation)**("field.nestedField")**.

><font color=Green>Note:</font>
>
>当在查询语句中使用"."，字段和嵌套文档字段必须在引号内。



#### 嵌套文档中的字段等值查询

下面的案例返回inventory集合中**size字段中嵌套文档字段uom**值等于**"in"**的所有文档。

```javascript
db.inventory.find( { "size.uom": "in" } )
```



#### 使用查询操作符查询

在[query filter document](https://docs.mongodb.com/v4.0/core/document/#document-query-filter)中可以使用[查询操作符](https://docs.mongodb.com/v4.0/reference/operator/query/#query-selectors)指定多个查询条件，格式如下:

```javascript
{ <field1>: { <operator1>: <value1> }, ... }
```



下面的查询语句在字段**size**中的嵌套文档字段**h**上面使用([$lt](https://docs.mongodb.com/v4.0/reference/operator/query/lt/#op._S_lt))操作符:

```javascript
db.inventory.find( { "size.h": { $lt: 15 } } )
```



#### 使用AND条件

下面的案例返回inventory集合中**size字段中嵌套文档字段h**值小于**15** 并且 **size字段中嵌套文档字段uom**值等于**"in"** 并且**status**字段等于**"D"**的所有文档。

```javascript
db.inventory.find( { "size.h": { $lt: 15 }, "size.uom": "in", status: "D" } )
```



### 其他查询导航 

#### 其他查询案例:

* [文档查询](https://docs.mongodb.com/v4.0/tutorial/query-documents/)

* [数组查询](https://docs.mongodb.com/v4.0/tutorial/query-arrays/)

* [数组中嵌套文档查询](https://docs.mongodb.com/v4.0/tutorial/query-array-of-documents/)

  

原文链接：https://docs.mongodb.com/manual/tutorial/query-embedded-documents/

译者：张芷嘉
