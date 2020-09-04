
# 地理空间查询
**在本页面：**

* [地理空间数据](#data)

* [地理空间索引](#indexes)

* [地理空间查询](#queries)

* [地理空间模型](#models)

* [例子](#example)

  MongoDB支持对地理空间数据的查询操作。 本节介绍MongoDB的地理空间功能。

## <span id="data">**地理空间数据**</span>

在MongoDB中，您可以将地理空间数据存储为[GeoJSON](https://docs.mongodb.com/master/geospatial-queries/#geospatial-geojson) 对象或遗留坐标对。

### GeoJSON对象

要计算类地球体的几何形状，请将位置数据存储为[GeoJSON 对象](https://docs.mongodb.com/master/reference/geojson/)。

要指定**GeoJSON**数据，请使用嵌入的文档:

* 一个名为**type**的字段，用于指定**GeoJSON**对象类型

* 一个名为坐标的字段，用于指定对象的坐标。

如果指定纬度和经度坐标，请先列出经度，然后再列出纬度：

* 有效的经度值在**-180**到**180**之间（包括两者）。

* 有效的纬度值在**-90**到**90**之间（包括两者之间）。

```shell
 <field>: { type: <GeoJSON type> , coordinates: <coordinates> }
```

 例如，要指定[GeoJSON Point](https://docs.mongodb.com/master/reference/geojson/#geojson-point):：

```shell
  location: {
      	type: "Point",  
      	coordinates: [-73.856077, 40.848447]
    }
```

有关MongoDB支持的**GeoJSON**对象的列表以及示例，请参阅[GeoJSON 对象](https://docs.mongodb.com/master/reference/geojson/)。

对**GeoJSON**对象的MongoDB地理空间查询是在球体上计算的； MongoDB使用[WGS84](https://docs.mongodb.com/master/reference/glossary/#term-wgs84)参考系统对**GeoJSON**对象进行地理空间查询。

### 旧版坐标对

在欧几里德平面上计算距离，请将您的位置数据存储为旧坐标对并使用[`2d`](https://docs.mongodb.com/master/geospatial-queries/#geo-2d)索引。 通过将数据转换为GeoJSON Point类型，MongoDB支持通过[`2dsphere`](https://docs.mongodb.com/master/geospatial-queries/#geo-2dsphere)索引对旧坐标对进行球面计算。

要将数据指定为旧版坐标对，可以使用数组(首选)或嵌入式文档。

#### 通过数组指定(首选)：

```shell
  <field>: [ <x>, <y> ]
```

如果指定纬度和经度坐标，请先列出经度，然后再列出纬度； 即：

```shell
  <field>: [<longitude>, <latitude> ]
```

* 有效的经度值在**[-180 180]**。

* 有效的纬度值在**[-90 90]**。

#### 通过嵌入式文档指定：

```shell
  <field>: { <field1>: <x>, <field2>: <y> }
```

如果指定纬度和经度坐标，第一个字段必须包含经度值，而第二个字段必须包含纬度值;即。

```shell
  <field>: { <field1>: <longitude>, <field2>: <latitude> }
```

* 有效的经度值在**[-180 180]**。
* 有效的纬度值在**[-90 90]**。

为了指定旧版坐标对，数组比嵌入式文档更可取，因为某些语言不能保证关联地图的排序。

## <span id="indexes">地理空间索引</span>

MongoDB提供以下地理空间索引类型以支持地理空间查询。

**2dsphere**

索引支持查询，该查询可在类似地球的球体上计算几何形状。

要创建**2dsphere**索引，请使用[`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)方法并指定字符串文字“ **2dsphere**”作为索引类型：

```shell
 db.collection.createIndex( { <location field> : "2dsphere" } )
```

其中**<`location field`>**是其值为GeoJSON对象或旧版坐标对的字段。

有关**2dsphere**索引的更多信息，请参见[`2dsphere`](https://docs.mongodb.com/manual/core/2dsphere/)索引。

**2d**

索引支持在[二维平面上计算几何的查询](https://docs.mongodb.com/master/geospatial-queries/#geospatial-geometry).尽管索引可以支持在球上进行计算的[`$nearSphere`](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere)查询，但如果可能，请对球面查询使用[2dsphere](https://docs.mongodb.com/master/geospatial-queries/#geo-2dsphere)索引。

要创建**2d**索引，请使用[`db.collection.createIndex()`](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex)方法，将location字段指定为键，并将字符串文字**“ 2d”**指定为索引类型：

```shell
db.collection.createIndex( { <location field> : "2d" } )
```

其中**<`location field`>**是一个值为[旧版坐标对](https://docs.mongodb.com/master/geospatial-queries/#geospatial-legacy)的字段。

有关**2d**索引的更多信息，请参见[2d 索引](https://docs.mongodb.com/manual/core/2d/)。

**地理空间索引和分片集合**

分片集合时，不能将地理空间索引用作分片键。但是，可以通过使用不同的字段作为分片键在分片集合上创建地理空间索引。

分片集合支持以下地理空间操作：

* [$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear)聚集阶段

* [$near](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near)和[$nearSphere](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere)查询运算符(从MongoDB 4.0开始).

从MongoDB 4.0开始，分片集合支持[`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near) 和 [`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere)查询。

在早期的MongoDB版本中，分片集合不支持[`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near) 和 [`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere) 查询。相反，对于分片群集，必须使用[`$geoNear`](https://docs.mongodb.com/master/reference/operator/aggregation/geoNear/#pipe._S_geoNear)聚合阶段或**geoNear**命令（在MongoDB 4.0及更低版本中可用）。

您还可以使用[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)和**$geoIntersect**查询分片群集的地理空间数据。

**涵盖查询**

[地理空间索引](https://docs.mongodb.com/master/geospatial-queries/#index-feature-geospatial)不能[覆盖查询](https://docs.mongodb.com/master/core/query-optimization/#covered-queries)。

## <span id="queries">地理空间查询</span>

> **[success] Note**
>
> 对于球形查询，请使用**2dsphere**索引结果。
>
> 将**2d**索引用于球形查询可能会导致错误的结果，例如将**2d**索引用于环绕两极的球形查询。

### 地理空间查询操作符

MongoDB提供以下地理空间查询操作符：

| 名字 | 说明 |
| --- | --- |
| [$geoIntersects](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#op._S_geoIntersects) | 选择与**[GeoJSON](https://docs.mongodb.com/master/reference/glossary/#term-geojson)**几何形状相交的几何形状。 [`2dsphere`](https://docs.mongodb.com/master/core/2dsphere/)索引支持[`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects). |
| [$geoWithin](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin) | 选择边界[GeoJSON几何](https://docs.mongodb.com/master/reference/geojson/#geospatial-indexes-store-geojson)图形内的几何图形。[`2dsphere`](https://docs.mongodb.com/master/core/2dsphere/)和2d索引支持[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin). |
| [$near](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near) | 返回球体上某个点附近的地理空间对象。 需要地理空间索引。 [`2dsphere`](https://docs.mongodb.com/master/core/2dsphere/)和[2d](https://docs.mongodb.com/master/core/2d/)索引支持[`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near). |
| [$nearSphere](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere) | 返回接近球体上某一点的地理空间对象。需要地理空间索引。[`2dsphere`](https://docs.mongodb.com/master/core/2dsphere/)和[2d](https://docs.mongodb.com/master/core/2d/)索引支持[`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere). |

有关更多细节(包括示例)，请参见个别参考页面。

### 地理空间聚集阶段

MongoDB提供以下地理空间聚合管道阶段：

| 步骤 | 说明 |
| --- | --- |
| [$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear) | 根据与地理空间点的接近程度返回有序的文档流。 合并了地理空间数据的[`$match`](https://docs.mongodb.com/master/reference/operator/aggregation/match/#pipe._S_match), [`$sort`](https://docs.mongodb.com/master/reference/operator/aggregation/sort/#pipe._S_sort), 和 [`$limit`](https://docs.mongodb.com/master/reference/operator/aggregation/limit/#pipe._S_limit)功能。 输出文档包括附加距离字段，并且可以包括位置标识符字段。<br />[`$geoNear`](https://docs.mongodb.com/master/reference/operator/aggregation/geoNear/#pipe._S_geoNear)需要一个[地理空间索引](https://docs.mongodb.com/master/core/geospatial-indexes/)。 |

有关更多详细信息(包括示例)，请参见[$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear)参考页。

## <span id="models">地理空间模型</span>

MongoDB地理空间查询可以解释平面或球体上的几何。

**2dsphere**索引仅支持球形查询（即解释球形表面几何形状的查询）。

**2d**索引支持平面查询（即解释平面上的几何图形的查询）和某些球形查询。 虽然**2d**索引支持某些球形查询，但是将**2d**索引用于这些球形查询可能会导致错误。 如果可能，请对球形查询使用**2dsphere**索引。

下表列出了每个地理空间操作所使用的地理空间查询运算符，受支持的查询：

| 操作方式 | 球面/平面查询 | 笔记 |
| --- | --- | --- |
| [$near](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near) ([GeoJSON](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-geojson) centroid point in this line and the following line, [2dsphere](https://docs.mongodb.com/manual/geospatial-queries/#geo-2dsphere) index) | 球形 | 另请参见 [$nearSphere](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere) 运算符，该运算符与GeoJSON和2dsphere索引一起使用时提供相同的功能。 |
| [$near](https://docs.mongodb.com/manual/reference/operator/query/near/#op._S_near) ([legacy coordinates](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-legacy), [2d](https://docs.mongodb.com/manual/geospatial-queries/#geo-2d) index) | 平面 |   |
| [$nearSphere](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere) ([GeoJSON](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-geojson) point, [2dsphere](https://docs.mongodb.com/manual/geospatial-queries/#geo-2dsphere) index) | 球形 | 提供与使用GeoJSON点和2dsphere索引的$ near操作相同的功能。对于球形查询，最好使用$ nearSphere而不是$ near运算符，后者在名称中显式指定球形查询。 |
| [$nearSphere](https://docs.mongodb.com/manual/reference/operator/query/nearSphere/#op._S_nearSphere) ([legacy coordinates](https://docs.mongodb.com/manual/geospatial-queries/#geospatial-legacy), [2d](https://docs.mongodb.com/manual/geospatial-queries/#geo-2d) index) | 球形 | 请改用[GeoJSON](https://docs.mongodb.com/manual/reference/glossary/#term-geojson) 点。 |
| [$geoWithin](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin) : { [$geometry](https://docs.mongodb.com/manual/reference/operator/query/geometry/#op._S_geometry): … } | 球形 |   |
| [$geoWithin](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin) : { [$box](https://docs.mongodb.com/manual/reference/operator/query/box/#op._S_box): … } | 平面 |   |
| [$geoWithin](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin) : { [$polygon](https://docs.mongodb.com/manual/reference/operator/query/polygon/#op._S_polygon): … } | 平面 |   |
| [$geoWithin](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin) : { [$center](https://docs.mongodb.com/manual/reference/operator/query/center/#op._S_center): … } | 平面 |   |
| [$geoWithin](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin) : { [$centerSphere](https://docs.mongodb.com/manual/reference/operator/query/centerSphere/#op._S_centerSphere): … } | 球形 |   |
| [$geoIntersects](https://docs.mongodb.com/manual/reference/operator/query/geoIntersects/#op._S_geoIntersects) | 球形 |   |
| [$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear) aggregation stage ([2dsphere](https://docs.mongodb.com/manual/geospatial-queries/#geo-2dsphere) index) | 球形 |   |
| [$geoNear](https://docs.mongodb.com/manual/reference/operator/aggregation/geoNear/#pipe._S_geoNear) aggregation stage ([2d](https://docs.mongodb.com/manual/geospatial-queries/#geo-2d) index) | 平面 |   |

## <span id="example">例子</span>

用以下文档创建一个集合**places**:

```shell
db.places.insert( { 
		name: "Central Park", 
		location: { type: "Point", coordinates: [ -73.97, 40.77 ] },
		category: "Parks"
} );
db.places.insert( {
		name: "Sara D. Roosevelt Park",
		location: { type: "Point", coordinates: [ -73.9928, 40.7193 ] }, 
		category: "Parks"
);
db.places.insert( {
		name: "Polo Grounds",
		location: { type: "Point", coordinates: [ -73.9375, 40.8303 ] },
		category: "Stadiums"} 
);
```

以下操作在**location**字段上创建**2dsphere**索引：

```shell
db.places.createIndex( { location: "2dsphere" } )
```

以下查询使用[$near](#)运算符返回距指定GeoJSON点至少1000米，最多5000米的文档，并按从最近到最远的顺序排序：<br />

```shell
db.places.find(  
		{   
			location:  
				{ $near:   
					{  
						$geometry: { type: "Point",  coordinates: [ -73.9667, 40.78 ] },   
						$minDistance: 1000,      
						$maxDistance: 5000    
					}   
				} 
		}
)
```

以下操作使用**geoNea**r聚合操作返回与查询过滤器**{category：“ Parks”}**匹配的文档，这些文档按从最接近指定GeoJSON点的最近到最远的顺序排序：

```shell
db.places.aggregate( [
		{     
			$geoNear: { 
				near: { type: "Point", coordinates: [ -73.9667, 40.78 ] }, 
				spherical: **true**,       
				query: { category: "Parks" },  
				distanceField: "calcDistance"  
			} 
		}
])
```



译者：杨帅

校对：杨帅
