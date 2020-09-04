# 用地理空间查询查找餐馆
**在本页面**

* [总览](#overview)

* [失真](#distortion)

* [搜索餐厅](#searching)
## 总览

MongoDB的地理空间索引使您可以高效地对包含地理空间形状和点的集合执行空间查询。为了展示地理空间要素的功能并比较不同的方法，本教程将指导您完成为简单地理空间应用程序编写查询的过程。

本教程将简要介绍地理空间索引的概念，然后演示它们在[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin), [`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects), 和 [`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere).中的用法。

假设您正在设计一个移动应用程序，以帮助用户找到纽约市的餐馆。该应用程序必须：

* 使用[`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects)确定用户当前所在的社区
  
* 使用[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)显示附近的餐馆数量，
  
* 使用[`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere)在用户指定距离内查找餐馆
  

本教程将使用**2dsphere**索引来查询有关球形几何的数据。

有关球面和平面几何的更多信息，请参见[Geospatial Models](https://docs.mongodb.com/master/geospatial-queries/#geospatial-geometry).

## <span id="distortion">失真</span>

由于将三维球体（例如地球）投影到平面上的性质，当在地图上可视化时，球形几何形状将显得失真。

例如，以经度纬度点`(0,0)`, `(80,0)`, `(80,80)`, and `(0,80)`. 定义的球形正方形的规格为例。下图描述了此区域覆盖的区域：<img src="https://docs.mongodb.com/manual/_images/geospatial-spherical-square.png" style="zoom: 50%;" />

## <span id="searching">搜索餐厅</span>

### 前提条件

从https://raw.githubusercontent.com/mongodb/docs-assets/geospatial/neighborhoods.json和https://raw.githubusercontent.com/mongodb/docs-assets/geospatial/restaurants.json下载示例数据集。它们分别包含收藏餐馆和社区。

下载数据集后，将它们导入数据库：

```shell
mongoimport <path to restaurants.json> -c=restaurants
mongoimport <path to neighborhoods.json> -c=neighborhoods
```

地理空间索引，几乎总是可以提高[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin) and [`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects) 查询的性能。

由于此数据是地理数据，因此请使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell在每个集合上创建**2dsphere**索引：

```shell
db.restaurants.createIndex({ location: "2dsphere" })
db.neighborhoods.createIndex({ geometry: "2dsphere" })
```

### 探索数据

从[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中检查新创建的餐厅集合中的条目：

```shell
db.restaurants.findOne()
```

该查询返回如下文档：

```shell
    { 
    		location:   
    				type: "Point", 
    				coordinates: [-73.856077, 40.848447]
    		},
    		name: "Morris Park Bake Shop"
    }
```

该餐厅文档对应于下图所示的位置：<br />![](https://docs.mongodb.com/manual/_images/geospatial-single-point.png)<br />由于本教程使用**2dsphere**索引，因此**location**字段中的几何数据必须遵循[GeoJSON 格式](https://docs.mongodb.com/master/reference/geojson/).

现在检查**neighborhoods**集合中的条目：

```shell
db.neighborhoods.findOne()
```

该查询将返回如下文档：

```shell
  {
    		geometry:  
    			type: "Polygon", 
    			coordinates: [[
    				[ -73.99, 40.75 ], 
    				...
    				[ -73.98, 40.76 ], 
    				[ -73.99, 40.75 ] 
    			]]  
    		},  
    		name: "Hell's Kitchen"
   }
```

该几何形状对应于下图所示的区域：

### <img src="https://docs.mongodb.com/manual/_images/geospatial-polygon-hells-kitchen.png" style="zoom: 67%;" /><br />找到当前的街区

假设用户的移动设备可以为用户提供相当准确的位置，那么使用[`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects).很容易找到用户当前的街区。

假设用户位于经度**-73.93414657**和纬度**40.82302903**。 要找到当前邻域，您将使用**GeoJSON**格式的特殊[$geometry](#)字段指定一个点：

```shell
db.neighborhoods.findOne({ geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } })
```

该查询将返回以下结果：

```shell
    {
    		"_id" : ObjectId("55cb9c666c522cafdb053a68"),
    		"geometry" :   
    				"type" : "Polygon",
    				"coordinates" : [
    						[             
    							[          
    									-73.93383000695911,
    									40.81949109558767 
    							],           
    							...     
    						]    
    				] 
    		},
    		"name" : "Central Harlem North-Polo Grounds"
    }
```

### 查找附近的所有餐厅

您还可以查询以查找给定社区中包含的所有餐馆。 在mongo shell中运行以下命令以查找包含用户的社区，然后计算该社区内的餐馆：

```shell
var neighborhood = db.neighborhoods.findOne( { geometry: { $geoIntersects: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] } } } } )
db.restaurants.find( { location: { $geoWithin: { $geometry: neighborhood.geometry } } } ).count()
```

该查询将告诉您，所请求的社区中有127家餐厅，如下图所示：<br />![](https://docs.mongodb.com/manual/_images/geospatial-all-restaurants.png)

### 查找附近的餐厅

要查找点指定距离内的餐厅，可以将[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)与 [`$centerSphere`](https://docs.mongodb.com/master/reference/operator/query/centerSphere/#op._S_centerSphere)一起按未排序的顺序返回结果，或者如果需要按距离对结果进行排序，则可以将**NearSphere**与[`$maxDistance`](https://docs.mongodb.com/master/reference/operator/query/maxDistance/#op._S_maxDistance)一起返回。

#### 未排序$geoWithin

要查找圆形区域内的餐厅，请将[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)与[`$centerSphere`](https://docs.mongodb.com/master/reference/operator/query/centerSphere/#op._S_centerSphere)一起使用。 [`$centerSphere`](https://docs.mongodb.com/master/reference/operator/query/centerSphere/#op._S_centerSphere).是MongoDB特定的语法，它通过以弧度指定中心和半径来表示圆形区域。

[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)不会以任何特定顺序返回文档，因此它可能会首先向用户显示最远的文档。

以下内容将查找距用户五英里范围内的所有餐馆：

```shell
db.restaurants.find({ location:
    { $geoWithin:   
    	{ $centerSphere: [ [ -73.93414657, 40.82302903 ], 5 / 3963.2 ] } } })
```

’s的第二个参数接受以弧度为单位的半径，因此您必须将其除以以英里为单位的地球半径。 有关在距离单位之间进行转换的更多信息，请参见[使用球面几何计算距离](https://docs.mongodb.com/master/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/) 。

#### 用$nearSphere排序

您也可以使用[`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere) 并以米为单位指定[`$maxDistance`](https://docs.mongodb.com/master/reference/operator/query/maxDistance/#op._S_maxDistance) 项。 这将按照从最近到最远的排序顺序返回用户五英里范围内的所有餐馆：

```shell
var METERS_PER_MILE = 1609.34
db.restaurants.find({ location: { $nearSphere: { $geometry: { type: "Point", coordinates: [ -73.93414657, 40.82302903 ] }, $maxDistance: 5 * METERS_PER_MILE } } })
```



译者：杨帅

校对：杨帅