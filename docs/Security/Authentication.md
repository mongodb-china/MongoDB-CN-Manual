# 身份验证

在本页面

- [身份验证方法](https://docs.mongodb.com/manual/core/authentication/#authentication-methods)
- [身份验证机制](https://docs.mongodb.com/manual/core/authentication/#authentication-mechanisms)
- [内部身份验证](https://docs.mongodb.com/manual/core/authentication/#internal-authentication)
- [分片集群中的身份验证](https://docs.mongodb.com/manual/core/authentication/#authentication-on-sharded-clusters)


身份验证是验证客户端身份的过程。当访问控制（即授权）开启的时候，MongoDB要求所有客户端进行身份认证，以确定他们的访问权限。

尽管身份认证和授权紧密相连，但是身份认证和授权是不同的。身份认证是验证用户的身份，授权决定已通过验证的用户对资源和操作的访问权限。


## 身份验证的方法

作为一个用户要进行身份验证，你必须提供一个用户名、密码和关联这个用户的认证数据库。

使用mongo shell 进行身份验证，可以：

- 当连接mongod或者mongos实例时，使用mongo命令行认证选项（--username、--password和--authenticationDatabase），也可以
- 先连接mongod或者mongos实例，然后在认证数据库上运行authenticate命令或者db.auth()方法。


> 重要：
>
> 当使用不同的用户进行多次身份验证时，不会删除已经通过身份认证的用户的凭证。这可能导致这个进行过多个用户身份认证的连接具有比用户预期更多的权限，并导致在一个逻辑会话中的操作引发错误。

关于使用MongoDB驱动程序进行身份验证的示例，请参阅驱动程序文档。


## 身份验证机制


MongoDB支持许多身份认证机制，客户端可以使用这些身份认证机制来验证自己的身份。MongoDB允许集成这些机制到已经存在的身份认证系统。

MongoDB支持多种身份验证机制：

- [SCRAM](https://docs.mongodb.com/manual/core/security-scram/#authentication-scram) (*默认的*)
- [x.509证书身份验证](https://docs.mongodb.com/manual/core/security-x.509/#security-auth-x509).


## 内部身份验证

除了验证客户端的身份之外，MongoDB能要求副本集和分片集群的成员对其各自的副本集或者分片集群的成员资格进行身份验证。更多的信息请参阅：内部/成员身份认证。

## 分片集群的身份验证

在分片集群中，客户端通常直接向mongos实例进行身份认证。然而，一些维护操作可能要求对特定的分片进行认证。更多关于身份认证和分片集群的信息，请参阅[分片集群用户](https://docs.mongodb.com/manual/core/security-users/#sharding-security)。



原文链接：https://docs.mongodb.com/manual/core/authentication/

译者：傅立
