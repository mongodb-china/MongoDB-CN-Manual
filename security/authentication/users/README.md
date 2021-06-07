# 用户

On this page

* [User Management Interface](https://docs.mongodb.com/v4.2/core/security-users/#user-management-interface)
* [Authentication Database](https://docs.mongodb.com/v4.2/core/security-users/#user-authentication-database)
* [Authenticate a User](https://docs.mongodb.com/v4.2/core/security-users/#authenticate-a-user)
* [Centralized User Data](https://docs.mongodb.com/v4.2/core/security-users/#centralized-user-data)
* [Sharded Cluster Users](https://docs.mongodb.com/v4.2/core/security-users/#sharded-cluster-users)
* [Localhost Exception](https://docs.mongodb.com/v4.2/core/security-users/#localhost-exception)

To authenticate a client in MongoDB, you must add a corresponding user to MongoDB.

在此页

* [用户管理接口](https://docs.mongodb.com/v4.2/core/security-users/#user-management-interface)
* [数据库认证](https://docs.mongodb.com/v4.2/core/security-users/#user-authentication-database)
* [用户认证](https://docs.mongodb.com/v4.2/core/security-users/#authenticate-a-user)
* [用户数据中心](https://docs.mongodb.com/v4.2/core/security-users/#centralized-user-data)
* [分片集群用户](https://docs.mongodb.com/v4.2/core/security-users/#sharded-cluster-users)
* [本地主机异常](https://docs.mongodb.com/v4.2/core/security-users/#localhost-exception)

要在MongoDB中认证客户端，必须向MongoDB数据库中添加相应的用户。

## User Management Interface

To add a user, MongoDB provides the [`db.createUser()`](https://docs.mongodb.com/v4.2/reference/method/db.createUser/#db.createUser) method. When adding a user, you can assign [roles](https://docs.mongodb.com/v4.2/core/authorization/) to the user in order to grant privileges.

NOTE

The first user created in the database should be a user administrator who has the privileges to manage other users. See [Enable Access Control](https://docs.mongodb.com/v4.2/tutorial/enable-authentication/).

You can also update existing users, such as to change password and grant or revoke roles. For a full list of user management methods, see [User Management](https://docs.mongodb.com/v4.2/reference/method/#user-management-methods).

A user is uniquely identified by the user’s name and associated authentication database. Starting in MongoDB 4.0.9, a users managed by MongoDB are assigned a unique `userId`. [\[1\]](https://docs.mongodb.com/v4.2/core/security-users/#userid)

SEE ALSO

[Add Users](https://docs.mongodb.com/v4.2/tutorial/create-users/)

## 用户管理接口

添加一个用户，MongoDB提供 [`db.createUser()`](https://docs.mongodb.com/v4.2/reference/method/db.createUser/#db.createUser) 方法，当添加一个用户，同时需要分配角色 [roles](https://docs.mongodb.com/v4.2/core/authorization/) 来向用户授予特权。

> 注意 创建第一个用户，需要具有管理其他用户特权的管理员用户。请参见 [启用访问控制](https://docs.mongodb.com/v4.2/tutorial/enable-authentication/) 。 你还可以更新现有用户，例如更改密码和授予或撤销角色。有关用户管理方法的完整列表，请参见 [用户管理](https://docs.mongodb.com/v4.2/reference/method/#user-management-methods) 。 用户是由用户名和关联的身份验证数据库唯一标识的。从MongoDB 4.0.9开始，MongoDB管理的用户被分配一个唯一的 `userId`。[详情请参考](https://docs.mongodb.com/v4.2/core/security-users/#userid)

## Authentication Database

When adding a user, you create the user in a specific database. This database is the authentication database for the user.

A user can have privileges across different databases; that is, a user’s privileges are not limited to their authentication database. By assigning to the user roles in other databases, a user created in one database can have permissions to act on other databases. For more information on roles, see [Role-Based Access Control](https://docs.mongodb.com/v4.2/core/authorization/).

The user’s name and authentication database serve as a unique identifier for that user. [\[1\]](https://docs.mongodb.com/v4.2/core/security-users/#userid) That is, if two users have the same name but are created in different databases, they are two separate users. If you intend to have a single user with permissions on multiple databases, create a single user with roles in the applicable databases instead of creating the user multiple times in different databases.

| \[1\] |  _\(_[_1_](https://docs.mongodb.com/v4.2/core/security-users/#id2) _,_ [_2_](https://docs.mongodb.com/v4.2/core/security-users/#id4) _\)_Starting in version 4.0.9, MongoDB associates a user with a unique `userId` upon creation in MongoDB.[LDAP managed users](https://docs.mongodb.com/v4.2/core/security-ldap/#security-ldap) created on the LDAP server do not have an associated document in the [system.users](https://docs.mongodb.com/v4.2/reference/system-users-collection/) collection, and hence, do not have a [`userId`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.userId) field associated with them. |
| :--- | :--- |
|  |  |

## 数据库认证

添加用户时，在特定数据库中创建用户。这个数据库是该用户的身份验证数据库。 一个用户可以拥有跨不同数据库进行授权；也就是说，用户的授权并不局限于他们的身份验证数据库。 通过将用户角色分配给其他数据库中的用户，在一个数据库中创建的用户可以拥有对其他数据库进行操作的权限。有关角色的更多信息，请参见 [基于角色的访问控制](https://docs.mongodb.com/v4.2/core/authorization/) 。 用户名和身份验证数据库作为该用户的唯一标识符。 [唯一标识UserID详解](https://docs.mongodb.com/v4.2/core/secur-users/#userid) ，也就是说，如果两个用户具有相同的名称，但在不同的数据库中创建，那么他们是两个独立的用户。如果您打算拥有一个对多个数据库具有权限的用户，请创建一个具有适用数据库中的角色的用户，而不是在不同的数据库中多次创建该用户。

## Authenticate a User

To authenticate as a user, you must provide a username, password, and the [authentication database](https://docs.mongodb.com/v4.2/reference/program/mongo/#mongo-shell-authentication-options) associated with that user.

To authenticate using the [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell, either:

* Use the [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) command-line authentication options \([`--username`](https://docs.mongodb.com/v4.2/reference/program/mongo/#cmdoption-mongo-username), [`--password`](https://docs.mongodb.com/v4.2/reference/program/mongo/#cmdoption-mongo-password), and [`--authenticationDatabase`](https://docs.mongodb.com/v4.2/reference/program/mongo/#cmdoption-mongo-authenticationdatabase)\) when connecting to the [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instance, or
* Connect first to the [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) or [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instance, and then run the [`authenticate`](https://docs.mongodb.com/v4.2/reference/command/authenticate/#dbcmd.authenticate) command or the [`db.auth()`](https://docs.mongodb.com/v4.2/reference/method/db.auth/#db.auth) method against the [authentication database](https://docs.mongodb.com/v4.2/reference/program/mongo/#mongo-shell-authentication-options).

  IMPORTANT

  Authenticating multiple times as different users does **not** drop the credentials of previously-authenticated users. This may lead to a connection having more permissions than intended by the user, and causes operations within a [logical session](https://docs.mongodb.com/v4.2/reference/server-sessions/) to raise an error.

For examples of authenticating using a MongoDB driver, see the [driver documentation](https://docs.mongodb.com/ecosystem/drivers/).

## 用户授权

作为用户进行身份验证，您必须提供与该用户相关联的用户名、密码。详情请看 [身份验证数据库](https://docs.mongodb.com/v4.2/reference/program/mongo/#mongo-shell-authentication-options) 。 使用 ['mongo'](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) Shell命令进行认证:

* 使用 ['mongo'](https://docs.mongodb.com/v4.2/reference/program/mongo/bin.mongo) 命令进行身份验证，选项为\( ['--usernmae'](https://docs.mongodb.com/v4.2/reference/program/mongo/#cmdoption-mongo-username), [`--password`](https://docs.mongodb.com/v4.2/reference/program/mongo/#cmdoption-mongo-password), 和 [`--authenticationDatabase`](https://docs.mongodb.com/v4.2/reference/program/mongo/cmdoption-mongo-authenticationdatabase) \),之后连接到 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/bin.mongod) 或 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 实例,或
* 首先连接到 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) 或 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 实例，然后运行 [`authenticate`](https://docs.mongodb.com/v4.2/reference/command/authenticate/dbcmd.authenticate) 命令或 [`db.auth()`](https://docs.mongodb.com/v4.2/reference/method/db.auth/db.auth) 方法,详情请参考 [数据库认证授权](https://docs.mongodb.com/v4.2/reference/program/mongo/#mongo-shell-authentication-options) 。

  > 重要的 不同用户多次身份验证，因为后面的用户不会删除之前经过身份验证的用户的凭据。这可能导致连接时，拥有比用户预期更多的权限，并导致 [逻辑会话](https://docs.mongodb.com/v4.2/reference/server-sessions/)中的操作引发错误。

* 有关使用MongoDB驱动程序进行身份验证的示例，请参阅 [驱动程序文档](https://docs.mongodb.com/ecostem/drivers/) 。

## Centralized User Data

For users created in MongoDB, MongoDB stores all user information, including [`name`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.user), [`password`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.credentials), and the [`user's authentication database`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.db), in the [system.users](https://docs.mongodb.com/v4.2/reference/system-users-collection/) collection in the `admin` database.

Do not access this collection directly but instead use the [user management commands](https://docs.mongodb.com/v4.2/reference/command/#user-management-commands).

## 用户数据集中心

在MongoDB创建的用户, MongoDB存储所有用户信息,包括 [`name`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.user), [`password`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.credentials) ,和 [`用户的权限认证数据库`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.db), 在 [system.users](https://docs.mongodb.com/v4.2/reference/systemusers-collection/) 集合的`admin`数据库中。 不要直接访问集合，而是使用 [用户管理命令](https://docs.mongodb.com/v4.2/reference/command/#user-management-commands) 。

## Sharded Cluster Users

To create users for a sharded cluster, connect to the [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instance and add the users. Clients then authenticate these users through the [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instances. In sharded clusters, MongoDB stores user configuration data in the `admin` database of the [config servers](https://docs.mongodb.com/v4.2/reference/glossary/#term-config-server).

## 分片集群用户

为分片集群创建用户，先连接到 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 实例并添加用户。然后客户端通过 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 实例对这些用户进行身份验证。在分片集群中，MongoDB将用户配置数据存储在 [配置服务器](https://docs.mongodb.com/v4.2/reference/glossary/#term-config-server) 的`admin` 数据库中。

### Shard Local Users

However, some maintenance operations, such as [`cleanupOrphaned`](https://docs.mongodb.com/v4.2/reference/command/cleanupOrphaned/#dbcmd.cleanupOrphaned), [`compact`](https://docs.mongodb.com/v4.2/reference/command/compact/#dbcmd.compact), [`rs.reconfig()`](https://docs.mongodb.com/v4.2/reference/method/rs.reconfig/#rs.reconfig), require direct connections to specific shards in a sharded cluster. To perform these operations, you must connect directly to the shard and authenticate as a _shard local_ administrative user.

To create a _shard local_ administrative user, connect directly to the shard and create the user. MongoDB stores _shard local_ users in the `admin` database of the shard itself.

These _shard local_ users are completely independent from the users added to the sharded cluster via [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos). _Shard local_ users are local to the shard and are inaccessible by [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos).

Direct connections to a shard should only be for shard-specific maintenance and configuration. In general, clients should connect to the sharded cluster through the [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos).

### 共享本地用户

一些维护操作,如 [`cleanupOrphaned`](https://docs.mongodb.com/v4.2/reference/command/cleanupOrphaned/#dbcmd.cleanupOrphaned), [`compact`](https://docs.mongodb.com/v4.2/reference/command/compact/#dbcmd.compact), [`rs.reconfig()`](https://docs.mongodb.com/v4.2/reference/method/rs.reconfig/#rs.reconfig) ,在分片集群中，需要直接连接到特定的分片。要执行这些操作，您必须直接连接到特定分片，并作为_本地分片_管理员用户进行身份验证。 创建一个_本地分片_管理员用户，请直接连接到该分片并创建该用户。MongoDB将_本地分片_用户存储在分片本身的 `admin` 数据库中。 这些_本地分片_用户完全独立于通过 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 添加到分片集群的用户。_本地分片_用户是分片的本地用户，并通过命令 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 无法访问。 与分片直接连接应该仅用于分片特定的维护和配置。通常，客户端应该通过 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 连接到分片集群。

## Localhost Exception

The localhost exception allows you to enable access control and then create the first user in the system. With the localhost exception, after you enable access control, connect to the localhost interface and create the first user in the `admin` database. The first user must have privileges to create other users, such as a user with the [`userAdmin`](https://docs.mongodb.com/v4.2/reference/built-in-roles/#userAdmin) or [`userAdminAnyDatabase`](https://docs.mongodb.com/v4.2/reference/built-in-roles/#userAdminAnyDatabase) role. Connections using the localhost exception _only_ have access to create the first user on the `admin` database.

_Changed in version 3.4:_ MongoDB 3.4 extended the localhost exception to permit execution of the [`db.createRole()`](https://docs.mongodb.com/v4.2/reference/method/db.createRole/#db.createRole) method. This method allows users authorizing via LDAP to create a role inside of MongoDB that maps to a role defined in LDAP. See [LDAP Authorization](https://docs.mongodb.com/v4.2/core/security-ldap-external/#security-ldap-external) for more information.

The localhost exception applies only when there are no users created in the MongoDB instance.

In the case of a sharded cluster, the localhost exception applies to each shard individually as well as to the cluster as a whole. Once you create a sharded cluster and add a user administrator through the [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instance, you must still prevent unauthorized access to the individual shards. Follow one of the following steps for each shard in your cluster:

* Create an administrative user, or
* Disable the localhost exception at startup. To disable the localhost exception, set the [`enableLocalhostAuthBypass`](https://docs.mongodb.com/v4.2/reference/parameters/#param.enableLocalhostAuthBypass) parameter to `0`.

## 本地主机异常

本地主机异常允许您启用访问控制，然后在系统中创建第一个用户。对于本地主机异常，在启用访问控制后，连接到本地主机接口并在 `admin` 数据库中创建第一个用户。第一个用户必须拥有创建其他用户的特权，例如具有 [`userAdmin`](https://docs.mongodb.com/v4.2/reference/en-in-roles/#userAdminAnyDatabase) 或 [`userAdminAnyDatabase`](https://docs.mongodb.com/v4.2/reference/built-in-roles/#userAdminAnyDatabase) 角色的用户。使用本地异常连接可以在 `admin` 数据库上创建第一个用户。 _在3.4版本中:_ MongoDB 3.4扩展了本地主机异常，允许执行 [`db.createRole()`](https://docs.mongodb.com/v4.2/reference/method/db.createRole/#db.createRole) 方法。该方法允许通过LDAP授权的用户在MongoDB中创建一个映射到LDAP中自定义的角色的角色。有关更多信息，请参见 [LDAP授权](https://docs.mongodb.com/v4.2/core/secur-ldap-external/#secur-ldap-external) 。 本地主机异常仅在MongoDB实例中没有创建用户时的场景。 在分片集群的情况下，本地主机异常分别应用于每个分片，也应用于整个集群。一旦您创建了一个分片集群并通过 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 实例添加了一个管理员用户，您仍然不能访问单个分片。要访问集群中单个分片，对集群中的每个分片执行操作步骤如下:

* 创建一个管理用户，或
* 在启动时禁用localhost异常。要禁用本地异常，请将 [`enableLocalhostAuthBypass`](https://docs.mongodb.com/v4.2/reference/parameters/#param.enableLocalhostAuthBypass) 参数设置为`0`。

英文原文地址：[https://docs.mongodb.com/v4.2/core/security-users/](https://docs.mongodb.com/v4.2/core/security-users/)

译者：管祥青

