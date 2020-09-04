## `2d`索引内部

**在本页面**

- [`2d`索引的Geohash值的计算](#计算)
- [`2d`索引的多位置文档](#文档)

本文对MongoDB的 **2d** 地理空间索引的内部原理进行了更深入的解释。本材料不是正常操作或应用程序开发所必需的，但对于故障排除和进一步理解可能很有用。

### <span id="计算">`2d`索引的Geohash值的计算</span>

当你创建一个地理空间索引[遗留坐标对](https://docs.mongodb.com/manual/reference/glossary/#term-legacy-coordinate-pairs),MongoDB计算[地理散列](https://docs.mongodb.com/manual/reference/glossary/#term-geohash)的坐标对在指定[位置范围](https://docs.mongodb.com/master/tutorial/build-a-2d-index/ # geospatial-indexes-range)和geohash值然后索引。

要计算geohash值，可以递归地将二维映射划分为象限。然后为每个象限分配一个两比特的值。例如，四个象限的两位表示为:

```powershell
01  11

00  10
```

这些两位值(`00`，`01`，`10`，和`11`)表示每个象限和每个象限内的所有点。对于具有两位分辨率的geohash，位于左下象限的所有点的geohash值都为 **00** 。左上角象限的geohash值为 **01** 。右下角和右上角的geohash值分别为 **10** 和 **11** 。

为了提供额外的精度，继续将每个象限划分为子象限。每个子象限都将包含象限的geohash值与子象限的值连接起来。右上象限的geohash为 **11** ，子象限的geohash分别为(从左上方向顺时针方向): **1101** 、 **1111** 、 **1110** 和 **1100** 。

### <span id="文档">用于“2d”索引的多位置文档</span>

> 注意
>
> 索引可以覆盖文档中的多个地理空间字段，并且可以使用[MultiPoint](https://docs.mongodb.com/master/reference/geojson/#geojson-multipoint)嵌入式文档表示点列表。

虽然 **2d** 地理空间索引在文档中不支持多个地理空间字段，但您可以使用[多键索引](https://docs.mongodb.com/master/core/index-multikey/#index-type-multi-key)在单个文档中索引多个坐标对。在最简单的例子中，你可能有一个字段。(例如:`locs `)，它包含一个坐标数组，如下例所示:

```powershell
db.places.save( {
  locs : [ [ 55.5 , 42.3 ] ,
           [ -74 , 44.74 ] ,
           { lng : 55.5 , lat : 42.3 } ]
} )
```

数组的值可以是数组(如`[55.5,42.3]`)，也可以是嵌入文档(如` {lng: 55.5, lat: 42.3} `)。

然后可以在 **locs** 字段上创建地理空间索引，如下所示:

```powershell
db.places.createIndex( { "locs": "2d" } )
```

您还可以将位置数据建模为嵌入文档中的一个字段。在这种情况下，文档将包含一个字段(例如。' **address** ')，它包含一个文档数组，其中每个文档都有一个字段(例如。' **loc:** ')保存位置坐标。例如:

```powershell
db.records.save( {
  name : "John Smith",
  addresses : [ {
                 context : "home" ,
                 loc : [ 55.5, 42.3 ]
                } ,
                {
                 context : "work",
                 loc : [ -74 , 44.74 ]
                }
              ]
} )
```

然后可以在 **addresses** 上创建地理空间索引。字段如下例所示:

```powershell
db.records.createIndex( { "addresses.loc": "2d" } )
```