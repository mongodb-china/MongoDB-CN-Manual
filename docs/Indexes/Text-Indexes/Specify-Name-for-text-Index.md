# 指定文本索引的名称

**在本页面**

- [指定`text`索引名称](#指定)
- [使用索引名称删除`text`索引](#使用)

> 在MONGODB 4.2中的改变
>
> 从4.2版本开始，由于[特性兼容性版本](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv)设置为**“4.2”**或更大，MongoDB删除了最大127字节的[索引名长度](https://docs.mongodb.com/master/reference/limits/#Index-Name-Length)限制。在以前的版本或[特性兼容性版本](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv)(fCV)设置为**“4.0”**的MongoDB版本中，索引名必须在这个限制之内。

索引的默认名称由与串联的每个索引字段名称组成`_text`。例如，下面的命令创建一个`text`上的字段索引`content`，`users.comments`和 `users.profiles`：

索引的默认名称由每个索引字段名和**_text**连接起来组成。例如，下面的命令在字段**content**、**users.comments**和**users.profiles**上创建一个文本索引:

```powershell
db.collection.createIndex(
   {
     content: "text",
     "users.comments": "text",
     "users.profiles": "text"
   }
)
```

索引的默认名称是：

```powershell
"content_text_users.comments_text_users.profiles_text"
```

## <span id="指定">指定`text`索引名称</span>

您可以将`name`选项传递给 [`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法：

```powershell
db.collection.createIndex(
   {
     content: "text",
     "users.comments": "text",
     "users.profiles": "text"
   },
   {
     name: "MyTextIndex"
   }
)
```

## <span id="使用">使用索引名称删除`text`索引</span>

无论是[文本](https://docs.mongodb.com/master/core/index-text/)索引具有默认名称或指定一个名称为[文本](https://docs.mongodb.com/master/core/index-text/)索引，删除该[文本](https://docs.mongodb.com/master/core/index-text/)索引，通过索引名称的[`db.collection.dropIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.dropIndex/#db.collection.dropIndex)方法。

例如，考虑以下操作创建的索引:

```powershell
db.collection.createIndex(
   {
     content: "text",
     "users.comments": "text",
     "users.profiles": "text"
   },
   {
     name: "MyTextIndex"
   }
)
```

然后，要删除此文本索引，请将名称传递`"MyTextIndex"`给 [`db.collection.dropIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.dropIndex/#db.collection.dropIndex)方法，如下所示：

```powershell
db.collection.dropIndex("MyTextIndex")
```

若要获取索引的名称，请使用 [`db.collection.getIndexes()`](https://docs.mongodb.com/master/reference/method/db.collection.getIndexes/#db.collection.getIndexes)方法。



译者：杨帅