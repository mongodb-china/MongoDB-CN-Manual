# system.roles 集合

在本页

- [`system.roles` 集合的Schema](https://docs.mongodb.com/manual/reference/system-roles-collection/#system-roles-schema)
- [例子](https://docs.mongodb.com/manual/reference/system-roles-collection/#examples)

admin数据库中的`system.roles`集合存储用户定义的角色。为了创建和管理这些用户自定义角色，MongoDB提供了[角色管理命令](https://docs.mongodb.com/manual/reference/command/#role-management-commands)。 


## system.roles 集合的Schema


`system.roles`集合中的文档具有以下的schema：

复制

```
{
  _id: <system-defined id>,
  role: "<role name>",
  db: "<database>",
  privileges:
      [
          {
              resource: { <resource> },
              actions: [ "<action>", ... ]
          },
          ...
      ],
  roles:
      [
          { role: "<role name>", db: "<database>" },
          ...
      ]
}
```


一个`system.roles`文档具有以下字段：

- `admin.system.roles.``role`


  该[`role`](https://docs.mongodb.com/manual/reference/system-roles-collection/#admin.system.roles.role)字段是一个字符串，用于指定角色的名称。

- `admin.system.roles.``db`


  该[`db`](https://docs.mongodb.com/manual/reference/system-roles-collection/#admin.system.roles.db)字段是一个字符串，用于指定角色所属的数据库。MongoDB通过名称（即[`role`](https://docs.mongodb.com/manual/reference/system-roles-collection/#admin.system.roles.role)）及其数据库的配对来唯一标识每个角色 。

- `admin.system.roles.``privileges`


  该[`privileges`](https://docs.mongodb.com/manual/reference/system-roles-collection/#admin.system.roles.privileges)数组包含权限文件，这些文件定义了角色的[权限](https://docs.mongodb.com/manual/core/authorization/#privileges)。


  权限文档具有以下语法：

  复制

  ```json
  {
    resource: { <resource> },
    actions: [ "<action>", ... ]
  }
  ```


  每个权限文档具有以下字段：

  admin.system.roles.privileges[n].`resource`


  一个文档，该文档指定权限[操作](https://docs.mongodb.com/manual/reference/system-roles-collection/#admin.system.roles.privileges[n].actions)所应用的资源。


  该文档具有以下格式之一：

复制

  ```json
  { db: <database>, collection: <collection> }
  ```

  或者

  ```json
  { cluster : true }
  ```


  有关更多详细信息，请阅读[资源文档](https://docs.mongodb.com/manual/reference/resource-document/#resource-document)。

  `admin.system.roles.privileges[n].actions`


  资源上允许的一系列操作， 有关操作列表，请参阅[权限操作](https://docs.mongodb.com/manual/reference/privilege-actions/#security-user-actions)

- `admin.system.roles.roles`


  该[`roles`](https://docs.mongodb.com/manual/reference/system-roles-collection/#admin.system.roles.roles)数组包含角色文档，这些角色文档指定了该角色从中[继承](https://docs.mongodb.com/manual/core/authorization/#inheritance)权限的角色。


  角色文档具有以下语法：

 复制

  ```json
  { role: "<role name>", db: "<database>" }
  ```

  角色文档具有以下字段：

  admin.system.roles.roles[n].`role`


  角色名称。角色可以是 MongoDB 提供的[内置](https://docs.mongodb.com/manual/reference/built-in-roles/#built-in-roles)角色，也可以是[用户定义的角色](https://docs.mongodb.com/manual/core/security-user-defined-roles/#user-defined-roles)。

  `admin.system.roles.roles[n].`db`


  定义角色的数据库的名称。


## 案例


考虑以下在admin 数据库的 system.roles 中发现的示例文档


### 用户自定义的角色指定权限


以下是为 myApp 数据库定义的自定义用户 appUser 的示例文档

复制

```
{
  _id: "myApp.appUser",
  role: "appUser",
  db: "myApp",
  privileges: [
       { resource: { db: "myApp" , collection: "" },
         actions: [ "find", "createCollection", "dbStats", "collStats" ] },
       { resource: { db: "myApp", collection: "logs" },
         actions: [ "insert" ] },
       { resource: { db: "myApp", collection: "data" },
         actions: [ "insert", "update", "remove", "compact" ] },
       { resource: { db: "myApp", collection: "system.js" },
         actions: [ "find" ] },
  ],
  roles: []
}
```

privileges数组列出了appUser角色指定的五个权限


- 第一个权限允许对 myApp 数据库中除 system 集合以外所有集合执行("find"`, `"createCollection"`, `"dbStats"`, `"collStats"`) 操作， 详见 [将数据库指定为操作资源](https://docs.mongodb.com/manual/reference/resource-document/#resource-specific-db).

- 后面的两个权限允许对 myApp 数据库中指定的集合 logs 和 data 上执行额外的操作，详见 [指定数据库中的集合作为操作资源](https://docs.mongodb.com/manual/reference/resource-document/#resource-specific-db-collection).

- 最后一个权限允许在 myApp 数据库的 [system 集合](https://docs.mongodb.com/manual/reference/system-collections/) 上操作。虽然第一个权限为查找操作授予了数据库范围，但是不能在 myApp 数据库的 system 集合上操作。为了授予访问 system 集合的权限，权限必须显示指定需要操作的集合。详见[操作资源文档](https://docs.mongodb.com/manual/reference/resource-document/).


空的roles数组指定 appUser 没有从其他角色继承权限。


### 用户自定义的角色继承其他角色权限


以下示例文档为 myApp 数据库定义了用户自定义角色 appAdmin ：文档显示 appAdmin 角色指定了权限，也从其他角色继承了权限。

复制

```
{
  _id: "myApp.appAdmin",
  role: "appAdmin",
  db: "myApp",
  privileges: [
      {
         resource: { db: "myApp", collection: "" },
         actions: [ "insert", "dbStats", "collStats", "compact" ]
      }
  ],
  roles: [
      { role: "appUser", db: "myApp" }
  ]
}
```


privileges 数组列举了 appAdmin 角色指定的权限，这个角色有一个权限，允许在除 system 集合外的所有集合上执行 ( `"insert"`, `"dbStats"`, `"collStats"`, `"compact"`)操作。详见[执行数据库作为操作资源](https://docs.mongodb.com/manual/reference/resource-document/#resource-specific-db).

roles数组列出了由角色名称和数据库标识的角色，角色 appAdmin 从中继承权限。


原文链接：[https://docs.mongodb.com/manual/reference/system-roles-collection/](https://docs.mongodb.com/manual/reference/system-roles-collection/)

译者：谢伟成
