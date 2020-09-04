## 退出代码和状态

退出时，MongoDB将返回以下代码和状态之一。使用本指南解释日志和故障排除与[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例的问题。

| 编码  | Cause                                                        |
| :---- | :----------------------------------------------------------- |
| `0`   | 成功退出时由MongoDB应用程序返回。                            |
| `2`   | 指定的选项出现错误或与其他选项不兼容。                       |
| `3`   | Returned by [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) if there is a mismatch between hostnames specified on the command line and in the `local.sources` collection, in master/slave mode.<br /><u>返回的[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)如果在命令行上指定的主机名和在`local.sources`中指定的主机名不匹配。源的收集，在主/从模式。<br/>[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)如果`local.sources`在主/从模式下，命令行和集合中指定的主机名之间不匹配，则 返回</u> |
| `4`   | 数据库的版本不同于[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)(或[`mongod.exe`](https://docs.mongodb.com/master/reference/program/mongod.exe/#bin.mongod.exe))实例所支持的版本。实例干净地退出。 |
| `5`   | 如果在初始化过程中遇到问题，返回[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)。 |
| `12`  | 当收到一个Control-C，关闭，中断或关闭事件，在Windows上返回[`mongod.exe`](https://docs.mongodb.com/master/reference/program/mongod.exe/#bin.mongod.exe)进程。 |
| `14`  | 由MongoDB应用程序返回，遇到一个不可恢复的错误，一个未捕获的异常或未捕获的信号。系统退出时不执行完全关闭。 |
| `20`  | *消息*：`错误:wsastartup failed<reason>`在wsastartup函数出现错误后，由Windows上的MongoDB应用程序返回，用于初始化网络子系统。*消息*：`NT Service ERROR`由于安装、启动或删除应用程序的NT服务失败而返回。 |
| `48`  | 由于一个错误,一个新启动的[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)或[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)不能开始监听传入连接。 |
| `62`  | 返回[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ # bin.mongod)如果datafiles [`--dbpath`](https://docs.mongodb.com/master/reference/program/mongod/ # cmdoption-mongod-dbpath)的版本不兼容[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)当前正在运行。 |
| `100` | 当进程抛出一个未捕获的异常时，返回[`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)。 |