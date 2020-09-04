# [ ](#)Map-Reduce转换到聚合管道

[]()

从4.4版本开始，MongoDB添加了`$accumulator`和`$function` aggregation运算符。这些运算符为用户提供了定义自定义聚合表达式的能力。使用这些操作，可以大致重写map-reduce表达式，如下表所示。

> **注意**
>
> 可以使用聚合管道操作符(如$group、$merge等)重写各种map-reduce表达式，而不需要自定义函数。
>
> 例如，请参见map-reduce示例。

## Map-Reduce到聚合管道转换表

这张表只是粗略的翻译。例如，该表显示了使用`$project`的`mapFunction`的近似转换。

* 然而，mapFunction逻辑可能需要额外的阶段，例如，如果逻辑包括对数组的迭代:

  ```powershell
  function() {
     this.items.forEach(function(item){ emit(item.sku, 1); });
  }
  ```

  然后，聚合管道包括一个`$unwind`和一个`$project`:

  ```powershell
  { $unwind: "$items "},
  { $project: { emits: { key: { "$items.sku" }, value: 1 } } },
  ```

* `$project`中的`emit`字段可以被命名为其他名称。为了进行可视化比较，选择了字段名称emit。

  
  
  | Map-Reduce                                                   | Aggregation Pipeline                                         |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | db.collection.mapReduce(<br /> &lt;mapFunction&gt;,<br /> &lt;reduceFunction&gt;,<br /> {<br /> query: &lt;queryFilter&gt;,<br /> sort: &lt;sortOrder&gt;,<br /> limit: &lt;number&gt;,<br /> finalize: &lt;finalizeFunction&gt;,<br /> out: &lt;collection&gt;<br /> } ) | db.collection.aggregate( [<br /> { $match: &lt;queryFilter&gt; },<br /> { $sort: &lt;sortOrder&gt; },<br /> { $limit: &lt;number&gt; },<br /> { $project: {  emits: { k: &lt;expression&gt;, v: &lt;expression&gt; } } },<br /> { $unwind: “$emits” },<br /> { $group: { <br />  _id: “$emits.k”}, <br />  value: { $accumulator: {<br />   init: &lt;initCode&gt;,<br />   accumulate: &lt;reduceFunction&gt;,<br />   accumulateArgs: [ “$emit.v”],<br />   merge: &lt;reduceFunction&gt;,<br />   finalize: &lt;finalizeFunction&gt;,<br />   lang: “js” }}<br /> } },<br /> { $out: &lt;collection&gt; }<br />] ) |
  | db.collection.mapReduce(<br/> &lt;mapFunction&gt;,<br/> &lt;reduceFunction&gt;,<br/> {<br/>  query: &lt;queryFilter&gt;,<br/>  sort: &lt;sortOrder&gt;,<br/>  limit: &lt;number&gt;,<br/>  finalize: &lt;finalizeFunction&gt;,<br/>  out: { merge: &lt;collection&gt;, db: &lt;db&gt; }<br/> }<br/>) | db.collection.aggregate( [<br/> { $match: &lt;queryFilter&gt; },<br/> { $sort: &lt;sortOrder&gt; },<br/> { $limit: &lt;number&gt; },<br/> { $project: { emits: { k: &lt;expression&gt;, v: &lt;expression&gt; } } },<br/> { $unwind: “$emits” },<br/> { $group: {<br/>  \_id: “$emits.k”},<br/>  value: { $accumulator: {<br/>   init: &lt;initCode&gt;,<br/>   accumulate: &lt;reduceFunction&gt;,<br/>   accumulateArgs: [ “$emit.v”],<br/>   merge: &lt;reduceFunction&gt;,<br/>   finalize: &lt;finalizeFunction&gt;,<br/>   lang: “js” }}<br/> } },<br/> { $out: { db: &lt;db&gt;, coll: &lt;collection&gt; } }<br/>] ) |
  | db.collection.mapReduce(<br/> &lt;mapFunction&gt;,<br/> &lt;reduceFunction&gt;,<br/> {<br/>  query: &lt;queryFilter&gt;,<br/>  sort: &lt;sortOrder&gt;,<br/>  limit: &lt;number&gt;,<br/>  finalize: &lt;finalizeFunction&gt;,<br/>  out: { merge: &lt;collection&gt;, db: &lt;db&gt; }<br/> }<br/>) | db.collection.aggregate( [<br/> { $match: &lt;queryFilter&gt; },<br/> { $sort: &lt;sortOrder&gt; },<br/> { $limit: &lt;number&gt; },<br/> { $project: { emits: { k: &lt;expression&gt;, v: &lt;expression&gt; } } },<br/> { $unwind: “$emits” },<br/> { $group: {<br/>  \_id: “$emits.k”},<br/>  value: { $accumulator: {<br/>   init: &lt;initCode&gt;,<br/>   accumulate: &lt;reduceFunction&gt;,<br/>   accumulateArgs: [ “$emit.v”],<br/>   merge: &lt;reduceFunction&gt;,<br/>   finalize: &lt;finalizeFunction&gt;,<br/>   lang: “js” }}<br/> } },<br/> { $merge: {<br/>  into: { db: &lt;db&gt;, coll: &lt;collection&gt;},<br/>  on: “_id”<br/>  whenMatched: “replace”,<br/>  whenNotMatched: “insert”<br/> } },<br/>] ) |
  | db.collection.mapReduce(<br/> &lt;mapFunction&gt;,<br/> &lt;reduceFunction&gt;,<br/> {<br/>  query: &lt;queryFilter&gt;,<br/>  sort: &lt;sortOrder&gt;,<br/>  limit: &lt;number&gt;,<br/>  finalize: &lt;finalizeFunction&gt;,<br/>  out: { merge: &lt;collection&gt;, db: &lt;db&gt; }<br/> }<br/>) | db.collection.aggregate( [<br/> { $match: &lt;queryFilter&gt; },<br/> { $sort: &lt;sortOrder&gt; },<br/> { $limit: &lt;number&gt; },<br/> { $project: { emits: { k: &lt;expression&gt;, v: &lt;expression&gt; } } },<br/> { $unwind: “$emits” },<br/> { $group: {<br/>  \_id: “$emits.k”},<br/>  value: { $accumulator: {<br/>   init: &lt;initCode&gt;,<br/>   accumulate: &lt;reduceFunction&gt;,<br/>   accumulateArgs: [ “$emit.v”],<br/>   merge: &lt;reduceFunction&gt;,<br/>   finalize: &lt;finalizeFunction&gt;,<br/>   lang: “js” }}<br/> } },<br/> { $merge: {<br/>  into: { db: &lt;db&gt;, coll: &lt;collection&gt; },<br/>  on: “\_id”<br/>  whenMatched: [<br/>   { $project: {<br/>    value: { $function: {<br/>     body: &lt;reduceFunction&gt;,<br/>     args: [<br/>      “$_id”,<br/>      [ “$value”, “$$new.value” ]<br/>     ],<br/>     lang: “js”<br/>    } }<br/>   } }<br/>  ]<br/>  whenNotMatched: “insert”<br/> } },<br/>] ) |
  | db.collection.mapReduce(<br/> &lt;mapFunction&gt;,<br/> &lt;reduceFunction&gt;,<br/> {<br/>  query: &lt;queryFilter&gt;,<br/>  sort: &lt;sortOrder&gt;,<br/>  limit: &lt;number&gt;,<br/>  finalize: &lt;finalizeFunction&gt;,<br/>  out: { inline: 1 }<br/> }<br/>) | db.collection.aggregate( [<br/> { $match: &lt;queryFilter&gt; },<br/> { $sort: &lt;sortOrder&gt; },<br/> { $limit: &lt;number&gt; },<br/> { $project: { emits: { k: &lt;expression&gt;, v: &lt;expression&gt; } } },<br/> { $unwind: “$emits” },<br/> { $group: {<br/>  _id: “$emits.k”},<br/>  value: { $accumulator: {<br/>  init: &lt;initCode&gt;,<br/>  accumulate: &lt;reduceFunction&gt;,<br/>  accumulateArgs: [ “$emit.v”],<br/>  merge: &lt;reduceFunction&gt;,<br/>  finalize: &lt;finalizeFunction&gt;,<br/>  lang: “js” }}<br/> } }<br/>] ) |

