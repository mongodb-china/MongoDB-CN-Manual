## 限制扫描条目的数量

本教程描述了如何创建索引来限制对包含[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)表达式和相等条件的查询扫描的索引条目的数量。

集合`inventory`包含以下文档：

```powershell
{ _id: 1, dept: "tech", description: "lime green computer" }
{ _id: 2, dept: "tech", description: "wireless red mouse" }
{ _id: 3, dept: "kitchen", description: "green placemat" }
{ _id: 4, dept: "kitchen", description: "red peeler" }
{ _id: 5, dept: "food", description: "green apple" }
{ _id: 6, dept: "food", description: "red potato" }
```

考虑由各个部门执行文本搜索的通用用例，例如:

```powershell
db.inventory.find( { dept: "kitchen", $text: { $search: "green" } } )
```

为了限制文本搜索只扫描特定部门内的那些文档，创建一个复合索引，首先在字段`dept`上指定一个升序/降序索引键，然后在字段描述上指定一个文本索引键:

```powershell
db.inventory.createIndex(
   {
     dept: 1,
     description: "text"
   }
)
```

然后，特定部门内的文本搜索将限制索引文档的扫描。例如，下面的查询只扫描那些**dept = kitchen**的文档:

```powershell
db.inventory.find( { dept: "kitchen", $text: { $search: "green" } } )
```

> **[success] 注意**
>
> * 复合`text`索引不能包含任何其他特殊索引类型，例如[多键](https://docs.mongodb.com/master/core/index-multikey/#index-type-multi-key)或 [地理空间](https://docs.mongodb.com/master/geospatial-queries/#index-feature-geospatial)索引字段。
>
> * 如果复合`text`索引在 索引键之前包含键，则要`text`执行[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)搜索，查询谓词必须在前面的键上包含**相等匹配条件**。
>
> * 创建复合`text`索引时，所有`text`索引键必须在索引规范文档中相邻列出。

也可以看看

[文字索引](https://docs.mongodb.com/master/core/index-text/)



译者：杨帅