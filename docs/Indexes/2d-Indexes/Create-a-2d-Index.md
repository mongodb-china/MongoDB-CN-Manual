## 创建一个`2d`索引

**在本页面**

- [定义`2d`索引的位置范围](#位置范围)
- [定义`2d`索引的位置精度](#位置精度)

要构建一个地理空间的 **2d **索引，使用[` db.collection.createIndex() `](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法并指定 **2d** 。使用以下语法:

```powershell
db.<collection>.createIndex( { <location field> : "2d" ,
                               <additional field> : <value> } ,
                             { <index-specification options> } )
```

 **2d** 索引使用以下可选的索引规范选项:

```powershell
{ min : <lower bound> , max : <upper bound> ,
  bits : <bit precision> }
```

### <span id="位置范围">定义`2d`索引的位置范围</span>

默认情况下， **2d** 索引假定经度和纬度，边界为[-180 , 180)。如果文档包含的坐标数据超出了指定的范围，MongoDB将返回一个错误。

> 重要
>
> 默认边界允许应用程序插入纬度大于90或小于-90的无效文档。对于这些无效点的地理空间查询行为没有定义。

在 **2d** 索引上，你可以改变位置范围。

您可以创建一个 **2d** 地理空间索引，其中包含默认位置范围之外的位置范围。在创建索引时使用' **min** '和' **max** '选项。使用以下语法:

```powershell
db.collection.createIndex( { <location field> : "2d" } ,
                           { min : <lower bound> , max : <upper bound> } )
```

### <span id="位置精度">定义“2d”索引的位置精度</span>

默认情况下，传统坐标对的“2d”索引使用26位精度，使用默认的-180到180的范围，相当于2英尺或60厘米的精度。精度由用于存储位置数据的[geohash](https://docs.mongodb.com/master/reference/glossary/#term-geohash)值的位大小来度量。您可以配置精度最高为32位的地理空间索引。

索引精度不影响查询精度。实际的网格坐标总是在最终的查询处理中使用。降低精度的优点是插入操作的处理开销更低，占用的空间更少。更高精度的一个优点是查询扫描索引的较小部分以返回结果。

若要配置默认位置以外的位置精度，请在创建索引时使用 **bits** 选项。使用下面的语法:

```powershell
db.<collection>.createIndex( {<location field> : "<index type>"} ,
                             { bits : <bit precision> } )
```

有关geohash值的内部信息，请参见[计算2d索引的geohash值](https://docs.mongodb.com/master/core/geospatial-indexes/#geospatial-indexes-geohash)。

