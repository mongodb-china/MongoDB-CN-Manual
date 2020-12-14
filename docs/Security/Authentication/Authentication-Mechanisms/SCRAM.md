# SCRAM

On this page

- [Features](https://docs.mongodb.com/v4.2/core/security-scram/#features)
- [Driver Support](https://docs.mongodb.com/v4.2/core/security-scram/#driver-support)
- [Additional Information](https://docs.mongodb.com/v4.2/core/security-scram/#additional-information)

此页面
- [特征](https://docs.mongodb.com/v4.2/core/security-scram/#features)
- [驱动支持](https://docs.mongodb.com/v4.2/core/security-scram/#driver-support)
- [其它信息](https://docs.mongodb.com/v4.2/core/security-scram/#additional-information)


NOTE

Starting in version 4.0, MongoDB removes support for the deprecated MongoDB Challenge-Response (`MONGODB-CR`) authentication mechanism.

If your deployment has user credentials stored in `MONGODB-CR` schema, you must upgrade to SCRAM **before** you upgrade to version 4.0. For information on upgrading to `SCRAM`, see [Upgrade to SCRAM](https://docs.mongodb.com/v4.2/release-notes/3.0-scram/).

Salted Challenge Response Authentication Mechanism (SCRAM) is the default authentication mechanism for MongoDB. SCRAM is based on the IETF [RFC 5802](https://tools.ietf.org/html/rfc5802) standard that defines best practices for implementation of challenge-response mechanisms for authenticating users with passwords.

Using SCRAM, MongoDB verifies the supplied user credentials against the user’s [`name`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.user), [`password`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.credentials) and [`authentication database`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.db). The authentication database is the database where the user was created, and together with the user’s name, serves to identify the user.

> 注意
> 从4.0版本开始，MongoDB删除了对已弃用的MongoDB质询-响应(`MONGODB-CR`)身份验证机制的支持。
如果您的部署有存储在`MONGODB-CR`模式中的用户凭证，您必须在升级到SCRAM*之前*升级到MongoDB4.0版本。有关升级到`SCRAM`的信息，请参阅[升级到SCRAM](https://docs.mongodb.com/v4.2/release-notes/3.0-scram/) 。
严肃的询问响应身份验证机制(SCRAM)是MongoDB的默认身份验证机制。SCRAM基于IETF [RFC 5802](https://tools.ietf.org/html/rfc5802) 标准，该标准定义了实现询问-响应机制的最佳实践，用于对用户进行密码验证。
使用SCRAM，MongoDB验证所提供的用户凭证 [`name`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.user) , [`password`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.credentials) 和 [`authentication database`](https://docs.mongodb.com/v4.2/reference/system-users-collection/#admin.system.users.db) 。身份验证数据库是创建用户的数据库，它与用户名一起用于标识用户。


## Features

MongoDB’s implementation of SCRAM provides:

- A tunable work factor (i.e. the iteration count),
- Per-user random salts, and
- Authentication of the server to the client as well as the client to the server.

## 特征
MongoDB的SCRAM实现提供:
- 可调的工作因素(如：迭代计数)，
- 每个用户随机salts，和
- 服务器对客户端的认证，以及客户对服务器的认证。

### SCRAM Mechanisms

MongoDB supports the following SCRAM mechanisms:

| SCRAM Mechanism | Description                                                  |
| :-------------- | :----------------------------------------------------------- |
| `SCRAM-SHA-1`   | Uses the SHA-1 hashing function.To modify the iteration count for `SCRAM-SHA-1`, see [`scramIterationCount`](https://docs.mongodb.com/v4.2/reference/parameters/#param.scramIterationCount). |
| `SCRAM-SHA-256` | Uses the SHA-256 hashing function and requires featureCompatibilityVersion (`fcv`) set to `4.0`.To modify the iteration count for `SCRAM-SHA-256`, see [`scramSHA256IterationCount`](https://docs.mongodb.com/v4.2/reference/parameters/#param.scramSHA256IterationCount).*New in version 4.0.* |

When creating or updating a SCRAM user, you can indicate the specific SCRAM mechanism as well as indicate whether the server or the client digests the password. When using `SCRAM-SHA-256`, MongoDB requires server-side password hashing, i.e. the server digests the password. For details, see [`db.createUser()`](https://docs.mongodb.com/v4.2/reference/method/db.createUser/#db.createUser) and [`db.updateUser()`](https://docs.mongodb.com/v4.2/reference/method/db.updateUser/#db.updateUser).

### SCRAM机制
MongoDB支持如下SCRAM机制：
| SCRAM机制 | 描述                                                  |
| :-------------- | :----------------------------------------------------------- |
| `SCRAM-SHA-1`   | 使用SHA-1哈希函数。要修改`SCRAM-SHA-1`的迭代计数，请参见[`scramIterationCount`](https://docs.mongodb.com/v4.2/reference/parameters/#param.scramIterationCount) 。|
| `SCRAM-SHA-256` | 使用SHA-256哈希函数，并要求特性兼容版本(`fcv`) 设置为 `4.0`。修改`SCRAM-SHA-256`的迭代计数，参见 [`scramSHA256IterationCount`](https://docs.mongodb.com/v4.2/reference/parameters/#param.scramSHA256IterationCount) .*新版本4.0.*|

在创建或更新SCRAM用户时，指示特定的SCRAM机制，以及指示是服务器还是客户端摘要密码。当使用`SCRAM-SHA-256`时，MongoDB需要服务器端密码散列，即服务器摘要密码。详细信息，请参见 [`db.createUser()`](https://docs.mongodb.com/v4.2/reference/method/db.createUser/#db.createUser) 和 [`db.updateUser()`](https://docs.mongodb.com/v4.2/reference/method/db.updateUser/#db.updateUser) 。

## Driver Support

To use SCRAM, you must upgrade your driver if your current driver version does not support `SCRAM`.

The minimum driver versions that support `SCRAM` are:

| Driver Language                                            | Version                                                      | Driver Language                                             | Version                                             |
| :--------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------------- | :-------------------------------------------------- |
| [C](https://docs.mongodb.com/ecosystem/drivers/c)          | [1.1.0](https://github.com/mongodb/mongo-c-driver/releases)  | [Perl](https://docs.mongodb.com/ecosystem/drivers/perl)     | [1.0.0](https://metacpan.org/release/MongoDB)       |
| [C++](https://github.com/mongodb/mongo-cxx-driver)         | [1.0.0](https://github.com/mongodb/mongo-cxx-driver/releases) | [PHP](https://docs.mongodb.com/ecosystem/drivers/php)       | [1.0](https://pecl.php.net/package/mongodb)         |
| [C#](https://docs.mongodb.com/ecosystem/drivers/csharp)    | [1.10](https://github.com/mongodb/mongo-csharp-driver/releases) | [Python](https://docs.mongodb.com/ecosystem/drivers/python) | [2.8](https://pypi.python.org/pypi/pymongo/)        |
| [Java](https://docs.mongodb.com/ecosystem/drivers/java)    | [2.13](https://github.com/mongodb/mongo-java-driver/releases) | [Motor](https://docs.mongodb.com/ecosystem/drivers/python)  | [0.4](https://pypi.python.org/pypi/motor/)          |
| [Node.js](https://docs.mongodb.com/ecosystem/drivers/node) | [1.4.29](https://github.com/mongodb/node-mongodb-native/releases) | [Ruby](https://docs.mongodb.com/ecosystem/drivers/ruby)     | [1.12](https://rubygems.org/gems/mongo)             |
|                                                            |                                                              | [Scala](https://docs.mongodb.com/ecosystem/drivers/scala)   | [2.8.0](https://github.com/mongodb/casbah/releases) |

## 驱动支持
如果您当前的驱动程序版本不支持`SCRAM`，您必须升级驱动程序才能使用SCRAM。
支持`SCRAM`的最小驱动程序版本如下所示:

| Driver Language                                            | Version                                                      | Driver Language                                             | Version                                             |
| :--------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------------- | :-------------------------------------------------- |
| [C](https://docs.mongodb.com/ecosystem/drivers/c)          | [1.1.0](https://github.com/mongodb/mongo-c-driver/releases)  | [Perl](https://docs.mongodb.com/ecosystem/drivers/perl)     | [1.0.0](https://metacpan.org/release/MongoDB)       |
| [C++](https://github.com/mongodb/mongo-cxx-driver)         | [1.0.0](https://github.com/mongodb/mongo-cxx-driver/releases) | [PHP](https://docs.mongodb.com/ecosystem/drivers/php)       | [1.0](https://pecl.php.net/package/mongodb)         |
| [C#](https://docs.mongodb.com/ecosystem/drivers/csharp)    | [1.10](https://github.com/mongodb/mongo-csharp-driver/releases) | [Python](https://docs.mongodb.com/ecosystem/drivers/python) | [2.8](https://pypi.python.org/pypi/pymongo/)        |
| [Java](https://docs.mongodb.com/ecosystem/drivers/java)    | [2.13](https://github.com/mongodb/mongo-java-driver/releases) | [Motor](https://docs.mongodb.com/ecosystem/drivers/python)  | [0.4](https://pypi.python.org/pypi/motor/)          |
| [Node.js](https://docs.mongodb.com/ecosystem/drivers/node) | [1.4.29](https://github.com/mongodb/node-mongodb-native/releases) | [Ruby](https://docs.mongodb.com/ecosystem/drivers/ruby)     | [1.12](https://rubygems.org/gems/mongo)             |
|                                                            |                                                              | [Scala](https://docs.mongodb.com/ecosystem/drivers/scala)   | [2.8.0](https://github.com/mongodb/casbah/releases) |


## Additional Information

- [Blog Post: Improved Password-Based Authentication in MongoDB 3.0: SCRAM Explained (Part 1)](https://www.mongodb.com/blog/post/improved-password-based-authentication-mongodb-30-scram-explained-part-1?tck=docs_server)
- [Blog Post: Improved Password-Based Authentication in MongoDB 3.0: SCRAM Explained (Part 2)](https://www.mongodb.com/blog/post/improved-password-based-authentication-mongodb-30-scram-explained-part-2?tck=docs_server)

## 其它信息
- [博客文章:MongoDB 3.0改进的基于密码的身份验证:SCRAM解释(Part 1)](https://www.mongodb.com/blog/post/improved-password-based-authentication-mongodb-30-scram-explained-part-1?tck=docs_server)
- [博客文章:MongoDB 3.0改进的基于密码的身份验证:SCRAM解释(Part 2)](https://www.mongodb.com/blog/post/improved-password-based-authentication-mongodb-30-scram-explained-part-2?tck=docs_server)



译者：管祥青