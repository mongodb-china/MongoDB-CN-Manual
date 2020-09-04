# [ ](#)db.collection.stats（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behaviors)

*   [例子](#examples)

## <span id="definition">定义</span>

*   `db.collection.` `stats`(选项)

返回有关集合的统计信息。

该方法包括以下参数：

| 参数               | 类型     | 描述                                                         |
| ------------------ | -------- | ------------------------------------------------------------ |
| `scale`            | number   | 可选的。输出中使用的比例显示项目的大小。默认情况下，输出显示`bytes`中的大小。要显示千字节而不是字节，请指定`1024` value `1024`。 <br/>如果您指定非整数比例因子，则MongoDB将使用指定因子的整数部分。例如，如果将比例因子指定为`1023.999`，则MongoDB将使用`1023`该比例因子。<br />从4.2版开始，输出包括`scaleFactor` 用于缩放大小值的输出。 |
| `indexDetails`     | boolean  | 可选的。如果为`true`，则`db.collection.stats()`除收集统计信息外，还返回 `index details`<br />仅适用于WiredTiger存储引擎。<br />默认为`false`。 |
| `indexDetailsKey`  | document | 可选的。如果`indexDetails`是`true`，则可以使用`indexDetailsKey`通过指定索引 key 规范来过滤索引详细信息。只返回与`indexDetailsKey`完全匹配的索引。 <br/>如果找不到匹配项，indexDetails将显示所有索引的统计信息。 <br/>使用getIndexes()发现索引键。你不能将`indexDetailsKey`与`indexDetailsName`一起使用。 |
| `indexDetailsName` | string   | 可选的。如果`indexDetails`是`true`，则可以使用`indexDetailsName`通过指定索引`name`来过滤索引详细信息。只返回与`indexDetailsName`完全匹配的索引名称。 <br/>如果找不到匹配项，indexDetails将显示所有索引的统计信息。 <br/>使用getIndexes()来发现索引名称。你不能将`indexDetailsName`与`indexDetailsField`一起使用。 |

仅指定`scale`因素，MongoDB支持旧格式：

```powershell
db.collection.stats(<number>)
```

| <br />   |                                                              |
| -------- | ------------------------------------------------------------ |
| 返回值： | 包含有关指定集合的统计信息的文档。请参阅`collStats`以获取返回统计信息的细分。 |

该`db.collection.stats()`方法提供了围绕数据库命令的包装`collStats`。

## <span id="behaviors">行为</span>

### 缩放大小

除非度量标准名称（例如`"bytes currently in the cache"` ）另外指定，与大小相关的值以字节为单位显示，可以按比例覆盖。

比例因子将受影响的大小值四舍五入为整数。

### 存储引擎

根据存储引擎，返回的数据可能不同。有关字段的详细信息，请参阅输出详细信息。

### 意外关机后的准确性

使用Wired Tiger存储引擎不正常关闭mongod后，db.collection.stats()报告的计数和大小统计信息可能不准确。

偏移量取决于在最后检查站和不干净关闭之间执行的 insert，update 或 delete 操作的数量。检查点通常每 60 秒发生一次。但是，使用 non-default --syncdelay设置运行mongod实例可能会有更多或更少的检查点。

在mongod上的每个集合上运行验证以在不正常关闭后恢复正确的统计信息。

### 索引过滤器行为

使用`indexDetailsKey`或`indexDetailsName`过滤`indexDetails`将仅_return 单个匹配的索引。如果未找到确切的 match，`indexDetails`将显示有关集合的所有索引的信息。

`indexDetailsKey`字段采用以下形式的文档：

```powershell
{ '<string>' : <value>, '<string>' : <value>, ... }
```

其中`<string>`是索引的字段，`<value>`是索引的方向，或特殊索引类型，如`text`或`2dsphere`。有关索引类型的完整列表，请参见索引类型。

### 意外停机和计数

对于使用WiredTiger存储引擎的 MongoDB 实例，在不正常关闭后，大小和计数的统计信息可能会被collStats，dbStats，计数报告最多 1000 个文档。要恢复集合的正确统计信息，请在集合上运行 run 验证。

### 进行中索引

从MongoDB 4.2开始，`db.collection.stats`包括有关当前正在构建的索引的信息。有关详细信息，请参见：

- `collStats.nindexes`
- `collStats.indexDetails`
- `collStats.indexBuilds`
- `collStats.totalIndexSize`
- `collStats.indexSizes`

## <span id="examples">例子</span>

> **注意**
>
> 您可以在入门指南中找到用于这些示例的集合数据

### 基本统计查询

以下操作返回`restaurants`集合上的统计信息：

```powershell
db.restaurants.stats()
```

操作返回：

```powershell
{
    "ns" : "guidebook.restaurants",
    "count" : 25359,
    "size" : 10630398,
    "avgObjSize" : 419,
    "storageSize" : 4104192
    "capped" : false,
    "wiredTiger" : {
        "metadata" : {
            "formatVersion" : 1
        },
        "creationString" : "allocation_size=4KB,app_metadata=(formatVersion=1),block_allocation=best,block_compressor=snappy,cache_resident=0,checksum=on,colgroups=,collator=,columns=,dictionary=0,encryption=(keyid=,name=),exclusive=0,extractor=,format=btree,huffman_key=,huffman_value=,immutable=0,internal_item_max=0,internal_key_max=0,internal_key_truncate=,internal_page_max=4KB,key_format=q,key_gap=10,leaf_item_max=0,leaf_key_max=0,leaf_page_max=32KB,leaf_value_max=64MB,log=(enabled=),lsm=(auto_throttle=,bloom=,bloom_bit_count=16,bloom_config=,bloom_hash_count=8,bloom_oldest=0,chunk_count_limit=0,chunk_max=5GB,chunk_size=10MB,merge_max=15,merge_min=0),memory_page_max=10m,os_cache_dirty_max=0,os_cache_max=0,prefix_compression=0,prefix_compression_min=4,source=,split_deepen_min_child=0,split_deepen_per_child=0,split_pct=90,type=file,value_format=u",
        "type" : "file",
        "uri" : "statistics:table:collection-2-7253336746667145592",
        "LSM" : {
            "bloom filters in the LSM tree" : 0,
            "bloom filter false positives" : 0,
            "bloom filter hits" : 0,
            "bloom filter misses" : 0,
            "bloom filter pages evicted from cache" : 0,
            "bloom filter pages read into cache" : 0,
            "total size of bloom filters" : 0,
            "sleep for LSM checkpoint throttle" : 0,
            "chunks in the LSM tree" : 0,
            "highest merge generation in the LSM tree" : 0,
            "queries that could have benefited from a Bloom filter that did not exist" : 0,
            "sleep for LSM merge throttle" : 0
        },
        "block-manager" : {
            "file allocation unit size" : 4096,
            "blocks allocated" : 338,
            "checkpoint size" : 4096000,
            "allocations requiring file extension" : 338,
            "blocks freed" : 0,
            "file magic number" : 120897,
            "file major version number" : 1,
            "minor version number" : 0,
            "file bytes available for reuse" : 0,
            "file size in bytes" : 4104192
        },
        "btree" : {
            "btree checkpoint generation" : 15,
            "column-store variable-size deleted values" : 0,
            "column-store fixed-size leaf pages" : 0,
            "column-store internal pages" : 0,
            "column-store variable-size leaf pages" : 0,
            "pages rewritten by compaction" : 0,
            "number of key/value pairs" : 0,
            "fixed-record size" : 0,
            "maximum tree depth" : 3,
            "maximum internal page key size" : 368,
            "maximum internal page size" : 4096,
            "maximum leaf page key size" : 3276,
            "maximum leaf page size" : 32768,
            "maximum leaf page value size" : 67108864,
            "overflow pages" : 0,
            "row-store internal pages" : 0,
            "row-store leaf pages" : 0
        },
        "cache" : {
            "bytes read into cache" : 9309503,
            "bytes written from cache" : 10817368,
            "checkpoint blocked page eviction" : 0,
            "unmodified pages evicted" : 0,
            "page split during eviction deepened the tree" : 0,
            "modified pages evicted" : 1,
            "data source pages selected for eviction unable to be evicted" : 0,
            "hazard pointer blocked page eviction" : 0,
            "internal pages evicted" : 0,
            "pages split during eviction" : 1,
            "in-memory page splits" : 1,
            "overflow values cached in memory" : 0,
            "pages read into cache" : 287,
            "overflow pages read into cache" : 0,
            "pages written from cache" : 337
        },
        "compression" : {
            "raw compression call failed, no additional data available" : 0,
            "raw compression call failed, additional data available" : 0,
            "raw compression call succeeded" : 0,
            "compressed pages read" : 287,
            "compressed pages written" : 333,
            "page written failed to compress" : 0,
            "page written was too small to compress" : 4
        },
        "cursor" : {
            "create calls" : 1,
            "insert calls" : 25359,
            "bulk-loaded cursor-insert calls" : 0,
            "cursor-insert key and value bytes inserted" : 10697901,
            "next calls" : 76085,
            "prev calls" : 1,
            "remove calls" : 0,
            "cursor-remove key bytes removed" : 0,
            "reset calls" : 25959,
            "restarted searches" : 0,
            "search calls" : 0,
            "search near calls" : 594,
            "update calls" : 0,
            "cursor-update value bytes updated" : 0
        },
        "reconciliation" : {
            "dictionary matches" : 0,
            "internal page multi-block writes" : 1,
            "leaf page multi-block writes" : 2,
            "maximum blocks required for a page" : 47,
            "internal-page overflow keys" : 0,
            "leaf-page overflow keys" : 0,
            "overflow values written" : 0,
            "pages deleted" : 0,
            "page checksum matches" : 0,
            "page reconciliation calls" : 4,
            "page reconciliation calls for eviction" : 1,
            "leaf page key bytes discarded using prefix compression" : 0,
            "internal page key bytes discarded using suffix compression" : 333
        },
        "session" : {
            "object compaction" : 0,
            "open cursor count" : 1
        },
        "transaction" : {
            "update conflicts" : 0
        }
    },
    "nindexes" : 4,
    "totalIndexSize" : 626688,
    "indexSizes" : {
        "_id_" : 217088,
        "borough_1_cuisine_1" : 139264,
        "cuisine_1" : 131072,
        "borough_1_address.zipcode_1" : 139264
    },
    "ok" : 1
}
```

由于统计数据未给出比例参数，因此所有大小值都在`bytes`中。

### 带有比例的统计查询

以下操作通过指定`scale`的`scale`来更改从`bytes`到`kilobytes`的数据比例：

```powershell
db.restaurants.stats( { scale : 1024 } )
```

操作返回：

```powershell
{
    "ns" : "guidebook.restaurants",
    "count" : 25359,
    "size" : 10381,
    "avgObjSize" : 419,
    "storageSize" : 4008,
    "capped" : false,
    "wiredTiger" : {
        ...
    },
    "nindexes" : 4,
    "totalIndexSize" : 612,
    "indexSizes" : {
        "_id_" : 212,
        "borough_1_cuisine_1" : 136,
        "cuisine_1" : 128,
        "borough_1_address.zipcode_1" : 136
    },
    "ok" : 1
}
```

### 带索引详细信息的统计查找

以下操作将创建一个`indexDetails`文档，其中包含与集合中每个索引相关的信息：

```powershell
db.restaurant.stats( { indexDetails : true } )
```

操作返回：

```powershell
{
    "ns" : "guidebook.restaurants",
    "count" : 25359,
    "size" : 10630398,
    "avgObjSize" : 419,
    "storageSize" : 4104192,
    "capped" : false,
    "wiredTiger" : {
        ...
    },
    "nindexes" : 4,
    "indexDetails" : {
        "_id_" : {
            "metadata" : {
                "formatVersion" : 6,
                "infoObj" : "{ "v" : 1, "key" : { "_id" : 1 }, "name" : "_id_", "ns" : "blogs.posts" }"
             },
            "creationString" : "allocation_size=4KB,app_metadata=(formatVersion=6,infoObj={ "v" : 1, "key" : { "_id" : 1 }, "name" : "_id_", "ns" : "blogs.posts" }),block_allocation=best,block_compressor=,cache_resident=0,checksum=on,colgroups=,collator=,columns=,dictionary=0,encryption=(keyid=,name=),exclusive=0,extractor=,format=btree,huffman_key=,huffman_value=,immutable=0,internal_item_max=0,internal_key_max=0,internal_key_truncate=,internal_page_max=16k,key_format=u,key_gap=10,leaf_item_max=0,leaf_key_max=0,leaf_page_max=16k,leaf_value_max=0,log=(enabled=),lsm=(auto_throttle=,bloom=,bloom_bit_count=16,bloom_config=,bloom_hash_count=8,bloom_oldest=0,chunk_count_limit=0,chunk_max=5GB,chunk_size=10MB,merge_max=15,merge_min=0),memory_page_max=5MB,os_cache_dirty_max=0,os_cache_max=0,prefix_compression=true,prefix_compression_min=4,source=,split_deepen_min_child=0,split_deepen_per_child=0,split_pct=75,type=file,value_format=u",
            "type" : "file",
            "uri" : "statistics:table:index-3-7253336746667145592",
            "LSM" : {
                ...
            },
            "block-manager" : {
                ...
            },
            "btree" : {
                ...
            },
            "cache" : {
               ...
            },
            "compression" : {
               ...
            },
            "cursor" : {
               ...
            },
            "reconciliation" : {
               ...
            },
            "session" : {
               ...
            },
            "transaction" : {
               ...
            }
        },
        "borough_1_cuisine_1" : {
            "metadata" : {
                "formatVersion" : 6,
                "infoObj" : "{ "v" : 1, "key" : { "borough" : 1, "cuisine" : 1 }, "name" : "borough_1_cuisine_1", "ns" : "blogs.posts" }"
        },
        "creationString" : "allocation_size=4KB,app_metadata=(formatVersion=6,infoObj={ "v" : 1, "key" : { "borough" : 1, "cuisine" : 1 }, "name" : "borough_1_cuisine_1", "ns" : "blogs.posts" }),block_allocation=best,block_compressor=,cache_resident=0,checksum=on,colgroups=,collator=,columns=,dictionary=0,encryption=(keyid=,name=),exclusive=0,extractor=,format=btree,huffman_key=,huffman_value=,immutable=0,internal_item_max=0,internal_key_max=0,internal_key_truncate=,internal_page_max=16k,key_format=u,key_gap=10,leaf_item_max=0,leaf_key_max=0,leaf_page_max=16k,leaf_value_max=0,log=(enabled=),lsm=(auto_throttle=,bloom=,bloom_bit_count=16,bloom_config=,bloom_hash_count=8,bloom_oldest=0,chunk_count_limit=0,chunk_max=5GB,chunk_size=10MB,merge_max=15,merge_min=0),memory_page_max=5MB,os_cache_dirty_max=0,os_cache_max=0,prefix_compression=true,prefix_compression_min=4,source=,split_deepen_min_child=0,split_deepen_per_child=0,split_pct=75,type=file,value_format=u",
        "type" : "file",
        "uri" : "statistics:table:index-4-7253336746667145592",
        "LSM" : {
            ...
        },
        "block-manager" : {
            ...
        },
        "btree" : {
            ...
        },
        "cache" : {
            ...
        },
        "compression" : {
            ...
        },
        "cursor" : {
            ...
        },
        "reconciliation" : {
            ...
        },
        "session" : {
            "object compaction" : 0,
            "open cursor count" : 0
        },
        "transaction" : {
            "update conflicts" : 0
        }
    },
    "cuisine_1" : {
        "metadata" : {
            "formatVersion" : 6,
            "infoObj" : "{ "v" : 1, "key" : { "cuisine" : 1 }, "name" : "cuisine_1", "ns" : "blogs.posts" }"
        },
        "creationString" : "allocation_size=4KB,app_metadata=(formatVersion=6,infoObj={ "v" : 1, "key" : { "cuisine" : 1 }, "name" : "cuisine_1", "ns" : "blogs.posts" }),block_allocation=best,block_compressor=,cache_resident=0,checksum=on,colgroups=,collator=,columns=,dictionary=0,encryption=(keyid=,name=),exclusive=0,extractor=,format=btree,huffman_key=,huffman_value=,immutable=0,internal_item_max=0,internal_key_max=0,internal_key_truncate=,internal_page_max=16k,key_format=u,key_gap=10,leaf_item_max=0,leaf_key_max=0,leaf_page_max=16k,leaf_value_max=0,log=(enabled=),lsm=(auto_throttle=,bloom=,bloom_bit_count=16,bloom_config=,bloom_hash_count=8,bloom_oldest=0,chunk_count_limit=0,chunk_max=5GB,chunk_size=10MB,merge_max=15,merge_min=0),memory_page_max=5MB,os_cache_dirty_max=0,os_cache_max=0,prefix_compression=true,prefix_compression_min=4,source=,split_deepen_min_child=0,split_deepen_per_child=0,split_pct=75,type=file,value_format=u",
        "type" : "file",
        "uri" : "statistics:table:index-5-7253336746667145592",
        "LSM" : {
            ...
        },
        "block-manager" : {
            ...
        },
        "btree" : {
            ...
        },
        "cache" : {
            ...
        },
        "compression" : {
            ...
        },
        "cursor" : {
            ...
        },
        "reconciliation" : {
            ...
        },
        "session" : {
            ...
        },
        "transaction" : {
            ...
        }
    },
    "borough_1_address.zipcode_1" : {
        "metadata" : {
            "formatVersion" : 6,
            "infoObj" : "{ "v" : 1, "key" : { "borough" : 1, "address.zipcode" : 1 }, "name" : "borough_1_address.zipcode_1", "ns" : "blogs.posts" }"
             },
            "creationString" : "allocation_size=4KB,app_metadata=(formatVersion=6,infoObj={ "v" : 1, "key" : { "borough" : 1, "address.zipcode" : 1 }, "name" : "borough_1_address.zipcode_1", "ns" : "blogs.posts" }),block_allocation=best,block_compressor=,cache_resident=0,checksum=on,colgroups=,collator=,columns=,dictionary=0,encryption=(keyid=,name=),exclusive=0,extractor=,format=btree,huffman_key=,huffman_value=,immutable=0,internal_item_max=0,internal_key_max=0,internal_key_truncate=,internal_page_max=16k,key_format=u,key_gap=10,leaf_item_max=0,leaf_key_max=0,leaf_page_max=16k,leaf_value_max=0,log=(enabled=),lsm=(auto_throttle=,bloom=,bloom_bit_count=16,bloom_config=,bloom_hash_count=8,bloom_oldest=0,chunk_count_limit=0,chunk_max=5GB,chunk_size=10MB,merge_max=15,merge_min=0),memory_page_max=5MB,os_cache_dirty_max=0,os_cache_max=0,prefix_compression=true,prefix_compression_min=4,source=,split_deepen_min_child=0,split_deepen_per_child=0,split_pct=75,type=file,value_format=u",
            "type" : "file",
            "uri" : "statistics:table:index-6-7253336746667145592",
            "LSM" : {
               ...
            },
            "block-manager" : {
               ...
            },
             "btree" : {
                ...
            },
            "cache" : {
               ...
            },
            "compression" : {
               ...
            },
            "cursor" : {
               ...
            },
            "reconciliation" : {
               ...
            },
            "session" : {
               ...
            },
            "transaction" : {
                ...
            }
        }
    },
    "totalIndexSize" : 626688,
    "indexSizes" : {
        "_id_" : 217088,
        "borough_1_cuisine_1" : 139264,
        "cuisine_1" : 131072,
        "borough_1_address.zipcode_1" : 139264
    },
    "ok" : 1
}
```

### 带有过滤索引详细信息的统计信息查找

要过滤indexDetails字段中的索引，可以使用`indexDetailsKey`选项指定索引键，也可以使用`indexDetailsName`指定索引 name。要发现集合的索引键和名称，请使用db.collection.getIndexes()。

给定以下索引：

```powershell
{
    "ns" : "guidebook.restaurants",
    "v" : 1,
    "key" : {
        "borough" : 1,
        "cuisine" : 1
    },
    "name" : "borough_1_cuisine_1"
}
```

以下操作将`indexDetails`文档过滤为`indexDetailsKey`文档定义的单个索引。

```powershell
db.restaurants.stats(
    {
        'indexDetails' : true,
        'indexDetailsKey' :
        {
            'borough' : 1,
            'cuisine' : 1
        }
    }
)
```

以下操作将`indexDetails`文档过滤为`indexDetailsName`文档定义的单个索引。

```powershell
db.restaurants.stats(
    {
        'indexDetails' : true,
        'indexDetailsName' : 'borough_1_cuisine_1'
    }
)
```

两个操作都会 return 相同的输出：

```powershell
{
    "ns" : "blogs.restaurants",
    "count" : 25359,
    "size" : 10630398,
    "avgObjSize" : 419,
    "storageSize" : 4104192,
    "capped" : false,
    "wiredTiger" : {
        ...
    },
    "nindexes" : 4,
    "indexDetails" : {
        "borough_1_cuisine_1" : {
            "metadata" : {
                "formatVersion" : 6,
                "infoObj" : "{ "v" : 1, "key" : { "borough" : 1, "cuisine" : 1 }, "name" : "borough_1_cuisine_1", "ns" : "blogs.posts" }"
             },
            "creationString" : "allocation_size=4KB,app_metadata=(formatVersion=6,infoObj={ "v" : 1, "key" : { "borough" : 1, "cuisine" : 1 }, "name" : "borough_1_cuisine_1", "ns" : "blogs.posts" }),block_allocation=best,block_compressor=,cache_resident=0,checksum=on,colgroups=,collator=,columns=,dictionary=0,encryption=(keyid=,name=),exclusive=0,extractor=,format=btree,huffman_key=,huffman_value=,immutable=0,internal_item_max=0,internal_key_max=0,internal_key_truncate=,internal_page_max=16k,key_format=u,key_gap=10,leaf_item_max=0,leaf_key_max=0,leaf_page_max=16k,leaf_value_max=0,log=(enabled=),lsm=(auto_throttle=,bloom=,bloom_bit_count=16,bloom_config=,bloom_hash_count=8,bloom_oldest=0,chunk_count_limit=0,chunk_max=5GB,chunk_size=10MB,merge_max=15,merge_min=0),memory_page_max=5MB,os_cache_dirty_max=0,os_cache_max=0,prefix_compression=true,prefix_compression_min=4,source=,split_deepen_min_child=0,split_deepen_per_child=0,split_pct=75,type=file,value_format=u",
            "type" : "file",
            "uri" : "statistics:table:index-4-7253336746667145592",
            "LSM" : {
               ...
            },
            "block-manager" : {
               ...
            },
            "btree" : {
               ...
            },
            "cache" : {
               ...
            },
            "compression" : {
               ...
            },
            "cursor" : {
               ...
            },
            "reconciliation" : {
               ...
            },
            "session" : {
               ...
            },
            "transaction" : {
               ...
            }
        }
    },
    "totalIndexSize" : 626688,
    "indexSizes" : {
        "_id_" : 217088,
        "borough_1_cuisine_1" : 139264,
        "cuisine_1" : 131072,
        "borough_1_address.zipcode_1" : 139264
    },
    "ok" : 1
}
```

有关输出的说明，请参阅输出细节。

> **也可以看看**
>
> $collStats



译者：李冠飞

校对：