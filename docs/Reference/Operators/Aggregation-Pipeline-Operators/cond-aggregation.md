# [ ](#)$cond (aggregation)
[]()

在本页面

*   [定义](#definition)

*   [例子](#example)

## <span id="definition">定义</span>

**$cond**

计算一个布尔表达式以返回两个指定的返回表达式之一。

该`$cond`表达式具有以下两种语法之一：

```powershell
{ $cond: { if: <boolean-expression>, then: <true-case>, else: <false-case> } }
```

or

```powershell
{ $cond: [ <boolean-expression>, <true-case>, <false-case> ] }
```

`$cond`要求任何（`if-then-else`）一种语法的所有三个参数。

如果将`<boolean-expression>`计算结果为`true`，则 `$cond`计算并返回`<true-case>`表达式的值 。否则，`$cond`求值并返回`<false-case>`表达式的值。

参数可以是任何有效的表达式。有关表达式的更多信息，请参见 表达式。

> 也可以看看
> 
> `$switch`

## <span id="example">例子</span>

以下示例将`inventory`集合与以下文档一起使用：

```powershell
{ "_id" : 1, "item" : "abc1", qty: 300 }
{ "_id" : 2, "item" : "abc2", qty: 200 }
{ "_id" : 3, "item" : "xyz1", qty: 250 }
```

下面的聚合操作使用$cond表达式，如果qty值大于或等于250，将折扣值设置为30，如果qty值小于250，则设置为20:

```powershell
db.inventory.aggregate(
   [
      {
         $project:
           {
             item: 1,
             discount:
               {
                 $cond: { if: { $gte: [ "$qty", 250 ] }, then: 30, else: 20 }
               }
           }
      }
   ]
)
```

该操作返回以下结果：

```powershell
{ "_id" : 1, "item" : "abc1", "discount" : 30 }
{ "_id" : 2, "item" : "abc2", "discount" : 20 }
{ "_id" : 3, "item" : "xyz1", "discount" : 30 }
```

以下操作使用`$cond`表达式的数组语法， 并返回相同的结果：

```powershell
db.inventory.aggregate(
   [
      {
         $project:
           {
             item: 1,
             discount:
               {
                 $cond: [ { $gte: [ "$qty", 250 ] }, 30, 20 ]
               }
           }
      }
   ]
)
```



译者：李冠飞

校对：