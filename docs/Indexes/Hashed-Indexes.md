## Hashed 索引
**在本页面**

- [Hashing 函数](#hashing)
- [创建Hashed索引](#创建)
- [注意事项](#注意)

Hashed索引使用索引字段值的hashes来维护条目。

Hashed索引支持使用hashes的分片键进行[分片](https://docs.mongodb.com/manual/sharding/)。[基于Hashed的分片](https://docs.mongodb.com/manual/core/hashed-sharding/#sharding-hashed-sharding)使用字段的散列索引作为分片键，以便跨分片集群对数据进行分区。

使用hashed的分片键对集合进行分片会导致数据分布更加随机。有关更多详细信息，请参见[Hashed分片](https://docs.mongodb.com/manual/core/hashed-sharding/#sharding-hashed-sharding)。

### <span id="hashing">Hashing 函数</span>
Hashed索引使用hashing函数来计算索引字段值的哈希。hashing函数折叠嵌入式文档并计算整个值的hash，但不支持多键（即数组）索引。

> 提示：
> 
> MongoDB在解析使用已排序索引的查询时自动计算hashed值。应用程序不需要计算hashes。

### <span id="创建">创建Hashed索引</span>
要创建hashed索引，请指定 **hashed** 作为索引键的值，如下例所示:

```powershell
db.collection.createIndex( { _id: "hashed" } )
```
### <span id="注意">注意事项</span>
MongoDB支持任何单个字段的 **hashed** 索引。hashing函数折叠嵌入的文档并计算整个值的hash值，但不支持多键(即.数组)索引。

您不能创建具有`hashed`索引字段的复合索引，也不能在索引上指定唯一约束`hashed`；但是，您可以`hashed`在同一字段上创建索引和升序/降序（即非哈希）索引：MongoDB将对范围查询使用标量索引。

#### 2<sup>53</sup> 限制

> 注意
>
> MongoDB `hashed`索引在散列之前将浮点数截断为64位整数。例如，`hashed`指数将存储用于持有的值的字段的值相同`2.3`，`2.2`和`2.9`。为避免冲突，请不要`hashed`对无法可靠转换为64位整数（然后再返回到浮点数）的浮点数使用索引。MongoDB `hashed`索引不支持大于2<sup>53</sup>的浮点值。
>
> 要查看键的hashed值是多少，请参阅[convertShardKeyToHashed()](https://docs.mongodb.com/manual/reference/method/convertShardKeyToHashed/#convertShardKeyToHashed)。

查看键对应的hashed请查看[`convertShardKeyToHashed()`](https://docs.mongodb.com/manual/reference/method/convertShardKeyToHashed/#convertShardKeyToHashed)。
#### PowerPC 和2<sup>63</sup>
对于[hashed索引](https://docs.mongodb.com/manual/core/index-hashed/#)，MongoDB 4.2确保PowerPC上浮点值2<sup>63</sup>的hashed值与其他平台一致。

尽管不支持字段上可能包含大于2<sup>53</sup>的浮点值的[hashed索引](https://docs.mongodb.com/manual/core/index-hashed/#)，但客户端仍然可以在索引字段值为2<sup>63</sup>的地方插入文档。

```powershell
db.adminCommand("listDatabases").databases.forEach(function(d){
   let mdb = db.getSiblingDB(d.name);
   mdb.getCollectionInfos({ type: "collection" }).forEach(function(c){
      let currentCollection = mdb.getCollection(c.name);
      currentCollection.getIndexes().forEach(function(idx){
        let idxValues = Object.values(Object.assign({}, idx.key));
        if (idxValues.includes("hashed")) {
          print("Hashed index: " + idx.name + " on " + idx.ns);
          printjson(idx);
        };
      });
   });
});
```
要检查索引字段是否包含值2<sup>63</sup>，对集合和索引字段执行以下操作:

- 如果一个collection中的索引字段数据内容仅是数值，不存在任何文档：
```powershell
// substitute the actual collection name for <collection>
// substitute the actual indexed field name for <indexfield>

db.<collection>.find( { <indexfield>: Math.pow(2,63) } );
```

- 如果一个collection中的索引字段是文档(或者数值)，可以执行：
```powershell
// substitute the actual collection name for <collection>
// substitute the actual indexed field name for <indexfield>
db.<collection>.find({
   $where: function() {
       function findVal(obj, val) {
           if (obj === val)
               return true;
           for (const child in obj) {
               if (findVal(obj[child], val)) {
                   return true;
               }
           }
           return false;
       }
       return findVal(this.<indexfield>, Math.pow(2, 63));
   }
})
```

译者：程哲欣
