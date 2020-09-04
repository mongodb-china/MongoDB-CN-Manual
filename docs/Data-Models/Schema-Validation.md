# 模式验证

在本页中

- [指定验证规则](https://docs.mongodb.com/manual/core/schema-validation/#specify-validation-rules)
- [JSON模式](https://docs.mongodb.com/manual/core/schema-validation/#json-schema)
- [其他查询表达式](https://docs.mongodb.com/manual/core/schema-validation/#other-query-expressions)
- [行为](https://docs.mongodb.com/manual/core/schema-validation/#behavior)
- [限制](https://docs.mongodb.com/manual/core/schema-validation/#restrictions)
- [绕过文档验证](https://docs.mongodb.com/manual/core/schema-validation/#bypass-document-validation)
- [附加信息](https://docs.mongodb.com/manual/core/schema-validation/#additional-information)


*版本3.2中的新功能*

MongoDB提供了在更新和插入期间执行模式验证的功能。



## 指定验证规则


验证规则基于每个集合。

要在创建新集合时指定验证规则，请使用[`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection)使用`validator`选项。

若要将文档验证添加到现有集合，请使用[`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod)带有 `validator` 选项的命令。

MongoDB还提供了以下相关选项：

- `validationLevel` 选项，该选项确定MongoDB在更新期间对现有文档应用验证规则的严格程度，以及
- `validationAction` 选项，该选项确定MongoDB是否应显示错误并拒绝违反验证规则的文档，或 `warn` 日志中的违反行为，但允许无效文档。



## JSON模式


*版本3.6中的新功能*

从3.6版开始，MongoDB支持JSON模式验证。要指定JSON模式验证，请使用 [`$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/#op._S_jsonSchema) 操作`validator`表达式中的运算符。

> 注意
>
> 推荐使用JSON模式执行模式验证。
>

例如，以下示例使用JSON模式指定验证规则：

```
db.createCollection("students", {
   validator: {
      $jsonSchema: {
         bsonType: "object",
         required: [ "name", "year", "major", "address" ],
         properties: {
            name: {
               bsonType: "string",
               description: "must be a string and is required"
            },
            year: {
               bsonType: "int",
               minimum: 2017,
               maximum: 3017,
               description: "must be an integer in [ 2017, 3017 ] and is required"
            },
            major: {
               enum: [ "Math", "English", "Computer Science", "History", null ],
               description: "can only be one of the enum values and is required"
            },
            gpa: {
               bsonType: [ "double" ],
               description: "must be a double if the field exists"
            },
            address: {
               bsonType: "object",
               required: [ "city" ],
               properties: {
                  street: {
                     bsonType: "string",
                     description: "must be a string if the field exists"
                  },
                  city: {
                     bsonType: "string",
                     "description": "must be a string and is required"
                  }
               }
            }
         }
      }
   }
})
```


有关详细信息，请参见 [`$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/#op._S_jsonSchema)。



## 其他查询表达式


除了使用[`$jsonSchema`](https://docs.mongodb.com/manual/reference/operator/query/jsonSchema/#op._S_jsonSchema) 操作查询运算符，MongoDB支持使用 [其他查询运算符](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors) 查询选择器，除了[`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near)，[$nearSphere](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/'35；操作/u nearSphere)， [`$text`](https://docs.mongodb.com/manual/reference/operator/query/text/#op._S_text)，和 [`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where) 运算符。

例如，以下示例使用查询表达式指定验证器规则：

```
db.createCollection( "contacts",
   { validator: { $or:
      [
         { phone: { $type: "string" } },
         { email: { $regex: /@mongodb\.com$/ } },
         { status: { $in: [ "Unknown", "Incomplete" ] } }
      ]
   }
} )
```


另请参见

[查询运算符](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors)



## 行为


在更新和插入期间进行验证。将验证添加到集合时，现有文档在修改之前不会进行验证检查。



### 现有文档


`validationLevel` 选项确定MongoDB应用验证规则的操作：

- 如果 `validationLevel` 是 `strict`（默认值），MongoDB会对所有插入和更新应用验证规则。
- 如果 `validationLevel`是 `moderate`，MongoDB将验证规则应用于已满足验证条件的现有文档的插入和更新。使用 `moderate` 级别时，不检查对不符合验证条件的现有文档的更新是否有效。

例如，使用以下文档创建 `contacts` 集合:

```
db.contacts.insert([
   { "_id": 1, "name": "Anne", "phone": "+1 555 123 456", "city": "London", "status": "Complete" },
   { "_id": 2, "name": "Ivan", "city": "Vancouver" }
])
```

发出以下命令将验证器添加到 `contacts` 集合：

```
db.runCommand( {
   collMod: "contacts",
   validator: { $jsonSchema: {
      bsonType: "object",
      required: [ "phone", "name" ],
      properties: {
         phone: {
            bsonType: "string",
            description: "must be a string and is required"
         },
         name: {
            bsonType: "string",
            description: "must be a string and is required"
         }
      }
   } },
   validationLevel: "moderate"
} )
```


这 `contacts` 集合现在有一个使用 `moderate` 验证级别的验证器：

- 如果试图更新 `_id`为1, MongoDB将应用验证规则，因为现有文档与条件匹配。
- 相反，MongoDB不会对`_id`为2的文档应用验证规则，因为它不符合验证规则。

要完全禁用验证，可以将`validationLevel`设置为`off`。


### 接受或拒绝无效文档


`validationAction`选项确定MongoDB如何处理违反验证规则的文档：

- 如果`validationAction` 为`error` （默认值），MongoDB将拒绝任何违反验证条件的插入或更新。
- 如果`validationAction` 为`warn`，MongoDB会记录任何冲突，但允许继续插入或更新。

例如，使用以下JSON模式验证器创建一个`contacts2`集合：

```
db.createCollection( "contacts2", {
   validator: { $jsonSchema: {
      bsonType: "object",
      required: [ "phone" ],
      properties: {
         phone: {
            bsonType: "string",
            description: "must be a string and is required"
         },
         email: {
            bsonType : "string",
            pattern : "@mongodb\.com$",
            description: "must be a string and match the regular expression pattern"
         },
         status: {
            enum: [ "Unknown", "Incomplete" ],
            description: "can only be one of the enum values"
         }
      }
   } },
   validationAction: "warn"
} )
```


使用`warn` [`validationAction`](https://docs.mongodb.com/manual/reference/command/collMod/#validationAction)，MongoDB会记录任何冲突，但允许继续插入或更新。



例如，以下插入操作违反了验证规则：

```
db.contacts2.insert( { name: "Amanda", status: "Updated" } )
```


不过，由于`validationAction` 仅为`warn` ，MongoDB只记录验证冲突消息并允许操作继续：

```
2017-12-01T12:31:23.738-0500 W STORAGE  [conn1] Document would fail validation collection: example.contacts2 doc: { _id: ObjectId('5a2191ebacbbfc2bdc4dcffc'), name: "Amanda", status: "Updated" }
```


## 限制


不能为`admin`、`local`和`config` 数据库中的集合指定验证器。

不能为 `system.*`集合指定验证器。



## 绕过文档验证


用户可以使用`bypassDocumentValidation` 选项绕过文档验证。

以下命令可以使用新选项`bypassDocumentValidation`跳过每个操作的验证：

- [`applyOps`](https://docs.mongodb.com/manual/reference/command/applyOps/#dbcmd.applyOps) 命令
- [`findAndModify`](https://docs.mongodb.com/manual/reference/command/findAndModify/#dbcmd.findAndModify) 命令和 [`db.collection.findAndModify()`](https://docs.mongodb.com/manual/reference/method/db.collection.findAndModify/#db.collection.findAndModify) 方法
- [`mapReduce`](https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce) 命令和 [`db.collection.mapReduce()`](https://docs.mongodb.com/manual/reference/method/db.collection.mapReduce/#db.collection.mapReduce) 方法
- [`insert`](https://docs.mongodb.com/manual/reference/command/insert/#dbcmd.insert) 命令
- [`update`](https://docs.mongodb.com/manual/reference/command/update/#dbcmd.update) 命令
-  为[`聚合`](https://docs.mongodb.com/manual/reference/command/aggregate/#dbcmd.aggregate) 命令和 [`db.collection.aggregate()`](https://docs.mongodb.com/manual/reference/method/db.collection.aggregate/#db.collection.aggregate) 方法提供的过程命令[`$out`](https://docs.mongodb.com/manual/reference/operator/aggregation/out/#pipe._S_out) 和 [`$merge`](https://docs.mongodb.com/manual/reference/operator/aggregation/merge/#pipe._S_merge) 

对于已启用访问控制的部署，若要绕过文档验证，经过身份验证的用户必须具有[`bypassDocumentValidation`](https://docs.mongodb.com/manual/reference/privilege-actions/#bypassDocumentValidation)行动。内置角色[`dbAdmin`](https://docs.mongodb.com/manual/reference/built-in-roles/#dbAdmin) 和 [`restore`](https://docs.mongodb.com/manual/reference/built-in-roles/#restore) 提供此操作。



## 附加信息


另请参见

[`collMod`](https://docs.mongodb.com/manual/reference/command/collMod/#dbcmd.collMod), [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/#db.createCollection), [`db.getCollectionInfos()`](https://docs.mongodb.com/manual/reference/method/db.getCollectionInfos/#db.getCollectionInfos)。



←  [数据建模简介](https://docs.mongodb.com/manual/core/data-modeling-introduction/)  [数据建模概念](https://docs.mongodb.com/manual/core/data-models/) →



译者：张鹏

原文链接：https://docs.mongodb.com/manual/core/schema-validation/
