# [ ](#)$comment

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#example)

## <span id="definition">定义</span>

**$comment**

`$comment`查询操作符将注释与任何具有查询谓词的表达式关联起来。

由于注释会传播到`profile`日志，因此添加注释可以使您的个人资料数据更易于解释和跟踪。

`$comment`运算符的形式为：

```powershell
db.collection.find( { <query>, $comment: <comment> } )
```

## <span id="behavior">行为</span>

您可以将_ `$comment`与任何带查询谓词的表达式一起使用，例如聚合管道中`db.collection.update()`或聚合`$match`阶段中的查询谓词 。有关示例，请参见对聚合表达式附加注释。

## <span id="example">例子</span>

### 附加评论到`find`

以下示例`$comment`在 `find()`操作中添加了：

```powershell
db.records.find(
   {
     x: { $mod: [ 2, 0 ] },
     $comment: "Find even values."
   }
)
```

### 在聚合表达式上附加注释

您可以对`$comment`带查询谓词的任何表达式使用。

以下示例在`$match`阶段中使用运算符`$comment`来阐明操作：

```powershell
db.records.aggregate( [
   { $match: { x: { $gt: 0 }, $comment: "Don't allow negative inputs." } },
   { $group : { _id: { $mod: [ "$x", 2 ] }, total: { $sum: "$x" } } }
] )
```

> **也可以看看**
>
> `$comment`



译者：李冠飞

校对：