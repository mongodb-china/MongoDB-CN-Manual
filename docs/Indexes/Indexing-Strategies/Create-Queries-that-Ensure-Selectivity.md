## 创建以确保选择性的查询

选择性是指查询使用索引缩小结果的能力。有效的索引更具选择性，允许MongoDB使用索引来完成与完成查询相关的大部分工作。

为了确保选择性，编写限制索引字段可能的文档数量的查询。编写相对于索引数据具有适当选择性的查询。

> 例子
>
> 假设您有一个名为**“status”**的字段，其中可能的值是`new`和`processing`。如果你在**“status”**上添加索引，你就创建了一个低选择性的索引。索引在查找记录方面帮助不大。
>
> 根据您的查询，一种更好的策略是创建一个包含低选择性字段和另一个字段的 [复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)。例如，您可以在`status`和上创建复合索引`created_at.`
>
> 另一个选择，同样取决于您的用例，可能是使用单独的集合，每个状态一个集合。



> 例子
>
> 考虑一个集合上的索引`{a: 1}`(即键**“a”**按升序排序的索引)，其中**“a”**有三个值均匀分布在集合中:

```powershell
{ _id: ObjectId(), a: 1, b: "ab" }
{ _id: ObjectId(), a: 1, b: "cd" }
{ _id: ObjectId(), a: 1, b: "ef" }
{ _id: ObjectId(), a: 2, b: "jk" }
{ _id: ObjectId(), a: 2, b: "lm" }
{ _id: ObjectId(), a: 2, b: "no" }
{ _id: ObjectId(), a: 3, b: "pq" }
{ _id: ObjectId(), a: 3, b: "rs" }
{ _id: ObjectId(), a: 3, b: "tv" }
```

如果你查询` {a: 2, b: "no"} `， MongoDB必须扫描3个[文档](https://docs.mongodb.com/master/reference/glossary/#term-document)在集合中返回一个匹配的结果。类似地，` {a: {$gt: 1}， b: "tv"} `的查询必须扫描6个文档，也要返回一个结果。

考虑一个集合上的相同索引，其中**“a”**有**9** 个值均匀分布在整个集合中:

```powershell
{ _id: ObjectId(), a: 1, b: "ab" }
{ _id: ObjectId(), a: 2, b: "cd" }
{ _id: ObjectId(), a: 3, b: "ef" }
{ _id: ObjectId(), a: 4, b: "jk" }
{ _id: ObjectId(), a: 5, b: "lm" }
{ _id: ObjectId(), a: 6, b: "no" }
{ _id: ObjectId(), a: 7, b: "pq" }
{ _id: ObjectId(), a: 8, b: "rs" }
{ _id: ObjectId(), a: 9, b: "tv" }
```

如果您查询` {a: 2, b: "cd"} `， MongoDB必须只扫描一个文档来完成查询。索引和查询更具选择性，因为' **a** '的值是均匀分布的，和查询可以使用索引选择特定的文档。

然而，尽管 **a **上的索引更具有选择性，但是像` {a: {$gt: 5}， b: "tv"} `这样的查询仍然需要扫描4个文档。

如果整体选择性很低，并且MongoDB必须读取大量文档才能返回结果，那么有些查询在没有索引的情况下可能执行得更快。要确定性能，请参阅[度量索引使用](https://docs.mongodb.com/master/tutorial/measure-index-use/#indexes-measuring-use)。