
# 聚合管道更新
从MongoDB 4.2开始，您可以将聚合管道用于更新操作。 通过更新操作，聚合管道可以包括以下阶段：

|                                                              |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| [$addFields](https://docs.mongodb.com/manual/reference/operator/aggregation/addFields/#pipe._S_addFields) | [$set](https://docs.mongodb.com/manual/reference/operator/aggregation/set/#pipe._S_set) |
| [$project](https://docs.mongodb.com/manual/reference/operator/aggregation/project/#pipe._S_project) | [$unset](https://docs.mongodb.com/manual/reference/operator/aggregation/unset/#pipe._S_unset) |
| [$replaceRoot](https://docs.mongodb.com/manual/reference/operator/aggregation/replaceRoot/#pipe._S_replaceRoot) | [$replaceWith](https://docs.mongodb.com/manual/reference/operator/aggregation/replaceWith/#pipe._S_replaceWith) |

使用聚合管道允许使用表达性更强的update语句，比如根据当前字段值表示条件更新，或者使用另一个字段的值更新一个字段。

## 例1

创建一个示例**students**学生集合（如果该集合当前不存在，则插入操作将创建该集合）：

```powershell
db.students.insertMany([
   { _id: 1, test1: 95, test2: 92, test3: 90, modified: new Date("01/05/2020") },
   { _id: 2, test1: 98, test2: 100, test3: 102, modified: new Date("01/05/2020") },
   { _id: 3, test1: 95, test2: 110, modified: new Date("01/04/2020") }
])
```

要验证，请查询集合：

```powershell
db.students.find()
```

以下[`db.collection.updateOne()`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne)操作使用聚合管道使用**_id**更新文档：**3**：

```shell
db.students.updateOne( { _id: 3 }, [ { $set: { "test3": 98, modified: "$$NOW"} } ] )
```

具体地说，管道包括[`$set`](https://docs.mongodb.com/master/reference/operator/aggregation/set/#pipe._S_set)阶段，该阶段将**test3**字段（并将其值设置为**98**）添加到文档中，并将修改后的字段设置为当前日期时间。 对于当前日期时间，该操作将聚合变量[`NOW`](https://docs.mongodb.com/master/reference/aggregation-variables/#variable.NOW) 用于（以访问变量，以**$$**为前缀并用引号引起来）。

要验证更新，您可以查询集合：

```powershell
db.students.find().pretty()
```

## 例2

创建一个示例**students2**集合(如果该集合当前不存在，则插入操作将创建该集合):

```shell
db.students2.insertMany([
		{ "_id" : 1, quiz1: 8, test2: 100, quiz2: 9, modified: new Date("01/05/2020") }, 
		{ "_id" : 2, quiz2: 5, test1: 80, test2: 89, modified: new Date("01/05/2020") },
])
```

要验证，请查询集合：

```powershell
db.students2.find()
```

以下[`db.collection.updateMany()`](https://docs.mongodb.com/master/reference/method/db.collection.updateMany/#db.collection.updateMany) 操作使用聚合管道来标准化文档的字段（即,集合中的文档应具有相同的字段）并更新修改后的字段：

```shell
db.students2.updateMany( {},
	[
		{ $replaceRoot: { newRoot: 
			{ $mergeObjects: [ { quiz1: 0, quiz2: 0, test1: 0, test2: 0 }, "$$ROOT" ] } 
	} },
		{ $set: { modified: "$$NOW"}  }
	]
)
```

具体来说，管道包括：

* [`$replaceRoot`](https://docs.mongodb.com/master/reference/operator/aggregation/replaceRoot/#pipe._S_replaceRoot) 阶段，带有 [`$mergeObjects`](https://docs.mongodb.com/master/reference/operator/aggregation/mergeObjects/#exp._S_mergeObjects)表达式，可为**quiz1**，**quiz2**，**test1**和**test2**字段设置默认值。 聚集变量[`ROOT`](https://docs.mongodb.com/master/reference/aggregation-variables/#variable.ROOT) 指的是正在修改的当前文档（以访问变量，以**$$**为前缀并用引号引起来）。 当前文档字段将覆盖默认值。

* [`$set`](https://docs.mongodb.com/master/reference/operator/aggregation/set/#pipe._S_set) 阶段用于将修改的字段更新到当前日期时间。 对于当前日期时间，该操作将聚合变量[NOW](#)用于（以访问变量，以**$$**为前缀并用引号引起来）。

要验证更新，您可以查询集合：

```powershell
db.students2.find()
```

### 例3

创建一个示例**students3**集合（如果该集合当前不存在，则插入操作将创建该集合）：

```powershell
db.students3.insert([
   { "_id" : 1, "tests" : [ 95, 92, 90 ], "modified" : ISODate("2019-01-01T00:00:00Z") },
   { "_id" : 2, "tests" : [ 94, 88, 90 ], "modified" : ISODate("2019-01-01T00:00:00Z") },
   { "_id" : 3, "tests" : [ 70, 75, 82 ], "modified" : ISODate("2019-01-01T00:00:00Z") }
]);
```

要验证，请查询集合：

```shell
db.students3.find()
```

以下 [`db.collection.updateMany()`](https://docs.mongodb.com/master/reference/method/db.collection.updateMany/#db.collection.updateMany)操作使用聚合管道以计算的平均成绩和字母成绩更新文档。 

```shell
   db.students3.updateMany(
   		{ }, 
   		[
   			{ $set: { average : { $trunc: [ { $avg: "$tests" }, 0 ] }, modified: "$$NOW" } },  
   			{ $set: { grade: { $switch:                      
  							 branches: [                     
  										 { case: { $gte: [ "$average", 90 ] }, then: "A" },     
  										 { case: { $gte: [ "$average", 80 ] }, then: "B" },  
  										 { case: { $gte: [ "$average", 70 ] }, then: "C" },   
  										 { case: { $gte: [ "$average", 60 ] }, then: "D" }   
  									 ],
  										 default: "F"   
  		 } } } }
   		]
   )
```

具体来说，管道包括：

* [`$set`](https://docs.mongodb.com/master/reference/operator/aggregation/set/#pipe._S_set)阶段来计算测试数组元素的截断平均值，并将修改后的字段更新为当前日期时间。 要计算截断的平均值，此阶段使用**$avg**和[`$trunc`](https://docs.mongodb.com/master/reference/operator/aggregation/trunc/#exp._S_trunc) 表达式。 对于当前日期时间，该操作将聚合变量[`NOW`](https://docs.mongodb.com/master/reference/aggregation-variables/#variable.NOW) 用于(以访问变量，以**$$**为前缀并用引号引起来).

* 一个[`$set`](https://docs.mongodb.com/master/reference/operator/aggregation/set/#pipe._S_set) 阶段，用于使用[`$switch`](https://docs.mongodb.com/master/reference/operator/aggregation/switch/#exp._S_switch) 表达式根据平均值添加年级字段。

 要验证更新，您可以查询集合：

 ```powershell
 db.students3.find()
 ```

## 例4

创建一个示例**students4**集合(如果该集合当前不存在，则插入操作将创建该集合)：

```powershell
db.students4.insertMany([
  { "_id" : 1, "quizzes" : [ 4, 6, 7 ] },
  { "_id" : 2, "quizzes" : [ 5 ] },
  { "_id" : 3, "quizzes" : [ 10, 10, 10 ] }
])
```

要验证，请查询集合：

```powershell
 db.students4.find()
```

以下[`db.collection.updateOne()`](https://docs.mongodb.com/master/reference/method/db.collection.updateOne/#db.collection.updateOne)操作使用聚合管道将测验分数添加到具有**_id**的文档中：**2**：

```powershell
db.students4.updateOne( { _id: 2 },
  [ { $set: { quizzes: { $concatArrays: [ "$quizzes", [ 8, 6 ]  ] } } } ]
)
```

要验证，请查询集合：

```powershell
db.students4.find()
```

 ## 例5

 创建一个示例**temperatures**集合，其中包含摄氏温度(如果该集合当前不存在，则插入操作将创建该集合)：

 ```powershell
db.temperatures.insertMany([
  { "_id" : 1, "date" : ISODate("2019-06-23"), "tempsC" : [ 4, 12, 17 ] },
  { "_id" : 2, "date" : ISODate("2019-07-07"), "tempsC" : [ 14, 24, 11 ] },
  { "_id" : 3, "date" : ISODate("2019-10-30"), "tempsC" : [ 18, 6, 8 ] }
])
 ```

 要验证，请查询集合：

 ```powershell
db.temperatures.find()
 ```

以下[`db.collection.updateMany()`](https://docs.mongodb.com/master/reference/method/db.collection.updateMany/#db.collection.updateMany)操作使用聚合管道以华氏度中的相应温度更新文档：

```powershell
db.temperatures.updateMany( { },
  [
    { $addFields: { "tempsF": {
          $map: {
             input: "$tempsC",
             as: "celsius",
             in: { $add: [ { $multiply: ["$$celsius", 9/5 ] }, 32 ] }
          }
    } } }
  ]
)
```

具体来说，管道由[`$addFields`](https://docs.mongodb.com/master/reference/operator/aggregation/addFields/#pipe._S_addFields)阶段组成，以添加一个新的数组字段**tempsF**，其中包含华氏温度。 要将**tempsC**数组中的每个摄氏温度转换为华氏温度，该阶段将[`$map`](https://docs.mongodb.com/master/reference/operator/aggregation/map/#exp._S_map)表达式与[`$add`](https://docs.mongodb.com/master/reference/operator/aggregation/add/#exp._S_add)和 [`$multiply`](https://docs.mongodb.com/master/reference/operator/aggregation/multiply/#exp._S_multiply)表达式一起使用。

要验证更新，您可以查询集合：

 ```powershell
db.temperatures.find()
 ```

## 其他例子

有关其他示例，另请参见各种更新方法页面：

- [db.collection.updateOne](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#updateone-example-agg)
- [db.collection.updateMany](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#updatemany-example-agg)
- [db.collection.update()](https://docs.mongodb.com/manual/reference/method/db.collection.update/#update-example-agg)
- [db.collection.findOneAndUpdate()](https://docs.mongodb.com/manual/reference/method/db.collection.findOneAndUpdate/#findoneandupdate-agg-pipeline)
- [db.collection.findAndModify()](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#findandmodify-agg-pipeline)
- [Bulk.find.update()](https://docs.mongodb.com/manual/reference/method/Bulk.find.update/#example-bulk-find-update-agg)
- [Bulk.find.updateOne()](https://docs.mongodb.com/manual/reference/method/Bulk.find.updateOne/#example-bulk-find-update-one-agg)
- [Bulk.find.upsert()](https://docs.mongodb.com/manual/reference/method/Bulk.find.upsert/#bulk-find-upsert-update-agg-example)



译者：杨帅

校对：杨帅