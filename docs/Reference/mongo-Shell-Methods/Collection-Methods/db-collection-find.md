# [ ](#)db.collection.find（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `find`(查询，投影)

       *   选择集合或视图中的文档，并将光标返回到所选文档。

| 参数         | 类型     | 描述                                                         |
| ------------ | -------- | ------------------------------------------------------------ |
| `query`      | document | 可选的。使用query operators指定选择过滤器。要 return 集合中的所有文档，请省略此参数或传递空文档(`{}`)。 |
| `projection` | document | 可选的。指定_retch 查询过滤器的文档中的 return 字段。要_retret 匹配文档中的所有字段，请省略此参数。有关详细信息，请参阅投影。 |

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 与匹配`query`标准的文档。当find()方法“返回文档”时，该方法实际上是将光标返回到文档。 |

## <span id="behavior">行为</span>

### 投影

`projection`参数确定匹配文档中返回的字段。 `projection`参数采用以下形式的文档：

```powershell
{ field1: <value>, field2: <value> ... }
```

`<value>`可以是以下任何一种：

*   `1`或`true`在 return 文档中包含该字段。
*   `0`或`false`排除该字段。
* 表达式使用投影操作员。

  find()视图操作不支持以下投影 operators：

  *   $
  *   $elemMatch
  *   $slice
  *   $meta
> **注意**
>
> 对于`_id`字段，您不必明确指定`_id: 1`来\_return `_id`字段。除非指定`_id: 0`以禁止字段，否则find()方法始终返回_id字段。

除了排除`_id`字段外，`projection`不能同时包含 include 和 exclude 规范。在明确包含字段的投影中，`_id`字段是您可以显式排除的唯一字段。

### 游标处理

在mongo shell 中执行db.collection.find()会自动迭代光标以显示前 20 个文档。键入`it`以继续迭代。

要使用驱动程序访问返回的文档，请使用适当的司机语言光标处理机制。

> **也可以看看**
>
> - 迭代返回的游标
> - 修改光标行为
> - 可用的mongo Shell游标方法

### 阅读关注

要为db.collection.find()指定阅读关注，请使用cursor.readConcern()方法。

### 包装类型

MongoDB 将某些数据类型视为等效用于比较目的。例如，数字类型在比较之前进行转换。但是，对于大多数数据类型，比较 operators仅对目标字段的BSON 类型与查询操作数的类型匹配的文档执行比较。考虑以下集合：

```powershell
{ "_id": "apples", "qty": 5 }
{ "_id": "bananas", "qty": 7 }
{ "_id": "oranges", "qty": { "in stock": 8, "ordered": 12 } }
{ "_id": "avocados", "qty": "fourteen" }
```

以下查询使用$gt来 return `qty`的 value 大于`4`的文档。

```powershell
db.collection.find( { qty: { $gt: 4 } } )
```

该查询返回以下文档：

```powershell
{ "_id": "apples", "qty": 5 }
{ "_id": "bananas", "qty": 7 }
```

不返回`_id`等于`"avocados"`的文档，因为`qty` value 的类型为`string`而$gt操作数的类型为`integer`。

不返回`_id`等于`"oranges"`的文档，因为其`qty` value 的类型为`object`。

> **注意**
>
> 要在集合中强制执行数据类型，请使用Schema 验证。

### 会话

*版本4.0中的新功能。*

对于在会话内创建的游标，不能`getMore`在会话外调用 。

同样，对于在会话外部创建的游标，不能`getMore`在会话内部调用 。

#### 会话空闲超时

从MongoDB 3.6开始，MongoDB驱动程序和`mongo`shell程序将所有操作与服务器会话相关联，但未确认的写操作除外。对于未与会话明确关联的操作（即使用`Mongo.startSession()`），MongoDB驱动程序和`mongo`外壳程序会创建一个隐式会话并将其与该操作相关联。

如果会话空闲时间超过30分钟，则MongoDB服务器会将会话标记为已过期，并可以随时关闭它。当MongoDB服务器关闭会话时，它还会终止所有正在进行的操作并打开与该会话关联的游标。这包括配置了30分钟`noCursorTimeout`或`maxTimeMS`30分钟以上的光标。

对于可能闲置超过30分钟的操作，请使用将该操作与显式会话相关联， `Session.startSession()`并使用该`refreshSessions`命令定期刷新该会话。请参阅以获取更多信息。`Session Idle Timeout`

### 交易

`db.collection.find()`可以在多文档交易中使用。

