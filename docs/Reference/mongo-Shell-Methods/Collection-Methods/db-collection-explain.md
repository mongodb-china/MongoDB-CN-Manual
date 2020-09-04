# [ ](#)db.collection.explain（）

[]()

在本页面

*   [定义](#definition)

*   [行为](#behavior)

*   [例子](#examples)

*   [输出](#output)

## <span id="definition">定义</span>

*   `db.collection.`  `explain` ()

返回有关以下方法的查询计划的信息：

| 从MongoDB 3.0开始                                            | 从MongoDB 3.2开始                   |
| ------------------------------------------------------------ | ----------------------------------- |
| `aggregate()`<br />`count()` <br />`find()`<br />`remove()` <br />`update()` | `distinct()`<br />`findAndModify()` |

若要使用`db.collection.explain()`，请将上述方法之一附加到`db.collection.explain()`：

```powershell
db.collection.explain().<method(...)>
```

例如，

```powershell
db.products.explain().remove( { category: "apparel" }, { justOne: true } )
```

有关更多示例，请参阅例子。有关可用于db.collection.explain()的方法列表，请参阅db.collection.explain().help()。

db.collection.explain()方法具有以下参数：

| 参数        | 类型   | 描述                                                         |
| ----------- | ------ | ------------------------------------------------------------ |
| `verbosity` | string | 可选的。指定解释输出的详细模式。该模式会影响`explain()`的行为并确定要 return 的信息量。可能的模式是：`"queryPlanner"`，`"executionStats"`和`"allPlansExecution"`。 <br/>默认模式为`"queryPlanner"`。 <br/>为了向后兼容早期版本的cursor.explain()，MongoDB 将`true`解释为`"allPlansExecution"`，将`false`解释为`"queryPlanner"`。 <br/>有关模式的更多信息，请参阅详细模式。 |

## <span id="behavior">行为</span>

### 详细模式

db.collection.explain()的行为和返回的信息量取决于`verbosity`模式。

#### queryPlanner 模式

默认情况下，db.collection.explain()在`queryPlanner`详细模式下运行。

MongoDB 运行查询优化器来为 evaluation 下的操作选择获胜计划。 db.collection.explain()返回已评估方法的queryPlanner信息。

#### executionStats 模式

MongoDB 运行查询优化器来选择获胜计划，执行获胜计划以完成，并返回描述获胜计划执行的统计数据。

对于写入操作，db.collection.explain()返回有关将执行的更新或删除操作的信息，但不会将修改应用于数据库。

db.collection.explain()返回已评估方法的queryPlanner和executionStats信息。但是，executionStats不提供被拒绝计划的查询执行信息。

#### allPlansExecution 模式

MongoDB 运行查询优化器来选择获胜计划并执行获胜计划以完成。在`"allPlansExecution"`模式中，MongoDB 返回描述获胜计划执行情况的统计信息以及计划选择期间捕获的其他候选计划的统计信息。

对于写入操作，db.collection.explain()返回有关将执行的更新或删除操作的信息，但不会将修改应用于数据库。

db.collection.explain()返回已评估方法的queryPlanner和executionStats信息。 executionStats包括获胜计划的完整查询执行信息。

如果查询优化器考虑了多个计划，executionStats信息还包括在计划选择阶段期间为获胜和被拒绝的候选计划捕获的部分执行信息。

### Explain and Write Operations

对于写操作，`db.collection.explain()`返回有关将要执行但实际上并未修改数据库的写操作的信息。

### Restrictions

在MongoDB中4.2开始，你不能运行`explain`命令/ `db.collection.explain()`在`executionStats`模式或`allPlansExecution`一个模式包含阶段。相反，您可以：`aggregation pipeline` `$out`

- 以`queryPlanner`方式运行说明
- 在`executionStats`模式或`allPlansExecution` 模式下运行说明，但没有该`$out`阶段以返回该阶段之前的`$out`阶段的信息。

### explain() Mechanics

db.collection.explain()方法包装说明命令，是 run 说明的首选方法。

`db.collection.explain().find()`类似于db.collection.find().explain()，以下 key 差异：

*   `db.collection.explain().find()`结构允许额外链接查询修饰符。有关查询修饰符的列表，请参见db.collection.explain().find().help()。
*   `db.collection.explain().find()`返回一个游标，它需要调用`.next()`或其别名`.finish()`来\_return `explain()`结果。如果在mongo shell 中以交互方式 run，mongo shell 会自动 calls `.finish()`来\_return 结果。但是，对于脚本，必须显式调用`.next()`或`.finish()`来_return 结果。有关 cursor-related 方法的列表，请参阅db.collection.explain().find().help()。

`db.collection.explain().aggregate()`相当于将说明选项传递给db.collection.aggregate()方法。

### help()

要查看db.collection.explain()，run 支持的操作列表：

```powershell
db.collection.explain().help()
```

`db.collection.explain().find()`返回一个游标，允许链接查询修饰符。要查看db.collection.explain().find()以及 cursor-related 方法支持的查询修饰符列表，润：

```powershell
db.collection.explain().find().help()
```

您可以将多个修改器链接到`db.collection.explain().find()`。对于 example，请参阅用修饰符解释 find()。

## <span id="examples">例子</span>

### queryPlanner 模式

默认情况下，db.collection.explain()在`"queryPlanner"`详细模式下运行。

以下 example 在“queryPlanner” 详细模式下运行db.collection.explain()以返回指定count()操作的查询计划信息：

```powershell
db.products.explain().count( { quantity: { $gt: 50 } } )
```

### executionStats 模式

以下 example 在“executionStats” verbosity 模式下运行db.collection.explain()以_return 指定find()操作的查询计划和执行信息：

```powershell
db.products.explain("executionStats").find(
    { quantity: { $gt: 50 }, category: "apparel" }
)
```

### allPlansExecution 模式

以下 example 在“allPlansExecution” verbosity 模式下运行db.collection.explain()。对于指定的update()操作，db.collection.explain()为所有考虑的计划返回queryPlanner和executionStats：

> **注意**
>
> 执行此解释不会修改数据，而是运行更新操作的查询谓词。对于候选计划，MongoDB 返回计划选择阶段期间捕获的执行信息。

```powershell
db.products.explain("allPlansExecution").update(
    { quantity: { $lt: 1000}, category: "apparel" },
    { $set: { reorder: true } }
)
```

### 用修饰符解释 find()

`db.collection.explain().find()` construct 允许链接查询修饰符。对于 example，以下操作使用sort()和hint()查询修饰符提供有关find()方法的信息。

```powershell
db.products.explain("executionStats").find(
    { quantity: { $gt: 50 }, category: "apparel" }
).sort( { quantity: -1 } ).hint( { category: 1, quantity: -1 } )
```

有关可用的查询修饰符列表，shell 中的 run：

```powershell
db.collection.explain().find().help()
```

### 迭代 explain().find() Return 游标

`db.collection.explain().find()`将光标返回到解释结果。如果在mongo shell 中以交互方式 run，mongo shell 将使用`.next()`方法自动迭代游标。但是，对于脚本，必须显式调用`.next()`(或其别名`.finish()`)来_return 结果：

```powershell
var explainResult = db.products.explain().find( { category: "apparel" } ).next();
```

## <span id="output">输出</span>

db.collection.explain()操作可以 return 以下信息：

*   queryPlanner，详细说明查询优化器选择的计划，列出被拒绝的计划;

*   executionStats，详细说明获胜计划的执行情况和被拒绝的计划;和

*   serverInfo，提供有关 MongoDB 实例的信息。

详细程度模式(即：`queryPlanner`，`executionStats`，`allPlansExecution`)确定结果是否包含executionStats以及executionStats是否包含计划选择期间捕获的数据。

有关输出的详细信息，请参阅解释结果。

> 对于具有 version 3.0 mongos和至少一个 2.6 mongod分片的混合 version 分片 cluster，当您在 version 3.0 mongo shell 中 run db.collection.explain()时，db.collection.explain()将使用$explain operator 重试_ret以 2.6 格式返回结果。



译者：李冠飞

校对：