# 优化查询性能

**在本页面**

- [创建索引以支持查询](#1)
- [限制查询结果数以减少网络需求](#2)
- [使用投影仅返回必要的数据](#3)
- [使用`$hint`选择一个特定的索引](#4)
- [使用增量运算符在服务器端执行操作](#5)

## <span id="1">创建索引以支持查询</span>

对于常见的查询，请创建[索引](https://docs.mongodb.com/manual/indexes/)。如果一个查询搜索多个字段，请创建一个[复合索引](https://docs.mongodb.com/manual/core/index-compound/#index-type-compound)。扫描索引比扫描集合快得多。索引结构小于文档参考，并按顺序存储参考。

> **例子**
>
> 如果你有一个包含博客帖子的帖子集合，并且你经常发出一个查询，对`author_name`字段排序，那么你可以通过在`author_name`字段上创建一个索引来优化查询:
>
> ```shell
> db.posts.createIndex( { author_name : 1 } )
> ```

索引还可以提高对给定字段进行常规排序的查询的效率。

> **例子**
>
> 如果您定期发出查询排序的`timestamp`字段，然后您可以优化查询创建一个索引的`timestamp`字段:
>
> 创建此索引：
>
> ```shell
> db.posts.createIndex( { timestamp : 1 } )
> ```
>
> 优化此查询：
>
> ```shell
> db.posts.find().sort( { timestamp : -1 } )
> ```

因为MongoDB可以按升序和降序读取索引，所以单键索引的方向并不重要。

索引支持查询，更新操作以及[聚合管道的](https://docs.mongodb.com/manual/core/aggregation-pipeline/#aggregation-pipeline-operators-and-performance)某些阶段 。

在以下情况下，`BinData`更有效地将类型为索引的键存储在索引中：

- 二进制子类型的值在0-7或128-135的范围内，并且
- 字节数组的长度为：0、1、2、3、4、5、6、7、8、10、12、14、16、20、24或32。

## <span id="2">限制查询的结果数以减少网络需求</span>

MongoDB [游标](https://docs.mongodb.com/manual/reference/glossary/#term-cursor)以多个文档为一组返回结果。如果知道所需结果的数量，则可以通过发出该[`limit()`](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit) 方法来减少对网络资源的需求。

这通常与排序操作结合使用。例如，如果您只需要从查询到`posts` 集合的10个结果，则可以发出以下命令：

```shell
db.posts.find().sort( { timestamp : -1 } ).limit(10)
```

有关限制结果的更多信息，请参见 [`limit()`](https://docs.mongodb.com/manual/reference/method/cursor.limit/#cursor.limit)

## <span id="3">使用投影仅返回必要的数据</span>

当您仅需要文档中字段的子集时，可以通过仅返回所需的字段来获得更好的性能：

例如，如果在查询中的`posts`集合，你只需要`timestamp`，`title`，`author`，和`abstract`领域，你会发出以下命令：

复制复制的

```
db 。职位。find （ {}， {  timestamp  ： 1  ， title  ： 1  ， author  ： 1  ， abstract  ： 1 }  ）。排序（ {  时间戳 ： - 1  }  ）
```

有关使用投影的更多信息，请参见 [要从查询返回的项目字段](https://docs.mongodb.com/manual/tutorial/project-fields-from-query-results/#read-operations-projection)。

## <span id="4">使用`$hint`选择一个特定的指数</span>

在大多数情况下，[查询优化器](https://docs.mongodb.com/manual/core/query-plans/#read-operations-query-optimization)为特定操作选择最佳索引。但是，您可以使用[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)方法强制MongoDB使用特定索引。使用 [`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint)以支持性能测试，或在某些查询，您必须选择包含在几个索引中的一个或多个字段。

## <span id="5">使用增量运算符在服务器端执行操作</span>

使用MongoDB的[`$inc`](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)操作符递增或递减文档中的值。操作符在服务器端增加字段的值，作为选择文档、在客户端进行简单修改然后将整个文档写入服务器的替代方法。[`$inc`](https://docs.mongodb.com/manual/reference/operator/update/inc/#up._S_inc)操作符还可以帮助避免竞争条件，当两个应用程序实例查询一个文档、手动增加一个字段并同时将整个文档保存回来时，可能会出现竞争条件。



译者：杨帅

校对：杨帅