- 对于在事务外部创建的游标，不能`getMore`在事务内部调用 。
- 对于在事务中创建的游标，不能`getMore`在事务外部调用 。

> **重要**
>
> 在大多数情况下，与单文档写入相比，多文档事务产生的性能成本更高，并且多文档事务的可用性不应替代有效的架构设计。在许多情况下， 非规范化数据模型（嵌入式文档和数组）将继续是您的数据和用例的最佳选择。也就是说，在许多情况下，适当地对数据建模将最大程度地减少对多文档交易的需求。
>
> 有关其他事务使用方面的注意事项（例如运行时限制和操作日志大小限制），另请参见 生产注意事项。

### 客户端断开

从MongoDB 4.2开始，如果发出`db.collection.find()` 断开连接的客户端在操作完成之前断开连接，则MongoDB将标记`db.collection.find()`为终止（即`killOp`在操作上）。

## <span id="examples">例子</span>

本节中的示例使用bios 集合中的文档，其中文档通常具有以下形式：

```powershell
{
    "_id" : <value>,
		"name" : { "first" : <string>, "last" : <string> },  // embedded document
    "birth" : <ISODate>,
    "death" : <ISODate>,
    "contribs" : [ <string>, ... ],  // Array of Strings
    "awards" : [
        { "award" : <string>, year: <number>, by: <string> }  // Array of embedded documents
        ...
    ]
}
```

要创建和填充`bios`集合，请参阅bios Example Collection。

### 查找集合中的所有文档

没有参数的find()方法返回集合中的所有文档，并返回文档的所有字段。对于 example，以下操作返回bios 系列中的所有文档：

```powershell
db.bios.find()
```

### 查找 Match 查询条件的文档

#### 查询平等

*   以下操作返回bios 系列中`_id`等于`5`的文档：
    ```powershell
    db.bios.find( { _id: 5 } )
    ```
    
*   以下操作返回bios 系列中的文档，其中`name`嵌入文档中的字段`last`等于`"Hopper"`：
	```powershell
   db.bios.find( { "name.last": "Hopper" } )
   ```
> **注意**
>
> 要访问嵌入文档中的字段，请使用点符号(`"<embedded document>.<field>"`)。

#### 使用 Operators 进行查询

要查找匹配一组选择条件的文档，请使用`<criteria>`参数调用find()。

MongoDB 提供各种query operators来指定标准。

*   以下操作使用$in operator return bios 系列中的文档，其中`_id`等于`5`或`ObjectId("507c35dd8fada716c89d0013")`：
    ```powershell
    db.bios.find(
       { _id: { $in: [ 5, ObjectId("507c35dd8fada716c89d0013") ] } }
    )
    ```
    
*   以下操作使用$gt operator 返回`bios`集合中`birth`大于`new Date('1950-01-01')`的所有文档：
	```powershell
    db.bios.find( { birth: { $gt: new Date('1950-01-01') } } )
	```
    
*   以下操作使用$regex operator return bios 系列中的文档，其中`name.last`字段以字母`N`(或`"LIKE N%"`)开头
	```powershell
    db.bios.find(
       { "name.last": { $regex: /^N/ } }
    )
    ```

有关查询 operators 的列表，请参阅查询 Selectors。

#### 查询范围

组合比较 operators 以指定字段的范围。以下操作从bios 系列文档返回，其中`birth`介于`new Date('1940-01-01')`和`new Date('1960-01-01')`之间(独占)：

```powershell
db.bios.find( { birth: { $gt: new Date('1940-01-01'), $lt: new Date('1960-01-01') } } )
```

有关查询 operators 的列表，请参阅查询 Selectors。

#### 查询多个条件

以下操作返回bios 系列中的所有文档，其中`birth`字段为比...更棒 `new Date('1950-01-01')`且`death`字段不存在：

```powershell
db.bios.find( {
    birth: { $gt: new Date('1920-01-01') },
    death: { $exists: false }
} )
```

有关查询 operators 的列表，请参阅查询 Selectors。

### 查询嵌入式文档

以下示例查询bios 系列中的`name`嵌入字段。

#### 查询嵌入式文档的精确匹配

以下操作返回bios 系列中的文档，其中嵌入的文档`name`正好是`{ first: "Yukihiro", last: "Matsumoto" }`，包括 order：

```powershell
db.bios.find(
    { name: { first: "Yukihiro", last: "Matsumoto" } }
)
```

