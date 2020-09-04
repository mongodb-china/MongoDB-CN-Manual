# 聚合管道中的文本搜索
**在本页面：**

*  [限制条件](#Restrictions)

*  [文字分数](#Text)

*  [计算包含单词的文章的总浏览量](#Calculate)

*  [返回结果按文本搜索分数排序](#Return)

*  [文字分数匹配](#Match)

*  [指定用于文本搜索的语言](#Specify)

在聚合管道中，可以在[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match) 阶段使用[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)查询运算符来进行文本搜索。

## <span id="Restrictions">限制条件</span>

有关常规的[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 运算符限制，请参见[运算符限制](https://docs.mongodb.com/manual/reference/operator/query/text/#text-query-operator-behavior)。

此外，聚合管道中的文本搜索具有以下限制：

* 包含[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)的[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match)阶段必须是管道中的第一个阶段。
* 文本运算符在阶段只能出现一次。
* 文本运算符表达式不能出现在[`$or`](https://docs.mongodb.com/master/reference/operator/aggregation/or/#exp._S_or) 或[`$not`](https://docs.mongodb.com/master/reference/operator/aggregation/not/#exp._S_not) 表达式中。
* 默认情况下，文本搜索不会按匹配分数的顺序返回匹配的文档。在[`$sort`](https://docs.mongodb.com/master/reference/operator/aggregation/sort/#pipe._S_sort)阶段使用[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#exp._S_meta)聚合表达式。

## <span id="Text">文字分数</span>

 [`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)操作符为索引字段中包含搜索词的每个文档分配一个分数。分数表示文档与给定文本搜索查询的相关性。分数可以是[`$sort`](https://docs.mongodb.com/master/reference/operator/aggregation/sort/#pipe._S_sort)管道规范的一部分，也可以是投影表达式的一部分。**{$meta: "textScore"}**表达式提供处理[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)操作的信息。有关访问投射或排序分数的详细信息，请参阅[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#exp._S_meta) 。

元数据仅在包含 [`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 操作的[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match)阶段之后可用。

### 例子

以下示例假定集合`articles`在字段`subject`上具有文本索引：

```shell
 db.articles.createIndex( { subject: "text" } )
```

## <span id="Calculate">计算包含单词的文章的总浏览量</span>

下面的聚合在[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match)阶段搜索术语cake，并在[`$group`](https://docs.mongodb.com/master/reference/operator/aggregation/group/#pipe._S_group) 阶段计算匹配文档的总视图。

```shell
 db.articles.aggregate(
      [
        { $match: { $text: { $search: "cake" } } },
        { $group: { _id: **null**, views: { $sum: "$views" } } }
      ]
  )
```

## <span id="Return">返回结果按文本搜索分数排序</span>

要根据文本搜索分数进行排序，在[`$sort`](https://docs.mongodb.com/master/reference/operator/aggregation/sort/#pipe._S_sort) 阶段包含[`{$meta: "textScore"}`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#exp._S_meta) 表达式。下面的示例匹配术语**cake**或**tea**，按**textScore**降序排序，并且只返回结果集中的**title**字段。

```shell
db.articles.aggregate(
    [
      { $match: { $text: { $search: "cake tea" } } }, 
      { $sort: { score: { $meta: "textScore" } } }, 
      { $project: { title: 1, _id: 0 } } 
    ]
  )		
```

指定的元数据决定排序顺序。例如，**“textScore”**元数据按降序排序。有关元数据的更多信息以及覆盖元数据的默认排序顺序的示例，请参见[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#exp._S_meta)。

## <span id="Match">文字分数匹配</span>

**“textScore”**元数据可用于包括[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 操作的[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match) 阶段之后的投影、排序和条件。

下面的示例匹配术语**cake**或**tea**，投影标题和分数字段，然后只返回分数大于**1.0**的文档。

```shell
 db.articles.aggregate(
    [
    	{ $match: { $text: { $search: "cake tea" } } },
    	{ $project: { title: 1, _id: 0, score: { $meta: "textScore" } } },
    	{ $match: { score: { $gt: 1.0 } } }
    ]
 )
```

## <span id="Specify">指定用于文本搜索的语言</span>

下面的聚合在[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match) 阶段中以西班牙语搜索包含术语**saber**而不是术语**claro**的文档，并计算[`$group`](https://docs.mongodb.com/master/reference/operator/aggregation/group/#pipe._S_group) 阶段中匹配文档的总视图。

```shell
db.articles.aggregate(
    [   
    		{ $match: { $text: { $search: "saber -claro", $language: "es" } } }, 
    		{ $group: { _id: null, views: { $sum: "$views" } } } 
    ]
 )
```

​    

译者：杨帅

校对：杨帅