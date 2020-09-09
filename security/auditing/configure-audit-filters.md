# 配置审计过滤器

在本页

* [`--auditFilter` 选项](configure-audit-filters.md#auditfilter-option)
* [示例](configure-audit-filters.md#examples)

MongoDB Atlas 中的审计

MongoDB Atlas支持对所有M10和更大的集群进行审计。

Atlas支持在[配置审计过滤器](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/Security/configure-audit-filters/README.md)中指定JSON格式的审计过滤器，并使用Atlas审计过滤器构建器来简化审计配置。

要了解更多信息，请参阅Atlas文档中的[设置数据库审计](https://docs.atlas.mongodb.com/database-auditing)和[配置自定义审计过滤器](https://docs.atlas.mongodb.com/tutorial/auditing-custom-filter)。

[MongoDB 企业版](https://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs)支持[审计](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/core/auditing/README.md#auditing)各种操作。

启用审计功能会默认的记录所有可审计的操作，如[审计事件操作，详细信息和结果](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/audit-message/README.md#audit-action-details-results)。

为了能指定那些事件需要被记录，审计功能包含`--auditFilter`选项。

注意

从mongoDB 3.6开始，[`mongod`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/program/mongod/README.md#bin.mongod) and [`mongos`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/program/mongos/README.md#bin.mongos)默认绑定localhost。

如果你部署的实例运行在不同的主机上或者如果你希望远程客户端连接到部署实例，你必须指定`--bind_ip` or [`net.bindIp`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md#net.bindIp).

更多信息，请查看[Localhost 绑定兼容性更改](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/release-notes/3.6-compatibility#bind-ip-compatibility)。

绑定到其他IP地址之前，请考虑启用[访问控制](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/administration/security-checklist/README.md#checklist-auth)和[“安全性检查表”](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/administration/security-checklist/README.md)中的列出的其他安全措施，以防止未经授权的访问。

## `--auditFilter` 选项[¶](configure-audit-filters.md#auditfilter-option)

--auditFilter\`选项采用以下查询文档的字符串的表示形式：

复制

```javascript
{ <field1>: <expression1>, ... }
```

* `<field>`可以是[审计消息](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/audit-message/README.md)中的任何字段，包括[param](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/audit-message/README.md#audit-action-details-results)文档中返回的字段。
* `<expression>`是一个[查询条件表达式](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/operator/query/README.md#query-selectors)。

指定一个审计过滤器，可以将过滤器文档括在单引号中使其转成字符串。

在[配置文件](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md)中指定审计过滤器，必须使用配置文件的YAML格式。

## 例子[¶](configure-audit-filters.md#examples)

### 多种操作类型的过滤器[¶](configure-audit-filters.md#filter-for-multiple-operation-types)

以下示例通过使用过滤器仅审计 [`createCollection`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/privilege-actions/README.md#createCollection) 和 [`dropCollection`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/privilege-actions/README.md#dropCollection)操作：

复制

```javascript
{ atype: { $in: ["createCollection", "dropCollection"] } }
```

指定一个审计过滤器，可以将过滤器文档括在单引号中使其转成字符串。

复制

```bash
mongod --dbpath data/db --auditDestination file --auditFilter '{ atype: { $in: [ "createCollection", "dropCollection" ] } }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，​​请指定--bind\_ip参数。更多信息，请参见[Localhost绑定兼容性更改](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/release-notes/3.6-compatibility#bind-ip-compatibility)。

在[配置文件](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md)中指定审计过滤器，必须使用配置文件的YAML格式。

复制

```text
storage:
   dbPath: data/db
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
   filter: '{ atype: { $in: [ "createCollection", "dropCollection" ] } }'
```

### 筛选单个数据库上的身份验证操作[¶](configure-audit-filters.md#filter-on-authentication-operations-on-a-single-database)

`<field>`可以包含[审计消息](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/audit-message/README.md)中的任何字段。对于身份认证操作\(即，`atype: "authenticate"`\)，审计消息中的 `param` 文档中包含 `db` 字段。

以下示例使用过滤器仅审计针对test数据库的身份验证操作。

复制

```javascript
{ atype: "authenticate", "param.db": "test" }
```

指定一个审计过滤器，可以将过滤器文档括在单引号中使其转成字符串。

复制

```bash
mongod --dbpath data/db --auth --auditDestination file --auditFilter '{ atype: "authenticate", "param.db": "test" }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定--bind\_ip参数。更多信息，请参见[Localhost绑定兼容性更改](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/release-notes/3.6-compatibility#bind-ip-compatibility)。

在[配置文件](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md)中指定审计过滤器，必须使用配置文件的YAML格式。

复制

```text
storage:
   dbPath: data/db
security:
   authorization: enabled
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
   filter: '{ atype: "authenticate", "param.db": "test" }'
```

要过滤数据库中的所有身份验证操作，请使用过滤器`{ atype: "authenticate" }`。

### 筛选单个数据库的集合创建和删除操作[¶](configure-audit-filters.md#filter-on-collection-creation-and-drop-operations-for-a-single-database)

`<field>`可以包含[审计消息](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/audit-message/README.md)中的任何字段。对于集合创建和删除操作\(即，`atype: "createCollection"`和`atype: "dropCollection"`\)，审计消息中的 `param` 文档中包含`ns` 字段。

以下示例使用过滤器仅审计针对test数据库的创建集合和删除集合操作。

注意

正则表达式需要两个反斜杠\(`\\`\)才能转义\(`.`\)

复制

```bash
{ atype: { $in: [ "createCollection", "dropCollection" ] }, "param.ns": /^test\\./ } }
```

将过滤器文档括在单引号中使其转成字符串来指定一个审计过滤器。

复制

```bash
mongod --dbpath data/db --auth --auditDestination file --auditFilter '{ atype: { $in: [ "createCollection", "dropCollection" ] }, "param.ns": /^test\\./ } }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定 `--bind_ip`参数。更多信息，请参见[Localhost绑定兼容性更改](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/release-notes/3.6-compatibility#bind-ip-compatibility)。

在[配置文件](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md)中指定审计过滤器，必须使用配置文件的YAML格式。

复制

```text
storage:
   dbPath: data/db
security:
   authorization: enabled
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
   filter: '{ atype: { $in: [ "createCollection", "dropCollection" ] }, "param.ns": /^test\\./ } }'
```

### 通过授权角色进行筛选[¶](configure-audit-filters.md#filter-by-authorization-role)

以下示例通过使用过滤器来审计`test`数据库上具有 [`readWrite`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/built-in-roles/README.md#readWrite)角色的用户的操作，包括具有从\[`readWrite`\]继承的角色的用户：

复制

```javascript
{ roles: { role: "readWrite", db: "test" } }
```

指定一个审计过滤器，可以将过滤器文档括在单引号中使其转成字符串。

复制

```bash
mongod --dbpath data/db --auth --auditDestination file --auditFilter '{ roles: { role: "readWrite", db: "test" } }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定 `--bind_ip`参数。更多信息，请参见[Localhost绑定兼容性更改](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/release-notes/3.6-compatibility#bind-ip-compatibility)。

在[配置文件](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md)中指定审计过滤器，必须使用配置文件的YAML格式。

复制

```text
storage:
   dbPath: data/db
security:
   authorization: enabled
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
   filter: '{ roles: { role: "readWrite", db: "test" } }'
```

### 读写操作中的过滤器[¶](configure-audit-filters.md#filter-on-read-and-write-operations)

要在审计中进行捕获读和写操作，必须设置[审计](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/parameters/README.md#param.auditAuthorizationSuccess)参数使审计系统记录身份验证成功。[1](configure-audit-filters.md#authorization-agnostic)

注意

启用[审计授权成功](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/parameters/README.md#param.auditAuthorizationSuccess)与仅记录授权失败相比会使性能下降更多。

下面的例子用来审计[`find()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.find#db.collection.find), [`insert()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.insert#db.collection.insert), [`remove()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.remove#db.collection.remove), [`update()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.update#db.collection.update), [`save()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.save#db.collection.save)和 [`findAndModify()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.findAndModify#db.collection.findAndModify)这些操作，过滤器如下：

复制

```javascript
{ atype: "authCheck", "param.command": { $in: [ "find", "insert", "delete", "update", "findandmodify" ] } }
```

指定一个审计过滤器，可以将过滤器文档括在单引号中使其转成字符串。

复制

```bash
mongod --dbpath data/db --auth --setParameter auditAuthorizationSuccess=true --auditDestination file --auditFilter '{ atype: "authCheck", "param.command": { $in: [ "find", "insert", "delete", "update", "findandmodify"] } }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定--bind\_ip参数。更多信息，请参见[Localhost绑定兼容性更改](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/release-notes/3.6-compatibility#bind-ip-compatibility)。

在[配置文件](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md)中指定审计过滤器，必须使用配置文件的YAML格式。

复制

```text
storage:
   dbPath: data/db
security:
   authorization: enabled
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
   filter: '{ atype: "authCheck", "param.command": { $in: [ "find", "insert", "delete", "update", "findandmodify" ] } }'
setParameter: { auditAuthorizationSuccess: true }
```

### 过滤集合的读写操作[¶](configure-audit-filters.md#filter-on-read-and-write-operations-for-a-collection)

要在审计中进行捕获读和写操作，还必须使用 [`auditAuthorizationSuccess`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/parameters/README.md#param.auditAuthorizationSuccess) 参数使审计系统能够记录授权成功。 [1](configure-audit-filters.md#authorization-agnostic)

注意

启用[审计授权成功](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/Security/parameters/README.md#param.auditAuthorizationSuccess)与仅记录授权失败相比，启用会使性能下降更多。

下面的例子用来审计在test数据库的orders集合上的[`find()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.find#db.collection.find), [`insert()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.insert#db.collection.insert), [`remove()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.remove#db.collection.remove), [`update()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.update#db.collection.update), [`save()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.save#db.collection.save), and [`findAndModify()`](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/method/db.collection.findAndModify#db.collection.findAndModify)操作，过滤器如下：

复制

```javascript
{ atype: "authCheck", "param.ns": "test.orders", "param.command": { $in: [ "find", "insert", "delete", "update", "findandmodify" ] } }
```

指定一个审计过滤器，可以将过滤器文档括在单引号中使其转成字符串。

复制

```bash
mongod --dbpath data/db --auth --setParameter auditAuthorizationSuccess=true --auditDestination file --auditFilter '{ atype: "authCheck", "param.ns": "test.orders", "param.command": { $in: [ "find", "insert", "delete", "update", "findandmodify" ] } }' --auditFormat BSON --auditPath data/db/auditLog.bson
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定 `--bind_ip`参数。有关更多信息，请参见[Localhost绑定兼容性更改](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/release-notes/3.6-compatibility#bind-ip-compatibility)。

在[配置文件](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/configuration-options/README.md)中指定审计过滤器，必须使用配置文件的YAML格式。

复制

```text
storage:
   dbPath: data/db
security:
   authorization: enabled
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
   filter: '{ atype: "authCheck", "param.ns": "test.orders", "param.command": { $in: [ "find", "insert", "delete", "update", "findandmodify" ] } }'
setParameter: { auditAuthorizationSuccess: true }
```

也可以看看

[配置审计](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/Security/configure-auditing/README.md), [审计](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/core/auditing/README.md), [系统事件审计消息](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/audit-message/README.md)

\[1\]（1，2）可以启用[审计授权成功](https://github.com/mongodb-china/MongoDB-CN-Manual/tree/8490376c81d56eff95abbaddc6ee414b1e1c9705/docs/reference/parameters/README.md#param.auditAuthorizationSuccess)参数不启用 `--auth`; 但是所有操作将返回成功以进行授权检查。

原文链接：[https://docs.mongodb.com/manual/tutorial/configure-audit-filters/](https://docs.mongodb.com/manual/tutorial/configure-audit-filters/)

译者：谢伟成