`name`字段必须完全匹配嵌入文档。查询将使文档与具有`name`以下任一值的字段进行匹配：

```powershell
{
       first: "Yukihiro",
       aka: "Matz",
       last: "Matsumoto"
    }
{
   last: "Matsumoto",
   first: "Yukihiro"
}
```

#### 查询嵌入文档的字段

以下操作返回bios 系列中的文档，其中嵌入文档`name`包含带有 value `"Yukihiro"`的字段`first`和带有 value `"Matsumoto"`的字段`last`。该查询使用点符号访问嵌入文档中的字段：

```powershell
db.bios.find(
    {
        "name.first": "Yukihiro",
        "name.last": "Matsumoto"
    }
)
```

该查询匹配`name`字段包含嵌入文档的文档，其中`first`字段包含 value `"Yukihiro"`，字段`last`包含 value `"Matsumoto"`。例如，查询将使用包含以下任一值的`name`字段匹配文档：

```powershell
{
    first: "Yukihiro",
    aka: "Matz",
    last: "Matsumoto"
}
{
    last: "Matsumoto",
    first: "Yukihiro"
}
```

有关更多信息和示例，另请参阅查询 Embedded/Nested 文档。

### 查询数组

#### 查询 Array 元素

以下示例查询bios 系列中的`contribs` array。

*   以下操作返回bios 系列中的文档，其中 array 字段`contribs`包含元素`"UNIX"`：
    ```powershell
    db.bios.find( { contribs: "UNIX" } )
    ```
    
*   以下操作返回bios 系列中的文档，其中 array 字段`contribs`包含元素`"ALGOL"`或`"Lisp"`：
	```powershell
    db.bios.find( { contribs: { $in: [ "ALGOL", "Lisp" ]} } )
	```
    
*   以下操作使用$all query operator return bios 系列中的文档，其中 array 字段`contribs`包含元素`"ALGOL"`和`"Lisp"`：
	```powershell
    db.bios.find( { contribs: { $all: [ "ALGOL", "Lisp" ] } } )
    ```

有关更多示例，请参阅$all。另见$elemMatch。

*   以下操作使用$size operator return bios 系列中`contribs`的 array 大小为 4 的文档：
    ```powershell
    db.bios.find( { contribs: { $size: 4 } } )
    ```

有关查询 array 的更多信息和示例，请参阅：

*   查询 Array
*   查询嵌入式文档的 Array

有关 array 特定查询 operators 的列表，请参阅Array。

#### 查询 Array of Documents

以下示例查询bios 系列中的`awards` array。

*   以下操作返回bios 系列中的文档，其中`awards` array 包含`award`字段等于`"Turing`的元素：
    ```powershell
    db.bios.find(
        { "awards.award": "Turing Award" }
    )
    ```
    
*   以下操作返回bios 系列中的文档，其中`awards` array 包含至少一个元素，其中`award`字段等于`"Turing Award"`且`year`字段大于 1980：
	```powershell
    db.bios.find(
        { awards: { $elemMatch: { award: "Turing Award", year: { $gt: 1980 } } } }
    )
    ```

使用$elemMatch operator 在 array 元素上指定多个条件。

有关查询 array 的更多信息和示例，请参阅：

*   查询 Array
*   查询嵌入式文档的 Array

    有关 array 特定查询 operators 的列表，请参阅Array。

### 预测

`projection`参数指定 return 的哪些字段。除非排除属于`_id`字段，否则该参数包含 include 或 exclude 规范，而不是两者。

> **注意**
>
> 除非在投影文档`_id: 0`中明确排除`_id`字段，否则返回`_id`字段。

#### 指定 Return 的字段

以下操作查找bios 系列中的所有文档，并仅返回`name`字段，`contribs`字段和`_id`字段：

```powershell
db.bios.find( { }, { name: 1, contribs: 1 } )
```

> **注意**
>
> 除非在投影文档`_id: 0`中明确排除`_id`字段，否则返回`_id`字段。

#### 明确排除的字段

以下操作查询bios 系列并返回除`name`嵌入文档和`birth`字段中的`first`字段之外的所有字段：

```powershell
db.bios.find(
    { contribs: 'OOP' },
    { 'name.first': 0, birth: 0 }
)
```

#### 明确排除_id 字段

> **注意**
>
> 除非在投影文档`_id: 0`中明确排除`_id`字段，否则返回`_id`字段。

以下操作在bios 系列中查找文档，并仅返回`name`字段和`contribs`字段：

