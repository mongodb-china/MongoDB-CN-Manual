# [ ](#)执行增量 Map-Reduce

[]()

在本页面

*   [数据设置](#data-setup)

*   [当前集合的初始 Map-Reduce](#initial-map-reduce-of-current-collection)

*   [后续增量 Map-Reduce](#subsequent-incremental-map-reduce)

Map-reduce 操作可以处理复杂的聚合任务。要执行 map-reduce 操作，MongoDB 提供[MapReduce]()命令，并在[mongo]() shell 中提供[db.collection.mapReduce()]() wrapper 方法。

如果 map-reduce 数据集不断增长，您可能希望执行增量 map-reduce 而不是每个 time 对整个数据集执行 map-reduce 操作。

执行增量 map-reduce：

*   在当前集合上运行 map-reduce job 并将结果输出到单独的集合。

*   如果有更多数据要进行 process，run 后续 map-reduce job：

    *   `query`参数指定仅匹配新文档的条件。

    *   `out`参数，指定将新结果合并到现有输出集合中的`reduce`操作。

请考虑以下 example，其中您在`sessions`集合上安排 map-reduce 操作，以在每天结束时运行 run。

[]()

## <span id="data-setup">数据设置</span>

`sessions`集合包含 log 用户每天会话的文档，例如：

```powershell
db.sessions.save( { userid: "a", ts: ISODate('2011-11-03 14:17:00'), length: 95 } );
db.sessions.save( { userid: "b", ts: ISODate('2011-11-03 14:23:00'), length: 110 } );
db.sessions.save( { userid: "c", ts: ISODate('2011-11-03 15:02:00'), length: 120 } );
db.sessions.save( { userid: "d", ts: ISODate('2011-11-03 16:45:00'), length: 45 } );

db.sessions.save( { userid: "a", ts: ISODate('2011-11-04 11:05:00'), length: 105 } );
db.sessions.save( { userid: "b", ts: ISODate('2011-11-04 13:14:00'), length: 120 } );
db.sessions.save( { userid: "c", ts: ISODate('2011-11-04 17:00:00'), length: 130 } );
db.sessions.save( { userid: "d", ts: ISODate('2011-11-04 15:37:00'), length: 65 } );
```

[]()

## <span id="initial-map-reduce-of-current-collection">当前集合的初始 Map-Reduce</span>

运行第一个 map-reduce 操作如下：

*   定义 map function _将`userid`映射到包含字段`userid`，`total_time`，`count`和`avg_time`的 object：
    
    ```powershell
    var mapFunction = function() {
        var key = this.userid;
        var value = {
            userid: this.userid,
            total_time: this.length,
            count: 1,
            avg_time: 0
        };
        emit( key, value );
    };
    ```
    
*   使用两个 arguments `key`和`values`定义相应的 reduce function 以计算总 time 和计数。 `key`对应于`userid`，`values`是 array，其元素对应于映射到`mapFunction`中`userid`的各个 object。
    
    ```powershell
    var reduceFunction = function(key, values) {
        var reducedObject = {
            userid: key,
            total_time: 0,
            count:0,
            avg_time:0
        };
        
        values.forEach( function(value) {
            reducedObject.total_time += value.total_time;
            reducedObject.count += value.count;
        });
        return reducedObject;
    };
    ```
    
*   使用两个 arguments `key`和`reducedValue`定义 finalize function。 function 修改`reducedValue`文档以添加另一个字段`average`并返回修改后的文档。
    
    ```powershell
    var finalizeFunction = function (key, reducedValue) {
        if (reducedValue.count > 0)
            reducedValue.avg_time = reducedValue.total_time / reducedValue.count;
        
        return reducedValue;
    };
    ```
    
* 使用`mapFunction`，`reduceFunction`和`finalizeFunction`函数在`session`集合上执行 map-reduce。将结果输出到集合`session_stat`。如果`session_stat`集合已存在，则操作将替换内容：

  ```powershell
  db.sessions.mapReduce( mapFunction,
      reduceFunction,
      {
          out: "session_stat",
          finalize: finalizeFunction
      }
  )
  ```


* 查询`session_stats`集合以验证结果：

  ```powershell
  db.session_stats.find().sort( { _id: 1 } )
  ```

  该操作返回以下文档：

  ```powershell
  { "_id" : "a", "value" : { "total_time" : 200, "count" : 2, "avg_time" : 100 } }
  { "_id" : "b", "value" : { "total_time" : 230, "count" : 2, "avg_time" : 115 } }
  { "_id" : "c", "value" : { "total_time" : 250, "count" : 2, "avg_time" : 125 } }
  { "_id" : "d", "value" : { "total_time" : 110, "count" : 2, "avg_time" : 55 } }
  ```

[]()

## <span id="subsequent-incremental-map-reduce">后续增量 Map-Reduce</span>

之后，随着`sessions`集合的增长，您可以运行其他 map-reduce 操作。对于 example，将新文档添加到`sessions`集合：

```powershell
db.sessions.save( { userid: "a", ts: ISODate('2011-11-05 14:17:00'), length: 100 } );
db.sessions.save( { userid: "b", ts: ISODate('2011-11-05 14:23:00'), length: 115 } );
db.sessions.save( { userid: "c", ts: ISODate('2011-11-05 15:02:00'), length: 125 } );
db.sessions.save( { userid: "d", ts: ISODate('2011-11-05 16:45:00'), length: 55 } );
```

最终，对`usersessions`集合执行增量map-reduce ，但使用该`query`字段仅选择新文档。将结果输出到collection `session_stats`，但是`reduce`将内容与增量map-reduce的结果进行比较：

```powershell
db.usersessions.mapReduce(
   mapFunction,
   reduceFunction,
   {
     query: { ts: { $gte: ISODate('2020-03-05 00:00:00') } },
     out: { reduce: "session_stats" },
     finalize: finalizeFunction
   }
);
```

查询`session_stats`集合以验证结果：

```powershell
db.session_stats.find().sort( { _id: 1 } )
```

该操作返回以下文档：

```powershell
{ "_id" : "a", "value" : { "total_time" : 330, "count" : 3, "avg_time" : 110 } }
{ "_id" : "b", "value" : { "total_time" : 270, "count" : 3, "avg_time" : 90 } }
{ "_id" : "c", "value" : { "total_time" : 360, "count" : 3, "avg_time" : 120 } }
{ "_id" : "d", "value" : { "total_time" : 210, "count" : 3, "avg_time" : 70 } }
```

## 聚合替代

前提条件：将集合设置为原始状态：

```powershell
db.usersessions.drop();

db.usersessions.insertMany([
   { userid: "a", start: ISODate('2020-03-03 14:17:00'), length: 95 },
   { userid: "b", start: ISODate('2020-03-03 14:23:00'), length: 110 },
   { userid: "c", start: ISODate('2020-03-03 15:02:00'), length: 120 },
   { userid: "d", start: ISODate('2020-03-03 16:45:00'), length: 45 },
   { userid: "a", start: ISODate('2020-03-04 11:05:00'), length: 105 },
   { userid: "b", start: ISODate('2020-03-04 13:14:00'), length: 120 },
   { userid: "c", start: ISODate('2020-03-04 17:00:00'), length: 130 },
   { userid: "d", start: ISODate('2020-03-04 15:37:00'), length: 65 }
])
```

使用可用的聚合管道运算符，您可以重写map-reduce示例，而无需定义自定义函数：

```powershell
db.usersessions.aggregate([
   { $group: { _id: "$userid", total_time: { $sum: "$length" }, count: { $sum: 1 }, avg_time: { $avg: "$length" } } },
   { $project: { value: { total_time: "$total_time", count: "$count", avg_time: "$avg_time" } } },
   { $merge: {
      into: "session_stats_agg",
      whenMatched: [ { $set: {
         "value.total_time": { $add: [ "$value.total_time", "$$new.value.total_time" ] },
         "value.count": { $add: [ "$value.count", "$$new.value.count" ] },
         "value.avg": { $divide: [ { $add: [ "$value.total_time", "$$new.value.total_time" ] },  { $add: [ "$value.count", "$$new.value.count" ] } ] }
      } } ],
      whenNotMatched: "insert"
   }}
])
```

1. 通过`userid`[`$group`]()，得出：

   - `total_time`使用`$sum`操作
   - `count`使用`$sum`操作
   - `avg_time`使用[`$avg`]()操作

   该操作返回以下文档：

   ```powershell
   { "_id" : "c", "total_time" : 250, "count" : 2, "avg_time" : 125 }
   { "_id" : "d", "total_time" : 110, "count" : 2, "avg_time" : 55 }
   { "_id" : "a", "total_time" : 200, "count" : 2, "avg_time" : 100 }
   { "_id" : "b", "total_time" : 230, "count" : 2, "avg_time" : 115 }
   ```

2. 该[`$project`]()阶段调整输出文档的形状以反映map-reduce的输出，该输出具有两个字段`_id`和 `value`。如果不需要镜像`_id`and `value`结构，则该阶段是可选的 。

   ```powershell
   { "_id" : "a", "value" : { "total_time" : 200, "count" : 2, "avg_time" : 100 } }
   { "_id" : "d", "value" : { "total_time" : 110, "count" : 2, "avg_time" : 55 } }
   { "_id" : "b", "value" : { "total_time" : 230, "count" : 2, "avg_time" : 115 } }
   { "_id" : "c", "value" : { "total_time" : 250, "count" : 2, "avg_time" : 125 } }
   ```

3. 该[`$merge`]()阶段将结果输出到 `session_stats_agg`集合。如果现有文档`_id`与新结果相同，则该操作将应用指定的管道，以根据结果和现有文档计算total_time，count和avg_time。如果是相同的，现有的文档`_id`中`session_stats_agg`，操作插入文档。

4. 查询`session_stats_agg`集合以验证结果：

   ```powershell
   db.session_stats_agg.find().sort( { _id: 1 } )
   ```

   该操作返回以下文档：

   ```powershell
   { "_id" : "a", "value" : { "total_time" : 200, "count" : 2, "avg_time" : 100 } }
   { "_id" : "b", "value" : { "total_time" : 230, "count" : 2, "avg_time" : 115 } }
   { "_id" : "c", "value" : { "total_time" : 250, "count" : 2, "avg_time" : 125 } }
   { "_id" : "d", "value" : { "total_time" : 110, "count" : 2, "avg_time" : 55 } }
   ```

5. 新文档添加到`usersessions`集合中：

   ```powershell
   db.usersessions.insertMany([
      { userid: "a", ts: ISODate('2020-03-05 14:17:00'), length: 130 },
      { userid: "b", ts: ISODate('2020-03-05 14:23:00'), length: 40 },
      { userid: "c", ts: ISODate('2020-03-05 15:02:00'), length: 110 },
      { userid: "d", ts: ISODate('2020-03-05 16:45:00'), length: 100 }
   ])
   ```

6. [`$match`]()在管道的开头添加一个阶段以指定日期过滤器：

   ```powershell
   db.usersessions.aggregate([
      { $match: { ts: { $gte: ISODate('2020-03-05 00:00:00') } } },
      { $group: { _id: "$userid", total_time: { $sum: "$length" }, count: { $sum: 1 }, avg_time: { $avg: "$length" } } },
      { $project: { value: { total_time: "$total_time", count: "$count", avg_time: "$avg_time" } } },
      { $merge: {
         into: "session_stats_agg",
         whenMatched: [ { $set: {
            "value.total_time": { $add: [ "$value.total_time", "$$new.value.total_time" ] },
            "value.count": { $add: [ "$value.count", "$$new.value.count" ] },
            "value.avg_time": { $divide: [ { $add: [ "$value.total_time", "$$new.value.total_time" ] },  { $add: [ "$value.count", "$$new.value.count" ] } ] }
         } } ],
         whenNotMatched: "insert"
      }}
   ])
   ```

7. 查询`session_stats_agg`集合以验证结果：

   ```powershell
   db.session_stats_agg.find().sort( { _id: 1 } )
   ```

   该操作返回以下文档：

   ```powershell
   { "_id" : "a", "value" : { "total_time" : 330, "count" : 3, "avg_time" : 110 } }
   { "_id" : "b", "value" : { "total_time" : 270, "count" : 3, "avg_time" : 90 } }
   { "_id" : "c", "value" : { "total_time" : 360, "count" : 3, "avg_time" : 120 } }
   { "_id" : "d", "value" : { "total_time" : 210, "count" : 3, "avg_time" : 70 } }
   ```

8. 可选的。为了避免[`$match`]()每次运行时都必须修改聚合管道的日期条件，可以在帮助函数中定义包装聚合：

   ```powershell
   updateSessionStats = function(startDate) {
      db.usersessions.aggregate([
         { $match: { ts: { $gte: startDate } } },
         { $group: { _id: "$userid", total_time: { $sum: "$length" }, count: { $sum: 1 }, avg_time: { $avg: "$length" } } },
         { $project: { value: { total_time: "$total_time", count: "$count", avg_time: "$avg_time" } } },
         { $merge: {
            into: "session_stats_agg",
            whenMatched: [ { $set: {
               "value.total_time": { $add: [ "$value.total_time", "$$new.value.total_time" ] },
               "value.count": { $add: [ "$value.count", "$$new.value.count" ] },
               "value.avg_time": { $divide: [ { $add: [ "$value.total_time", "$$new.value.total_time" ] },  { $add: [ "$value.count", "$$new.value.count" ] } ] }
            } } ],
            whenNotMatched: "insert"
         }}
      ]);
   };
   ```

   然后，要运行，您只需将开始日期传递给该`updateSessionStats()`函数：

   ```powershell
   updateSessionStats(ISODate('2020-03-05 00:00:00'))
   ```

也可以看看

- [$ merge示例]()
- [按需实例化视图]()



译者：李冠飞

校对：