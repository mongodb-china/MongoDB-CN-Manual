## 安全

MongoD提供了各种各样的功能让你安全地部署MongoDB，诸如：身份认证、访问控制、加密。一些关键的安全功能包括：


| Authentication                                               | Authorization                                                | TLS/SSL                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [身份认证](https://docs.mongodb.com/manual/core/authentication/)  [SCRAM](https://docs.mongodb.com/manual/core/security-scram/)  [x.509](https://docs.mongodb.com/manual/core/security-x.509/) | [基于角色的访问控制](https://docs.mongodb.com/manual/core/authorization/)  [启动访问控制](https://docs.mongodb.com/manual/tutorial/enable-authentication/)  [用户与角色管理](https://docs.mongodb.com/manual/tutorial/manage-users-and-roles/) | [TLS/SSL (传输加密)](https://docs.mongodb.com/manual/core/security-transport-encryption/)  [使用TLS/SSL配置mongod和mongos](https://docs.mongodb.com/manual/tutorial/configure-ssl/) [为客户端配置TLS/SSL ](https://docs.mongodb.com/manual/tutorial/configure-ssl-clients/) |

| Enterprise Only                                              | Encryption                                                   |      |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :--- |
| [Kerberos 验证](https://docs.mongodb.com/manual/core/kerberos/)  [LDAP 代理验证](https://docs.mongodb.com/manual/core/security-ldap/)  [静态加密](https://docs.mongodb.com/manual/core/security-encryption-at-rest/)  [审计](https://docs.mongodb.com/manual/core/auditing/) | [客户端字段级加密](https://docs.mongodb.com/manual/core/security-client-side-encryption/) |      |


译者：傅立
