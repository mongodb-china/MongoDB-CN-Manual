
# 文本搜索运算符
**在本页面**

*   [查询框架](#query)

*  [聚合框架](#aggregation)

> **[success] Note**
>
> 视图不支持文本搜索。

## <span id="query">查询框架</span>

使用[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)查询操作符对具有文本索引的集合执行文本搜索。

[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)将使用空格和大多数标点作为分隔符对搜索字符串进行标记，并在搜索字符串中对所有这些标记执行逻辑或操作。

例如，您可以使用以下查询来查找包含“coffee”、“shop”和“java”列表中任何术语的所有商店:

```shell
db.stores.find( { $text: { $search: "java coffee shop" } } )
```

使用[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#proj._S_meta)查询操作符获取每个匹配文档的相关性分数并进行排序。例如，要按相关性排序一份coffee shops 列表，运行以下命令:

```shell
db.stores.find(
  		{ $text: { $search: "coffee shop cake" } },
  		{ score: { $meta: "textScore" } }
 ).sort( { score: { $meta: "textScore" } } )
```

有关 [`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 和[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#proj._S_meta) 操作符的更多信息，包括限制和行为，请参见:

 - [$text 参考页面](https://docs.mongodb.com/manual/reference/operator/query/text/#op._S_text)
 - [$text 查询示例](https://docs.mongodb.com/manual/reference/operator/query/text/#text-query-examples)
 - [$meta](https://docs.mongodb.com/manual/reference/operator/projection/meta/#proj._S_meta) projection operator

## <span id="aggregation">聚合框架</span>

在使用[聚合](https://docs.mongodb.com/master/aggregation/)框架时，使用[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match) 和[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 表达式来执行文本搜索查询。要按照相关性评分对结果排序，请在[`$sort`](https://docs.mongodb.com/master/reference/operator/aggregation/sort/#pipe._S_sort) 阶段使用[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#exp._S_meta)聚合操作符

有关聚合框架中文本搜索的更多信息和示例，请参见[聚合管道中的文本搜索](https://docs.mongodb.com/manual/tutorial/text-search-in-aggregation/).。



译者：杨帅

校对：杨帅