## 例子

可以使用聚合管道操作符(如`$group`、`$merge`等)重写各种map-reduce表达式，而不需要自定义函数。但是，为了说明目的，下面的例子提供了两种选择。

### 示例1

通过`cust_id`对订单集合组执行以下`map-reduce`操作，并计算每个`cust_id`的价格总和:

```powershell
var mapFunction1 = function() {
   emit(this.cust_id, this.price);
};

var reduceFunction1 = function(keyCustId, valuesPrices) {
   return Array.sum(valuesPrices);
};

db.orders.mapReduce(
   mapFunction1,
   reduceFunction1,
   { out: "map_reduce_example" }
)
```

**备选方案1:(推荐)**您可以重写操作到聚合管道，而不将map-reduce函数转换为等效的管道阶段:

```powershell
db.orders.aggregate([
   { $group: { _id: "$cust_id", value: { $sum: "$price" } } },
   { $out: "agg_alternative_1" }
])
```

**备选方案2:(仅为说明目的)**下面的聚合管道提供了各种map-reduce函数的转换，使用`$accumulator`定义自定义函数:

```powershell
db.orders.aggregate( [
   { $project: { emit: { key: "$cust_id", value: "$price" } } },  // equivalent to the map function
   { $group: {                                                    // equivalent to the reduce function
        _id: "$emit.key",
        valuesPrices: { $accumulator: {
                    init: function() { return 0; },
                    initArgs: [],
                    accumulate: function(state, value) { return state + value; },
                    accumulateArgs: [ "$emit.value" ],
                    merge: function(state1, state2) { return state1 + state2; },
                    lang: "js"
        } }
   } },
   { $out: "agg_alternative_2" }
] )
```

