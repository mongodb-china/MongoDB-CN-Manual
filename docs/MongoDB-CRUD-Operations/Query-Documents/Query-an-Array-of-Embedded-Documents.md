# 查询嵌入式文档数组
本页提供使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中的[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)方法对嵌套文档数组进行查询操作的示例。 此页面上的示例使用**inventory**收集。 要填充**inventory**收集，请运行以下命令：

```shell
db.inventory.insertMany( [
	{ item: "journal", instock: [ { warehouse: "A", qty: 5 }, { warehouse: "C", qty: 15 } ] },  
	{ item: "notebook", instock: [ { warehouse: "C", qty: 5 } ] },
	{ item: "paper", instock: [ { warehouse: "A", qty: 60 }, { warehouse: "B", qty: 15 } ] },
	{ item: "planner", instock: [ { warehouse: "A", qty: 40 }, { warehouse: "B", qty: 5 } ] }, 
	{ item: "postcard", instock: [ { warehouse: "B", qty: 15 }, { warehouse: "C", qty: 35 } ] }
]);
```

## 查询嵌套在数组中的文档

下面的示例选择库存数组中的元素与指定文档匹配的所有文档：

```shell
db.inventory.find( { "instock": { warehouse: "A", qty: 5 } } )
```

整个嵌入式/嵌套文档上的相等匹配要求与指定文档（包括字段顺序）完全匹配。 例如，以下查询与**inventory**中的任何文档都不匹配：

```shell
db.inventory.find( { "instock": { qty: 5, warehouse: "A" } } )
```

## 在文档数组中的字段上指定查询条件

### 在嵌入文档数组中的字段上指定查询条件

如果您不知道嵌套在数组中的文档的索引位置，请使用点(.)和嵌套文档中的字段名称来连接数组字段的名称。

下面的示例选择所有**inventory**数组中包含至少一个嵌入式文档的嵌入式文档，这些文档包含值小于或等于**20**的字段**qty**：

```shell
db.inventory.find( { 'instock.qty': { $lte: 20 } } )
```

### 使用数组索引来查询嵌入式文档中的字段

使用[点表示法](https://docs.mongodb.com/master/reference/glossary/#term-dot-notation)，可以为文档中特定索引或数组位置的字段指定查询条件。 该数组使用基于零的索引。

> **[success] Note**
>
> 使用点表示法查询时，字段和索引必须在引号内。

下面的示例选择所有库存文件，其中库存数组的第一个元素是包含值小于或等于20的字段qty的文档：

下面的例子选择了所有**instock**数组的第一个元素是一个包含值小于或等于**20**的字段**qty**的文档:

```shell
db.inventory.find( { 'instock.0.qty': { $lte: 20 } } )
```

## 为文档数组指定多个条件

在嵌套在文档数组中的多个字段上指定条件时，可以指定查询，以使单个文档满足这些条件，或者数组中文档的任何组合(包括单个文档)都满足条件。

### 单个嵌套文档在嵌套字段上满足多个查询条件

使用[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch) 运算符可在一组嵌入式文档上指定多个条件，以使至少一个嵌入式文档满足所有指定条件。

下面的示例查询**instock**数组中至少有一个嵌入式文档的文档，这些文档包含数量等于5的字段和数量等于A的字段仓库：

```shell
db.inventory.find( { "instock": { $elemMatch: { qty: 5, warehouse: "A" } } } )
```

下面的示例查询**instock**数组中至少有一个嵌入式文档的文档，该嵌入式文档的**qty**字段大于10且小于或等于20：

```shell
db.inventory.find( { "instock": { $elemMatch: { qty: { $gt: 10, $lte: 20 } } } } )
```

### 元素组合满足标准

如果数组字段上的复合查询条件未使用[$ [`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch) 运算符，则查询将选择其数组包含满足条件的元素的任意组合的那些文档。

例如，以下查询匹配文档，其中嵌套在**instock**阵列中的任何文档的数量字段大于10，并且阵列中任何文档（但不一定是同一嵌入文档）的数量字段小于或等于20：

```shell
db.inventory.find( { "instock.qty": { $gt: 10,  $lte: 20 } } )
```

以下示例查询**instock**数组中至少一个包含数量等于**5**的嵌入式文档和至少一个包含等于**A**的字段仓库的嵌入式文档（但不一定是同一嵌入式文档）的文档：

```shell
db.inventory.find( { "instock.qty": 5, "instock.warehouse": "A" } )
```

## 附加查询教程

有关其他查询示例，请参见：

- [Query an Array](https://docs.mongodb.com/manual/tutorial/query-arrays/)
- [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)
- [Query on Embedded/Nested Documents](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/)



译者：杨帅

校对：杨帅