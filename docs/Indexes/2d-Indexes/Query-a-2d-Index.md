## 查询一个“2d”索引

**在本页面**

- [在平面上定义的形状内的点](#id1)
- [球体上定义的圆内的点](#id2)
- [接近平面上的一点](#id3)
- [在平面上精确匹配](#id4)

下面的部分描述了 **2d** 索引支持的查询。

### <span id="id1">在平面上定义的形状内的点</span>

要选择平面上给定形状中的所有旧坐标对，请使用[`$geoWithin `](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)操作符和一个形状操作符。使用以下语法:

```powershell
db.<collection>.find( { <location field> :
                         { $geoWithin :
                            { $box|$polygon|$center : <coordinates>
                      } } } )
```

下面查询由左下角的`[0,0]`和右上角的`[100,100]`定义的矩形内的文档。

```powershell
db.places.find( { loc :
                  { $geoWithin :
                     { $box : [ [ 0 , 0 ] ,
                                [ 100 , 100 ] ]
                 } } } )
```

下面查询以`[-74,40.74]`为圆心，半径为` 10 `的圆内的文档:

```powershell
db.places.find( { loc: { $geoWithin :
                          { $center : [ [-74, 40.74 ] , 10 ]
                } } } )
```

关于每种形状的语法和示例，请看下面:

- [`$box`](https://docs.mongodb.com/master/reference/operator/query/box/#op._S_box)
- [`$polygon`](https://docs.mongodb.com/master/reference/operator/query/polygon/#op._S_polygon)
- [`$center`](https://docs.mongodb.com/master/reference/operator/query/center/#op._S_center) (defines a circle)

### <span id="id2">球体上定义的圆内的点</span>

由于遗留的原因，MongoDB支持平面“2d”索引上的基本球形查询。通常，球形计算应该使用`2dsphere `索引，如[2dsphere索引](https://docs.mongodb.com/master/core/2dsphere/)中所述。

要在球体的“球冠”中查询传统坐标对，请[`$geoWithin`](https://docs.mongodb.com/manual/reference/operator/query/geoWithin/#op._S_geoWithin)与[`$centerSphere`](https://docs.mongodb.com/manual/reference/operator/query/centerSphere/#op._S_centerSphere)运算符一起使用。指定一个包含以下内容的数组：

- 圆心的网格坐标
- 圆的半径，以弧度为单位。要计算弧度，请参见 [使用球面几何计算距离](https://docs.mongodb.com/manual/tutorial/calculate-distances-using-spherical-geometry-with-2d-geospatial-indexes/)。

使用以下语法:

```powershell
db.<collection>.find( { <location field> :
                         { $geoWithin :
                            { $centerSphere : [ [ <x>, <y> ] , <radius> ] }
                      } } )
```

下面的示例查询返回以经度 **88 W** 和纬度 **30 N** 为半径的10英里范围内的所有文档。这个例子通过将距离除以地球赤道半径3963.2英里来将距离转换为弧度:

```powershell
db.<collection>.find( { loc : { $geoWithin :
                                 { $centerSphere :
                                    [ [ 88 , 30 ] , 10 / 3963.2 ]
                      } } } )
```

### <span id="id3">接近平面上的一点</span>

接近查询返回最接近定义点的遗留坐标对，并按距离对结果排序。使用[`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near)操作符。操作符需要一个 **2d** 索引。

[`$near`](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near)操作符使用以下语法:

```powershell
db.<collection>.find( { <location field> :
                         { $near : [ <x> , <y> ]
                      } } )
```

例如，请参见[` $near `](https://docs.mongodb.com/master/reference/operator/query/near/#op._S_near)。

### <span id="id4">在平面上的精确匹配</span>

不能使用**2d**索引返回坐标对的精确匹配。在存储坐标的字段上使用升序或降序标量索引，以返回准确的匹配。

在下面的例子中，[` find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)操作将返回一个精确匹配的位置，如果你有一个`{'loc': 1}`索引:

```powershell
db.<collection>.find( { loc: [ <x> , <y> ] } )
```

该查询将返回值为`[`<x>`，` <y>`] `的所有文档。

