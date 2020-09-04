# [审计](https://docs.mongodb.com/manual/core/auditing/)

在本页

- [启用和配置审计输出](https://docs.mongodb.com/manual/core/auditing/#enable-and-configure-audit-output)
- [审计事件和过滤器](https://docs.mongodb.com/manual/core/auditing/#audit-events-and-filter)
- [审计保证](https://docs.mongodb.com/manual/core/auditing/#audit-guarantee)

MongoDB 企业版包含针对 mongod 和 mongos 实例的审计功能 。审核功能使管理员和用户可以跟踪具有多个用户和多个客户端应用的 mongodb 的运行情况。


## [启用和配置审计输出](https://docs.mongodb.com/manual/core/auditing/#enable-and-configure-audit-output)

审计功能可以将审计事件写入控制台console，syslog，JSON 文件或 BSON 文件。要为 MongoDB 企业版启用审计，请参阅[配置审计](https://docs.mongodb.com/manual/tutorial/configure-auditing/)。

有关审计日志消息的信息，请参阅[系统事件审计消息](https://docs.mongodb.com/manual/reference/audit-message/)。

## [审计事件和过滤器](https://docs.mongodb.com/manual/core/auditing/#audit-events-and-filter)

启用后，审计系统可以记录以下操作[1]:

- 模式（DDL）,
- 副本集集群和分片集群，
- 认证和授权，以及
- CRUD操作（要求auditAuthorizationSuccess设置为true）。

有关审计的操作的详细信息，请参阅[审计事件操作，详细信息和结果](https://docs.mongodb.com/manual/reference/audit-message/#audit-action-details-results)。

使用审计系统，您可以[设置过滤器](https://docs.mongodb.com/manual/tutorial/configure-audit-filters/#audit-filter)以限制捕获的事件。要设置过滤器，请参阅[“配置审计过滤器”](https://docs.mongodb.com/manual/tutorial/configure-audit-filters/)。

在一个被中止的事务中[1]中的操作任然会生成一个审计事件，但是没有一个审计事件指示事务被中止了。

## [审计保证](https://docs.mongodb.com/manual/core/auditing/#audit-guarantee)

审计系统将每个审计事件[2](https://docs.mongodb.com/manual/core/auditing/#filter)写入审计事件的内存缓冲区中。MongoDB定期将此缓冲区写入磁盘。对于从任何单个连接收集的事件，这些事件具有总顺序：如果MongoDB将一个事件写入磁盘，系统将保证已将该连接的所有先前事件写入磁盘。

如果审计事件条目对应的操作影响数据库的持久状态，如修改数据的操作，则MongoDB始终会在将审计事件写入磁盘之前将事件条目写入日志

也就是说，在将操作添加到[日志](https://docs.mongodb.com/manual/reference/glossary/#term-journal)之前，MongoDB会在触发该操作的连接上写入所有审计事件，直到并包括该操作的条目。

这些审计保证要求MongoDB在[journaling](https://docs.mongodb.com/manual/reference/configuration-options/#storage.journal.enabled)启用的情况下运行 。


## 警告

如果服务器在将事件提交到审计日志之前终止，则MongoDB可能会丢失事件。在MongoDB提交审计日志之前，客户端可能会收到事件确认。

例如，在审计聚合操作时，服务器可能在返回结果之后但在刷新审计日志之前崩溃。

[2](https://docs.mongodb.com/manual/core/auditing/#id3)审计配置可以包括一个[筛选器](https://docs.mongodb.com/manual/tutorial/configure-audit-filters/#audit-filter)，以限制要审计的事件。

## 附录：
Configure Auditing 配置审计：[https://docs.mongodb.com/manual/tutorial/configure-auditing/](https://docs.mongodb.com/manual/tutorial/configure-auditing/)

Configure Audit Filters  配置审计过滤器：[https://docs.mongodb.com/manual/tutorial/configure-audit-filters/](https://docs.mongodb.com/manual/tutorial/configure-audit-filters/)

System Event Audit Messages 系统事件审计消息： [https://docs.mongodb.com/manual/reference/audit-message/](https://docs.mongodb.com/manual/reference/audit-message/)



原文链接：https://docs.mongodb.com/manual/core/auditing/

译者：谢伟成
