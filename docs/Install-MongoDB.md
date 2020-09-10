# 安装 MongoDB

在本页面



- [MongoDB社区版安装教程](https://docs.mongodb.com/v4.2/installation/#mongodb-community-edition-installation-tutorials)
- [MongoDB企业版安装教程](https://docs.mongodb.com/v4.2/installation/#mongodb-enterprise-edition-installation-tutorials)
- [将社区版升级到企业版教程](https://docs.mongodb.com/v4.2/installation/#upgrade-community-edition-to-enterprise-edition-tutorials)
- [支持平台](https://docs.mongodb.com/v4.2/installation/#supported-platforms)



MongoDB有两个服务器版本：*社区版*和 *企业版*。



MONGODB ATLAS

[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是MongoDB公司提供的MongoDB云服务，无需安装开销，并提供免费的入门套餐。



手册的这部分包含有关安装MongoDB的信息。

- 有关将当前部署升级到MongoDB 4.2的说明，请参阅[升级过程](https://docs.mongodb.com/v4.2/release-notes/4.2/#upgrade)。
- 有关升级到当前版本的最新修补程序版本的说明，请参阅[升级到MongoDB的最新版本](https://docs.mongodb.com/v4.2/tutorial/upgrade-revision/)。



## MongoDB社区版安装教程

MongoDB社区版安装教程包括：

| 平台    | 对应教程                                                     |
| :------ | :----------------------------------------------------------- |
| Linux   | [在Red Hat或CentOS上安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-red-hat/)<br/>[在Ubuntu上安装MongoDB Community Edition](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-ubuntu/)<br/>[在Debian上安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-debian/)<br/>[在SUSE上安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-suse/)<br/>[在Amazon Linux上安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-amazon/) |
| macOS   | [在macOS上安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/) |
| Windows | [在Windows上安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/) |



## MongoDB企业版安装教程

MongoDB企业版安装教程包括：

| 平台    | 对应教程                                                     |
| :------ | :----------------------------------------------------------- |
| Linux   | [在Red Hat或CentOS上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-red-hat/)<br/>[在Ubuntu上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-ubuntu/)<br/>[在Debian上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-debian/)<br/>[在SUSE上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-suse/)<br/>[在Amazon Linux上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-amazon/) |
| macOS   | [在macOS上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-os-x/) |
| Windows | [在Windows上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-windows/) |
| Docker  | [使用Docker安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-with-docker/) |



## 将社区版升级到企业版教程

重要

不要使用这些说明升级到另一个发行版本。要升级发行版本，请参阅相应的发行升级说明，例如[Upgrade to MongoDB 4.2](https://docs.mongodb.com/v4.2/release-notes/4.2/#upgrade)。

- [升级到MongoDB企业版（单节点）](https://docs.mongodb.com/v4.2/tutorial/upgrade-to-enterprise-standalone/)
- [升级到MongoDB企业版（副本集）](https://docs.mongodb.com/v4.2/tutorial/upgrade-to-enterprise-replica-set/)
- [升级到MongoDB企业版（分片集群）](https://docs.mongodb.com/v4.2/tutorial/upgrade-to-enterprise-sharded-cluster/)



## 支持的平台

*在版本3.4中进行了更改：* MongoDB不再支持32位x86平台。

### x86_64

平台支持停产通知

| Ubuntu 14.04 | 支持已在MongoDB 4.2+中删除。 |
| ------------ | ---------------------------- |
| Debian 8     | 支持已在MongoDB 4.2+中删除。 |
| macOS 10.11  | 支持已在MongoDB 4.2+中删除。 |

*即将停产的通知*：

| Windows 8.1 / 2012R2 | MongoDB将在将来的版本中终止支持。 |
| -------------------- | --------------------------------- |
| Windows 8/2012       | MongoDB将在后续版本中终止支持。   |
| Windows 7 / 2008R2   | MongoDB将在后续版本中终止支持。   |



| 平台                                                         | 4.2社区版与企业版 | 4.0社区版与企业版 | 3.6社区版与企业版 | 3.4社区版与企业版 |
| :----------------------------------------------------------- | :---------------: | :---------------: | :---------------: | :---------------: |
| Amazon Linux 2                                               |         ✓         |         ✓         |                   |                   |
| Amazon Linux 2013.03及更高版本                               |         ✓         |         ✓         |         ✓         |         ✓         |
| Debian 10                                                    |      4.2.1+       |                   |                   |                   |
| Debian 9                                                     |         ✓         |         ✓         |      3.6.5+       |                   |
| Debian 8                                                     |                   |         ✓         |         ✓         |         ✓         |
| RHEL / CentOS / Oracle Linux [[1\]](https://docs.mongodb.com/v4.2/installation/#oracle-linux) 8.0及更高版本 |      4.2.1+       |      4.0.14+      |      3.6.17+      |                   |
| RHEL / CentOS / Oracle Linux [[1\]](https://docs.mongodb.com/v4.2/installation/#oracle-linux) 7.0及更高版本 |         ✓         |         ✓         |         ✓         |         ✓         |
| RHEL / CentOS / Oracle Linux [[1\]](https://docs.mongodb.com/v4.2/installation/#oracle-linux) 6.2及更高版本 |         ✓         |         ✓         |         ✓         |         ✓         |
| SLES 15                                                      |      4.2.1+       |                   |                   |                   |
| SLES 12                                                      |         ✓         |         ✓         |         ✓         |         ✓         |
| Solaris 11 64位                                              |                   |                   |                   |     仅社区版      |
| Ubuntu 18.04                                                 |         ✓         |      4.0.1+       |                   |                   |
| Ubuntu 16.04                                                 |         ✓         |         ✓         |         ✓         |         ✓         |
| Ubuntu 14.04                                                 |                   |         ✓         |         ✓         |         ✓         |
| Windows Server 2019                                          |         ✓         |                   |                   |                   |
| Windows 10 /Server 2016                                      |         ✓         |         ✓         |         ✓         |         ✓         |
| Windows 8.1 / Server 2012 R2                                 |         ✓         |         ✓         |         ✓         |         ✓         |
| Windows 8 /Server 012                                        |         ✓         |         ✓         |         ✓         |         ✓         |
| Windows 7 / Server 2008 R2                                   |         ✓         |         ✓         |         ✓         |         ✓         |
| Windows Vista                                                |                   |                   |                   |         ✓         |
| macOS 10.13及更高版本                                        |         ✓         |         ✓         |                   |                   |
| macOS 10.12                                                  |         ✓         |         ✓         |         ✓         |         ✓         |
| macOS 10.11                                                  |                   |         ✓         |         ✓         |         ✓         |
| macOS 10.10                                                  |                   |                   |         ✓         |         ✓         |

| [1]  | *（[1](https://docs.mongodb.com/v4.2/installation/#id1)，[2](https://docs.mongodb.com/v4.2/installation/#id2)，[3](https://docs.mongodb.com/v4.2/installation/#id3)）*的MongoDB仅支持运行Red Hat Compatible Kernel (RHCK)的Oracle的Linux。MongoDB不支持Unbreakable Enterprise Kernel (UEK)。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |





### ARM64 

平台支持停产通知

| Ubuntu 16.04 ARM64 | 支持已在MongoDB Community 4.2+中删除。 |
| ------------------ | -------------------------------------- |
|                    |                                        |

| 平台         | 4.2社区版与企业版 | 4.0社区版与企业版 | 3.6社区版与企业版 | 3.4社区版与企业版 |
| :----------- | :---------------: | :---------------: | :---------------: | :---------------: |
| Ubuntu 18.04 |     仅社区版      |                   |                   |                   |
| Ubuntu 16.04 |     仅企业版      |         ✓         |         ✓         |         ✓         |



### PPC64LE（MongoDB企业版）

平台支持停产通知

| Ubuntu 16.04 PPC64LE | 支持已在MongoDB 4.2+中删除。 |
| -------------------- | ---------------------------- |
|                      |                              |

| 平台            | 4.2企业 | 4.0企业 |     3.6企业      |     3.4企业      |
| :-------------- | :-----: | :-----: | :--------------: | :--------------: |
| RHEL / CentOS 7 |    ✓    |    ✓    |        ✓         |        ✓         |
| Ubuntu 18.04    |    ✓    |         |                  |                  |
| Ubuntu 16.04    |         |    ✓    | 从3.6.13开始删除 | 从3.4.21开始删除 |



### s390x 

| 平台            | 4.2社区版与企业版 | 4.0企业版 |    3.6企业版     |    3.4企业版     |
| :-------------- | :---------------: | :-------: | :--------------: | :--------------: |
| RHEL / CentOS 7 |         ✓         |  4.0.6+   | 从3.6.17开始删除 | 从3.4.14开始删除 |
| RHEL / CentOS 6 |         ✓         |     ✓     | 从3.6.14开始删除 | 从3.4.22开始删除 |
| SLES12          |         ✓         |  4.0.6+   | 从3.6.17开始删除 | 从3.4.15开始删除 |
| Ubuntu 18.04    |      4.2.1+       |  4.0.6+   |                  |                  |

←  [MongoDB扩展JSON（v1）](https://docs.mongodb.com/v4.2/reference/mongodb-extended-json-v1/)<br/>[安装MongoDB社区版](https://docs.mongodb.com/v4.2/administration/install-community/) →

原文链接：https://docs.mongodb.com/v4.2/installation/



译者：桂陈


### MongoDB中文社区

![MongoDB&#x4E2D;&#x6587;&#x793E;&#x533A;&#x2014;MongoDB&#x7231;&#x597D;&#x8005;&#x6280;&#x672F;&#x4EA4;&#x6D41;&#x5E73;&#x53F0;](https://mongoing.com/wp-content/uploads/2020/09/6de8a4680ef684d-2.png)

| 资源列表推荐 | 资源入口 |
| :--- | :--- |
| MongoDB中文社区官网 | [https://mongoing.com/](https://mongoing.com/) |
| 微信服务号 ——最新资讯和优质文章 | Mongoing中文社区（mongoing-mongoing） |
| 微信订阅号 ——发布文档翻译内容 | MongoDB中文用户组（mongoing123） |
| 官方微信号 —— 官方最新资讯 | MongoDB数据库（MongoDB-China） |
| MongoDB中文社区组委会成员介绍 | [https://mongoing.com/core-team-members](https://mongoing.com/core-team-members) |
| MongoDB中文社区翻译小组介绍 | [https://mongoing.com/translators](https://mongoing.com/translators) |
| MongoDB中文社区微信技术交流群 | 添加社区助理小芒果微信（ID:mongoingcom），并备注 mongo |
| MongoDB中文社区会议及文档资源 | [https://mongoing.com/resources](https://mongoing.com/resources) |
| MongoDB中文社区大咖博客 | [基础知识](https://mongoing.com/basic-knowledge)  [性能优化](https://mongoing.com/performance-optimization)  [原理解读](https://mongoing.com/interpretation-of-principles)  [运维监控](https://mongoing.com/operation-and-maintenance-monitoring)  [最佳实践](https://mongoing.com/best-practices) |
| MongoDB白皮书 | [https://mongoing.com/mongodb-download-white-paper](https://mongoing.com/mongodb-download-white-paper) |
| MongoDB初学者教程-7天入门 | [https://mongoing.com/mongodb-beginner-tutorial](https://mongoing.com/mongodb-beginner-tutorial) |
| 社区活动邮件订阅 | [https://sourl.cn/spszjN](https://sourl.cn/spszjN) |


