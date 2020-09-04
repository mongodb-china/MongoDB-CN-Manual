# 文本索引

**在本页面**

- [概述](#概述)
- [版本](#版本)
- [创建文本索引](#创建)
- [不区分大小写](#不区分)
- [变音符号不敏感](#不敏感)
- [标记化分隔符](#分隔符)
- [索引条目](#条目)
- [支持的语言和停用词](#支持)
- [`sparse` 属性](#sparse)
- [限制条件](#限制)
- [存储要求和性能成本](#成本)
- [文字搜索支持](#搜索)

> MONGODB地图搜索
>
> [Atlas Search](https://docs.atlas.mongodb.com/atlas-search)可以很容易地在MongoDB数据上构建快速、基于相关性的搜索功能。在MongoDB Atlas上试试吧，这是我们的完全托管数据库服务。

## <span id="概述">概述</span>

MongoDB提供[文本索引](https://docs.mongodb.com/master/core/index-text/#index-feature-text)以支持对字符串内容的文本搜索查询。`text`索引可以包含任何值为字符串或字符串元素数组的字段。

## <span id="版本">版本</span>

| 文本索引版本 | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| 版本3        | MongoDB引入了`text`索引的第3版。版本3是`text`在MongoDB 3.2和更高版本中创建的索引的默认版本。 |
| 版本2        | MongoDB 2.6引入了`text`索引的版本2 。版本2是`text`在MongoDB 2.6和3.0系列中创建的索引的默认版本。 |
| 版本1        | MongoDB 2.4引入了`text`索引的版本1 。MongoDB 2.4仅支持版本`1`。 |

要覆盖默认版本并指定不同的版本，在创建索引时包括选项**{"textIndexVersion":` <version>`}**。

## <span id="创建">创建文本索引</span>

> **[success] 重要**
>
> 一个集合最多可以有**一个** `text`索引。

若要创建`text`索引，请使用 [`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法。若要索引包含字符串或字符串元素数组的字段，请包含该字段并在索引文档中指定字符串字面量`“text”`，如下例所示:

```powershell
db.reviews.createIndex( { comments: "text" } )
```

您可以为索引建立多个字段的`text`索引。以下示例`text`在字段`subject`和 `comments`上创建索引：

```powershell
db.reviews.createIndex(
   {
     subject: "text",
     comments: "text"
   }
 )
```

[复合索引](https://docs.mongodb.com/master/core/index-compound/)可以包含文本索引键和升序/降序索引键。有关更多信息，请参见[复合索引](https://docs.mongodb.com/master/core/index-compound/)。

为了删除`text`索引，请使用索引名称。有关更多信息，请参见[使用索引名称删除文本索引](https://docs.mongodb.com/master/tutorial/avoid-text-index-name-limit/#drop-text-index)。

### 指定权重

对于文本索引，索引字段的权重表示该字段相对于其他索引字段在文本搜索分数方面的重要性。

对于文档中的每个索引字段，MongoDB将匹配的数量乘以权重并对结果进行求和。然后，MongoDB使用这个总和计算文档的分数。有关按文本分数返回和排序的详细信息，请参阅[`$meta`](https://docs.mongodb.com/master/reference/operator/aggregation/meta/#proj._S_meta)操作符。

索引字段的默认权重为1。要调整索引字段的权重，请在[`db.collection.createIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.createIndex/#db.collection.createIndex)方法中包含权重选项。

有关使用权重控制文本搜索结果的更多信息，请参见[使用权重控制搜索结果](https://docs.mongodb.com/master/tutorial/control-results-of-text-search/)。

### 通配符文本索引

> **[succress] 注意**
>
> 通配符文本索引不同于[通配符索引](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-index-core)。通配符索引不支持使用[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)操作符的查询。
>
> 尽管通配符文本索引和[通配符索引](https://docs.mongodb.com/master/core/index-wildcard/#wildcard-index-core)共享通配符`$**`字段模式，但它们是不同的索引类型。仅通配符文本索引支持[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)运算符。

在多个字段上创建文本索引时，还可以使用通配符说明符(`$**`)。通过通配符文本索引，MongoDB为集合中每个文档包含字符串数据的每个字段建立索引。下面的示例使用通配符创建一个文本索引:

```powershell
db.collection.createIndex( { "$**": "text" } )
```

该索引允许对所有具有字符串内容的字段进行文本搜索。如果不清楚在文本索引中包含哪些字段或用于特殊查询，那么这种索引对于高度非结构化数据非常有用。

通配符文本索引是多个字段上的文本索引。因此，您可以在创建索引期间为特定字段分配权重，以控制结果的排序。有关使用权重控制文本搜索结果的详细信息，请参见 [使用权重控制搜索结果](https://docs.mongodb.com/master/tutorial/control-results-of-text-search/)。

通配符文本索引(与所有文本索引一样)可以是复合索引的一部分。例如，下面在字段`a`以及通配符上创建一个复合索引:

```powershell
db.collection.createIndex( { a: 1, "$**": "text" } )
```

与所有[复合文本索引](https://docs.mongodb.com/master/core/index-text/#text-index-compound)一样，由于`a`位于文本索引键之前，为了使用该索引执行[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)搜索，查询谓词必须包含一个相等匹配条件`a`。有关复合文本索引的信息，请参见[复合文本索引](https://docs.mongodb.com/master/core/index-text/#text-index-compound)。

## <span id="不区分">不区分大小写</span>

*在版本3.2中更改*

版本3文本索引支持常用的`C`语言、简单的`S`语言，对于土耳其语言，支持[Unicode 8.0字符数据库大小写](http://www.unicode.org/Public/8.0.0/ucd/CaseFolding.txt)折叠中指定的特殊**T**大小写折叠。

此案的大小写扩展文本索引包括字符不区分大小写的区分标志,如 `é`和`É`,从非拉丁字母和字符,如“И”和“и”西里尔字母。

文本索引的版本3也不支持[变音符号](https://docs.mongodb.com/master/core/index-text/#text-index-diacritic-insensitivity)。因此，索引也不区分 `é`, `É`, `e`, and `E`.

以前版本的文本索引只对[`A-z`]不区分大小写;例如，只对非变音符拉丁字符不区分大小写。对于所有其他字符，早期版本的文本索引将它们视为不同的字符。

## <span id="不敏感">变音符号不敏感</span>

*在版本3.2中更改*

在版本3中，`text`索引不区分音素。即，索引不包含变音符号和它们的未标记的对应，如字符区分`é`，`ê`和 `e`。更具体地说，文本索引去除[Unicode 8.0字符数据库道具列表](http://www.unicode.org/Public/8.0.0/ucd/PropList.txt)中分类为变音符号的字符。

`text`索引的第3版对带有变音符号的字符也不[区分大小写](https://docs.mongodb.com/master/core/index-text/#text-index-case-insensitivity)。这样，索引也没有区分之间`é`，`É`，`e`，和`E`。

`text`索引的早期版本将带变音符号的字符视为不同的字符。

## <span id="分隔符">标记化分隔符</span>

*在版本3.2中更改*

对于符号化，第3版`text`索引使用下分类的分隔符`Dash`，`Hyphen`，`Pattern_Syntax`， `Quotation_Mark`，`Terminal_Punctuation`，和`White_Space`中 [的Unicode 8.0字符数据库道具列表](http://www.unicode.org/Public/8.0.0/ucd/PropList.txt)。

在[Unicode 8.0字符数据库Prop列表](http://www.unicode.org/Public/8.0.0/ucd/PropList.txt)中，版本3文本索引使用分界符分类在破折号、连字符、`Pattern_Syntax`、`Quotation_Mark`、`Terminal_Punctuation`和`White_Space`中。

例如，如果给定的一个字符串，该索引对待，和空格作为分隔符。`"Il a dit qu'il «était le meilleur joueur du monde»"``text``«``»`

例如，如果给定一个字符串**“Il a dit qu'il«etait le meilleur joueur du monde»”**，文本索引将`«,»`和空格作为分隔符。

该指数治疗的早期版本`«`作为术语的一部分 `"«était"`，and`»`作为长期的一部分`"monde»"`。

## <span id="条目">索引条目</span>

文本索引对索引项的索引字段中的术语进行标记和词根处理。文本索引在集合中每个文档的每个索引字段中为每个唯一的词根项存储一个索引项。索引使用简单的[特定语言](https://docs.mongodb.com/master/core/index-text/#text-index-supported-languages)的后缀词干。

## <span id="支持">支持的语言和停用词</span>

MongoDB支持多种语言的文本搜索。`text`指数下降特定语言的停用词（如英语，`the`，`an`， `a`，`and`，等）和使用简单的语言特定的后缀而产生。有关支持的语言的列表，请参见[文本搜索语言](https://docs.mongodb.com/master/reference/text-search-languages/#text-search-languages)。

如果您将语言值指定为`"none"`，则`text`索引将使用简单的标记化，不包含停止词列表和词干分析。

要为文本索引指定一种语言，请参见 [为文本索引指定语言](https://docs.mongodb.com/master/tutorial/specify-language-for-text-index/)。

## <span id="sparse">`sparse`属性</span>

`text`索引总是[`sparse`](https://docs.mongodb.com/master/core/index-sparse/)并且忽略 [`sparse`](https://docs.mongodb.com/master/core/index-sparse/)选项。如果文档缺少`text`索引字段（或者该字段是`null`或为空数组），则MongoDB不会将文档条目添加到`text`索引中。对于插入，MongoDB会插入文档，但不会添加到`text`索引中。

对于包含`text`索引键和其他类型的键的复合索引，只有`text`索引字段才能确定索引是否引用文档。其他键不能确定索引是否引用文档。

## <span id="限制">限制条件</span>

### 每个集合有一个文本索引

一个集合最多可以有**一个** `text`索引。

### 文字搜索和提示

如果查询包含[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)查询表达式，则不能使用[`hint()`](https://docs.mongodb.com/master/reference/method/cursor.hint/#cursor.hint)。

### 文本索引和排序

排序操作无法从`text`索引获得排序顺序，即使从[复合文本索引](https://docs.mongodb.com/master/core/index-text/#text-index-compound)也无法获得排序顺序；即排序操作不能使用文本索引中的顺序。

### 复合索引

[复合索引](https://docs.mongodb.com/master/core/index-compound/)可以包含文本索引键和升序/降序索引键。但是，这些复合索引有以下限制:

- 复合文本索引不能包含任何其他特殊索引类型，例如[多键](https://docs.mongodb.com/master/core/index-multikey/#index-type-multi-key)或[地理空间](https://docs.mongodb.com/master/geospatial-queries/#index-feature-geospatial)索引字段。
- 如果复合文本索引在文本索引键之前包含键，那么要执行[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)搜索，查询谓词必须包含前面键的相等匹配条件。
- 在创建复合文本索引时，必须在索引规范文档中邻接列出所有文本索引键。

另请参见[文本索引和排序](https://docs.mongodb.com/master/core/index-text/#text-index-and-sort)。

有关复合文本索引的示例，请参见[限制扫描的条目数](https://docs.mongodb.com/master/tutorial/limit-number-of-items-scanned-for-text-search/)。

### 删除文本索引

要删除`text`索引，请将索引*名称*传递给 [`db.collection.dropIndex()`](https://docs.mongodb.com/master/reference/method/db.collection.dropIndex/#db.collection.dropIndex)方法。要获取索引的名称，请运行该[`db.collection.getIndexes()`](https://docs.mongodb.com/master/reference/method/db.collection.getIndexes/#db.collection.getIndexes)方法。

有关`text`索引的默认命名方案以及覆盖默认名称的信息，请参见[指定文本索引的名称](https://docs.mongodb.com/master/tutorial/avoid-text-index-name-limit/)。

### 排序选项

`text`索引仅支持简单的二进制比较，不支持[排序](https://docs.mongodb.com/master/reference/bson-type-comparison-order/#collation)。

要在具有非简单排序规则的集合上创建文本索引，必须在创建索引时显式指定`{collation: {locale: "simple"}}`。

## <span id="成本">存储要求和性能成本</span>

文本索引有以下存储要求和性能成本:

- `text`索引可以很大。对于每个插入的文档，每个索引字段中的每个唯一后词形词都包含一个索引条目。
- 构建`text`索引与构建大型多键索引非常相似，并且比在相同数据上构建简单的有序(标量)索引要花更长的时间。
- 在`text`现有集合上建立较大索引时，请确保对打开文件描述符的限制足够高。请参阅[建议的设置](https://docs.mongodb.com/master/reference/ulimit/)。
- `text` 索引会影响插入吞吐量，因为MongoDB必须在每个新源文档的每个索引字段中为每个唯一的词干词添加一个索引条目。
- 此外，`text`索引不存储短语或有关文档中单词接近度的信息。结果，当整个集合放入RAM中时，短语查询将更有效地运行。

## <span id="搜索">文本搜索支持</span>

文本索引支持[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)查询操作。有关文本搜索的示例，请参见[`$text引用页面`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text)。有关聚合管道中的[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 操作示例，请参见[聚合管道中的文本搜索](https://docs.mongodb.com/master/tutorial/text-search-in-aggregation/)。



译者：杨帅