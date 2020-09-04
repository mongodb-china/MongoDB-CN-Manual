# Configuration File Options

在本页面

- [配置文件](#configuration-file)
  - [文件格式](#file-format)
  - [配置文件的使用](#use-the-configuration-file)
- [核心选项](#core-options)
  - [`systemLog` 选项](#systemlog-options)
  - [`processManagement` 选项](#processmanagement-options)
  - [`cloud` 选项](#cloud-options)
  - [`net` 选项](#net-options)
  - [`security` 选项](#security-options)
  - [`setParameter` 选项](#setparameter-option)
  - [`storage` 选项](#storage-options)
  - [`operationProfiling` 选项](#operationprofiling-options)
  - [`replication` 选项](#replication-options)
  - [`sharding` 选项](#sharding-options)
  - [`auditLog` 选项](#auditlog-options)
  - [`snmp` 选项](#snmp-options)
- [`mongos`-only 选项](#mongos-only-options)
- [Windows Service 选项](#windows-service-options)
- [Removed MMAPv1 选项](#removed-mmapv1-options)

下面的页面描述了MongoDB 4.4中可用的配置选项。有关其他版本MongoDB的配置文件选项，请参阅相应版本的MongoDB手册。



## <span id="configuration-file">配置文件</span>

您可以在启动时使用配置文件配置[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) 实例。配置文件包含的设置相当于[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)命令行选项。参见[配置文件设置和命令行选项映射](https://docs.mongodb.com/master/reference/configuration-file-settings-command-line-options-mapping/#conf-file-command-line-mapping)。

使用配置文件使管理[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)选项更容易，特别是对于大规模部署。您还可以向配置文件添加注释，以解释服务器的设置。

> **[success] 默认配置文件**
>
> * 在Linux上，当使用包管理器安装MongoDB时，会包含一个默认的`/etc/mongod.conf`配置文件。
> * 在Windows上，安装过程中会包含一个默认的`<install directory>/bin/mongod.cfg` 配置文件。
> * 在macOS上，当从MongoDB的官方自制程序安装时，会包含一个默认的`/usr/local/etc/mongod.conf`配置文件。

### <span id="file-format">文件格式</span>

MongoDB配置文件使用[YAML](http://www.yaml.org/)格式[[1\]](https://docs.mongodb.com/master/reference/configuring-options/# yaml-json)。

以下示例配置文件包含几个[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)设置，您可以适应您的本地配置:

> **[success] 注意**
>
> YAML不支持制表符缩进：使用空格代替。

```powershell
systemLog:
   destination: file
   path: "/var/log/mongodb/mongod.log"
   logAppend: true
storage:
   journal:
      enabled: true
processManagement:
   fork: true
net:
   bindIp: 127.0.0.1
   port: 27017
setParameter:
   enableLocalhostAuthBypass: false
...
```

Linux包init脚本包含在官方MongoDB包依赖于特定的值[`systemLog.path`](https://docs.mongodb.com/master/reference/configuration-options/ # systemLog.path)，[`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/ # storage.dbPath)，和[`processManagement.fork`](https://docs.mongodb.com/master/reference/configuration-options/ # processManagement.fork)。如果您在默认配置文件中修改这些设置，[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)可能不会启动。

| <br />                                                       |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [[1\]](https://docs.mongodb.com/master/reference/configuration-options/#id1) | YAML是[JSON](https://docs.mongodb.com/master/reference/glossary/#term-json)的超集。 |

#### 外部来源的值

*4.2版本中的新功能*： MongoDB支持在配置文件中使用[扩展指令](https://docs.mongodb.com/master/reference/expans-directives/ # expan- directives)来加载外部源值。扩展指令可以加载特定的[配置文件选项](https://docs.mongodb.com/master/reference/configuring-options/#configuring-options) *或*加载整个配置文件。

可以使用以下扩展指令:

| 扩展指令                                                     | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) | 允许用户指定一个`REST`端点作为配置文件选项*或*完整配置文件的外部源。<br />如果配置文件包括[`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/ # configexpansion.__rest)扩展,在Linux / macOS对配置文件的读访问必须是仅限于运行[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ bin.mongos)进程的用户。 |
| [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) | 允许用户指定一个shell或终端命令作为配置文件选项*或*完整配置文件的外部源。<br />如果配置文件包括[`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/ # configexpansion.__exec)扩展,在Linux / macOS对配置文件的写访问必须是仅限于运行[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ bin.mongos)过程的用户。 |

要获得完整的文档，请参见[外部来源的配置文件值](https://docs.mongodb.com/master/reference/expans-directives/ #external -sourced-values)。

### <span id="use-the-configuration-file">配置文件的使用</span>

使用配置文件配置[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)，使用`--config`选项或`-f`选项指定一个或多个配置文件，如下例所示:

例如，下面使用 [`mongod --config <configuration file>`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-config) [`mongos --config <configuration file>`](https://docs.mongodb.com/master/reference/program/mongos/#cmdoption-mongos-config):

```powershell
mongod --config /etc/mongod.conf

mongos --config /etc/mongos.conf
```

您还可以使用`-f`别名来指定配置文件，如下所示：

```powershell
mongod -f /etc/mongod.conf

mongos -f /etc/mongos.conf
```

如果您从包中安装并使用系统的[init脚本](https://docs.mongodb.com/master/reference/glossary/#term-init-script)启动了MongoDB，那么您已经使用了一个配置文件。

#### 扩展指令和 `--configExpand`

如果您正在使用[扩展指令](https://docs.mongodb.com/master/reference/expansion-directives/expansion-directives)配置文件,您必须包括[`——configExpand`](https://docs.mongodb.com/master/reference/program/mongod/ cmdoption-mongod-configexpand)选项时启动[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ # bin.mongos)。例如:

```
mongod --config /etc/mongod.conf  --configExpand "rest,exec"
mongos --config /etc/mongos.conf  --configExpand "rest,exec"
```

如果配置文件包括一个扩展指令启动[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod) /[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ # bin.mongos)没有指定的指令[`——configExpand`](https://docs.mongodb.com/master/reference/program/mongod/ # cmdoption-mongod-configexpand)选项,[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)启动失败。

要获得完整的文档，请参见[外部来源的配置文件值](https://docs.mongodb.com/master/reference/expans-directives/ #external -sourced-values)。

## <span id="core-options">核心选项</span>



### <span id="systemlog-options">`systemLog` 选项</span>

```
systemLog:
   verbosity: <int>
   quiet: <boolean>
   traceAllExceptions: <boolean>
   syslogFacility: <string>
   path: <string>
   logAppend: <boolean>
   logRotate: <string>
   destination: <string>
   timeStampFormat: <string>
   component:
      accessControl:
         verbosity: <int>
      command:
         verbosity: <int>

      # COMMENT additional component verbosity settings omitted for brevity
```

- `systemLog.verbosity`

  *Type*：integer

  *Default*：0
  [components](https://docs.mongodb.com/master/reference/log-messages/#log-message-components)的默认[log message](https://docs.mongodb.com/master/reference/log-messages/)详细程度级别。详细级别决定了MongoDB输出的[Informational and Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息的数量。[[2\]](https://docs.mongodb.com/master/reference/configuration-options/#log-message)详细程度可以从`0`到`5`：

  * `0`是MongoDB的默认日志详细程度，包括[Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息。
  * `1`到`5`增加了包含[Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息的详细级别。

  若要为命名组件使用不同的详细级别，请使用组件的详细设置。例如，使用[`systemLog.component.accessControl.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.accessControl.verbosity)为[`ACCESS`](https://docs.mongodb.com/master/reference/log-messages/#ACCESS)组件设置详细级别。

  请参阅`systemLog.component.<name>.verbosity`设置以获得特定组件的详细设置。

  有关设置日志详细级别的各种方法，请参阅[配置日志详细级别](https://docs.mongodb.com/master/reference/log-messages/#log-messages-configure-verbosity)。

  | <br />                                                       |                                                              |
  | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | [[2\]](https://docs.mongodb.com/master/reference/configuration-options/#id3) | 从4.2版本开始，MongoDB在[log messages](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)中包含了调试详细级别(1-5)。例如，如果详细级别是2,MongoDB记录的日志是`D2`。在以前的版本中，MongoDB日志消息只指定`D`作为调试级别。 |

- `systemLog.quiet`

  *Type*: boolean

  运行[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/ bin.mongos)或[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ # bin.mongod)在一个安静的模式,试图限制输出。[`systemLog.quiet`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.quiet) *不*推荐给生产系统,因为它可能使跟踪问题特定的连接更加困难。

- `systemLog.traceAllExceptions`

  *Type*: boolean

  打印详细信息以便调试。用于附加日志，用于与支持相关的故障排除。

- `systemLog.syslogFacility`

  *Type*: string

  *Default*: user

  将消息记录到syslog时使用的设施级别。您指定的值必须由您的操作系统的syslog实现支持。要使用此选项，必须将[`systemLog.destination`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.destination)设置为`syslog`。

- `systemLog.path`

  *Type*: string

  日志文件的路径,[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)应该发送诊断日志记录所有信息,而不是标准输出或主机的[syslog](https://docs.mongodb.com/master/reference/glossary/#term-syslog)。MongoDB在指定的路径上创建日志文件。

  Linux软件包的初始化脚本不希望[`systemLog.path`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.path)更改默认值。如果您使用Linux包并更改[`systemLog.path`](https://docs.mongodb.com/master/reference/configuring-options/#systemlog.path)，您将不得不使用自己的init脚本并禁用内置脚本。

- `systemLog.logAppend`

  *Type*: boolean

  *Default*: false

  当为`true`的时候，[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)或[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)追加新的条目到现有的日志文件时，结束[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)或[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod) 实例重新启动。如果没有此选项，[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)将备份现有日志并创建一个新文件。

- `systemLog.logRotate`

  *Type*: string

  *Default*: rename

  命令的行为。指定`rename`或`reopen`：

  * `rename` 重命名日志文件。

  * `reopen`按照典型的Linux / Unix日志轮换行为，关闭并重新打开日志文件。使用`reopen`的Linux / Unix logrotate的工具，以避免日志丢失时。

    如果指定`reopen`，则还必须设置[`systemLog.logAppend`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.logAppend)为`true`。

- `systemLog.destination`

  *Type*: string

  MongoDB将所有日志输出发送到的目标。指定 `file`或`syslog`。如果指定`file`，则还必须指定 [`systemLog.path`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.path)。

  如果未指定[`systemLog.destination`](https://docs.mongodb.com/manual/reference/configuration-options/#systemLog.destination)，则MongoDB将所有日志输出发送到标准输出。

  > **[warning] 警告**
  >
  > `syslog`守护进程在记录消息时生成时间戳，而不是在MongoDB发出消息时生成。这可能会导致日志条目的时间戳出现错误，特别是在系统处于高负载时。我们建议在生产系统中使用`file`选项，以确保准确的时间戳。

- `systemLog.timeStampFormat`

  *Type*: string
  
  *Default*: iso8601-local
  
  日志消息中时间戳的时间格式。指定以下值之一:
  
  | 值              | 描述                                                         |
  | --------------- | ------------------------------------------------------------ |
  | `iso8601-utc`   | 以ISO-8601格式的协调世界时(UTC)显示时间戳。例如，对于纪元之初的纽约:`1970-01-01T00:00:00.000Z` |
  | `iso8601-local` | 以ISO-8601格式本地时间显示时间戳。例如，对于纪元初期的纽约:`1969-12-31T19:00:00.000-05:00` |
  
  > **[success] 注意**
  >
  > 从MongoDB 4.4开始，[`systemLog.timeStampFormat`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.timeStampFormat)不再支持`ctime`。`ctime`格式化的日期的一个例子是:`Wed Dec 31 18:17:54.811`。

#### `systemLog.component` 选项

```
systemLog:
   component:
      accessControl:
         verbosity: <int>
      command:
         verbosity: <int>

      # COMMENT some component verbosity settings omitted for brevity

      replication:
         verbosity: <int>
         election:
            verbosity: <int>
         heartbeats:
            verbosity: <int>
         initialSync:
            verbosity: <int>
         rollback:
            verbosity: <int>
      storage:
         verbosity: <int>
         journal:
            verbosity: <int>
         recovery:
            verbosity: <int>
      write:
         verbosity: <int>
```

> **[success] 注意**
>
> 从4.2版本开始，MongoDB在[log messages](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)中包含了调试详细级别(1-5)。例如，如果冗余级别是2,MongoDB记录的日志是`D2`。在以前的版本中，MongoDB日志消息只指定`D`作为调试级别。

- `systemLog.component.accessControl.verbosity`

  *Type*: integer

  *Default*: 0

  与访问控制相关的组件的日志消息详细程度级别。查看[`ACCESS`](https://docs.mongodb.com/master/reference/log-messages/#ACCESS)组件。

  详细程度可以从`0`到`5`:

  * `0`是MongoDB的默认日志冗余级别，用于包含 [Informational](https://docs.mongodb.com/manual/reference/log-messages/#log-severity-levels)信息。
  * `1`到`5`增加了包含[Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息的冗余级别。

- `systemLog.component.command.verbosity`

  *Type*: integer

  *Default*: 0

  与命令相关的组件的日志消息详细级别。查看[`COMMAND`](https://docs.mongodb.com/manual/reference/log-messages/#COMMAND)组件。

  详细程度的范围为`0`到`5`：

  * `0`是MongoDB的默认日志级别，包括  [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) 信息。
  * `1`到`5`增加详细级别，以包括 [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息。

- `systemLog.component.control.verbosity`

  *Type*: integer

  *Default*: 0

  与控制操作相关的组件的日志消息详细级别。查看[`CONTROL`](https://docs.mongodb.com/manual/reference/log-messages/#CONTROL)组件。

  详细程度的范围为`0`到`5`：

  * `0`是MongoDB的默认日志级别，包括[Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)信息。
  * `1`到`5`增加详细级别，以包括 [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息。

- `systemLog.component.ftdc.verbosity`

  *Type*: integer

  *Default*: 0

  *version 3.2中的新功能。*

  与诊断数据收集操作相关的组件的日志消息详细级别。查看[`FTDC`](https://docs.mongodb.com/manual/reference/log-messages/#FTDC)组件。

  详细程度的范围为`0`到`5`：

  * `0`是MongoDB的默认日志级别，包括 [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)信息。
  * `1`到`5`增加详细级别，以包括 [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息。

- `systemLog.component.geo.verbosity`

  *Type*: integer

  *Default*: 0

  与地理空间解析操作相关的组件的日志消息详细级别。查看[`GEO`](https://docs.mongodb.com/manual/reference/log-messages/#GEO)组件。

  详细程度的范围为`0`到`5`：

  * `0`是MongoDB的默认日志级别，包括[Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)信息。
  * `1`到`5`增加详细级别，以包括[Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)信息。

- `systemLog.component.index.verbosity`

  *Type*: integer

  *Default*: 0

  与索引操作相关的组件的日志消息详细级别。查看[`INDEX`](https://docs.mongodb.com/manual/reference/log-messages/#INDEX)组件。

  详细程度的范围为`0`到`5`：

  * `0`是MongoDB的默认日志级别，包括 [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) 消息。
  * `1`到`5`增加详细级别，以包括[Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) 消息。

- `systemLog.component.network.verbosity`

  *Type*: integer

  *Default*: 0

  与联网操作相关的组件的日志消息详细级别。查看[`NETWORK`](https://docs.mongodb.com/manual/reference/log-messages/#NETWORK)组件。

  详细程度的范围为`0`到`5`：

  * `0`是MongoDB的默认日志级别，包括 [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息。
  * `1`到`5`增加详细级别，以包括 [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels)消息。

- `systemLog.component.query.``verbosity`

  *Type*: integer*Default*: 0The log message verbosity level for components related to query operations. See [`QUERY`](https://docs.mongodb.com/master/reference/log-messages/#QUERY) components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.replication.``verbosity`

  *Type*: integer*Default*: 0The log message verbosity level for components related to replication. See [`REPL`](https://docs.mongodb.com/master/reference/log-messages/#REPL) components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.replication.election.``verbosity`

  *Type*: integer*Default*: 0*New in version 4.2.*The log message verbosity level for components related to election. See [`ELECTION`](https://docs.mongodb.com/master/reference/log-messages/#ELECTION) components.If [`systemLog.component.replication.election.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.election.verbosity) is unset, [`systemLog.component.replication.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.verbosity) level also applies to election components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.replication.heartbeats.``verbosity`

  *Type*: integer*Default*: 0*New in version 3.6.*The log message verbosity level for components related to heartbeats. See [`REPL_HB`](https://docs.mongodb.com/master/reference/log-messages/#REPL_HB) components.If [`systemLog.component.replication.heartbeats.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.heartbeats.verbosity) is unset, [`systemLog.component.replication.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.verbosity) level also applies to heartbeats components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.replication.initialSync.``verbosity`

  *Type*: integer*Default*: 0*New in version 4.2.*The log message verbosity level for components related to initialSync. See [`INITSYNC`](https://docs.mongodb.com/master/reference/log-messages/#INITSYNC) components.If [`systemLog.component.replication.initialSync.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.initialSync.verbosity) is unset, [`systemLog.component.replication.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.verbosity) level also applies to initialSync components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.replication.rollback.``verbosity`

  *Type*: integer*Default*: 0*New in version 3.6.*The log message verbosity level for components related to rollback. See [`ROLLBACK`](https://docs.mongodb.com/master/reference/log-messages/#ROLLBACK) components.If [`systemLog.component.replication.rollback.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.rollback.verbosity) is unset, [`systemLog.component.replication.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.replication.verbosity) level also applies to rollback components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.sharding.``verbosity`

  *Type*: integer*Default*: 0The log message verbosity level for components related to sharding. See [`SHARDING`](https://docs.mongodb.com/master/reference/log-messages/#SHARDING) components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.storage.``verbosity`

  *Type*: integer*Default*: 0The log message verbosity level for components related to storage. See [`STORAGE`](https://docs.mongodb.com/master/reference/log-messages/#STORAGE) components.If [`systemLog.component.storage.journal.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.journal.verbosity) is unset, [`systemLog.component.storage.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.verbosity) level also applies to journaling components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.storage.journal.``verbosity`

  *Type*: integer*Default*: 0The log message verbosity level for components related to journaling. See [`JOURNAL`](https://docs.mongodb.com/master/reference/log-messages/#JOURNAL) components.If [`systemLog.component.storage.journal.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.journal.verbosity) is unset, the journaling components have the same verbosity level as the parent storage components: i.e. either the [`systemLog.component.storage.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.verbosity) level if set or the default verbosity level.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.storage.recovery.``verbosity`

  *Type*: integer*Default*: 0*New in version 4.0.*The log message verbosity level for components related to recovery. See [`RECOVERY`](https://docs.mongodb.com/master/reference/log-messages/#RECOVERY) components.If [`systemLog.component.storage.recovery.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.recovery.verbosity) is unset, [`systemLog.component.storage.verbosity`](https://docs.mongodb.com/master/reference/configuration-options/#systemLog.component.storage.verbosity) level also applies to recovery components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.transaction.``verbosity`

  *Type*: integer*Default*: 0*New in version 4.0.2.*The log message verbosity level for components related to transaction. See [`TXN`](https://docs.mongodb.com/master/reference/log-messages/#TXN) components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

- `systemLog.component.write.``verbosity`

  *Type*: integer*Default*: 0The log message verbosity level for components related to write operations. See [`WRITE`](https://docs.mongodb.com/master/reference/log-messages/#WRITE) components.The verbosity level can range from `0` to `5`:`0` is the MongoDB’s default log verbosity level, to include [Informational](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.`1` to `5` increases the verbosity level to include [Debug](https://docs.mongodb.com/master/reference/log-messages/#log-severity-levels) messages.

### <span id="processmanagement-options">`processManagement` 选项</span>

copycopied

```
processManagement:
   fork: <boolean>
   pidFilePath: <string>
   timeZoneInfo: <string>
```

- `processManagement.``fork`

  *Type*: boolean*Default*: falseEnable a [daemon](https://docs.mongodb.com/master/reference/glossary/#term-daemon) mode that runs the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process in the background. By default [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) does not run as a daemon: typically you will run [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) as a daemon, either by using [`processManagement.fork`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.fork) or by using a controlling process that handles the daemonization process (e.g. as with `upstart` and `systemd`).The [`processManagement.fork`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.fork) option is not supported on Windows.The Linux package init scripts do not expect [`processManagement.fork`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.fork) to change from the defaults. If you use the Linux packages and change [`processManagement.fork`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.fork), you will have to use your own init scripts and disable the built-in scripts.

- `processManagement.``pidFilePath`

  *Type*: stringSpecifies a file location to store the process ID (PID) of the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process. The user running the `mongod` or `mongos` process must be able to write to this path. If the [`processManagement.pidFilePath`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.pidFilePath) option is not specified, the process does not create a PID file. This option is generally only useful in combination with the [`processManagement.fork`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.fork) setting.LINUXOn Linux, PID file management is generally the responsibility of your distro’s init system: usually a service file in the `/etc/init.d` directory, or a systemd unit file registered with `systemctl`. Only use the [`processManagement.pidFilePath`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.pidFilePath) option if you are not using one of these init systems. For more information, please see the respective [Installation Guide](https://docs.mongodb.com/master/installation/) for your operating system.MACOSOn macOS, PID file management is generally handled by `brew`. Only use the [`processManagement.pidFilePath`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.pidFilePath) option if you are not using `brew` on your macOS system. For more information, please see the respective [Installation Guide](https://docs.mongodb.com/master/installation/) for your operating system.

- `processManagement.``timeZoneInfo`

  *Type*: stringThe full path from which to load the time zone database. If this option is not provided, then MongoDB will use its built-in time zone database.The configuration file included with Linux and macOS packages sets the time zone database path to `/usr/share/zoneinfo` by default.The built-in time zone database is a copy of the [Olson/IANA time zone database](https://www.iana.org/time-zones). It is updated along with MongoDB releases, but the release cycle of the time zone database differs from the release cycle of MongoDB. A copy of the most recent release of the time zone database can be downloaded from https://downloads.mongodb.org/olson_tz_db/timezonedb-latest.zip.



### <span id="cloud-options">`cloud` 选项</span>

*New in version 4.0.*

copycopied

```
cloud:
   monitoring:
      free:
         state: <string>
         tags: <string>
```

- `cloud.monitoring.free.``state`

  *Type*: string*New in version 4.0:* Available for MongoDB Community Edition.Enables or disables [free MongoDB Cloud monitoring](https://docs.mongodb.com/master/administration/free-monitoring/). [`cloud.monitoring.free.state`](https://docs.mongodb.com/master/reference/configuration-options/#cloud.monitoring.free.state) accepts the following values:`runtime`Default. You can enable or disable free monitoring during runtime.To enable or disable free monitoring during runtime, see [`db.enableFreeMonitoring()`](https://docs.mongodb.com/master/reference/method/db.enableFreeMonitoring/#db.enableFreeMonitoring) and [`db.disableFreeMonitoring()`](https://docs.mongodb.com/master/reference/method/db.disableFreeMonitoring/#db.disableFreeMonitoring).To enable or disable free monitoring during runtime when running with access control, users must have required privileges. See [`db.enableFreeMonitoring()`](https://docs.mongodb.com/master/reference/method/db.enableFreeMonitoring/#db.enableFreeMonitoring) and [`db.disableFreeMonitoring()`](https://docs.mongodb.com/master/reference/method/db.disableFreeMonitoring/#db.disableFreeMonitoring) for details.`on`Enables free monitoring at startup; i.e. registers for free monitoring. When enabled at startup, you cannot disable free monitoring during runtime.`off`Disables free monitoring at startup, regardless of whether you have previously registered for free monitoring. When disabled at startup, you cannot enable free monitoring during runtime.Once enabled, the free monitoring state remains enabled until explicitly disabled. That is, you do not need to re-enable each time you start the server.For the corresponding command-line option, see [`--enableFreeMonitoring`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-enablefreemonitoring).

- `cloud.monitoring.free.``tags`

  *Type*: string*New in version 4.0:* Available for MongoDB Community Edition.Optional tag to describe environment context. The tag can be sent as part of the [free MongoDB Cloud monitoring](https://docs.mongodb.com/master/administration/free-monitoring/) registration at start up.For the corresponding command-line option, see [`--freeMonitoringTag`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-freemonitoringtag).

### <span id="net-options">`net` 选项</span>

*Changed in version 4.2:* MongoDB 4.2 deprecates `ssl` options in favor of `tls` options with identical functionality.

copycopied

```
net:
   port: <int>
   bindIp: <string>
   bindIpAll: <boolean>
   maxIncomingConnections: <int>
   wireObjectCheck: <boolean>
   ipv6: <boolean>
   unixDomainSocket:
      enabled: <boolean>
      pathPrefix: <string>
      filePermissions: <int>
   tls:
      certificateSelector: <string>
      clusterCertificateSelector: <string>
      mode: <string>
      certificateKeyFile: <string>
      certificateKeyFilePassword: <string>
      clusterFile: <string>
      clusterPassword: <string>
      CAFile: <string>
      clusterCAFile: <string>
      CRLFile: <string>
      allowConnectionsWithoutCertificates: <boolean>
      allowInvalidCertificates: <boolean>
      allowInvalidHostnames: <boolean>
      disabledProtocols: <string>
      FIPSMode: <boolean>
   compression:
      compressors: <string>
   serviceExecutor: <string>
```

- `net.``port`

  *Type*: integer*Default*:27017 for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) (if not a shard member or a config server member) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance27018 if [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) is a [`shard member`](https://docs.mongodb.com/master/reference/configuration-options/#sharding.clusterRole)27019 if [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) is a [`config server member`](https://docs.mongodb.com/master/reference/configuration-options/#sharding.clusterRole)The TCP port on which the MongoDB instance listens for client connections.

- `net.``bindIp`

  *Type*: string*Default*: localhostNOTEStarting in MongoDB 3.6, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) bind to localhost by default. See [Default Bind to Localhost](https://docs.mongodb.com/master/release-notes/3.6/#bind-to-localhost).The hostnames and/or IP addresses and/or full Unix domain socket paths on which [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) should listen for client connections. You may attach [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) to any interface. To bind to multiple addresses, enter a list of comma-separated values.EXAMPLE`localhost,/tmp/mongod.sock`You can specify both IPv4 and IPv6 addresses, or hostnames that resolve to an IPv4 or IPv6 address.EXAMPLE`localhost, 2001:0DB8:e132:ba26:0d5c:2774:e7f9:d513`NOTEIf specifying an IPv6 address *or* a hostname that resolves to an IPv6 address to [`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp), you must start [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) with [`net.ipv6 : true`](https://docs.mongodb.com/master/reference/configuration-options/#net.ipv6) to enable IPv6 support. Specifying an IPv6 address to [`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) does not enable IPv6 support.If specifying a [link-local IPv6 address](https://en.wikipedia.org/w/index.php?title=Link-local_address&oldid=880793020#IPv6) (`fe80::/10`), you must append the [zone index](https://en.wikipedia.org/w/index.php?title=IPv6_address&oldid=877601778#Scoped_literal_IPv6_addresses) to that address (i.e. `fe80::<address>%<adapter-name>`).EXAMPLE`localhost,fe80::a00:27ff:fee0:1fcf%enp0s3`TIPWhen possible, use a logical DNS hostname instead of an ip address, particularly when configuring replica set members or sharded cluster members. The use of logical DNS hostnames avoids configuration changes due to ip address changes.WARNINGBefore binding to a non-localhost (e.g. publicly accessible) IP address, ensure you have secured your cluster from unauthorized access. For a complete list of security recommendations, see [Security Checklist](https://docs.mongodb.com/master/administration/security-checklist/). At minimum, consider [enabling authentication](https://docs.mongodb.com/master/administration/security-checklist/#checklist-auth) and [hardening network infrastructure](https://docs.mongodb.com/master/core/security-hardening/).For more information about IP Binding, refer to the [IP Binding](https://docs.mongodb.com/master/core/security-mongodb-configuration/) documentation.To bind to all IPv4 addresses, enter `0.0.0.0`.To bind to all IPv4 and IPv6 addresses, enter `::,0.0.0.0` or starting in MongoDB 4.2, an asterisk `"*"` (enclose the asterisk in quotes to distinguish from [YAML alias nodes](https://yaml.org/spec/1.2/spec.html#alias/)). Alternatively, use the [`net.bindIpAll`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIpAll) setting.NOTE[`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) and [`net.bindIpAll`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIpAll) are mutually exclusive. That is, you can specify one or the other, but not both.The command-line option `--bind_ip` overrides the configuration file setting [`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp).

- `net.``bindIpAll`

  *Type*: boolean*Default*: false*New in version 3.6.*If true, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance binds to all IPv4 addresses (i.e. `0.0.0.0`). If [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) starts with [`net.ipv6 : true`](https://docs.mongodb.com/master/reference/configuration-options/#net.ipv6), [`net.bindIpAll`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIpAll) also binds to all IPv6 addresses (i.e. `::`).[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) only supports IPv6 if started with [`net.ipv6 : true`](https://docs.mongodb.com/master/reference/configuration-options/#net.ipv6). Specifying [`net.bindIpAll`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIpAll) alone does not enable IPv6 support.WARNINGBefore binding to a non-localhost (e.g. publicly accessible) IP address, ensure you have secured your cluster from unauthorized access. For a complete list of security recommendations, see [Security Checklist](https://docs.mongodb.com/master/administration/security-checklist/). At minimum, consider [enabling authentication](https://docs.mongodb.com/master/administration/security-checklist/#checklist-auth) and [hardening network infrastructure](https://docs.mongodb.com/master/core/security-hardening/).For more information about IP Binding, refer to the [IP Binding](https://docs.mongodb.com/master/core/security-mongodb-configuration/) documentation.Alternatively, set [`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) to `::,0.0.0.0` or, starting in MongoDB 4.2, to an asterisk `"*"` (enclose the asterisk in quotes to distinguish from [YAML alias nodes](https://yaml.org/spec/1.2/spec.html#alias/)) to bind to all IP addresses.NOTE[`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) and [`net.bindIpAll`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIpAll) are mutually exclusive. Specifying both options causes [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) to throw an error and terminate.

- `net.``maxIncomingConnections`

  *Type*: integer*Default*: 65536The maximum number of simultaneous connections that [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will accept. This setting has no effect if it is higher than your operating system’s configured maximum connection tracking threshold.Do not assign too low of a value to this option, or you will encounter errors during normal application operation.This is particularly useful for a [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) if you have a client that creates multiple connections and allows them to timeout rather than closing them.In this case, set [`maxIncomingConnections`](https://docs.mongodb.com/master/reference/configuration-options/#net.maxIncomingConnections) to a value slightly higher than the maximum number of connections that the client creates, or the maximum size of the connection pool.This setting prevents the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) from causing connection spikes on the individual [shards](https://docs.mongodb.com/master/reference/glossary/#term-shard). Spikes like these may disrupt the operation and memory allocation of the [sharded cluster](https://docs.mongodb.com/master/reference/glossary/#term-sharded-cluster).

- `net.``wireObjectCheck`

  *Type*: boolean*Default*: trueWhen `true`, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance validates all requests from clients upon receipt to prevent clients from inserting malformed or invalid BSON into a MongoDB database.For objects with a high degree of sub-document nesting, [`net.wireObjectCheck`](https://docs.mongodb.com/master/reference/configuration-options/#net.wireObjectCheck) can have a small impact on performance.

- `net.``ipv6`

  *Type*: boolean*Default*: falseSet [`net.ipv6`](https://docs.mongodb.com/master/reference/configuration-options/#net.ipv6) to `true` to enable IPv6 support. [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)/[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) disables IPv6 support by default.Setting [`net.ipv6`](https://docs.mongodb.com/master/reference/configuration-options/#net.ipv6) does *not* direct the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)/[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) to listen on any local IPv6 addresses or interfaces. To configure the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)/[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) to listen on an IPv6 interface, you must either:Configure [`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) with one or more IPv6 addresses or hostnames that resolve to IPv6 addresses, **or**Set [`net.bindIpAll`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIpAll) to `true`.

#### `net.unixDomainSocket` 选项

copycopied

```
net:
   unixDomainSocket:
      enabled: <boolean>
      pathPrefix: <string>
      filePermissions: <int>
```

- `net.unixDomainSocket.``enabled`

  *Type*: boolean*Default*: trueEnable or disable listening on the UNIX domain socket. [`net.unixDomainSocket.enabled`](https://docs.mongodb.com/master/reference/configuration-options/#net.unixDomainSocket.enabled) applies only to Unix-based systems.When [`net.unixDomainSocket.enabled`](https://docs.mongodb.com/master/reference/configuration-options/#net.unixDomainSocket.enabled) is `true`, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) listens on the UNIX socket.The [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process always listens on the UNIX socket unless one of the following is true:[`net.unixDomainSocket.enabled`](https://docs.mongodb.com/master/reference/configuration-options/#net.unixDomainSocket.enabled) is `false``--nounixsocket` is set. The command line option takes precedence over the configuration file setting.[`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) is not set[`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) does not specify `localhost` or its associated IP address[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) installed from official [.deb](https://docs.mongodb.com/master/tutorial/install-mongodb-on-debian/) and [.rpm](https://docs.mongodb.com/master/tutorial/install-mongodb-on-red-hat/) packages have the `bind_ip` configuration set to `127.0.0.1` by default.

- `net.unixDomainSocket.``pathPrefix`

  *Type*: string*Default*: /tmpThe path for the UNIX socket. [`net.unixDomainSocket.pathPrefix`](https://docs.mongodb.com/master/reference/configuration-options/#net.unixDomainSocket.pathPrefix) applies only to Unix-based systems.If this option has no value, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process creates a socket with `/tmp` as a prefix. MongoDB creates and listens on a UNIX socket unless one of the following is true:[`net.unixDomainSocket.enabled`](https://docs.mongodb.com/master/reference/configuration-options/#net.unixDomainSocket.enabled) is `false``--nounixsocket` is set[`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) is not set[`net.bindIp`](https://docs.mongodb.com/master/reference/configuration-options/#net.bindIp) does not specify `localhost` or its associated IP address

- `net.unixDomainSocket.``filePermissions`

  *Type*: int*Default*: `0700`Sets the permission for the UNIX domain socket file.[`net.unixDomainSocket.filePermissions`](https://docs.mongodb.com/master/reference/configuration-options/#net.unixDomainSocket.filePermissions) applies only to Unix-based systems.

#### `net.http` 选项

*Changed in version 3.6:* MongoDB 3.6 removes the deprecated `net.http` options. The options have been deprecated since version 3.2.



#### `net.tls` 选项

*New in version 4.2:* The `tls` options provide identical functionality as the previous `ssl` options.

copycopied

```
net:
   tls:
      mode: <string>
      certificateKeyFile: <string>
      certificateKeyFilePassword: <string>
      certificateSelector: <string>
      clusterCertificateSelector: <string>
      clusterFile: <string>
      clusterPassword: <string>
      CAFile: <string>
      clusterCAFile: <string>
      CRLFile: <string>
      allowConnectionsWithoutCertificates: <boolean>
      allowInvalidCertificates: <boolean>
      allowInvalidHostnames: <boolean>
      disabledProtocols: <string>
      FIPSMode: <boolean>
```

- `net.tls.``mode`

  *Type*: string*New in version 4.2.*Enables TLS used for all network connections. The argument to the [`net.tls.mode`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.mode) setting can be one of the following:ValueDescription`disabled`The server does not use TLS.`allowTLS`Connections between servers do not use TLS. For incoming connections, the server accepts both TLS and non-TLS.`preferTLS`Connections between servers use TLS. For incoming connections, the server accepts both TLS and non-TLS.`requireTLS`The server uses and accepts only TLS encrypted connections.If `--tlsCAFile` or `tls.CAFile` is not specified and you are not using x.509 authentication, the system-wide CA certificate store will be used when connecting to an TLS-enabled server.If using x.509 authentication, `--tlsCAFile` or `tls.CAFile` must be specified unless using [`--tlsCertificateSelector`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-tlscertificateselector).For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``certificateKeyFile`

  *Type*: string*New in version 4.2:* The `.pem` file that contains both the TLS certificate and key.Starting with MongoDB 4.0 on macOS or Windows, you can use the [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) setting to specify a certificate from the operating system’s secure certificate store instead of a PEM key file. [`certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile) and [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) are mutually exclusive. You can only specify one.On Linux/BSD, you must specify [`net.tls.certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile) when TLS is enabled.On Windows or macOS, you must specify either [`net.tls.certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile) or [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) when TLS is enabled.IMPORTANTFor Windows **only**, MongoDB 4.0 and later do not support encrypted PEM files. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails to start if it encounters an encrypted PEM file. To securely store and access a certificate for use with TLS on Windows, use [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector).For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``certificateKeyFilePassword`

  *Type*: string*New in version 4.2:* The password to de-crypt the certificate-key file (i.e. [`certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile)). Use the `net.tls.certificateKeyPassword` option only if the certificate-key file is encrypted. In all cases, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will redact the password from all logging and reporting output.Starting in MongoDB 4.0:On Linux/BSD, if the private key in the PEM file is encrypted and you do not specify the `net.tls.certificateKeyFukePassword` option, MongoDB will prompt for a passphrase. See [TLS/SSL Certificate Passphrase](https://docs.mongodb.com/master/tutorial/configure-ssl/#ssl-certificate-password).On macOS, if the private key in the PEM file is encrypted, you must explicitly specify the [`net.tls.certificateKeyFilePassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFilePassword) option. Alternatively, you can use a certificate from the secure system store (see [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector)) instead of a PEM key file or use an unencrypted PEM file.On Windows, MongoDB does not support encrypted certificates. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails if it encounters an encrypted PEM file. Use [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) instead.For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``certificateSelector`

  *Type*: string*New in version 4.2:* Available on Windows and macOS as an alternative to [`net.tls.certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile). In MongoDB 4.0, see [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector).Specifies a certificate property in order to select a matching certificate from the operating system’s certificate store to use for TLS/SSL.[`net.tls.certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile) and [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) options are mutually exclusive. You can only specify one.[`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) accepts an argument of the format `<property>=<value>` where the property can be one of the following:PropertyValue typeDescription`subject`ASCII stringSubject name or common name on certificate`thumbprint`hex stringA sequence of bytes, expressed as hexadecimal, used to identify a public key by its SHA-1 digest.The `thumbprint` is sometimes referred to as a `fingerprint`.When using the system SSL certificate store, OCSP (Online Certificate Status Protocol) is used to validate the revocation status of certificates.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) searches the operating system’s secure certificate store for the CA certificates required to validate the full certificate chain of the specified TLS certificate. Specifically, the secure certificate store must contain the root CA and any intermediate CA certificates required to build the full certificate chain to the TLS certificate. Do **not** use [`net.tls.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CAFile) or [`net.tls.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterFile) to specify the root and intermediate CA certificateFor example, if the TLS certificate was signed with a single root CA certificate, the secure certificate store must contain that root CA certificate. If the TLS certificate was signed with an intermediate CA certificate, the secure certificate store must contain the intermedia CA certificate *and* the root CA certificate.

- `net.tls.``clusterCertificateSelector`

  *Type*: string*New in version 4.2:* Available on Windows and macOS as an alternative to [`net.tls.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterFile).Specifies a certificate property to select a matching certificate from the operating system’s secure certificate store to use for [internal x.509 membership authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#internal-auth-x509).[`net.tls.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterFile) and [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector) options are mutually exclusive. You can only specify one.[`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector) accepts an argument of the format `<property>=<value>` where the property can be one of the following:PropertyValue typeDescription`subject`ASCII stringSubject name or common name on certificate`thumbprint`hex stringA sequence of bytes, expressed as hexadecimal, used to identify a public key by its SHA-1 digest.The `thumbprint` is sometimes referred to as a `fingerprint`.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) searches the operating system’s secure certificate store for the CA certificates required to validate the full certificate chain of the specified cluster certificate. Specifically, the secure certificate store must contain the root CA and any intermediate CA certificates required to build the full certificate chain to the cluster certificate. Do **not** use [`net.tls.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CAFile) or [`net.tls.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCAFile) to specify the root and intermediate CA certificate.For example, if the cluster certificate was signed with a single root CA certificate, the secure certificate store must contain that root CA certificate. If the cluster certificate was signed with an intermediate CA certificate, the secure certificate store must contain the intermediate CA certificate *and* the root CA certificate.*Changed in version 4.4:* [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) / [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) logs a warning on connection if the presented x.509 certificate expires within `30` days of the `mongod/mongos` host system time. See [x.509 Certificates Nearing Expiry Trigger Warnings](https://docs.mongodb.com/master/release-notes/4.4/#rel-notes-certificate-expiration-warning) for more information.

- `net.tls.``clusterFile`

  *Type*: string*New in version 4.2:* The `.pem` file that contains the x.509 certificate-key file for [membership authentication](https://docs.mongodb.com/master/tutorial/configure-x509-member-authentication/#x509-internal-authentication) for the cluster or replica set.Starting with MongoDB 4.0 on macOS or Windows, you can use the [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector) option to specify a certificate from the operating system’s secure certificate store instead of a PEM key file. [`net.tls.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterFile) and [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector) options are mutually exclusive. You can only specify one.If [`net.tls.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterFile) does not specify the `.pem` file for internal cluster authentication or the alternative [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector), the cluster uses the `.pem` file specified in the [`certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile) setting or the certificate returned by the [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector).If using x.509 authentication, `--tlsCAFile` or `tls.CAFile` must be specified unless using [`--tlsCertificateSelector`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-tlscertificateselector).*Changed in version 4.4:* [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) / [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) logs a warning on connection if the presented x.509 certificate expires within `30` days of the `mongod/mongos` host system time. See [x.509 Certificates Nearing Expiry Trigger Warnings](https://docs.mongodb.com/master/release-notes/4.4/#rel-notes-certificate-expiration-warning) for more information.For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .IMPORTANTFor Windows **only**, MongoDB 4.0 and later do not support encrypted PEM files. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails to start if it encounters an encrypted PEM file. To securely store and access a certificate for use with membership authentication on Windows, use [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector).

- `net.tls.``clusterPassword`

  *Type*: string*New in version 4.2:* The password to de-crypt the x.509 certificate-key file specified with `--sslClusterFile`. Use the [`net.tls.clusterPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterPassword) option only if the certificate-key file is encrypted. In all cases, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will redact the password from all logging and reporting output.Starting in MongoDB 4.0:On Linux/BSD, if the private key in the x.509 file is encrypted and you do not specify the [`net.tls.clusterPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterPassword) option, MongoDB will prompt for a passphrase. See [TLS/SSL Certificate Passphrase](https://docs.mongodb.com/master/tutorial/configure-ssl/#ssl-certificate-password).On macOS, if the private key in the x.509 file is encrypted, you must explicitly specify the [`net.tls.clusterPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterPassword) option. Alternatively, you can either use a certificate from the secure system store (see [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector)) instead of a cluster PEM file or use an unencrypted PEM file.On Windows, MongoDB does not support encrypted certificates. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails if it encounters an encrypted PEM file. Use [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector).For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``CAFile`

  *Type*: string*New in version 4.2:* The `.pem` file that contains the root certificate chain from the Certificate Authority. Specify the file name of the `.pem` file using relative or absolute paths.Windows/macOS OnlyIf using [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) and/or [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector), do **not** use [`net.tls.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CAFile) to specify the root and intermediate CA certificates. Store all CA certificates required to validate the full trust chain of the [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) and/or [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector) certificates in the secure certificate store.For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``clusterCAFile`

  *Type*: string*New in version 4.2:* The `.pem` file that contains the root certificate chain from the Certificate Authority used to validate the certificate presented by a client establishing a connection. Specify the file name of the `.pem` file using relative or absolute paths. [`net.tls.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCAFile) requires that [`net.tls.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CAFile) is set.If [`net.tls.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCAFile) does not specify the `.pem` file for validating the certificate from a client establishing a connection, the cluster uses the `.pem` file specified in the [`net.tls.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CAFile) option.[`net.tls.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCAFile) lets you use separate Certificate Authorities to verify the client to server and server to client portions of the TLS handshake.Starting in 4.0, on macOS or Windows, you can use a certificate from the operating system’s secure store instead of a PEM key file. See [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector). When using the secure store, you do not need to, but can, also specify the [`net.tls.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCAFile).Windows/macOS OnlyIf using [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) and/or [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector), do **not** use [`net.tls.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCAFile) to specify the root and intermediate CA certificates. Store all CA certificates required to validate the full trust chain of the [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) and/or [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector) certificates in the secure certificate store.For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``CRLFile`

  *Type*: string*New in version 4.2:* In MongoDB 4.0 and earlier, see [`net.ssl.CRLFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CRLFile).The `.pem` file that contains the Certificate Revocation List. Specify the file name of the `.pem` file using relative or absolute paths.NOTEStarting in MongoDB 4.0, you cannot specify [`net.tls.CRLFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CRLFile) on macOS. Instead, you can use the system SSL certificate store, which uses OCSP (Online Certificate Status Protocol) to validate the revocation status of certificates. See [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) in MongoDB 4.0 and [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) in MongoDB 4.2+ to use the system SSL certificate store.Starting in version 4.4, to check for certificate revocation, MongoDB [`enables`](https://docs.mongodb.com/master/reference/parameters/#param.ocspEnabled) the use of OCSP (Online Certificate Status Protocol) by default as an alternative to specifying a CRL file or using the system SSL certificate store.ource/reference/configuration-options.txt .. include:: /includes/extracts/tls-facts-see-more.rst

- `net.tls.``allowConnectionsWithoutCertificates`

  *Type*: boolean*New in version 4.2.*For clients that do not present certificates, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) bypasses TLS/SSL certificate validation when establishing the connection.For clients that present a certificate, however, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) performs certificate validation using the root certificate chain specified by [`CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CAFile) and reject clients with invalid certificates.Use the [`net.tls.allowConnectionsWithoutCertificates`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.allowConnectionsWithoutCertificates) option if you have a mixed deployment that includes clients that do not or cannot present certificates to the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``allowInvalidCertificates`

  *Type*: boolean*New in version 4.2.*Enable or disable the validation checks for TLS certificates on other servers in the cluster and allows the use of invalid certificates to connect.NOTEIf you specify `--tlsAllowInvalidCertificates` or `tls.allowInvalidCertificates: true` when using x.509 authentication, an invalid certificate is only sufficient to establish a TLS connection but is *insufficient* for authentication.When using the [`net.tls.allowInvalidCertificates`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.allowInvalidCertificates) setting, MongoDB logs a warning regarding the use of the invalid certificate.For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``allowInvalidHostnames`

  *Type*: boolean*Default*: falseWhen [`net.tls.allowInvalidHostnames`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.allowInvalidHostnames) is `true`, MongoDB disables the validation of the hostnames in TLS certificates, allowing [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) to connect to MongoDB instances if the hostname their certificates do not match the specified hostname.For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.tls.``disabledProtocols`

  *Type*: string*New in version 4.2.*Prevents a MongoDB server running with TLS from accepting incoming connections that use a specific protocol or protocols. To specify multiple protocols, use a comma separated list of protocols.[`net.tls.disabledProtocols`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.disabledProtocols) recognizes the following protocols: `TLS1_0`, `TLS1_1`, `TLS1_2`, and starting in version 4.0.4 (and 3.6.9), `TLS1_3`.On macOS, you cannot disable `TLS1_1` and leave both `TLS1_0` and `TLS1_2` enabled. You must disable at least one of the other two, for example, `TLS1_0,TLS1_1`.To list multiple protocols, specify as a comma separated list of protocols. For example `TLS1_0,TLS1_1`.Specifying an unrecognized protocol will prevent the server from starting.The specified disabled protocols overrides any default disabled protocols.Starting in version 4.0, MongoDB disables the use of TLS 1.0 if TLS 1.1+ is available on the system. To enable the disabled TLS 1.0, specify `none` to [`net.tls.disabledProtocols`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.disabledProtocols). See [Disable TLS 1.0](https://docs.mongodb.com/master/release-notes/4.0/#disable-tls).Members of replica sets and sharded clusters must speak at least one protocol in common.SEE ALSO[Disallow Protocols](https://docs.mongodb.com/master/tutorial/configure-ssl/#ssl-disallow-protocols)

- `net.tls.``FIPSMode`

  *Type*: boolean*New in version 4.2.*Enable or disable the use of the FIPS mode of the TLS library for the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod). Your system must have a FIPS compliant library to use the [`net.tls.FIPSMode`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.FIPSMode) option.NOTEFIPS-compatible TLS/SSL is available only in [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server). See [Configure MongoDB for FIPS](https://docs.mongodb.com/master/tutorial/configure-fips/) for more information.



#### `net.ssl` 选项

IMPORTANT

All SSL options are deprecated since 4.2. Use the [TLS counterparts](https://docs.mongodb.com/master/reference/configuration-options/#net-tls-conf-options) instead, as they have identical functionality to the SSL options. The SSL protocol is deprecated and MongoDB supports TLS 1.0 and later.

copycopied

```
net:
   ssl:                            # deprecated since 4.2
      sslOnNormalPorts: <boolean>  # deprecated since 2.6
      mode: <string>
      PEMKeyFile: <string>
      PEMKeyPassword: <string>
      certificateSelector: <string>
      clusterCertificateSelector: <string>
      clusterFile: <string>
      clusterPassword: <string>
      CAFile: <string>
      clusterCAFile: <string>
      CRLFile: <string>
      allowConnectionsWithoutCertificates: <boolean>
      allowInvalidCertificates: <boolean>
      allowInvalidHostnames: <boolean>
      disabledProtocols: <string>
      FIPSMode: <boolean>
```

- `net.ssl.``sslOnNormalPorts`

  *Type*: boolean*Deprecated since version 2.6:* Use [`net.tls.mode: requireTLS`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.mode) instead.Enable or disable TLS/SSL for [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).With [`net.ssl.sslOnNormalPorts`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.sslOnNormalPorts), a [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) requires TLS/SSL encryption for all connections on the default MongoDB port, or the port specified by [`net.port`](https://docs.mongodb.com/master/reference/configuration-options/#net.port). By default, [`net.ssl.sslOnNormalPorts`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.sslOnNormalPorts) is disabled.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``mode`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.mode`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.mode) instead.Enables TLS/SSL or mixed TLS/SSL used for all network connections. The argument to the [`net.ssl.mode`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.mode) setting can be one of the following:ValueDescription`disabled`The server does not use TLS/SSL.`allowSSL`Connections between servers do not use TLS/SSL. For incoming connections, the server accepts both TLS/SSL and non-TLS/non-SSL.`preferSSL`Connections between servers use TLS/SSL. For incoming connections, the server accepts both TLS/SSL and non-TLS/non-SSL.`requireSSL`The server uses and accepts only TLS/SSL encrypted connections.Starting in version 3.4, if `--tlsCAFile`/`net.tls.CAFile` (or their aliases `--sslCAFile`/`net.ssl.CAFile`) is not specified and you are not using x.509 authentication, the system-wide CA certificate store will be used when connecting to an TLS/SSL-enabled server.To use x.509 authentication, `--tlsCAFile` or `net.tls.CAFile` must be specified unless using `--tlsCertificateSelector` or `--net.tls.certificateSelector`. Or if using the `ssl` aliases, `--sslCAFile` or `net.ssl.CAFile` must be specified unless using `--sslCertificateSelector` or `net.ssl.certificateSelector`.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``PEMKeyFile`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.certificateKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFile) instead.The `.pem` file that contains both the TLS/SSL certificate and key.Starting with MongoDB 4.0 on macOS or Windows, you can use the [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) setting to specify a certificate from the operating system’s secure certificate store instead of a PEM key file. [`PEMKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyFile) and [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) are mutually exclusive. You can only specify one.On Linux/BSD, you must specify [`net.ssl.PEMKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyFile) when TLS/SSL is enabled.On Windows or macOS, you must specify either [`net.ssl.PEMKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyFile) or [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) when TLS/SSL is enabled.IMPORTANTFor Windows **only**, MongoDB 4.0 and later do not support encrypted PEM files. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails to start if it encounters an encrypted PEM file. To securely store and access a certificate for use with TLS/SSL on Windows, use [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector).For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``PEMKeyPassword`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.certificateKeyFilePassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFilePassword) instead.The password to de-crypt the certificate-key file (i.e. [`PEMKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyFile)). Use the [`net.ssl.PEMKeyPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyPassword) option only if the certificate-key file is encrypted. In all cases, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will redact the password from all logging and reporting output.Starting in MongoDB 4.0:On Linux/BSD, if the private key in the PEM file is encrypted and you do not specify the [`net.ssl.PEMKeyPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyPassword) option, MongoDB will prompt for a passphrase. See [TLS/SSL Certificate Passphrase](https://docs.mongodb.com/master/tutorial/configure-ssl/#ssl-certificate-password).On macOS, if the private key in the PEM file is encrypted, you must explicitly specify the [`net.ssl.PEMKeyPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyPassword) option. Alternatively, you can use a certificate from the secure system store (see [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector)) instead of a PEM key file or use an unencrypted PEM file.On Windows, MongoDB does not support encrypted certificates. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails if it encounters an encrypted PEM file. Use [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) instead.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``certificateSelector`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) instead.*New in version 4.0:* Available on Windows and macOS as an alternative to [`net.ssl.PEMKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyFile).Specifies a certificate property in order to select a matching certificate from the operating system’s certificate store to use for TLS/SSL.[`net.ssl.PEMKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyFile) and [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) options are mutually exclusive. You can only specify one.[`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) accepts an argument of the format `<property>=<value>` where the property can be one of the following:PropertyValue typeDescription`subject`ASCII stringSubject name or common name on certificate`thumbprint`hex stringA sequence of bytes, expressed as hexadecimal, used to identify a public key by its SHA-1 digest.The `thumbprint` is sometimes referred to as a `fingerprint`.When using the system SSL certificate store, OCSP (Online Certificate Status Protocol) is used to validate the revocation status of certificates.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) searches the operating system’s secure certificate store for the CA certificates required to validate the full certificate chain of the specified TLS/SSL certificate. Specifically, the secure certificate store must contain the root CA and any intermediate CA certificates required to build the full certificate chain to the TLS/SSL certificate. Do **not** use [`net.ssl.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CAFile) or [`net.ssl.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterFile) to specify the root and intermediate CA certificateFor example, if the TLS/SSL certificate was signed with a single root CA certificate, the secure certificate store must contain that root CA certificate. If the TLS/SSL certificate was signed with an intermediate CA certificate, the secure certificate store must contain the intermedia CA certificate *and* the root CA certificate.

- `net.ssl.``clusterCertificateSelector`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCertificateSelector) instead.*New in version 4.0:* Available on Windows and macOS as an alternative to [`net.ssl.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterFile).Specifies a certificate property to select a matching certificate from the operating system’s secure certificate store to use for [internal x.509 membership authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#internal-auth-x509).[`net.ssl.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterFile) and [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector) options are mutually exclusive. You can only specify one.[`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector) accepts an argument of the format `<property>=<value>` where the property can be one of the following:PropertyValue typeDescription`subject`ASCII stringSubject name or common name on certificate`thumbprint`hex stringA sequence of bytes, expressed as hexadecimal, used to identify a public key by its SHA-1 digest.The `thumbprint` is sometimes referred to as a `fingerprint`.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) searches the operating system’s secure certificate store for the CA certificates required to validate the full certificate chain of the specified cluster certificate. Specifically, the secure certificate store must contain the root CA and any intermediate CA certificates required to build the full certificate chain to the cluster certificate. Do **not** use [`net.ssl.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CAFile) or [`net.ssl.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterFile) to specify the root and intermediate CA certificate.For example, if the cluster certificate was signed with a single root CA certificate, the secure certificate store must contain that root CA certificate. If the cluster certificate was signed with an intermediate CA certificate, the secure certificate store must contain the intermedia CA certificate *and* the root CA certificate.

- `net.ssl.``clusterFile`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterFile) instead.The `.pem` file that contains the x.509 certificate-key file for [membership authentication](https://docs.mongodb.com/master/tutorial/configure-x509-member-authentication/#x509-internal-authentication) for the cluster or replica set.Starting with MongoDB 4.0 on macOS or Windows, you can use the [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector) option to specify a certificate from the operating system’s secure certificate store instead of a PEM key file. [`net.ssl.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterFile) and [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector) options are mutually exclusive. You can only specify one.If [`net.ssl.clusterFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterFile) does not specify the `.pem` file for internal cluster authentication or the alternative [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector), the cluster uses the `.pem` file specified in the [`PEMKeyFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.PEMKeyFile) setting or the certificate returned by the [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector).To use x.509 authentication, `--tlsCAFile` or `net.tls.CAFile` must be specified unless using `--tlsCertificateSelector` or `--net.tls.certificateSelector`. Or if using the `ssl` aliases, `--sslCAFile` or `net.ssl.CAFile` must be specified unless using `--sslCertificateSelector` or `net.ssl.certificateSelector`.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .IMPORTANTFor Windows **only**, MongoDB 4.0 and later do not support encrypted PEM files. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails to start if it encounters an encrypted PEM file. To securely store and access a certificate for use with membership authentication on Windows, use [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector).

- `net.ssl.``clusterPassword`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.clusterPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterPassword) instead.The password to de-crypt the x.509 certificate-key file specified with `--sslClusterFile`. Use the [`net.ssl.clusterPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterPassword) option only if the certificate-key file is encrypted. In all cases, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will redact the password from all logging and reporting output.Starting in MongoDB 4.0:On Linux/BSD, if the private key in the x.509 file is encrypted and you do not specify the [`net.ssl.clusterPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterPassword) option, MongoDB will prompt for a passphrase. See [TLS/SSL Certificate Passphrase](https://docs.mongodb.com/master/tutorial/configure-ssl/#ssl-certificate-password).On macOS, if the private key in the x.509 file is encrypted, you must explicitly specify the [`net.ssl.clusterPassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterPassword) option. Alternatively, you can either use a certificate from the secure system store (see [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector)) instead of a cluster PEM file or use an unencrypted PEM file.On Windows, MongoDB does not support encrypted certificates. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) fails if it encounters an encrypted PEM file. Use [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector).For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``CAFile`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CAFile) instead.The `.pem` file that contains the root certificate chain from the Certificate Authority. Specify the file name of the `.pem` file using relative or absolute paths.Windows/macOS OnlyIf using [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) and/or [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector), do **not** use [`net.ssl.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CAFile) to specify the root and intermediate CA certificates. Store all CA certificates required to validate the full trust chain of the [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) and/or [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector) certificates in the secure certificate store.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``clusterCAFile`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.clusterCAFile) instead.The `.pem` file that contains the root certificate chain from the Certificate Authority used to validate the certificate presented by a client establishing a connection. Specify the file name of the `.pem` file using relative or absolute paths. [`net.ssl.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCAFile) requires that [`net.ssl.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CAFile) is set.If [`net.ssl.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCAFile) does not specify the `.pem` file for validating the certificate from a client establishing a connection, the cluster uses the `.pem` file specified in the [`net.ssl.CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CAFile) option.[`net.ssl.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCAFile) lets you use separate Certificate Authorities to verify the client to server and server to client portions of the TLS handshake.Starting in 4.0, on macOS or Windows, you can use a certificate from the operating system’s secure store instead of a PEM key file. See [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector). When using the secure store, you do not need to, but can, also specify the [`net.ssl.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCAFile).Windows/macOS OnlyIf using [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) and/or [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector), do **not** use [`net.ssl.clusterCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCAFile) to specify the root and intermediate CA certificates. Store all CA certificates required to validate the full trust chain of the [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) and/or [`net.ssl.clusterCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.clusterCertificateSelector) certificates in the secure certificate store.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``CRLFile`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.CRLFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.CRLFile) instead.The `.pem` file that contains the Certificate Revocation List. Specify the file name of the `.pem` file using relative or absolute paths.NOTEStarting in MongoDB 4.0, you cannot specify [`net.ssl.CRLFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CRLFile) on macOS. Instead, you can use the system SSL certificate store, which uses OCSP (Online Certificate Status Protocol) to validate the revocation status of certificates. See [`net.ssl.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.certificateSelector) in MongoDB 4.0 and [`net.tls.certificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateSelector) in MongoDB 4.2 to use the system SSL certificate store.Starting in version 4.4, MongoDB [enables](https://docs.mongodb.com/master/core/security-transport-encryption/#ocsp-support), by default, the use of OCSP (Online Certificate Status Protocol) to check for certificate revocation as an alternative to specifying a CRL file or using the system SSL certificate store.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``allowConnectionsWithoutCertificates`

  *Type*: boolean*Deprecated since version 4.2:* Use [`net.tls.allowConnectionsWithoutCertificates`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.allowConnectionsWithoutCertificates) instead.For clients that do not present certificates, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) bypasses TLS/SSL certificate validation when establishing the connection.For clients that present a certificate, however, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) performs certificate validation using the root certificate chain specified by [`CAFile`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.CAFile) and reject clients with invalid certificates.Use the [`net.ssl.allowConnectionsWithoutCertificates`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.allowConnectionsWithoutCertificates) option if you have a mixed deployment that includes clients that do not or cannot present certificates to the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``allowInvalidCertificates`

  *Type*: boolean*Deprecated since version 4.2:* Use [`net.tls.allowInvalidCertificates`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.allowInvalidCertificates) instead.Enable or disable the validation checks for TLS/SSL certificates on other servers in the cluster and allows the use of invalid certificates to connect.NOTEStarting in MongoDB 4.0, if you specify `--sslAllowInvalidCertificates` or `net.ssl.allowInvalidCertificates: true` (or in MongoDB 4.2, the alias `--tlsAllowInvalidateCertificates` or `net.tls.allowInvalidCertificates: true`) when using x.509 authentication, an invalid certificate is only sufficient to establish a TLS/SSL connection but is *insufficient* for authentication.When using the [`net.ssl.allowInvalidCertificates`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.allowInvalidCertificates) setting, MongoDB logs a warning regarding the use of the invalid certificate.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``allowInvalidHostnames`

  *Type*: boolean*Default*: false*Deprecated since version 4.2.*Use [`net.tls.allowInvalidHostnames`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.allowInvalidHostnames) instead.When [`net.ssl.allowInvalidHostnames`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.allowInvalidHostnames) is `true`, MongoDB disables the validation of the hostnames in TLS/SSL certificates, allowing [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) to connect to MongoDB instances if the hostname their certificates do not match the specified hostname.For more information about TLS/SSL and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `net.ssl.``disabledProtocols`

  *Type*: string*Deprecated since version 4.2:* Use [`net.tls.disabledProtocols`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.disabledProtocols) instead.Prevents a MongoDB server running with TLS/SSL from accepting incoming connections that use a specific protocol or protocols. To specify multiple protocols, use a comma separated list of protocols.[`net.ssl.disabledProtocols`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.disabledProtocols) recognizes the following protocols: `TLS1_0`, `TLS1_1`, `TLS1_2`, and starting in version 4.0.4 (and 3.6.9), `TLS1_3`.On macOS, you cannot disable `TLS1_1` and leave both `TLS1_0` and `TLS1_2` enabled. You must disable at least one of the other two, for example, `TLS1_0,TLS1_1`.To list multiple protocols, specify as a comma separated list of protocols. For example `TLS1_0,TLS1_1`.Specifying an unrecognized protocol will prevent the server from starting.The specified disabled protocols overrides any default disabled protocols.Starting in version 4.0, MongoDB disables the use of TLS 1.0 if TLS 1.1+ is available on the system. To enable the disabled TLS 1.0, specify `none` to [`net.ssl.disabledProtocols`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.disabledProtocols). See [Disable TLS 1.0](https://docs.mongodb.com/master/release-notes/4.0/#disable-tls).Members of replica sets and sharded clusters must speak at least one protocol in common.SEE ALSO[Disallow Protocols](https://docs.mongodb.com/master/tutorial/configure-ssl/#ssl-disallow-protocols)

- `net.ssl.``FIPSMode`

  *Type*: boolean*Deprecated since version 4.2:* Use [`net.tls.FIPSMode`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.FIPSMode) instead.Enable or disable the use of the FIPS mode of the TLS/SSL library for the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod). Your system must have a FIPS compliant library to use the [`net.ssl.FIPSMode`](https://docs.mongodb.com/master/reference/configuration-options/#net.ssl.FIPSMode) option.NOTEFIPS-compatible TLS/SSL is available only in [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server). See [Configure MongoDB for FIPS](https://docs.mongodb.com/master/tutorial/configure-fips/) for more information.

#### `net.compression` 选项

copycopied

```
net:
   compression:
      compressors: <string>
```

- `net.compression.``compressors`

  *Default*: snappy,zstd,zlib*New in version 3.4.*Specifies the default compressor(s) to use for communication between this [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance and:other members of the deployment if the instance is part of a replica set or a sharded clustera [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shelldrivers that support the `OP_COMPRESSED` message format.MongoDB supports the following compressors:[snappy](https://docs.mongodb.com/master/reference/glossary/#term-snappy)[zlib](https://docs.mongodb.com/master/reference/glossary/#term-zlib) (Available starting in MongoDB 3.6)[zstd](https://docs.mongodb.com/master/reference/glossary/#term-zstd) (Available starting in MongoDB 4.2)**In versions 3.6 and 4.0**, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) enable network compression by default with `snappy` as the compressor.**Starting in version 4.2**, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instances default to both `snappy,zstd,zlib` compressors, in that order.To disable network compression, set the value to `disabled`.IMPORTANTMessages are compressed when both parties enable network compression. Otherwise, messages between the parties are uncompressed.If you specify multiple compressors, then the order in which you list the compressors matter as well as the communication initiator. For example, if a [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell specifies the following network compressors `zlib,snappy` and the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) specifies `snappy,zlib`, messages between [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell and [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) uses `zlib`.If the parties do not share at least one common compressor, messages between the parties are uncompressed. For example, if a [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell specifies the network compressor `zlib` and [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) specifies `snappy`, messages between [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell and [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) are not compressed.

- `net.``serviceExecutor`

  *Type*: string*Default*: synchronous*New in version 3.6.*Determines the threading and execution model [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) uses to execute client requests. The `--serviceExecutor` option accepts one of the following values:ValueDescription`synchronous`The [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) uses synchronous networking and manages its networking thread pool on a per connection basis. Previous versions of MongoDB managed threads in this way.`adaptive`The [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) uses the new experimental asynchronous networking mode with an adaptive thread pool which manages threads on a per request basis. This mode should have more consistent performance and use less resources when there are more inactive connections than database requests.

### <span id="security-options">`security` 选项</span>

copycopied

```
security:
   keyFile: <string>
   clusterAuthMode: <string>
   authorization: <string>
   transitionToAuth: <boolean>
   javascriptEnabled:  <boolean>
   redactClientLogData: <boolean>
   clusterIpSourceWhitelist:
     - <string>
   sasl:
      hostName: <string>
      serviceName: <string>
      saslauthdSocketPath: <string>
   enableEncryption: <boolean>
   encryptionCipherMode: <string>
   encryptionKeyFile: <string>
   kmip:
      keyIdentifier: <string>
      rotateMasterKey: <boolean>
      serverName: <string>
      port: <string>
      clientCertificateFile: <string>
      clientCertificatePassword: <string>
      clientCertificateSelector: <string>
      serverCAFile: <string>
      connectRetries: <int>
      connectTimeoutMS: <int>
   ldap:
      servers: <string>
      bind:
         method: <string>
         saslMechanisms: <string>
         queryUser: <string>
         queryPassword: <string>
         useOSDefaults: <boolean>
      transportSecurity: <string>
      timeoutMS: <int>
      userToDNMapping: <string>
      authz:
         queryTemplate: <string>
      validateLDAPServerConfig: <boolean>
```

- `security.``keyFile`

  *Type*: stringThe path to a key file that stores the shared secret that MongoDB instances use to authenticate to each other in a [sharded cluster](https://docs.mongodb.com/master/reference/glossary/#term-sharded-cluster) or [replica set](https://docs.mongodb.com/master/reference/glossary/#term-replica-set). [`keyFile`](https://docs.mongodb.com/master/reference/configuration-options/#security.keyFile) implies [`security.authorization`](https://docs.mongodb.com/master/reference/configuration-options/#security.authorization). See [Internal/Membership Authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#inter-process-auth) for more information.Starting in MongoDB 4.2, [keyfiles for internal membership authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#internal-auth-keyfile) use YAML format to allow for multiple keys in a keyfile. The YAML format accepts content of:a single key string (same as in earlier versions),multiple key strings (each string must be enclosed in quotes), orsequence of key strings.The YAML format is compatible with the existing single-key keyfiles that use the text file format.

- `security.``clusterAuthMode`

  *Type*: string*Default*: keyFileThe authentication mode used for cluster authentication. If you use [internal x.509 authentication](https://docs.mongodb.com/master/tutorial/configure-x509-member-authentication/#x509-internal-authentication), specify so here. This option can have one of the following values:ValueDescription`keyFile`Use a keyfile for authentication. Accept only keyfiles.`sendKeyFile`For rolling upgrade purposes. Send a keyfile for authentication but can accept both keyfiles and x.509 certificates.`sendX509`For rolling upgrade purposes. Send the x.509 certificate for authentication but can accept both keyfiles and x.509 certificates.`x509`Recommended. Send the x.509 certificate for authentication and accept only x.509 certificates.If `--tlsCAFile` or `tls.CAFile` is not specified and you are not using x.509 authentication, the system-wide CA certificate store will be used when connecting to an TLS-enabled server.If using x.509 authentication, `--tlsCAFile` or `tls.CAFile` must be specified unless using [`--tlsCertificateSelector`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-tlscertificateselector).For more information about TLS and MongoDB, see [Configure mongod and mongos for TLS/SSL](https://docs.mongodb.com/master/tutorial/configure-ssl/) and [TLS/SSL Configuration for Clients](https://docs.mongodb.com/master/tutorial/configure-ssl-clients/) .

- `security.``authorization`

  *Type*: string*Default*: disabledEnable or disable Role-Based Access Control (RBAC) to govern each user’s access to database resources and operations.Set this option to one of the following:ValueDescription`enabled`A user can access only the database resources and actions for which they have been granted privileges.`disabled`A user can access any database and perform any action.See [Role-Based Access Control](https://docs.mongodb.com/master/core/authorization/) for more information.The [`security.authorization`](https://docs.mongodb.com/master/reference/configuration-options/#security.authorization) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).

- `security.``transitionToAuth`

  *Type*: boolean*Default*: false*New in version 3.4:* Allows the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) to accept and create authenticated and non-authenticated connections to and from other [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instances in the deployment. Used for performing rolling transition of replica sets or sharded clusters from a no-auth configuration to [internal authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#inter-process-auth). Requires specifying a [internal authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#inter-process-auth) mechanism such as [`security.keyFile`](https://docs.mongodb.com/master/reference/configuration-options/#security.keyFile).For example, if using [keyfiles](https://docs.mongodb.com/master/core/security-internal-authentication/#internal-auth-keyfile) for [internal authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#inter-process-auth), the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) creates an authenticated connection with any [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) in the deployment using a matching keyfile. If the security mechanisms do not match, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) utilizes a non-authenticated connection instead.A [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) running with [`security.transitionToAuth`](https://docs.mongodb.com/master/reference/configuration-options/#security.transitionToAuth) does not enforce [user access controls](https://docs.mongodb.com/master/core/authorization/#authorization). Users may connect to your deployment without any access control checks and perform read, write, and administrative operations.NOTEA [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) running with [internal authentication](https://docs.mongodb.com/master/core/security-internal-authentication/#inter-process-auth) and *without* [`security.transitionToAuth`](https://docs.mongodb.com/master/reference/configuration-options/#security.transitionToAuth) requires clients to connect using [user access controls](https://docs.mongodb.com/master/core/authorization/#authorization). Update clients to connect to the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) using the appropriate [user](https://docs.mongodb.com/master/core/security-users/#users) prior to restarting [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) without [`security.transitionToAuth`](https://docs.mongodb.com/master/reference/configuration-options/#security.transitionToAuth).

- `security.``javascriptEnabled`

  *Type*: boolean*Default*: trueEnables or disables [server-side JavaScript execution](https://docs.mongodb.com/master/core/server-side-javascript/). When disabled, you cannot use operations that perform server-side execution of JavaScript code, such as the [`$where`](https://docs.mongodb.com/master/reference/operator/query/where/#op._S_where) query operator, [`mapReduce`](https://docs.mongodb.com/master/reference/command/mapReduce/#dbcmd.mapReduce) command, [`$accumulator`](https://docs.mongodb.com/master/reference/operator/aggregation/accumulator/#grp._S_accumulator), and [`$function`](https://docs.mongodb.com/master/reference/operator/aggregation/function/#exp._S_function).If you do not use these operations, disable server-side scripting.Starting in version 4.4, the [`security.javascriptEnabled`](https://docs.mongodb.com/master/reference/configuration-options/#security.javascriptEnabled) is available for both [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos). In earlier versions, the setting is only available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).

- `security.``redactClientLogData`

  *Type*: boolean*New in version 3.4:* Available in MongoDB Enterprise only.A [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) running with [`security.redactClientLogData`](https://docs.mongodb.com/master/reference/configuration-options/#security.redactClientLogData) redacts any message accompanying a given log event before logging. This prevents the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) from writing potentially sensitive data stored on the database to the diagnostic log. Metadata such as error or operation codes, line numbers, and source file names are still visible in the logs.Use [`security.redactClientLogData`](https://docs.mongodb.com/master/reference/configuration-options/#security.redactClientLogData) in conjunction with [Encryption at Rest](https://docs.mongodb.com/master/core/security-encryption-at-rest/) and [TLS/SSL (Transport Encryption)](https://docs.mongodb.com/master/core/security-transport-encryption/) to assist compliance with regulatory requirements.For example, a MongoDB deployment might store Personally Identifiable Information (PII) in one or more collections. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) logs events such as those related to CRUD operations, sharding metadata, etc. It is possible that the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) may expose PII as a part of these logging operations. A [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) running with [`security.redactClientLogData`](https://docs.mongodb.com/master/reference/configuration-options/#security.redactClientLogData) removes any message accompanying these events before being output to the log, effectively removing the PII.Diagnostics on a [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) running with [`security.redactClientLogData`](https://docs.mongodb.com/master/reference/configuration-options/#security.redactClientLogData) may be more difficult due to the lack of data related to a log event. See the [process logging](https://docs.mongodb.com/master/administration/monitoring/#monitoring-log-redaction) manual page for an example of the effect of [`security.redactClientLogData`](https://docs.mongodb.com/master/reference/configuration-options/#security.redactClientLogData) on log output.On a running [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos), use [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter) with the [`redactClientLogData`](https://docs.mongodb.com/master/reference/parameters/#param.redactClientLogData) parameter to configure this setting.

- `security.``clusterIpSourceWhitelist`

  *Type*: list*New in version 3.6.*A list of IP addresses/CIDR ([Classless Inter-Domain Routing](https://tools.ietf.org/html/rfc4632)) ranges against which the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) validates authentication requests from other members of the replica set and, if part of a sharded cluster, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instances. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) verifies that the originating IP is either explicitly in the list or belongs to a CIDR range in the list. If the IP address is not present, the server does not authenticate the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos).[`security.clusterIpSourceWhitelist`](https://docs.mongodb.com/master/reference/configuration-options/#security.clusterIpSourceWhitelist) has no effect on a [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) started without [authentication](https://docs.mongodb.com/master/core/authentication/#authentication).[`security.clusterIpSourceWhitelist`](https://docs.mongodb.com/master/reference/configuration-options/#security.clusterIpSourceWhitelist) requires specifying each IPv4/6 address or Classless Inter-Domain Routing ([CIDR](https://tools.ietf.org/html/rfc4632)) range as a YAML list:copycopied`security:  clusterIpSourceWhitelist:    - 192.0.2.0/24    - 127.0.0.1    - ::1 `IMPORTANTEnsure [`security.clusterIpSourceWhitelist`](https://docs.mongodb.com/master/reference/configuration-options/#security.clusterIpSourceWhitelist) includes the IP address *or* CIDR ranges that include the IP address of each replica set member or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) in the deployment to ensure healthy communication between cluster components.



#### 密钥管理配置选项

copycopied

```
security:
   enableEncryption: <boolean>
   encryptionCipherMode: <string>
   encryptionKeyFile: <string>
   kmip:
      keyIdentifier: <string>
      rotateMasterKey: <boolean>
      serverName: <string>
      port: <string>
      clientCertificateFile: <string>
      clientCertificatePassword: <string>
      clientCertificateSelector: <string>
      serverCAFile: <string>
      connectRetries: <int>
      connectTimeoutMS: <int>
```

- `security.``enableEncryption`

  *Type*: boolean*Default*: false*New in version 3.2:* Enables encryption for the WiredTiger storage engine. You must set to `true` to pass in encryption keys and configurations.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.``encryptionCipherMode`

  *Type*: string*Default*: `AES256-CBC`*New in version 3.2.*The cipher mode to use for encryption at rest:ModeDescription`AES256-CBC`256-bit Advanced Encryption Standard in Cipher Block Chaining Mode`AES256-GCM`256-bit Advanced Encryption Standard in Galois/Counter ModeAvailable only on Linux.*Changed in version 4.0:* MongoDB Enterprise on Windows no longer supports `AES256-GCM`. This cipher is now available only on Linux.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.``encryptionKeyFile`

  *Type*: string*New in version 3.2.*The path to the local keyfile when managing keys via process *other than* KMIP. Only set when managing keys via process other than KMIP. If data is already encrypted using KMIP, MongoDB will throw an error.Requires [`security.enableEncryption`](https://docs.mongodb.com/master/reference/configuration-options/#security.enableEncryption) to be `true`.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``keyIdentifier`

  *Type*: string*New in version 3.2.*Unique KMIP identifier for an existing key within the KMIP server. Include to use the key associated with the identifier as the system key. You can only use the setting the first time you enable encryption for the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance. Requires [`security.enableEncryption`](https://docs.mongodb.com/master/reference/configuration-options/#security.enableEncryption) to be true.If unspecified, MongoDB will request that the KMIP server create a new key to utilize as the system key.If the KMIP server cannot locate a key with the specified identifier or the data is already encrypted with a key, MongoDB will throw an error.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``rotateMasterKey`

  *Type*: boolean*Default*: false*New in version 3.2.*If true, rotate the master key and re-encrypt the internal keystore.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.SEE ALSO[KMIP Master Key Rotation](https://docs.mongodb.com/master/tutorial/rotate-encryption-key/#kmip-master-key-rotation)

- `security.kmip.``serverName`

  *Type*: string*New in version 3.2.*Hostname or IP address of the KMIP server to connect to. Requires [`security.enableEncryption`](https://docs.mongodb.com/master/reference/configuration-options/#security.enableEncryption) to be true.Starting in MongoDB 4.2.1 (and 4.0.14), you can specify multiple KMIP servers as a comma-separated list, e.g. `server1.example.com,server2.example.com`. On startup, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will attempt to establish a connection to each server in the order listed, and will select the first server to which it can successfully establish a connection. KMIP server selection occurs only at startup.When connecting to a KMIP server, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) verifies that the specified [`security.kmip.serverName`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.serverName) matches the Subject Alternative Name `SAN` (or, if `SAN` is not present, the Common Name `CN`) in the certificate presented by the KMIP server. If `SAN` is present, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) does not match against the `CN`. If the hostname does not match the `SAN` (or `CN`), the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will fail to connect.Starting in MongoDB 4.2, when performing comparison of SAN, MongoDB supports comparison of DNS names or IP addresses. In previous versions, MongoDB only supports comparisons of DNS names.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``port`

  *Type*: string*Default*: 5696*New in version 3.2.*Port number to use to communicate with the KMIP server. Requires [`security.kmip.serverName`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.serverName). Requires [`security.enableEncryption`](https://docs.mongodb.com/master/reference/configuration-options/#security.enableEncryption) to be true.If specifying multiple KMIP servers with [`security.kmip.serverName`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.serverName), the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will use the port specified with [`security.kmip.port`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.port) for all provided KMIP servers.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``clientCertificateFile`

  *Type*: string*New in version 3.2.*String containing the path to the client certificate used for authenticating MongoDB to the KMIP server. Requires that a [`security.kmip.serverName`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.serverName) be provided.NOTEStarting in 4.0, on macOS or Windows, you can use a certificate from the operating system’s secure store instead of a PEM key file. See [`security.kmip.clientCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.clientCertificateSelector).ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``clientCertificatePassword`

  *Type*: string*New in version 3.2.*The password to decrypt the client certificate (i.e. [`security.kmip.clientCertificateFile`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.clientCertificateFile)), used to authenticate MongoDB to the KMIP server. Use the option only if the certificate is encrypted.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``clientCertificateSelector`

  *Type*: string*New in version 4.0:* Available on Windows and macOS as an alternative to [`security.kmip.clientCertificateFile`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.clientCertificateFile).[`security.kmip.clientCertificateFile`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.clientCertificateFile) and [`security.kmip.clientCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.clientCertificateSelector) options are mutually exclusive. You can only specify one.Specifies a certificate property in order to select a matching certificate from the operating system’s certificate store to authenticate MongoDB to the KMIP server.[`security.kmip.clientCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.clientCertificateSelector) accepts an argument of the format `<property>=<value>` where the property can be one of the following:PropertyValue typeDescription`subject`ASCII stringSubject name or common name on certificate`thumbprint`hex stringA sequence of bytes, expressed as hexadecimal, used to identify a public key by its SHA-1 digest.The `thumbprint` is sometimes referred to as a `fingerprint`.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``serverCAFile`

  *Type*: string*New in version 3.2.*Path to CA File. Used for validating secure client connection to KMIP server.NOTEStarting in 4.0, on macOS or Windows, you can use a certificate from the operating system’s secure store instead of a PEM key file. See [`security.kmip.clientCertificateSelector`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.clientCertificateSelector). When using the secure store, you do not need to, but can, also specify the [`security.kmip.serverCAFile`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.serverCAFile).ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``connectRetries`

  *Type*: int*Default*: 0*New in version 4.4.*How many times to retry the initial connection to the KMIP server. Use together with [`connectTimeoutMS`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.connectTimeoutMS) to control how long the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) waits for a response between each retry.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.

- `security.kmip.``connectTimeoutMS`

  *Type*: int*Default*: 5000*New in version 4.4.*Timeout in milliseconds to wait for a response from the KMIP server. If the [`connectRetries`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.connectRetries) setting is specified, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will wait up to the value specified with [`connectTimeoutMS`](https://docs.mongodb.com/master/reference/configuration-options/#security.kmip.connectTimeoutMS) for each retry.Value must be `1000` or greater.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.



#### `security.sasl` 选项

copycopied

```
security:
   sasl:
      hostName: <string>
      serviceName: <string>
      saslauthdSocketPath: <string>
```

- `security.sasl.``hostName`

  *Type*: stringA fully qualified server domain name for the purpose of configuring SASL and Kerberos authentication. The SASL hostname overrides the hostname only for the configuration of SASL and Kerberos.For [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell and other MongoDB tools to connect to the new [`hostName`](https://docs.mongodb.com/master/reference/configuration-options/#security.sasl.hostName), see the `gssapiHostName` option in the [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell and other tools.

- `security.sasl.``serviceName`

  *Type*: stringRegistered name of the service using SASL. This option allows you to override the default [Kerberos](https://docs.mongodb.com/master/tutorial/control-access-to-mongodb-with-kerberos-authentication/) service name component of the [Kerberos](https://docs.mongodb.com/master/tutorial/control-access-to-mongodb-with-kerberos-authentication/) principal name, on a per-instance basis. If unspecified, the default value is `mongodb`.MongoDB permits setting this option only at startup. The [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter) can not change this setting.This option is available only in MongoDB Enterprise.IMPORTANTEnsure that your driver supports alternate service names. For [`mongo`](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell and other MongoDB tools to connect to the new [`serviceName`](https://docs.mongodb.com/master/reference/configuration-options/#security.sasl.serviceName), see the `gssapiServiceName` option.

- `security.sasl.``saslauthdSocketPath`

  *Type*: stringThe path to the UNIX domain socket file for `saslauthd`.



#### `security.ldap` 选项

copycopied

```
security:
   ldap:
      servers: <string>
      bind:
         method: <string>
         saslMechanisms: <string>
         queryUser: <string>
         queryPassword: <string>
         useOSDefaults: <boolean>
      transportSecurity: <string>
      timeoutMS: <int>
      userToDNMapping: <string>
      authz:
         queryTemplate: <string>
      validateLDAPServerConfig: <boolean>
```

- `security.ldap.``servers`

  *Type*: string*New in version 3.4:* Available in MongoDB Enterprise only.The LDAP server against which the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) authenticates users or determines what actions a user is authorized to perform on a given database. If the LDAP server specified has any replicated instances, you may specify the host and port of each replicated server in a comma-delimited list.If your LDAP infrastructure partitions the LDAP directory over multiple LDAP servers, specify *one* LDAP server or any of its replicated instances to [`security.ldap.servers`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.servers). MongoDB supports following LDAP referrals as defined in [RFC 4511 4.1.10](https://www.rfc-editor.org/rfc/rfc4511.txt). Do not use [`security.ldap.servers`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.servers) for listing every LDAP server in your infrastructure.This setting can be configured on a running [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) using [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter).If unset, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) cannot use [LDAP authentication or authorization](https://docs.mongodb.com/master/core/security-ldap/).

- `security.ldap.bind.``queryUser`

  *Type*: string*New in version 3.4:* Available in MongoDB Enterprise only.The identity with which [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) binds as, when connecting to or performing queries on an LDAP server.Only required if any of the following are true:Using [LDAP authorization](https://docs.mongodb.com/master/core/security-ldap-external/#security-ldap-external).Using an LDAP query for [`security.ldap.userToDNMapping`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping).The LDAP server disallows anonymous bindsYou must use [`queryUser`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryUser) with [`queryPassword`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryPassword).If unset, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) will not attempt to bind to the LDAP server.This setting can be configured on a running [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) using [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter).NOTEWindows MongoDB deployments can use `bindWithOSDefaults` instead of [`queryUser`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryUser) and [`queryPassword`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryPassword). You cannot specify both [`queryUser`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryUser) and `bindWithOSDefaults` at the same time.

- `security.ldap.bind.``queryPassword`

  *Type*: string*New in version 3.4:* Available in MongoDB Enterprise only.The password used to bind to an LDAP server when using [`queryUser`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryUser). You must use [`queryPassword`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryPassword) with [`queryUser`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryUser).If unset, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) will not attempt to bind to the LDAP server.This setting can be configured on a running [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) using [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter).NOTEWindows MongoDB deployments can use `bindWithOSDefaults` instead of [`queryPassword`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryPassword) and [`queryPassword`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryPassword). You cannot specify both [`queryPassword`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryPassword) and `bindWithOSDefaults` at the same time.

- `security.ldap.bind.``useOSDefaults`

  *Type*: boolean*Default*: false*New in version 3.4:* Available in MongoDB Enterprise for the Windows platform only.Allows [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) to authenticate, or bind, using your Windows login credentials when connecting to the LDAP server.Only required if:Using [LDAP authorization](https://docs.mongodb.com/master/core/security-ldap-external/#security-ldap-external).Using an LDAP query for [`username transformation`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping).The LDAP server disallows anonymous bindsUse [`useOSDefaults`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.useOSDefaults) to replace [`queryUser`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryUser) and [`queryPassword`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.queryPassword).

- `security.ldap.bind.``method`

  *Type*: string*Default*: simple*New in version 3.4:* Available in MongoDB Enterprise only.The method [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) uses to authenticate to an LDAP server. Use with `queryUser` and `queryPassword` to connect to the LDAP server.[`method`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.method) supports the following values:`simple` - [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) uses simple authentication.`sasl` - [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) uses SASL protocol for authenticationIf you specify `sasl`, you can configure the available SASL mechanisms using [`security.ldap.bind.saslMechanisms`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.saslMechanisms). [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) defaults to using `DIGEST-MD5` mechanism.

- `security.ldap.bind.``saslMechanisms`

  *Type*: string*Default*: DIGEST-MD5*New in version 3.4:* Available in MongoDB Enterprise only.A comma-separated list of SASL mechanisms [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) can use when authenticating to the LDAP server. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) and the LDAP server must agree on at least one mechanism. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) dynamically loads any SASL mechanism libraries installed on the host machine at runtime.Install and configure the appropriate libraries for the selected SASL mechanism(s) on both the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) host and the remote LDAP server host. Your operating system may include certain SASL libraries by default. Defer to the documentation associated with each SASL mechanism for guidance on installation and configuration.If using the `GSSAPI` SASL mechanism for use with [Kerberos Authentication](https://docs.mongodb.com/master/core/kerberos/#security-kerberos), verify the following for the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) host machine:`Linux`The `KRB5_CLIENT_KTNAME` environment variable resolves to the name of the client [Linux Keytab Files](https://docs.mongodb.com/master/core/kerberos/#keytab-files) for the host machine. For more on Kerberos environment variables, please defer to the [Kerberos documentation](https://web.mit.edu/kerberos/krb5-1.13/doc/admin/env_variables.html).The client keytab includes a [User Principal](https://docs.mongodb.com/master/core/kerberos/#kerberos-user-principal) for the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) to use when connecting to the LDAP server and execute LDAP queries.`Windows`If connecting to an Active Directory server, the Windows Kerberos configuration automatically generates a [Ticket-Granting-Ticket](https://msdn.microsoft.com/en-us/library/windows/desktop/aa380510(v=vs.85).aspx) when the user logs onto the system. Set [`useOSDefaults`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.useOSDefaults) to `true` to allow [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) to use the generated credentials when connecting to the Active Directory server and execute queries.Set [`method`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.bind.method) to `sasl` to use this option.NOTEFor a complete list of SASL mechanisms see the [IANA listing](http://www.iana.org/assignments/sasl-mechanisms/sasl-mechanisms.xhtml). Defer to the documentation for your LDAP or Active Directory service for identifying the SASL mechanisms compatible with the service.MongoDB is not a source of SASL mechanism libraries, nor is the MongoDB documentation a definitive source for installing or configuring any given SASL mechanism. For documentation and support, defer to the SASL mechanism library vendor or owner.For more information on SASL, defer to the following resources:For Linux, please see the [Cyrus SASL documentation](https://www.cyrusimap.org/sasl/).For Windows, please see the [Windows SASL documentation](https://msdn.microsoft.com/en-us/library/cc223500.aspx).

- `security.ldap.``transportSecurity`

  *Type*: string*Default*: tls*New in version 3.4:* Available in MongoDB Enterprise only.By default, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) creates a TLS/SSL secured connection to the LDAP server.For Linux deployments, you must configure the appropriate TLS Options in `/etc/openldap/ldap.conf` file. Your operating system’s package manager creates this file as part of the MongoDB Enterprise installation, via the `libldap` dependency. See the documentation for `TLS Options` in the [ldap.conf OpenLDAP documentation](http://www.openldap.org/software/man.cgi?query=ldap.conf&manpath=OpenLDAP+2.4-Release) for more complete instructions.For Windows deployment, you must add the LDAP server CA certificates to the Windows certificate management tool. The exact name and functionality of the tool may vary depending on operating system version. Please see the documentation for your version of Windows for more information on certificate management.Set [`transportSecurity`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.transportSecurity) to `none` to disable TLS/SSL between [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) and the LDAP server.WARNINGSetting [`transportSecurity`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.transportSecurity) to `none` transmits plaintext information and possibly credentials between [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) and the LDAP server.

- `security.ldap.``timeoutMS`

  *Type*: int*Default*: 10000*New in version 3.4:* Available in MongoDB Enterprise only.The amount of time in milliseconds [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) should wait for an LDAP server to respond to a request.Increasing the value of [`timeoutMS`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.timeoutMS) may prevent connection failure between the MongoDB server and the LDAP server, if the source of the failure is a connection timeout. Decreasing the value of [`timeoutMS`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.timeoutMS) reduces the time MongoDB waits for a response from the LDAP server.This setting can be configured on a running [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) using [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter).

- `security.ldap.``userToDNMapping`

  *Type*: string*New in version 3.4:* Available in MongoDB Enterprise only.Maps the username provided to [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) for authentication to a LDAP Distinguished Name (DN). You may need to use [`userToDNMapping`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping) to transform a username into an LDAP DN in the following scenarios:Performing LDAP authentication with simple LDAP binding, where users authenticate to MongoDB with usernames that are not full LDAP DNs.Using an [`LDAP authorization query template`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-ldapauthzquerytemplate) that requires a DN.Transforming the usernames of clients authenticating to Mongo DB using different authentication mechanisms (e.g. x.509, kerberos) to a full LDAP DN for authorization.[`userToDNMapping`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping) expects a quote-enclosed JSON-string representing an ordered array of documents. Each document contains a regular expression `match` and either a `substitution` or `ldapQuery` template used for transforming the incoming username.Each document in the array has the following form:copycopied`{  match: "<regex>"  substitution: "<LDAP DN>" | ldapQuery: "<LDAP Query>" } `FieldDescriptionExample`match`An ECMAScript-formatted regular expression (regex) to match against a provided username. Each parenthesis-enclosed section represents a regex capture group used by `substitution` or `ldapQuery`.`"(.+)ENGINEERING"` `"(.+)DBA"``substitution`An LDAP distinguished name (DN) formatting template that converts the authentication name matched by the `match` regex into a LDAP DN. Each curly bracket-enclosed numeric value is replaced by the corresponding [regex capture group](http://www.regular-expressions.info/refcapture.html) extracted from the authentication username via the `match` regex.The result of the substitution must be an [RFC4514](https://www.ietf.org/rfc/rfc4514.txt) escaped string.`"cn={0},ou=engineering, dc=example,dc=com"``ldapQuery`A LDAP query formatting template that inserts the authentication name matched by the `match` regex into an LDAP query URI encoded respecting RFC4515 and RFC4516. Each curly bracket-enclosed numeric value is replaced by the corresponding [regex capture group](http://www.regular-expressions.info/refcapture.html) extracted from the authentication username via the `match` expression. [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) executes the query against the LDAP server to retrieve the LDAP DN for the authenticated user. [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) requires exactly one returned result for the transformation to be successful, or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) skips this transformation.`"ou=engineering,dc=example, dc=com??one?(user={0})"`NOTEAn explanation of [RFC4514](https://www.ietf.org/rfc/rfc4514.txt), [RFC4515](https://tools.ietf.org/search/rfc4515), [RFC4516](https://tools.ietf.org/html/rfc4516), or LDAP queries is out of scope for the MongoDB Documentation. Please review the RFC directly or use your preferred LDAP resource.For each document in the array, you must use either `substitution` or `ldapQuery`. You *cannot* specify both in the same document.When performing authentication or authorization, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) steps through each document in the array in the given order, checking the authentication username against the `match` filter. If a match is found, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) applies the transformation and uses the output for authenticating the user. [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) does not check the remaining documents in the array.If the given document does not match the provided authentication name, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) continues through the list of documents to find additional matches. If no matches are found in any document, or the transformation the document describes fails, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) returns an error.Starting in MongoDB 4.4, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) also returns an error if one of the transformations cannot be evaluated due to networking or authentication failures to the LDAP server. [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) rejects the connection request and does not check the remaining documents in the array.EXAMPLEThe following shows two transformation documents. The first document matches against any string ending in `@ENGINEERING`, placing anything preceeding the suffix into a regex capture group. The second document matches against any string ending in `@DBA`, placing anything preceeding the suffix into a regex capture group.IMPORTANTYou must pass the array to [`userToDNMapping`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping) as a string.copycopied`"[   {      match: "(.+)@ENGINEERING.EXAMPLE.COM",      substitution: "cn={0},ou=engineering,dc=example,dc=com"   },   {      match: "(.+)@DBA.EXAMPLE.COM",      ldapQuery: "ou=dba,dc=example,dc=com??one?(user={0})"    } ]" `A user with username `alice@ENGINEERING.EXAMPLE.COM` matches the first document. The regex capture group `{0}` corresponds to the string `alice`. The resulting output is the DN `"cn=alice,ou=engineering,dc=example,dc=com"`.A user with username `bob@DBA.EXAMPLE.COM` matches the second document. The regex capture group `{0}` corresponds to the string `bob`. The resulting output is the LDAP query `"ou=dba,dc=example,dc=com??one?(user=bob)"`. [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) executes this query against the LDAP server, returning the result `"cn=bob,ou=dba,dc=example,dc=com"`.If [`userToDNMapping`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping) is unset, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) applies no transformations to the username when attempting to authenticate or authorize a user against the LDAP server.This setting can be configured on a running [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) using the [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter) database command.

- `security.ldap.authz.``queryTemplate`

  *Type*: string*New in version 3.4:* Available in MongoDB Enterprise only.A relative LDAP query URL formatted conforming to [RFC4515](https://tools.ietf.org/search/rfc4515) and [RFC4516](https://tools.ietf.org/html/rfc4516) that [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) executes to obtain the LDAP groups to which the authenticated user belongs to. The query is relative to the host or hosts specified in [`security.ldap.servers`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.servers).In the URL, you can use the following substituion tokens:Substitution TokenDescription`{USER}`Substitutes the authenticated username, or the [`transformed`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping) username if a [`userToDNMapping`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping) is specified.`{PROVIDED_USER}`Substitutes the supplied username, i.e. before either authentication or [`LDAP transformation`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.userToDNMapping).*New in version 4.2.*When constructing the query URL, ensure that the order of LDAP parameters respects RFC4516:copycopied`[ dn  [ ? [attributes] [ ? [scope] [ ? [filter] [ ? [Extensions] ] ] ] ] ] `If your query includes an attribute, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) assumes that the query retrieves a list of the DNs which this entity is a member of.If your query does not include an attribute, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) assumes the query retrieves all entities which the user is member of.For each LDAP DN returned by the query, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) assigns the authorized user a corresponding role on the `admin` database. If a role on the on the `admin` database exactly matches the DN, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) grants the user the roles and privileges assigned to that role. See the [`db.createRole()`](https://docs.mongodb.com/master/reference/method/db.createRole/#db.createRole) method for more information on creating roles.EXAMPLEThis LDAP query returns any groups listed in the LDAP user object’s `memberOf` attribute.copycopied`"{USER}?memberOf?base" `Your LDAP configuration may not include the `memberOf` attribute as part of the user schema, may possess a different attribute for reporting group membership, or may not track group membership through attributes. Configure your query with respect to your own unique LDAP configuration.If unset, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) cannot authorize users using LDAP.This setting can be configured on a running [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) using the [`setParameter`](https://docs.mongodb.com/master/reference/command/setParameter/#dbcmd.setParameter) database command.NOTEAn explanation of [RFC4515](https://tools.ietf.org/search/rfc4515), [RFC4516](https://tools.ietf.org/html/rfc4516) or LDAP queries is out of scope for the MongoDB Documentation. Please review the RFC directly or use your preferred LDAP resource.

- `security.ldap.``validateLDAPServerConfig`

  *Type*: boolean*Default*: true*Available in MongoDB Enterprise*A flag that determines if the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance checks the availability of the [`LDAP server(s)`](https://docs.mongodb.com/master/reference/configuration-options/#security.ldap.servers) as part of its startup:If `true`, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance performs the availability check and only continues to start up if the LDAP server is available.If `false`, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance skips the availability check; i.e. the instance starts up even if the LDAP server is unavailable.

### <span id="setparameter-option">`setParameter` 选项</span>

- `setParameter`

  Set MongoDB parameter or parameters described in [MongoDB Server Parameters](https://docs.mongodb.com/master/reference/parameters/)To set parameters in the YAML configuration file, use the following format:copycopied`setParameter:   <parameter1>: <value1>   <parameter2>: <value2> `For example, to specify the [`enableLocalhostAuthBypass`](https://docs.mongodb.com/master/reference/parameters/#param.enableLocalhostAuthBypass) in the configuration file:copycopied`setParameter:   enableLocalhostAuthBypass: false `

#### LDAP参数

- `setParameter.``ldapUserCacheInvalidationInterval`

  *Type*: int*Default*: 30For use with [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) servers using [LDAP Authorization](https://docs.mongodb.com/master/core/security-ldap-external/#security-ldap-external).The interval (in seconds) [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) waits between external user cache flushes. After [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) flushes the external user cache, MongoDB reacquires authorization data from the LDAP server the next time an LDAP-authorized user issues an operation.Increasing the value specified increases the amount of time [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and the LDAP server can be out of sync, but reduces the load on the LDAP server. Conversely, decreasing the value specified decreases the time [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and the LDAP server can be out of sync while increasing the load on the LDAP server.

copycopied

```
setParameter:
   ldapUserCacheInvalidationInterval: <int>
```

### <span id="storage-options">`storage` 选项</span>

STARTING IN VERSION 4.4

- MongoDB removes the `storage.indexBuildRetry` option and the corresponding `--noIndexBuildRetry` command-line option.
- MongoDB deprecates `storage.wiredTiger.engineConfig.maxCacheOverflowFileSizeGB` option. The option has no effect starting in MongoDB 4.4.

copycopied

```
storage:
   dbPath: <string>
   journal:
      enabled: <boolean>
      commitIntervalMs: <num>
   directoryPerDB: <boolean>
   syncPeriodSecs: <int>
   engine: <string>
   wiredTiger:
      engineConfig:
         cacheSizeGB: <number>
         journalCompressor: <string>
         directoryForIndexes: <boolean>
         maxCacheOverflowFileSizeGB: <number> // deprecated in MongoDB 4.4
      collectionConfig:
         blockCompressor: <string>
      indexConfig:
         prefixCompression: <boolean>
   inMemory:
      engineConfig:
         inMemorySizeGB: <number>
   oplogMinRetentionHours: <double>
```

- `storage.``dbPath`

  *Type*: string*Default*:`/data/db` on Linux and macOS`\data\db` on WindowsThe directory where the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance stores its data.The [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).CONFIGURATION FILESThe default `mongod.conf` configuration file included with package manager installations uses the following platform-specific default values for `storage.dbPath`:PlatformPackage ManagerDefault `storage.dbPath`RHEL / CentOS and Amazon`yum``/var/lib/mongo`SUSE`zypper``/var/lib/mongo`Ubuntu and Debian`apt``/var/lib/mongodb`macOS`brew``/usr/local/var/mongodb`The Linux package init scripts do not expect [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath) to change from the defaults. If you use the Linux packages and change [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath), you will have to use your own init scripts and disable the built-in scripts.

- `storage.journal.``enabled`

  *Type*: boolean*Default*: `true` on 64-bit systems, `false` on 32-bit systemsEnable or disable the durability [journal](https://docs.mongodb.com/master/reference/glossary/#term-journal) to ensure data files remain valid and recoverable. This option applies only when you specify the [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath) setting. [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) enables journaling by default.The [`storage.journal.enabled`](https://docs.mongodb.com/master/reference/configuration-options/#storage.journal.enabled) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).Not available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances that use the [in-memory storage engine](https://docs.mongodb.com/master/core/inmemory/).Starting in MongoDB 4.0, you cannot specify [`--nojournal`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-nojournal) option or [`storage.journal.enabled: false`](https://docs.mongodb.com/master/reference/configuration-options/#storage.journal.enabled) for replica set members that use the WiredTiger storage engine.

- `storage.journal.``commitIntervalMs`

  *Type*: number*Default*: 100The maximum amount of time in milliseconds that the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process allows between journal operations. Values can range from 1 to 500 milliseconds. Lower values increase the durability of the journal, at the expense of disk performance.On WiredTiger, the default journal commit interval is 100 milliseconds. Additionally, a write that includes or implies `j:true` will cause an immediate sync of the journal. For details or additional conditions that affect the frequency of the sync, see [Journaling Process](https://docs.mongodb.com/master/core/journaling/#journal-process).The [`storage.journal.commitIntervalMs`](https://docs.mongodb.com/master/reference/configuration-options/#storage.journal.commitIntervalMs) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).Not available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances that use the [in-memory storage engine](https://docs.mongodb.com/master/core/inmemory/).NOTEKnown Issue in 4.2.0: The [`storage.journal.commitIntervalMs`](https://docs.mongodb.com/master/reference/configuration-options/#storage.journal.commitIntervalMs) is missing in 4.2.0.

- `storage.``directoryPerDB`

  *Type*: boolean*Default*: falseWhen `true`, MongoDB uses a separate directory to store data for each database. The directories are under the [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath) directory, and each subdirectory name corresponds to the database name.The [`storage.directoryPerDB`](https://docs.mongodb.com/master/reference/configuration-options/#storage.directoryPerDB) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).Not available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances that use the [in-memory storage engine](https://docs.mongodb.com/master/core/inmemory/).To change the [`storage.directoryPerDB`](https://docs.mongodb.com/master/reference/configuration-options/#storage.directoryPerDB) option for existing deployments:For standalone instances:Use [`mongodump`](https://docs.mongodb.com/database-tools/mongodump/#bin.mongodump) on the existing [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance to generate a backup.Stop the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance.Add the [`storage.directoryPerDB`](https://docs.mongodb.com/master/reference/configuration-options/#storage.directoryPerDB) value **and** configure a new data directoryRestart the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance.Use [`mongorestore`](https://docs.mongodb.com/database-tools/mongorestore/#bin.mongorestore) to populate the new data directory.For replica sets:Stop a secondary member.Add the [`storage.directoryPerDB`](https://docs.mongodb.com/master/reference/configuration-options/#storage.directoryPerDB) value **and** configure a new data directory to that secondary member.Restart that secondary.Use [initial sync](https://docs.mongodb.com/master/core/replica-set-sync/#replica-set-initial-sync) to populate the new data directory.Update remaining secondaries in the same fashion.Step down the primary, and update the stepped-down member in the same fashion.

- `storage.``syncPeriodSecs`

  *Type*: number*Default*: 60The amount of time that can pass before MongoDB flushes data to the data files via an [fsync](https://docs.mongodb.com/master/reference/glossary/#term-fsync) operation.**Do not set this value on production systems.** In almost every situation, you should use the default setting.WARNINGIf you set [`storage.syncPeriodSecs`](https://docs.mongodb.com/master/reference/configuration-options/#storage.syncPeriodSecs) to `0`, MongoDB will not sync the memory mapped files to disk.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process writes data very quickly to the journal and lazily to the data files. [`storage.syncPeriodSecs`](https://docs.mongodb.com/master/reference/configuration-options/#storage.syncPeriodSecs) has no effect on the [`journal`](https://docs.mongodb.com/master/reference/configuration-options/#storage.journal.enabled) files or [journaling](https://docs.mongodb.com/master/core/journaling/), but if [`storage.syncPeriodSecs`](https://docs.mongodb.com/master/reference/configuration-options/#storage.syncPeriodSecs) is set to `0` the journal will eventually consume all available disk space. If you set [`storage.syncPeriodSecs`](https://docs.mongodb.com/master/reference/configuration-options/#storage.syncPeriodSecs) to `0` for testing purposes, you should also set [`--nojournal`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-nojournal) to `true`.The [`serverStatus`](https://docs.mongodb.com/master/reference/command/serverStatus/#dbcmd.serverStatus) command reports the background flush thread’s status via the `backgroundFlushing` field.The [`storage.syncPeriodSecs`](https://docs.mongodb.com/master/reference/configuration-options/#storage.syncPeriodSecs) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).Not available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances that use the [in-memory storage engine](https://docs.mongodb.com/master/core/inmemory/).

- `storage.``engine`

  *Default*: `wiredTiger`NOTEStarting in version 4.2, MongoDB removes the deprecated MMAPv1 storage engine.The storage engine for the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) database. Available values include:ValueDescription`wiredTiger`To specify the [WiredTiger Storage Engine](https://docs.mongodb.com/master/core/wiredtiger/).`inMemory`To specify the [In-Memory Storage Engine](https://docs.mongodb.com/master/core/inmemory/).*New in version 3.2:* Available in MongoDB Enterprise only.If you attempt to start a [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) with a [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath) that contains data files produced by a storage engine other than the one specified by [`storage.engine`](https://docs.mongodb.com/master/reference/configuration-options/#storage.engine), [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) will refuse to start.

- `storage.``oplogMinRetentionHours`

  *Type*: double*New in version 4.4:* Specifies the minimum number of hours to preserve an oplog entry, where the decimal values represent the fractions of an hour. For example, a value of `1.5` represents one hour and thirty minutes.The value must be greater than or equal to `0`. A value of `0` indicates that the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) should truncate the oplog starting with the oldest entries to maintain the configured maximum oplog size.Defaults to `0`.A [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) started with `oplogMinRetentionHours` only removes an oplog entry *if*:The oplog has reached the maximum configured oplog size *and*The oplog entry is older than the configured number of hours based on the host system clock.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) has the following behavior when configured with a minimum oplog retention period:The oplog can grow without constraint so as to retain oplog entries for the configured number of hours. This may result in reduction or exhaustion of system disk space due to a combination of high write volume and large retention period.If the oplog grows beyond its maximum size, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) may continue to hold that disk space even if the oplog returns to its maximum size *or* is configured for a smaller maximum size. See [Reducing Oplog Size Does Not Immediately Return Disk Space](https://docs.mongodb.com/master/reference/command/replSetResizeOplog/#replsetresizeoplog-cmd-compact).The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) compares the system wall clock to an oplog entries creation wall clock time when enforcing oplog entry retention. Clock drift between cluster components may result in unexpected oplog retention behavior. See [Clock Synchronization](https://docs.mongodb.com/master/administration/production-notes/#production-notes-clock-synchronization) for more information on clock synchronization across cluster members.To change the minimum oplog retention period after starting the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod), use [`replSetResizeOplog`](https://docs.mongodb.com/master/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog). [`replSetResizeOplog`](https://docs.mongodb.com/master/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) enables you to resize the oplog dynamically without restarting the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process. To persist the changes made using [`replSetResizeOplog`](https://docs.mongodb.com/master/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) through a restart, update the value of [`oplogMinRetentionHours`](https://docs.mongodb.com/master/reference/configuration-options/#storage.oplogMinRetentionHours).

#### `storage.wiredTiger` 选项

copycopied

```
storage:
   wiredTiger:
      engineConfig:
         cacheSizeGB: <number>
         journalCompressor: <string>
         directoryForIndexes: <boolean>
         maxCacheOverflowFileSizeGB: <number>   // Deprecated in MongoDB 4.4
      collectionConfig:
         blockCompressor: <string>
      indexConfig:
         prefixCompression: <boolean>
```

- `storage.wiredTiger.engineConfig.``cacheSizeGB`

  *Type*: floatDefines the maximum size of the internal cache that WiredTiger will use for all data. The memory consumed by an index build (see [`maxIndexBuildMemoryUsageMegabytes`](https://docs.mongodb.com/master/reference/parameters/#param.maxIndexBuildMemoryUsageMegabytes)) is separate from the WiredTiger cache memory.Values can range from `0.25` GB to `10000` GB.Starting in MongoDB 3.4, the default WiredTiger internal cache size is the larger of either:50% of (RAM - 1 GB), or256 MB.For example, on a system with a total of 4GB of RAM the WiredTiger cache will use 1.5GB of RAM (`0.5 * (4 GB - 1 GB) = 1.5 GB`). Conversely, a system with a total of 1.25 GB of RAM will allocate 256 MB to the WiredTiger cache because that is more than half of the total RAM minus one gigabyte (`0.5 * (1.25 GB - 1 GB) = 128 MB < 256 MB`).NOTEIn some instances, such as when running in a container, the database can have memory constraints that are lower than the total system memory. In such instances, this memory limit, rather than the total system memory, is used as the maximum RAM available.To see the memory limit, see [`hostInfo.system.memLimitMB`](https://docs.mongodb.com/master/reference/command/hostInfo/#hostInfo.system.memLimitMB).Avoid increasing the WiredTiger internal cache size above its default value.With WiredTiger, MongoDB utilizes both the WiredTiger internal cache and the filesystem cache.Via the filesystem cache, MongoDB automatically uses all free memory that is not used by the WiredTiger cache or by other processes.NOTEThe [`storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB) limits the size of the WiredTiger internal cache. The operating system will use the available free memory for filesystem cache, which allows the compressed MongoDB data files to stay in memory. In addition, the operating system will use any free RAM to buffer file system blocks and file system cache.To accommodate the additional consumers of RAM, you may have to decrease WiredTiger internal cache size.The default WiredTiger internal cache size value assumes that there is a single [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance per machine. If a single machine contains multiple MongoDB instances, then you should decrease the setting to accommodate the other [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances.If you run [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) in a container (e.g. `lxc`, `cgroups`, Docker, etc.) that does *not* have access to all of the RAM available in a system, you must set [`storage.wiredTiger.engineConfig.cacheSizeGB`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.engineConfig.cacheSizeGB) to a value less than the amount of RAM available in the container. The exact amount depends on the other processes running in the container. See [`memLimitMB`](https://docs.mongodb.com/master/reference/command/hostInfo/#hostInfo.system.memLimitMB).

- `storage.wiredTiger.engineConfig.``journalCompressor`

  *Default*: snappySpecifies the type of compression to use to compress WiredTiger journal data.Available compressors are:`none`[snappy](https://docs.mongodb.com/master/reference/glossary/#term-snappy)[zlib](https://docs.mongodb.com/master/reference/glossary/#term-zlib)[zstd](https://docs.mongodb.com/master/reference/glossary/#term-zstd) (Available starting in MongoDB 4.2)

- `storage.wiredTiger.engineConfig.``directoryForIndexes`

  *Type*: boolean*Default*: falseWhen [`storage.wiredTiger.engineConfig.directoryForIndexes`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.engineConfig.directoryForIndexes) is `true`, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) stores indexes and collections in separate subdirectories under the data (i.e. [`storage.dbPath`](https://docs.mongodb.com/master/reference/configuration-options/#storage.dbPath)) directory. Specifically, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) stores the indexes in a subdirectory named `index` and the collection data in a subdirectory named `collection`.By using a symbolic link, you can specify a different location for the indexes. Specifically, when [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance is **not** running, move the `index` subdirectory to the destination and create a symbolic link named `index` under the data directory to the new destination.

- `storage.wiredTiger.engineConfig.``maxCacheOverflowFileSizeGB`

  *Type*: floatDEPRECATED IN MONGODB 4.4MongoDB deprecates the `storage.wiredTiger.engineConfig.maxCacheOverflowFileSizeGB` option. The option has no effect starting in MongoDB 4.4.Specifies the maximum size (in GB) for the “lookaside (or cache overflow) table” file `WiredTigerLAS.wt` for MongoDB 4.2.1-4.2.x and 4.0.12-4.0.x. The file no longer exists starting in version 4.4.The setting can accept the following values:ValueDescription`0`The default value. If set to `0`, the file size is unbounded.number >= 0.1The maximum size (in GB). If the `WiredTigerLAS.wt` file exceeds this size, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) exits with a fatal assertion. You can clear the `WiredTigerLAS.wt` file and restart [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).To change the maximum size during runtime, use the [`wiredTigerMaxCacheOverflowSizeGB`](https://docs.mongodb.com/master/reference/parameters/#param.wiredTigerMaxCacheOverflowSizeGB) parameter.*Available starting in MongoDB 4.2.1 (and 4.0.12)*

- `storage.wiredTiger.collectionConfig.``blockCompressor`

  *Default*: snappySpecifies the default compression for collection data. You can override this on a per-collection basis when creating collections.Available compressors are:`none`[snappy](https://docs.mongodb.com/master/reference/glossary/#term-snappy)[zlib](https://docs.mongodb.com/master/reference/glossary/#term-zlib)[zstd](https://docs.mongodb.com/master/reference/glossary/#term-zstd) (Available starting MongoDB 4.2)[`storage.wiredTiger.collectionConfig.blockCompressor`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.collectionConfig.blockCompressor) affects all collections created. If you change the value of [`storage.wiredTiger.collectionConfig.blockCompressor`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.collectionConfig.blockCompressor) on an existing MongoDB deployment, all new collections will use the specified compressor. Existing collections will continue to use the compressor specified when they were created, or the default compressor at that time.

- `storage.wiredTiger.indexConfig.``prefixCompression`

  *Default*: trueEnables or disables [prefix compression](https://docs.mongodb.com/master/reference/glossary/#term-prefix-compression) for index data.Specify `true` for [`storage.wiredTiger.indexConfig.prefixCompression`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.indexConfig.prefixCompression) to enable [prefix compression](https://docs.mongodb.com/master/reference/glossary/#term-prefix-compression) for index data, or `false` to disable prefix compression for index data.The [`storage.wiredTiger.indexConfig.prefixCompression`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.indexConfig.prefixCompression) setting affects all indexes created. If you change the value of [`storage.wiredTiger.indexConfig.prefixCompression`](https://docs.mongodb.com/master/reference/configuration-options/#storage.wiredTiger.indexConfig.prefixCompression) on an existing MongoDB deployment, all new indexes will use prefix compression. Existing indexes are not affected.

#### `storage.inmemory` 选项

copycopied

```
storage:
   inMemory:
      engineConfig:
         inMemorySizeGB: <number>
```

- `storage.inMemory.engineConfig.``inMemorySizeGB`

  *Type*: float*Default*: 50% of physical RAM less 1 GB*Changed in version 3.4:* Values can range from 256MB to 10TB and can be a float.Maximum amount of memory to allocate for [in-memory storage engine](https://docs.mongodb.com/master/core/inmemory/) data, including indexes, oplog if the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) is part of replica set, replica set or sharded cluster metadata, etc.By default, the in-memory storage engine uses 50% of physical RAM minus 1 GB.ENTERPRISE FEATUREAvailable in MongoDB Enterprise only.



### <span id="operationprofiling-options">`operationProfiling` 选项</span>

copycopied

```
operationProfiling:
   mode: <string>
   slowOpThresholdMs: <int>
   slowOpSampleRate: <double>
```



- `operationProfiling.``mode`

  *Type*: string*Default*: `off`Specifies which operations should be [profiled](https://docs.mongodb.com/master/tutorial/manage-the-database-profiler/). The following profiler levels are available:LevelDescription`off`The profiler is off and does not collect any data. This is the default profiler level.`slowOp`The profiler collects data for operations that take longer than the value of `slowms`.`all`The profiler collects data for all operations.IMPORTANTProfiling can impact performance and shares settings with the system log. Carefully consider any performance and security implications before configuring and enabling the profiler on a production deployment.See [Profiler Overhead](https://docs.mongodb.com/master/tutorial/manage-the-database-profiler/#database-profiling-overhead) for more information on potential performance degradation.



- `operationProfiling.``slowOpThresholdMs`

  *Type*: integer*Default*: 100The *slow* operation time threshold, in milliseconds. Operations that run for longer than this threshold are considered *slow*.When [`logLevel`](https://docs.mongodb.com/master/reference/parameters/#param.logLevel) is set to `0`, MongoDB records *slow* operations to the diagnostic log at a rate determined by [`slowOpSampleRate`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpSampleRate). Starting in MongoDB 4.2, the secondaries of replica sets log [all oplog entry messages that take longer than the slow operation threshold to apply](https://docs.mongodb.com/master/release-notes/4.2/#slow-oplog) regardless of the sample rate.At higher [`logLevel`](https://docs.mongodb.com/master/reference/parameters/#param.logLevel) settings, all operations appear in the diagnostic log regardless of their latency with the following exception: the logging of [slow oplog entry messages by the secondaries](https://docs.mongodb.com/master/release-notes/4.2/#slow-oplog). The secondaries log only the slow oplog entries; increasing the [`logLevel`](https://docs.mongodb.com/master/reference/parameters/#param.logLevel) does not log all oplog entries.*Changed in version 4.0:* The [`slowOpThresholdMs`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpThresholdMs) setting is available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos). In earlier versions, [`slowOpThresholdMs`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpThresholdMs) is available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) only.For [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances, the setting affects both the diagnostic log and, if enabled, the profiler.For [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instances, the setting affects the diagnostic log only and not the profiler since profiling is not available on [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos).



- `operationProfiling.``slowOpSampleRate`

  *Type*: double*Default*: 1.0The fraction of *slow* operations that should be profiled or logged. [`operationProfiling.slowOpSampleRate`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpSampleRate) accepts values between 0 and 1, inclusive.[`operationProfiling.slowOpSampleRate`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpSampleRate) does not affect the [slow oplog entry logging](https://docs.mongodb.com/master/release-notes/4.2/#slow-oplog) by the secondary members of a replica set. Secondary members log all oplog entries that take longer than the slow operation threshold regardless of the [`operationProfiling.slowOpSampleRate`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpSampleRate).*Changed in version 4.0:* The [`slowOpSampleRate`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpSampleRate) setting is available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos). In earlier versions, [`slowOpSampleRate`](https://docs.mongodb.com/master/reference/configuration-options/#operationProfiling.slowOpSampleRate) is available for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) only.For [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances, the setting affects both the diagnostic log and, if enabled, the profiler.For [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instances, the setting affects the diagnostic log only and not the profiler since profiling is not available on [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos).



### <span id="replication-options">`replication` 选项</span>

copycopied

```
replication:
   oplogSizeMB: <int>
   replSetName: <string>
   enableMajorityReadConcern: <boolean>
```

- `replication.``oplogSizeMB`

  *Type*: integerThe maximum size in megabytes for the replication operation log (i.e., the [oplog](https://docs.mongodb.com/master/reference/glossary/#term-oplog)).NOTEStarting in MongoDB 4.0, the oplog can grow past its configured size limit to avoid deleting the [`majority commit point`](https://docs.mongodb.com/master/reference/command/replSetGetStatus/#replSetGetStatus.optimes.lastCommittedOpTime).By default, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process creates an [oplog](https://docs.mongodb.com/master/reference/glossary/#term-oplog) based on the maximum amount of space available. For 64-bit systems, the oplog is typically 5% of available disk space.Once the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) has created the oplog for the first time, changing the [`replication.oplogSizeMB`](https://docs.mongodb.com/master/reference/configuration-options/#replication.oplogSizeMB) option will not affect the size of the oplog. To change the maximum oplog size after starting the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod), use [`replSetResizeOplog`](https://docs.mongodb.com/master/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog). [`replSetResizeOplog`](https://docs.mongodb.com/master/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) enables you to resize the oplog dynamically without restarting the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) process. To persist the changes made using [`replSetResizeOplog`](https://docs.mongodb.com/master/reference/command/replSetResizeOplog/#dbcmd.replSetResizeOplog) through a restart, update the value of [`oplogSizeMB`](https://docs.mongodb.com/master/reference/configuration-options/#replication.oplogSizeMB).See [Oplog Size](https://docs.mongodb.com/master/core/replica-set-oplog/#replica-set-oplog-sizing) for more information.The [`replication.oplogSizeMB`](https://docs.mongodb.com/master/reference/configuration-options/#replication.oplogSizeMB) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).

- `replication.``replSetName`

  *Type*: stringThe name of the replica set that the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) is part of. All hosts in the replica set must have the same set name.If your application connects to more than one replica set, each set should have a distinct name. Some drivers group replica set connections by replica set name.The [`replication.replSetName`](https://docs.mongodb.com/master/reference/configuration-options/#replication.replSetName) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).Starting in MongoDB 4.0:The setting [`replication.replSetName`](https://docs.mongodb.com/master/reference/configuration-options/#replication.replSetName) cannot be used in conjunction with `storage.indexBuildRetry`.For the WiredTiger storage engine, [`storage.journal.enabled: false`](https://docs.mongodb.com/master/reference/configuration-options/#storage.journal.enabled) cannot be used in conjunction with [`replication.replSetName`](https://docs.mongodb.com/master/reference/configuration-options/#replication.replSetName).

- `replication.``enableMajorityReadConcern`

  *Default*: trueStarting in MongoDB 3.6, MongoDB enables support for [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") read concern by default.You can disable read concern [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") to prevent the storage cache pressure from immobilizing a deployment with a three-member primary-secondary-arbiter (PSA) architecture. For more information about disabling read concern [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority"), see [Disable Read Concern Majority](https://docs.mongodb.com/master/reference/read-concern-majority/#disable-read-concern-majority).To disable, set [`replication.enableMajorityReadConcern`](https://docs.mongodb.com/master/reference/configuration-options/#replication.enableMajorityReadConcern) to false. [`replication.enableMajorityReadConcern`](https://docs.mongodb.com/master/reference/configuration-options/#replication.enableMajorityReadConcern) has no effect for MongoDB versions: 4.0.0, 4.0.1, 4.0.2, 3.6.0.IMPORTANTIn general, avoid disabling [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") read concern unless necessary. However, if you have a three-member replica set with a primary-secondary-arbiter (PSA) architecture or a sharded cluster with a three-member PSA shards, disable to prevent the storage cache pressure from immobilizing the deployment.Disabling [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") read concern affects support for [transactions](https://docs.mongodb.com/master/core/transactions/) on sharded clusters. Specifically:A transaction cannot use read concern [`"snapshot"`](https://docs.mongodb.com/master/reference/read-concern-snapshot/#readconcern."snapshot") if the transaction involves a shard that has [disabled read concern “majority”](https://docs.mongodb.com/master/reference/read-concern-majority/#disable-read-concern-majority).A transaction that writes to multiple shards errors if any of the transaction’s read or write operations involves a shard that has disabled read concern [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority").However, it does not affect [transactions](https://docs.mongodb.com/master/core/transactions/) on replica sets. For transactions on replica sets, you can specify read concern [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") (or [`"snapshot"`](https://docs.mongodb.com/master/reference/read-concern-snapshot/#readconcern."snapshot") or [`"local"`](https://docs.mongodb.com/master/reference/read-concern-local/#readconcern."local") ) for multi-document transactions even if read concern [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") is disabled.Disabling [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") read concern prevents [`collMod`](https://docs.mongodb.com/master/reference/command/collMod/#dbcmd.collMod) commands which modify an index from [rolling back](https://docs.mongodb.com/master/core/replica-set-rollbacks/#replica-set-rollbacks). If such an operation needs to be rolled back, you must resync the affected nodes with the [primary](https://docs.mongodb.com/master/reference/glossary/#term-primary) node.Disabling [`"majority"`](https://docs.mongodb.com/master/reference/read-concern-majority/#readconcern."majority") read concern disables support for [Change Streams](https://docs.mongodb.com/master/changeStreams/) for MongoDB 4.0 and earlier. For MongoDB 4.2+, disabling read concern `"majority"` has no effect on change streams availability.

### <span id="sharding-options">`sharding` 选项</span>

copycopied

```
sharding:
   clusterRole: <string>
   archiveMovedChunks: <boolean>
```

- `sharding.``clusterRole`

  *Type*: stringThe role that the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance has in the sharded cluster. Set this setting to one of the following:ValueDescription`configsvr`Start this instance as a [config server](https://docs.mongodb.com/master/reference/glossary/#term-config-server). The instance starts on port `27019` by default.`shardsvr`Start this instance as a [shard](https://docs.mongodb.com/master/reference/glossary/#term-shard). The instance starts on port `27018` by default.NOTESetting `sharding.clusterRole` requires the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instance to be running with replication. To deploy the instance as a replica set member, use the [`replSetName`](https://docs.mongodb.com/master/reference/configuration-options/#replication.replSetName) setting and specify the name of the replica set.The [`sharding.clusterRole`](https://docs.mongodb.com/master/reference/configuration-options/#sharding.clusterRole) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).

- `sharding.``archiveMovedChunks`

  *Type*: boolean*Changed in version 3.2:* Starting in 3.2, MongoDB uses `false` as the default.During chunk migration, a shard does not save documents migrated from the shard.

### <span id="auditlog-options">`auditLog` 选项</span>

NOTE

Available only in [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) and [MongoDB Atlas](https://cloud.mongodb.com/user#/atlas/login).

copycopied

```
auditLog:
   destination: <string>
   format: <string>
   path: <string>
   filter: <string>
```

- `auditLog.``destination`

  *Type*: stringWhen set, [`auditLog.destination`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.destination) enables [auditing](https://docs.mongodb.com/master/core/auditing/) and specifies where [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) sends all audit events.[`auditLog.destination`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.destination) can have one of the following values:ValueDescription`syslog`Output the audit events to syslog in JSON format. Not available on Windows. Audit messages have a syslog severity level of `info` and a facility level of `user`.The syslog message limit can result in the truncation of audit messages. The auditing system will neither detect the truncation nor error upon its occurrence.`console`Output the audit events to `stdout` in JSON format.`file`Output the audit events to the file specified in [`auditLog.path`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.path) in the format specified in [`auditLog.format`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.format).NOTEAvailable only in [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) and [MongoDB Atlas](https://cloud.mongodb.com/user#/atlas/login).

- `auditLog.``format`

  *Type*: stringThe format of the output file for [auditing](https://docs.mongodb.com/master/core/auditing/) if [`destination`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.destination) is `file`. The [`auditLog.format`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.format) option can have one of the following values:ValueDescription`JSON`Output the audit events in JSON format to the file specified in [`auditLog.path`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.path).`BSON`Output the audit events in BSON binary format to the file specified in [`auditLog.path`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.path).Printing audit events to a file in JSON format degrades server performance more than printing to a file in BSON format.NOTEAvailable only in [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) and [MongoDB Atlas](https://cloud.mongodb.com/user#/atlas/login).

- `auditLog.``path`

  *Type*: stringThe output file for [auditing](https://docs.mongodb.com/master/core/auditing/) if [`destination`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.destination) has value of `file`. The [`auditLog.path`](https://docs.mongodb.com/master/reference/configuration-options/#auditLog.path) option can take either a full path name or a relative path name.NOTEAvailable only in [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) and [MongoDB Atlas](https://cloud.mongodb.com/user#/atlas/login).

- `auditLog.``filter`

  *Type*: string representation of a documentThe filter to limit the [types of operations](https://docs.mongodb.com/master/reference/audit-message/#audit-action-details-results) the [audit system](https://docs.mongodb.com/master/core/auditing/) records. The option takes a string representation of a query document of the form:copycopied`{ <field1>: <expression1>, ... } `The `<field>` can be [any field in the audit message](https://docs.mongodb.com/master/reference/audit-message/), including fields returned in the [param](https://docs.mongodb.com/master/reference/audit-message/#audit-action-details-results) document. The `<expression>` is a [query condition expression](https://docs.mongodb.com/master/reference/operator/query/#query-selectors).To specify an audit filter, enclose the filter document in single quotes to pass the document as a string.To specify the audit filter in a [configuration file](https://docs.mongodb.com/master/reference/configuration-options/#), you must use the YAML format of the configuration file.NOTEAvailable only in [MongoDB Enterprise](http://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) and [MongoDB Atlas](https://cloud.mongodb.com/user#/atlas/login).

### <span id="snmp-options">`snmp` 选项</span>

NOTE

MongoDB Enterprise on macOS does *not* include support for SNMP due to [SERVER-29352](https://jira.mongodb.org/browse/SERVER-29352).

copycopied

```
snmp:
   disabled: <boolean>
   subagent: <boolean>
   master: <boolean>
```

- `snmp.``disabled`

  *Type*: boolean*Default*: falseDisables SNMP access to [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod). The option is incompatible with [`snmp.subagent`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.subagent) and [`snmp.master`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.master).Set to `true` to disable SNMP access.The [`snmp.disabled`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.disabled) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).*New in version 4.0.6.*

- `snmp.``subagent`

  *Type*: booleanWhen [`snmp.subagent`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.subagent) is `true`, SNMP runs as a subagent. The option is incompatible with [`snmp.disabled`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.disabled) set to `true`.The [`snmp.subagent`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.subagent) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).

- `snmp.``master`

  *Type*: booleanWhen [`snmp.master`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.master) is `true`, SNMP runs as a master. The option is incompatible with [`snmp.disabled`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.disabled) set to `true`.The [`snmp.master`](https://docs.mongodb.com/master/reference/configuration-options/#snmp.master) setting is available only for [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod).

SEE ALSO

- [Monitor MongoDB With SNMP on Linux](https://docs.mongodb.com/master/tutorial/monitor-with-snmp/)
- [Monitor MongoDB Windows with SNMP](https://docs.mongodb.com/master/tutorial/monitor-with-snmp-on-windows/)
- [Troubleshoot SNMP](https://docs.mongodb.com/master/tutorial/troubleshoot-snmp/)

## <span id="mongos-only-options">`mongos`-only 选项</span>

*Changed in version 3.4:* MongoDB 3.4 removes `sharding.chunkSize` and `sharding.autoSplit` settings.

copycopied

```
replication:
   localPingThresholdMs: <int>

sharding:
   configDB: <string>
```

- `replication.``localPingThresholdMs`

  *Type*: integer*Default*: 15The ping time, in milliseconds, that [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) uses to determine which secondary replica set members to pass read operations from clients. The default value of `15` corresponds to the default value in all of the client [drivers](https://docs.mongodb.com/ecosystem/drivers).When [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) receives a request that permits reads to [secondary](https://docs.mongodb.com/master/reference/glossary/#term-secondary) members, the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) will:Find the member of the set with the lowest ping time.Construct a list of replica set members that is within a ping time of 15 milliseconds of the nearest suitable member of the set.If you specify a value for the [`replication.localPingThresholdMs`](https://docs.mongodb.com/master/reference/configuration-options/#replication.localPingThresholdMs) option, [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) will construct the list of replica members that are within the latency allowed by this value.Select a member to read from at random from this list.The ping time used for a member compared by the [`replication.localPingThresholdMs`](https://docs.mongodb.com/master/reference/configuration-options/#replication.localPingThresholdMs) setting is a moving average of recent ping times, calculated at most every 10 seconds. As a result, some queries may reach members above the threshold until the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) recalculates the average.See the [Read Preference for Replica Sets](https://docs.mongodb.com/master/core/read-preference-mechanics/#replica-set-read-preference-behavior-member-selection) section of the [read preference](https://docs.mongodb.com/master/core/read-preference/) documentation for more information.

- `sharding.``configDB`

  *Type*: string*Changed in version 3.2.*The [configuration servers](https://docs.mongodb.com/master/core/sharded-cluster-config-servers/#sharding-config-server) for the [sharded cluster](https://docs.mongodb.com/master/reference/glossary/#term-sharded-cluster).Starting in MongoDB 3.2, config servers for sharded clusters can be deployed as a [replica set](https://docs.mongodb.com/master/replication/). The replica set config servers must run the [WiredTiger storage engine](https://docs.mongodb.com/master/core/wiredtiger/). MongoDB 3.2 deprecates the use of three mirrored [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) instances for config servers.Specify the config server replica set name and the hostname and port of at least one of the members of the config server replica set.copycopied`sharding:  configDB: <configReplSetName>/cfg1.example.net:27019, cfg2.example.net:27019,... `The [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instances for the sharded cluster must specify the same config server replica set name but can specify hostname and port of different members of the replica set.

## <span id="windows-service-options">Windows Service 选项</span>

copycopied

```
processManagement:
   windowsService:
      serviceName: <string>
      displayName: <string>
      description: <string>
      serviceUser: <string>
      servicePassword: <string>
```

- `processManagement.windowsService.``serviceName`

  *Type*: string*Default*: MongoDBThe service name of [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) when running as a Windows Service. Use this name with the `net start <name>` and `net stop <name>` operations.You must use [`processManagement.windowsService.serviceName`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.windowsService.serviceName) in conjunction with either the `--install` or `--remove` option.

- `processManagement.windowsService.``displayName`

  *Type*: string*Default*: MongoDBThe name listed for MongoDB on the Services administrative application.

- `processManagement.windowsService.``description`

  *Type*: string*Default*: MongoDB ServerRun [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) service description.You must use [`processManagement.windowsService.description`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.windowsService.description) in conjunction with the `--install` option.For descriptions that contain spaces, you must enclose the description in quotes.

- `processManagement.windowsService.``serviceUser`

  *Type*: stringThe [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) service in the context of a certain user. This user must have “Log on as a service” privileges.You must use [`processManagement.windowsService.serviceUser`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.windowsService.serviceUser) in conjunction with the `--install` option.

- `processManagement.windowsService.``servicePassword`

  *Type*: stringThe password for `<user>` for [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) when running with the [`processManagement.windowsService.serviceUser`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.windowsService.serviceUser) option.You must use [`processManagement.windowsService.servicePassword`](https://docs.mongodb.com/master/reference/configuration-options/#processManagement.windowsService.servicePassword) in conjunction with the `--install` option.

## <span id="removed-mmapv1-options">Removed MMAPv1 选项</span>

Starting in version 4.2, MongoDB removes the deprecated MMAPv1 storage engine and the MMAPv1-specific configuration options:

| Removed Configuration File Setting        | Removed Command-line Option  |
| :---------------------------------------- | :--------------------------- |
| `storage.mmapv1.journal.commitIntervalMs` |                              |
| `storage.mmapv1.journal.debugFlags`       | `mongod --journalOptions`    |
| `storage.mmapv1.nsSize`                   | `mongod --nssize`            |
| `storage.mmapv1.preallocDataFiles`        | `mongod --noprealloc`        |
| `storage.mmapv1.quota.enforced`           | `mongod --quota`             |
| `storage.mmapv1.quota.maxFilesPerDB`      | `mongod --quotaFiles`        |
| `storage.mmapv1.smallFiles`               | `mongod --smallfiles`        |
| `storage.repairPath`                      | `mongod --repairpath`        |
| `replication.secondaryIndexPrefetch`      | `mongod --replIndexPrefetch` |

For earlier versions of MongoDB, refer to the corresponding version of the manual. For example:

- [https://docs.mongodb.com/v4.0](https://docs.mongodb.com/v4.0/reference/configuration-options)
- [https://docs.mongodb.com/v3.6](https://docs.mongodb.com/v3.6/reference/configuration-options)
- [https://docs.mongodb.com/v3.4](https://docs.mongodb.com/v3.4/reference/configuration-options)