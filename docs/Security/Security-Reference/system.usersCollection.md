# system.users 集合

在本页

- [system.users 集合的Schema](https://docs.mongodb.com/manual/reference/system-users-collection/#system-users-schema)
- [例子](https://docs.mongodb.com/manual/reference/system-users-collection/#example)

system.users 集合在 admin 数据库中，保存了用户[身份验证](https://docs.mongodb.com/manual/core/authentication/#authentication)和[授权](https://docs.mongodb.com/manual/core/authorization/#authorization)的信息。为了管理这个集合的数据，MongoDB 提供了用户管理指令。


## system.users 集合的Schema

system.users 集合中的文档具有以下的 schema：

复制

```
{
  _id: <system defined id>,
  userId : <system assigned UUID>,  // Starting in MongoDB 4.0.9
  user: "<name>",
  db: "<database>",
  credentials: { <authentication credentials> },
  roles: [
           { role: "<role name>", db: "<database>" },
           ...
         ],
  customData: <custom information>,
  authenticationRestrictions : [ <documents> ] // Starting in MongoDB 4.0
 }
```


每个 system.users 文档都有以下字段： 

- `admin.system.users.``userId`


  创建时分配给用户的唯一标识符。[userId](https://docs.mongodb.com/manual/reference/system-users-collection/#admin.system.users.userId) 适用于在MongoDB 4.0.9 及更高的版本[创建](https://docs.mongodb.com/manual/reference/method/db.createUser/#db.createUser)的用户

- `admin.system.users.``user`


  用户名。用户位于单个逻辑数据库的上下文中(请参考资料[`admin.system.users.db)`](https://docs.mongodb.com/manual/reference/system-users-collection/#admin.system.users.db)，但可以通过[`roles`](https://docs.mongodb.com/manual/reference/system-users-collection/#admin.system.users.roles)组中指定的角色访问其他数据库。 

- `admin.system.users.``db`


  与用户关联的[身份验证数据库](https://docs.mongodb.com/manual/core/security-users/#authentication-database)。用户的权限不一定限于此数据库。用户可以通过该[`roles`](https://docs.mongodb.com/manual/reference/system-users-collection/#admin.system.users.roles)组在其他数据库中拥有特权。

- `admin.system.users.``credentials`


  用户的身份验证信息。对于具有外部存储的身份验证凭据的用户，例如使用 [Kerberos](https://docs.mongodb.com/manual/tutorial/control-access-to-mongodb-with-kerberos-authentication/) 或x.509证书进行身份验证的`system.users` 用户，该用户的文档不包含该 [`credentials`](https://docs.mongodb.com/manual/reference/system-users-collection/#admin.system.users.credentials)字段。对于 [SCRAM](https://docs.mongodb.com/manual/core/security-scram/#authentication-scram)用户凭据，该信息包括机制，迭代计数和身份验证参数。

  也可以看看

  - [`scramSHA256IterationCount`](https://docs.mongodb.com/manual/reference/parameters/#param.scramSHA256IterationCount)
  - [`scramIterationCount`](https://docs.mongodb.com/manual/reference/parameters/#param.scramIterationCount)

- `admin.system.users.``roles`


 授予用户的一系列角色。该组包含 [内置角色](https://docs.mongodb.com/manual/reference/built-in-roles/#built-in-roles)和[用户定义角色](https://docs.mongodb.com/manual/core/security-user-defined-roles/#user-defined-roles)。


  角色文档具有以下语法：

  复制

  ```js
  { role: "<role name>", db: "<database>" }
  ```

  角色文档有以下字段

  `admin.system.users.roles[n].``role


  ​           角色名称。角色可以是 MongoDB 提供的[内置](https://docs.mongodb.com/manual/reference/built-in-roles/#built-in-roles)角色，也可以是[用户自定义角色](https://docs.mongodb.com/manual/core/security-user-defined-roles/#user-defined-roles)。

  `admin.system.users.roles[n].``db`


  ​          定义角色的数据库的名称。


  使用[角色管理](https://docs.mongodb.com/manual/reference/command/#role-management-commands)或[用户管理](https://docs.mongodb.com/manual/reference/command/#user-management-commands)命令指定`"readWrite"`角色时，如果运行命令的数据库中存在该角色，则可以单独指定角色名称（例如“ readWrite”）。

- `admin.system.users.``customData`


  有关用户的可选自定义信息。

- `admin.system.users.``authenticationRestrictions`

  
  服务器为用户强制执行的一系列身份验证限制。该数组包含 IP 地址和 CIDR 范围的列表，允许用户从中连接到服务器或服务器可以从中接受用户。
  
  版本4.0中的新功能。
  

## Example


考虑`system.users`集合中的以下文档：

复制

```
{
   "_id" : "home.Kari",
   "userId" : UUID("ec1eced7-055a-4ca8-8737-60dd02c52793"),  // Available starting in MongoDB 4.0.9
   "user" : "Kari",
   "db" : "home",
   "credentials" : {
      "SCRAM-SHA-1" : {
         "iterationCount" : 10000,
         "salt" : "S/xM2yXFosynbCu4GzFDgQ==",
         "storedKey" : "Ist4cgpEd1vTbnRnQLdobgmOsBA=",
         "serverKey" : "e/0DyzS6GPboAA2YNBkGYm87+cg="
      },
      "SCRAM-SHA-256" : {
         "iterationCount" : 15000,
         "salt" : "p1G+fZadAeYAbECN8F/6TMzXGYWBaZ3DtWM0ig==",
         "storedKey" : "LEgLOqZQmkGhd0owm/+6V7VdJUYJcXBhPUvi9z+GBfk=",
         "serverKey" : "JKfnkVv9iXwxyc8JaapKVwLPy6SfnmB8gMb1Pr15T+s="
      }
   },
   "authenticationRestrictions" : [                           // Available starting in MongoDB 4.0
      { "clientSource" : [ "69.89.31.226" ], "serverAddress" : [ "172.16.254.1" ] }
   ],
   "customData" : {
      "zipCode" : "64157"
   },
   "roles" : [
      {
         "role" : "read",
         "db" : "home"
      },
      {
         "role" : "readWrite",
         "db" : "test"
      }
   ]
}
```


该文档显示用户`Kari`的身份验证数据库是 home`数据库。在数据库中`Kari 具有 [read](https://docs.mongodb.com/manual/reference/built-in-roles/#read) 角色，在 test 数据库中具有[`readWrite`](https://docs.mongodb.com/manual/reference/built-in-roles/#readWrite)角色 。


原文链接：[https://docs.mongodb.com/manual/reference/system-users-collection/](https://docs.mongodb.com/manual/reference/system-users-collection/)

译者：谢伟成
