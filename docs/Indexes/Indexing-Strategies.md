## 索引策略

应用程序的最佳索引必须考虑许多因素，包括预期的查询类型、读写比率和系统上的空闲内存量。

在开发索引策略时，您应该对应用程序的查询有深刻的理解。在构建索引之前，要映射出将要运行的查询类型，以便构建引用这些字段的索引。索引会带来性能成本，但与频繁查询大型数据集的成本相比，它更值得。考虑应用程序中每个查询的相对频率，以及该查询是否适合使用索引。

设计索引的最佳总体策略是使用与您将在生产环境中运行的数据集相似的数据集来分析各种索引配置，以查看哪种配置性能最佳。检查为您的集合创建的当前索引，以确保它们支持您当前和计划中的查询。如果不再使用索引，请删除该索引。

通常，MongoDB只使用一个索引来完成大多数查询。然而，一个[`$or`](https://docs.mongodb.com/master/reference/operator/query/or/#op._S_or)查询的每个子句可能使用一个不同的索引，此外，MongoDB可以使用多个索引的[交集](https://docs.mongodb.com/master/core/index-intersection/)。

下面的文档介绍了索引策略:

- [创建索引支持查询](./Indexing-Strategies/Create-Indexes-to-Support-Your-Queries.md)

  当索引包含查询扫描的所有字段时，索引就支持查询。创建支持查询的索引可以极大地提高查询性能。

- [使用索引对查询结果进行排序](./Indexing-Strategies/Use-Indexes-to-Sort-Query-Results.md)

  为了支持高效查询，在指定索引字段的顺序和排序顺序时，请使用这里的策略。

- [确保索引适合RAM](./Indexing-Strategies/Ensure-Indexes-Fit-in-RAM.md)

  当索引适合RAM时，系统可以避免从磁盘读取索引，从而获得最快的处理速度。

- [创建以确保选择性的查询](./Indexing-Strategies/Create-Queries-that-Ensure-Selectivity.md)

  选择性是指查询使用索引缩小结果的能力。选择性允许MongoDB在与完成查询相关的大部分工作中使用索引。