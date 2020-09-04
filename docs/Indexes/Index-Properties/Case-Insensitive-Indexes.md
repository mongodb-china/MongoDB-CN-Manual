## 不分大小写索引

**在本页面**

- [行为](#行为)
- [例子](#例子)
  - [创建不区分大小写的索引](#不区分)
  - [具有默认排序规则的集合中的区分大小写的索引](#默认)

*3.4版本新功能*

不区分大小写索引支持执行不考虑大小写的字符串比较的查询。

通过指定**“collation”**参数作为选项，可以使用[' db.collection.createIndex() '](https://docs.mongodb.com/master/reference/method/db.collection.createindex/# db.collection.createIndex)创建大小写不敏感索引。例如:

```powershell
db.collection.createIndex( { "key" : 1 },
                           { collation: {
                               locale : <locale>,
                               strength : <strength>
                             }
                           } )
```

要为区分大小写的索引指定排序规则，请包括:

- **locale**:指定语言规则。参见[Collation locale](https://docs.mongodb.com/master/reference/coll-locales-defaults/ # coll-languages-locales)获取可用locale列表。
- **strength**:确定比较规则。值“**1**”或“**2**”表示排序规则不区分大小写。

有关其他排序字段，请参见[collation](https://docs.mongodb.com/master/reference/collation/#coll-document-fields)。

### <span id="行为">行为</span>

使用不区分大小写的索引不会影响查询的结果，但可以提高性能;请参阅[Indexes](https://docs.mongodb.com/master/indexes/)以获得关于索引成本和收益的详细讨论。

若要使用指定排序规则的索引，查询和排序操作必须指定与索引相同的排序规则。如果集合定义了排序规则，所有查询和索引都会继承该排序规则，除非它们显式指定不同的排序规则。

### <span id="例子">例子</span>

#### <span id="不区分">创建不区分大小写的索引</span>

使用一个不分大小写指数在一组没有默认排序,创建一个索引排序和“**strength**”参数设置为“1”或“2”(见[排序](https://docs.mongodb.com/master/reference/collation/ collation-document-fields)的strength参数的详细描述)。必须在查询级别指定相同的排序规则，才能使用索引级别的排序规则。

下面的示例创建一个没有默认排序规则的集合，然后在“**type**”字段上使用大小写不敏感排序规则添加索引。

```shell
db.createCollection("fruit")

db.fruit.createIndex( { type: 1},
                      { collation: { locale: 'en', strength: 2 } } )
```

要使用索引，查询必须指定相同的排序规则。

```powershell
db.fruit.insert( [ { type: "apple" },
                   { type: "Apple" },
                   { type: "APPLE" } ] )

db.fruit.find( { type: "apple" } ) // does not use index, finds one result

db.fruit.find( { type: "apple" } ).collation( { locale: 'en', strength: 2 } )
// uses the index, finds three results

db.fruit.find( { type: "apple" } ).collation( { locale: 'en', strength: 1 } )
// does not use the index, finds three results
```

#### <span id="默认">具有默认排序规则的集合中的区分大小写的索引</span>

使用默认排序规则创建集合时，除非指定不同的排序规则，否则随后创建的所有索引都会继承该排序规则。所有没有指定不同排序规则的查询也继承默认排序规则。

下面的示例使用默认排序规则创建名为“**names**”的集合，然后在“**first_name”**字段上创建索引。

```powershell
db.createCollection("names", { collation: { locale: 'en_US', strength: 2 } } )

db.names.createIndex( { first_name: 1 } ) // inherits the default collation
```

插入少量名称:

```powershell
db.names.insert( [ { first_name: "Betsy" },
                   { first_name: "BETSY"},
                   { first_name: "betsy"} ] )
```

对该集合的查询默认情况下使用指定的排序规则，如果可能还使用索引。

```powershell
db.names.find( { first_name: "betsy" } )
// inherits the default collation: { collation: { locale: 'en_US', strength: 2 } }
// finds three results
```

上述操作使用集合的默认排序规则并查找所有三个文档。它使用' **first_name** '字段上的索引以获得更好的性能。

通过在查询中指定不同的排序规则，仍然可以对这个集合执行区分大小写的搜索:

```powershell
db.names.find( { first_name: "betsy" } ).collation( { locale: 'en_US' } )
// does not use the collection's default collation, finds one result
```

上面的操作只找到一个文档，因为它使用的排序规则没有指定**strength**值。它不使用集合的默认排序规则或索引。

