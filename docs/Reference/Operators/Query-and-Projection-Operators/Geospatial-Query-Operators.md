# [ ](#)地理空间查询运算符

[]()

在本页面

*   [运算符](#operators)


> **注意**
>
> 有关特定运算符的详细信息，包括语法和示例，请单击特定运算符以转到其参考页。

## <span id="operators">运算符</span>

### 查询选择器

| 名称                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| [`$geoIntersects`]() | 选择与GeoJSON几何形状相交的几何形状。2dsphere索引支持 `$geoIntersects`。 |
| [`$geoWithin`]()     | 选择边界GeoJSON几何内的几何。2dsphere和2D指标支持 `$geoWithin`。 |
| [`$near`]()          | 返回点附近的地理空间对象。需要地理空间索引。2dsphere和2D指标支持 `$near`。 |
| [`$nearSphere`]()    | 返回球体上某个点附近的地理空间对象。需要地理空间索引。2dsphere和2D指标支持 `$nearSphere`。 |

### 几何说明符

| 名称                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| [`$box`]()          | 使用传统坐标对来指定一个矩形框进行 `$geoWithin`查询。所述2D指数支持 `$box`。 |
| [`$center`]()       | 使用平面几何时，使用旧坐标对指定圆以进行`$geoWithin`查询。所述2D指数支持`$center`。 |
| [`$centerSphere`]() | 使用球形几何图形时，使用传统坐标对或GeoJSON格式指定一个圆 用于`$geoWithin`查询。2dsphere和 2D指标支持`$centerSphere`。 |
| [`$geometry`]()     | 为地理空间查询运算符指定GeoJSON格式的几何。                  |
| [`$maxDistance`]()  | 指定最大距离以限制`$near` 和`$nearSphere`查询的结果。2dsphere和2D指标支持 `$maxDistance`。 |
| [`$minDistance`]()  | 指定最小距离以限制`$near` 和`$nearSphere`查询的结果。`2dsphere`仅用于索引。 |
| [`$polygon`]()      | 指定用于`$geoWithin`查询的旧坐标对的多边形。2d索引支持`$center`。 |
| [`$uniqueDocs`]()   | 不推荐使用。修改`$geoWithin`和`$near`查询以确保即使文档多次匹配查询，查询也只返回一次文档。 |



译者：李冠飞

校对：