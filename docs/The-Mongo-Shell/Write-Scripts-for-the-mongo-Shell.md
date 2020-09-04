
# 为mongo Shell编写脚本
**在本页面**

* [打开新连接](#新连接)

* [交互式mongo与脚本mongo的区别](区别)

* [脚本编写](#脚本)

>**[success] Note**
>
>下面的文档是[MongoDB服务器下载](https://www.mongodb.com/try/download/community?tck=docs_server).中包含的[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell。有关新的MongoDB Shell ，**mongosh**的信息，请参考[mongosh文档](https://docs.mongodb.com/mongodb-shell/)。
>
>要了解这两种shell的区别，请参阅[Comparison of the mongo Shell and mongosh](https://docs.mongodb.com/master/mongo/#compare-mongosh-mongo).

您可以为[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell编写JavaScript 的脚本，来处理MongoDB中的数据或执行管理操作。 
本章节介绍了通过[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell编写的`JavaScript`的方法 来访问 Mongodb的方式。

## <span id="新连接">打开新连接</span>

在[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell或JavaScript文件中，您可以使用[`Mongo()`](https://docs.mongodb.com/master/reference/method/Mongo/#Mongo)构造函数实例化数据库连接：

```shell
Mongo()
new Mongo(<host>)
new Mongo(<host:port>)
```

请考虑以下示例，该示例实例化与在默认端口上的localhost上运行的MongoDB实例的新连接，并使[`getDB()`](https://docs.mongodb.com/master/reference/method/Mongo.getDB/#Mongo.getDB)方法将全局db变量设置为myDatabase：
```powershell
conn = new Mongo();
db = conn.getDB("myDatabase");
```

如果连接到已经开启了访问控制的MongoDB实例，则可以使用[`db.auth()`](https://docs.mongodb.com/master/reference/method/db.auth/#db.auth) 方法进行身份验证。  
此外，您可以使用`connect()`方法连接到MongoDB实例。 以下示例使用非默认端口**27020**连接到在localhost上运行的MongoDB实例，并设置全局db变量：

```powershell
db = connect("localhost:27020/myDatabase");
```

另可参考：<br />[mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/)

## <span id="区别">交互式mongo与脚本mongo的区别</span>

> **[success] Note**
>
> 从4.2版开始，[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell提供了[`isInteractive()`](https://docs.mongodb.com/master/reference/method/isInteractive/#isInteractive) 方法，该方法返回一个布尔值，该值指示[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell是在交互模式还是脚本模式下运行。

**为mongo shell编写脚本时，请考虑以下事项：**

* 要设置db全局变量，请使用[`getDB()`](https://docs.mongodb.com/master/reference/method/Mongo.getDB/#Mongo.getDB) 方法或`onnect()`方法。您可以将数据库引用分配给**db**以外的其他变量。

* [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell中的写操作默认情况下使用[{ w: 1 }](https://docs.mongodb.com/master/reference/write-concern/#wc-w)的写入策略。 如果执行批量操作，请使用`Bulk()`方法。 有关更多信息，请参见：[Write Method Acknowledgements](https://docs.mongodb.com/manual/release-notes/2.6-compatibility/#write-methods-incompatibility)）。

* 您不能在JavaScript文件中使用任何shell帮助程序（例如，使用`<dbname>`，**show dbs**等），因为它们不是有效的JavaScript。<br />下表将最常见的mongo shell助手映射到其JavaScript等效项：


| Shell帮助 | 等价JavaScript |
| --- | --- |
| show  dbs, show  databases | db.adminCommand('listDatabases') |
| use `<db>` | db = db.getSiblingDB('`<db>`') |
| show collections |  db.getCollectionNames() |
| show users | db.getUsers() |
| show roles | db.getRoles({showBuiltinRoles: **true**}) |
| show log` <logname>` | db.adminCommand({ 'getLog' : '`<logname>`' }) |
| show logs | db.adminCommand({ 'getLog' : '*' }) |
| it | cursor = db.collection.find()<br />**if** ( cursor.hasNext() ){ <br />  cursor.next();<br />} |

在交互模式下， [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) 打印操作结果，包括所有游标的内容。 在脚本中，使用JavaScript **print()**函数或 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) 特定的**printjson()**函数，该函数返回格式化的JSON。

例子:

要在mongo shell脚本中打印结果游标中的所有项目，请使用以下惯用法：

```java
cursor = db.collection.find();
while ( cursor.hasNext() ) {
	printjson( cursor.next() );
	}
```

## <span id="脚本">脚本编写</span>

在系统提示下，使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) 评估JavaScript。

 **--eval选项**

使用[--eval]()选项 让Mongo来执行一个JavaScript片段，如下所示：

```shell
mongo test --eval "printjson(db.getCollectionNames())"
```

这将使用连接到在本地主机接口上的端口27017上运行的[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例的[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell返回[`db.getCollectionNames()`](https://docs.mongodb.com/master/reference/method/db.getCollectionNames/#db.getCollectionNames) 的输出。<br />
**执行一个JavaScript文件**

您可以在mongo shell中指定.js文件，然后mongo将直接执行JavaScript。 考虑以下示例：

```shell
mongo localhost:27017/test myjsfile.js
```

此操作在mongo shell中执行`myjsfile.js`脚本，该脚本连接到可通过端口27017上的localhost接口访问的mongod实例上的测试数据库。或者，您可以使用`Mongo()`构造函数在javascript文件中指定mongodb连接参数。 <br />
有关更多信息，请参见：[打开新连接](https://docs.mongodb.com/manual/tutorial/write-scripts-for-the-mongo-shell/#mongo-shell-new-connections) 。<br />
您可以使用`load()`函数从mongo shell中执行.js文件，如下所示：

```shell
load("myjstest.js")
```

此函数加载并执行**myjstest.js**文件。<br />
**load()**方法接受相对路径和绝对路径。 如果mongo shell的当前工作目录为**/ data / db**，而**myjstest.js**位于**/ data / db / scripts**目录中，则mongo shell中的以下调用将是等效的：

```shell
load("scripts/myjstest.js")
load("/data/db/scripts/myjstest.js")
```

> **[success] Note**
>
> **load（）函数没有搜索路径。 如果所需的脚本不在当前工作目录或完整的指定路径中，则[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)将无法访问该文件。**



译者：王恒

校对：杨帅