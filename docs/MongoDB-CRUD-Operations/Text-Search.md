# 文本搜索

**在本页面**

*   [总览](#Overview)

*   [例子](#Example)

*   [语言支持](#Language)

> MONGODB ATLAS搜索
>
> [Atlas搜索](https://docs.atlas.mongodb.com/atlas-search)可以很容易地在MongoDB数据上构建快速、基于相关性的搜索功能。今天就在[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server),上试试吧，我们的完全托管数据库是一种服务。

## <span id="Overview">总览</span>

MongoDB支持执行字符串内容的文本搜索的查询操作。 为了执行文本搜索，MongoDB使用文本索引和[`$text`](#)运算符。

> **[success] Note**
>
> 视图不支持文本搜索

## <span id="Example">例子</span>

此示例演示了如何在仅指定文本字段的情况下构建文本索引并使用它来coffee shops。

使用以下文档创建一个集合存储：

```shell
  db.stores.insert(
  		[
  			{ _id: 1, name: "Java Hut", description: "Coffee and cakes" },
  			{ _id: 2, name: "Burger Buns", description: "Gourmet hamburgers" },
  			{ _id: 3, name: "Coffee Shop", description: "Just coffee" },  
  			{ _id: 4, name: "Clothes Clothes Clothes", description: "Discount clothing" }, 
  			{ _id: 5, name: "Java Shopping", description: "Indonesian goods" } 
  		]
  )
```

### 文字索引

MongoDB提供了[文本索引](https://docs.mongodb.com/master/core/index-text/#index-feature-text)来支持对字符串内容的文本搜索查询。文本索引可以包含值为字符串或字符串元素数组的任何字段。

要执行文本搜索查询，您必须在集合上有一个文本索引。一个集合只能有一个文本搜索索引，但是该索引可以覆盖多个字段。

例如，您可以在[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中运行以下命令，以允许在名称和描述字段中进行文本搜索：

```shell
db.stores.createIndex( { name: "text", description: "text" } )
```

### $text运算符

使用[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)查询操作符对具有文本索引的集合执行[文本索引](https://docs.mongodb.com/master/core/index-text/#index-feature-text)。

[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 将使用空格和大多数标点作为分隔符对搜索字符串进行标记，并在搜索字符串中对所有这些标记执行逻辑或操作。

例如，您可以使用以下查询来查找包含“coffee”、“shop”和“java”列表中任何术语的所有商店:

```shell
db.stores.find( { $text: { $search: "java coffee shop" } } )
```

### 准确的短语

您还可以通过将短语包装在双引号中来搜索精确的短语。如果**$search**字符串包含一个短语和单个术语，文本搜索将只匹配包含该短语的文档。

例如，以下将查找包含“coffee shop”的所有文档：

```shell
db.stores.find( { $text: { $search: "\"coffee shop\"" } } )
```

更多信息参见：请看[Phrases](https://docs.mongodb.com/manual/reference/operator/query/text/#text-operator-phrases).

### 期限排除

要排除一个单词，可以在前面加上一个“-”字符。例如，要查找所有包含“java”或“shop”但不包含“coffee”的商店，请使用以下方法:

```shell
db.stores.find( { $text: { $search: "java shop -coffee" } } )
```

### 排序

默认情况下，MongoDB将以无序的顺序返回结果。但是，文本搜索查询将为每个文档计算一个相关性分数，该分数指定文档与查询的匹配程度。

为了排序的结果在相关性分数的顺序，你必须明确项目[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#proj._S_meta)  **textScore**字段和排序:

```shell
db.stores.find( 
  	{ $text: { $search: "java coffee shop" } },
  	{ score: { $meta: "textScore" } }
  ).sort( { score: { $meta: "textScore" } } )
```

文本搜索也可以在聚合管道中使用。

## <span id="Language">语言支持</span>

MongoDB支持多种语言的文本搜索。 有关支持的语言列表，请参见[文本搜索语言](https://docs.mongodb.com/manual/reference/text-search-languages/)。



译者：杨帅

校对：杨帅