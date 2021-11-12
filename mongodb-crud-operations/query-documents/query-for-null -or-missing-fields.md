# 查询Null或者不存在的字段

在MongoDB中不同的查询操作符对于**null**值处理方式不同。

本文提供了使用 mongo shell 中的[db.collection.find()](https://docs.mongodb.com/v4.0/reference/method/db.collection.find/#db.collection.find) 方法查询**null**值的操作案例。案例中使用的**inventory**集合数据可以通过下面的语句产生。

```javascript
db.inventory.insertMany([
   { _id: 1, item: null },
   { _id: 2 }
])
```



## 等值匹配

当使用**{item:null}**作为查询条件的时候，返回的是**item**字段值为**null**的文档或者不包含**item**字段的文档。

```javascript
db.inventory.find( { item: null } )
```

该查询返回inventory集合中的所有文档。



## 类型检查

当使用**{item:{$type:10}}**作为查询条件的时候，仅返回item字段值为null的文档。**item**字段的值是[BSON TYPE](https://docs.mongodb.com/v4.0/reference/bson-types/) **NULL**(type number 10)

```javascript
db.inventory.find( { item : { $type: 10 } } )
```

该查询仅返回**item**字段值为**null**的文档。



## 存在检查

当使用**{item:{$exists:false}}**作为查询条件的时候，返回不包含**item**字段的文档。

```javascript
db.inventory.find( { item : { $exists: false } } )
```

该查询仅返回不包含item字段的文档。



## 相关文档

[$type](https://docs.mongodb.com/manual/reference/operator/query/type/#mongodb-query-op.-type)

[$exists](https://docs.mongodb.com/manual/reference/operator/query/exists/#mongodb-query-op.-exists)



原文链接：https://docs.mongodb.com/manual/tutorial/query-for-null-fields/

译者：张芷嘉

