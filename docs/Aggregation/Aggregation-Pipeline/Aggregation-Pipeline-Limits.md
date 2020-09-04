# [ ](#)聚合管道限制

[]()

在本页面

*   [结果大小限制](#result-size-restrictions)

*   [Memory 限制](#memory-restrictions)

使用[聚合]()命令的聚合操作具有以下限制。

[]()

## <span id="result-size-restrictions">结果大小限制</span>

 Mongodb 3.6版本的改变：MongoDB 3.6 删除[聚合]()命令以将其结果作为单个文档返回的选项。

[聚合]()命令可以返回一个游标或将结果存储集合中。返回游标或将结果存储在集合中时，结果集中的每个文档都受[BSON 文件大小]()限制，目前为 16 兆字节;如果任何单个文档超过[BSON 文件大小]()限制，该命令将产生错误。该限制仅适用于返回的文件;在管道处理期间，文档可能超过此大小。 [db.collection.aggregate()]()方法默认返回游标。

[]()

[]()

## <span id="memory-restrictions">Memory 限制</span>

管道阶段的 RAM 限制为 100M（100\*1024\*1024字节）。如果某个阶段超出此限制，MongoDB 将产生错误。要允许处理大型数据集，可以在`aggregate()`方法中设置`allowDiskUse`选项。`allowDiskUse`选项允许大多数聚合管道操作可以将数据写入临时文件。 以下聚合操作是`allowDiskUse`选项的例外； 这些操作必须在内存限制内：

* `$graphLookup`阶段 
* `$group`阶段中使用的`$addToSet`累加器表达式（从版本4.2.3、4.0.14、3.6.17开始）  
* `$group`阶段使用的`$push`累加器表达式(从版本4.2.3、4.0.14、3.6.17开始)

如果管道包含在`aggregate()`操作中观察`allowDiskUse: true`的其他阶段，那么`allowDiskUse: true`选项对这些其他阶段有效。 

从MongoDB 4.2开始，如果任何聚合阶段由于内存限制而将数据写到临时文件，则分析器日志消息和诊断日志消息包括一个usedDisk指示器。


> **[success] 可以看看**
>
> [$sort and Memory Restrictions]()和[$group Operator and Memory]()。



译者：李冠飞

校对：李冠飞