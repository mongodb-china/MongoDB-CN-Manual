# [ ](#)db.collection.createIndexes（）

[]()

在本页面

*   [定义](#definition)

*   [选项](#options)

*   [行为](#behavior)

*   [例子](#examples)

*   [附加信息](#additional-information)

## <span id="definition">定义</span>

*   `db.collection. createIndexes([*keyPatterns,]options)`
   *   version 3.2 中的新内容。

在集合上创建一个或多个索引。

| 参数          | 类型     | 描述                                                         |
| ------------- | -------- | ------------------------------------------------------------ |
| `keyPatterns` | document | 包含字段和 value 对的文档，其中字段是索引 key，value 描述该字段的索引类型。对于字段的升序索引，请指定的 value;对于降序索引，请指定`-1`的 value。 <br/> MongoDB 支持几种不同的索引类型，包括文本，地理空间和哈希索引。有关更多信息，请参见索引类型。 <br/>从 3.6 开始，您不能将`*`指定为索引 name。<br />MongoDB支持几种不同的索引类型，包括 text，geospatial和hashed索引。有关 更多信息，请参见索引类型。<br />*在版本4.2中进行了更改：* MongoDB 4.2 通配符索引 支持工作负载，用户可以在其中查询自定义字段或集合中各种字段：<br />要在文档中的所有字段和子字段上创建通配符索引，请指定为索引键。创建通配符索引时，不能指定降序索引键。`{ "$**" : 1 }`，您还可以使用可选参数在索引中包括*或*排除特定字段及其子字段 `wildcardProjection`。<br />`_id`默认情况下，通配符索引会忽略该字段。要将`_id`字段包含 在通配符索引中，必须在`wildcardProjection`文档中明确包含它：<br />{<br />   “ wildcardProjection”：{ <br />    “ \_id”：1<br />     “ &lt;field&gt;”：0 &#124; 1<br />   }<br /> }<br />除了显式包含 `_id`字段外，您无法在`wildcardProjection`文档中组合包含和排除语句 。<br />您可以在特定字段及其子路径上创建通配符索引，方法是将该字段的完整路径指定为索引键并附`"$**"`加到该路径：<br />{ "path.to.field.$\*\*" : 1 }<br />特定于路径的通配符索引语法与该`wildcardProjection`选项不兼容 。您不能在指定的路径上指定其他包含或排除。<br />通配符索引键**必须**使用上面列出的语法之一。例如，您不能指定 复合索引键。有关通配符索引的更完整文档（包括对其创建的限制），请参阅通配符索引限制。<br />该featureCompatibilityVersion必须创建通配符索引。有关设置fCV的说明，请参阅 在MongoDB 4.2部署上设置功能兼容版本。`mongod` `4.2` |
| `options`     | document | 可选的。包含一组控制索引创建的选项的文档。有关详细信息，请参阅选项。 |


db.collection.createIndexes()是createIndexes命令周围的 wrapper。

要最小化 building 索引对副本\_set 和分片群集的影响，请使用在副本_Set 上建立索引中所述的滚动索引 build 过程。

## <span id="options">选项</span>

`options`文档包含一组控制索引创建的选项。不同的索引类型可以具有特定于该类型的附加选项。

> **重要**
>
> 为 db.collection.createIndexes()指定选项时，这些选项适用于所有指定的索引。对于 example，如果指定了排序规则选项，则所有创建的索引都将包含该排序规则。
> 如果尝试使用不兼容的选项创建索引，db.collection.createIndexes()将_return 错误。有关更多信息，请参阅选项说明。

更改了 version 3.4：添加了对整理的支持。

### 所有索引类型的选项

除非另有说明，否则以下选项适用于所有索引类型：

更改 version 3.0：`dropDups`选项不再可用。

| 参数                      | 类型     | 描述                                                         |
| ------------------------- | -------- | ------------------------------------------------------------ |
| `background`              | boolean  | 可选的。在后台构建索引，以便操作不会阻止其他数据库活动。在后台指定`true`到 build。默认的 value 是`false`。 |
| `unique`                  | boolean  | 可选的。指定`keyPatterns` array 中指定的每个索引都是独特的指数。唯一索引不接受索引 key value 与索引中现有 value 匹配的文档的插入或更新。 <br/>指定`true`以创建唯一索引。默认的 value 是`false`。 <br/>该选项不适用于哈希索引。 |
| `name`                    | string   | 可选的。索引的 name。如果未指定，MongoDB 通过连接索引字段和 sort order 的名称来生成索引 name。 <br/>无论是用户指定还是生成 MongoDB，索引名称(包括其完整命名空间(即：`database.collection`))都不能超过索引名称限制。 <br/>为db.collection.createIndexes指定的选项适用于 key pattern array 中包含的所有**索引规范**。由于索引名称必须是唯一的，因此如果使用db.collection.createIndexes创建单个索引，则只能指定 name。 |
| `partialFilterExpression` | document | 可选的。如果指定，则仅索引 reference 文档匹配过滤器表达式。有关更多信息，请参见部分索引。 <br/>过滤器表达式可以包括：<br/>等式表达式(即.`field: value`或使用`$eq` operator)，<br/> $exists：true表达式，<br/> $gt，$gte，$lt，$lte表达式，<br/> $type表达式，<br/> $and operator 仅 top-level <br/>您可以指定所有 MongoDB 索引类型的`partialFilterExpression`选项。 <br/> version 3.2 中的新内容。 |
| `sparse`                  | boolean  | 可选的。如果是`true`，索引只有 reference 文件带有指定的字段。这些索引使用较少的空间，但在某些情况下(特别是排序)表现不同。默认的 value 是`false`。有关更多信息，请参见稀疏索引。 <br/>在 version 3.2 中更改：从 MongoDB 3.2 开始，MongoDB 提供了创建部分索引的选项。部分索引提供了稀疏索引功能的超集。如果您使用 MongoDB 3.2 或更高版本，则部分索引应优先于稀疏索引。 <br/> version 2.6 中更改：2 dsphere索引默认为稀疏，并忽略此选项。对于包含`2dsphere` index key(s)的复合索引以及其他类型的键，只有`2dsphere`索引字段确定索引是否引用文档。 <br/> 2 d，geoHaystack和文本索引的行为与2 dsphere索引类似。 |
| `expireAfterSeconds`      | integer  | 可选的。将 value(以秒为单位)指定为TTL，以控制 long MongoDB 如何保留此集合中的文档。有关此功能的更多信息，请参见通过设置 TTL 使集合中的数据过期。这仅适用于TTL索引。 |
| `storageEngine`           | document | 可选的。允许用户为创建的索引配置存储引擎。 <br/> `storageEngine`选项应采用以下形式：<br/> `storageEngine: { <storage-engine-name>: <options> }` <br/>在验证 creating 索引时指定的存储引擎 configuration 选项，并在复制期间记录到OPLOG以支持具有使用不同存储引擎的成员的副本_set。 <br/> version 3.0 中的新内容。 |

### 整理选项

| 参数        | 类型     | 描述                                                         |
| ----------- | -------- | ------------------------------------------------------------ |
| `collation` | document | 可选的。指定索引的整理。 <br/> 整理允许用户为 string 比较指定 language-specific 规则，例如字母和重音标记的规则。 <br/>如果已在集合 level 中指定了排序规则，则：<br/>如果在创建索引时未指定排序规则，MongoDB 将使用集合的默认排序规则创建索引。 <br/>如果在创建索引时指定了排序规则，MongoDB 将使用指定的排序规则创建索引。 <br/>排序规则选项具有以下语法：<br/>排序规则：{<br/> locale：&lt;string&gt;，<br/> caseLevel：&lt;boolean&gt;，<br/> caseFirst：&lt;string&gt;，<br/> strength：&lt;int&gt;，<br/> numericOrdering：&lt;boolean&gt;，<br/> alternate：&lt;string&gt;，<br/> maxVariable：&lt;string&gt;，<br/> backwards ：&lt;boolean&gt; <br/>} <br/>指定排序规则时，`locale`字段是必填字段;所有其他校对字段都是可选的。有关字段的说明，请参阅整理文件。 <br/> version 3.4 中的新内容。 |


以下索引仅支持简单的二进制比较，不支持整理：

*   文本索引，

*   2 d索引和

*   geoHaystack索引。
> **建议**
>
> 要在具有 non-simple 归类的集合上创建`text`，`2d`或`geoHaystack`索引，必须在创建索引时显式指定`{collation: {locale: "simple"} }`。

#### 整理和索引使用

如果已在集合 level 中指定了排序规则，则：

*   如果在创建索引时未指定排序规则，MongoDB 将使用集合的默认排序规则创建索引。

*   如果在创建索引时指定了排序规则，MongoDB 将使用指定的排序规则创建索引。
> **建议**
>
> 通过指定`1`或`2`的归类`strength`，可以创建 case-insensitive 索引。 `1`的整理`strength`的索引是变音符号和 case-insensitive。

与其他索引选项不同，您可以使用不同的排序规则在同一 key(s) 上创建多个索引。要使用相同的 key pattern 但不同的排序规则创建索引，必须提供唯一的索引名称。

要使用索引进行 string 比较，操作还必须指定相同的排序规则。也就是说，如果操作指定了不同的排序规则，则具有排序规则的索引不能支持对索引字段执行 string 比较的操作。

对于 example，集合`myColl`在 string 字段`category`上有一个索引，其中包含整理 locale `"fr"`。

```powershell
db.myColl.createIndex( { category: 1 }, { collation: { locale: "fr" } } )
```

以下查询操作(指定与索引相同的排序规则)可以使用索引：

```powershell
db.myColl.find( { category: "cafe" } ).collation( { locale: "fr" } )
```

但是，以下查询操作(默认情况下使用“简单”二进制文件夹)无法使用索引：

```powershell
db.myColl.find( { category: "cafe" } )
```

对于索引前缀键不是 strings，数组和嵌入文档的复合索引，指定不同排序规则的操作仍然可以使用索引来支持对索引前缀键的比较。

对于 example，集合`myColl`在数字字段`score`和`price`以及 string 字段`category`上具有复合索引;使用用于 string 比较的排序规则 locale `"fr"`创建索引：

```powershell
db.myColl.createIndex(
    { score: 1, price: 1, category: 1 },
    { collation: { locale: "fr" } } )
```

以下操作(使用`"simple"`二进制排序规则进行 string 比较)可以使用索引：

```powershell
db.myColl.find( { score: 5 } ).sort( { price: 1 } )
db.myColl.find( { score: 5, price: { $gt: NumberDecimal( "10" ) } } ).sort( { price: 1 } )
```

以下操作在索引的`category`字段上使用`"simple"`二进制排序规则进行 string 比较，可以使用索引仅满足查询的`score: 5`部分：

```powershell
db.myColl.find( { score: 5, category: "cafe" } )
```

### 文本索引的选项

以下选项仅适用于文本索引：

| 参数                | 类型     | 描述                                                         |
| ------------------- | -------- | ------------------------------------------------------------ |
| `weights`           | document | 可选的。对于文本索引，包含字段和权重对的文档。权重是 1 到 99,999 之间的整数，并且表示该字段相对于其他索引字段在分数方面的重要性。您可以为部分或全部索引字段指定权重。请参阅使用权重控制搜索结果以调整分数。默认的 value 是`1`。 |
| `default_language`  | string   | 可选的。对于文本索引，确定停用词列表的语言以及词干分析器和标记生成器的规则。有关可用语言，请参阅文本搜索语言;有关详细信息和示例，请参阅指定文本索引的语言。默认的 value 是`english`。 |
| `language_override` | string   | 可选的。对于文本索引，集合文档中字段的 name 包含文档的覆盖语言。默认的 value 是`language`。有关 example，请参阅使用任何字段指定文档的语言。 |
| `textIndexVersion`  | integer  | 可选的。 `text`索引 version number。用户可以使用此选项覆盖默认的 version number。 <br/>有关可用版本，请参阅版本。 <br/> version 2.6 中的新内容。 |

### 2dsphere 索引的选项

以下选项仅适用于2 dsphere索引：

| 参数                   | 类型    | 描述                                                         |
| ---------------------- | ------- | ------------------------------------------------------------ |
| `2dsphereIndexVersion` | integer | 可选的。 `2dsphere`索引 version number。用户可以使用此选项覆盖默认的 version number。 <br/>有关可用版本，请参阅版本。 <br/> version 2.6 中的新内容。 |

### 2d 索引的选项
以下选项仅适用于2 d索引：

| 参数   | 类型    | 描述                                                         |
| ------ | ------- | ------------------------------------------------------------ |
| `bits` | integer | 可选的。对于2 d索引，存储位置数据的地理散列 value 的精度数。 <br/> `bits` value 的范围是 1 到 32(含)。默认的 value 是`26`。 |
| `min`  | number  | 可选的。对于2 d索引，经度和纬度值的下包含边界。默认的 value 是`-180.0` |
| `max`  | number  | 可选的。对于2 d索引，经度和纬度值的上包含边界。默认的 value 是`180.0`。 |

### geoHaystack 索引的选项

以下选项仅适用于geoHaystack索引：

| 参数         | 类型   | 描述                                                         |
| ------------ | ------ | ------------------------------------------------------------ |
| `bucketSize` | number | 对于geoHaystack索引，请指定要对位置值进行分组的单位数; 即：group 在同一个存储桶中的那些位置值在指定的单位数内。 <br/> value 必须大于 0。 |

### wildcard索引的选项

以下选项仅适用于 通配符索引：

| 参数               | 类型     | 描述                                                         |
| ------------------ | -------- | ------------------------------------------------------------ |
| wildcardProjection | document | 可选的。允许用户从通配符索引中包括或排除特定的字段路径 。仅当创建通配符索引时，此选项才有效。<br />该`wildcardProjection`选项采用以下形式：<br />wildcardProjection: {<br />   "path.to.field.a" : &lt;value&gt;,<br />   "path.to.field.b" : &lt;value&gt;<br /> }<br />该`<value>`可以是以下几点：<br />1. `1`或`true`将该字段包括在通配符索引中。<br />2. `0`或`false`从通配符索引中排除该字段。<br />`_id`默认情况下，通配符索引会忽略该字段。要将`_id`字段包含 在通配符索引中，必须在`wildcardProjection`文档中明确包含它<br />{<br />   "wildcardProjection" : {<br />     "\_id" : 1,<br />     "&lt;field&gt;" : 0 &#124; 1<br/>}<br/>}<br/>除了显式包含 `_id`字段外，您无法在`wildcardProjection`文档中组合包含和排除语句 。<br/>指定的选项`db.collection.createIndexes`适用于键模式数组中包括的**所有**索引规范。`wildcardProjection`仅在使用创建单个通配符索引时 指定 `db.collection.createIndexes` |

## <span id="behavior">行为</span>

### 并发

*在版本4.2中进行了更改。*

对于featureCompatibilityVersion `"4.2"`，`db.collection.createIndexes()` 使用优化的构建过程，该过程在索引构建的开始和结束时获取并持有对指定集合的排他锁。集合上的所有后续操作必须等到`db.collection.createIndexes()`释放排他锁。`db.collection.createIndexes()`允许在大多数索引构建期间交错进行读写操作。

对于featureCompatibilityVersion `"4.0"`，`db.collection.createIndexes()`使用4.2之前的索引构建过程，默认情况下会在构建过程的整个过程中获取父数据库的互斥锁。4.2之前的构建过程将阻止对数据库*及其*所有集合的所有操作，直到操作完成。`background`索引不使用排他锁。

有关的锁定行为的更多信息`db.collection.createIndexes()`，请参见 填充集合的索引构建。

### 重塑现有的索引

如果您要求`db.collection.createIndexes()`一个或多个已经存在的索引，MongoDB不会重新创建现有的一个或多个索引。

### 指数期权

#### 非归类选项

除排序规则选项外，如果您创建具有一组索引选项的索引，然后尝试重新创建相同的索引但具有不同的索引选项，则MongoDB不会更改选项，也不会重新创建索引。

要更改这些索引选项，请`db.collection.dropIndex()`在`db.collection.createIndexes()`使用新选项运行之前 删除现有索引 。

#### 排序规则选项

与其他索引选项不同，您可以在具有不同排序规则的同一键上创建多个索引。要创建具有相同键模式但排序规则不同的索引，必须提供唯一的索引名称。

### 索引键长度限制

对于将featureCompatibilityVersion（fCV）设置为`"4.0"`或更早版本的MongoDB 2.6至MongoDB版本， 如果现有文档的索引条目超过，则MongoDB **不会**在集合上创建索引。`Maximum Index Key Length`

### 通配符索引

*4.2版中的新功能。*

* `_id`默认情况下，通配符索引会忽略该字段。要将`_id`字段包含 在通配符索引中，必须在`wildcardProjection`文档中明确包含它：

  ```powershell
  {
    "wildcardProjection" : {
      "_id" : 1,
      "<field>" : 0|1
    }
  }
  ```

  除了显式包含 `_id`字段外，您无法在`wildcardProjection`文档中组合包含和排除语句 。

* 该featureCompatibilityVersion必须创建通配符索引。有关设置fCV的说明，请参阅 在MongoDB 4.2部署上设置功能兼容版本。`mongod` `4.2`

* 通配符索引不支持以下索引类型或属性：

  * Compound
  * TTL
  * Text
  * 2d (Geospatial)
  * 2dsphere (Geospatial)
  * Hashed
  * Unique

> **注意**
>
> 通配符索引与通配符文本索引不同并且不兼容 。通配符索引不能支持使用$text运算符的查询。

有关通配符索引限制的完整文档，请参见 通配符索引限制。

有关创建通配符索引的示例，请参见 创建通配符索引。有关通配符索引的完整文档，请参见通配符索引。

## <span id="examples">例子</span>

> **也可以看看**
>
> db.collection.createIndex()用于各种索引规范的示例。

### 创建没有选项的索引

考虑包含类似于以下内容的文档的`restaurants`集合：

```powershell
{
    location: {
        type: "Point",
        coordinates: [-73.856077, 40.848447]
    },
    name: "Morris Park Bake Shop",
    cuisine: "Cafe",
    borough: "Bronx",
}
```

以下 example 在`restaurants`集合上创建两个索引：`borough`字段上的升序索引和`location`字段上的2 dsphere索引。

```powershell
db.restaurants.createIndexes([{"borough": 1}, {"location": "2dsphere"}])
```

### 使用指定的排序规则创建索引

以下 example 在`products`集合上创建两个索引：`manufacturer`字段上的升序索引和`category`字段上的升序索引。两个索引都使用整理指定 locale `fr`和比较强度`2`：

```powershell
db.products.createIndexes( [ { "manufacturer": 1}, { "category": 1 } ],
   { collation: { locale: "fr", strength: 2 } })
```

对于使用相同排序规则的索引键的查询或排序操作，MongoDB 可以使用索引。有关详细信息，请参阅整理和索引使用。

### 创建一个通配符指数

*新的4.2版：*`mongod`featureCompatibilityVersion必须是`4.2`创建通配符索引。有关设置fCV的说明，请参阅 在MongoDB 4.2部署上设置功能兼容版本。 

有关通配符索引的完整文档，请参见 通配符索引。

以下列出了创建通配符索引的示例：

* 在单个字段路径上创建通配符索引
* 在所有字段路径上创建通配符索引
* 在多个特定字段路径上创建通配符索引
* 创建排除多个特定字段路径的通配符索引

#### 在单个字段路径上创建通配符索引

考虑一个集合`products_catalog`，其中文档可能包含一个 `product_attributes`字段。该`product_attributes`字段可以包含任意嵌套的字段，包括嵌入式文档和数组：

```powershell
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0b"),
  "product_name" : "Blaster Gauntlet",
  "product_attributes" : {
     "price" : {
       "cost" : 299.99
       "currency" : USD
     }
     ...
  }
},
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0c"),
  "product_name" : "Super Suit",
  "product_attributes" : {
     "superFlight" : true,
     "resistance" : [ "Bludgeoning", "Piercing", "Slashing" ]
     ...
  },
}
```

以下操作在`product_attributes`字段上创建通配符索引 ：

```powershell
use inventory
db.products_catalog.createIndexes(
  [ { "product_attributes.$**" : 1 } ]
)
```

使用此通配符索引，MongoDB索引的所有标量值 `product_attributes`。如果字段是嵌套的文档或数组，则通配符索引将递归到文档/数组中，并为文档/数组中的所有标量字段建立索引。

通配符索引可以支持`product_attributes`对其嵌套字段之一或其嵌套字段进行任意单字段查询 ：

```powershell
db.products_catalog.find( { "product_attributes.superFlight" : true } )
db.products_catalog.find( { "product_attributes.maxSpeed" : { $gt : 20 } } )
db.products_catalog.find( { "product_attributes.elements" : { $eq: "water" } } )
```

>**注意**
>
>特定于路径的通配符索引语法与该`wildcardProjection`选项不兼容 。有关更多信息，请参见参数文档。

#### 在所有字段路径上创建通配符索引

考虑一个集合`products_catalog`，其中文档可能包含一个 `product_attributes`字段。该`product_attributes`字段可以包含任意嵌套的字段，包括嵌入式文档和数组：

```powershell
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0b"),
  "product_name" : "Blaster Gauntlet",
  "product_attributes" : {
     "price" : {
       "cost" : 299.99
       "currency" : USD
     }
     ...
  }
},
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0c"),
  "product_name" : "Super Suit",
  "product_attributes" : {
     "superFlight" : true,
     "resistance" : [ "Bludgeoning", "Piercing", "Slashing" ]
     ...
  },
}
```

以下操作在所有标量字段（不包括`_id`字段）上创建通配符索引：

```powershell
use inventory
db.products_catalog.createIndexes(
  [ { "$**" : 1 } ]
)
```

使用此通配符索引，MongoDB可以索引集合中每个文档的所有标量字段。如果给定字段是嵌套文档或数组，则通配符索引将递归到文档/数组中，并为文档/数组中的所有标量字段建立索引。

创建的索引可以支持对集合中文档中任意字段的查询：

```powershell
db.products_catalog.find( { "product_price" : { $lt : 25 } } )
db.products_catalog.find( { "product_attributes.elements" : { $eq: "water" } } )
```

>**注意**
>
>\_id`默认情况下，通配符索引会忽略该字段。要将`_id`字段包括 在通配符索引中，必须在`wildcardProjection`文档中明确包含它。有关更多信息，请参见参数文档。

#### 在多个特定字段路径上创建通配符索引

考虑一个集合`products_catalog`，其中文档可能包含一个 `product_attributes`字段。该`product_attributes`字段可以包含任意嵌套的字段，包括嵌入式文档和数组：

```powershell
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0b"),
  "product_name" : "Blaster Gauntlet",
  "product_attributes" : {
     "price" : {
       "cost" : 299.99
       "currency" : USD
     }
     ...
  }
},
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0c"),
  "product_name" : "Super Suit",
  "product_attributes" : {
     "superFlight" : true,
     "resistance" : [ "Bludgeoning", "Piercing", "Slashing" ]
     ...
  },
}
```

以下操作将创建一个通配符索引，并使用该`wildcardProjection`选项在索引中仅包含`product_attributes.elements`和`product_attributes.resistance` 字段的标量值 。

```powershell
use inventory
db.products_catalog.createIndexes(
  [ { "$**" : 1 } ],
  {
    "wildcardProjection" : {
      "product_attributes.elements" : 1,
      "product_attributes.resistance" : 1
    }
  }
)
```

尽管键模式`"$**"`涵盖了文档中的所有字段，但该 `wildcardProjection`字段将索引限制为仅包含的字段。有关的完整文档`wildcardProjection`，请参阅 通配符索引的选项。

如果字段是嵌套文档或数组，则通配符索引将递归到文档/数组中，并索引文档/数组中的所有标量字段。

创建的索引可以支持对以下内容中包含的任何标量字段的查询`wildcardProjection`：

```powershell
db.products_catalog.find( { "product_attributes.elements" : { $eq: "Water" } } )
db.products_catalog.find( { "product_attributes.resistance" : "Bludgeoning" } )
```

>**注意**
>
>通配符索引不支持在`wildcardProjection`文档中混合包含和排除语句，*除非*明确包含该`_id`字段。有关更多信息 `wildcardProjection`，请参见参数文档。

#### 创建一个排除多个特定字段路径的通配符索引

考虑一个集合`products_catalog`，其中文档可能包含一个 `product_attributes`字段。该`product_attributes`字段可以包含任意嵌套的字段，包括嵌入式文档和数组：

```powershell
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0b"),
  "product_name" : "Blaster Gauntlet",
  "product_attributes" : {
     "price" : {
       "cost" : 299.99
       "currency" : USD
     }
     ...
  }
},
{
  "_id" : ObjectId("5c1d358bf383fbee028aea0c"),
  "product_name" : "Super Suit",
  "product_attributes" : {
     "superFlight" : true,
     "resistance" : [ "Bludgeoning", "Piercing", "Slashing" ]
     ...
  },
}
```

以下操作创建一个通配符指数，并使用`wildcardProjection`文件索引的所有标量场的每个文档的集合中，*排除*了 `product_attributes.elements`和`product_attributes.resistance` 字段：

```powershell
use inventory
db.products_catalog.createIndexes(
  [ { "$**" : 1 } ],
  {
    "wildcardProjection" : {
      "product_attributes.elements" : 0,
      "product_attributes.resistance" : 0
    }
  }
)
```

尽管键模式`"$**"`涵盖了文档中的所有字段，但 `wildcardProjection`该字段从索引中排除了指定的字段。有关的完整文档`wildcardProjection`，请参阅 通配符索引的选项。

如果字段是嵌套文档或数组，则通配符索引将递归到文档/数组中，并索引文档/数组中的所有标量字段。

创建的索引可以支持对任何标量字段的查询，**但** 以下项除外`wildcardProjection`：

```powershell
db.products_catalog.find( { "product_attributes.maxSpeed" : { $gt: 25 } } )
db.products_catalog.find( { "product_attributes.superStrength" : true } )
```

>**注意**
>
>通配符索引不支持在`wildcardProjection`文档中混合包含和排除语句，*除非*明确包含该`_id`字段。有关更多信息 `wildcardProjection`，请参见参数文档。


## <span id="additional-information">附加信息</span>

有关索引的其他信息，请参阅：

*   本手册的索引部分用于 MongoDB 中索引和索引的完整文档。

*   db.collection.getIndexes()查看集合的现有索引的规范。

*   文字索引有关 creating `text`索引的详细信息。

*   地理空间索引和geoHaystack 索引用于地理空间查询。

*   TTL 指数表示数据到期。



译者：李冠飞

校对：