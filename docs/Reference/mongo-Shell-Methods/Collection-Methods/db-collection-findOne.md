# [ ](#)db.collection.findOne（）

[]()

在本页面

*   [定义](#definition)
*   [行为](#behavior)
*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `findOne`(查询，投影)

       *   返回一个满足集合或视图上指定查询条件的文档。如果多个文档满足查询，则此方法根据自然订单返回第一个文档，该文档反映磁盘上文档的 order。在上限集合中，natural order 与 insert order 相同。如果没有文档满足查询，则该方法返回 null。

| 参数         | 类型     | 描述                                                         |
| ------------ | -------- | ------------------------------------------------------------ |
| `query`      | document | 可选的。使用query operators指定查询选择条件。                |
| `projection` | document | 可选的。使用投影操作员指定要 return 的字段。省略此参数以 return 匹配文档中的所有字段。 |

`projection`参数采用以下形式的文档：

```powershell
{ field1: <boolean>, field2: <boolean> ... }
```

`<boolean>`可以是以下包含或排除值之一：

*   `1`或`true`包括。即使未在投影参数中明确指定字段，findOne()方法也始终包含_id字段。
*   `0`或`false`排除。

projection 参数不能混合 include 和 exclude 规则，而 exception 则排除`_id`字段。

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| 返回： | 一个文档满足指定为此方法的第一个参数的条件。如果指定`projection`参数，findOne()将返回仅包含`projection`字段的文档。除非您明确排除，否则始终包含`_id`字段。 <br/>虽然类似于find()方法，findOne()方法返回文档而不是游标。 |

## <span id="behavior">行为</span>

### 客户端断开

从MongoDB 4.2开始，如果发出`db.collection.findOne()`断开连接的客户端在操作完成之前断开连接，则MongoDB将标记`db.collection.findOne()`为终止（即`killOp`在操作上）。

## <span id="examples">例子</span>

### 使用空查询规范

以下操作从bios 系列返回单个文档：

```powershell
db.bios.findOne()
```

### 使用查询规范

以下操作返回bios 系列中的第一个匹配文档，其中嵌入文档`name`中的字段`first`以字母`G` **开头，或**字段`birth`小于`new Date('01/01/1945')`：

```powershell
db.bios.findOne(
    {
        $or: [
            { 'name.first' : /^G/ },
            { birth: { $lt: new Date('01/01/1945') } }
        ]
    }
)
```

### 用投影

`projection`参数指定 return 的哪些字段。除非排除属于`_id`字段，否则该参数包含 include 或 exclude 规范，而不是两者。

#### 指定 Return 的字段

以下操作在bios 系列中查找文档，并仅返回`name`，`contribs`和`_id`字段：

```powershell
db.bios.findOne(
    { },
    { name: 1, contribs: 1 }
)
```

#### 除了排除的字段外返回所有内容

以下操作返回bios 系列中的文档，其中`contribs`字段包含元素`OOP`，并返回除`_id`字段，`name`嵌入文档中的`first`字段和`birth`字段之外的所有字段：

```powershell
db.bios.findOne(
    { contribs: 'OOP' },
    { _id: 0, 'name.first': 0, birth: 0 }
)
```

### findOne 结果文档

您不能将游标方法应用于findOne()的结果，因为返回单个文档。您可以直接访问该文档：

```powershell
var myDocument = db.bios.findOne();

if (myDocument) {
   var myName = myDocument.name;

   print (tojson(myName));
}
```



译者：李冠飞

校对：