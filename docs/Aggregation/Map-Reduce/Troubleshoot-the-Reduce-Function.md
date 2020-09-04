# [ ](#)排除 Reduce Function 问题

[]()

在本页面

*   [确认输出类型](#confirm-output-type)

*   [确保对映射值的 Order 不敏感](#ensure-insensitivity-to-the-order-of-mapped-values)

*   [确保减少 Function Idempotence](#ensure-reduce-function-idempotence)

`reduce` function 是一个 JavaScript function，它在[map-reduce]()操作期间“减少”到单个 object 与特定 key 关联的所有值。 `reduce` function 必须满足各种要求。本教程有助于验证`reduce` function 是否符合以下条件：

*   `reduce` function 必须_retject 一个 object，其类型必须**与`map` function 发出的`value`的类型相同**。

*   `valuesArray`中元素的 order 不应影响`reduce` function 的输出。

*   `reduce` function 必须是幂等的。

有关`reduce` function 的所有要求的列表，请参阅[MapReduce]()或[mongo]() shell 辅助方法[db.collection.mapReduce()]()。

[]()

## <span id="confirm-output-type">确认输出类型</span>

您可以测试`reduce` function 返回的 value 与`map` function 发出的 value 的类型相同。

* 定义一个`reduceFunction1` function，它接受 arguments `keyCustId`和`valuesPrices`。 `valuesPrices`是整数的 array：

  ```powershell
  var reduceFunction1 = function(keyCustId, valuesPrices) {
      return Array.sum(valuesPrices);
  };
  ```

*   定义 sample array 整数：
    ```powershell
    var myTestValues = [ 5, 5, 10 ];
	```
    
*   使用`myTestValues`调用`reduceFunction1`：
    ```powershell
    reduceFunction1('myKey', myTestValues);
	```
    
*   验证`reduceFunction1`返回 integer：
    ```powershell
    20
	```
    
*   定义一个`reduceFunction2` function，它接受 arguments `keySKU`和`valuesCountObjects`。 `valuesCountObjects`是包含两个字段`count`和`qty`的 array 文档：
    ```powershell
    var reduceFunction2 = function(keySKU, valuesCountObjects) {
    reducedValue = { count: 0, qty: 0 };
        for (var idx = 0; idx <; valuesCountObjects.length; idx++) {
            reducedValue.count += valuesCountObjects[idx].count;
            reducedValue.qty += valuesCountObjects[idx].qty;
        }
    
        return reducedValue;
    };
    ```

*   定义 sample array 文档：
    ```powershell
    var myTestObjects = [
        { count: 1, qty: 5 },
        { count: 2, qty: 10 },
        { count: 3, qty: 15 }
    ];
	```
    
*   使用`myTestObjects`调用`reduceFunction2`：
    ```powershell
    reduceFunction2('myKey', myTestObjects);
	```
    
*   验证`reduceFunction2`返回的文档中包含`count`和`qty`字段：
    ```powershell
    { "count" : 6, "qty" : 30 }
	```
    

[]()
    
## <span id="ensure-insensitivity-to-the-order-of-mapped-values">确保对映射值的 Order 不敏感</span>

`reduce` function 以`key`和`values` array 为参数。您可以测试`reduce` function 的结果不依赖于`values` array 中元素的 order。
    
*   定义 sample `values1` array 和 sample `values2` array，它们只在 array 元素的 order 中有所不同：
    ```powershell
    var values1 = [
        { count: 1, qty: 5 },
        { count: 2, qty: 10 },
        { count: 3, qty: 15 }
];
    var values2 = [
        { count: 3, qty: 15 },
        { count: 1, qty: 5 },
        { count: 2, qty: 10 }
    ];
    ```

*   定义一个`reduceFunction2` function，它接受 arguments `keySKU`和`valuesCountObjects`。 `valuesCountObjects`是包含两个字段`count`和`qty`的 array 文档：
    ```powershell
    var reduceFunction2 = function(keySKU, valuesCountObjects) {
    reducedValue = { count: 0, qty: 0 };
        for (var idx = 0; idx < valuesCountObjects.length; idx++) {
            reducedValue.count += valuesCountObjects[idx].count;
            reducedValue.qty += valuesCountObjects[idx].qty;
        }
    
        return reducedValue;
    };
    ```

*   先使用`values1`然后使用`values2`调用`reduceFunction2`：
    ```powershell
    reduceFunction2('myKey', values1);
    reduceFunction2('myKey', values2);
	```
    
*   验证`reduceFunction2`返回相同的结果：
    ```powershell
    { "count" : 6, "qty" : 30 }
	```
    

[]()

## <span id="ensure-reduce-function-idempotence">确保减少 Function Idempotence</span>

因为 map-reduce 操作可能会为同一个 key 多次调用`reduce`，并且不会为工作集中的 key 的单个实例调用`reduce`，`reduce` function 必须 return 与从该值发出的 value 相同类型的 value。 `map` function。您可以测试`reduce` function process“减少”值而不影响最终的 value。
    
*   定义一个`reduceFunction2` function，它接受 arguments `keySKU`和`valuesCountObjects`。 `valuesCountObjects`是包含两个字段`count`和`qty`的 array 文档：
    ```powershell
    var reduceFunction2 = function(keySKU, valuesCountObjects) {
    reducedValue = { count: 0, qty: 0 };
        for (var idx = 0; idx <; valuesCountObjects.length; idx++) {
            reducedValue.count += valuesCountObjects[idx].count;
            reducedValue.qty += valuesCountObjects[idx].qty;
        }
    
        return reducedValue;
    };
    ```

*   定义 sample key：
    ```powershell
    var myKey = 'myKey';
	```
    
*   定义 sample `valuesIdempotent` array，其中包含一个调用`reduceFunction2` function 的元素：
    ```powershell
    var valuesIdempotent = [
        { count: 1, qty: 5 },
        { count: 2, qty: 10 },
        reduceFunction2(myKey, [ { count:3, qty: 15 } ] )
    ];
	```
    
*   定义一个 sample `values1` array，它结合了传递给`reduceFunction2`的值：
    ```powershell
    var values1 = [
        { count: 1, qty: 5 },
        { count: 2, qty: 10 },
        { count: 3, qty: 15 }
    ];
	```
    
*   首先使用`myKey`和`valuesIdempotent`调用`reduceFunction2`，然后使用`myKey`和`values1`调用`reduceFunction2`：
    ```powershell
    reduceFunction2(myKey, valuesIdempotent);
    reduceFunction2(myKey, values1);
	```
    
*   验证`reduceFunction2`返回相同的结果：
    
    ```powershell
    { "count" : 6, "qty" : 30 }
    ```



译者：李冠飞

校对：