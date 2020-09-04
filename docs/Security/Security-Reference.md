# 安全参考

在本页

- [mongo shell中的安全相关方法]((https://docs.mongodb.com/manual/reference/security/#security-methods-in-the-mongo-shell))
- [安全参考文档](https://docs.mongodb.com/manual/reference/security/#security-reference-documentation)

下面列举了mongo shell 中可用的与安全相关的方法，以及其他[安全相关材料](https://docs.mongodb.com/manual/reference/security/#security-reference-materials)。


## mongo shell中的安全相关方法

### 用户管理和认证方法

| Name                                                         | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`db.auth()`](https://docs.mongodb.com/manual/reference/method/db.auth/#db.auth) | 向数据库验证用户         |
| [`db.changeUserPassword()`](https://docs.mongodb.com/manual/reference/method/db.changeUserPassword/#db.changeUserPassword) | 改变用户的密码           |
| [`db.createUser()`](https://docs.mongodb.com/manual/reference/method/db.createUser/#db.createUser) | 创建一个新用户                           |
| [`db.dropUser()`](https://docs.mongodb.com/manual/reference/method/db.dropUser/#db.dropUser) | 删除一个用户                           |
| [`db.dropAllUsers()`](https://docs.mongodb.com/manual/reference/method/db.dropAllUsers/#db.dropAllUsers) | 删除与数据库相关的用户 |
| [`db.getUser()`](https://docs.mongodb.com/manual/reference/method/db.getUser/#db.getUser) | 返回指定用户信息 |
| [`db.getUsers()`](https://docs.mongodb.com/manual/reference/method/db.getUsers/#db.getUsers) | 返回所有与数据库相关的用户信息 |
| [`db.grantRolesToUser()`](https://docs.mongodb.com/manual/reference/method/db.grantRolesToUser/#db.grantRolesToUser) | 授予用户角色和角色包含的权限 |
| [`db.removeUser()`](https://docs.mongodb.com/manual/reference/method/db.removeUser/#db.removeUser) | 弃用，从数据库删除用户 |
| [`db.revokeRolesFromUser()`](https://docs.mongodb.com/manual/reference/method/db.revokeRolesFromUser/#db.revokeRolesFromUser) | 删除用户的角色                    |
| [`db.updateUser()`](https://docs.mongodb.com/manual/reference/method/db.updateUser/#db.updateUser) | 更新用户数据                               |
| [`passwordPrompt()`](https://docs.mongodb.com/manual/reference/method/passwordPrompt/#passwordPrompt) | 提示输入密码，作为在各种mongo shell用户管理方法中直接指定密码的替代方法 |


### 角色管理方法

| Name                                                         | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`db.createRole()`](https://docs.mongodb.com/manual/reference/method/db.createRole/#db.createRole) | 创建一个角色和指定其权限 |
| [`db.dropRole()`](https://docs.mongodb.com/manual/reference/method/db.dropRole/#db.dropRole) | 删除一个用户自定义角色           |
| [`db.dropAllRoles()`](https://docs.mongodb.com/manual/reference/method/db.dropAllRoles/#db.dropAllRoles) | 删除与数据库关联的所有用户自定义的角色 |
| [`db.getRole()`](https://docs.mongodb.com/manual/reference/method/db.getRole/#db.getRole) | 返回指定角色的信息 |
| [`db.getRoles()`](https://docs.mongodb.com/manual/reference/method/db.getRoles/#db.getRoles) | 返回数据库中所有用户自定义角色的信息 |
| [`db.grantPrivilegesToRole()`](https://docs.mongodb.com/manual/reference/method/db.grantPrivilegesToRole/#db.grantPrivilegesToRole) | 给指定用户分配权限 |
| [`db.revokePrivilegesFromRole()`](https://docs.mongodb.com/manual/reference/method/db.revokePrivilegesFromRole/#db.revokePrivilegesFromRole) | 从用户自定义角色中删除指定权限 |
| [`db.grantRolesToRole()`](https://docs.mongodb.com/manual/reference/method/db.grantRolesToRole/#db.grantRolesToRole) | 指定用户定义的角色从哪些角色继承特权。 |
| [`db.revokeRolesFromRole()`](https://docs.mongodb.com/manual/reference/method/db.revokeRolesFromRole/#db.revokeRolesFromRole) | 从角色中删除继承的角色   |
| [`db.updateRole()`](https://docs.mongodb.com/manual/reference/method/db.updateRole/#db.updateRole) | 更新用户自定义的角色。           |



### 安全相关文档

- [system.roles Collection](https://docs.mongodb.com/manual/reference/system-roles-collection/)

  描述存储用户自定义角色的集合的内容。

- [system.users Collection](https://docs.mongodb.com/manual/reference/system-users-collection/)

  描述存储用户凭据和角色分配的集合的内容。

- [Resource Document](https://docs.mongodb.com/manual/reference/resource-document/)

  描述角色的资源文档。

- [Privilege Actions](https://docs.mongodb.com/manual/reference/privilege-actions/)

  可用于权限的操作列表。

原文链接：[https://docs.mongodb.com/manual/reference/security/](https://docs.mongodb.com/manual/reference/security/)

译者：谢伟成
