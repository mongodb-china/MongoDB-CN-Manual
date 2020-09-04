# [ ](#)SQL 到聚合映射图表

[]()

在本页面

*   [例子](#examples)

[聚合管道](../Aggregation-Pipeline.md)允许 MongoDB 提供与 SQL 中许多 common 数据聚合操作相对应的本机聚合功能。

以下 table 概述了 common SQL 聚合术语，函数和概念以及相应的 MongoDB [聚合运算符]()：

| SQL 术语，函数和概念  | Mongo聚合命令                        |
| --------------------- | ------------------------------------ |
| WHERE                 | [$match]()                           |
| GROUP BY              | [$group]()                           |
| HAVING                | [$match]()                           |
| SELECT                | [$project]()                         |
| ORDER BY              | [$sort]()                            |
| LIMIT                 | [$limit]()                           |
| SUM()                 | [$sum]()                             |
| COUNT()               | [$sum]()<br/>[$sortByCount]()        |
| join                  | [$lookup]()                          |
| SELECT INTO NEW_TABLE | [$out]()                             |
| MERGE INTO TABLE      | [$merge]() （从MongoDB 4.2开始可用） |


有关所有聚合管道和表达式 operators 的列表，请参阅[聚合管道快速参考](Aggregation-Pipeline-Quick-Reference.md)。

> **也可以看看**
>
> SQL 到 MongoDB 映射图表](SQL-to-Aggregation-Mapping-Chart.md)

[]()

## <span id="examples">例子</span>

以下 table 提供了 SQL 聚合 statements 和相应的 MongoDB statements 的快速 reference。 table 中的示例假定以下条件：

*   SQL 示例假设两个表`orders`和`order_lineitem`由`order_lineitem.order_id`和`orders.id`列连接。

*   MongoDB 示例假设一个集合`orders`包含以下原型的文档：

    ```powershell
    {
      cust_id: "abc123",
      ord_date: ISODate("2012-11-02T17:04:11.102Z"),
      status: 'A',
      price: 50,
      items: [ { sku: "xxx", qty: 25, price: 1 },
               { sku: "yyy", qty: 25, price: 1 } ]
    }
    ```
    
    

| SQL语句                                                      | MongoDB语句                                                  | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **SELECT** **COUNT**(\*) **AS** count **FROM** orders        | db.orders.aggregate( [ { $group: { _id: null, count: { $sum: 1  }  } } ] ) | 计算来自`orders`的所有记录                                   |
| **SELECT** **SUM**(price) **AS** total **FROM** orders       | db.orders.aggregate( [ { $group: { _id: null, total: { $sum: "$price" } } } ] ) | `orders`中对`price`字段求和                                  |
| **SELECT** cust_id,        **SUM**(price) **AS** total **FROM** orders **GROUP BY** cust_id | db.orders.aggregate( [ { $group: { _id: "$cust_id", total: { $sum: "$price" } } } ] ) | 对于每个唯一`cust_id`，对`price`字段进行求和。               |
| **SELECT** cust_id,        **SUM**(price) **AS** total **FROM** orders **GROUP BY** cust_id **ORDER BY** total | db.orders.aggregate( [ { $group: { _id: "$cust_id", total: { $sum: "$price" } } }, { $sort: { total: 1 } } ] ) | 对于每个唯一`cust_id`，求和`price`字段，结果按总和排序。     |
| **SELECT** cust_id,        ord_date,        **SUM**(price) **AS** total **FROM** orders **GROUP BY** cust_id,          ord_date | db.orders.aggregate( [ { $group: { _id: {  cust_id: "$cust_id", ord_date: { $dateToString: { format: "%Y-%m-%d", date: "$ord_date" }} },  total: { $sum: "$price" } } } ] ) | 对于每个唯一的`cust_id`，通过`ord_date`分组，将`price`字段相加。排除 data 的 time 部分。 |
| **SELECT** cust_id,        **count**(\*) **FROM** orders **GROUP BY** cust_id **HAVING** **count**(\*) > 1 | db.orders.aggregate( [ { $group: { _id: "$cust_id", count: { $sum: 1 } } }, { $match: { count: { $gt: 1 } } } ] ) | 对于具有多个记录的`cust_id`，返回`cust_id`和相应的 record 计数。 |
| **SELECT** cust_id,        ord_date,        **SUM**(price) **AS** total **FROM** orders **GROUP BY** cust_id,          ord_date **HAVING** total > 250 | db.orders.aggregate( [ { $group: { _id: { cust_id: "$cust_id", ord_date: { $dateToString: { format: "%Y-%m-%d", date: "$ord_date" }} }, total: { $sum: "$price" } } }, { $match: { total: { $gt: 250 } } } ] ) | 对于每个唯一的`cust_id`，通过`ord_date`分组，仅在总和大于 250 的情况下对`price`字段和 return 求和。排除 date 的 time 部分 |
| **SELECT** cust_id,        SUM(price) **as** total **FROM** orders **WHERE** status = 'A' **GROUP BY** cust_id | db.orders.aggregate( [ { $match: { status: 'A' } },    { $group: { _id: "$cust_id", total: { $sum: "$price" } } } ] ) | 对于状态为`A`的每个唯一`cust_id`，请对`price`字段求和。      |
| **SELECT** cust_id,        **SUM**(price) **as** total **FROM** orders **WHERE** status = 'A' **GROUP BY** cust_id **HAVING** total > 250 | db.orders.aggregate( [ { $match: { status: 'A' } }, { $group: { _id: "$cust_id", total: { $sum: "$price" } } }, { $match: { total: { $gt: 250 } } } ] ) | 对于状态为`A`的每个唯一`cust_id`，仅对总和大于 250 的`price`字段和 return 求和。 |
| **SELECT** cust_id,        **SUM**(li.qty) **as** qty **FROM** orders o,      order_lineitem li **WHERE** li.order_id = o.id **GROUP BY** cust_id | db.orders.aggregate( [   { $unwind: "$items" }, { $group: { _id: "$cust_id", qty: { $sum: "$items.qty" } } } ] ) | 对于每个唯一`cust_id`，将与订单关联的相应 line item `qty`字段相加。 |
| **SELECT** **COUNT**(\*) **FROM** (**SELECT** cust_id,             ord_date      **FROM** orders      **GROUP** **BY** cust_id,               ord_date)      **as** DerivedTable | db.orders.aggregate( [ { $group: { _id: { cust_id: "$cust_id", ord_date: { $dateToString: { format: "%Y-%m-%d", date: "$ord_date" }} } } }, { $group: { _id: **null**, count: { $sum: 1 } } } ] ) | 计算不同的`cust_id`，`ord_date`分组的数量。排除 date 的 time 部分。 |




> **也可以看看**<br />

*   [SQL 到 MongoDB 映射图表](SQL-to-Aggregation-Mapping-Chart.md)

*   [聚合管道快速参考](Aggregation-Pipeline-Quick-Reference.md)

*   [db.collection.aggregate()](../../Reference/mongo-Shell-Methods/Collection-Methods/db-collection-aggregate.md)



译者：李冠飞

校对：