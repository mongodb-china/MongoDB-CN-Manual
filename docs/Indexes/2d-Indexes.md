# `2d`索引

在本页面

- [注意事项](#注意)
- [行为](#行为)
- [`sparse` 属性](#属性)
- [排序选项](#选项)

对存储为二维平面上的点的数据使用**2d**索引。**2d**索引用于MongoDB 2.2和更早版本中使用的[旧式坐标对](https://docs.mongodb.com/master/geospatial-queries/#geospatial-legacy)。

在以下情况下使用`2d`索引：

- 您的数据库具有来自MongoDB 2.2或更早版本的旧版[旧版坐标对](https://docs.mongodb.com/master/geospatial-queries/#geospatial-legacy),
- 您不打算将任何位置数据存储为[GeoJSON](https://docs.mongodb.com/master/reference/glossary/#term-geojson)对象。

有关地理空间查询的更多信息，请参见[地理空间查询](https://docs.mongodb.com/master/geospatial-queries/)。

## <span id="注意">注意事项</span>

从MongoDB 4.0开始，您可以在[`$geoNear`](https://docs.mongodb.com/master/reference/operator/aggregation/geoNear/#pipe._S_geoNear)管道阶段指定一个关键选项，以指示要使用的索引字段路径。这允许[`$geoNear`](https://docs.mongodb.com/master/reference/operator/aggregation/geoNear/#pipe._S_geoNear)阶段被用于包含多个2d索引和/或多个 [2dsphere索引](https://docs.mongodb.com/master/core/2dsphere/)的集合:

- 如果您的集合具有多个`2d`索引和/或多个 [2dsphere索引](https://docs.mongodb.com/master/core/2dsphere/)，则必须使用该`key`选项来指定要使用的索引字段路径。
- 如果不指定`key`，则不能有多个 `2d`索引和/或多个[2dsphere索引，](https://docs.mongodb.com/master/core/2dsphere/)因为如果没有使用`key`，则多个`2d`索引或 `2dsphere`索引之间的索引选择是不明确的。

> **[success] 注意**
>
> 如果未指定`key`，并且最多只有一个 `2d`索引索引和/或只有一个`2d`索引索引，则MongoDB首先会寻找`2d`要使用的索引。如果`2d`索引不存在，则MongoDB查找`2dsphere`要使用的索引。

如果位置数据包含GeoJSON对象，则不要使用**2d**索引。要同时在[旧式坐标对](https://docs.mongodb.com/master/geospatial-queries/#geospatial-legacy) 和 [GeoJSON对象上](https://docs.mongodb.com/master/geospatial-queries/#geospatial-geojson)[建立](https://docs.mongodb.com/master/core/2dsphere/)索引，请使用[2dsphere](https://docs.mongodb.com/master/core/2dsphere/)索引。

在对集合进行切分时，不能使用2d索引作为[分片键](https://docs.mongodb.com/master/reference/glossary/#term-shard-key)。但是，您可以通过使用不同的字段作为分片键在切分集合上创建地理空间索引。

## <span id="行为">行为</span>

该`2d`索引支持在[平坦的欧几里德平面](https://docs.mongodb.com/master/geospatial-queries/#geospatial-geometry)上进行的计算。**2d**索引还支持在球体上只计算距离(即 [`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere))，但是对于球体上的几何计算(例如[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin))，将数据存储为[GeoJSON objects](https://docs.mongodb.com/master/geospatial-queries/#geospatial-geojson)并使用2dsphere索引。

**2d**索引可以引用两个字段。第一个必须是位置字段。**2d**复合索引构造首先在location字段上选择的查询，然后根据附加条件过滤这些结果。复合**2d**索引可以覆盖查询。

## <span id="属性">`sparse`属性</span>

`2d`索引总是[稀疏的](https://docs.mongodb.com/master/core/index-sparse/)并且忽略[稀疏](https://docs.mongodb.com/master/core/index-sparse/)选项。如果文档缺少`2d`索引字段（或者该字段是`null`或为空数组），则MongoDB不会将文档条目添加到 `2d`索引中。对于插入，MongoDB会插入文档，但不会添加到`2d`索引中。

对于包含`2d`索引键和其他类型的键的复合索引，只有`2d`索引字段才能确定索引是否引用文档。

## <span id="选项">排序选项</span>

`2d`索引仅支持简单的二进制比较，不支持[排序](https://docs.mongodb.com/master/reference/bson-type-comparison-order/#collation)选项。

要在具有非简单排序规则的集合上创建2d索引，必须在创建索引时显式指定**{collation: {locale: "simple"}}**。



译者：杨帅