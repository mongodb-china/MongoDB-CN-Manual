# GeoJSON对象

**在本页面**

*   [总览](#概观)

*   [`Point`](#点)

*   [`LineString`](#线串)

*   [多边形](#多边形)

*   [多点](#多点)

*   [MULTILINESTRING](#id1)

*   [MultiPolygon](#id2)

*   [GeometryCollection](#id3)

## <span id="概观">总览</span>

MongoDB 支持此页面上列出的 GeoJSON object 类型。

要指定 GeoJSON 数据，请使用嵌入式文档：

*   一个名为`type`的字段，用于指定[GeoJSON对象类型](https://docs.mongodb.com/master/reference/geojson/#)

*   一个名为`coordinates`的字段，用于指定 object 的坐标。

如果指定纬度和经度坐标，请首先列出**经度**，然后列出**纬度**：

*   有效的经度值介于**[-180 180]**。

* 有效纬度值介于**[-90 90]**。

```powershell
<field>: { type: <GeoJSON type> , coordinates: <coordinates> }
```

GeoJSON objects 上的 MongoDB 地理空间查询在球体上计算; MongoDB 使用[`WGS84`](https://docs.mongodb.com/master/reference/glossary/#term-wgs84)参考系统对 GeoJSON objects 进行地理空间查询。

## <span id="点">`Point`</span>

以下 example 指定了 GeoJSON [点](https://tools.ietf.org/html/rfc7946#section-3.1.2)：

```shell
{type:"Point",coordinates:[40,5]}
```


## <span id="线串">`LineString`</span>

以下 example 指定了GeoJSON[LineString](https://tools.ietf.org/html/rfc7946#section-3.1.4)：

```shell
{ type: "LineString", coordinates: [ [ 40, 5 ], [ 41, 6 ] ] } 
```


## <span id="多边形">多边形</span>

 多边形由一组 GeoJSON `LinearRing`坐标数组组成。这些`LinearRings`已关闭`LineStrings`。 Closed `LineStrings`至少有四个坐标对，并指定与第一个和最后一个坐标相同的位置。

 连接曲面上两个点的 line 可能包含也可能不包含在平面上连接这两个点的同一组 co-ordinates。连接曲面上两点的 line 将是一个测地线。仔细检查点以避免共享边缘的错误，以及重叠和其他类型的交叉点。

#### 单环多边形

以下 example 指定具有外环并且没有内环(或孔)的 GeoJSON `Polygon`。第一个和最后一个坐标必须 order 在 order 中才能关闭多边形：

```shell
{
  type: "Polygon",
  coordinates: [ [ [ 0 , 0 ] , [ 3 , 6 ] , [ 6 , 1 ] , [ 0 , 0  ] ] ]
}
```

对于具有单个环的多边形，环不能 self-intersect。

#### 具有多个环的多边形

对于具有多个环的多边形：

* 第一个描述的环必须是外环。

* 外圈不能 self-intersect。

* 任何内圈必须完全由外圈包含。

*   内圈不能相互交叉或重叠。内圈不能共享边缘。

以下 example 表示具有内部环的 GeoJSON 多边形：

```shell
{
    type : "Polygon",
    coordinates : [
    [ [ 0 , 0 ] , [ 3 , 6 ] , [ 6 , 1 ] , [ 0 , 0 ] ],
    [ [ 2 , 2 ] , [ 3 , 3 ] , [ 4 , 2 ] , [ 2 , 2 ] ]
    ]
  }
```

![Diagram of a Polygon with internal ring.](https://docs.mongodb.com/master/_images/index-2dsphere-polygon-with-ring.bakedsvg.svg)

## <span id="多点">多点</span>

需要的[版本](https://docs.mongodb.com/master/core/2dsphere/#dsphere-v2)

GeoJSON[MultiPoint](https://tools.ietf.org/html/rfc7946#section-3.1.3)嵌入式文档编码点列表。

```shell
{
    type: "MultiPoint",
    coordinates: [
       [ -73.9580, 40.8003 ],
       [ -73.9498, 40.7968 ],
       [ -73.9737, 40.7648 ],
       [ -73.9814, 40.7681 ]
  ]
    }
```

## <span id="id1">`MultiLineString`</span>

需要的[版本](https://docs.mongodb.com/master/core/2dsphere/#dsphere-v2)

以下 example 指定了 GeoJSON [MultiLineString](https://tools.ietf.org/html/rfc7946#section-3.1.5):

```shell
 {
  type: "MultiLineString",
      coordinates: [
         [ [ -73.96943, 40.78519 ], [ -73.96082, 40.78095 ] ],
         [ [ -73.96415, 40.79229 ], [ -73.95544, 40.78854 ] ],
         [ [ -73.97162, 40.78205 ], [ -73.96374, 40.77715 ] ],
         [ [ -73.97880, 40.77247 ], [ -73.97036, 40.76811 ] ]
           ]
      }
```

## <span id="id2">`MultiPolygon`</span>

需要的[版本](https://docs.mongodb.com/master/core/2dsphere/#dsphere-v2)

以下 example 指定了GeoJSON[MultiPolygon](https://tools.ietf.org/html/rfc7946#section-3.1.7):

```shell
{
  type: "MultiPolygon",
      coordinates: [
         [ [ [ -73.958, 40.8003 ], [ -73.9498, 40.7968 ], [ -73.9737, 40.7648 ], [ -73.9814, 40.7681 ], [ -73.958, 40.8003 ] ] ],
         [ [ [ -73.958, 40.8003 ], [ -73.9498, 40.7968 ], [ -73.9737, 40.7648 ], [ -73.958, 40.8003 ] ] ]
      ]
    }
```

## <span id="id3">`GeometryCollection`</span>

需要的[版本](https://docs.mongodb.com/master/core/2dsphere/#dsphere-v2)

以下 example store GeoJSON类型 [GeometryCollection](https://tools.ietf.org/html/rfc7946#section-3.1.8)的坐标:

```shell
{
    type: "GeometryCollection",
      geometries: [
         {
           type: "MultiPoint",
           coordinates: [
              [ -73.9580, 40.8003 ],
              [ -73.9498, 40.7968 ],
              [ -73.9737, 40.7648 ],
              [ -73.9814, 40.7681 ]
           ]
         },
         {
           type: "MultiLineString",
           coordinates: [
              [ [ -73.96943, 40.78519 ], [ -73.96082, 40.78095 ] ],
              [ [ -73.96415, 40.79229 ], [ -73.95544, 40.78854 ] ],
              [ [ -73.97162, 40.78205 ], [ -73.96374, 40.77715 ] ],
              [ [ -73.97880, 40.77247 ], [ -73.97036, 40.76811 ] ]
           ]
        }
    ]
  }
```

​    

译者：杨帅

校对：杨帅

 