```powershell
db.bios.find(
    { },
    { name: 1, contribs: 1, _id: 0 }
)
```

#### 关于数组和嵌入式文档

以下操作查询bios 系列并返回`name`嵌入文档中的`last`字段和`contribs` array 中的前两个元素：

```powershell
db.bios.find(
    { },
    {
        _id: 0,
        'name.last': 1,
        contribs: { $slice: 2 }
    }
)
```

> **也可以看看**
>
> 从查询返回的项目字段

### 迭代返回的光标

find()方法返回光标到结果。

在mongo shell 中，如果未使用`var`关键字将返回的游标分配给变量，则会自动迭代游标以访问最匹配查询的前 20 个文档。您可以设置`DBQuery.shellBatchSize`变量以更改自动迭代文档的数量。

要手动迭代结果，请将返回的光标分配给带有`var`关键字的变量，如以下部分所示。

#### 使用 Variable Name

以下 example 使用变量`myCursor`迭代游标并打印匹配的文档：

```powershell
var myCursor = db.bios.find( );

myCursor
```

#### 使用 next()方法

以下 example 使用游标方法next()来访问文档：

```powershell
var myCursor = db.bios.find( );
var myDocument = myCursor.hasNext() ? myCursor.next() : null;

if (myDocument) {
    var myName = myDocument.name;
    print (tojson(myName));
}
```

要打印，您还可以使用`printjson()`方法而不是`print(tojson())`：

```powershell
if (myDocument) {
    var myName = myDocument.name;
    printjson(myName);
}
```

#### 使用 forEach()方法

以下 example 使用游标方法forEach()来迭代游标并访问文档：

```powershell
var myCursor = db.bios.find( );

myCursor.forEach(printjson);
```

### 修改游标行为

mongo shell 和司机提供了几个游标方法，这些方法调用find()方法返回的游标来修改其行为。

#### Order 结果集中的文档

sort()方法对结果集中的文档进行排序。以下操作返回`name`中按`name`字段按升序排序的文档中的文档：

```powershell
db.bios.find().sort( { name: 1 } )
```

sort()对应于 SQL 中的`ORDER BY`语句。

#### 将文档数限制为 Return

limit()方法限制结果集中的文档数。以下操作最多返回bios 系列中的`5`个文档：

```powershell
db.bios.find().limit( 5 )
```

limit()对应于 SQL 中的`LIMIT`语句。

#### 设置结果集的起始点

skip()方法控制结果集的起始点。以下操作会跳过bios 系列中的第一个`5`文档并返回所有剩余文档：

```powershell
db.bios.find().skip( 5 )
```

#### 指定排序规则

整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。

collation()方法为db.collection.find()操作指定整理。

```powershell
db.bios.find( { "name.last": "hopper" } ).collation( { locale: "en_US", strength: 1 } )
```

#### 结合光标方法

以下 statements 链游标方法limit()和sort()：

```powershell
db.bios.find().sort( { name: 1 } ).limit( 5 )
db.bios.find().limit( 5 ).sort( { name: 1 } )
```

这两个语句是等价的; 即：你链接limit()和sort()方法的 order 并不重要。两个 statements return 前五个文档，由'name'上的升序排序 order 确定。

#### 可用的`mongo`Shell游标方法

- [`cursor.allowDiskUse()`]()
- [`cursor.allowPartialResults()`]()
- [`cursor.batchSize()`]()
- [`cursor.close()`]()
- [`cursor.isClosed()`]()
- [`cursor.collation()`]()
- [`cursor.comment()`]()
- [`cursor.count()`]()
- [`cursor.explain()`]()
- [`cursor.forEach()`]()
- [`cursor.hasNext()`]()
- [`cursor.hint()`]()
- [`cursor.isExhausted()`]()
- [`cursor.itcount()`]()
- [`cursor.limit()`]()
- [`cursor.map()`]()

- [`cursor.max()`]()
- [`cursor.maxTimeMS()`]()
- [`cursor.min()`]()
- [`cursor.next()`]()
- [`cursor.noCursorTimeout()`]()
- [`cursor.objsLeftInBatch()`]()
- [`cursor.pretty()`]()
- [`cursor.readConcern()`]()
- [`cursor.readPref()`]()
- [`cursor.returnKey()`]()
- [`cursor.showRecordId()`]()
- [`cursor.size()`]()
- [`cursor.skip()`]()
- [`cursor.sort()`]()
- [`cursor.tailable()`]()
- [`cursor.toArray()`]()



译者：李冠飞

校对：