1. 首先，`$project`阶段输出带有emit字段的文档。emit字段是一个包含以下字段的文档:

   - `key`包含`cust_id`文档的值
   - `value`包含`price`文档的值

   ```powershell
   { "_id" : 1, "emit" : { "key" : "Ant O. Knee", "value" : 25 } }
   { "_id" : 2, "emit" : { "key" : "Ant O. Knee", "value" : 70 } }
   { "_id" : 3, "emit" : { "key" : "Busby Bee", "value" : 50 } }
   { "_id" : 4, "emit" : { "key" : "Busby Bee", "value" : 25 } }
   { "_id" : 5, "emit" : { "key" : "Busby Bee", "value" : 50 } }
   { "_id" : 6, "emit" : { "key" : "Cam Elot", "value" : 35 } }
   { "_id" : 7, "emit" : { "key" : "Cam Elot", "value" : 25 } }
   { "_id" : 8, "emit" : { "key" : "Don Quis", "value" : 75 } }
   { "_id" : 9, "emit" : { "key" : "Don Quis", "value" : 55 } }
   { "_id" : 10, "emit" : { "key" : "Don Quis", "value" : 25 } }
   ```

2. 然后，`$group`使用`$accumulator`操作符来添加发出的值:

   ```powershell
   { "_id" : "Don Quis", "valuesPrices" : 155 }
   { "_id" : "Cam Elot", "valuesPrices" : 60 }
   { "_id" : "Ant O. Knee", "valuesPrices" : 95 }
   { "_id" : "Busby Bee", "valuesPrices" : 125 }
   ```

3. 最后，`$out`将输出写入集合`agg_alternative_2`。或者，您可以使用`$merge`而不是`$out`。

### 示例2

以下字段对`orders`集合组的map-reduce操作，`item.sku`并计算每个sku的订单数量和总订购量。然后，该操作将为每个sku值计算每个订单的平均数量，并将结果合并到输出集合中。

