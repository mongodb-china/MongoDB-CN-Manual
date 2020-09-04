# 服务器会话

**在本页面**

- [概述](#overview)
- [命令选项](#command)
- [会话命令](#session)
- [会话和访问控制](#control)

*新增3.6版*

## <span id="overview">概述</span>

MongoDB的服务器会话或逻辑会话是[客户端会话](https://docs.mongodb.com/master/release-notes/3.6/#client-sessions)用来支持[因果一致性](https://docs.mongodb.com/master/core/read-isolation-consistency-recency/#causal-consistency)和可[重试写入](https://docs.mongodb.com/master/core/retryable-writes/#retryable-writes)的基础框架。

> 重要
>
> 应用程序使用[客户端会话](https://docs.mongodb.com/master/relee-notes/3.6 /# clients -sessions)与服务器会话进行接口。
>
> 服务器会话仅可用于复制集和分片集群。

## <span id="command">命令选项</span>

从3.6开始，MongoDB驱动程序将所有操作与一个服务器会话关联起来，除了未确认的写操作。以下选项可用于所有命令，以支持与服务器会话的关联:

> 重要
>
> [` mongo `](https://docs.mongodb.com/master/reference/program/mongo/#bin.mongo) shell和驱动程序在会话中将这些选项分配给命令。

| 选项        | 类型     | 描述                                                         |
| :---------- | :------- | :----------------------------------------------------------- |
| `lsid`      | 文档     | 指定与命令关联的会话的唯一id的文档。如果指定了 **txnNumber** ，则需要 **lsid** 。 |
| `txnNumber` | 64位整数 | 一个严格递增的非负数，用于在命令的会话中唯一标识该命令。如果指定了该命令，则该命令还必须包含 **lsid** 选项。 |

[`删除`](https://docs.mongodb.com/master/reference/command/delete/ dbcmd.delete),[`插入`](https://docs.mongodb.com/master/reference/command/insert/ # dbcmd.insert),和[`更新`](https://docs.mongodb.com/master/reference/command/update/ # dbcmd.update)命令,采取一系列的语句,也可用以下选择:

对于[`delete`](https://docs.mongodb.com/master/reference/command/delete/#dbcmd.delete)，[`insert`](https://docs.mongodb.com/master/reference/command/insert/#dbcmd.insert)和[`update`](https://docs.mongodb.com/master/reference/command/update/#dbcmd.update) 命令,采取一系列的语句，以下选项也可：

> 重要
>
> 不要手动设置 **stmtIds** 。MongoDB将 **stmtIds** 设置为严格的非负数递增。

| 选项      | Type           | Description                                |
| :-------- | :------------- | :----------------------------------------- |
| `stmtIds` | 32位整数的数组 | 在写命令中唯一标识其各自写操作的数字数组。 |

## <span id="session">会话命令</span>

The following commands can be used to list, manage, and kill server sessions throughout MongoDB clusters:

| 命令                                                         | 描述                                 |
| :----------------------------------------------------------- | :----------------------------------- |
| [`endSessions`](https://docs.mongodb.com/master/reference/command/endSessions/#dbcmd.endSessions) | 指定的服务器会话过期。               |
| [`killAllSessions`](https://docs.mongodb.com/master/reference/command/killAllSessions/#dbcmd.killAllSessions) | 终止所有服务器会话。                 |
| [`killAllSessionsByPattern`](https://docs.mongodb.com/master/reference/command/killAllSessionsByPattern/#dbcmd.killAllSessionsByPattern) | 终止与指定模式匹配的所有服务器会话。 |
| [`killSessions`](https://docs.mongodb.com/master/reference/command/killSessions/#dbcmd.killSessions) | 终止指定的服务器会话。               |
| [`refreshSessions`](https://docs.mongodb.com/master/reference/command/refreshSessions/#dbcmd.refreshSessions) | 刷新空闲服务器会话。                 |
| [`startSession`](https://docs.mongodb.com/master/reference/command/startSession/#dbcmd.startSession) | 启动一个新的服务器会话。             |

## <span id="control">会话和访问控制</span>

如果部署强制执行身份验证/授权，则必须对用户进行身份验证才能启动会话，并且只有该用户才能使用该会话

*在版本3.6.3中更改:*使用 **$external** 身份验证用户(例如，Kerberos、LDAP、x.509个用户)，用户名不能大于10k字节。

如果部署不强制执行身份验证/授权，则创建的会话没有所有者，并且可以由任何用户在任何连接上使用。如果用户对不执行身份验证/授权的部署进行身份验证并创建会话，则该用户将拥有该会话。但是，任何连接上的任何用户都可以使用该会话。

如果部署在没有任何停机的情况下转换到身份验证，则不能使用任何没有所有者的会话。

另请参阅：

[`maxSessions`](https://docs.mongodb.com/master/reference/parameters/#param.maxSessions)