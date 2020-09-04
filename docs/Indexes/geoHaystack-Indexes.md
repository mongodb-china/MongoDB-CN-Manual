## geoHaystack索引

**在本页面**

- [行为](#行为)
- [`sparse` 属性](#属性)
- [创建`geoHaystack`索引](#索引)

**“geoHaystack”**索引是一种特殊的索引，优化后可以在小范围内返回结果。**“geoHaystack”**索引提高了使用平面几何图形查询的性能。

对于使用球形几何的查询，**2dsphere索引**比haystack索引**更好**。[2dsphere索引](https://docs.mongodb.com/master/core/2dsphere/)允许字段重新排序；`geoHaystack`索引要求第一个字段为位置字段。另外，`geoHaystack` 索引只能通过命令使用，因此总是一次返回所有结果。

### <span id="行为">行为</span>

`geoHaystack`索引从同一地理区域创建文档的“存储桶”，以提高限于该区域的查询的性能。`geoHaystack`索引中的每个存储段都包含在给定经度和纬度指定邻近范围内的所有文档。

### <span id="属性">sparse属性</span>

`geoHaystack`索引默认为[sparse](https://docs.mongodb.com/master/core/index-sparse/)，忽略[sparse: true](https://docs.mongodb.com/master/core/index-sparse/)选项。如果一个文档缺少一个`geoHaystack`索引字段(或者该字段是“null”或空数组)，MongoDB不会为该文档添加一个条目到`geoHaystack`索引中。对于插入，MongoDB插入文档，但不添加到`geoHaystack`索引。

`geoHaystack`索引包括一个`geoHaystack`索引键和一个非地理空间索引键;但是，只有`geoHaystack`索引字段决定索引是否引用文档。

#### 排序选项

`geoHaystack`索引只支持简单的二进制比较，不支持[collation](https://docs.mongodb.com/master/reference/bson-type-comparison-order/#collation)。

要在具有非简单排序规则的集合上创建`geoHaystack`索引，必须在创建索引时显式指定`{collation: {locale: "simple"}}`。

### <span id="索引">创建`geoHaystack`索引</span>

要创建`geoHaystack`索引，请参见[创建Haystack索引](https://docs.mongodb.com/master/tutorial/build-a-geohaystack-index/)。有关查询haystack索引的信息和示例，请参见[查询haystack索引](https://docs.mongodb.com/master/tutorial/query-a-geohaystack-index/)。