```powershell
var mapFunction2 = function() {
    for (var idx = 0; idx < this.items.length; idx++) {
       var key = this.items[idx].sku;
       var value = { count: 1, qty: this.items[idx].qty };

       emit(key, value);
    }
};

var reduceFunction2 = function(keySKU, countObjVals) {
   reducedVal = { count: 0, qty: 0 };

   for (var idx = 0; idx < countObjVals.length; idx++) {
       reducedVal.count += countObjVals[idx].count;
       reducedVal.qty += countObjVals[idx].qty;
   }

   return reducedVal;
};

var finalizeFunction2 = function (key, reducedVal) {
  reducedVal.avg = reducedVal.qty/reducedVal.count;
  return reducedVal;
};

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

**备选方案1:(推荐)**您可以重写操作到聚合管道，而不将map-reduce函数转换为等效的管道阶段:

```powershell
db.orders.aggregate( [
   { $match: { ord_date: { $gte: new Date("2020-03-01") } } },
   { $unwind: "$items" },
   { $group: { _id: "$items.sku", qty: { $sum: "$items.qty" }, orders_ids: { $addToSet: "$_id" } }  },
   { $project: { value: { count: { $size: "$orders_ids" }, qty: "$qty", avg: { $divide: [ "$qty", { $size: "$orders_ids" } ] } } } },
   { $merge: { into: "agg_alternative_3", on: "_id", whenMatched: "replace",  whenNotMatched: "insert" } }
] )
```

**备选方案2:(仅为说明目的)**下面的聚合管道提供了各种map-reduce函数的转换，使用`$accumulator`定义自定义函数:

```powershell
db.orders.aggregate( [
    { $match: { ord_date: {$gte: new Date("2020-03-01") } } },
    { $unwind: "$items" },
    { $project: { emit: { key: "$items.sku", value: { count: { $literal: 1 }, qty: "$items.qty" } } } },
    { $group: {
           _id: "$emit.key",
           value: { $accumulator: {
             init: function() { return { count: 0, qty: 0 }; },
             initArgs: [],
             accumulate: function(state, value) {
                  state.count += value.count;
                  state.qty += value.qty;
                  return state;
             },
             accumulateArgs: [ "$emit.value" ],
             merge: function(state1, state2) {
                return { count: state1.count + state2.count, qty: state1.qty + state2.qty };
             },
             finalize: function(state) {
                state.avg = state.qty / state.count;
                return state;
             },
             lang: "js"}
          }
    } },
    { $merge: {
       into: "agg_alternative_4",
       on: "_id",
       whenMatched: "replace",
       whenNotMatched: "insert"
    } }
] )
```

1. `$match`阶段只选择那些ord_date大于或等于new Date("2020-03-01")的文档。

2. `$unwinds`阶段按items数组字段分解文档，为每个数组元素输出一个文档。例如:

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

3. `$project`阶段输出带有emit字段的文档。emit字段是一个包含以下字段的文档:

   - `key`包含`items.sku`值
   - `value`包含具有`qty`值和`count`值的文档

   ```powershell
   { "_id" : 1, "emit" : { "key" : "oranges", "value" : { "count" : 1, "qty" : 5 } } }
   { "_id" : 1, "emit" : { "key" : "apples", "value" : { "count" : 1, "qty" : 5 } } }
   { "_id" : 2, "emit" : { "key" : "oranges", "value" : { "count" : 1, "qty" : 8 } } }
   { "_id" : 2, "emit" : { "key" : "chocolates", "value" : { "count" : 1, "qty" : 5 } } }
   { "_id" : 3, "emit" : { "key" : "oranges", "value" : { "count" : 1, "qty" : 10 } } }
   { "_id" : 3, "emit" : { "key" : "pears", "value" : { "count" : 1, "qty" : 10 } } }
   { "_id" : 4, "emit" : { "key" : "oranges", "value" : { "count" : 1, "qty" : 10 } } }
   { "_id" : 5, "emit" : { "key" : "chocolates", "value" : { "count" : 1, "qty" : 5 } } }
   ...
   ```

4. `$group`使用`$accumulator`操作符来添加发出的计数和数量，并计算avg字段:

   ```powershell
   { "_id" : "chocolates", "value" : { "count" : 3, "qty" : 15, "avg" : 5 } }
   { "_id" : "oranges", "value" : { "count" : 7, "qty" : 63, "avg" : 9 } }
   { "_id" : "carrots", "value" : { "count" : 2, "qty" : 15, "avg" : 7.5 } }
   { "_id" : "apples", "value" : { "count" : 4, "qty" : 35, "avg" : 8.75 } }
   { "_id" : "pears", "value" : { "count" : 1, "qty" : 10, "avg" : 10 } }
   ```

5. 最后，`$merge`将输出写入集合`agg_alternative_4`。如果现有文档具有与新结果相同的键_id，则操作将覆盖现有文档。如果没有具有相同密钥的现有文档，操作将插入该文档。

> **也可以看看**<br />[聚合命令比较]()



译者：李冠飞

校对：
