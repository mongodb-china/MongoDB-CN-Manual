# 添加用户

On this page

* [Overview](https://docs.mongodb.com/v4.2/tutorial/create-users/#overview)
* [Prerequisites](https://docs.mongodb.com/v4.2/tutorial/create-users/#prerequisites)
* Examples
  * [Username/Password Authentication](https://docs.mongodb.com/v4.2/tutorial/create-users/#username-password-authentication)
  * [Kerberos Authentication](https://docs.mongodb.com/v4.2/tutorial/create-users/#kerberos-authentication)
  * [LDAP Authentication](https://docs.mongodb.com/v4.2/tutorial/create-users/#ldap-authentication)
  * [x.509 Client Certificate Authentication](https://docs.mongodb.com/v4.2/tutorial/create-users/#x-509-client-certificate-authentication)

在此页

* [概述](https://docs.mongodb.com/v4.2/tutorial/create-users/#overview)
* [要求](https://docs.mongodb.com/v4.2/tutorial/create-users/#prerequisites)
* 例子
  * [用户名/密码身份认证](https://docs.mongodb.com/v4.2/tutorial/create-users/#username-password-authentication)
  * [Kerberos认证](https://docs.mongodb.com/v4.2/tutorial/create-users/#kerberos-authentication)
  * [LDAP认证](https://docs.mongodb.com/v4.2/tutorial/create-users/#ldap-authentication)
  * [客户端x.509证书认证](https://docs.mongodb.com/v4.2/tutorial/create-users/#x-509-client-certificate-authentication)

## Overview

MongoDB employs role-based access control \(RBAC\) to determine access for users. A user is granted one or more [roles](https://docs.mongodb.com/v4.2/core/authorization/#roles) that determine the user’s access or privileges to MongoDB [resources](https://docs.mongodb.com/v4.2/reference/resource-document/#resource-document) and the [actions](https://docs.mongodb.com/v4.2/reference/privilege-actions/#security-user-actions) that user can perform. A user should have only the minimal set of privileges required to ensure a system of [least privilege](https://docs.mongodb.com/v4.2/reference/glossary/#term-least-privilege).

Each application and user of a MongoDB system should map to a distinct user. This _access isolation_ facilitates access revocation and ongoing user maintenance.

## 概述

MongoDB采用基于角色的访问控制\(RBAC\)来确定用户的访问权限。一个用户可以授予一个或多个角色 [role](https://docs.mongodb.com/v4.2/core/authorization/#roles) ,来决定访问或授权MongoDB 的[resources](https://docs.mongodb.com/v4.2/reference/resource-document/resource-document) 和 [actions](https://docs.mongodb.com/v4.2/reference/privilege-actions/security-user-actions) 可执行的用户。用户应该具有最小需要的特权集合来确保系统具有 [最小权限](https://docs.mongodb.com/v4.2/reference/glossary/#term-les-privilege) 。

MongoDB系统的每个应用程序及用户应该映射到一个不同的用户。这种_访问隔离_简化了访问撤销和正在进行的用户维护。

## Prerequisites

If you have enabled access control for your deployment, you can use the [localhost exception](https://docs.mongodb.com/v4.2/core/security-users/#localhost-exception) to create the first user in the system. This first user must have privileges to create other users. As of MongoDB 3.0, with the localhost exception, you can only create users on the `admin` database. Once you create the first user, you must authenticate as that user to add subsequent users. [Enable Access Control](https://docs.mongodb.com/v4.2/tutorial/enable-authentication/) provides more detail about adding users when enabling access control for a deployment.

For routine user creation, you must possess the following permissions:

* To create a new user in a database, you must have the [`createUser`](https://docs.mongodb.com/v4.2/reference/privilege-actions/#createUser) [action](https://docs.mongodb.com/v4.2/reference/privilege-actions/#security-user-actions) on that [database resource](https://docs.mongodb.com/v4.2/reference/resource-document/#resource-specific-db).
* To grant roles to a user, you must have the [`grantRole`](https://docs.mongodb.com/v4.2/reference/privilege-actions/#grantRole) [action](https://docs.mongodb.com/v4.2/reference/privilege-actions/#security-user-actions) on the role’s database.

The [`userAdmin`](https://docs.mongodb.com/v4.2/reference/built-in-roles/#userAdmin) and [`userAdminAnyDatabase`](https://docs.mongodb.com/v4.2/reference/built-in-roles/#userAdminAnyDatabase) built-in roles provide [`createUser`](https://docs.mongodb.com/v4.2/reference/privilege-actions/#createUser) and [`grantRole`](https://docs.mongodb.com/v4.2/reference/privilege-actions/#grantRole) actions on their respective [resources](https://docs.mongodb.com/v4.2/reference/resource-document/).

## 要求

如果您已经部署并启用了访问控制，那么您可以使用 [本地主机异常](https://docs.mongodb.com/v4.2/core/secur-users/#localhostst-exception) 来创建系统中的第一个用户。第一个用户必须具有创建其他用户的权限。在MongoDB 3.0中，本地主机异常，只能在`admin` 数据库上创建用户。创建第一个用户后，必须验证该用户之后才能添加后续用户。[启用访问控制](https://docs.mongodb.com/v4.2/tutorial/enable-authentication/) 提供了关于部署应用启用访问控制进行添加用户的更多细节。

对于常规用户创建，必须使用以下步骤进行权限操作:

* 在数据库中创建一个新用户,您必须拥有 [`createUser`](https://docs.mongodb.com/v4.2/reference/privilege-actions/#createUser) 和 [action](https://docs.mongodb.com/v4.2/reference/privilege-actions/security-user-actions) 权限在 [数据库资源](https://docs.mongodb.com/v4.2/reference/resource-document/#resource-specific-db) 上。
* 给用户授予角色，您必须使用 [`grantRole`](https://docs.mongodb.com/v4.2/reference/privilege-actions/#grantRole) [action](https://docs.mongodb.com/v4.2/reference/privilege-actions/#secur-user-actions) 来创建数据库角色。

  [`userAdmin`](https://docs.mongodb.com/v4.2/reference/built-in-roles/userAdmin) 和 [`userAdminAnyDatabase`](https://docs.mongodb.com/v4.2/reference/built-in-roles/#userAdminAnyDatabase) 内置角色提供 [`createUser`](https://docs.mongodb.com/v4.2/reference/privilege-actions/createUser) 和 [`grantRole`](https://docs.mongodb.com/v4.2/reference/privilege-actions/#grantRole) 行为在各自的 [资源](https://docs.mongodb.com/v4.2/reference/resource-document/) 上。

## Examples

To create a user in a MongoDB deployment, you connect to the deployment, and then use the [`db.createUser()`](https://docs.mongodb.com/v4.2/reference/method/db.createUser/#db.createUser) method or [`createUser`](https://docs.mongodb.com/v4.2/reference/command/createUser/#dbcmd.createUser) command to add the user.

## 例子

在MongoDB中创建一个用户，先连接到MongoDB，然后使用 [`db.createUser()`](https://docs.mongodb.com/v4.2/reference/method/db.createUser/#db.createUser) 方法或 [`createUser`](https://docs.mongodb.com/v4.2/reference/command/createUser/#dbcmd.createUser) 命令来添加用户。

### Username/Password Authentication

The following operation creates a user in the `reporting` database with the specified name, password, and roles.

TIP

Starting in version 4.2 of the [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell, you can use the [`passwordPrompt()`](https://docs.mongodb.com/v4.2/reference/method/passwordPrompt/#passwordPrompt) method in conjunction with various user authentication/management methods/commands to prompt for the password instead of specifying the password directly in the method/command call. However, you can still specify the password directly as you would with earlier versions of the [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell.

```text
use reporting
db.createUser(
  {
    user: "reportsUser",
    pwd: passwordPrompt(),  // or cleartext password
    roles: [
       { role: "read", db: "reporting" },
       { role: "read", db: "products" },
       { role: "read", db: "sales" },
       { role: "readWrite", db: "accounts" }
    ]
  }
)
```

[Enable Access Control](https://docs.mongodb.com/v4.2/tutorial/enable-authentication/) provides more details about enforcing authentication for your MongoDB deployment.

### 用户名和密码认证

以下操作在 `reporting`数据库中创建具有指定用户名、密码和角色的用户。

> 提示 从4.2版开始 [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) 命令,可以使用 [`passwordPrompt()`](https://docs.mongodb.com/v4.2/reference/method/passwordPrompt/passwordPrompt) 方法结合不同用户认证/管理，方法/命令提示输入密码,而不是直接指定密码/命令调用的方法。但是，在以前的版本中，您仍然需要通过 [`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) 直接指定密码。

```text
use reporting
db.createUser(
  {
    user: "reportsUser",
    pwd: passwordPrompt(),  // or cleartext password
    roles: [
       { role: "read", db: "reporting" },
       { role: "read", db: "products" },
       { role: "read", db: "sales" },
       { role: "readWrite", db: "accounts" }
    ]
  }
)
```

[启用访问控制](https://docs.mongodb.com/v4.2/tutorial/enable-authentication/) 提供了更多关于加强MongoDB部署身份验证的细节。

### Kerberos Authentication

Users that will authenticate to MongoDB using an external authentication mechanism, such as Kerberos, must be created in the `$external` database, which allows [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) to consult an external source for authentication.

_Changed in version 3.6.3:_ To use sessions with `$external` authentication users \(i.e. Kerberos, LDAP, x.509 users\), the usernames cannot be greater than 10k bytes.

For Kerberos authentication, you must add the Kerberos principal as the username. You do not need to specify a password.

The following operation adds the Kerberos principal `reportingapp@EXAMPLE.NET` with read-only access to the `records` database.

```text
use $external
db.createUser(
    {
      user: "reportingapp@EXAMPLE.NET",
      roles: [
         { role: "read", db: "records" }
      ]
    }
)
```

[Configure MongoDB with Kerberos Authentication on Linux](https://docs.mongodb.com/v4.2/tutorial/control-access-to-mongodb-with-kerberos-authentication/) and [Configure MongoDB with Kerberos Authentication on Windows](https://docs.mongodb.com/v4.2/tutorial/control-access-to-mongodb-windows-with-kerberos-authentication/) provide more details about setting up Kerberos authentication for your MongoDB deployment.

### Kerberos认证

用户使用外部身份验证机制来认证MongoDB,例如Kerberos，先必须在`$external`数据库，它允许 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/bin.mongos) 或 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) 来咨询外部源进行身份认证。 _在版本3.6.3中:_使用 `$external` 会话机制来认证用户\(如Kerberos、LDAP、x.509用户\)，而且用户名不能大于10k字节。 对于Kerberos身份验证，必须添加Kerberos主体作为用户名，不需要指定密码。 下面的操作添加Kerberos主体`reportingapp@EXAMPLE.NET`在`records`数据库中添加只读访问权限。

```text
use $external
db.createUser(
    {
      user: "reportingapp@EXAMPLE.NET",
      roles: [
         { role: "read", db: "records" }
      ]
    }
)
```

[在Linux上配置MongoDB数据库Kerberos身份验证](https://docs.mongodb.com/v4.2/tutorial/control-access-to-mongodb-with-kerberos-authentication/) 和 [在Windows上配置MongoDB数据库Kerberos身份验证](https://docs.mongodb.com/v4.2/tutorial/control-access-to-mongodb-windows-with-kerberos-authentication/) 提供更多细节关于MongoDB部署Kerberos身份验证相关的配置。

### LDAP Authentication

Users that will authenticate to MongoDB using an external authentication mechanism, such as LDAP, must be created in the `$external` database, which allows [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) to consult an external source for authentication.

_Changed in version 3.6.3:_ To use sessions with `$external` authentication users \(i.e. Kerberos, LDAP, x.509 users\), the usernames cannot be greater than 10k bytes.

For LDAP authentication, you must specify a username. You do not need to specify the password, as that is handled by the LDAP service.

The following operation adds the `reporting` user with read-only access to the `records` database.

```text
use $external
db.createUser(
    {
      user: "reporting",
      roles: [
         { role: "read", db: "records" }
      ]
    }
)
```

[Authenticate Using SASL and LDAP with ActiveDirectory](https://docs.mongodb.com/v4.2/tutorial/configure-ldap-sasl-activedirectory/) and [Authenticate Using SASL and LDAP with OpenLDAP](https://docs.mongodb.com/v4.2/tutorial/configure-ldap-sasl-openldap/) provide more detail about using authenticating using LDAP.

### LDAP认证

用户可以使用外部身份验证机制验证MongoDB，如LDAP，先必须创建`$external`数据库，它允许 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/bin.mongos) 或 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) 来咨询外部源进行身份验证。 _在版本3.6.3中:_使用 `$external` 会话来进行身份验证\(如Kerberos、LDAP、x.509用户\)，其中用户名不能大于10k字节。 对于LDAP身份验证，必须指定用户名，不需要指定密码，因为它由LDAP服务进行处理。 下面的操作添加`reporting` 用户，具有访问`records` 数据库的只读权限。

```text
use $external
db.createUser(
    {
      user: "reporting",
      roles: [
         { role: "read", db: "records" }
      ]
    }
)
```

[使用SASL和LDAP域来进行权限认证](https://docs.mongodb.com/v4.2/tutorial/configure-ldap-sasl-activedirectory/) 和 [使用SASL和OpenLDAP进行权限认证](https://docs.mongodb.com/v4.2/tutorial/configure-ldap-sasl-openldap/) 提供更多细节关于使用LDAP身份验证。

### x.509 Client Certificate Authentication

Users that will authenticate to MongoDB using an external authentication mechanism, such as x.509 Client Certificate Authentication, must be created in the `$external` database, which allows [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) or [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) to consult an external source for authentication.

_Changed in version 3.6.3:_ To use sessions with `$external` authentication users \(i.e. Kerberos, LDAP, x.509 users\), the usernames cannot be greater than 10k bytes.

For x.509 Client Certificate authentication, you must add the value of the `subject` from the client certificate as a MongoDB user. Each unique x.509 client certificate corresponds to a single MongoDB user. You do not need to specify a password.

The following operation adds the client certificate subject `CN=myName,OU=myOrgUnit,O=myOrg,L=myLocality,ST=myState,C=myCountry` user with read-only access to the `records` database.

```text
use $external
db.createUser(
    {
      user: "CN=myName,OU=myOrgUnit,O=myOrg,L=myLocality,ST=myState,C=myCountry",
      roles: [
         { role: "read", db: "records" }
      ]
    }
)
```

[Use x.509 Certificates to Authenticate Clients](https://docs.mongodb.com/v4.2/tutorial/configure-x509-client-authentication/) provides details about setting up x.509 Client Certificate authentication for your MongoDB deployment.

### 客户端x.509证书认证

用户可以使用外部身份验证机制认证MongoDB数据库，如客户端x.509证书认证，先必须创建`$external`数据库，它允许 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/bin.mongos) 或 [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) 来咨询外部源进行身份验证。 _在版本3.6.3中:_使用 `$external` 会话来进行身份验证\(如Kerberos、LDAP、x.509用户\)，其中用户名不能大于10k字节。 对于客户端x.509证书身份验证，您必须从客户端证书中作为一个MongoDB用户添加`subject`值。每个唯一的x.509客户端证书对应一个MongoDB用户，您不需要指定密码。 下面的操作，添加客户机证书`subject`值为`CN=myName,OU=myOrgUnit,O=myOrg,L=myLocality,ST=myState,C=myCountry`的用户，该用户对`records`数据库具有只读访问权限。

```text
use $external
db.createUser(
    {
      user: "CN=myName,OU=myOrgUnit,O=myOrg,L=myLocality,ST=myState,C=myCountry",
      roles: [
         { role: "read", db: "records" }
      ]
    }
)
```

[使用客户端x.509证书认证](https://docs.mongodb.com/v4.2/tutorial/configure-x509-client-authentication/) 提供了关于为MongoDB部署设置x.509客户端证书认证的详细信息。

英文原文：[https://docs.mongodb.com/v4.2/tutorial/create-users/](https://docs.mongodb.com/v4.2/tutorial/create-users/)

译者：管祥青

