## 创建Haystack索引

> 弃用
>
> MongoDB 4.4不支持[geoHaystack](https://docs.mongodb.com/master/core/geohaystack/)索引和地理搜索命令。使用[`$geoNear`](https://docs.mongodb.com/master/reference/operator/aggregation/geoNear/#pipe._S_geoNear)或[`$geoWithin`](https://docs.mongodb.com/master/reference/operator/query/geoWithin/#op._S_geoWithin) 的2d索引。

**haystack**索引必须引用两个字段:位置字段和第二个字段。第二个字段用于精确匹配。**Haystack**索引基于位置和对单个附加条件的精确匹配返回文档。这些索引不一定适合于将最近的文档返回到特定位置。

要构建一个**haystack**索引，请使用以下语法:

```shell
db.coll.createIndex( { <location field> : "geoHaystack" ,
                       <additional field> : 1 } ,
                     { bucketSize : <bucket value> } )
```

要构建**haystack**索引，必须在创建索引时指定**' bucketSize '**选项。**' bucketSize '**为**' 5 '**创建一个索引，该索引将指定经度和纬度的5个单位内的位置值分组。**“bucketSize”**还决定了索引的粒度。您可以根据数据的分布调整参数，以便通常只搜索很小的区域。bucket定义的区域可以重叠。文档可以存在于多个桶中。

> 例子
>
> 如果您有一个包含类似以下字段的文档集合:

```shell
{ _id : 100, pos: { lng : 126.9, lat : 35.2 } , type : "restaurant"}
{ _id : 200, pos: { lng : 127.5, lat : 36.1 } , type : "restaurant"}
{ _id : 300, pos: { lng : 128.0, lat : 36.7 } , type : "national park"}
```

下面的操作创建了一个带有**bucket**的**haystack**索引，该bucket将键存储在一个经度或纬度单位内。

```shell
db.places.createIndex( { pos : "geoHaystack", type : 1 } ,
                       { bucketSize : 1 } )
```

这个索引将值为**200**的**“_id”**字段存储在两个不同的存储桶中:

- 在包含**“_id”**字段值为**“100”**的文档的bucket中
- 在包含**“_id”**字段值为**“300”**的文档的bucket中

要使用haystack索引进行查询，可以使用[ geoSearch ](https://docs.mongodb.com/master/reference/command/geoSearch/#dbcmd.geoSearch) 命令。参见[查询Haystack索引](https://docs.mongodb.com/master/tutorial/query-a-geohaystack-index/#geospatial-indexes-haystack-queries).
默认情况下，使用haystack索引的查询返回50个文档。

