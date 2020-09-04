# 查询一个`2dsphere`索引

**在本页面**

- [多边形绑定的GeoJSON对象](#对象)
- [GeoJSON对象的交集](#交集)
- [接近GeoJSON点](#接近)
- [球体上定义的圆内的点](#球体)

以下各节描述了`2dsphere`索引支持的查询。

## <span id="对象">多边形绑定的GeoJSON对象</span>

该[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)操作符查询在GeoJSON多边形中找到的位置数据。您的位置数据必须以GeoJSON格式存储。使用以下语法:

```powershell
db.<collection>.find( { <location field> :
                         { $geoWithin :
                           { $geometry :
                             { type : "Polygon" ,
                               coordinates : [ <coordinates> ]
                      } } } } )
```

下面的例子选择了全部存在于GeoJSON多边形中的所有点和形状:

```powershell
db.places.find( { loc :
                  { $geoWithin :
                    { $geometry :
                      { type : "Polygon" ,
                        coordinates : [ [
                                          [ 0 , 0 ] ,
                                          [ 3 , 6 ] ,
                                          [ 6 , 1 ] ,
                                          [ 0 , 0 ]
                                        ] ]
                } } } } )
```

## <span id="交集">GeoJSON对象的交集</span>

该[`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects)操作符查询与指定GeoJSON对象相交的位置。如果交点非空，则该位置与该对象相交。这包括具有共享优势的文档。

该[`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects)操作符使用以下语法:

```powershell
db.<collection>.find( { <location field> :
                         { $geoIntersects :
                           { $geometry :
                             { type : "<GeoJSON object type>" ,
                               coordinates : [ <coordinates> ]
                      } } } } )
```

下面的示例使用[`$geoIntersects`](https://docs.mongodb.com/master/reference/operator/query/geoIntersects/#op._S_geoIntersects)选择与`coordinates`数组定义的多边形相交的所有索引点和形状。

```powershell
db.places.find( { loc :
                  { $geoIntersects :
                    { $geometry :
                      { type : "Polygon" ,
                        coordinates: [ [
                                         [ 0 , 0 ] ,
                                         [ 3 , 6 ] ,
                                         [ 6 , 1 ] ,
                                         [ 0 , 0 ]
                                       ] ]
                } } } } )
```

## <span id="接近">接近GeoJSON点</span>

接近查询返回最接近定义点的点，并按距离对结果进行排序。对GeoJSON数据的接近度查询需要一个`2dsphere`索引。

要查询与GeoJSON点的接近程度，请使用任一 [`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near)运算符。距离以米为单位。

该[`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near)使用的语法如下：

```powershell
db.<collection>.find( { <location field> :
                         { $near :
                           { $geometry :
                              { type : "Point" ,
                                coordinates : [ <longitude> , <latitude> ] } ,
                             $maxDistance : <distance in meters>
                      } } } )
```

有关示例，请参见[`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near)。

参见[`$nearSphere`](https://docs.mongodb.com/master/reference/operator/query/nearSphere/#op._S_nearSphere)操作符和:pipeline:$geoNear聚合管道阶段。

## <span id="球体">球体上定义的圆内的点</span>

要在球体的“球冠”中选择所有网格坐标，请[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)与[`$centerSphere`](https://docs.mongodb.com/master/reference/operator/query/centerSphere/#op._S_centerSphere)运算符一起使用 。指定一个包含以下内容的数组：

- 圆心的网格坐标
- 圆的半径，以弧度为单位。要计算弧度，请参见 [使用球面几何计算距离](https://docs.mongodb.com/master/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/)。

使用以下语法：

```powershell
db.<collection>.find( { <location field> :
                         { $geoWithin :
                           { $centerSphere :
                              [ [ <x>, <y> ] , <radius> ] }
                      } } )
```

下面的示例查询网格坐标并返回所有半径为经度 **88 W** 和纬度 **30 N** 的10英里内的文档。示例将10英里的距离转换为弧度，通过除以地球近似的赤道半径3963.2英里:

```powershell
db.places.find( { loc :
                  { $geoWithin :
                    { $centerSphere :
                       [ [ -88 , 30 ] , 10 / 3963.2 ]
                } } } )
```



译者：杨帅