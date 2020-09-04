# [ ](#)db.collection.estimatedDocumentCount（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

## <span id="definition">定义</span>

`db.collection.estimatedDocumentCount`（options）

*版本4.0.3中的新功能。*

返回集合或视图中所有文档的计数。该方法包装`count`命令。

```powershell
db.collection.estimatedDocumentCount( <options> )
```

| 参数      | 类型     | 描述                             |
| --------- | -------- | -------------------------------- |
| `options` | document | 可选的。影响计数行为的其他选项。 |

该`options`文档可以包含以下内容：

| 字段        | 类型    | 描述                             |
| ----------- | ------- | -------------------------------- |
| `maxTimeMS` | integer | 可选的。允许计数运行的最长时间。 |

## <span id="behavior">行为</span>

### 结构

`db.collection.estimatedDocumentCount()`不使用查询过滤器，而是使用元数据返回集合的计数。

### 分片集群

在分片群集上，结果计数将无法正确过滤出 `orphaned document`。

### 不正常关机

不正常关机后，计数可能不正确。

`mongod`使用Wired Tiger存储引擎不正常关闭后，所报告的计数统计信息 `db.collection.estimatedDocumentCount()`可能不准确。

偏移量取决于在最后一个`checkpoint`与异常关闭之间执行的插入，更新或删除操作的数量。检查点通常每60秒出现一次。但是，`mongod`使用非默认`--syncdelay`设置运行的实例可能具有或多或少的频繁检查点。

`validate`在`mongod`异常关闭后，对上的每个集合运行以恢复正确的统计信息。

### 客户端断开

从MongoDB 4.2开始，如果发出`db.collection.estimatedDocumentCount()`断开连接的客户端在操作完成之前断开连接，则MongoDB将标记为终止`db.collection.estimatedDocumentCount()`（即在操作上`killOp`）。

## <span id="examples">例子</span>

以下示例用于 `db.collection.estimatedDocumentCount`检索`orders`集合中所有文档的计数：

```powershell
db.orders.estimatedDocumentCount({})
```

>也可以看看
>
>* `db.collection.countDocuments()`
>* `count`
>* 带有count选项的collStats pipeline stage。



译者：李冠飞

校对：