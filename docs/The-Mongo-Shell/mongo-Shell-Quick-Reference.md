
# mongo Shell 快速参考
**在本页面**

* [mongo Shell命令历史](#命令历史)

* [命令行选项](#命令行选项)

* [命令助手](#助手)

* [Shell基本JavaScript操作](#shell)

* [键盘快捷键](#快捷键)

* [查询](#查询)

* [错误检查方法](#错误检查)

* [行政命令助手](#行政命令助手)

* [打开其他连接](#其他连接)

* [多样式](#多样式)

* [其他资源](#其他资源)

> **[success] Note**
>
> 下面的文档是[MongoDB服务器下载](https://www.mongodb.com/try/download/community?tck=docs_server).中包含的[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell。有关新的MongoDB Shell ，**mongosh**的信息，请参考[mongosh文档](https://docs.mongodb.com/mongodb-shell/)。
>
> 要了解这两种shell的区别，请参阅[Comparison of the mongo Shell and mongosh](https://docs.mongodb.com/master/mongo/#compare-mongosh-mongo).

## <span id="命令历史">mongo Shell命令历史</span>

 您可以使用上下箭头键检索在 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell中发布的先前命令。 命令历史记录存储在**〜/ .dbshell**文件中。 有关更多信息，请参见[.dbshell](https://docs.mongodb.com/master/reference/program/mongo/#mongo-dbshell-file) 。

### <span id="命令行选项">命令行选项</span>

 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell可以使用许多选项启动。 有关所有可用选项的详细信息，请参见[mongo shell](https://docs.mongodb.com/master/reference/program/mongo/) 页面。

下表显示了mongo的一些常用选项：

| 选项 | 说明 |
| --- | --- |
| [--help](#) | 显示命令行选项 |
| [--nodb](#) | 在不连接数据库的情况下启动mongo shell。<br />要稍后连接，请参阅[Opening New Connections](https://docs.mongodb.com/manual/tutorial/write-scripts-for-the-mongo-shell/#mongo-shell-new-connections)。 |
| [--shell](#) | 与JavaScript文件（即<[file.js](#)>]）结合使用，以在运行JavaScript文件后在mongo shell中继续。<br />有关示例，请参见 [JavaScript file](https://docs.mongodb.com/manual/tutorial/write-scripts-for-the-mongo-shell/#mongo-shell-javascript-file)。 |

## <span id="助手">**命令助手**</span>

[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell提供了各种帮助。下表显示了一些常见的帮助方法和命令：

| 帮助方法和命令 | 描述 |
| --- | --- |
| help() | 打印当前数据库的列表 |
| [`db.help()`](https://docs.mongodb.com/master/reference/method/db.help/#db.help) | 打印当前数据库的所有角色的列表，包括用户定义的角色和内置角色。 |
| db.`<collection>`.help() | 打印耗时1毫秒或更长时间的五个最新操作。 有关更多信息，请参见数据库分析器上的文档。 |
| show dbs | 打印所有可用数据库的列表。<br />该操作对应于[`listDatabases`](https://docs.mongodb.com/master/reference/command/listDatabases/#dbcmd.listDatabases)命令。 如果部署使用访问控制运行，则该操作将根据用户权限返回不同的值。 有关详细信息，请参见 [listDatabases](https://docs.mongodb.com/manual/reference/command/listDatabases/#dbcmd.listDatabases)。 |
| use<`db`> | 将当前数据库切换到<`db`>。 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell变量**db**设置为当前数据库。 |
| show collections | 打印当前数据库的所有集合的列表。<br />另可参考：<br />[show collections](https://docs.mongodb.com/manual/release-notes/4.0-compatibility/#compat-show-collections) |
| show users | 打印当前数据库列表 |
| show roles | 打印当前数据库的所有角色的列表，包括用户定义角色和内置角色。 |
| show profile | 打印耗时1毫秒或更长时间的五个最新操作。 有关更多信息，请参见 [database profiler](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/)。 |
| show databases | 打印所有可用数据库的列表。<br />该操作对应于 [listDatabases](https://docs.mongodb.com/manual/reference/command/listDatabases/#dbcmd.listDatabases) 命令。 如果部署使用访问控制运行，则该操作将根据用户权限返回不同的值。 有关详细信息，请参见 [listDatabases](https://docs.mongodb.com/manual/reference/command/listDatabases/#dbcmd.listDatabases)。 |
| load() | 执行一个JavaScript文件。 有关更多信息，请参见 [Write Scripts for the mongo Shell](https://docs.mongodb.com/manual/tutorial/write-scripts-for-the-mongo-shell/)。 |

## <span id="shell">**Shell基本JavaScript操作**</span>

[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell提供了用于数据库操作的[JavaScript API](https://docs.mongodb.com/master/reference/method/) 。

在mongo shell中，**db**是引用当前数据库的变量。该变量自动设置为默认数据库测试，或者在**use <`db`>**切换当前数据库时设置。

下表显示了一些常见的JavaScript操作：

| JavaScript数据库操作 | 说明 |
| --- | --- |
| [db.auth()](https://docs.mongodb.com/manual/reference/method/db.auth/#db.auth) | 如果以安全模式运行，请对用户进行身份验证。 |
| coll = db.<`collection`> | 将当前数据库中的特定集合设置为变量coll，如以下示例所示：<br />coll = db.myCollection;<br />您可以使用变量在myCollection上执行操作，如以下示例所示：<br />coll.find(); |
| [db.collection.find()](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find)<br />  | 查找集合中的所有文档并返回一个游标。<br />有关更多信息和示例，请参见db.collection.find（）和查询文档。<br />有关在mongo shell中处理游标的信息，请参阅在mongo Shell中迭代游标。 |
| [db.collection.insertOne()](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#db.collection.insertOne) | 将新文档插入集合中。 |
| [db.collection.insertMany()](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#db.collection.insertMany) | 将多个新文档插入集合中。 |
| [db.collection.updateOne()](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/#db.collection.updateOne) | 更新集合中的单个现有文档。 |
| [db.collection.updateMany()](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany) | 更新集合中的多个现有文档。 |
|     [db.collection.save()](https://docs.mongodb.com/manual/reference/method/db.collection.save/#db.collection.save) | 插入新文档或更新集合中的现有文档。 |
| [db.collection.deleteOne()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#db.collection.deleteOne) | 从集合中删除单个文档。 |
| [db.collection.deleteMany()](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/#db.collection.deleteMany) | 从集合中删除多个文档 |
| [db.collection.drop()](https://docs.mongodb.com/manual/reference/method/db.collection.drop/#db.collection.drop) | 完全删除或除去集合。 |
| [db.collection.createIndex()](https://docs.mongodb.com/manual/reference/method/db.collection.createIndex/#db.collection.createIndex) | 如果索引不存在，则在集合上创建一个新索引；否则，该操作无效。 |
| [db.getSiblingDB()](https://docs.mongodb.com/manual/reference/method/db.getSiblingDB/#db.getSiblingDB) | 使用相同的连接返回对另一个数据库的引用，而无需显式切换当前数据库。 这允许跨数据库查询。 |

有关在shell中执行操作的更多信息，请参见：

- [MongoDB CRUD Operations](https://docs.mongodb.com/manual/crud/)
- [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/#js-administrative-methods)

## <span id="快捷键">**键盘快捷键**</span>

 shell提供了大多数键盘快捷键，类似于**bash shell**或**Emacs**中的快捷键。 对于某些功能，[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) 提供了多个键绑定，以适应几种熟悉的范例。

下表列举了 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell支持的按键：

| 按键 | 功能 |
| --- | --- |
| Up-arrow | 以前的历史 |
| Down-arrow | 下一个历史 |
| Home | 行起点 |
| End | 行尾 |
| Tab | 自动完成 |
| Left-arrow | 后退字符 |
| Right-arrow | 向前字符 |
| Ctrl-left-arrow | 后向词 |
| Ctrl-right-arrow | 前向词 |
| Meta-left-arrow | 后向词 |
| Meta-right-arrow | 前向词 |
| Ctrl-A | 上线 |
| Ctrl-B | 向后字符 |
| Ctrl-C | 退出 |
| Ctrl-D | 删除字符（或退出） |
| Ctrl-E | 行结束 |
| Ctrl-F | 转发字符 |
| Ctrl-G | 中止 |
| Ctrl-J | 接受线 |
| Ctrl-K | 杀死线 |
| Ctrl-L | 清除屏幕 |
| Ctrl-M | 接受线 |
| Ctrl-N | 下一个历史记录 |
| Ctrl-P | 以前的历史记录 |
| Ctrl-R | 反向搜索历史 |
| Ctrl-S | 正向搜索历史 |
| Ctrl-T | 转置字符 |
| Ctrl-U | 丢弃Unix线 |
| Ctrl-W | Unix单词清除 |
| Ctrl-Y | 拉动 |
| Ctrl-Z | 挂起（作业控制在Linux中有效） |
| Ctrl-H (i.e. Backspace) | 向后删除字符 |
| Ctrl-I (i.e. Tab) | 完成 |
| Meta-B | 后退词 |
| Meta-C | 大写词 |
| Meta-D | 杀死命令 |
| Meta-F | 转发字 |
| Meta-L | 小写词 |
| Meta-U | 大写词 |
| Meta-Y | yank-pop |
| Meta-[Backspace] | 撤销杀死命令 |
| Meta-< | 历史开始 |
| Meta->   | 历史结束 |

## <span id="查询">**查询**</span>

在mongo shell中，使用[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find) 和[`findOne()`](https://docs.mongodb.com/master/reference/method/db.collection.findOne/#db.collection.findOne) 方法执行读取操作。<br />[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法返回一个游标对象，[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell对其进行迭代以在屏幕上打印文档。 默认情况下，[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) 打印前20个结果。[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell将提示用户“输入”以继续迭代接下来的20个结果。<br />下表提供了mongo shell中的一些常见读取操作：

| 读取操作 | 说明描述 |
| --- | --- |
| [`db.collection.find(<query>`)](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) | 在集合中找到符合<`query`>条件的文档。 如果未指定<`query`>条件或该条件为空（即{}），则读取操作将选择集合中的所有文档。<br />以下示例在用户集合中选择name字段等于“ Joe”的文档：coll = db.users;coll.find( { name: "Joe" } );有关指定<`query`>条件的更多信息，请参见：<br />[Specify Equality Condition](https://docs.mongodb.com/manual/tutorial/query-documents/#read-operations-query-argument).<br /> |
| [`db.collection.find(<query>,` `<projection>)`](https://docs.mongodb.com/manual/reference/method/db.collection.find/#db.collection.find) | 查找符合<`query`>条件的文档，并仅返回<`projection`>中的特定字段。<br />以下示例从集合中选择所有文档，但仅返回名称字段和**_id**字段。 除非明确指定不返回，否则始终返回**_id**。<br />**coll = db.users;<br />coll.find（{}，{name：true}）;<br />**有关指定<`projection`>的更多信息，请参见[Project Fields to Return from Query](https://docs.mongodb.com/master/tutorial/project-fields-from-query-results/#read-operations-projection).。 |
| [`db.collection.find().sort(<sort order>)`](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort) | 以指定的<`sort order`>返回结果。<br />以下示例从集合中选择所有文档，并返回按名称字段升序+1排序的结果。 使用-1降序：<br />**coll = db.users;<br />coll.find（）。sort（{name：1}）;** |
| [`db.collection.find(<query>).sort(<sort` `order`>)](https://docs.mongodb.com/manual/reference/method/cursor.sort/#cursor.sort) | 以指定的<`sort order`>返回符合<`query`>条件的文档。 |
| [`db.collection.find( ... ).limit( )`](https://docs.mongodb.com/master/reference/method/cursor.limit/#cursor.limit) | 将结果限制为<`n`>行。 如果只需要一定数量的行以获得最佳性能，则强烈建议使用。 |
| [`db.collection.find( ... ).skip( )`](https://docs.mongodb.com/master/reference/method/cursor.skip/#cursor.skip) | 跳过<`n`>个结果。 |
| [db.collection.count()](https://docs.mongodb.com/manual/reference/method/db.collection.count/#db.collection.count) | 返回集合中的文档总数。 |
| [`db.collection.find().count()`](https://docs.mongodb.com/master/reference/method/cursor.count/#cursor.count) | 返回与查询匹配的文档总数。<br />[`count()`](https://docs.mongodb.com/master/reference/method/cursor.count/#cursor.count)忽略[`limit()`](https://docs.mongodb.com/master/reference/method/cursor.limit/#cursor.limit)和[`skip()`](https://docs.mongodb.com/master/reference/method/cursor.skip/#cursor.skip).例如，如果有100条记录匹配，但限制为10，则[`count()`](https://docs.mongodb.com/master/reference/method/cursor.count/#cursor.count)将返回100。这比迭代自己的速度更快，但仍然需要时间。 |
| [`db.collection.findOne()`](https://docs.mongodb.com/master/reference/method/db.collection.findOne/#db.collection.findOne) | 查找并返回一个文档。 如果找不到，则返回null。<br />以下示例在用户集合中选择一个名称与“ Joe”匹配的文档：<br />**coll = db.users;<br />coll.findOne（{name：“ Joe”}）;<br />**在内部，**[`findOne()`](https://docs.mongodb.com/master/reference/method/db.collection.findOne/#db.collection.findOne)**方法是带有[`limit(1)`](https://docs.mongodb.com/master/reference/method/cursor.limit/#cursor.limit)的[`find()`](https://docs.mongodb.com/master/reference/method/db.collection.find/#db.collection.find)方法。 |

有关更多信息和示例，请参阅[Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/) 。 请参阅[Query and Projection Operators](https://docs.mongodb.com/manual/reference/operator/query/)。<br />

## <span id="错误检查">**错误检查方法**</span>

mongo shell write方法将**Write Concern**直接集成到方法执行中，并返回一个**WriteResult()**对象，该对象包含操作结果，包括所有写错误和写关注错误。<br />

<span id="行政命令助手">**行政命令助手**</span>

下表列出了一些支持数据库管理的常用方法：

| JavaScript数据库管理 | 方法说明 |
| --- | --- |
| [`db.fromColl.renameCollection(<toColl>)`](https://docs.mongodb.com/manual/reference/method/db.collection.renameCollection/#db.collection.renameCollection) | 将集合从**fromColl**重命名为<`toColl`>。 请参阅[Naming Restrictions](https://docs.mongodb.com/manual/reference/limits/#restrictions-on-db-names)。 |
| [`db.getCollectionNames()`](https://docs.mongodb.com/manual/reference/method/db.getCollectionNames/#db.getCollectionNames) | 获取当前数据库中所有集合的列表。 |
| [`db.dropDatabase()`](https://docs.mongodb.com/manual/reference/method/db.dropDatabase/#db.dropDatabase) | 删除当前数据库。 |

另请参见[administrative database methods](https://docs.mongodb.com/manual/reference/method/#js-administrative-methods)以获取方法的完整列表。<br />

## <span id="其他连接">**打开其他连接**</span>

您可以在mongo shell中创建新的连接。<br />下表显示了创建连接的方法：

| JavaScript连接创建方法 | 说明 |
| --- | --- |
| db = connect("<`host`><:port>/<`dbname`>") | 打开一个新的数据库连接。 |
| conn = **new** Mongo()<br />db = conn.getDB("dbname") | 使用新的Mongo（）打开与新服务器的连接。<br />使用连接的getDB（）方法选择数据库。 |

另请参阅 [Opening New Connections](https://docs.mongodb.com/manual/tutorial/write-scripts-for-the-mongo-shell/#mongo-shell-new-connections)以获取有关从mongo shell打开新连接的更多信息。<br />

## <span id="多样式">**多样式**</span>

下表显示了一些其他方法：

| 方法 | 描述 |
| --- | --- |
| Object.bsonsize(<`document`>) | Prints the [BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson) size of a <`document`> in bytes |

## <span id="其他资源">**其他资源**</span>

考虑以下解决mongo shell及其接口的参考资料：

- [mongo](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo)
- [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/#js-administrative-methods)
- [Database Commands](https://docs.mongodb.com/manual/reference/command/#database-commands)
- [Aggregation Reference](https://docs.mongodb.com/manual/reference/aggregation/#aggregation-reference)
- [Getting Started Guide](https://docs.mongodb.com/getting-started/shell)

另外，MongoDB源代码存储库包括一个[jstests](https://github.com/mongodb/mongo/tree/master/jstests/)目录，该目录包含许多mongo shell脚本。



译者：王恒

校对：杨帅