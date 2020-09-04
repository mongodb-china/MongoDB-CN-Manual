# 系统事件审计消息[¶](#system-event-audit-messages "Permalink to this headline")

在本页

*   [审计消息](#audit-message)
*   [审计事件操作，详情和结果](#audit-event-actions-details-and-results)


注意

仅在[MongoDB 企业版](http://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs)和[MongoDB Atlas](https://cloud.mongodb.com/user#/atlas/login)可用。


## 审计消息[¶](#audit-message "Permalink to this headline")


[事件审计功能](../../core/auditing/)可以用JSON格式记录事件。配置审计输出，请参阅[配置审计](../../tutorial/configure-auditing/)。

记录的JSON消息格式如下：

复制

```js
{
  atype: <String>,
  ts : { "$date": <timestamp> },
  local: { ip: <String>, port: <int> },
  remote: { ip: <String>, port: <int> },
  users : [ { user: <String>, db: <String> }, ... ],
  roles: [ { role: <String>, db: <String> }, ... ],
  param: <document>,
  result: <int>
}
```


| 字段 | 类型 | 描述|
|:---:|:---:|:---|
|`atype` | string | 操作类型. 详情请看[审计事件操作，详情和结果](#audit-action-details-results). |
|`ts`| document| 文档包含日期和UTC时间格式为ISO 8601 |
|`local`| document | 文档包含运行实例本地IP和端口 |
|`remote`| document | 文档包含与事件相关的传入连接的远程ip和端口号 |
|`users` | array | 数组包含一组用户识别文档。由于MongoDB允许会话以每个数据库的不同用户身份登录，因此该数组可以包含多个用户。每个文档都包含user字段记录用户名和db字段记录验证该用户的数据库名 |
|`roles`| array | 数组包含文档，用于指定授予用户的角色。每个文档包含一个role字段记录角色名和一个db字段记录与该角色相关的数据库名 |
| `param` | document | 事件的详细信息。请看[审计事件操作，详情和结果](#audit-action-details-results). |
|`result` | integer | 错误码。请看[审计事件操作，详情和结果](#audit-action-details-results). |



## 审计事件操作，详情和结果[¶](#audit-event-actions-details-and-results "Permalink to this headline")


下表列出了每种atype或操作类型，相关的param详细信息和result值(如果有)。



| `atype`                                                      | `param`                                                      | `result`                                                     |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `authenticate`                                               | `{  user: ,  db: ,  mechanism:  }`                           | `0` - Success  `18` - Authentication Failed                  |
| `authCheck`                                                  | `{  command:,  ns:.,  args:}` ns字段是可选的。args字段可能已经被修改了。 | `0` - Success`13` - Unauthorized to perform the operation.默认情况下，审计系统仅记录授权失败。要使系统记录授权成功，请使用[`auditAuthorizationSuccess`](https://docs.mongodb.com/manual/reference/parameters/#param.auditAuthorizationSuccess)参数。[[1\]](https://docs.mongodb.com/manual/reference/audit-message/index.html#performance) |
| [`createCollection`](https://docs.mongodb.com/manual/reference/privilege-actions/#createCollection) | `{ ns:.}`                                                    | `0` - Success                                                |
| `createDatabase`                                             | `{ ns:}`                                                     | `0` - Success                                                |
| [`createIndex`](https://docs.mongodb.com/manual/reference/privilege-actions/#createIndex) | `{  ns:.,  indexName:,  indexSpec:}`                         | `0` - Success                                                |
| `renameCollection`                                           | `{  old:.,  new:.}`                                          | `0` - Success                                                |
| [`dropCollection`](https://docs.mongodb.com/manual/reference/privilege-actions/#dropCollection) | `{ ns:.}`                                                    | `0` - Success                                                |
| [`dropDatabase`](https://docs.mongodb.com/manual/reference/privilege-actions/#dropDatabase) | `{ ns:}`                                                     | `0` - Success                                                |
| [`dropIndex`](https://docs.mongodb.com/manual/reference/privilege-actions/#dropIndex) | `{  ns:.,  indexName:}`                                      | `0` - Success                                                |
| [`createUser`](https://docs.mongodb.com/manual/reference/privilege-actions/#createUser) | `{  user:,  db:,  customData:,  roles: [      {        role: ,        db:       },      ...   ] }`customData字段是可选的。 | `0` - Success                                                |
| [`dropUser`](https://docs.mongodb.com/manual/reference/privilege-actions/#dropUser) | `{  user:,  db:}`                                            | `0` - Success                                                |
| `dropAllUsersFromDatabase`                                   | `{ db:}`                                                     | `0` - Success                                                |
| `updateUser`                                                 | `{  user:,  db:,  passwordChanged:,  customData:,  roles: [     {       role:,       db: },     ...  ] }` `customData` 字段是可选的。 | `0` - Success                                                |
| `grantRolesToUser`                                           | `{   user: ,   db: ,   roles: [      {        role: ,        db:       },      ...   ] }` | `0` - Success                                                |
| `revokeRolesFromUser`                                        | `{   user: ,   db: ,   roles: [      {        role: ,        db:       },      ...   ] }` | `0` - Success                                                |
| [`createRole`](https://docs.mongodb.com/manual/reference/privilege-actions/#createRole) | `{  role:,  db:,  roles: [     {       role:,       db:     },     ...  ],  privileges: [    {      resource:,      actions: [, ... ]    },    ...  ] }`roles和privileges字段是可选的，关于resource文档详情，请查看[Resource Document](https://docs.mongodb.com/manual/reference/resource-document/#resource-document).关于操作列表，请查看[Privilege Actions](https://docs.mongodb.com/manual/reference/privilege-actions/#security-user-actions). | `0` - Success                                                |
| `updateRole`                                                 | `{  role:,  db:,  roles: [     {       role:,       db:     },     ...  ],  privileges: [    {      resource:,      actions: [, ... ]    },    ...  ] }`roles和privileges字段是可选的。关于resource文档详情，请查看[Resource Document](https://docs.mongodb.com/manual/reference/resource-document/#resource-document).关于操作列表，请查看[Privilege Actions](https://docs.mongodb.com/manual/reference/privilege-actions/#security-user-actions). | `0` - Success                                                |
| [`dropRole`](https://docs.mongodb.com/manual/reference/privilege-actions/#dropRole) | `{  role:,  db: }`                                           | `0` - Success                                                |
| `dropAllRolesFromDatabase`                                   | `{ db: }`                                                    | `0` - Success                                                |
| `grantRolesToRole`                                           | `{  role:,  db:,  roles: [     {       role:,       db:     },     ...  ] }` | `0` - Success                                                |
| `revokeRolesFromRole`                                        | `{  role:,  db:,  roles: [     {       role:,       db:     },     ...  ] }` | `0` - Success                                                |
| `grantPrivilegesToRole`                                      | `{  role:,  db:,  privileges: [    {      resource:,      actions: [, ... ]    },    ...  ] }`关于resource这个字段对应的文档，请查看[Resource Document](https://docs.mongodb.com/manual/reference/resource-document/#resource-document).关于操作列表，请查看[Privilege Actions](https://docs.mongodb.com/manual/reference/privilege-actions/#security-user-actions). | `0` - Success                                                |
| `revokePrivilegesFromRole`                                   | `{  role:,  db:,  privileges: [    {      resource:,      actions: [, ... ]    },    ...  ] }`关于resource这个字段对应的文档，请查看[Resource Document](https://docs.mongodb.com/manual/reference/resource-document/#resource-document).关于操作列表，请查看[Privilege Actions](https://docs.mongodb.com/manual/reference/privilege-actions/#security-user-actions). | `0` - Success                                                |
| `replSetReconfig`                                            | `{  old: {   _id:<replicaSetName>,   version:<number>,   ...   members: [ ... ],   settings: { ... }  },  new: {   _id:<replicaSetName>,   version:<number>,   ...   members: [ ... ],   settings: { ... }  } }关于副本集配置对应的文档, 请查看 [Replica Set Configuration](https://docs.mongodb.com/manual/reference/replica-configuration/). | `0` - Success                                                |
| [`enableSharding`](https://docs.mongodb.com/manual/reference/privilege-actions/#enableSharding) | `{ ns:}`                                                     | `0` - Success                                                |
| `shardCollection`                                            | `{  ns:.,  key:,  options: { unique:} }`                     | `0` - Success                                                |
| [`addShard`](https://docs.mongodb.com/manual/reference/privilege-actions/#addShard) | `{  shard:,  connectionString::,  maxSize:}`当分片是副本集时，connectionString包含副本集集群名字和可以包含其他副本集成员。 | `0` - Success                                                |
| [`removeShard`](https://docs.mongodb.com/manual/reference/privilege-actions/#removeShard) | `{ shard:}`                                                  | `0` - Success                                                |
| [`shutdown`](https://docs.mongodb.com/manual/reference/privilege-actions/#shutdown) | `{ }`Indicates commencement of database shutdown. 指明数据库开始关闭 | `0` - Success                                                |
| [`applicationMessage`](https://docs.mongodb.com/manual/reference/privilege-actions/#applicationMessage) | `{ msg:}`请查看[logApplicationMessage](https://docs.mongodb.com/manual/reference/command/logApplicationMessage/#dbcmd.logApplicationMessage). | `0` - Success                                                |


[[1]](#id1)启用[审计授权成功](../parameters/#param.auditAuthorizationSuccess "auditAuthorizationSuccess")与仅记录授权失败相比，启用会使性能下降更多。



原文链接：https://docs.mongodb.com/manual/reference/audit-message/

译者：谢伟成

