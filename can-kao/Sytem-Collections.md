# 系统集合

## 概要

MongoDB将系统信息存储在使用`<database>.system.*`[命名空间](https://docs.mongodb.com/manual/reference/glossary/#std-term-namespace)的集合中，这些集合是MongoDB保留供内部使用的。用户请不要创建以`system`开头的集合。 

MongoDB还将一些额外的本地元数据存储在[`local`数据库](https://docs.mongodb.com/manual/reference/local-database/)中，专门用于主从复制；并在[`config`数据库](https://docs.mongodb.com/manual/reference/config-database/)中存储[会话信息](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#std-label-sessions)。

## 集合

系统集合包括存储在`admin`数据库中的以下集合：

- **`admin.system.roles`**

  [`admin.system.roles`](https://docs.mongodb.com/manual/reference/system-collections/#mongodb-data-admin.system.roles)集合存储管理员创建并分配给用户的自定义角色，以提供对特定资源的访问。

- **`admin.system.users`**

  [`admin.system.users`](https://docs.mongodb.com/manual/reference/system-collections/#mongodb-data-admin.system.users)集合存储用户的身份验证凭据以及分配给该用户的所有角色。用户可以在 [`admin.system.roles`](https://docs.mongodb.com/manual/reference/system-collections/#mongodb-data-admin.system.roles)集合中定义授权角色。

- **`admin.system.version`**

  [`admin.system.version`](https://docs.mongodb.com/manual/reference/system-collections/#mongodb-data-admin.system.version)集合存储元数据以支持内部操作。除非本文档或MongoDB支持工程师明确指示，否则请勿修改此集合。 

系统集合包括存储在`config`数据库中的以下集合：

- **`config.system.indexBuilds`**

  *4.4版本新引入*

  `indexBuilds`集合存储了与正在进行的索引创建有关的信息。


系统集合还包括以下直接存储在每个数据库中的集合： 

- **`<database>.system.namespaces`**

  > **注意**
  >
  > **4.2版本中被移除**
  >
  > 从MongoDB 4.2开始，`<database>.system.namespaces`已被删除（从3.0开始不推荐使用该集合）。要列出数据库中的集合，请改用[`listCollections`](https://docs.mongodb.com/manual/reference/command/listCollections/#mongodb-dbcommand-dbcmd.listCollections)命令。 

- **`<database>.system.indexes`**

  > **注意**
  >
  > **4.2版本中被移除**
  >
  > 从MongoDB 4.2开始，`<database>.system.indexes`已被删除（从3.0开始不推荐使用该集合）。要列出数据库中的集合，请改用[`listIndexes`](https://docs.mongodb.com/manual/reference/command/listIndexes/#mongodb-dbcommand-dbcmd.listIndexes)命令。 

- **`<database>.system.profile`**

  `<database>.system.profile`集合存储数据库分析信息。有关分析的信息，请参见[数据库分析](https://docs.mongodb.com/manual/administration/analyzing-mongodb-performance/#std-label-database-profiling)。

- **`<database>.system.js`**

  `<database>.system.js`集合包含用于[服务器端JavaScript](https://docs.mongodb.com/manual/core/server-side-javascript/)的特殊JavaScript代码。有关更多信息，请参见在[服务器上存储JavaScript函数](https://docs.mongodb.com/manual/tutorial/store-javascript-function-on-server/)。

- **`<database>.system.views`**

  `<database>.system.views`集合包含有关数据库中每个[视图](https://docs.mongodb.com/manual/core/views/)的信息。
  
 从 MongoDB 5.0 开始，对于设置为或更高的featureCompatibilityVersion"5.0"，用户不能再直接写入<database>.system.views集合。
  

--------

译者：phoenix

时间： 2021.04.26

原文： https://docs.mongodb.com/manual/reference/system-collections/

版本： 4.4
