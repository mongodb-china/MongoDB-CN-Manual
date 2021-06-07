# 权限认证机制

On this page

* [Default Authentication Mechanism](https://docs.mongodb.com/v4.2/core/authentication-mechanisms/#default-authentication-mechanism)
* [Specify Authentication Mechanism](https://docs.mongodb.com/v4.2/core/authentication-mechanisms/#specify-authentication-mechanism)

此页面

* [默认权限认证机制](https://docs.mongodb.com/v4.2/core/authentication-mechanisms/#default-authentication-mechanism)
* [指定权限认证机制](https://docs.mongodb.com/v4.2/core/authentication-mechanisms/#specify-authentication-mechanism)

NOTE

Starting in version 4.0, MongoDB removes support for the deprecated MongoDB Challenge-Response \(`MONGODB-CR`\) authentication mechanism.

MongoDB supports the following authentication mechanisms:

* [SCRAM](https://docs.mongodb.com/v4.2/core/security-scram/) \(_默认_\)
* [x.509 Certificate Authentication](https://docs.mongodb.com/v4.2/core/security-x.509/).

In addition, MongoDB Enterprise provides integration with a number of external authentication mechanisms, including Kerberos and LDAP. See [Enterprise Authentication Mechanisms](https://docs.mongodb.com/v4.2/core/authentication-mechanisms-enterprise/) for the additional authentication mechanisms supported by MongoDB Enterprise.

> 注意 从4.0版本开始，MongoDB删除了对已弃用的MongoDB询问-响应\(`MONGODB-CR`\)身份验证机制的支持。 MongoDB支持以下身份验证机制:
>
> * [SCRAM](https://docs.mongodb.com/v4.2/core/security-scram/) \(_默认_\)
> * [x.509证书认证](https://docs.mongodb.com/v4.2/core/security-x.509/).

## Default Authentication Mechanism

As of MongoDB 3.0, [Salted Challenge Response Authentication Mechanism \(SCRAM\)](https://docs.mongodb.com/v4.2/core/security-scram/#authentication-scram) is the default authentication mechanism for MongoDB.

## 默认权限认证机制

在MongoDB 3.0中，[严格询问响应认证机制 \(SCRAM\)](https://docs.mongodb.com/v4.2/core/secur-scram/#authentic-SCRAM) 是MongoDB的默认身份验证机制。

## Specify Authentication Mechanism

To specify the authentication mechanism to use, set the [`authenticationMechanisms`](https://docs.mongodb.com/v4.2/reference/parameters/#param.authenticationMechanisms) parameter for [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos).

Clients specify the authentication mechanism in the [`db.auth()`](https://docs.mongodb.com/v4.2/reference/method/db.auth/#db.auth) method. For the [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell and the MongoDB tools, you can also specify the authentication mechanism from the command line.

## 指定权限认证机制

指定要使用的身份验证机制,设置 [`authenticationMechanisms`](https://docs.mongodb.com/v4.2/reference/parameters/#param.authenticationMechanisms) 参数，使用 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) 和 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) 。 客户端在 [`db.auth()`](https://docs.mongodb.com/v4.2/reference/method/db.auth/#db.auth) 方法中指定身份验证机制。对于 [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) 命令和MongoDB工具，还可以从命令行指定身份验证机制。

英文原文地址：[https://docs.mongodb.com/v4.2/core/authentication-mechanisms/](https://docs.mongodb.com/v4.2/core/authentication-mechanisms/)

译者：管祥青

