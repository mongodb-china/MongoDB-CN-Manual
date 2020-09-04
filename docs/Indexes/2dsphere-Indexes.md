# `2dsphere`索引
**在本页面**

- [概述](#概述)
- [版本号](#版本号)
- [注意事项](#注意)
- [创建`2dsphere`索引](#创建)
## <span id="概述">概述</span>
`2dsphere`索引支持计算类似地球的球体上的几何形状的查询。`2dsphere`索引支持所有MongoDB地理空间查询：包含、相交和邻近度查询。 有关地理空间查询的更多信息，请参见[地理空间查询](https://docs.mongodb.com/manual/geospatial-queries/)。

`2dsphere`索引支持存储为[GeoJSON对象](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-geojson)和[旧版坐标对](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-legacy)的数据(另请参阅2dsphere索引字段限制)。对于遗留坐标对，索引将数据转换为GeoJSON[Point](https://docs.mongodb.com/manual/reference/geojson/#geojson-point)。

## <span id="版本号">版本号</span>
| 2dsphere索引版本 | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| 版本3            | MongoDB 3.2引入了一个版本3的**2dsphere**索引。版本3是在MongoDB 3.2和更高版本中创建的**2dsphere**索引的默认版本。 |
| 版本2            | MongoDB 2.6引入了**2dsphere**索引的版本2。版本2是在MongoDB 2.6和3.0系列中创建的**2dsphere**索引的默认版本。 |

要覆盖默认版本并指定其他版本，请在创建索引时包含选项`{“ 2dsphereIndexVersion”：<version>}`。

### `sparse`属性
版本2和更高版本的`2dsphere`索引始终为[sparse](https://docs.mongodb.com/manual/core/index-sparse/)且忽略[sparse](https://docs.mongodb.comhttps://docs.mongodb.com/manual/core/index-sparse/)选项。如果文档缺少`2dsphere`索引所在字段（或者该字段为null或空数组），则MongoDB不会将文档条目添加到索引中。对于插入，MongoDB会插入文档，但不添加到`2dsphere`索引。对于包含`2dsphere`索引键以及其他类型键的复合索引，该索引是否引用文档只取决于`2dsphere`索引字段。

对于包含`2dsphere`索引键和其他类型的键的复合索引，只有`2dsphere`索引字段确定索引是否引用文档。

MongoDB的早期版本仅支持`2dsphere (Version 1)`索引。 默认情况下，`2dsphere (Version 1)`索引不是sparse索引，并且拒绝该字段为空的文档。

### 其他GeoJSON对象
版本2和更高版本的`2dsphere`索引包含对其他GeoJSON对象的支持：[MultiPoint](https://docs.mongodb.com/manual/reference/geojson/#geojson-multipoint)，[MultiLineString](https://docs.mongodb.com/manual/reference/geojson/#geojson-multilinestring)，[MultiPolygon](https://docs.mongodb.com/manual/reference/geojson/#geojson-multipolygon)和[GeometryCollection](https://docs.mongodb.com/manual/reference/geojson/#geojson-geometrycollection)。有关所有受支持的GeoJSON对象的详细信息，请参见[GeoJSON对象](https://docs.mongodb.com/manual/reference/geojson/)。
## <span id="注意">注意事项</span>
### `geoNear`和`$geoNear`的限制
从MongoDB 4.0开始，您可以为[`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear)管道指定一个`key`选项以明确指示要使用的索引字段路径。这使得[`$geoNear`](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear)在具有多个`2dsphere`索引或多个[2d](https://docs.mongodb.com/manual/core/2d/)索引的文档中也能被使用：

- 如果您的集合具有多个`2dsphere`索引或多个[2d](https://docs.mongodb.com/manual/core/2d/)索引，则必须使用`key`选项来指定使用哪个索引字段路径。
- 如果未指定`key`，您将无法使用多个`2dsphere`索引或多个[2d](https://docs.mongodb.com/manual/core/2d/)索引。 因为没有指定`key`时，在多个`2d`索引或`2dsphere`索引中选择索引将变得无法明确。

> **[success] 注意**
>
> 如果您不指定`key`，您将最多只能拥有一个`2dsphere`索引或一个`2dsphere`索引，MongoDB首先寻找`2d`索引。 如果不存在`2d`索引，则MongoDB会寻找`2dsphere`索引。

### 分片键限制
对集合做分片时，不能将`2dsphere`索引用作[分片键](https://docs.mongodb.com/manual/reference/glossary/#term-shard-key)。 但是，您可以通过使用一个不同的字段作为分片键来在分片集合上创建地理空间索引。
### `2dsphere`索引字段限制
具有[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/#)索引的字段必须包含[坐标对](https://docs.mongodb.com/manual/reference/glossary/#term-legacy-coordinate-pairs)或[GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#term-geojson)形式的数据。如果您尝试插入一个在`2dsphere`索引字段中包含非几何数据的文档，或者在一个索引字段中包含非几何数据的集合上构建`2dsphere`索引，该操作将失败。
## <span id="创建">创建`2dsphere`索引</span>
要创建`2dsphere`索引，请使用[`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) 方法并指定字符串`"2dsphere"`作为索引类型：
```sql
db.collection.createIndex( { <location field> : "2dsphere" } )
```
其中的`<location field>`是其值为[GeoJSON对象](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-geojson)或[旧式坐标对](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-legacy)的字段。

与只能引用一个位置字段和另一个字段的复合[2d](https://docs.mongodb.com/manual/core/2d/)索引不同的是，[复合](https://docs.mongodb.com/manual/core/index-compound/#index-type-compound)`2dsphere`索引可以引用多个位置字段及非位置字段。

以下示例，基于一个`places`集合，该集合的文档将位置数据以[GeoJSON Point](https://docs.mongodb.com/manual/reference/geojson/#geojson-point)形式存储在`loc`字段中：

```powershell
db.places.insert(
   {
      loc : { type: "Point", coordinates: [ -73.97, 40.77 ] },
      name: "Central Park",
      category : "Parks"
   }
)
db.places.insert(
   {
      loc : { type: "Point", coordinates: [ -73.88, 40.78 ] },
      name: "La Guardia Airport",
      category : "Airport"
   }
)
```
### 创建`2dsphere`索引
以下操作在位置字段`loc`上创建一个[2dsphere](https://docs.mongodb.com/manual/core/2dsphere/#)索引：
```powershell
db.places.createIndex( { loc : "2dsphere" } )
```
### 使用`2dsphere`索引键创建复合索引
[复合索引](https://docs.mongodb.com/manual/core/index-compound/#index-type-compound)可以包含`2dsphere`索引键和非地理空间索引键。例如，以下操作将创建一个复合索引，其中第一个键`loc`是`2dsphere`索引键，其余键`category`和`names`是非地理空间索引键，并分别指定降序（`-1`）和升序（`1`）。

```powershell
db.places.createIndex( { loc : "2dsphere" , category : -1, name: 1 } )
```
与[2d](https://docs.mongodb.com/manual/core/2d/)索引不同，复合`2dsphere`索引不需要将位置字段作为第一个索引字段。 例如：
```powershell
db.places.createIndex( { category : 1 , loc : "2dsphere" } )
```



译者：杨帅 周正
