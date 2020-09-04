## 查询Haystack索引

> 弃用
>
> MongoDB 4.4不支持[geoHaystack](https://docs.mongodb.com/master/core/geohaystack/)索引和[ geoSearch ](https://docs.mongodb.com/master/reference/command/geoSearch/#dbcmd.geoSearch)命令。使用[2d索引](https://docs.mongodb.com/master/reference/operator/aggregation/geoNear/#pipe._S_geoNear)或[ $geoWithin ](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin)代替。

**Haystack**索引是一种特殊的**2d**地理空间索引，优化后可以在小区域内返回结果。要创建一个haystack索引，请参见[创建一个haystack索引](https://docs.mongodb.com/master/tutorial/build-a-geohaystack-index/#geospatial-indexes-haystack-index)。

要查询一个**haystack**索引，使用[ geoSearch ](https://docs.mongodb.com/master/reference/command/geosearch/#dbcmd.geosearch)命令。您必须为[geoSearch](https://docs.mongodb.com/master/reference/command/geosearch/#dbcmd.geosearch)指定坐标和附加字段。例如，要返回示例点附近的**type**字段中值为**restaurant**的所有文档，命令如下:

```shell
db.runCommand( { geoSearch : "places" ,
                 search : { type: "restaurant" } ,
                 near : [-74, 40.74] ,
                 maxDistance : 10 } )
```

> 注意
>
> **Haystack**索引不适合查询最接近特定位置的完整文档列表。与存储桶大小相比，最近的文档可能更远。

> 请注意
>
> haystack索引目前不支持[球形查询操作](https://docs.mongodb.com/master/tutorial/calculate-distances-using-sphery-geometry -with-2d-geospatial-indexes/)。
>
> [find()](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法不能访问**haystack**索引。

