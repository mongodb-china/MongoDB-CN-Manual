## 使用索引对查询结果进行排序

**在本页面**

- [使用单个字段索引排序](#单个)
- [对多个字段上排序](#多个)
- [索引使用和排序](#使用)

由于索引包含有序的记录，MongoDB可以从包含排序字段的索引中获得排序结果。MongoDB 可能使用多个索引来支持排序操作如果排序使用相同的索引作为查询谓词。

如果MongoDB不能使用一个或多个索引来获取排序顺序，MongoDB必须对数据执行阻塞排序操作。阻塞排序表示MongoDB在返回结果之前必须使用和处理所有输入文档。阻塞排序不会阻塞对集合或数据库的并发操作。

如果MongoDB要求对阻塞排序操作使用超过100 MB的系统内存，则除非查询指定[`cursor.allowDiskUse()`](https://docs.mongodb.com/master/reference/method/cursor.allowDiskUse/#cursor.allowDiskUse)（MongoDB 4.4中的New），否则MongoDB返回错误。 [`allowDiskUse()`](https://docs.mongodb.com/master/reference/method/cursor.allowDiskUse/#cursor.allowDiskUse)允许MongoDB在处理阻塞排序操作时使用磁盘上的临时文件存储超过100兆字节系统内存限制的数据。

使用索引的排序操作通常比阻塞排序具有更好的性能。有关创建索引以支持排序操作的更多信息，请参见[使用索引对查询结果排序](https://docs.mongodb.com/master/tutorial/sort-results-with-indexes/# soring-with-indexes)。

> 注意
>
> 由于在MongoDB 3.6中对数组字段排序行为的改变，当排序一个数组索引[多键索引](https://docs.mongodb.com/master/core/index-multikey/)查询计划包括一个阻塞排序阶段。新的排序行为可能会对性能产生负面影响。
>
> 在阻塞排序中，在生成输出之前，排序步骤必须使用所有输入。在非阻塞排序或索引排序中，排序步骤扫描索引以按请求的顺序生成结果。

### <span id="单个">使用单个字段索引排序</span>

如果在单个字段上有升序或降序索引，则字段上的排序操作可以是任意方向的。

例如，为一个集合`records`在字段**“a”**上创建一个升序索引:

```powershell
db.records.createIndex( { a: 1 } )
```

这个索引可以支持对**“a”**的升序排序:

```powershell
db.records.find().sort( { a: 1 } )
```

索引也可以支持以下对**“a”**的降序排序，以逆序遍历索引:

```powershell
db.records.find().sort( { a: -1 } )
```

### <span id="多个">对多个字段进行排序</span>

创建一个[复合索引](https://docs.mongodb.com/master/core/index-compound/#index-type-compound)来支持在多个字段上排序。

可以对索引的所有键或子集指定排序;但是，排序键必须按照它们在索引中出现的相同顺序列出。例如，一个索引键模式`{a: 1, b: 1}`可以支持` {a: 1, b: 1} `上的排序，但不支持` {b: 1, a: 1} `上的排序。

为一个查询使用复合索引排序,指定的排序方向所有键[`cursor.sort ()`](https://docs.mongodb.com/master/reference/method/cursor.sort/ # cursor.sort)文件必须匹配索引键模式*或*匹配索引键的反模式。例如，索引键模式`{a: 1, b: -1}`可以支持对`{a: 1, b: -1}`和`{a: -1, b: 1}`的排序，但对`{a: -1, b: -1}`或`{a: 1, b: 1}`的排序不支持。

#### 排序和索引前缀

如果排序键对应于索引键或索引前缀，MongoDB可以使用索引对查询结果排序。复合索引的**prefix**是由索引键模式开头的一个或多个键组成的子集。

例如，在**data**集合上创建一个复合索引:

```powershell
db.data.createIndex( { a:1, b: 1, c: 1, d: 1 } )
```

那么，以下是该索引的前缀:

```powershell
{ a: 1 }
{ a: 1, b: 1 }
{ a: 1, b: 1, c: 1 }
```

下面的查询和排序操作使用索引前缀对结果进行排序。这些操作不需要在内存中对结果集排序。

| 例子                                                       | 索引的前缀             |
| :--------------------------------------------------------- | :--------------------- |
| `db.data.find().sort( { a: 1 } )`                          | `{ a: 1 }`             |
| `db.data.find().sort( { a: -1 } )`                         | `{ a: 1 }`             |
| `db.data.find().sort( { a: 1, b: 1 } )`                    | `{ a: 1, b: 1 }`       |
| `db.data.find().sort( { a: -1, b: -1 } )`                  | `{ a: 1, b: 1 }`       |
| `db.data.find().sort( { a: 1, b: 1, c: 1 } )`              | `{ a: 1, b: 1, c: 1 }` |
| `db.data.find( { a: { $gt: 4 } } ).sort( { a: 1, b: 1 } )` | `{ a: 1, b: 1 }`       |

考虑下面的例子，索引的前缀键同时出现在查询谓词和排序中:

```powershell
db.data.find( { a: { $gt: 4 } } ).sort( { a: 1, b: 1 } )
```

在这种情况下，MongoDB可以使用索引来按照排序指定的顺序检索文档。如示例所示，查询谓词中的索引前缀可以与排序中的前缀不同。

#### 索引的排序和非前缀子集

索引可以支持对索引键模式的非前缀子集进行排序操作。为此，查询必须在排序键之前的所有前缀键上包含**相等**条件。

例如，集合`data`有以下索引:

```powershell
{ a: 1, b: 1, c: 1, d: 1 }
```

下面的操作可以使用索引来获取排序顺序:

| 例子                                                      | Index Prefix            |
| :-------------------------------------------------------- | :---------------------- |
| `db.data.find( { a: 5 } ).sort( { b: 1, c: 1 } )`         | `{ a: 1 , b: 1, c: 1 }` |
| `db.data.find( { b: 3, a: 4 } ).sort( { c: 1 } )`         | `{ a: 1, b: 1, c: 1 }`  |
| `db.data.find( { a: 5, b: { $lt: 3} } ).sort( { b: 1 } )` | `{ a: 1, b: 1 }`        |

如最后一个操作所示，只有排序子集之前的索引字段在查询文档中必须具有相等条件；其他索引字段可以指定其他条件。

如果查询没有在排序规范之前或重叠的索引前缀上指定相等条件，则操作将无法有效地使用索引。例如，下面的操作指定一个排序文档为`{c: 1}`，但是查询文档不包含前面索引字段**“a”**和**“b”**的相等匹配:

```powershell
db.data.find( { a: { $gt: 2 } } ).sort( { c: 1 } )
db.data.find( { c: 5 } ).sort( { c: 1 } )
```

这些操作不能有效地使用索引`{a: 1, b: 1, c: 1, d: 1}`，甚至不能使用索引来检索文档。

### <span id="使用">索引的使用和排序</span>

若要使用索引进行字符串比较，操作还必须指定相同的排序规则。也就是说，如果索引指定了不同的排序规则，则具有排序规则的索引不能支持对索引字段执行字符串比较的操作。

例如，集合`myColl `在字符串字段`category `上有一个索引，其排序区域设置为**fr** 。

```powershell
db.myColl.createIndex( { category: 1 }, { collation: { locale: "fr" } } )
```

下面的查询操作指定了与索引相同的排序规则，可以使用索引:

```powershell
db.myColl.find( { category: "cafe" } ).collation( { locale: "fr" } )
```

但是，以下查询操作，默认使用`simple`二进制排序器，不能使用索引:

```powershell
db.myColl.find( { category: "cafe" } )
```

对于索引前缀键不是字符串、数组和嵌入文档的复合索引，指定不同排序规则的操作仍然可以使用索引来支持对索引前缀键的比较。

例如，集合` myColl `在数值字段` score `和` price `以及字符串字段` category `上有一个复合索引;索引是用collation locale ' **fr** '创建的，用于字符串比较:

```powershell
db.myColl.createIndex(
   { score: 1, price: 1, category: 1 },
   { collation: { locale: "fr" } } )
```

以下使用' **simple** '二进制排序来进行字符串比较的操作可以使用索引:

```powershell
db.myColl.find( { score: 5 } ).sort( { price: 1 } )
db.myColl.find( { score: 5, price: { $gt: NumberDecimal( "10" ) } } ).sort( { price: 1 } )
```

下面的操作使用**"simple"**二进制排序对索引的`category `字段进行字符串比较，可以使用索引来完成查询的` score: 5 `部分:

```powershell
db.myColl.find( { score: 5, category: "cafe" } )
```

