## 通配符索引限制

**在本页面**

- [不兼容的索引类型或属性](#不兼容)
- [不支持的查询和聚合模式](#不支持)
- [分片](#分片)

### <span id="不兼容">不兼容的索引类型或属性</span>

通配符索引不支持以下索引类型或属性：

- [复合](https://docs.mongodb.com/master/core/index-compound/)
- [TTL](https://docs.mongodb.com/master/core/index-ttl/)
- [文本](https://docs.mongodb.com/master/core/index-text/)
- [2d（地理空间）](https://docs.mongodb.com/master/core/geospatial-indexes/)
- [2dsphere（地理空间）](https://docs.mongodb.com/master/core/2dsphere/)
- [Hashed](https://docs.mongodb.com/master/core/index-hashed/)
- [Unique](https://docs.mongodb.com/master/core/index-unique/)

> 注意
>
> 通配符索引与[通配符文本索引](https://docs.mongodb.com/master/core/index-text/#text-index-wildcard)不同，也不兼容。通配符索引不支持使用[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)操作符的查询。

### <span id="不支持">不支持的查询和聚合模式</span>

#### 字段不存在

通配符索引是[sparse](https://docs.mongodb.com/master/core/index-sparse/)的，不索引空字段。因此通配符索引不支持查询字段不存在的文档。

例如，考虑一个在`product_attributes`上具有通配符索引的集合目录。通配符索引不能支持以下查询:

```powershell
db.inventory.find( {"product_attributes" : { $exists : false } } )

db.inventory.aggregate([
  { $match : { "product_attributes" : { $exists : false } } }
])
```

#### 字段等于文档或数组

通配符索引为文档或数组的内容生成条目，而不是文档/数组本身。因此通配符索引不能支持精确的文档/数组相等匹配。通配符索引可以支持查询字段等于空文档{}的位置。

例如，考虑一个在	product_attributes	上具有通配符索引的集合目录。通配符索引不能支持以下查询:

```powershell
db.inventory.find({ "product_attributes" : { "price" : 29.99 } } )
db.inventory.find({ "product_attributes.tags" : [ "waterproof", "fireproof" ] } )

db.inventory.aggregate([{
  $match : { "product_attributes" : { "price" : 29.99 } }
}])

db.inventory.aggregate([{
  $match : { "product_attributes.tags" : ["waterproof", "fireproof" ] } }
}])
```

#### 字段不等于文档或数组

通配符索引为文档或数组的内容生成条目，而不是文档/数组本身。因此通配符索引不能支持精确的文档/数组不等匹配。

例如，考虑一个在`product_attributes`上具有通配符索引的集合目录。通配符索引不能支持以下查询:

```powershell
db.inventory.find( { $ne : [ "product_attributes", { "price" : 29.99 } ] } )
db.inventory.find( { $ne : [ "product_attributes.tags",  [ "waterproof", "fireproof" ] ] } )

db.inventory.aggregate([{
  $match : { $ne : [ "product_attributes", { "price" : 29.99 } ] }
}])

db.inventory.aggregate([{
  $match : { $ne : [ "product_attributes.tags", [ "waterproof", "fireproof" ] ] }
}])
```

#### 字段不等于null

如果给定字段是集合中任何文档中的数组，通配符索引不能支持查询该字段不等于null的文档。

例如，考虑一个在`product_attributes`上具有通配符索引的集合目录。如果`product_attributes`通配符索引不能支持以下查询。标签是集合中任意文档的数组:

```powershell
db.inventory.find( { $ne : [ "product_attributes.tags", null ] } )

db.inventory.aggregate([{
  $match : { $ne : [ "product_attributes.tags", null ] }
}])
```

### 分片

您不能使用通配符索引来分片集合。在要分片的一个或多个字段上创建一个非通配符索引。有关分片键选择的更多信息，请参见[分片 键](https://docs.mongodb.com/master/core/sharding-shard-key/#sharding-shard-key)。