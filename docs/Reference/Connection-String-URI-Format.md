## 连接字符串URI格式

**在本页面**

- [连接字符串格式](#格式)
- [连接字符串选项](选项)
- [例子](#例子)

本文档介绍了URI格式，用于在官方MongoDB [drivers](https://docs.mongodb.com/ecosystem/drivers).定义应用程序和MongoDB实例之间的连接 。有关驱动程序的列表和驱动程序文档的链接，请参见[drivers](https://docs.mongodb.com/ecosystem/drivers)。

### <span id="格式">连接字符串格式</span>

你可以指定MongoDB连接字符串使用任何:

- the [标准连接字符串格式](https://docs.mongodb.com/master/reference/connection-string/#connections-standard-connection-string-format) 
- the [DNS Seedlist 连接格式](https://docs.mongodb.com/master/reference/connection-string/#connections-dns-seedlist).

#### 标准连接字符串格式

本节描述用于连接到MongoDB部署的MongoDB连接URI的标准格式:独立、复制集或分片集群。

标准的URI连接方案有如下形式:

```powershell
mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]
```

#### 例子

* 单机版

   1.对于独立版本

```powershell
mongodb://mongodb0.example.com:27017
```

​        2. 对于一个[强制访问控制](https://docs.mongodb.com/master/tutorial/enable-authentication/):

```powershell
mongodb://myDBReader:D1fficultP%40ssw0rd@mongodb0.example.com:27017/?authSource=admin
```

* 复制集

> 注意
>
> 对于一个复制集，指定副本集配置中列出的[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例的主机名。
>
> 对于复制集，包括[`replicaSet`](https://docs.mongodb.com/master/reference/connection-string/#urioption.replicaSet) 选项。

​        1.对于一个复制集:

```powershell
mongodb://mongodb0.example.com:27017,mongodb1.example.com:27017,mongodb2.example.com:27017/?replicaSet=myRepl
```

​        2.对于[强制访问控制](https://docs.mongodb.com/master/tutorial/enable-authentication/)的复制集，包括用户凭证:

```powershell
mongodb://myDBReader:D1fficultP%40ssw0rd@mongodb0.example.com:27017,mongodb1.example.com:27017,mongodb2.example.com:27017/?authSource=admin&replicaSet=myRepl
```

* 分片集群

> 注意
>
> 对于分片集群的连接字符串，在连接字符串中指定[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)主机。

​       1.对于分片集群:

```powershell
mongodb://mongos0.example.com:27017,mongos1.example.com:27017,mongos2.example.com:27017
```

​        2.对于实施访问控制的分片集群，包括用户凭证:

```powershell
mongodb://myDBReader:D1fficultP%40ssw0rd@mongos0.example.com:27017,mongos1.example.com:27017,mongos2.example.com:27017/?authSource=admin
```

如果用户名或密码包含at符号`@`，冒号`:`，斜杠`/`或百分号`%`字符，请使用[百分比编码](https://tools.ietf.org/html/rfc3986#section-2.1)

更多示例，请参见[examples](https://docs.mongodb.com/master/reference/connec-string/# connections-connec-examples)。

#### 组件

标准的URI连接字符串包括以下组件:

| 组件                 | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| `mongodb://`         | 标识这是标准连接格式的字符串的必需前缀。                     |
| `username:password@` | 可选的。身份验证凭据。<br />如果指定，则客户端将尝试向验证用户[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)。如果 [`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)未指定，则客户端将尝试向验证用户`defaultauthdb`。如果`defaultauthdb`未指定，则发送到`admin` 数据库。<br />如果用户名或密码包含at符号`@`，冒号`:`，斜杠`/`或百分号`%`字符，请使用[百分比编码](https://tools.ietf.org/html/rfc3986#section-2.1)。<br />另请参阅[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)。 |
| `host[:port]`        | [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例（或分片[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) 群集的实例）运行所在的主机（和可选的端口号） 。您可以指定主机名，IP地址或UNIX域套接字。根据您的部署拓扑指定尽可能多的主机：<br/>对于独立[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例，请指定独立实例的主机名 。<br />对于复制集，请指定[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 复制集配置中列出的实例的主机名。<br />对于分片群集，请指定[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例的主机名 。<br />如果未指定端口号，`27017` 则使用默认端口。 |
| `/defaultauthdb`     | 可选的。如果连接字符串包含`username:password@` 身份验证凭据但未[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)指定选项，则使用的身份验证数据库。<br />如果两个[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)和`defaultauthdb`未指定，客户端将尝试以指定用户的身份验证`admin`数据库。 |
| `?<options>`         | 可选的。查询字符串，将连接特定的选项指定为`<name>=<value>`对。有关这些选项的完整说明，请参见 [连接字符串选项](https://docs.mongodb.com/master/reference/connection-string/#connections-connection-options)。<br />如果连接字符串没有指定数据库/您必须在最后一台主机和开始选项字符串的问号之间指定一个斜杠(/)。 |

#### DNS Seedlist 连接格式

*新增3.6版*

除了标准的连接格式，MongoDB还支持一个DNS构造的Seedlist列表。使用DNS构造可用服务器列表允许更灵活的部署，并允许在不重新配置客户机的情况下轮流更改服务器。

为了利用DNSSeedlist列表，使用一个连接字符串前缀 **mongodb+srv:**，而不是标准的 **mongodb:**。`+srv `向客户端表明后面的主机名对应于一个DNS srv记录。驱动程序或[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell将查询DNS记录，以确定哪些主机正在运行[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例。

> 注意
>
> 使用 **+srv** 连接字符串修饰符自动将该连接的[`tls`](https://docs.mongodb.com/master/reference/connec-string/#urioption.tls)(或等效的[`ssl`](https://docs.mongodb.com/master/reference/connec-string/#urioption.ssl))选项设置为**true**。您可以通过将查询字符串中的[`tls`](https://docs.mongodb.com/master/reference/connection-string/#urioption.tls) (或等效[`ssl`](https://docs.mongodb.com/master/reference/connection-string/#urioption.ssl))选项显式设置为`false`通过 `tls=false`（或`ssl=false`）来覆盖此行为。

下面的例子显示了一个典型的DNS seedlist连接字符串的连接字符串:

```powershell
mongodb+srv://server.example.com/
```

对应的DNS配置如下:

```powershell
Record                            TTL   Class    Priority Weight Port  Target
_mongodb._tcp.server.example.com. 86400 IN SRV   0        5      27317 mongodb1.example.com.
_mongodb._tcp.server.example.com. 86400 IN SRV   0        5      27017 mongodb2.example.com.
```

> 重要
>
> SRV记录中返回的主机名必须与给定的主机名共享相同的父域(在本例中为`example.com`)。如果父域名和主机名不匹配，您将无法连接。

与标准连接字符串一样，DNS seedlist连接字符串支持将选项指定为查询字符串。使用DNSseedlist列表连接字符串，您还可以通过TXT记录指定以下选项:

- `replicaSet`
- `authSource`

您只能为每个[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例指定一个TXT记录。如果多个TXT记录出现在DNS/或如果TXT记录包含一个选项，而不是`replicaSet`或`authSource`，客户端将返回一个错误。

TXT记录`server.example.com`DN条目类似于：

```powershell
    记录                    TTL      Class                     Text
server.example.com.       86400    IN TXT“            replicaSet = mySet＆authSource = authDB”
```

综合起来，DNS SRV记录和TXT记录中指定的选项解析为以下标准格式的连接字符串:

```powershell
mongodb://mongodb1.example.com:27317,mongodb2.example.com:27017/?replicaSet=mySet&authSource=authDB
```

您可以通过在查询字符串中传递选项来重写TXT记录中指定的选项。在下面的示例中，查询字符串为TXT中配置的`authSource`选项提供了重写记录上面的DNS条目。

```powershell
mongodb + srv：//server.example.com/？connectTimeoutMS = 300000＆authSource = aDifferentAuthDB
```

给定`authSource`的重写，标准格式的等效连接字符串为:

```powershell
mongodb：//mongodb1.example.com：27317，mongodb2.example.com：27017 /？connectTimeoutMS = 300000＆replicaSet = mySet＆authSource = aDifferentAuthDB
```

> 注意
>
> 如果没有与连接字符串中标识的主机名对应的可用DNS记录，**mongodb+srv**选项将失败。此外，使用`+srv`连接字符串修饰符会自动为连接设置[`tls`](https://docs.mongodb.com/master/reference/connection-string/#urioption.tls)（或等效 [`ssl`](https://docs.mongodb.com/master/reference/connection-string/#urioption.ssl)）选项`true`。您可以通过将查询字符串中的[`tls`](https://docs.mongodb.com/master/reference/connection-string/#urioption.tls) (或等效[`ssl`](https://docs.mongodb.com/master/reference/connection-string/#urioption.ssl))选项显式设置为`false`通过 `tls=false`（或`ssl=false`）来覆盖此行为。

请看：

[使用DNS`Seedlist`列表连接格式连接到复制集](https://docs.mongodb.com/master/reference/program/mongo/#example-connect-mongo-using-srv)提供一个使用DNS`seedlist`列表连接格式连接[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)  shell到复制集的示例。

### <span id="选项">连接字符串选项</span>

本节列出了所有连接选项。

连接选项是成对的，格式如下：`name=value`。

- `name`使用驱动程序时，该选项不区分大小写。
- `name`使用4.2+版本的[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)Shell 时，该选项不区分大小写 。
- `name`使用4.0版或更早版本的[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)Shell 时，此选项区分大小写。
- `value`始终是区分大小写的。

用`&`字符分隔选项(即:`&`)`name1=value1&name2=value2`。在以下示例中，连接包括[`replicaSet`](https://docs.mongodb.com/master/reference/connection-string/#urioption.replicaSet)和 [`connectTimeoutMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.connectTimeoutMS)选项：

```powershell
mongodb：//db1.example.net：27017，db2.example.net：2500 /？replicaSet = test＆connectTimeoutMS = 300000
```

用于连接字符串参数的分号分隔符:

为了提供向后兼容性，驱动程序目前接受分号(即`;`)作为选项分隔符。

#### 复制集选项

以下连接字符串到一个名为`myRepl`的复制集，成员运行在指定的主机上:

```powershell
mongodb：//db0.example.com：27017，db1.example.com：27017，db2.example.com：27017 /？replicaSet = myRepl
```

| 连接选项     | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| `replicaSet` | 如果[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)是复制集的成员，则指定[复制集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)的名称。<br />当连接到复制集时，向`uri`的**host[:port]**组件提供复制集成员的seed列表。有关具体细节，请参考您的驱动程序文档。 |

#### 连接选项

##### TLS选项

下面的连接字符串到一个复制集包括 [`tls=true`](https://docs.mongodb.com/master/reference/connection-string/#urioption.tls)选项(在MongoDB 4.2可用):

```powershell
mongodb：//db0.example.com,db1.example.com,db2.example.com/？replicaSet = myRepl＆tls = true
```

或者，你也可以使用等价的[`ssl=true`](https://docs.mongodb.com/master/reference/connection-string/#urioption.ssl)选项:

```powershell
mongodb：//db0.example.com,db1.example.com,db2.example.com/？replicaSet = myRepl＆ssl = true
```

| 连接选项                        | 描述                                                         |
| :------------------------------ | :----------------------------------------------------------- |
| `tls`                           | 为连接启用或禁用TLS / SSL：<br />`true`：使用TLS / SSL启动连接。[DNS`seedlist`列表连接格式的](https://docs.mongodb.com/master/reference/connection-string/#connections-dns-seedlist)默认设置 。<br />`false`：在没有TLS / SSL的情况下启动连接。[标准连接字符串格式的](https://docs.mongodb.com/master/reference/connection-string/#connections-standard-connection-string-format)默认设置 。<br />注意<br />该[`tls`](https://docs.mongodb.com/master/reference/connection-string/#urioption.tls)选项等效于该 [`ssl`](https://docs.mongodb.com/master/reference/connection-string/#urioption.ssl)选项。<br/>如果**mongo** shell从命令行指定了额外的[`tls / ssl`](https://docs.mongodb.com/master/reference/program/mongo/#mongo-shell-tls)选项，则使用[`--tls`](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-tls)命令行选项。<br/>*4.2版中的新功能。* |
| `ssl`                           | 用于连接启用或禁用TLS / SSL的布尔值：<br />`true`：使用TLS / SSL启动连接。[DNS`seedlist`列表连接格式的](https://docs.mongodb.com/master/reference/connection-string/#connections-dns-seedlist)默认设置 。<br />`false`：在没有TLS / SSL的情况下启动连接。[标准连接字符串格式的](https://docs.mongodb.com/master/reference/connection-string/#connections-standard-connection-string-format)默认设置。<br />注意<br />该[`ssl`](https://docs.mongodb.com/master/reference/connection-string/#urioption.ssl)选项等效于该 [`tls`](https://docs.mongodb.com/master/reference/connection-string/#urioption.tls)选项。<br />如果**mongo** shell从命令行指定了额外的[`tls / ssl`](https://docs.mongodb.com/master/reference/program/mongo/#mongo-shell-tls)选项，则使用[`--tls`](https://docs.mongodb.com/master/reference/program/mongo/#cmdoption-mongo-tls)命令行选项。 |
| `tlsCertificateKeyFile`         | 指定`.pem`包含客户端的TLS / SSL X.509证书或客户端的TLS / SSL证书和密钥的本地文件的位置。<br />客户端将此文件呈现给 [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/ [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例。<br />*4.4版本改变：*[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) / [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)记录连接上的警告如果给出x.509证书在`mongod/mongos`主机系统时间的30天内到期。有关更多信息，请参见 [x.509证书即将过期触发警告](https://docs.mongodb.com/master/release-notes/4.4/#rel-notes-certificate-expiration-warning)。<br/>并非所有驱动程序都支持此选项。请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。<br />此连接字符串选项不适用于`mongo` shell。请改用命令行选项。<br />*4.2版中的新功能。* |
| `tlsCertificateKeyFilePassword` | 指定用于反加密[`tlsCertificateKeyFile`](https://docs.mongodb.com/master/reference/connection-string/#urioption.tlsCertificateKeyFile)的密码。<br />并非所有驱动程序都支持此选项。请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。<br />此连接字符串选项不适用于`mongo` shell。请改用命令行选项。<br />*4.2版中的新功能。* |
| `tlsCAFile`                     | 指定`.pem`包含来自证书颁发机构的根证书链的本地文件的位置。此文件用于验证[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/ [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) 实例提供的证书。<br />并非所有驱动程序都支持此选项。请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。<br />此连接字符串选项不适用于`mongo` shell。请改用命令行选项。<br />*4.2版中的新功能。* |
| `tlsAllowInvalidCertificates`   | 绕过[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/ [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例提供的证书的验证<br />设置为`true`连接到MongoDB实例，即使服务器当前存在无效证书。<br />并非所有驱动程序都支持此选项。请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。<br />此连接字符串选项不适用于`mongo` shell。请改用命令行选项。<br />警告<br />禁用证书验证会产生漏洞。<br />*4.2版中的新功能。* |
| `tlsAllowInvalidHostnames`      | 禁用[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/ [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例提供的证书的主机名验证。<br />设置为`true`连接到MongoDB实例，即使服务器证书中的主机名与服务器的主机不匹配。<br />并非所有驱动程序都支持此选项。请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。<br />此连接字符串选项不适用于`mongo` shell。请改用命令行选项。<br />警告<br />禁用证书验证会产生漏洞。<br />*4.2版中的新功能。* |
| `tlsInsecure`                   | 禁用各种证书验证。<br />设置为`true`禁用证书验证。禁用的确切验证因驱动程序而异。请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。<br />此连接字符串选项不适用于`mongo` shell。请改用命令行选项。<br />警告<br />禁用证书验证会产生漏洞。<br />*4.2版中的新功能。* |

##### 超时选项

| 连接选项           | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `connectTimeoutMS` | 超时之前尝试连接的时间（以毫秒为单位）。默认值是永不超时，尽管不同的驱动程序可能有所不同。请参阅[驱动程序](https://docs.mongodb.com/ecosystem/drivers) 文档。 |
| `socketTimeoutMS`  | 尝试超时之前在套接字上尝试发送或接收的时间（以毫秒为单位）。默认值是永不超时，尽管不同的驱动程序可能有所不同。请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。 |

##### 压缩选项

| 连接选项               | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| `compressors`          | 由逗号分隔的压缩器字符串，用于在此客户端和[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/ [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例之间进行通信时启用网络压缩。<br>您可以指定以下压缩器：<br/>[snappy](https://docs.mongodb.com/master/reference/glossary/#term-snappy)<br />[zlib](https://docs.mongodb.com/master/reference/glossary/#term-zlib)（在MongoDB 3.6或更高版本中可用）<br />[zstd](https://docs.mongodb.com/master/reference/glossary/#term-zstd)（在MongoDB 4.2或更高版本中可用）<br />如果指定多个压缩器，那么列出压缩器的顺序和通信启动器的顺序都很重要。例如，如果客户端指定了以下网络压缩器**“zlib,snappy”**，而[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)指定了**“snappy,zlib”**，那么客户端和[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)之间的消息就使用**zlib**。<br />重要<br />当双方都启用网络压缩时，消息将被压缩。否则，各方之间的消息将不被压缩。<br />如果各方不共享一个公共压缩器，则各方之间的消息将不被压缩。<br />从MongoDB 4.0.5（和MongoDB 3.6.10）开始, [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell支持uri连接字符串选项[`compressors`](https://docs.mongodb.com/master/reference/connection-string/#urioption.compressors)。 |
| `zlibCompressionLevel` | 如果使用[zlib](https://docs.mongodb.com/master/reference/glossary/#term-zlib)进行[`网络压缩`](https://docs.mongodb.com/master/reference/connection-string/#urioption.compressors)，则指定压缩级别的整数。<br />您可以指定一个从`-1`到`9`的整数值:：<br />值                                    笔记<br />`-1`                        默认压缩级别，通常是`6`级压缩。<br />`0`                          无压缩<br />`1` -- `9`                  增加压缩级别但以速度为代价，具有：<br />                                              `1` 提供最佳速度，但压缩最少，<br />                                              `9` 提供最佳压缩效果，但速度最慢。<br />不被[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell支持。 |

#### 连接池选项

大多数驱动程序实现某种类型的连接池处理。有些驱动程序不支持连接池。有关连接池实现的更多信息，请参见[驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。这些选项允许应用程序在连接到MongoDB部署时配置连接池。

| 连接选项             | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| `maxPoolSize`        | 连接池中的最大连接数。默认值为`100`。                        |
| `minPoolSize`        | 连接池中的最小连接数。默认值为`0`。<br />注意<br />并不是所有驱动程序都支持[`minPoolSize`](https://docs.mongodb.com/master/reference/connection-string/#urioption.minPoolSize)选项。有关驱动程序的信息，请参阅 [驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。 |
| `maxIdleTimeMS`      | 在删除和关闭连接之前，连接在池中可以保持空闲状态的最大毫秒数。<br />并非所有驱动程序都支持此选项。 |
| `waitQueueMultiple`  | 驱动程序将[`maxPoolSize`](https://docs.mongodb.com/master/reference/connection-string/#urioption.maxPoolSize) 值乘以一个数字，以提供允许等待池中的连接可用的最大线程数。有关默认值，请参见[/ drivers](https://docs.mongodb.com/ecosystem/drivers) 文档。<br />并非所有驱动程序都支持此选项。 |
| `waitQueueTimeoutMS` | 线程可以等待连接可用的最长时间（以毫秒为单位）。有关默认值，请参见 [/ drivers](https://docs.mongodb.com/ecosystem/drivers)文档。<br />并非所有驱动程序都支持此选项。 |

##### 写关注选项

[写关注](https://docs.mongodb.com/master/reference/write-concern/#write-concern)描述了MongoDB请求的确认级别。下列情况支持写关注选项：

- MongoDB驱动程序
- [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell
- [`mongofiles`](https://docs.mongodb.com/database-tools/mongofiles/#bin.mongofiles)
- [`mongoimport`](https://docs.mongodb.com/database-tools/mongoimport/#bin.mongoimport)
- [`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore)

您可以在连接字符串中指定写关注，也可以将其指定为诸如`insert`或`update`等方法的参数。如果在两个地方都指定了写关注点，则method参数将覆盖连接字符串设置。

下面的连接字符串到一个复制集指定 [`"majority"`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority")写关注和5秒超时使用[`wtimeoutMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.wtimeoutMS)写关注参数:

```powershell
mongodb：//db0.example.com,db1.example.com,db2.example.com/？replicaSet = myRepl＆w = majority＆wtimeoutMS = 5000
```

| 连接选项     | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| `w`          | 对应于写关注[w Option](https://docs.mongodb.com/master/reference/write-concern/#wc-w)。该`w`选项请求确认写操作已传播到指定数量的[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)实例或 [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)具有指定标签的实例。<br />您可以指定[`number`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern.)，字符串[`majority`](https://docs.mongodb.com/master/reference/write-concern/#writeconcern."majority")或`tag set`<br/>有关详细信息，请参见[w Option](https://docs.mongodb.com/master/reference/write-concern/#wc-w)。 |
| `wtimeoutMS` | 对应于写关注点[wtimeout](https://docs.mongodb.com/master/reference/write-concern/#wc-wtimeout). [`wtimeoutMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.wtimeoutMS)为写关注指定了一个时间限制，以毫秒为单位。<br />如果`wtimeoutMS`是`0`，写操作永远不会超时。有关更多信息，请参见[wtimeout](https://docs.mongodb.com/master/reference/write-concern/#wc-wtimeout)。 |
| `journal`    | 对应于写关注点[j Option](https://docs.mongodb.com/master/reference/write-concern/#wc-j)选项。该 [`journal`](https://docs.mongodb.com/master/reference/connection-string/#urioption.journal)选项要求MongoDB确认已将写操作写入 [日志](https://docs.mongodb.com/master/core/journaling/)。有关详细信息，请参见[j选项](https://docs.mongodb.com/master/reference/write-concern/#wc-j)。<br />如果设置[`journal`](https://docs.mongodb.com/master/reference/connection-string/#urioption.journal)为`true`，并指定[`w`](https://docs.mongodb.com/master/reference/connection-string/#urioption.w)小于1 的 值，则[`journal`](https://docs.mongodb.com/master/reference/connection-string/#urioption.journal)优先。<br />如果您将[`journal`](https://docs.mongodb.com/master/reference/connection-string/#urioption.journal)设置为true，并且 [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)未启用日志功能（如） [`storage.journal.enabled`](https://docs.mongodb.com/master/reference/configuration-options/#storage.journal.enabled)，则MongoDB将出错。 |

有关更多信息，请参见[写关注点](https://docs.mongodb.com/master/reference/write-concern/)。

##### `readConcern`选项

*版本3.2中的新特性*:对于WiredTiger存储引擎，MongoDB 3.2为复制集和复制集分片引入了readConcern选项。

[Read Concern](https://docs.mongodb.com/master/reference/read-concern/)允许客户端为从复制集读取选择隔离级别。

下面的复制集连接字符串指定 [`readConcernLevel=majority`](https://docs.mongodb.com/master/reference/connection-string/#urioption.readConcernLevel)：

```powershell
mongodb：//db0.example.com,db1.example.com,db2.example.com/？replicaSet = myRepl＆readConcernLevel = majority
```

| 连接选项           | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `readConcernLevel` | 隔离的程度。可以接受下列值之一:<br />[`local`](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local")<br />[`majority`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority")<br />[`linearizable`](https://docs.mongodb.com/master/reference/read-concern-linearizable/#readconcern."linearizable")<br />[`available`](https://docs.mongodb.com/master/reference/read-concern-available/#readconcern."available")<br />此连接字符串选项不适用于 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)shell。指定read关注点作为[特定操作的选项](https://docs.mongodb.com/master/reference/read-concern/#read-concern-operations)。 |

有关更多信息，请参见[读关注](https://docs.mongodb.com/master/reference/read-concern/)。

##### 阅读首选项选项

[读取首选项](https://docs.mongodb.com/master/core/read-preference/)描述了与[复制集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)相关的读取操作的行为。这些参数允许您在连接字符串中以每个连接为基础指定读取首选项。

> 注意
>
> 要使用驱动程序指定已对冲的读取选项，请参考 [驱动程序的读取首选项API](https://docs.mongodb.com/drivers/drivers)。

例如：

- 以下到复制集的连接字符串指定 [`secondary`](https://docs.mongodb.com/master/core/read-preference/#secondary)读取首选项模式和[`maxStalenessSeconds`](https://docs.mongodb.com/master/reference/connection-string/#urioption.maxStalenessSeconds)120秒的值：

  ```powershell
mongodb：//db0.example.com,db1.example.com,db2.example.com/？replicaSet = myRepl＆readPreference = secondary＆maxStalenessSeconds = 120
  ```
  
- 以下到分片群集的连接字符串指定 [`secondary`](https://docs.mongodb.com/master/core/read-preference/#secondary)读取首选项模式和[`maxStalenessSeconds`](https://docs.mongodb.com/master/reference/connection-string/#urioption.maxStalenessSeconds)120秒的值：

  ```powershell
mongodb：//mongos1.example.com,mongos2.example.com/？readPreference = secondary＆maxStalenessSeconds = 120
  ```
  
- 以下到分片群集的连接字符串指定 [`secondary`](https://docs.mongodb.com/master/core/read-preference/#secondary)读取首选项模式以及三种 [`readPreferenceTags`](https://docs.mongodb.com/master/reference/connection-string/#urioption.readPreferenceTags)：

  ```powershell
mongodb：//mongos1.example.com,mongos2.example.com/？readPreference = secondary＆readPreferenceTags = dc：ny，rack：r1＆readPreferenceTags = dc：ny＆readPreferenceTags =
  ```

使用多个`readPreferenceTags`时，顺序很重要。按顺序尝试`readPreferenceTags`，直到找到匹配项为止。一旦找到，该规范将用于查找所有符合条件的匹配成员，并忽略任何剩余的`readPreferenceTags`。有关详细信息，请参见[标签匹配顺序](https://docs.mongodb.com/master/core/read-preference-tags/#read-preference-tag-order-matching)。

| 连接选项              | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| `readPreference`      | 指定此连接的[读取首选项](https://docs.mongodb.com/master/core/read-preference/)。可能的值为：<br />[`primary`](https://docs.mongodb.com/master/core/read-preference/#primary)（*默认*）<br />[`primaryPreferred`](https://docs.mongodb.com/master/core/read-preference/#primaryPreferred)<br />[`secondary`](https://docs.mongodb.com/master/core/read-preference/#secondary)<br />[`secondaryPreferred`](https://docs.mongodb.com/master/core/read-preference/#secondaryPreferred)<br />[`nearest`](https://docs.mongodb.com/master/core/read-preference/#nearest)<br />包含读取操作的[多文档事务](https://docs.mongodb.com/master/core/transactions/)必须使用读取首选项[`primary`](https://docs.mongodb.com/master/core/read-preference/#primary)。给定事务中的所有操作必须路由到同一成员。<br />此连接字符串选项不适用于 `mongo`shell。<br />请参阅[`cursor.readPref()`](https://docs.mongodb.com/master/reference/method/cursor.readPref/#cursor.readPref)和 [`Mongo.setReadPref()`](https://docs.mongodb.com/master/reference/method/Mongo.setReadPref/#Mongo.setReadPref)。 |
| `maxStalenessSeconds` | 指定以秒为单位的秒数，表示客户机在停止将其用于读取操作之前会过时。有关详细信息，请参见 [阅读首选项maxStalenessSeconds](https://docs.mongodb.com/master/core/read-preference-staleness/#replica-set-read-preference-max-staleness)。<br />默认情况下，没有最大的过时度，客户机在选择将读操作指向何处时不会考虑辅助服务器的延迟。<br />最小值[`maxStalenessSeconds`](https://docs.mongodb.com/master/reference/connection-string/#urioption.maxStalenessSeconds)为90秒。指定0到90秒之间的值将产生错误。MongoDB驱动程序将`maxStalenessSeconds`值`-1`视为“没有最大过时性”，就好像`maxStalenessSeconds`被忽略了一样 。<br />重要<br />要使用`maxStalenessSeconds`，部署中的所有MongoDB实例都必须使用MongoDB 3.4或更高版本。如果任何实例在MongoDB的早期版本上，则驱动程序或[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/ [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)将引发错误。<br />*3.4版的新功能。* |
| `readPreferenceTags`  | 将[标签文档](https://docs.mongodb.com/master/core/read-preference-tags/#replica-set-read-preference-tag-sets)指定为以冒号分隔的键/值对的列表，以逗号分隔。例如，<br />要指定标签文档`{"dc": "ny"， "rack": "r1"}`，在连接字符串中使用`readPreferenceTags=dc:ny,rack:r1。`<br /> 若要指定空标记文档`{}`，请使用**readPreferenceTags=**而不设置值。<br />要指定标签文档列表，请使用多个`readPreferenceTags`。例如，`readPreferenceTags=dc:ny,rack:r1&readPreferenceTags=`.<br />使用多个`readPreferenceTags`时，顺序很重要。按顺序尝试readPreferenceTags，直到找到匹配项为止。有关详细信息，请参见[标记匹配的顺序](https://docs.mongodb.com/master/core/read-preference-tags/#read-preference-tag-order-matching)。<br />这个连接字符串选项对mongo shell不可用。请参阅[`cursor.readPref()`](https://docs.mongodb.com/master/reference/method/cursor.readPref/#cursor.readPref)和 [`Mongo.setReadPref()`](https://docs.mongodb.com/master/reference/method/Mongo.setReadPref/#Mongo.setReadPref)。 |

有关更多信息，请参阅[阅读首选项](https://docs.mongodb.com/master/core/read-preference/)。

##### 验证选项

下面到复制集的连接字符串指定`admin`数据库的 [`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)。也就是说，根据`admin`数据库对用户凭据进行身份验证。

```powershell
mongodb：// myDBReader：D1fficultP%40ssw0rd@mongodb0.example.com：27017，mongodb1.example.com：27017，mongodb2.example.com：27017 /？replicaSet = myRepl ＆authSource = admin
```

如果用户名或密码包含'at'符号`@`，冒号`:`，斜杠`/`或百分号`%`字符，请使用[百分比编码](https://tools.ietf.org/html/rfc3986#section-2.1)。

| 连接选项                  | 描述                                                         |
| :------------------------ | :----------------------------------------------------------- |
| `authSource`              | 指定与用户凭据关联的数据库名称。如果[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)未指定，则 [`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)默认为`defaultauthdb` 连接字符串中指定的。如果`defaultauthdb`未指定，则[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)默认为`admin`。<br />普通身份验证机制(LDAP)、`GSSAPI `(Kerberos)和`MONGODB-AWS (IAM)`要求将[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)设置为`$external`，因为这些机制将凭据存储委托给外部服务。<br />如果在连接字符串中或通过**--username**参数中没有提供用户名，MongoDB将忽略[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)值。 |
| `authMechanism`           | 指定MongoDB将用于认证连接的认证机制。可能的值包括：<br />[SCRAM-SHA-1](https://docs.mongodb.com/master/core/security-scram/#authentication-scram-sha-1)<br />[SCRAM-SHA-256](https://docs.mongodb.com/master/core/security-scram/#authentication-scram-sha-256)（ *MongoDB 4.0中添加*）<br />[MONGODB-X509](https://docs.mongodb.com/master/core/security-x.509/#security-auth-x509)<br />`MONGODB-AWS`（*在MongoDB 4.4中添加了*）<br />[GSSAPI](https://docs.mongodb.com/master/core/authentication-mechanisms-enterprise/#security-auth-kerberos)（Kerberos）<br />[普通](https://docs.mongodb.com/master/core/authentication-mechanisms-enterprise/#security-auth-ldap)（LDAP SASL）<br />MongoDB 4.0删除了对`MONGODB-CR` 身份验证机制的支持。`MONGODB-CR`连接到MongoDB 4.0+部署时，不能指定为身份验证机制。<br />仅MongoDB Enterprise [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和 [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例提供`GSSAPI`（Kerberos）和 `PLAIN`（LDAP）机制。<br />要使用`MONGODB-X509`，您必须启用TLS / SSL。<br />要使用`MONGODB-AWS`，您必须连接到已配置为支持通过[AWS IAM凭证](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) （即AWS访问密钥ID和秘密访问密钥，以及可选的[AWS会话令牌](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)）进行身份验证的 [MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server)集群 。该认证机制需要 设置为。`MONGODB-AWS`[`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)`$external`使用时`MONGODB-AWS`，请提供您的AWS访问密钥ID作为用户名，并提供秘密访问密钥作为密码。如果还使用 [AWS会话令牌](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) ，请为其提供`AWS_SESSION_TOKEN` [`authMechanismProperties`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authMechanismProperties)值。<br />如果AWS访问密钥ID，秘密访问密钥或会话令牌包含'at'符号`@`，冒号`:`，斜杠 `/`或百分号`%`字符，则必须使用[百分比编码](https://tools.ietf.org/html/rfc3986#section-2.1)转换这些字符。<br />或者，如果您使用各自的[AWS IAM环境变量](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html#envvars-list) 在平台上定义了AWS访问密钥ID，秘密访问密钥或会话令牌，则 [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)Shell将使用这些环境变量值进行身份验证；您无需在连接字符串中指定它们。有关同时使用连接字符串和环境变量方法的身份验证机制的用法，请参阅[连接到Atlas群集](https://docs.mongodb.com/master/reference/connection-string/#connections-string-example-mongodb-aws)`MONGODB-AWS`。有关MongoDB中身份验证系统的更多信息，请参阅[身份](https://docs.mongodb.com/master/core/authentication/)验证。另请考虑 [使用x.509证书对客户端](https://docs.mongodb.com/master/tutorial/configure-x509-client-authentication/)进行[身份验证](https://docs.mongodb.com/master/tutorial/configure-x509-client-authentication/)，以获取有关x509身份验证的更多信息。 |
| `authMechanismProperties` | 将指定的属性指定[`authMechanism`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authMechanism) 为以逗号分隔的冒号分隔的键/值对列表。可能的键值对为：`SERVICE_NAME:<string>`连接到Kerberized MongoDB实例时，设置Kerberos服务名称。该值必须与您要连接的MongoDB实例上设置的服务名称匹配。仅在使用[GSSAPI](https://docs.mongodb.com/master/core/authentication-mechanisms-enterprise/#security-auth-kerberos) 身份验证机制时有效。`SERVICE_NAME``mongodb`所有客户端和MongoDB实例默认为。如果更改[`saslServiceName`](https://docs.mongodb.com/master/reference/parameters/#param.saslServiceName)MongoDB实例上的 设置，则必须进行设置`SERVICE_NAME`以匹配该设置。仅在使用[GSSAPI](https://docs.mongodb.com/master/core/authentication-mechanisms-enterprise/#security-auth-kerberos) 身份验证机制时有效。`CANONICALIZE_HOST_NAME:true|false`连接到Kerberos服务器时，规范化客户端主机的主机名。当主机报告的主机名与Kerberos数据库中的主机名不同时，可能需要这样做。默认为`false`。仅在使用[GSSAPI](https://docs.mongodb.com/master/core/authentication-mechanisms-enterprise/#security-auth-kerberos)身份验证机制时有效 。`SERVICE_REALM:<string>`为MongoDB服务设置Kerberos领域。这对于支持跨域身份验证可能是必需的，在该跨域身份验证中，用户位于一个领域中，而服务位于另一个领域中。仅在使用[GSSAPI](https://docs.mongodb.com/master/core/authentication-mechanisms-enterprise/#security-auth-kerberos)身份验证机制时有效。`AWS_SESSION_TOKEN:<security_token>`在使用[AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) 请求或使用指定该值的AWS资源（例如Lambda）时，设置AWS会话令牌以使用临时凭证进行身份验证。仅在使用`MONGODB-AWS` 身份验证机制时有效。您还必须具有一个AWS访问密钥ID和一个秘密访问密钥。有关示例用法，请参见 [连接到Atlas群集](https://docs.mongodb.com/master/reference/connection-string/#connections-string-example-mongodb-aws)。 |
| `gssapiServiceName`       | 连接到Kerberized MongoDB实例时，设置Kerberos服务名称。该值必须与您要连接的MongoDB实例上设置的服务名称匹配。[`gssapiServiceName`](https://docs.mongodb.com/master/reference/connection-string/#urioption.gssapiServiceName)`mongodb`所有客户端和MongoDB实例默认为。如果更改 [`saslServiceName`](https://docs.mongodb.com/master/reference/parameters/#param.saslServiceName)MongoDB实例上的设置，则必须进行设置[`gssapiServiceName`](https://docs.mongodb.com/master/reference/connection-string/#urioption.gssapiServiceName)以匹配该设置。[`gssapiServiceName`](https://docs.mongodb.com/master/reference/connection-string/#urioption.gssapiServiceName)是不推荐使用的别名 [`authMechanismProperties=SERVICE_NAME:mongodb`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authMechanismProperties)。有关驱动程序支持哪些选项以及它们之间的相对优先级的更多信息，请参考首选驱动程序版本的文档。 |

### 服务器选择和查找选项

MongoDB提供以下选项来配置MongoDB驱动程序和[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例如何选择要将读取或写入操作定向到的服务器。

| 连接选项                   | 描述                                                         |
| :------------------------- | :----------------------------------------------------------- |
| `localThresholdMS`         | 在多个合适的MongoDB实例中进行选择的等待时间窗口的大小（以毫秒为单位）。*默认值*：15毫秒。所有驱动程序都使用[`localThresholdMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.localThresholdMS)。`localThreshold`将延迟窗口大小指定为时，请使用 别名[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)。 |
| `serverSelectionTimeoutMS` | 指定在引发异常之前为选择服务器而阻塞的时间（以毫秒为单位）。*默认值*：30,000毫秒。 |
| `serverSelectionTryOnce`   | **仅单线程驱动程序**。如果为`true`，则指示驱动程序在服务器选择失败后立即扫描MongoDB部署一次，然后选择服务器或引发错误。当为时`false`，驱动程序将阻止并搜索不超过该[`serverSelectionTimeoutMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.serverSelectionTimeoutMS)值的服务器。 *默认值*：`true`。并且[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)不支持 多线程驱动程序[`serverSelectionTryOnce`](https://docs.mongodb.com/master/reference/connection-string/#urioption.serverSelectionTryOnce)。 |
| `heartbeatFrequencyMS`     | [`heartbeatFrequencyMS`](https://docs.mongodb.com/master/reference/connection-string/#urioption.heartbeatFrequencyMS)控制驱动程序何时检查MongoDB部署的状态。指定两次检查之间的间隔（以毫秒为单位），从上一次检查的结束到下一次检查的开始计算。*默认值*：单线程驱动程序：60秒。多线程驱动程序：10秒。[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) 不支持更改心跳检查的频率。 |

### 杂项配置

| 连接选项             | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| `appName`            | 指定自定义应用名称。应用名称出现在[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[日志](https://docs.mongodb.com/master/reference/log-messages/)，[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)命令和方法输出中的[`currentOp.appName`](https://docs.mongodb.com/master/reference/command/currentOp/#currentOp.appName)字段，[`currentOp`](https://docs.mongodb.com/master/reference/command/currentOp/#dbcmd.currentOp)[`db.currentOp()`](https://docs.mongodb.com/master/reference/method/db.currentOp/#db.currentOp)[数据库探查器](https://docs.mongodb.com/master/reference/database-profiler/)输出中的[`system.profile.appName`](https://docs.mongodb.com/master/reference/database-profiler/#system.profile.appName)字段。如果您未指定自定义应用程序名称，则[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) 外壳程序将使用默认的“ ”。`MongoDB Shell`*版本4.0中的新功能。* |
| `retryReads`         | 启用可[重试的读取](https://docs.mongodb.com/master/core/retryable-reads/#retryable-reads)。可能的值为：`true`。启用连接的可重试读取。与MongoDB Server 4.2及更高版本兼容的官方MongoDB驱动程序默认为`true`。`false`。禁用连接的可重试读取。在[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)外壳不支持重试读取。*4.2版中的新功能。* |
| `retryWrites`        | 启用可[重试写入](https://docs.mongodb.com/master/core/retryable-writes/#retryable-writes)。可能的值为：`true`。启用连接的可重试写入。兼容MongoDB 4.2的官方驱动程序默认为`true`。`false`。禁用该连接的可重试写入。官方的MongoDB 4.0和3.6兼容驱动程序默认为`false`。MongoDB驱动程序将重试 [事务提交和中止操作，](https://docs.mongodb.com/master/core/transactions-in-applications/#transactions-retry) 而与的值无关[`retryWrites`](https://docs.mongodb.com/master/reference/connection-string/#urioption.retryWrites)。有关事务可重试性的更多信息，请参见 [事务错误处理](https://docs.mongodb.com/master/core/transactions-in-applications/#transactions-retry)。*3.6版的新功能。* |
| `uuidRepresentation` | 可能的值为：`standard`标准二进制表示形式。`csharpLegacy`C＃驱动程序的默认表示。`javaLegacy`Java驱动程序的默认表示形式。`pythonLegacy`Python驱动程序的默认表示形式。对于默认设置，请参阅[驱动程序的驱动程序](https://docs.mongodb.com/ecosystem/drivers) 文档。注意并非所有驱动程序都支持该[`uuidRepresentation`](https://docs.mongodb.com/master/reference/connection-string/#urioption.uuidRepresentation) 选项。有关驱动程序的信息，请参阅[驱动程序](https://docs.mongodb.com/ecosystem/drivers)文档。 |



## 例子

以下提供了用于公共连接目标的示例URI字符串。

### 在本地运行的数据库服务器

以下连接到在默认端口上本地运行的数据库服务器：

```
mongodb：//本地主机
```

### `admin`数据库

以下内容`admin`以用户身份`sysop`使用密码连接并登录到数据库 `moon`：

```
mongodb：// sysop：moon @ localhost
```

### `records`数据库

以下内容`records`以用户身份`sysop`使用密码连接并登录到数据库 `moon`：

```
mongodb：// sysop：moon @ localhost / records
```

### UNIX域套接字

连接到UNIX域套接字时，请使用URL编码的连接字符串。

以下连接到具有文件路径的UNIX域套接字 `/tmp/mongodb-27017.sock`：

```
mongodb：//%2Ftmp%2Fmongodb-27017.sock
```

注意

并非所有驱动程序都支持UNIX域套接字。有关驱动程序的信息，请参阅[驱动程序](https://docs.mongodb.com/ecosystem/drivers) 文档。

### 在不同计算机上具有成员的副本集

以下内容连接到具有两个成员的[副本集](https://docs.mongodb.com/master/reference/glossary/#term-replica-set)，一个成员`db1.example.net`在另一个成员 上`db2.example.net`：

注意

对于副本集，请指定[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 副本集配置中列出的实例的主机名。

```
mongodb：//db1.example.net,db2.example.com/？replicaSet = test
```

### 带有成员的副本集`localhost`

下面连接到副本集具有三个成员上运行`localhost`的端口`27017`，`27018`以及`27019`：

注意

对于副本集，请指定[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 副本集配置中列出的实例的主机名。

```
mongodb：//本地主机，本地主机：27018，本地主机：27019 /？replicaSet = test
```

### 具有读取分布的副本集

以下内容连接到具有三个成员的副本集，并将读取内容分发给[第二](https://docs.mongodb.com/master/reference/glossary/#term-secondary)副本：

注意

对于副本集，请指定[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 副本集配置中列出的实例的主机名。

```
mongodb：//example1.com,example2.com,example3.com/？replicaSet = test＆readPreference = secondary
```

### 具有写关注级别的副本集

以下内容连接到具有写关注点的副本集，该副本集被配置为等待跨大多数数据承载投票成员的复制成功，并具有两秒的超时。

注意

对于副本集，请指定[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) 副本集配置中列出的实例的主机名。

```
mongodb：//example1.com,example2.com,example3.com/？replicaSet = test＆w = majority＆wtimeoutMS = 2000
```

### 分片群集

以下连接到具有三个[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例的分片群集：

```
mongodb：//router1.example.com：27017，router2.example2.com：27017，router3.example3.com：27017 /
```



### MongoDB Atlas集群

*版本4.4中的新功能。*

以下连接到已配置为支持通过[AWS IAM凭证进行](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)身份验证的[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server)集群：

```
mongo'mongodb + srv：// <aws访问密钥ID>：<aws秘密访问密钥> @ cluster0.example.com / testdb？authSource = $ external＆authMechanism = MONGODB-AWS'
```

如本示例所示，以这种方式使用AWS IAM凭据连接到Atlas使用 和和。`MONGODB-AWS` [``$external` [`authSource`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authSource)

如果还使用[AWS会话令牌](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)，请为其提供`AWS_SESSION_TOKEN` [`authMechanismProperties`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authMechanismProperties)值，如下所示：

```
mongo'mongodb + srv：// <aws访问密钥ID>：<aws秘密访问密钥> @ cluster0.example.com / testdb？authSource = $ external＆authMechanism = MONGODB-AWS＆authMechanismProperties = AWS_SESSION_TOKEN：<aws会话令牌>'
```

如果AWS访问密钥ID，秘密访问密钥或会话令牌包括'at'符号`@`，冒号`:`，斜杠`/`或百分号`%`字符，则必须使用[百分比编码](https://tools.ietf.org/html/rfc3986#section-2.1)转换这些字符 。

您也可以使用标准[AWS IAM环境变量](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html#envvars-list)在平台上设置这些凭证 。使用[`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo)以下命令时，shell将检查以下环境变量：`MONGODB-AWS` [`authentication mechanism`](https://docs.mongodb.com/master/reference/connection-string/#urioption.authMechanism)

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_SESSION_TOKEN`

如果设置，则无需在连接字符串中指定这些凭据。

以下示例在`bash` 外壳中设置这些环境变量：

```
导出AWS_ACCESS_KEY_ID ='<aws访问密钥ID>'
导出AWS_SECRET_ACCESS_KEY ='<aws秘密访问密钥>'
导出AWS_SESSION_TOKEN ='<aws会话令牌>'
```

在其他shell中设置环境变量的语法将有所不同。有关更多信息，请查阅您平台的文档。

您可以使用以下命令验证是否已设置这些环境变量：

```
env | grep AWS
```

设置完成后，以下示例将使用以下环境变量连接到MongoDB Atlas集群：

```
mongo'mongodb + srv：//cluster0.example.com/testdb？authSource = $ external＆authMechanism 
```


