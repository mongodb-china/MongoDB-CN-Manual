

# The mongo Shell

**在本页面**

* [启动mongo Shell并连接到MongoDB](#启动连接)

* [使用mongo Shell](#使用mongo-Shell)

* [制表符完成和其他键盘快捷键](#制表符完成和其他键盘快捷键)

* [mongorc.js文件](#mongorcjs文件)

* [退出Shell](#退出Shell)

mongo shell是MongoDB的交互式JavaScript接口。您可以使用mongo shell查询和更新数据以及执行管理操作。
mongo shell作为MongoDB Server安装的一部分包含在内。 MongoDB还提供mongo shell作为独立软件包。如何下载独立的mongo shell软件包：
1.打开下载中心。对于mongo Enterprise Shell，选择MongoDB Enterprise Server选项卡。
2.从下拉列表中选择您的首选版本和操作系统。
3.选择要根据您的系统下载的安装包：

| 系统 | 下载包 |
| --- | --- |
| Win | 选择ZIP以下载包含mongo shell的安装包 |
| Mac | 选择TGZ以下载包含mongo shell的安装包 |
| Linux | 选择shell以下载包含mongo shell的安装包 |

安装并启动MongoDB之后，将mongo shell连接到正在运行的MongoDB实例。
## <span id="启动连接">启动mongo Shell并连接到MongoDB</span>
> **前提条件**
>
> 在尝试启动mongo shell之前，请确保MongoDB正在运行。

打开终端窗口（或Windows的命令提示符），然后转`<mongodb安装目录>/bin` 目录。

```shell
cd <mongodb安装目录>/bin
```

> **[success] Note**
>
> 将`<mongodb安装目录> / bin`添加到PATH环境变量中，可以键入mongo，而不必转到`<mongodb安装目录> / bin`目录或指定二进制文件的完整路径。

#### **默认端口上的本地MongoDB实例**

您可以在不使用任何命令行选项的情况下运行mongo shell，以使用默认端口27017连接到在本地主机上运行的MongoDB实例：

```shell
mongo
```

#### **非默认端口上的本地MongoDB实例**

要显式指定端口，请包括--port命令行选项。例如，要使用非默认端口28015连接到在localhost上运行的MongoDB实例，请执行以下操作：

#### **非默认端口上的本地MongoDB实例**

要显式指定端口，请包括--port命令行选项。例如，要使用非默认端口28015连接到在localhost上运行的MongoDB实例，请执行以下操作：

```shell
mongo --port 28015
```

#### **远程主机上的MongoDB实例**

要明确指定主机名或端口：

* 您可以指定一个[连接字符串](https://docs.mongodb.com/master/reference/connection-string/)。例如：要连接到在远程主机上运行的MongoDB实例，请执行以下操作：

```shell
mongo "mongodb://mongodb0.example.com:28015"
```

* 您可以使用命令行选项[--host <`host`>:<`port`>](https://docs.mongodb.com/manual/reference/program/mongo/#cmdoption-mongo-host)。例如，要连接到在远程主机上运行的MongoDB实例，请执行以下操作：

```shell
mongo --host mongodb0.example.com:28015
```

您可以使用[`--host `<`host`>](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-host)和[`--port ` <`port`>](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-port) 命令行选项。例如，要连接到在远程主机上运行的MongoDB实例，请执行以下操作：

```shell
mongo --host mongodb0.example.com --port 28015
```

#### **具有身份验证的MongoDB实例**

要连接到MongoDB实例，需要进行身份验证：

您可以在连接字符串中指定用户名，身份验证数据库以及可选的密码。例如：以**alice**用户身份连接并认证到远程MongoDB实例：

> **[success]  Note**
>
> 如果未在连接字符串中指定密码，则shell程序将提示您输入密码。

```shell
mongo "mongodb://alice@mongodb0.examples.com:28015/?authSource=admin"
```

您可以使用[`--username `<`user`>](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-username) 和[`--password`](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-password), [`--authenticationDatabase <db>`](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-authenticationdatabase)命令行选项。 例如，以**alice**用户身份连接并认证到远程MongoDB实例：

> **[success]  Note**
>
> 如果您指定**--password**而不输入用户密码，则shell程序将提示您输入密码。

```shell
mongo --username alice --password --authenticationDatabase admin --host mongodb0.examples.com --port 28015
```

#### 连接到MongoDB复制集

要连接到复制集：

* 您可以在连接字符串中指定复制集名称和成员

```shell
mongo "mongodb://mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017/?replicaSet=replA"
```

* 如果使用[DNS Seedlist 链接格式](https://docs.mongodb.com/master/reference/connection-string/#connections-dns-seedlist)，则可以指定连接字符串：

```shell
mongo "mongodb+srv://server.example.com/"
```

> **[success]  Note**
>
> 对于连接，使用**+ srv**连接字符串修饰符会自动将ssl选项设置为true。

* 您可以从 [ --host `<replica set name>/<host1>:<port1>,<host2>:<port2>`,... ](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-host) 命令行选项中指定复制集名称和成员。 例如，要连接到名为**replA**的复制集，请执行以下操作：

```shell
mongo --host replA/mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017
```

#### **TLS/SSL 连接**

关于TLS/SS连接：

* 您可以在[连接字符串](https://docs.mongodb.com/master/reference/connection-string/)中指定**ssl = true**选项。

```shell
mongo "mongodb://mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017/?replicaSet=replA&ssl=true"
```

* 如果使用[DNS Seedlist 链接格式](https://docs.mongodb.com/master/reference/connection-string/#connections-dns-seedlist)，则可以包括**+srv**连接字符串修饰符：

```shell
mongo "mongodb+srv://server.example.com/"
```

> **[success]  Note**
>
> 对于连接，使用**+srv**连接字符串修饰符会自动将ssl选项设置为true。

* 您可以指定[`--ssl`](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-ssl)命令行选项。 例如，要连接到名为**replA**的复制集，请执行以下操作：

```shell
mongo --ssl --host replA/mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017
```

  另：有关连接示例中使用的选项以及其他选项的更多信息，请参阅([mongo参考](https://docs.mongodb.com/manual/reference/program/mongo/)和 [启动mongo的示例](https://docs.mongodb.com/manual/reference/program/mongo/#mongo-usage-examples))。

## <span id="使用mongo-Shell">使用mongoShell</span>

要显示您正在使用的数据库，请键入**db**：

```shell
db
```

该操作应返回**test** 数据库名，这是默认数据库。
要切换数据库，请发出**use <`db`>**帮助器，如以下示例所示：<br />

```shell
use <database>
```

另请参见[`db.getSiblingDB()`](https://docs.mongodb.com/master/reference/method/db.getSiblingDB/#db.getSiblingDB)方法，以从当前数据库访问其他数据库，而无需切换当前数库上下文（即**db**）。<br />
要列出用户可用的数据库，可使用：**show dbs**  <显示用户列表里所有数据库>

您可以切换到不存在的数据库。首次将数据存储在数据库中（例如通过创建集合）时，MongoDB会创建数据库。 例如，以下代码在insertOne（）操作期间创建数据库**myNewDatabase**和[集合](https://docs.mongodb.com/master/reference/glossary/#term-collection) **myCollection**：<br />

```shell
use myNewDatabase
db.myCollection.insertOne( { x: 1 } );
```

 是mongo shell中可用的方法之一。

* **db**是指当前数据库。

* **myCollection**是集合的名称。

  如果[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell不接受集合的名称，则可以使用替代的 [`db.getCollection()`](https://docs.mongodb.com/master/reference/method/db.getCollection/#db.getCollection)语法。例如，如果集合名称包含空格或连字符，以数字开头或与内置函数冲突：

```shell
db.getCollection("3 test").find()
db.getCollection("3-test").find()
db.getCollection("stats").find()
```

[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)   shell提示符每行的限制为4095个字符（code points）。 如果您输入的行中包含4095个以上的字符（code points），则Shell将截断它。
有关mongo shell中MongoDB基本操作的更多文档，请参阅：

- [Getting Started Guide](https://docs.mongodb.com/getting-started/shell)
- [Insert Documents](https://docs.mongodb.com/manual/tutorial/insert-documents/)
- [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)
- [Update Documents](https://docs.mongodb.com/manual/tutorial/update-documents/)
- [Delete Documents](https://docs.mongodb.com/manual/tutorial/remove-documents/)
- [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/)

如果部署使用访问控制运行，则该操作将根据用户权限返回不同的值。 有关详细信息，请参见[listDatabases Behavior](https://docs.mongodb.com/manual/reference/command/listDatabases/#listdatabases-behavior)。<br />

### **格式化打印结果**<br />
`db.collection.find()`方法是用于从集合中检索文档的JavaScript方法。 `db.collection.find()`方法将游标返回到结果。
 但是，在mongo shell中，如果未使用**var**关键字将返回的游标分配给变量，则该游标会自动迭代最多20次，来打印与查询匹配的前20个文档。 
 mongo shell将提示 输入`it`以使其再次迭代20次。<br />

要格式化打印结果，可以将`.pretty()`添加到操作中，如下所示：


```shell
db.myCollection.find().pretty()
```

此外，您可以在mongo shell中使用以下显式打印方法：

- print() to print without formatting
- print(tojson(<obj>)) to print with [JSON](https://docs.mongodb.com/manual/reference/glossary/#term-json) formatting and equivalent to printjson()
- printjson() to print with [JSON](https://docs.mongodb.com/manual/reference/glossary/#term-json) formatting and equivalent to print(tojson(<obj>))

有关在mongo shell中处理光标的更多信息和示例，请参阅[terate a Cursor in the mongo](https://docs.mongodb.com/manual/tutorial/iterate-a-cursor/)。 另请参阅[Cursor Help](https://docs.mongodb.com/manual/tutorial/access-mongo-shell-help/#mongo-shell-help-cursor) ，以获取mongo shell中的游标帮助列表。

### mongo Shell中的多行操作

如果您以开括号（'（'），大括号（'{'）或开括号（'['）结束一行，则后续行以省略号（“ ...”）开头，直到您 输入相应的右括号（'）'，右括号（'}'）或右括号（']'）。 mongo shell在评估代码之前等待右括号，右括号或右括号，如以下示例所示：

```shell
>if ( x > 0 ) {
... count++;
... print (x);
... }
```

如果输入两个空行，则可以退出行继续模式，如以下示例所示：<br />

```shell
> if (x > 0
...
...
>
```

## <span id="制表符完成和其他键盘快捷键">制表符完成和其他键盘快捷键</span>

 shell支持键盘快捷键。 例如：

* 使用向上/向下箭头键滚动浏览命令历史记录。有关.dbshell文件的更多信息，请参见[.dbshell](https://docs.mongodb.com/manual/reference/program/mongo/#mongo-dbshell-file) 文档。
* 使用<`Tab`>来自动完成或列出完成可能性，如以下示例中所示，该示例使用<`Tab`>来完成以字母'c'开头的方法名称：<br />

```shell
db.myCollection.c<Tab>
```

因为有许多以字母'**c**'开头的收集方法，所以<`Tab`>将列出以**'c'**开头的各种方法。<br />
有关快捷键的完整列表，请参见：Shell 快捷命令（[Shell Keyboard Shortcuts](https://docs.mongodb.com/manual/reference/program/mongo/#mongo-keyboard-shortcuts)）。

## <span id="mongorcjs文件">mongorc.js文件</span>

启动时，mongo将在用户的HOME目录中检查名为`.mongorc.js`的JavaScript文件。 如果找到，mongo会在首次显示提示之前解释`.mongorc.js`的内容。如果您使用舍shell程序来评估JavaScript文件或表达式，或者通过在命令行上使用--eval选项，或者通过将.js文件指定给mongo，则mongo将在JavaScript完成处理后读取`.mongorc.js`文件。 您可以使用--norc选项防止加载`.mongorc.js`。<br />

## <span id="退出Shell"> 退出Shell </span>

要退出shell，请键入`quit（）`或使用 `<Ctrl-C>`快捷方式。<br />

另可参考：

- [Getting Started Guide](https://docs.mongodb.com/getting-started/shell)
- [mongo](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo) Reference Page



译者：王恒  金江

校对：杨帅
