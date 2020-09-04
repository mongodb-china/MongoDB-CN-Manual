# [ ](#)对 Map Function 进行故障排除

`map` function 是一个 JavaScript function，它将 value 与 key 关联或“maps”，并在[map-reduce]()操作期间发出 key 和 value 对。

要验证`map` function 发出的`key`和`value`对，请编写自己的`emit` function。

考虑一个包含以下原型文档的集合`orders`：

```powershell
{
     _id: ObjectId("50a8240b927d5d8b5891743c"),
     cust_id: "abc123",
     ord_date: new Date("Oct 04, 2012"),
     status: 'A',
     price: 250,
     items: [ { sku: "mmm", qty: 5, price: 2.5 },
              { sku: "nnn", qty: 5, price: 2.5 } ]
}
```

*   为每个文档定义_ma 功能
    ```powershell
    var map = function() {
        emit(this.cust_id, this.price);
    };
    ```
    
* 定义`emit` function 以打印 key 和 value：

    ```powershell
    var emit = function(key, value) {
        print("emit");
        print("key: " + key + "  value: " + tojson(value));
    }
    ```
    
* 使用`orders`集合中的单个文档调用`map` function：

    ```powershell
    var myDoc = db.orders.findOne( { _id: ObjectId("50a8240b927d5d8b5891743c") } );
    map.apply(myDoc);
    ```

* 验证 key 和 value 对是否符合预期。

    ```powershell
    emit
    key: abc123 value:250
    ```
    
* 使用`orders`集合中的多个文档调用`map` function：

    ```powershell
    var myCursor = db.orders.find( { cust_id: "abc123" } );
    while (myCursor.hasNext()) {
        var doc = myCursor.next();
        print ("document _id= " + tojson(doc._id));
        map.apply(doc);
        print();
    }
    ```

* 验证 key 和 value 对是否符合预期。

> **也可以看看**
>
> map` function 必须满足各种要求。有关`map` function 的所有要求的列表，请参阅[MapReduce]()或[mongo]() shell 辅助方法[db.collection.mapReduce()]()。



译者：李冠飞

校对：