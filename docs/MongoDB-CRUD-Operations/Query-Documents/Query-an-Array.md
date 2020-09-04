
# 查询数组
本页提供使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell中的[`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)方法对数组字段进行查询操作的示例。 此页面上的示例使用**inventory**收集。 要填充**inventory**收集，请运行以下命令：

```shell
db.inventory.insertMany([
	{ item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] }, 
	{ item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
	{ item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
	{ item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },  
	{ item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] }
]);
```

## 匹配数组

要在数组上指定相等条件，请使用查询文档{**<`field`>：<`value`>**}，其中**<`value`>**是要匹配的精确数组，包括元素的顺序。

下面的示例查询所有文档，其中字段标签值是按指定顺序恰好具有两个元素**"red"** 和**"blank"**的数组：

```shell
db.inventory.find( { tags: ["red", "blank"] } )
```

相反，如果您希望找到一个同时包含元素**“ red”**和**“ blank”**的数组，而不考虑顺序或该数组中的其他元素，请使用[`$all`](https://docs.mongodb.com/master/reference/operator/query/all/#op._S_all)运算符：

```shell
db.inventory.find( { tags: { $all: ["red", "blank"] } } )
```

## 查询数组中的元素

要查询数组字段是否包含至少一个具有指定值的元素，请使用过滤器**{<`field`>：<`value`>}**，其中**<`value`>**是元素值。

以下示例查询所有文档，其中**tag**是一个包含字符串**“ red”**作为其元素之一的数组：

```shell
db.inventory.find( { tags: "red" } )
```

要在数组字段中的元素上指定条件，请在[`query filter document`](https://docs.mongodb.com/manual/core/document/#document-query-filter)中使用[`query operators`](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors) ：

```shell
{ <array field>: { <operator1>: <value1>, ... } }
```

例如，以下操作查询数组**dim_cm**包含至少一个值大于**25**的元素的所有文档。

```shell
db.inventory.find( { dim_cm: { $gt: 25 } } )
```

## 为数组元素指定多个条件

在数组元素上指定复合条件时，可以指定查询，以使单个数组元素满足这些条件，或者数组元素的任何组合均满足条件。

### 使用数组元素上的复合过滤条件查询数组

以下示例查询文档，其中**dim_cm**数组包含某种组合满足查询条件的元素； 例如，一个元素可以满足大于**15**的条件，而另一个元素可以满足小于**20**的条件，或者单个元素可以满足以下两个条件：

```shell
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )
```

### 查询满足多个条件的数组元素

使用[`$elemMatch`](https://docs.mongodb.com/master/reference/operator/query/elemMatch/#op._S_elemMatch)运算符可在数组的元素上指定多个条件，以使至少一个数组元素满足所有指定的条件。

以下示例查询在**dim_cm**数组中包含至少一个同时大于([`$gt`](https://docs.mongodb.com/master/reference/operator/query/gt/#op._S_gt))**22**和小于 ([`$lt`](https://docs.mongodb.com/master/reference/operator/query/lt/#op._S_lt)) **30**的元素的文档：

```shell
db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )
```

### 通过数组索引位置查询元素

使用[点表示法](https://docs.mongodb.com/master/reference/glossary/#term-dot-notation)，可以为元素在数组的特定索引或位置处指定查询条件。 该数组使用基于零的索引。

> **[success] Note**
>
> **使用点符号查询时，字段和嵌套字段必须在引号内。**

以下示例查询数组**dim_cm**中第二个元素大于**25**的所有文档：

```shell
db.inventory.find( { "dim_cm.1": { $gt: 25 } } )
```

### 通过数组长度查询数组

使用[`$size`](https://docs.mongodb.com/master/reference/operator/query/size/#op._S_size)运算符可按元素数量查询数组。 例如，以下选择数组**tags**具有**3**个元素的文档.

```shell
db.inventory.find( { "tags": { $size: 3 } } )
```

### 附加查询教程

有关其他查询示例，请参见：

- [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)
- [Query on Embedded/Nested Documents](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/)
- [Query an Array of Embedded Documents](https://docs.mongodb.com/manual/tutorial/query-array-of-documents/)




译者：杨帅

校对：杨帅