# 用权重控制搜索结果

文本搜索为索引字段中包含搜索词的每个文档分配一个分数。分数决定了文档与给定搜索查询的相关性。

对于文本索引，索引字段的权重表示该字段相对于其他索引字段在文本搜索分数方面的重要性。

对于文档中的每个索引字段，MongoDB将匹配的数量乘以权重并对结果进行求和。然后，MongoDB使用这个总和计算文档的分数。有关按文本分数返回和排序的详细信息，请参阅 [`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#proj._S_meta)操作符。

索引字段的默认权重为1。要调整索引字段的权重，请在[`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法中包含权重选项。

>  **[warning] warning**
>
> 仔细选择权重，以防止需要重新索引。

集合`blog`包含以下文档：

```powershell
{
  _id: 1,
  content: "This morning I had a cup of coffee.",
  about: "beverage",
  keywords: [ "coffee" ]
}

{
  _id: 2,
  content: "Who doesn't like cake?",
  about: "food",
  keywords: [ "cake", "food", "dessert" ]
}
```

要为内容字段和关键字字段创建具有不同字段权重的文本索引，请包含[`createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法的权重选项。例如，下面的命令在三个字段上创建一个索引，并为其中两个字段分配权重:

```powershell
db.blog.createIndex(
   {
     content: "text",
     keywords: "text",
     about: "text"
   },
   {
     weights: {
       content: 10,
       keywords: 5
     },
     name: "TextIndex"
   }
 )
```

文本索引有以下字段和权重:

- `content`的权重是10，
- `keywords`的权重为5，
- `about`的默认权重为1。

这些权重表示索引字段之间的相对重要性。例如，`content`字段中的term匹配有:

- 2倍(即**10:5**)的影响，作为一个词匹配的关键字字段
- 10倍(即**10:1**)的影响，作为一场关于领域的学期比赛的影响。



译者：杨帅