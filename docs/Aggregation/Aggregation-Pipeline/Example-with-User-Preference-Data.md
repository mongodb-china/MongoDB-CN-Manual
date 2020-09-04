# [ ](#)使用用户首选项数据进行聚合

[]()

在本页面

*   [数据模型](#data-model)

*   [规范化和排序文档](#normalize-and-sort-documents)

*   [返回按月加入订单的用户名](#return-usernames-ordered-by-join-month)

*   [返回每月的联接总数](#return-total-number-of-joins-per-month)

*   [Return 五个 Common“喜欢”](#return-the-five-most-common-likes)

[]()

## <span id="data-model">数据模型</span>

考虑一个假设的体育俱乐部，其数据库包含一个`users`集合，用于跟踪用户的加入日期，运动偏好，并将这些数据存储在类似于以下内容的文档中：

```powershell
{
    _id : “jane“,
    joined : ISODate(“2011-03-02“),
    likes : [“golf“, “racquetball“]
}
{
    _id : “joe“,
    joined : ISODate(“2012-07-02“),
    likes : [“tennis“, “golf“, “swimming“]
}
```

[]()

## <span id="normalize-and-sort-documents">规范化和排序文档</span>

以下操作以大写和字母 order 返回用户名。聚合包括`users`集合中所有文档的用户名。您可以这样做以规范化用户名以进行处理。

```powershell
db.users.aggregate([
    { $project : { name:{$toUpper:“$_id“} , _id:0 } },
    { $sort : { name : 1 } }
])
```

`users`集合中的所有文档都通过管道传递，该管道包含以下操作：

* [$project](reference-operator-aggregation-project.html#pipe._S_project) 操作：
  * 创建一个名为`name`的新字段。

  * 使用[$toUpper](reference-operator-aggregation-toUpper.html#exp._S_toUpper) operator 将`_id`的 value 转换为大写。然后[$project](reference-operator-aggregation-project.html#pipe._S_project)创建一个名为`name`的新字段来保存此 value。

  * 抑制`id`字段。除非明确禁止，否则[$project](reference-operator-aggregation-project.html#pipe._S_project)将默认通过`_id`字段。

*    operator 按`name`字段对结果进行排序。

聚合的结果类似于以下内容：

```powershell
{
    "name" : "JANE"
},
{
    "name" : "JILL"
},
{
    "name" : "JOE"
}
```

[]()
    

## <span id="return-usernames-ordered-by-join-month">返回按月加入订单的用户名</span>

以下聚合操作返回按其加入的月份排序的用户名。这种聚合可以帮助生成会员续订通知。

```powershell
db.users.aggregate([
    { $project :
        {
            month_joined : { $month : “$joined“ },
            name : “$_id“,
            _id : 0
        }
    },
    { $sort : { month_joined : 1 } }
])
```

管道通过以下操作传递`users`集合中的所有文档：

* [$project](reference-operator-aggregation-project.html#pipe._S_project) operator：

  * 创建两个新字段：`month_joined`和`name`。

  * 从结果中抑制`id`。除非明确禁止，否则[aggregate()](reference-method-db.collection.aggregate.html#db.collection.aggregate)方法包含`_id`。

* [$month](reference-operator-aggregation-month.html#exp._S_month) operator 将`joined`字段的值转换为月份的 integer 表示。然后[$project](reference-operator-aggregation-project.html#pipe._S_project) operator 将这些值分配给`month_joined`字段。

*   [$sort](reference-operator-aggregation-sort.html#pipe._S_sort) operator 按`month_joined`字段对结果进行排序。

该操作返回类似于以下内容的结果：

```powershell
{
    “month_joined“ : 1,
    “name“ : “ruth“
},
{
    “month_joined“ : 1,
    “name“ : “harold“
},
{
    “month_joined“ : 1,
    “name“ : “kate“
},
{
    “month_joined“ : 2,
    “name“ : “jill“
}
```

[]()

## <span id="return-total-number-of-joins-per-month">返回每月的联接总数</span>

以下操作显示了一年中每个月加入的人数。您可以将此汇总数据用于招聘和营销策略。

```powershell
db.users.aggregate([
    { $project : { month_joined : { $month : “$joined“ } } } ,
    { $group : { _id : {month_joined:“$month_joined“} , number : { $sum : 1 } } },
    { $sort : { “_id.month_joined“ : 1 } }
])
```

管道通过以下操作传递`users`集合中的所有文档：

* [$project](reference-operator-aggregation-project.html#pipe._S_project) operator 创建一个名为`month_joined`的新字段。

* [$month](reference-operator-aggregation-month.html#exp._S_month) operator 将`joined`字段的值转换为月份的 integer 表示。然后[$project](reference-operator-aggregation-project.html#pipe._S_project) operator 将值分配给`month_joined`字段。

* [$group](reference-operator-aggregation-group.html#pipe._S_group) operator 收集具有给定`month_joined` value 的所有文档，并计算该 value 的文档数量。具体来说，对于每个唯一 value，[$group](reference-operator-aggregation-group.html#pipe._S_group)创建一个包含两个字段的新“per-month”文档：

  * `_id`，包含带有`month_joined`字段及其 value 的嵌套文档。

  * `number`，这是一个生成的字段。对于包含给定`month_joined` value 的每个文档，[$sum](reference-operator-aggregation-sum.html#grp._S_sum) operator 将此字段递增 1。

*   [$sort](reference-operator-aggregation-sort.html#pipe._S_sort) operator 根据`month_joined`字段的内容对[$group](reference-operator-aggregation-group.html#pipe._S_group)创建的文档进行排序。

此聚合操作的结果类似于以下内容：

```powershell
{
    “_id“ : {
        “month_joined“ : 1
    },
    “number“ : 3
},
{
    “_id“ : {
        “month_joined“ : 2
    },
    “number“ : 9
},
{
    “_id“ : {
        “month_joined“ : 3
    },
    “number“ : 5
}
```

[]()

## <span id="return-the-five-most-common-likes">Return 五个 Common“喜欢”</span>

以下聚合收集数据集中前五个最“喜欢”的活动。这种分析有助于规划和未来发展。

```powershell
db.users.aggregate([
    { $unwind : “$likes“ },
    { $group : { _id : “$likes“ , number : { $sum : 1 } } },
    { $sort : { number : -1 } },
    { $limit : 5 }
])
```

管道从`users`集合中的所有文档开始，并通过以下操作传递这些文档：

*   [$unwind](reference-operator-aggregation-unwind.html#pipe._S_unwind) operator 分隔`likes` array 中的每个 value，并为 array 中的每个元素创建源文档的新 version。
    
> **[success] 例子**
>
> 给出来自用户集合的以下文档：
>
> ```powershell
> {
> _id : "jane",
> joined : ISODate("2011-03-02"),
> likes : ["golf", "racquetball"]
> }
> ```
>
> `$unwind`运算符将创建下列文件：
>
> ```powershell
> {
>  _id : “jane“,
>  joined : ISODate(“2011-03-02“),
>  likes : “golf“
> }
> {
>  _id : “jane“,
>  joined : ISODate(“2011-03-02“),
>  likes : “racquetball“
> }
> ```

*   [$group](reference-operator-aggregation-group.html#pipe._S_group) operator 收集`likes`字段具有相同 value 的所有文档，并计算每个分组。有了这些信息，[$group](reference-operator-aggregation-group.html#pipe._S_group)创建了一个包含两个字段的新文档：

    *   `_id`，其中包含`likes` value。
    *   `number`，这是一个生成的字段。对于包含给定`likes` value 的每个文档，[$sum](reference-operator-aggregation-sum.html#grp._S_sum) operator 将此字段递增 1。
*   [$sort](reference-operator-aggregation-sort.html#pipe._S_sort) operator 按字段在 reverse order 中对这些文档进行排序。
*   [$limit](reference-operator-aggregation-limit.html#pipe._S_limit) operator 仅包含前 5 个结果文档。

聚合的结果类似于以下内容：

```powershell
{
    “_id“ : “golf“,
    “number“ : 33
},
{
    “_id“ : “racquetball“,
    “number“ : 31
},
{
    “_id“ : “swimming“,
    “number“ : 24
},
{
    “_id“ : “handball“,
    “number“ : 19
},
{
    “_id“ : “tennis“,
    “number“ : 18
}
```



译者：李冠飞

校对：李冠飞