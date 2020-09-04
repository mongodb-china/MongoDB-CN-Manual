## 确保索引适合RAM

**在本页面**

- [仅在RAM中保存最近值的索引](#索引)

为了实现最快的处理，请确保索引完全适合RAM，以便系统可以避免从磁盘读取索引。

要检查索引的大小，使用[ db.collection.totalIndexSize() ](https://docs.mongodb.com/master/reference/method/db.collection.totalIndexSize/#db.collection.totalIndexSize)帮助器，该帮助程序以字节为单位返回数据：

```powershell
> db.collection.totalIndexSize()
4294976499
```

上面的示例显示了一个接近4.3GB的索引大小。为了确保该索引适合RAM，您不仅必须拥有多于该数量的可用RAM，而且还必须为其余[工作集](https://docs.mongodb.com/master/reference/glossary/#term-working-set)提供RAM 。还请记住：

如果您拥有并使用多个集合，则必须考虑所有集合上所有索引的大小。索引和工作集必须能够同时装入内存。

在一些有限的情况下，索引不需要装入内存。参见[只在RAM中保存最近值的索引](https://docs.mongodb.com/master/tutorial/ensureindexes-fit-ram/#indexing-right -handed)。

也可以看看：

[`collStats`](https://docs.mongodb.com/master/reference/command/collStats/#dbcmd.collStats) 和 [`db.collection.stats()`](https://docs.mongodb.com/master/reference/method/db.collection.stats/#db.collection.stats)



### <span id="索引">仅在RAM中保存最近值的索引</span>

索引不必在所有情况下都完全适合RAM。如果索引字段的值随每次插入而增加，并且大多数查询选择最近添加的文档；那么MongoDB只需要将索引中保留最新或“最右边”值的部分保留在RAM中。这样可以有效地将索引用于读取和写入操作，并最大程度地减少支持索引所需的RAM数量。