# [ ](#)db.collection.validate（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `validate`(true)

       *   验证集合。该方法扫描集合数据和索引的正确性并返回结果。有关输出的详细信息，请参阅验证输出。

db.collection.validate()方法具有以下语法：

```powershell
db.collection.validate( {
   full: <boolean>         // Optional
} )
```

要指定`full`选项，您还可以使用：

```powershell
db.collection.validate( <boolean> ) // full option
```

db.collection.validate()方法可以使用以下可选文档参数：

| 字段   | 类型    | 描述                                                         |
| ------ | ------- | ------------------------------------------------------------ |
| `full` | boolean | 可选的。一个标志，用于确定命令是执行较慢但更彻底的检查还是更快但不太彻底的检查。 <br/>1. 如果`true`，则执行更彻底的检查。 <br/>2. 如果`false`，省略一些检查，但不太彻底的检查。 <br/>默认为`false`。 <br/>从 MongoDB 3.6 开始，对于 WiredTiger 存储引擎，只有`full` 验证过程将强制检查点并将所有内存中数据刷新到磁盘，然后再验证磁盘上的数据。 <br/>在以前的版本中，WT 存储引擎的数据验证 process 总是强制检查点。 |


db.collection.validate()方法是验证 数据库命令的包装。

## <span id="behaviors">行为</span>

`db.collection.validate()`方法可能会占用大量资源，并且可能会影响MongoDB实例的性能。

db.collection.validate()方法获取集合的排他锁。这将阻止对集合的所有读取和写入，直到操作完成。当运行在辅助节点上时，该操作可以阻止该辅助节点上的所有其他操作，直到它完成。

db.collection.validate()方法可能很慢，特别是在较大的数据集上。

> **注意**
>
> 由于验证扫描数据结构的方式，即使完整的集合验证也无法检测到 MMAPv1 存储引擎数据 files 上的所有形式的损坏。

## <span id="examples">例子</span>

*   使用默认设置(即：`full: false`)验证集合`myCollection`
    ```powershell
    db.myCollection.validate()
    ```
    
*   要对集合进行完整验证`myCollection`
	```powershell
	db.myCollection.validate( { full: true } )
    
    db.myCollection.validate(true)
    ```

有关输出的详细信息，请参阅验证输出。



译者：李冠飞

校对：