
# 按需物化视图


注意

本页的内容讨论了按需物化视图。有关视图的讨论，请参阅[视图](https://docs.mongodb.com/manual/core/views/)。

从4.2版本开始，MongoDB为[aggregation pipeline](https://docs.mongodb.com/manual/core/aggregation-pipeline/)添加了[`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)阶段。此阶段可以将管道结果合并到现有集合中，而不是完全替换现有集合。此功能允许用户创建按需物化视图，每次运行管道时都可以更新输出集合的内容。


## 示例


假设现在接近2019年1月末，集合`bakesales`包含按项目分类的销售信息：

```
db.bakesales.insertMany( [
   { date: new ISODate("2018-12-01"), item: "Cake - Chocolate", quantity: 2, amount: new NumberDecimal("60") },
   { date: new ISODate("2018-12-02"), item: "Cake - Peanut Butter", quantity: 5, amount: new NumberDecimal("90") },
   { date: new ISODate("2018-12-02"), item: "Cake - Red Velvet", quantity: 10, amount: new NumberDecimal("200") },
   { date: new ISODate("2018-12-04"), item: "Cookies - Chocolate Chip", quantity: 20, amount: new NumberDecimal("80") },
   { date: new ISODate("2018-12-04"), item: "Cake - Peanut Butter", quantity: 1, amount: new NumberDecimal("16") },
   { date: new ISODate("2018-12-05"), item: "Pie - Key Lime", quantity: 3, amount: new NumberDecimal("60") },
   { date: new ISODate("2019-01-25"), item: "Cake - Chocolate", quantity: 2, amount: new NumberDecimal("60") },
   { date: new ISODate("2019-01-25"), item: "Cake - Peanut Butter", quantity: 1, amount: new NumberDecimal("16") },
   { date: new ISODate("2019-01-26"), item: "Cake - Red Velvet", quantity: 5, amount: new NumberDecimal("100") },
   { date: new ISODate("2019-01-26"), item: "Cookies - Chocolate Chip", quantity: 12, amount: new NumberDecimal("48") },
   { date: new ISODate("2019-01-26"), item: "Cake - Carrot", quantity: 2, amount: new NumberDecimal("36") },
   { date: new ISODate("2019-01-26"), item: "Cake - Red Velvet", quantity: 5, amount: new NumberDecimal("100") },
   { date: new ISODate("2019-01-27"), item: "Pie - Chocolate Cream", quantity: 1, amount: new NumberDecimal("20") },
   { date: new ISODate("2019-01-27"), item: "Cake - Peanut Butter", quantity: 5, amount: new NumberDecimal("80") },
   { date: new ISODate("2019-01-27"), item: "Tarts - Apple", quantity: 3, amount: new NumberDecimal("12") },
   { date: new ISODate("2019-01-27"), item: "Cookies - Chocolate Chip", quantity: 12, amount: new NumberDecimal("48") },
   { date: new ISODate("2019-01-27"), item: "Cake - Carrot", quantity: 5, amount: new NumberDecimal("36") },
   { date: new ISODate("2019-01-27"), item: "Cake - Red Velvet", quantity: 5, amount: new NumberDecimal("100") },
   { date: new ISODate("2019-01-28"), item: "Cookies - Chocolate Chip", quantity: 20, amount: new NumberDecimal("80") },
   { date: new ISODate("2019-01-28"), item: "Pie - Key Lime", quantity: 3, amount: new NumberDecimal("60") },
   { date: new ISODate("2019-01-28"), item: "Cake - Red Velvet", quantity: 5, amount: new NumberDecimal("100") },
] );
```




### 1.定义按需物化视图



下面的`updateMonthlySales`函数定义了一个`monthlybakesales`物化视图，其中包含累积的每月销售信息。在示例中，该函数采用了一个日期参数来更新从特定日期开始的每月销售信息。

```
updateMonthlySales = function(startDate) {
   db.bakesales.aggregate( [
      { $match: { date: { $gte: startDate } } },
      { $group: { _id: { $dateToString: { format: "%Y-%m", date: "$date" } }, sales_quantity: { $sum: "$quantity"}, sales_amount: { $sum: "$amount" } } },
      { $merge: { into: "monthlybakesales", whenMatched: "replace" } }
   ] );
};
```



- [`$match`](https://docs.mongodb.com/manual/reference/operator/aggregation/match/#mongodb-pipeline-pipe.-match)阶段过滤数据以仅处理那些销售额大于或等于`startDate`

- 阶段按年-月对销售信息进行分组。此阶段输出的文档具有以下形式：

  ```
  { "_id" : "<YYYY-mm>", "sales_quantity" : <num>, "sales_amount" : <NumberDecimal> }
  ```

- [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)阶段将输出写入到`monthlybakesales`集合

  基于[on](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-on)`_id`字段（未分片输出集合的默认值），此阶段会检查聚合结果中的文档是否 [匹配](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-whenMatched) 集合中的现有文档：

  + [当匹配时](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-whenMatched)（即同年月的文档已经存在于集合中），此阶段会使用来自聚合结果的文档[替换现有文档](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-whenMatched-replace)；
  + [当不匹配时](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-whenNotMatched)，此阶段将聚合结果中的文档插入到集合中（不匹配时的默认行为）。





### 2. 执行初始运行



对于初始运行，你可以传入一个日期`new ISODate("1970-01-01")`：

```
updateMonthlySales(new ISODate("1970-01-01"));
```

初始运行后，`monthlybakesales`包含以下文档；即`db.monthlybakesales.find().sort( { _id: 1 } )`返回以下内容：

```
{ "_id" : "2018-12", "sales_quantity" : 41, "sales_amount" : NumberDecimal("506") }
{ "_id" : "2019-01", "sales_quantity" : 86, "sales_amount" : NumberDecimal("896") }
```





### 3. 刷新物化视图



假设到了2019年2月的第一周，`bakesales`集合更新了新的销售信息；具体来说就是一月和二月新增的销售。

```
db.bakesales.insertMany( [
   { date: new ISODate("2019-01-28"), item: "Cake - Chocolate", quantity: 3, amount: new NumberDecimal("90") },
   { date: new ISODate("2019-01-28"), item: "Cake - Peanut Butter", quantity: 2, amount: new NumberDecimal("32") },
   { date: new ISODate("2019-01-30"), item: "Cake - Red Velvet", quantity: 1, amount: new NumberDecimal("20") },
   { date: new ISODate("2019-01-30"), item: "Cookies - Chocolate Chip", quantity: 6, amount: new NumberDecimal("24") },
   { date: new ISODate("2019-01-31"), item: "Pie - Key Lime", quantity: 2, amount: new NumberDecimal("40") },
   { date: new ISODate("2019-01-31"), item: "Pie - Banana Cream", quantity: 2, amount: new NumberDecimal("40") },
   { date: new ISODate("2019-02-01"), item: "Cake - Red Velvet", quantity: 5, amount: new NumberDecimal("100") },
   { date: new ISODate("2019-02-01"), item: "Tarts - Apple", quantity: 2, amount: new NumberDecimal("8") },
   { date: new ISODate("2019-02-02"), item: "Cake - Chocolate", quantity: 2, amount: new NumberDecimal("60") },
   { date: new ISODate("2019-02-02"), item: "Cake - Peanut Butter", quantity: 1, amount: new NumberDecimal("16") },
   { date: new ISODate("2019-02-03"), item: "Cake - Red Velvet", quantity: 5, amount: new NumberDecimal("100") }
] )
```

为了刷新1月和2月的`monthlybakesales`数据，需要再次运行该函数以重新运行聚合管道，日期参数值从`new ISODate("2019-01-01")`开始。

```
updateMonthlySales(new ISODate("2019-01-01"));
```

`monthlybakesales`的内容已更新，并能反映出`bakesales`集合中的最新数据；即`db.monthlybakesales.find().sort( { _id: 1 } )`返回以下内容：

```
{ "_id" : "2018-12", "sales_quantity" : 41, "sales_amount" : NumberDecimal("506") }
{ "_id" : "2019-01", "sales_quantity" : 102, "sales_amount" : NumberDecimal("1142") }
{ "_id" : "2019-02", "sales_quantity" : 15, "sales_amount" : NumberDecimal("284") }
```






## 附加信息



 [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)阶段:

+ 可以输出到相同或不同数据库中的集合。
+ 如果输出集合不存在，则会创建一个新集合。
+ 可以将结果（插入新文档、合并文档、替换文档、保留现有文档、操作失败、使用自定义更新管道处理文档）合并到现有集合中。
+ 可以输出到分片的集合中。输入集合也可以是分片集合。

参考[`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)：

+ 有关[`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#mongodb-pipeline-pipe.-merge)和可用选项的更多信息
+ 示例：[按需物化视图：初始创建](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-mat-view-init-creation)
+ 示例：[按需物化视图：更新/替换数据](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-mat-view-refresh)
+ 示例：[仅插入新数据](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#std-label-merge-mat-view-insert-only)



原文链接：https://docs.mongodb.com/manual/core/materialized-views/

译者：李正洋
