# 安全检查列表

MongoDB还为如何保护MongoDB部署提供了一个建议的操作列表即[安全检查列表]((https://docs.mongodb.com/manual/administration/security-checklist/))

*最后更新于：2019-12-05*

这个文档提供了一个保护MongoDB应该实施的安全措施列表。这个列表并不是完整无遗的。

**生产环境前的检查列表/注意事项**


### ➤启动访问控制和强制身份认证


启动访问控制和指定身份认证的机制。你可以使用MongoDB的SCRMA或者x.509身份认证机制或者集成你已经使用的Kerberos/LDAP基础设施。身份认证要求所有的客户端和服务端在连接到系统之前提供有效的凭证。

请参阅[身份认证](https://docs.mongodb.com/manual/core/authentication/)和[开启访问控制](https://docs.mongodb.com/manual/tutorial/enable-authentication/)。


### ➤ 配置基于角色的访问控制


**首先**创建一个管理员用户，然后再创建其他的用户。为每一人/应用程序创建唯一的用户以访问系统。


遵循最小权限原则。为一组用户创建他们所需的确切访问权限的角色。然后创建用户并且仅为他们分配执行操作所需的角色。一个用户可以是个人或者一个客户端程序。

>提示：
>
>一个用户在不同数据库可以拥有不同的权限。如果一个用户要求在多个数据库的权限，使用有多个可授予适当数据库权限的角色来创建一个单一用户，而不是给不同的数据库创建多个用户。


请参阅[基于角色的访问控制](https://docs.mongodb.com/manual/core/authorization/)和[用户与角色管理](https://docs.mongodb.com/manual/tutorial/manage-users-and-roles/)。


### ➤ 加密通信（TLS/SSL）


配置MongoDB为所有传入和传出连接使用TLS/SSL。使用TLS/SSL加密MongoDB部署的[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)组件以及所有应用程序和MongoDB之间的通信。

从4.0版本开始，MongoDB使用操作系统原生的TLS/SSL库：

| 操作系统  | 使用的系统库     |
| :-------- | ---------------- |
| Linux/BSD | OpenSSL          |
| macOS     | Secure Transport |

> 注意
>
> 从4.0版本开始，在支持TLS1.1+的系统上，MongoDB会禁用TLS1.0加密。更多详细信息，请参阅 [禁用TLS1.0](https://docs.mongodb.com/manual/tutorial/configure-ssl/).

请[参阅使用TLS/SSL配置mongod和mongos](https://docs.mongodb.com/manual/tutorial/configure-ssl/)


### ➤加密和保护数据

从MongoDB 3.2企业版开始，你可以使用WiredTiger存储引擎的本地[静态加密](https://docs.mongodb.com/manual/core/security-encryption-at-rest/)来加密存储层的数据。


如果你没有使用WiredTiger的静态加密，MongoDB的数据应该在每台主机上使用文件系统、设备或物理加密（例如dm-crypt）。使用文件系统权限保护MongoDB数据。MongoDB数据包括数据文件、配置文件、审计日志以及秘钥文件。

将日志收集到一个中央日志存储区。这些日志包含了DB身份认证尝试及其源IP地址.


### ➤ 限制网络暴露


确保MongoDB运行在受信任的网络环境中并且配置防火墙或者安全组来控制MongoDB实例的入站和出站流量。

只允许受信任的客户端访问MongoDB实例所在的网络接口和端口。例如，使用白名单机制允许受信任的IP地址访问。

> 注意
>
> 从MongoDB 3.6开始，MongoDB的二进制文件：[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)和[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)会默认绑定在`localhost`上。MongoDB 2.6到3.4版本，只有官方MongoDB RPM（Red Hat、CentOS、Fedora Linux和衍生品）和DEB（Debian、Ubuntu和衍生品）包中的二进制文件默认绑定在localhost。了解更多关于这个改变的信息，请参阅[localhost绑定兼容变更](https://docs.mongodb.com/manual/release-notes/3.6-compatibility/#bind-ip-compatibility)

请参阅：

- [网络和配置加固](https://docs.mongodb.com/manual/core/security-hardening/)
- [`net.bindIp`](https://docs.mongodb.com/manual/reference/configuration-options/#net.bindIp)配置设定
- [`security.clusterIpSourceWhitelist`](https://docs.mongodb.com/manual/reference/configuration-options/#security.clusterIpSourceWhitelist)配置设定
- [authenticationRestrictions](https://docs.mongodb.com/manual/reference/method/db.createUser/#db-createuser-authenticationrestrictions)为每个用户指定IP白名单

禁用直接SSH root访问。


### ➤系统活动审计


跟踪对数据库配置和数据的访问和更改。[MongoDB企业版](http://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs)包含了一个系统审计工具，可以记录MongoDB实例上的系统事件（例如用户操作、连接事件）。这些审计记录使审查分析得以进行并且允许管理员去验证适当的控制。可以设置过滤器来记录特定的事件，例如身份认证事件。

请参阅[Auditing](https://docs.mongodb.com/manual/core/auditing/) 和[Configure Auditing](https://docs.mongodb.com/manual/tutorial/configure-auditing/)


### ➤使用专用用户运行MongoDB


使用一个专用的操作系统账户运行MongoDB进程。确保这个账户除了访问数据，没有不必要的权限。

关于运行MongoDB的更多信息，请参阅[MongoDB安装](https://docs.mongodb.com/manual/installation/)


### ➤ 使用安全的配置选项运行MongoDB


MongoDB支持使用JavaScript代码对服务器端执行特定的操作，包括：[`mapReduce`](https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce)和[`$where`](https://docs.mongodb.com/manual/reference/operator/query/where/#op._S_where)。如果你不使用这些操作，在命令行使用[`--noscripting`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-noscripting)选项来禁用服务器端脚本。

确保启用了输入验证。MongoDB默认通过[`net.wireObjectCheck`](https://docs.mongodb.com/manual/reference/configuration-options/#net.wireObjectCheck)设置启用输入验证。这确保了[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)实例存储的所有文档都是有效的[BSON](https://docs.mongodb.com/manual/reference/glossary/#term-bson)。

请参阅：[网络和配置加固](https://docs.mongodb.com/manual/core/security-hardening/)


### ➤索取安全技术实施指南（如适用）


安全技术实施指南（STIG）包含美国国防部内部部署的安全指南。MongoDB公司为需要的情况提供了它的STIG。请[索取一个副本](http://www.mongodb.com/lp/contact/stig-requests)以获取更多信息。


### ➤考虑安全标准的合规性


对于需要遵循HIPAA或者PCI-DSS的应用程序，请参看[MongoDB安全参考架构](https://www.mongodb.com/collateral/mongodb-security-architecture)以了解更多关于如何使用关键安全功能来构建合规的应用程序基础设施。


### 定期/持续的产品检查


定期检查[MongoDB产品通用漏洞披露](https://www.mongodb.com/alerts)并且更新你的产品。

查询[MongoDB的生命周期终止日期](https://www.mongodb.com/support-policy)并升级你的MongoDB。一般来说，尽量使用最新的版本。

确保你的信息安全管理的系统策略和程序在你安装的MongoDB上生效，包括执行以下操作：

- 定期对你的设备打补丁并且检查操作指南
- 检查策略及流程变更，尤其是网络规则的更改，以防无意中将MongoDB暴露在互联网。
- 检查MongoDB数据库用户并定期进行轮换。

原文链接：https://docs.mongodb.com/manual/security/

https://docs.mongodb.com/manual/administration/security-checklist/

译者：傅立
