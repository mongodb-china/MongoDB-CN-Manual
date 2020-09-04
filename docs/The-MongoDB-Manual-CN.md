# MongoDB 4.2用户手册

MONGODB 4.2发布于2019年8月13日

有关MongoDB 4.2中的新功能，请参阅MongoDB 4.2 [发行说明](https://docs.mongodb.com/v4.2/release-notes/4.2/)。

欢迎使用MongoDB 4.2手册！MongoDB是一个文档数据库，旨在简化开发和扩展。该手册介绍了MongoDB中的关键概念，介绍了查询语言，并提供了操作和管理方面的考虑因素和过程以及全面的参考部分。该手册也以[HTML tar.gz](https://docs.mongodb.com/v4.2/manual.tar.gz)和[EPUB的形式提供](https://docs.mongodb.com/v4.2/MongoDB-manual.epub)。

MongoDB提供数据库的*社区*版和*企业*版：

- MongoDB社区版是MongoDB的[开源和免费](https://github.com/mongodb/mongo/)版本。
- MongoDB企业版作为MongoDB高级企业版订阅的一部分提供，并包括对MongoDB部署的全面支持。MongoDB企业版还添加了以企业为中心的功能，例如LDAP和Kerberos支持，磁盘上的加密以及审计。

MongoDB还提供 [Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server)（云中托管的MongoDB企业版服务选项），无需安装开销，并提供免费的入门套餐。

该手册记录了MongoDB社区版和企业版的特性和功能。



## 入门

MongoDB 在以下版本中提供了“ [入门指南”](https://docs.mongodb.com/getting-started/shell)。

| [mongo Shell版](https://docs.mongodb.com/v4.2/tutorial/getting-started/)<br/>[Node.JS版](http://mongodb.github.io/node-mongodb-native/3.4/quick-start/quick-start/) | [Python版](https://docs.mongodb.com/drivers/pymongo)<br/>[C ++版](https://mongodb.github.io/mongo-cxx-driver/mongocxx-v3/tutorial/) | [Java版](https://mongodb.github.io/mongo-java-driver/)<br/>[C＃版](http://mongodb.github.io/mongo-csharp-driver/) | [Ruby版](https://docs.mongodb.com/ruby-driver/current/quick-start/) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |                                                              |                                                              |



完成《入门指南》后，您可能会发现以下有用的主题。

| 介绍                                                         | 开发者                                                       | 管理员                                                       | 参考                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [MongoDB简介](https://docs.mongodb.com/v4.2/introduction/)<br/>[安装指南](https://docs.mongodb.com/v4.2/installation/)<br/>[数据库和集合](https://docs.mongodb.com/v4.2/core/databases-and-collections/)<br/>[文档资料](https://docs.mongodb.com/v4.2/core/document/) | [CRUD操作](https://docs.mongodb.com/v4.2/crud/)<br/>[聚合](https://docs.mongodb.com/v4.2/aggregation/)<br/>[SQL到MongoDB](https://docs.mongodb.com/v4.2/reference/sql-comparison/)<br/>[索引](https://docs.mongodb.com/v4.2/indexes/) | [生产须知](https://docs.mongodb.com/v4.2/administration/production-notes/)<br/>[副本集](https://docs.mongodb.com/v4.2/replication/)<br/>[分片集群](https://docs.mongodb.com/v4.2/sharding/)<br/>[MongoDB安全](https://docs.mongodb.com/v4.2/security/) | [shell方法](https://docs.mongodb.com/v4.2/reference/method/)<br/>[查询运算符](https://docs.mongodb.com/v4.2/reference/operator/)<br/>[参考](https://docs.mongodb.com/v4.2/reference/)[词汇表](https://docs.mongodb.com/v4.2/reference/glossary/) |





## 支持

### MongoDB社区

如有疑问，讨论或常规技术支持，请访问 [MongoDB社区论坛](https://community.mongodb.com/)。MongoDB社区论坛是与其他MongoDB用户联系，提出问题并获得答案的集中场所。

> 译者注：MongoDB中文社区提供MongoDB中文用户原创博客/文档翻译/技术问答/技术大会/线上活动等板块平台交流服务，访问MongoDB中文社区网站请点击：https://mongoing.com/
> 进入技术交流社群请联系小芒果，微信ID：mongoingcom


### MongoDB Atlas或Cloud 

如有技术支持问题，请登录您的[MongoDB Cloud帐户](https://cloud.mongodb.com/user)并打开工单。



### MongoDB Enterprise或Ops Manager

如有技术支持问题，请通过[MongoDB支持门户](https://support.mongodb.com/)提交工单 。



## 问题

有关如何为MongoDB服务或相关项目之一提交JIRA工单的说明，请参阅 https://github.com/mongodb/mongo/wiki/Submit-Bug-Reports。



## 社区

参与MongoDB社区是与其他才华横溢，志趣相投的工程师建立关系，提高对正在从事的有趣工作的认识并提高技能的一种好方法。要了解MongoDB社区，请参阅 [参与MongoDB](http://www.mongodb.org/get-involved?tck=docs_server)。

> 译者注：MongoDB中文社区提供MongoDB中文用户原创博客/文档翻译/技术问答/技术大会/线上活动等板块平台交流服务，访问MongoDB中文社区网站请点击：https://mongoing.com/
> 进入技术交流社群请联系小芒果，微信ID：mongoingcom


## 学习

除了文档外，还有许多学习使用MongoDB的方法。您可以：

- 在[MongoDB大学](https://university.mongodb.com/?tck=docs_server)注册免费的在线课程
- 浏览[MongoDB演示文稿](https://www.mongodb.com/presentations?tck=docs_server)的存档
- 加入本地的[MongoDB用户组（MUG）](https://www.mongodb.org/user-groups?tck=docs_server)
- 参加即将举行的MongoDB [活动](http://www.mongodb.com/events?tck=docs_server)或 [网络研讨会](http://www.mongodb.com/webinars?tck=docs_server)
- 阅读[MongoDB博客](http://www.mongodb.com/blog?tck=docs_server)
- 下载[架构指南](https://www.mongodb.com/lp/whitepaper/architecture-guide?tck=docs_server)



## 许可

该手册已根据[知识共享署名-非商业性-相同方式共享3.0美国许可证进行了许可](http://creativecommons.org/licenses/by-nc-sa/3.0/us/)

有关MongoDB许可的信息，请参阅[MongoDB许可](https://www.mongodb.org/about/licensing/)。



## 其他资源

- [MongoDB，Inc.](https://www.mongodb.com/?tck=docs_server)

  MongoDB背后的公司。

- [MongoDB Atlas](https://www.mongodb.com/cloud?tck=docs_server)

  数据库即服务。

- [MongoDB Cloud Manager](https://www.mongodb.com/cloud/cloud-manager/?tck=docs_server)

  适用于MongoDB的基于云的托管运营管理解决方案。

- [MongoDB Ops Manager](https://docs.opsmanager.mongodb.com/current/?tck=docs_server)

  MongoDB的企业运营管理解决方案：包括自动化，备份和监控。

- [MongoDB生态系统](https://docs.mongodb.com/ecosystem/?tck=docs_server)

  可用于MongoDB的驱动程序，框架，工具和服务的文档。



原文链接：https://docs.mongodb.com/v4.2/

译者：小芒果
