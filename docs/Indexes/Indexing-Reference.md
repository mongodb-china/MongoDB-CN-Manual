## 索引参考

**在本页面**

- [`mongo`Shell中的索引方法](#方法)
- [索引数据库命令](#命令)
- [地理空间查询选择器](#选择器)
- [索引查询修饰符](#修饰符)

### <span id="方法">`mongo`Shell中的索引方法</span>

| 名称                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) | 在集合上建立索引。                                           |
| [`db.collection.dropIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndex/#db.collection.dropIndex) | 删除集合上的指定索引。                                       |
| [`db.collection.dropIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.dropIndexes/#db.collection.dropIndexes) | 删除集合上的所有索引。                                       |
| [`db.collection.getIndexes()`](https://docs.mongodb.com/manual/reference/method/db.collection.getIndexes/#db.collection.getIndexes) | 返回描述集合中现有索引的文档数组。                           |
| [`db.collection.reIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.reIndex/#db.collection.reIndex) | 重建集合上的所有现有索引。                                   |
| [`db.collection.totalIndexSize()`](https://docs.mongodb.com/manual/reference/method/db.collection.totalIndexSize/#db.collection.totalIndexSize) | 报告集合上的索引使用的总大小。提供围绕输出[`totalIndexSize`](https://docs.mongodb.com/manual/reference/command/collStats/#collStats.totalIndexSize)字段的包装器[`collStats`](https://docs.mongodb.com/manual/reference/command/collStats/#dbcmd.collStats)。 |
| [`cursor.explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain) | 报告有关游标的查询执行计划。                                 |
| [`cursor.hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint) | 强制MongoDB对查询使用特定的索引。                            |
| [`cursor.max()`](https://docs.mongodb.com/manual/reference/method/cursor.max/#cursor.max) | 指定游标的排他上限。用于[`cursor.hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint) |
| [`cursor.min()`](https://docs.mongodb.com/manual/reference/method/cursor.min/#cursor.min) | 指定一个游标的下限值。用于[`cursor.hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint) |

### <span id="命令">索引数据库命令</span>

| 名称                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`createIndexes`](https://docs.mongodb.com/manual/reference/command/createIndexes/#dbcmd.createIndexes) | 为一个集合构建一个或多个索引。                               |
| [`dropIndexes`](https://docs.mongodb.com/manual/reference/command/dropIndexes/#dbcmd.dropIndexes) | 从集合中删除索引。                                           |
| [`compact`](https://docs.mongodb.com/manual/reference/command/compact/#dbcmd.compact) | 对集合进行碎片整理并重建索引。                               |
| [`reIndex`](https://docs.mongodb.com/manual/reference/command/reIndex/#dbcmd.reIndex) | 重建集合上的所有索引。                                       |
| [`validate`](https://docs.mongodb.com/manual/reference/command/validate/#dbcmd.validate) | 内部命令，用于扫描集合的数据并为正确性编制索引。             |
| [`geoSearch`](https://docs.mongodb.com/manual/reference/command/geoSearch/#dbcmd.geoSearch) | 执行使用MongoDB的[干草堆索引](https://docs.mongodb.com/manual/reference/glossary/#term-haystack-index)功能的地理空间查询。 |
| [`checkShardingIndex`](https://docs.mongodb.com/manual/reference/command/checkShardingIndex/#dbcmd.checkShardingIndex) | 验证分片键索引的内部命令。                                   |

### <span id="选择器">地理空间查询选择器</span>

| 名称                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin) | 选择边界[GeoJSON](https://docs.mongodb.com/manual/reference/geojson/#geospatial-indexes-store-geojson)几何内的[几何](https://docs.mongodb.com/manual/reference/geojson/#geospatial-indexes-store-geojson)。该[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/)和[2D](https://docs.mongodb.com/manual/core/2d/)指标支持 [`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin)。 |
| [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#op._S_geoIntersects) | 选择与[GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#term-geojson)几何形状相交的几何形状。该[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/)索引支持 [`$geoIntersects`](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#op._S_geoIntersects)。 |
| [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near) | 返回点附近的地理空间对象。需要地理空间索引。该[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/)和[2D](https://docs.mongodb.com/manual/core/2d/)指标支持 [`$near`](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near)。 |
| [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere) | 返回球体上某个点附近的地理空间对象。需要地理空间索引。该[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/)和[2D](https://docs.mongodb.com/manual/core/2d/)指标支持 [`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere)。 |

### <span id="修饰符">索引查询修饰符</span>

| 名称                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`$explain`](https://docs.mongodb.com/manual/reference/operator/meta/explain/#metaOp._S_explain) | 强制MongoDB报告查询执行计划。请参阅[`explain()`](https://docs.mongodb.com/manual/reference/method/cursor.explain/#cursor.explain)。 |
| [`$hint`](https://docs.mongodb.com/manual/reference/operator/meta/hint/#metaOp._S_hint) | 强制MongoDB使用特定索引。看到[`hint()`](https://docs.mongodb.com/manual/reference/method/cursor.hint/#cursor.hint) |
| [`$max`](https://docs.mongodb.com/manual/reference/operator/meta/max/#metaOp._S_max) | 指定要在查询中使用的索引的*排他*上限。请参阅[`max()`](https://docs.mongodb.com/manual/reference/method/cursor.max/#cursor.max)。 |
| [`$min`](https://docs.mongodb.com/manual/reference/operator/meta/min/#metaOp._S_min) | 指定一个*包容性的*下限为索引在查询中使用。请参阅[`min()`](https://docs.mongodb.com/manual/reference/method/cursor.min/#cursor.min)。 |
| [`$returnKey`](https://docs.mongodb.com/manual/reference/operator/meta/returnKey/#metaOp._S_returnKey) | 强制光标只返回索引中包含的字段。                             |

