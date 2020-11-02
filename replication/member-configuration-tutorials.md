# Member Configuration Tutorials

# 副本集成员配置教程

The following tutorials provide information in configuring replica set members to support specific operations, such as to provide dedicated backups, to support reporting, or to act as a cold standby.

以下教程为您介绍如何配置副本集成员以支持特定的操作，例如提供专用备份，支持报告或充当冷备用。

WARNING

警告

Avoid reconfiguring replica sets that contain members of different MongoDB versions as validation rules may differ across MongoDB versions.

由于不同的MongoDB版本所使用的验证规则可能有所不同，重新配置副本集时，应当避免该副本集包含不同版本的成员。

- [Adjust Priority for Replica Set Member](https://docs.mongodb.com/v4.2/tutorial/adjust-replica-set-member-priority/)

- [调整副本集成员的优先级](https://docs.mongodb.com/v4.2/tutorial/adjust-replica-set-member-priority/)

  Change the precedence given to a replica set members in an election for primary.

  更改在选择主节点时赋予复制集成员的优先级。

- [Prevent Secondary from Becoming Primary](https://docs.mongodb.com/v4.2/tutorial/configure-secondary-only-replica-set-member/)

- [防止从节点变成主节点](https://docs.mongodb.com/v4.2/tutorial/configure-secondary-only-replica-set-member/)

  Make a secondary member ineligible for election as primary.

  设置指定的从节点没有资格被选举为主节点。

- [Configure a Hidden Replica Set Member](https://docs.mongodb.com/v4.2/tutorial/configure-a-hidden-replica-set-member/)

- [配置一个隐藏的副本集成员](https://docs.mongodb.com/v4.2/tutorial/configure-a-hidden-replica-set-member/)

  Configure a secondary member to be invisible to applications in order to support significantly different usage, such as a dedicated backups.

  将从节点配置为对应用程序不可见，以便支持某些特殊场景，例如专用备份。

- [Configure a Delayed Replica Set Member](https://docs.mongodb.com/v4.2/tutorial/configure-a-delayed-replica-set-member/)

- [配置一个延迟的副本集成员](https://docs.mongodb.com/v4.2/tutorial/configure-a-delayed-replica-set-member/)

  Configure a secondary member to keep a delayed copy of the data set in order to provide a rolling backup.

  设置指定的从节点用来保留数据集的延迟副本，以便提供滚动备份。

- [Configure Non-Voting Replica Set Member](https://docs.mongodb.com/v4.2/tutorial/configure-a-non-voting-replica-set-member/)

- [配置无选举权的副本集成员](https://docs.mongodb.com/v4.2/tutorial/configure-a-non-voting-replica-set-member/)

  Create a secondary member that keeps a copy of the data set but does not vote in an election.

  创建一个从节点，该节点保留数据集的副本但不参与选举。

- [Convert a Secondary to an Arbiter](https://docs.mongodb.com/v4.2/tutorial/convert-secondary-into-arbiter/)

- [将从节点转换为仲裁节点](https://docs.mongodb.com/v4.2/tutorial/convert-secondary-into-arbiter/)

  将一个从节点转换为仲裁节点。
  
  原文链接：https://docs.mongodb.com/v4.2/administration/replica-set-member-configuration/
  
  译者：桂陈
  
  
