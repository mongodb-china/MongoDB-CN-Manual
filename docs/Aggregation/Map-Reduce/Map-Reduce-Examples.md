# [ ](#)Map-Reduce 例子

[]()

在本页面

*   [返回每位客户的总价格](#return-the-total-price-per-customer)

*   [用每个项目的平均数量计算订单和总数量](#calculate-order-and-total-quantity-with-average-quantity-per-item)

在[mongo]() shell 中，[db.collection.mapReduce()]()方法是[MapReduce]()命令周围的 wrapper。以下示例使用[db.collection.mapReduce()]()方法：

> **聚合管道作为替代**
>
> [聚合管道]() 比map-reduce提供更好的性能和更一致的接口。
>
> 各种map-reduce表达式可以使用被重写[聚合管道运算符]()，诸如[`$group`]()， [`$merge`]()等
>
> 下面的示例包括聚合管道备选方案。

`orders`使用以下文档创建样本集合：

```powershell
db.orders.insertMany([
   { _id: 1, cust_id: "Ant O. Knee", ord_date: new Date("2020-03-01"), price: 25, items: [ { sku: "oranges", qty: 5, price: 2.5 }, { sku: "apples", qty: 5, price: 2.5 } ], status: "A" },
   { _id: 2, cust_id: "Ant O. Knee", ord_date: new Date("2020-03-08"), price: 70, items: [ { sku: "oranges", qty: 8, price: 2.5 }, { sku: "chocolates", qty: 5, price: 10 } ], status: "A" },
   { _id: 3, cust_id: "Busby Bee", ord_date: new Date("2020-03-08"), price: 50, items: [ { sku: "oranges", qty: 10, price: 2.5 }, { sku: "pears", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 4, cust_id: "Busby Bee", ord_date: new Date("2020-03-18"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 5, cust_id: "Busby Bee", ord_date: new Date("2020-03-19"), price: 50, items: [ { sku: "chocolates", qty: 5, price: 10 } ], status: "A"},
   { _id: 6, cust_id: "Cam Elot", ord_date: new Date("2020-03-19"), price: 35, items: [ { sku: "carrots", qty: 10, price: 1.0 }, { sku: "apples", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 7, cust_id: "Cam Elot", ord_date: new Date("2020-03-20"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 8, cust_id: "Don Quis", ord_date: new Date("2020-03-20"), price: 75, items: [ { sku: "chocolates", qty: 5, price: 10 }, { sku: "apples", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 9, cust_id: "Don Quis", ord_date: new Date("2020-03-20"), price: 55, items: [ { sku: "carrots", qty: 5, price: 1.0 }, { sku: "apples", qty: 10, price: 2.5 }, { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" },
   { _id: 10, cust_id: "Don Quis", ord_date: new Date("2020-03-23"), price: 25, items: [ { sku: "oranges", qty: 10, price: 2.5 } ], status: "A" }
])
```

[]()
    

## <span id="return-the-total-price-per-customer">返回每位客户的总价格</span>

对`orders`集合执行map-reduce操作，以对进行分组`cust_id`，并计算`price`每个的 的总和`cust_id`：

1. 定义map函数来处理每个输入文档：

- 在函数中，`this`指的是map-reduce操作正在处理的文档。
- 该函数将映射`price`到`cust_id`每个文档的，并发出`cust_id`和`price`对。

```powershell
var mapFunction1 = function() {
   emit(this.cust_id, this.price);
};
```

2. 使用两个参数`keyCustId`和定义相应的reduce函数 `valuesPrices`：

- `valuesPrices`是一个数组，其元素是`price` 由map功能发射并由分组值`keyCustId`。
- 该函数将`valuesPrice`数组简化为其元素的总和。

```powershell
var reduceFunction1 = function(keyCustId, valuesPrices) {
   return Array.sum(valuesPrices);
};
```

3. `orders`使用`mapFunction1`map函数和`reduceFunction1` reduce函数对集合中的所有文档执行map-reduce 。

```powershell
db.orders.mapReduce(
   mapFunction1,
   reduceFunction1,
   { out: "map_reduce_example" }
)
```

此操作将结果输出到名为的集合 `map_reduce_example`。如果`map_reduce_example`集合已经存在，则该操作将用此map-reduce操作的结果替换内容。

4. 查询`map_reduce_example`集合以验证结果：

```powershell
db.map_reduce_example.find().sort( { _id: 1 } )
```

​	该操作返回以下文档：

```powershell
{ "_id" : "Ant O. Knee", "value" : 95 }
{ "_id" : "Busby Bee", "value" : 125 }
{ "_id" : "Cam Elot", "value" : 60 }
{ "_id" : "Don Quis", "value" : 155 }
```

### 聚合替代

使用可用的聚合管道运算符，您可以重写map-reduce操作，而无需定义自定义函数：

```powershell
db.orders.aggregate([
   { $group: { _id: "$cust_id", value: { $sum: "$price" } } },
   { $out: "agg_alternative_1" }
])
```

1. [`$group`]()由平台组`cust_id`并计算`value`字段（参见`$sum`）。该 `value`字段包含`price`每个的总计`cust_id`。

   该阶段将以下文档输出到下一阶段：

   ```powershell
   { "_id" : "Don Quis", "value" : 155 }
   { "_id" : "Ant O. Knee", "value" : 95 }
   { "_id" : "Cam Elot", "value" : 60 }
   { "_id" : "Busby Bee", "value" : 125 }
   ```

2. 然后，[`$out`]()将输出写入collection `agg_alternative_1`。或者，您可以使用 [`$merge`]()代替[`$out`]()。

3. 查询`agg_alternative_1`集合以验证结果：

   ```powershell
   db.agg_alternative_1.find().sort( { _id: 1 } )
   ```

   该操作返回以下文档：

   ```powershell
   { "_id" : "Ant O. Knee", "value" : 95 }
   { "_id" : "Busby Bee", "value" : 125 }
   { "_id" : "Cam Elot", "value" : 60 }
   { "_id" : "Don Quis", "value" : 155 }
   ```

## <span id="calculate-order-and-total-quantity-with-average-quantity-per-item">用每个项目的平均数量计算订单和总数量</span>

在此示例中，您将对值大于或等于的`orders`所有文档在集合上执行map-reduce操作 。工序按字段分组 ，并计算每个的订单数量和总订购量。然后，该操作将为每个值计算每个订单的平均数量，并将结果合并到输出集合中。合并结果时，如果现有文档的密钥与新结果相同，则该操作将覆盖现有文档。如果不存在具有相同密钥的文档，则该操作将插入该文档。

1. 定义map函数来处理每个输入文档：

   - 在函数中，`this`指的是map-reduce操作正在处理的文档。
   - 对于每个商品，该函数将其`sku`与一个新对象相关联，该对象`value`包含订单的`count`of `1`和该商品`qty`，并发出`sku`and `value`对。

   ```powershell
   var mapFunction2 = function() {
       for (var idx = 0; idx < this.items.length; idx++) {
          var key = this.items[idx].sku;
          var value = { count: 1, qty: this.items[idx].qty };
   
          emit(key, value);
       }
   };
   ```

2. 使用两个参数`keySKU`和定义相应的reduce函数 `countObjVals`：

   - `countObjVals`是一个数组，其元素是映射到`keySKU`由map函数传递给reducer函数的分组值的对象。
   - 该函数将`countObjVals`数组简化为`reducedValue`包含`count`和 `qty`字段的单个对象。
   - 在中`reducedVal`，该`count`字段包含 `count`各个数组元素的`qty`字段总和，而该字段包含各个数组元素的 字段总和`qty`。

   ```powershell
   var reduceFunction2 = function(keySKU, countObjVals) {
      reducedVal = { count: 0, qty: 0 };
   
      for (var idx = 0; idx < countObjVals.length; idx++) {
          reducedVal.count += countObjVals[idx].count;
          reducedVal.qty += countObjVals[idx].qty;
      }
   
      return reducedVal;
   };
   ```

3. 定义有两个参数的函数确定`key`和 `reducedVal`。该函数修改`reducedVal`对象以添加一个名为`avg`的计算字段，并返回修改后的对象：

   ```powershell
   var finalizeFunction2 = function (key, reducedVal) {
     reducedVal.avg = reducedVal.qty/reducedVal.count;
     return reducedVal;
   };
   ```

4. 在执行的map-reduce操作`orders`使用集合`mapFunction2`，`reduceFunction2`和 `finalizeFunction2`功能。

   ```powershell
   db.orders.mapReduce(
      mapFunction2,
      reduceFunction2,
      {
        out: { merge: "map_reduce_example2" },
        query: { ord_date: { $gte: new Date("2020-03-01") } },
        finalize: finalizeFunction2
      }
    );
   ```

   此操作使用该`query`字段选择仅`ord_date`大于或等于的那些文档。然后将结果输出到集合 。`new Date("2020-03-01")` `map_reduce_example2`

   如果`map_reduce_example2`集合已经存在，则该操作会将现有内容与此map-reduce操作的结果合并。也就是说，如果现有文档具有与新结果相同的密钥，则该操作将覆盖现有文档。如果不存在具有相同密钥的文档，则该操作将插入该文档。

5. 查询`map_reduce_example2`集合以验证结果：

   ```powershell
   db.map_reduce_example2.find().sort( { _id: 1 } )
   ```

   该操作返回以下文档：

   ```powershell
   { "_id" : "apples", "value" : { "count" : 3, "qty" : 30, "avg" : 10 } }
   { "_id" : "carrots", "value" : { "count" : 2, "qty" : 15, "avg" : 7.5 } }
   { "_id" : "chocolates", "value" : { "count" : 3, "qty" : 15, "avg" : 5 } }
   { "_id" : "oranges", "value" : { "count" : 6, "qty" : 58, "avg" : 9.666666666666666 } }
   { "_id" : "pears", "value" : { "count" : 1, "qty" : 10, "avg" : 10 } }
   ```

## 聚合替代

   使用可用的聚合管道运算符，您可以重写map-reduce操作，而无需定义自定义函数：

   ```powershell
   db.orders.aggregate( [
      { $match: { ord_date: { $gte: new Date("2020-03-01") } } },
      { $unwind: "$items" },
      { $group: { _id: "$items.sku", qty: { $sum: "$items.qty" }, orders_ids: { $addToSet: "$_id" } }  },
      { $project: { value: { count: { $size: "$orders_ids" }, qty: "$qty", avg: { $divide: [ "$qty", { $size: "$orders_ids" } ] } } } },
      { $merge: { into: "agg_alternative_3", on: "_id", whenMatched: "replace",  whenNotMatched: "insert" } }
   ] )
   ```

1. 该[`$match`]()阶段仅选择`ord_date`大于或等于`new Date("2020-03-01")`的那些文档。

2. 该`$unwinds`阶段按`items`数组字段细分文档，以输出每个数组元素的文档。例如：

      ```powershell
      { "_id" : 1, "cust_id" : "Ant O. Knee", "ord_date" : ISODate("2020-03-01T00:00:00Z"), "price" : 25, "items" : { "sku" : "oranges", "qty" : 5, "price" : 2.5 }, "status" : "A" }
      { "_id" : 1, "cust_id" : "Ant O. Knee", "ord_date" : ISODate("2020-03-01T00:00:00Z"), "price" : 25, "items" : { "sku" : "apples", "qty" : 5, "price" : 2.5 }, "status" : "A" }
      { "_id" : 2, "cust_id" : "Ant O. Knee", "ord_date" : ISODate("2020-03-08T00:00:00Z"), "price" : 70, "items" : { "sku" : "oranges", "qty" : 8, "price" : 2.5 }, "status" : "A" }
      { "_id" : 2, "cust_id" : "Ant O. Knee", "ord_date" : ISODate("2020-03-08T00:00:00Z"), "price" : 70, "items" : { "sku" : "chocolates", "qty" : 5, "price" : 10 }, "status" : "A" }
      { "_id" : 3, "cust_id" : "Busby Bee", "ord_date" : ISODate("2020-03-08T00:00:00Z"), "price" : 50, "items" : { "sku" : "oranges", "qty" : 10, "price" : 2.5 }, "status" : "A" }
      { "_id" : 3, "cust_id" : "Busby Bee", "ord_date" : ISODate("2020-03-08T00:00:00Z"), "price" : 50, "items" : { "sku" : "pears", "qty" : 10, "price" : 2.5 }, "status" : "A" }
      { "_id" : 4, "cust_id" : "Busby Bee", "ord_date" : ISODate("2020-03-18T00:00:00Z"), "price" : 25, "items" : { "sku" : "oranges", "qty" : 10, "price" : 2.5 }, "status" : "A" }
      { "_id" : 5, "cust_id" : "Busby Bee", "ord_date" : ISODate("2020-03-19T00:00:00Z"), "price" : 50, "items" : { "sku" : "chocolates", "qty" : 5, "price" : 10 }, "status" : "A" }
      ...
      ```
      
3. [`$group`]()由平台组`items.sku`，计算每个SKU：

    - 该`qty`字段。该`qty`字段包含`qty`每个订单的总数`items.sku`（请参阅参考资料`$sum`）。
    - `orders_ids`列表。该`orders_ids`字段包含不同顺序的列表`_id`的对`items.sku`（参见 `$addToSet`）。

      ```powershell
      { "_id" : "chocolates", "qty" : 15, "orders_ids" : [ 2, 5, 8 ] }
      { "_id" : "oranges", "qty" : 63, "orders_ids" : [ 4, 7, 3, 2, 9, 1, 10 ] }
      { "_id" : "carrots", "qty" : 15, "orders_ids" : [ 6, 9 ] }
      { "_id" : "apples", "qty" : 35, "orders_ids" : [ 9, 8, 1, 6 ] }
      { "_id" : "pears", "qty" : 10, "orders_ids" : [ 3 ] }
      ```

4. 该[`$project`]()阶段调整输出文档的形状以反映map-reduce的输出，该输出具有两个字段`_id`和 `value`。该[`$project`]()设置：

    - `value.count`到的尺寸`orders_ids`数组。（请参阅[`$size`]()）
    - 在`value.qty`到`qty`输入文档的数量字段。
    - `value.avg`平均每笔订购的数量。（请参阅[`$divide`]()和[`$size`]()）

      ```powershell
      { "_id" : "apples", "value" : { "count" : 4, "qty" : 35, "avg" : 8.75 } }
      { "_id" : "pears", "value" : { "count" : 1, "qty" : 10, "avg" : 10 } }
      { "_id" : "chocolates", "value" : { "count" : 3, "qty" : 15, "avg" : 5 } }
      { "_id" : "oranges", "value" : { "count" : 7, "qty" : 63, "avg" : 9 } }
      { "_id" : "carrots", "value" : { "count" : 2, "qty" : 15, "avg" : 7.5 } }
      ```

5. 最后，[`$merge`]()将输出写入collection `agg_alternative_3`。如果现有文档的密钥`_id`与新结果相同，则该操作将覆盖现有文档。如果不存在具有相同密钥的文档，则该操作将插入该文档。

6. 查询`agg_alternative_3`集合以验证结果：

    ```powershell
    db.agg_alternative_3.find().sort( { _id: 1 } )
    ```

    该操作返回以下文档：

    ```powershell
    { "_id" : "apples", "value" : { "count" : 4, "qty" : 35, "avg" : 8.75 } }
    { "_id" : "carrots", "value" : { "count" : 2, "qty" : 15, "avg" : 7.5 } }
    { "_id" : "chocolates", "value" : { "count" : 3, "qty" : 15, "avg" : 5 } }
    { "_id" : "oranges", "value" : { "count" : 7, "qty" : 63, "avg" : 9 } }
    { "_id" : "pears", "value" : { "count" : 1, "qty" : 10, "avg" : 10 } }
    ```

    

译者：李冠飞

校对：