# 文本索引
MongoDB提供了[文本索引](https://docs.mongodb.com/master/core/index-text/#index-feature-text)来支持对字符串内容的文本搜索查询。`text`索引可以包含值为字符串或字符串元素数组的任何字段。

要执行文本搜索查询，您必须在集合上有一个**text**索引。一个集合只能有一个文本搜索索引，但是该索引可以覆盖多个字段。

例如，你可以运行以下在一个m[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell允许文本搜索的名称和描述字段:

```shell
db.stores.createIndex( { name: "text", description: "text" } )
```

有关文本索引的完整引用，包括行为、标记和属性，请参阅[Text Indexes](https://docs.mongodb.com/manual/core/index-text/)部分。



译者：杨帅

校对：杨帅