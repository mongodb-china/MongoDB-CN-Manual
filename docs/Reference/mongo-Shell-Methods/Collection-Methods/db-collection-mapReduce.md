

# [ ](#)db.collection.mapReduce（）

[]()

在本页面

*   [map功能要求](#requirements-for-the-map-function)

*   [reduce功能要求](#requirements-for-the-reduce-function)

*   [选项](#out-options)

*   [finalize功能要求](#requirements-for-the-finalize-function)

*   [Map-Reduce 例子](#map-reduce-examples)

*   [输出](#output)

*   [附加信息](#additional-information)

`db.collection. mapReduce`( map，reduce，{&lt;out&gt;，&lt;query&gt;，&lt;sort&gt;，&lt;limit&gt;，&lt;finalize&gt;，&lt;scope&gt;，&lt;jsMode&gt;，&lt;verbose&gt;})

> **注意**
>
> 从4.2版开始，MongoDB弃用：
>
> - 地图-reduce选项来*创建*一个新的分片集合以及使用的分片供选择的map-reduce。要输出到分片集合，请首先创建分片集合。MongoDB 4.2还不建议替换现有分片集合。
> - nonAtomic：false选项的显式规范。

db.collection.mapReduce()方法为MapReduce命令提供了包装。

> **注意**
>
> 视图不支持 map-reduce 操作。

db.collection.mapReduce()具有以下语法：

```powershell
db.collection.mapReduce(
    <map>,
    <reduce>,
    {
        out: <collection>,
        query: <document>,
        sort: <document>,
        limit: <number>,
        finalize: <function>,
        scope: <document>,
        jsMode: <boolean>,
        verbose: <boolean>,
        bypassDocumentValidation: <boolean>
    }
)
```

db.collection.mapReduce()采用以下参数：

| 参数                       | 类型     | 描述                                                         |
| -------------------------- | -------- | ------------------------------------------------------------ |
| `map`                      | function | 一个 JavaScript function 将与`key`关联或“maps”并发出`key`和 value `pair`。 <br/>有关详细信息，请参阅map Function 的要求。 |
| `reduce`                   | function | 一个 JavaScript function，它“减少”到一个 object 所有与特定`key`关联的`values`。 <br/>有关详细信息，请参阅reduce Function 的要求。 |
| `options`                  | document | 为db.collection.mapReduce()指定其他参数的文档。              |
| `bypassDocumentValidation` | boolean  | 可选的。允许MapReduce在操作期间绕过文档验证。这使您可以插入不符合验证要求的文档。 <br/> version 3.2 中的新内容。 |

下表描述了db.collection.mapReduce()可以接受的其他参数。

| 领域        | 类型               | 描述                                                         |
| ----------- | ------------------ | ------------------------------------------------------------ |
| `out`       | string or document | 指定 map-reduce 操作结果的位置。您可以输出到集合，输出到具有操作的集合，或输出内联。在对集合的主要成员执行 map-reduce 操作时，您可以输出到集合;在次要成员上，您只能使用`inline`输出。 <br/>有关详细信息，请参阅选项。 |
| `query`     | document           | 使用query operators指定选择条件，以确定输入到`map` function 的文档。 |
| `sort`      | document           | 对输入文档进行排序。此选项对优化很有用。对于 example，请将 sort key 指定为与 emit key 相同，以便减少 reduce 操作。 sort key 必须位于此集合的现有索引中。 |
| `limit`     | number             | 指定输入`map` function 的最大文档数。                        |
| `finalize`  | function           | 可选的。遵循`reduce`方法并修改输出。 <br/>有关详细信息，请参阅finalize Function 的要求。 |
| `scope`     | document           | 指定`map`，`reduce`和`finalize`函数中可访问的 global 变量。  |
| `jsMode`    | boolean            | 指定是否在执行`map`和`reduce`函数之间将中间数据转换为 BSON 格式。 <br/>默认为`false`。 <br/>如果`false`：<br/>1. 在内部，MongoDB 将`map` function 发出的 JavaScript objects 转换为 BSON objects。然后在调用`reduce` function 时将这些 BSON objects 转换回 JavaScript objects。 <br/> 2. map-reduce 操作将中间 BSON object 放置在临时的 on-disk 存储中。这允许 map-reduce 操作在任意大的数据集上执行。 <br/>如果`true`：<br/>1. 在内部，`map` function 期间发出的 JavaScript objects 仍然是 JavaScript objects。无需为`reduce` function 转换 objects，这可以加快执行速度。 <br/>2. 您只能将`jsMode`用于映射器`emit()` function 中少于 500,000 个不同`key` arguments 的结果_set。 |
| `verbose`   | boolean            | 指定是否在结果信息中包含`timing`信息。将`verbose`设置为`true`以包含`timing`信息。 <br/>默认为`false`。 |
| `collation` | document           | 可选的。 <br/>指定要用于操作的排序规则。 <br/> 整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>排序规则选项具有以下语法：<br/>排序规则：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/>如果未指定排序规则但集合具有默认排序规则(请参阅db.createCollection())，则操作将使用为集合指定的排序规则。 <br/>如果没有为集合或操作指定排序规则，MongoDB 使用先前版本中用于 string 比较的简单二进制比较。 <br/>您无法为操作指定多个排序规则。对于 example，您不能为每个字段指定不同的排序规则，或者如果使用排序执行查找，则不能对查找使用一个排序规则，而对排序使用另一个排序规则。 <br/> version 3.4 中的新内容。 |

> **注意**
>
> map-reduce operations， group 命令和$where 运算表达式无法访问 mongo shell 中可用的某些 global 函数或 properties，例如 db。
> 可用的 PropertiesAvailable 函数

以下JavaScript函数和属性**可**用于 和 运算符表达式：`map-reduce operations`、`$where`

| 可用属性                           | 可用功能                                                     |
| :--------------------------------- | ------------------------------------------------------------ |
| `args`<br />`MaxKey`<br />`MinKey` | `assert()`<br/>`BinData()`<br/>`DBPointer()`<br/>`DBRef()`<br/>`doassert()`<br/>`emit()`<br/>`gc()`<br/>`HexData()`<br/>`hex_md5()`<br/>`isNumber()`<br/>`isObject()`<br/>`ISODate()`<br/>`isString()`<br/>`Map()`<br/>`MD5()`<br/>`NumberInt()`<br/>`NumberLong()`<br/>`ObjectId()`<br/>`print()`<br/>`printjson()`<br/>`printjsononeline()`<br/>`sleep()`<br/>`Timestamp()`<br/>`tojson()`<br/>`tojsononeline()`<br/>`tojsonObject()`<br/>`UUID()`<br/>`version()` |

## <span id="requirements-for-the-map-function">map功能要求</span>

`map` function 负责将每个输入文档转换为零个或多个文档。它可以访问`scope`参数中定义的变量，并具有以下原型：

```powershell
function() {
    ...
    emit(key, value);
}
```

`map` function 具有以下要求：

*   在`map` function 中，在 function 中将当前文档作为`this`引用。
*   `map` function 不应出于任何原因访问数据库。
*   `map` function 应该是纯的，或者在 function 之外没有影响(即：side effects.)
*   单个发射只能容纳 MongoDB 的最大 BSON 文件大小的一半。
*   `map` function 可以选择多次调用`emit(key,value)`来创建一个将`key`与`value`相关联的输出文档。
*   在MongoDB 4.2和更早版本中，单个发射只能容纳MongoDB 最大BSON文档大小的一半。从版本4.4开始，MongoDB删除了此限制。
*   从MongoDB 4.4开始，它的功能`mapReduce`不再支持范围（即BSON类型15）的已弃用JavaScript 。该`map` 函数必须是BSON类型的String（即BSON类型2）或BSON类型的JavaScript（即BSON类型13）。要确定变量的范围，请使用 `scope`参数。

`map`自版本4.2.1起，该功能不建议在范围内使用JavaScript 

以下`map` function 将调用`emit(key,value)` 0 或 1 次，具体取决于输入文档的`status`字段的 value：

```powershell
function() {
    if (this.status == 'A')
    emit(this.cust_id, 1);
}
```

以下`map` function 可能会多次调用`emit(key,value)`，具体取决于输入文档的`items`字段中的元素数：

```powershell
function() {
    this.items.forEach(function(item){ emit(item.sku, 1); });
}
```

## <span id="requirements-for-the-reduce-function">reduce功能要求</span>

`reduce` function 具有以下原型：

```powershell
function(key, values) {
    ...
    return result;
}
```

`reduce` function 表现出以下行为：

* `reduce` function 不应该访问数据库，甚至不应该执行读操作。
* `reduce` function 不应影响外部系统。
* MongoDB 不会为只有一个 value 的 key 调用`reduce` function。 `values`参数是一个 array，其元素是`value` objects，它们被“映射”到`key`。
* MongoDB 可以为同一个 key 多次调用`reduce` function。在这种情况下，该 key 的`reduce` function 的前一个输出将成为该 key 的下一个`reduce` function 调用的输入值之一。
* `reduce` function 可以访问`scope`参数中定义的变量。
*   `reduce`的输入不得大于 MongoDB 的最大 BSON 文件大小的一半。返回大型文档然后在后续的`reduce`步骤中将其连接在一起时，可能会违反此要求。
*   从版本4.2.1开始，MongoDB在该功能的作用域（即BSON类型15）中弃用JavaScript `reduce`。要确定变量的范围，请改用`scope` 参数。

因为可以为同一个 key 多次调用`reduce` function，所以以下 properties 需要 true：

* return object 的类型必须与`map` function 发出的`value`的类型相同。

* `reduce` function 必须是关联的。以下语句必须是 true：
    ```powershell
    reduce(key, [ C, reduce(key, [ A, B ]) ] ) == reduce( key, [ C, A, B ] )
    ```
    
* `reduce` function 必须是幂等的。确保以下语句是 true：

    ```powershell
    reduce( key, [ reduce(key, valuesArray) ] ) == 	reduce( key, valuesArray )
    ```

* `reduce` function 应该是可交换的：也就是说，`valuesArray`中元素的 order 不应该影响`reduce` function 的输出，因此以下语句是 true：

  ```powershell
  reduce( key, [ A, B ] ) == reduce( key, [ B, A ] )
  ```

## <span id="out-options">选项</span>

您可以为`out`参数指定以下选项：

### 输出到集合

此选项输出到新集合，并且在副本集的辅助成员上不可用。

```powershell
out: <collectionName>
```

### 输出到带有 Action 的 Collection

> **注意**
>
> 从4.2版开始，MongoDB弃用：
>
> - 地图-reduce选项来*创建*一个新的分片集合以及使用的分片供选择的map-reduce。要输出到分片集合，请首先创建分片集合。MongoDB 4.2还不建议替换现有分片集合。
> - nonAtomic：false选项的显式规范。

此选项仅在将已存在的集合传递给`out`时可用。它不适用于副本集 的辅助成员。

```powershell
out: { <action>: <collectionName>
    [, db: <dbName>]
    [, sharded: <boolean> ]
    [, nonAtomic: <boolean> ] }
```

当您输出带有操作的集合时，`out`具有以下参数：

* `<action>`：指定以下操作之一：
  * `replace`

    如果具有`<collectionName>`的集合存在，则替换`<collectionName>`的内容。

  * `merge`

    如果输出集合已存在，则将新结果与现有结果合并。如果现有文档与新结果具有相同的 key，则覆盖该现有文档。

  * `reduce`

    如果输出集合已存在，则将新结果与现有结果合并。如果现有文档与新结果具有相同的 key，则将`reduce` function 应用于新文档和现有文档，并使用结果覆盖现有文档。

*   `db` :

    可选的。您希望 map-reduce 操作写入其输出的数据库的 name。默认情况下，这将是与输入集合相同的数据库。

*   `sharded` :

    可选的。如果`true`并且您已在输出数据库上启用了分片，则 map-reduce 操作将使用`_id`字段分割输出集合作为分片 key。

    如果`true`和`collectionName`是现有的未整数集合，map-reduce 将失败。

*   `nonAtomic` :

    > **注意**
    >
> 开始在MongoDB中4.2，明确设置`nonAtomic`到`false`已被弃用。

    可选的。将输出操作指定为 non-atomic。这仅对**`merge`和`reduce`输出模式应用**，这可能需要几分钟才能执行。
    
    默认情况下`nonAtomic`是`false`，map-reduce 操作在 post-processing 期间锁定数据库。
    
    如果`nonAtomic`是`true`，则 post-processing step 会阻止 MongoDB 锁定数据库：在此 time 期间，其他 clients 将能够读取输出集合的中间状态。

### 输出内联

在 memory 中执行 map-reduce 操作并 return 结果。此选项是副本集的辅助成员上`out`的唯一可用选项。

```powershell
out: { inline: 1 }
```

结果必须符合BSON 文档的最大大小。

## <span id="requirements-for-the-finalize-function">finalize功能要求</span>

`finalize` function 具有以下原型：

```powershell
function(key, reducedValue) {
    ...
    return modifiedObject;
}
```

`finalize` function 接收value 作为其 arguments 和`reduce` function 的`reducedValue`。意识到：

*   `finalize` function 不应出于任何原因访问数据库。
*   `finalize` function 应该是纯的，或者在 function 之外没有影响(即：side effects.)
*   `finalize` function 可以访问`scope`参数中定义的变量。
*   从版本4.2.1开始，MongoDB在该功能的作用域（即BSON类型15）中弃用JavaScript `finalize`。要确定变量的范围，请改用`scope` 参数。

## <span id="map-reduce-examples">Map-Reduce 例子</span>

> 聚合管道作为替代
> 
> 聚合管道比map-reduce提供更好的性能和更一致的接口。
> 
> 各种map-reduce表达式可以使用被重写聚合管道运算符，诸如`$group`， `$merge`等
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

### 返回每位客户的总价格

通过`cust_id`对`orders`集合执行 map-reduce 操作到 group，并为每个`cust_id`计算`price`的总和：

1. 定义map功能来处理每个输入文档：

* 在 function 中，`this`指的是 map-reduce 操作正在处理的文档。

* function maps 为每个文档的`cust_id`并发出`cust_id`和`price`键值对。
  ```powershell
  var mapFunction1 = function() {
      emit(this.cust_id, this.price);
  };
  ```

2. 使用两个参数 `keyCustId`和`valuesPrices`定义相应的 reduce function：

* `valuesPrices`是一个数组，其元素是 map function 发出的`price`值，并按`keyCustId`分组。

*   function 将`valuesPrice` array 缩减为其元素的总和。
	```powershell
    var reduceFunction1 = function(keyCustId, valuesPrices) {
        return Array.sum(valuesPrices);
    };
	```
    
3. 使用`mapFunction1` map function 和`reduceFunction1` reduce function 对`orders`集合中的所有文档执行 map-reduce。
    ```powershell
    db.orders.mapReduce(
        mapFunction1,
        reduceFunction1,
        { out: "map_reduce_example" }
    )
    ```
    
    此操作将结果输出到名为`map_reduce_example`的集合。如果`map_reduce_example`集合已存在，则操作将使用此 map-reduce 操作的结果替换内容。
    
4. 查询`map_reduce_example`集合以验证结果：

    ```powershell
    db.map_reduce_example.find().sort( { _id: 1 } )
    ```

    该操作返回以下文档：

    ```powershell
    { "_id" : "Ant O. Knee", "value" : 95 }
    { "_id" : "Busby Bee", "value" : 125 }
    { "_id" : "Cam Elot", "value" : 60 }
    { "_id" : "Don Quis", "value" : 155 }
    ```

#### 聚合替代

使用可用的聚合管道运算符，您可以重写map-reduce操作，而无需定义自定义函数：

```powershell
db.orders.aggregate([
   { $group: { _id: "$cust_id", value: { $sum: "$price" } } },
   { $out: "agg_alternative_1" }
])
```

1. `$group`由平台组`cust_id`并计算`value`字段（参见`$sum`）。该 `value`字段包含`price`每个的总计`cust_id`。

   该阶段将以下文档输出到下一阶段：

   ```powershell
   { "_id" : "Don Quis", "value" : 155 }
   { "_id" : "Ant O. Knee", "value" : 95 }
   { "_id" : "Cam Elot", "value" : 60 }
   { "_id" : "Busby Bee", "value" : 125 }
   ```

2. 然后，`$out`将输出写入collection `agg_alternative_1`。或者，您可以使用 `$merge`代替`$out`。

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

### 使用 Item 的平均数量计算 Order 和总数量

在此事例中，您将对`orders`集合执行 map-reduce 操作，以处理`ord_date` value 大于`01/01/2012`的所有文档。操作按`item.sku`字段分组，并计算每个`sku`的订单数量和订购总数量。然后，该操作将为每个值计算每个订单的平均数量，并将结果合并到输出集合中。合并结果时，如果现有文档的密钥与新结果相同，则该操作将覆盖现有文档。如果不存在具有相同密钥的文档，则该操作将插入该文档。

1. 定义map功能来处理每个输入文档：

* 在 function 中，`this`指的是 map-reduce 操作正在处理的文档。

* 对于每个 item，函数将`sku`与一个新的 object `value`相关联，该对象 `value`包含订单的`count`和_ite用于 order 并发出`sku`和`value`对。

  ```powershell
  var mapFunction2 = function() {
      for (var idx = 0; idx < this.items.length; idx++) {
          var key = this.items[idx].sku;
          var value = {
              count: 1,
              qty: this.items[idx].qty
          };
          emit(key, value);
      }
  };
  ```

2. 使用两个 arguments `keySKU`和`countObjVals`定义相应的 reduce function：

* `countObjVals`是一个 array，其元素是映射到 map function 传递给 reducer function 的分组`keySKU`值的 objects。

* function 将`countObjVals` array 缩减为包含`count`和`qty`字段的单个 object `reducedValue`。

*   在`reducedVal`中，`count`字段包含来自各个 array 元素的`count`字段的总和，`qty`字段包含来自各个 array 元素的`qty`字段的总和。
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

3. 使用两个 arguments `key`和`reducedVal`定义 finalize function。 function 修改`reducedVal` object 以添加名为`avg`的计算字段并返回修改后的 object：
   
    ```powershell
    var finalizeFunction2 = function (key, reducedVal) {
        reducedVal.avg = reducedVal.qty/reducedVal.count;
    
        return reducedVal;
    
    };
    ```
    
4. 使用`mapFunction2`，`reduceFunction2`和`finalizeFunction2`函数对`orders`集合执行 map-reduce 操作。

   ```powershell
   db.orders.mapReduce( mapFunction2,
       reduceFunction2,
       {
           out: { merge: "map_reduce_example" },
           query: { ord_date:
               { $gt: new Date('01/01/2012') }
           },
           finalize: finalizeFunction2
       }
   )
   ```

   此操作使用`query`字段仅选择`ord_date`大于`new Date(01/01/2012)`的文档。然后它将结果输出到集合`map_reduce_example`。如果`map_reduce_example`集合已存在，则操作将现有内容与此 map-reduce 操作的结果合并。也就是说，如果现有文档具有与新结果相同的密钥，则该操作将覆盖现有文档。如果不存在具有相同密钥的文档，则该操作将插入该文档。

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

#### 聚合替代

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

1. 该`$match`阶段仅选择`ord_date`大于或等于的那些文档。`new Date("2020-03-01")`

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

3. `$group`由平台组`items.sku`，计算每个SKU：

   * `qty`字段。该`qty`字段包含`qty`每个订单的总数`items.sku`（请参阅参考资料`$sum`）。

   - `orders_ids`阵列。该`orders_ids`字段包含不同顺序的阵列`_id`的对`items.sku`（参见 `$addToSet`）。

   ```powershell
   { "_id" : "chocolates", "qty" : 15, "orders_ids" : [ 2, 5, 8 ] }
   { "_id" : "oranges", "qty" : 63, "orders_ids" : [ 4, 7, 3, 2, 9, 1, 10 ] }
   { "_id" : "carrots", "qty" : 15, "orders_ids" : [ 6, 9 ] }
   { "_id" : "apples", "qty" : 35, "orders_ids" : [ 9, 8, 1, 6 ] }
   { "_id" : "pears", "qty" : 10, "orders_ids" : [ 3 ] }
   ```

4. `$project`阶段调整输出文档的形状以反映map-reduce的输出，该输出具有两个字段`_id`和 `value`。该`$project`sets：

   - `value.count`的尺寸在`orders_ids`数组中。（请参阅`$size`。）
   - `value.qty`在`qty`输入文档的字段。
   - `value.avg`每订购数量的平均数目。（请参阅`$divide`和`$size`。）

   ```powershell
   { "_id" : "apples", "value" : { "count" : 4, "qty" : 35, "avg" : 8.75 } }
   { "_id" : "pears", "value" : { "count" : 1, "qty" : 10, "avg" : 10 } }
   { "_id" : "chocolates", "value" : { "count" : 3, "qty" : 15, "avg" : 5 } }
   { "_id" : "oranges", "value" : { "count" : 7, "qty" : 63, "avg" : 9 } }
   { "_id" : "carrots", "value" : { "count" : 2, "qty" : 15, "avg" : 7.5 } }
   ```

5. 最后，`$merge`将输出写入collection `agg_alternative_3`。如果现有文档的密钥`_id`与新结果相同，则该操作将覆盖现有文档。如果不存在具有相同密钥的文档，则该操作将插入该文档。

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

## <span id="output">输出</span>

db.collection.mapReduce()方法的输出与MapReduce命令的输出相同。有关db.collection.mapReduce()输出的信息，请参阅MapReduce命令的产量部分。

## 限制

MongoDB驱动程序会自动将afterClusterTime设置为与因果一致的会话相关联的操作。从MongoDB 4.2开始， `db.collection.mapReduce()`不再支持 afterClusterTime。因此， `db.collection.mapReduce()`不能与因果一致的会话相关联 。

## <span id="additional-information">附加信息</span>

*   对 Map Function 进行故障排除

*   排除 Reduce Function 问题

*   MapReduce命令

*   聚合

*   Map-Reduce

*   执行增量 Map-Reduce



译者：李冠飞

校对：