# 通配符索引

**在本页面**

- [创建通配符索引](#创建)
- [注意事项](#注意)
- [行为](#行为)
- [限制条件](#限制)
- [通配符索引查询/排序支持](#查询)

MongoDB支持在一个或一组字段上创建索引，以支持查询。由于MongoDB支持动态模式，应用程序可以查询不能提前知道名称或任意名称的字段。

*MongoDB版本中的新功能：* 4.2

MongoDB 4.2引入了通配符索引，以支持针对未知或任意字段的查询。

考虑一个应用程序，该应用程序在该`userMetadata`字段下捕获用户定义的数据 并支持查询该数据：

```powershell

{ "userMetadata" : { "likes" : [ "dogs", "cats" ] } }
{ "userMetadata" : { "dislikes" : "pickles" } }
{ "userMetadata" : { "age" : 45 } }
{ "userMetadata" : "inactive" }
```

管理员希望创建索引来支持对`userMetadata`的任何子字段的查询。

在通配符索引`userMetadata` 可以支持单场查询`userMetadata`， `userMetadata.likes`，`userMetadata.dislikes`，和 `userMetadata.age`：

```powershell
db.userData.createIndex( { "userMetadata.$**" : 1 } )
```

该索引可以支持以下查询：

```powershell
db.userData.find({ "userMetadata.likes" : "dogs" })
db.userData.find({ "userMetadata.dislikes" : "pickles" })
db.userData.find({ "userMetadata.age" : { $gt : 30 } })
db.userData.find({ "userMetadata" : "inactive" })
```

`userMetadata`上的非通配符索引只能支持对`userMetadata`的查询。

> **[warning] 重要**
>
> 通配符索引并非旨在替代基于工作负载的索引计划。有关创建索引以支持查询的更多信息，请参见[创建索引以支持查询](https://docs.mongodb.com/master/tutorial/create-indexes-to-support-queries/#create-indexes-to-support-queries)。有关通配符索引限制的完整文档，请参阅[通配符索引限制](https://docs.mongodb.com/master/reference/index-wildcard-restrictions/#wildcard-index-restrictions)。

## <span id="创建">创建通配符索引</span>

> **[warning] 重要**
>
> 该[featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv)必须创建通配符索引。有关设置fCV的说明，请[参阅MongoDB 4.4部署的特性兼容性版本](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#set-fcv)。

可以使用[`createIndexes`](https://docs.mongodb.com/master/reference/command/createIndexes/#dbcmd.createIndexes)数据库命令或其shell助手[`createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)或[`createIndexes()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndexes/#db.collection.createIndexes)创建通配符索引。

### 在字段上创建通配符索引

索引特定字段的值:

```powershell
db.collection.createIndex( { "fieldA.$**" : 1 } )
```

使用这个通配符索引，MongoDB将索引`fieldA`的所有值。如果字段是嵌套的文档或数组，通配符索引将递归到文档/数组中，并存储文档/数组中所有字段的值。

例如，`product_catalog`集合中的文档可能包含`product_attributes`字段。`product_attributes`字段可以包含任意嵌套的字段，包括嵌入的文档和数组:

```powershell
{
  "product_name" : "Spy Coat",
  "product_attributes" : {
    "material" : [ "Tweed", "Wool", "Leather" ]
    "size" : {
      "length" : 72,
      "units" : "inches"
    }
  }
}

{
  "product_name" : "Spy Pen",
  "product_attributes" : {
     "colors" : [ "Blue", "Black" ],
     "secret_feature" : {
       "name" : "laser",
       "power" : "1000",
       "units" : "watts",
     }
  }
}
```

下面的操作在`product_attributes`字段上创建一个通配符索引:

```powershell
db.products_catalog.createIndex( { "product_attributes.$**" : 1 } )
```

通配符索引可以支持对`product_attributes`或其内嵌字段的任意单字段查询:

```powershell

db.products_catalog.find( { "product_attributes.size.length" : { $gt : 60 } } )
db.products_catalog.find( { "product_attributes.material" : "Leather" } )
db.products_catalog.find( { "product_attributes.secret_feature.name" : "laser" } )
```

> **[success] 注意**
>
> 特定于路径的通配符索引语法与该`wildcardProjection`选项不兼容 。有关更多信息，请参见[通配符索引的选项](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-option)。

有关示例，请参见[在单字段路径上创建通配符索引](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-onepath)。

### 在所有字段上创建通配符索引

要索引文档中所有字段的值(不包括`_id`)，指定`“$**”`作为索引键:

```powershell
db.collection.createIndex( { "$**" : 1 } )
```

使用这个通配符索引，MongoDB为集合中每个文档的所有字段建立索引。如果给定字段是嵌套的文档或数组，通配符索引将递归到文档/数组中，并存储文档/数组中所有字段的值。

有关示例，请参见[在所有字段路径上创建通配符索引](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-allpaths)。

> **[success] 注意**
>
> 通配符索引默认情况下省略**_id**字段。要在通配符索引中包含**_id**字段，必须显式地将其包含在**wildcardProjection**文档中。有关更多信息，请参见[通配符索引选项](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-option)。

### 在多个特定字段上创建通配符索引

索引一个文档中多个特定字段的值:

```powershell
db.collection.createIndex(
  { "$**" : 1 },
  { "wildcardProjection" :
    { "fieldA" : 1, "fieldB.fieldC" : 1 }
  }
)
```

使用这个通配符索引，MongoDB为集合中每个文档的指定字段的所有值建立索引。如果给定字段是嵌套的文档或数组，通配符索引将递归到文档/数组中，并存储文档/数组中所有字段的值。

> **[success] 注意**
>
> 通配符索引不支持在`wildcardProjection`文档中混合包含和排除语句，除非明确包含该`_id`字段。有关详细信息 `wildcardProjection`，请参阅[通配符索引选项](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-option)。

有关示例，请参阅[在通配符索引覆盖范围中包括特定字段](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-inclusion)。

### 创建排除多个特定字段的通配符索引

要为文档中除特定字段路径之外的所有字段的字段建立索引，请执行以下操作 ：

```powershell
db.collection.createIndex(
  { "$**" : 1 },
  { "wildcardProjection" :
    { "fieldA" : 0, "fieldB.fieldC" : 0 }
  }
)
```

使用这个通配符索引，MongoDB为集合中每个文档的所有字段建立索引，不包括指定的字段路径。如果给定字段是嵌套的文档或数组，通配符索引将递归到文档/数组中，并存储文档/数组中所有字段的值。

有关示例，请参见[从通配符索引覆盖率中忽略特定字段](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-exclusion)。

> **[success] 注意**
>
> 通配符索引不支持在`wildcardProjection`文档中混合包含和排除语句，*除非*明确包含该`_id`字段。有关详细信息 `wildcardProjection`，请参阅[通配符索引选项](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#createindex-method-wildcard-option)。

## <span id="注意">注意事项</span>

- 通配符索引可以在任何给定查询谓词中最多支持*一个*字段。有关通配符索引查询支持的更多信息，请参见[通配符索引查询/排序支持](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-index-query-sort-support)。
- 该[featureCompatibilityVersion](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#view-fcv)必须创建通配符索引。有关设置fCV的说明，请参阅 [在MongoDB 4.4部署上设置功能兼容版本](https://docs.mongodb.com/master/reference/command/setFeatureCompatibilityVersion/#set-fcv)。[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) `4.2`
- 通配符索引默认情况下省略`_id`字段。要在通配符索引中包含`_id`字段，必须显式地将其包含在wildcardProjection文档中(即**{“_id”:1}**)。
- 您可以在一个集合中创建多个通配符索引。
- 通配符索引可能与集合中的其他索引覆盖相同的字段。
- 通配符索引是[sparse索引](https://docs.mongodb.com/master/core/index-sparse/)，即使索引字段包含空值，也仅包含具有索引字段的文档的条目。

## <span id="行为">行为</span>

通配符索引在索引对象(例如嵌入的文档)或数组字段时有特定的行为:

- 如果该字段是对象，则通配符索引会下降到该对象中并为其内容建立索引。通配符索引继续下降到它遇到的任何其他嵌入式文档中。
- 如果该字段是一个数组，则通配符索引将遍历该数组并索引每个元素：
  - 如果数组中的元素是对象，则通配符索引会下降到该对象中以如上所述索引其内容。
  - 如果该元素是一个数组--也就是说，其被直接嵌入父阵列内的阵列-然后通配符指数并不能遍历嵌入式阵列，但索引的整个阵列作为一个单一的值。
- 对于所有其他字段，将原始(非对象/数组)值记录到索引中。

通配符索引将继续遍历任何其他嵌套对象或数组，直到达到原始值(即不是对象或数组的字段)为止。然后，它将索引此原始值以及该字段的完整路径。

例如，考虑以下文档：

```powershell
{
  "parentField" : {
    "nestedField" : "nestedValue",
    "nestedObject" : {
      "deeplyNestedField" : "deeplyNestedValue"
    },
    "nestedArray" : [
      "nestedArrayElementOne",
      [ "nestedArrayElementTwo" ]
    ]
  }
}
```

包含`parentField`的通配符索引记录了以下条目:

- `"parentField.nestedField" : "nestedValue"`
- `"parentField.nestedObject.deeplyNestedField" : "deeplyNestedValue"`
- `"parentField.nestedArray" : "nestedArrayElementOne"`
- `"parentField.nestedArray" : ["nestedArrayElementTwo"]`

注意，记录`parentField.nestedArray`不包含每个元素的数组位置。当将元素记录到索引中时，通配符索引会忽略数组元素的位置。通配符索引仍然可以支持包含显式数组索引的查询。有关更多信息，请参见[具有显式数组索引的查询](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-query-support-explicit-array-indices)。

有关嵌套对象的通配符索引行为的更多信息，请参见[嵌套对象](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-index-nested-objects)。

有关嵌套数组的通配符索引行为的更多信息，请参见[嵌套数组](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-index-nested-arrays)。

### 嵌套对象

当通配符索引遇到嵌套对象时，它下降到该对象并对其内容进行索引。例如:

```powershell
{
  "parentField" : {
    "nestedField" : "nestedValue",
    "nestedArray" : ["nestedElement"]
    "nestedObject" : {
      "deeplyNestedField" : "deeplyNestedValue"
    }
  }
}
```

包含`parentField`的通配符索引向下遍历对象并索引其内容:

- 对于本身就是对象（即嵌入式文档）的每个字段，请进入该对象以为其内容编制索引。
- 对于每个是数组的字段，遍历该数组并为其内容建立索引。
- 对于所有其他字段，将原始（非对象/数组）值记录到索引中。

通配符索引继续遍历任何附加的嵌套对象或数组，直到它到达一个基本值(即一个不是对象或数组的字段)。然后，它为这个原始值以及该字段的完整路径建立索引。

给定样本文档，通配符索引将以下记录添加到索引中：

- `"parentField.nestedField" : "nestedValue"`
- `"parentField.nestedObject.deeplyNestedField" : "deeplyNestedValue"`
- `"parentField.nestedArray" : "nestedElement"`

有关嵌套数组的通配符索引行为的更多信息，请参见[嵌套数组](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-index-nested-arrays)。

### 嵌套数组

当通配符索引遇到嵌套数组时，它尝试遍历该数组以索引其元素。如果数组本身是父数组(即嵌入式数组)中的一个元素，通配符索引会将整个数组记录为一个值，而不是遍历其内容。例如:

```powershell
{
  "parentArray" : [
    "arrayElementOne",
    [ "embeddedArrayElement" ],
    "nestedObject" : {
      "nestedArray" : [
        "nestedArrayElementOne",
        "nestedArrayElementTwo"
      ]
    }
  ]
}
```

包含`parentArray`的通配符索引向下到数组中遍历和索引它的内容:

- 对于作为数组（即嵌入式数组）的每个元素，将*整个*数组索引为一个值。
- 对于作为对象的每个元素，请进入该对象以遍历并为其内容编制索引。
- 对于所有其他字段，将原始（非对象/数组）值记录到索引中。

通配符索引继续遍历任何附加的嵌套对象或数组，直到它到达一个基本值(即一个不是对象或数组的字段)。然后，它为这个原始值以及该字段的完整路径建立索引。

给定样本文档，通配符索引将以下记录添加到索引中：

- `"parentArray" : "arrayElementOne"`
- `"parentArray" : ["embeddedArrayElement"]`
- `"parentArray.nestedObject.nestedArray" : "nestedArrayElementOne"`
- `"parentArray.nestedObject.nestedArray" : "nestedArrayElementTwo"`

注意，记录`parentField.nestedArray`不包含每个元素的数组位置。当将元素记录到索引中时，通配符索引会忽略数组元素的位置。通配符索引仍然可以支持包含显式数组索引的查询。有关更多信息，请参见 [具有显式数组索引的查询](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-query-support-explicit-array-indices)。

也可以看看：[`Nested Depth for BSON Documents`](https://docs.mongodb.com/master/reference/limits/#Nested-Depth-for-BSON-Documents).

## <span id="限制">限制条件</span>

- 您不能使用通配符索引来分片集合。在要分片的一个或多个字段上创建一个非通配符索引。有关分片键选择的更多信息，请参见[分片 键](https://docs.mongodb.com/master/core/sharding-shard-key/#sharding-shard-key)。
- 您不能创建[复合](https://docs.mongodb.com/master/core/index-compound/)索引。
- 您不能为通配符索引指定以下属性：
  - [TTL](https://docs.mongodb.com/master/core/index-ttl/)
  - [Unique](https://docs.mongodb.com/master/core/index-unique/)
- 您不能使用通配符语法创建以下索引类型：
  - [2d（地理空间）](https://docs.mongodb.com/master/core/geospatial-indexes/)
  - [2dsphere（地理空间）](https://docs.mongodb.com/master/core/2dsphere/)
  - [Hashed](https://docs.mongodb.com/master/core/index-hashed/)

> **[warning] 重要**
>
> 通配符索引与[通配符文本索引](https://docs.mongodb.com/master/core/index-text/#text-index-wildcard)不同并且不兼容 。通配符索引不能支持使用[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)运算符的查询。

有关通配符索引创建限制的完整文档，请参阅 [不兼容的索引类型或属性](https://docs.mongodb.com/master/reference/index-wildcard-restrictions/#wildcard-index-restrictions-create)。

## <span id="查询">通配符索引查询/排序支持</span>

### 覆盖查询

仅当满足以下所有条件时，通配符索引才能支持[覆盖的查询](https://docs.mongodb.com/master/core/query-optimization/#covered-queries) ：

- 查询计划者选择通配符索引来满足查询谓词。
- 查询谓词*恰好*指定了通配符索引覆盖的一个字段。
- 该投影显式排除`_id`并仅包括查询字段。
- 指定的查询字段永远不会是数组。

考虑`employees`集合上的以下通配符索引：

```powershell
db.products.createIndex( { "$**" : 1 } )
```

下面的操作查询单个字段的姓，并从结果文档中抽取所有其他字段:

```powershell
db.products.find(
  { "lastName" : "Doe" },
  { "_id" : 0, "lastName" : 1 }
)
```

假设指定的`lastName`对象永远不是数组，MongoDB可以使用`$**`通配符索引来支持覆盖查询。

### 包含多个字段的查询谓词

通配符索引最多可以支持一个查询谓词字段。那是：

- MongoDB无法使用非通配符索引来满足查询谓词的一部分，而不能使用通配符索引来满足另一部分。
- MongoDB无法使用一个通配符索引来满足查询谓词的一部分，而使用另一个通配符索引来满足另一部分。
- 即使单个通配符索引可以支持多个查询字段，MongoDB也可以使用通配符索引来仅支持其中一个查询字段。解析所有其余字段而没有索引。

但是，MongoDB可以使用相同的通配符索引来满足查询[`$or`](https://docs.mongodb.com/master/reference/operator/query/or/#op._S_or)或聚合 [`$or`](https://docs.mongodb.com/master/reference/operator/aggregation/or/#exp._S_or)运算符的每个独立参数。

### 查询和排序

MongoDB可以使用通配符索引来满足[`sort()`](https://docs.mongodb.com/master/reference/method/cursor.sort/#cursor.sort)，只有当所有这些都是真的:

- 查询计划者选择通配符索引来满足查询谓词。
- 该`sort()`指定**唯一的**查询谓词场。
- 指定的字段永远不会是数组。

如果不满足上述条件，则MongoDB无法使用通配符索引进行排序。MongoDB不支持[`sort`](https://docs.mongodb.com/master/reference/method/cursor.sort/#cursor.sort) 需要与查询谓词不同的索引的操作。有关更多信息，请参见[索引交集和排序](https://docs.mongodb.com/master/core/index-intersection/#index-intersection-sort)。

考虑以下`products`集合上的通配符索引:

```powershell
db.products.createIndex( { "product_attributes.$**" : 1 } )
```

下面的操作查询单个字段`product_attributes.price`和种类在同一领域:

```powershell
db.products.find(
  { "product_attributes.price" : { $gt : 10.00 } },
).sort(
  { "product_attributes.price" : 1 }
)
```

假设指定的`price`对象永远不是数组，MongoDB可以使用`product_attributes.$**`通配符索引来满足[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)和[`sort()`](https://docs.mongodb.com/master/reference/method/cursor.sort/#cursor.sort)。

### 不支持的查询模式

- 通配符索引不支持查询条件，该条件检查字段是否不存在。
- 通配符索引不支持查询条件，该条件检查字段是否等于文档或数组
- 通配符索引不能支持检查字段是否不等于null的查询条件。

有关详细信息，请参阅[不支持的查询和聚合模式](https://docs.mongodb.com/master/reference/index-wildcard-restrictions/#wildcard-index-restrictions-query-aggregation)。

### 用明确的数组索引查询

MongoDB通配符索引不会在索引期间记录数组中任何给定元素的数组位置。但是，MongoDB仍然可以选择通配符索引来回答包含具有一个或多个显式数组索引（例如，`parentArray.0.nestedArray.0`）的字段路径的查询 。由于为每个连续的嵌套数组定义索引范围的复杂性越来越高，因此，如果该路径包含的`8`显式数组索引不多，MongoDB不会考虑使用通配符索引来回答查询中的给定字段路径。MongoDB仍然可以考虑使用通配符索引来回答查询中的其他字段路径。

例如：

```powershell
{
  "parentObject" : {
    "nestedArray" : [
       "elementOne",
       {
         "deeplyNestedArray" : [ "elementTwo" ]
       }
     ]
  }
}
```

MongoDB可以选择一个通配符索引，其中包括`parentObject`，以满足以下查询:

- `"parentObject.nestedArray.0" : "elementOne"`
- `"parentObject.nestedArray.1.deeplyNestedArray.0" : "elementTwo"`

如果查询谓词中的给定字段路径指定了8个以上的显式数组索引，则MongoDB不会考虑使用通配符索引来回答该字段路径。相反，MongoDB要么选择另一个符合条件的索引来回答查询，*要么*执行集合扫描。

请注意，通配符索引本身对索引时遍历文档的深度没有任何限制；该限制仅适用于明确指定确切数组索引的查询。通过发出没有显式数组索引的相同查询，MongoDB可以选择通配符索引来回答该查询：

- `"parentObject.nestedArray" : "elementOne"`
- `"parentObject.nestedArray.deeplyNestedArray" : "elementTwo"`

也可以看看

[`Nested Depth for BSON Documents`](https://docs.mongodb.com/master/reference/limits/#Nested-Depth-for-BSON-Documents)



译者：杨帅