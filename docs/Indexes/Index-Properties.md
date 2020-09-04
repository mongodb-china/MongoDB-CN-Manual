## 索引特性
### 索引的unique特性
开启[unique](https:_docs.mongodb.com_manual_core_index-unique)选项，索引对应字段具有唯一性，对应字段拒绝重复值。除唯一性约束外，在功能上，索引的unique特性可与其他特性交替使用。
### 索引的Partial特性
3.2版本新特性

特性相关选项设置后，将仅索引集合中满足指定筛选表达式的文档。对集合中的文档子集创建索引，设置了partial特性的索引将占用更低的存储，并降低mongodb创建索引和维护索引的性能开销。

在功能上，索引的**Partial**特性是sparse特性的超集，当一个索引同时拥有两种特性时，以Partial特性优先。

### 索引的Sparse特性
索引的 [sparse](https:_docs.mongodb.com_manual_core_index-sparse) 特性确保只对存在索引字段的文档创建索引。创建索引时将会跳过那些没有对应字段值的文档。

你或许可以将**sparse**选项和**unique**选项结合使用，以防止索引字段插入重复值，并对对应索引字段缺失的文档不创建索引，提升数据库效率。

### 索引的TTL特性
索引的[TTL](https:_docs.mongodb.com_manual_core_index-ttl)特性，允许MongoDB在一定时间后自动从集合中移除文档。这非常适合某些类型的信息，例如：机器生成的事件数据、日志和会话信息，这些信息只需要在数据库中保留有限的时间。

有关实现说明，请参见：[Expire Data from Collections by Setting TTL](https:_docs.mongodb.com_manual_tutorial_expire-data)

译者：程哲欣
