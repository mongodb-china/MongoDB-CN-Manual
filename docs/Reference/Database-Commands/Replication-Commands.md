# [ ](#)复制命令

[]()

> **注意**
>
> 有关特定命令的详细信息，包括语法和示例，请单击特定命令以转到其参考页面。

| 名称                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| [`applyOps`]()                   | 应用于内部命令OPLOG条目到当前数据集。                        |
| [`isMaster`]()                   | 显示有关此成员在副本集中的角色的信息，包括它是否为主角色。   |
| [`replSetAbortPrimaryCatchUp`]() | 强制选择的主数据库中止同步（追赶），然后完成到主数据库的过渡。 |
| [`replSetFreeze`]()              | 防止当前成员在一段时间内寻求选举为主。                       |
| [`replSetGetConfig`]()           | 返回副本集的配置对象。                                       |
| [`replSetGetStatus`]()           | 返回报告副本集状态的文档。                                   |
| [`replSetInitiate`]()            | 初始化新的副本集。                                           |
| [`replSetMaintenance`]()         | 启用或禁用维护模式，该模式将辅助节点置于一种`RECOVERING`状态。 |
| [`replSetReconfig`]()            | 将新配置应用于现有副本集。                                   |
| [`replSetResizeOplog`]()         | 动态调整副本集成员的操作日志的大小。仅适用于WiredTiger存储引擎。 |
| [`replSetStepDown`]()            | 当前`primary`下台,成为一个`secondary`，迫使选举。            |
| [`replSetSyncFrom`]()            | 显式覆盖用于选择要复制的成员的默认逻辑。                     |



译者：李冠飞

校对：
