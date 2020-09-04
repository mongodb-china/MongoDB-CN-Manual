# [ ](#)db.collection.dropIndex（）

[]()

在本页面

*   [定义](#definition)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.dropIndex`(索引)
   *   从集合中删除或删除指定的索引。 db.collection.dropIndex()方法在dropIndexes命令周围提供包装。
> **注意**
>
> 您不能删除`_id`字段上的默认索引。
>
> 从MongoDB 4.2开始，您不能指定 `db.collection.dropIndex("*")`删除所有非`_id`索引。使用 `db.collection.dropIndexes()`代替。

db.collection.dropIndex()方法采用以下参数：

| 参数    | 类型               | 描述                                                         |
| ------- | ------------------ | ------------------------------------------------------------ |
| `index` | string or document | 指定要删除的索引。您可以通过索引 name 或索引规范文档指定索引。 [[1]]() <br/>要删除文本索引，请指定索引 name。<br />从MongoDB 4.2开始，您不能指定`"*"`删除所有非`_id`索引。使用 `db.collection.dropIndexes()`代替。 |


要获取db.collection.dropIndex()方法的索引 name 或索引规范文档，请使用db.collection.getIndexes()方法。

> **警告**
>
> 此命令在受影响的数据库上获取写锁定，并将阻止其他操作，直到完成为止。

## 行为

从MongoDB 4.2开始，该`dropIndex()`操作只会终止使用正在删除的索引的查询。这可能包括将索引视为查询计划一部分的 查询。

在MongoDB 4.2之前，在集合上删除索引将杀死该集合上所有打开的查询。

### 资源锁定

*在版本4.2中进行了更改。*

`db.collection.dropIndex()`在操作期间获得对指定集合的排他锁。集合上的所有后续操作都必须等到`db.collection.dropIndex()`释放锁为止。

在MongoDB 4.2之前的版本中，`db.collection.dropIndex()`获得了对父数据库的排他锁，阻止了对数据库*及其*所有集合的所有操作，直到操作完成。

## <span id="examples">例子</span>

考虑一个`pets`集合。在`pets`集合上调用getIndexes()方法将返回以下索引：

```powershell
[
    {
        “v“ : 1,
        “key“ : { “_id“ : 1 },
        “ns“ : “test.pets“,
        “name“ : “_id_“
    },
    {
        “v“ : 1,
        “key“ : { “cat“ : -1 },
        “ns“ : “test.pets“,
        “name“ : “catIdx“
    },
    {
        “v“ : 1,
        “key“ : { “cat“ : 1, “dog“ : -1 },
        “ns“ : “test.pets“,
        “name“ : “cat_1_dog_-1“
    }
]
```

字段上的单个字段索引`cat`具有用户指定的名称`catIdx` [[2]]()和索引指定文档为 。`{ "cat" : -1 }`

要删除索引`catIdx`，可以使用索引 name：

```powershell
db.pets.dropIndex( “catIdx“ )
```

或者您可以使用索引规范文档`{ “cat“ : -1 }`：

```powershell
db.pets.dropIndex( { “cat“ : -1 } )
```

| <br /> |                                                              |
| ------ | ------------------------------------------------------------ |
| [1]    | (1，2)在 2.2.2 之前使用mongo shell version 时，如果在创建索引期间指定了 name，则必须使用 name 删除索引。 |

| <br/> |                                                              |
| ----- | ------------------------------------------------------------ |
| [2]   | 在创建索引期间，如果用户不**指定索引 name，则系统通过将索引 key 字段和 value 与下划线如：连接来生成 name。 `cat_1`。 |



译者：李冠飞

校对：