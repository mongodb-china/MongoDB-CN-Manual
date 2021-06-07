# 副本集维护教程

## Replica Set Maintenance Tutorials

## 副本集维护教程

The following tutorials provide information in maintaining existing replica sets.

下述教程为您提供关于维护现有副本集的相关信息。

* [Change the Size of the Oplog](https://docs.mongodb.com/v4.2/tutorial/change-oplog-size/)
* [更改Oplog的大小](https://docs.mongodb.com/v4.2/tutorial/change-oplog-size/)

  Increase the size of the [oplog](https://docs.mongodb.com/v4.2/reference/glossary/#term-oplog) which logs operations. In most cases, the default oplog size is sufficient.

  增加记录操作的[oplog](https://docs.mongodb.com/v4.2/reference/glossary/#term-oplog)日志的大小。大多数情况下，默认oplog设置的大小已经足够满足需求。

* [Perform Maintenance on Replica Set Members](https://docs.mongodb.com/v4.2/tutorial/perform-maintence-on-replica-set-members/)
* [对副本集的成员执行维护](https://docs.mongodb.com/v4.2/tutorial/perform-maintence-on-replica-set-members/)

  Perform maintenance on a member of a replica set while minimizing downtime.

  对副本集的成员进行维护，同时尽可能地减少停机时间。

* [Force a Member to Become Primary](https://docs.mongodb.com/v4.2/tutorial/force-member-to-be-primary/)
* [强制将副本集成员转变为主节点](https://docs.mongodb.com/v4.2/tutorial/force-member-to-be-primary/)

  Force a replica set member to become primary.

  强制将副本集成员转变为主节点。

* [Resync a Member of a Replica Set](https://docs.mongodb.com/v4.2/tutorial/resync-replica-set-member/)
* [为副本集的成员重新同步数据](https://docs.mongodb.com/v4.2/tutorial/resync-replica-set-member/)

  Sync the data on a member. Either perform initial sync on a new member or resync the data on an existing member that has fallen too far behind to catch up by way of normal replication.

  在副本集的成员上执行同步数据。 在新成员上执行初始同步，或者在已落后太远而无法通过常规复制赶平的现有成员上重新同步数据。

* [Configure Replica Set Tag Sets](https://docs.mongodb.com/v4.2/tutorial/configure-replica-set-tag-sets/)
* [配置副本集标记集](https://docs.mongodb.com/v4.2/tutorial/configure-replica-set-tag-sets/)

  Assign tags to replica set members for use in targeting read and write operations to specific members.

  为给副本集成员配置标记，以用于针对特定成员定位读写操作。

* [Reconfigure a Replica Set with Unavailable Members](https://docs.mongodb.com/v4.2/tutorial/reconfigure-replica-set-with-unavailable-members/)
* [重新配置包含不可用成员的副本集](https://docs.mongodb.com/v4.2/tutorial/reconfigure-replica-set-with-unavailable-members/)

  Reconfigure a replica set when a majority of replica set members are down or unreachable.

  当大部分副本集成员宕机或无法访问时，重新配置副本集。

* [Manage Chained Replication](https://docs.mongodb.com/v4.2/tutorial/manage-chained-replication/)
* [管理链式复制](https://docs.mongodb.com/v4.2/tutorial/manage-chained-replication/)

  Disable or enable chained replication. Chained replication occurs when a secondary replicates from another secondary instead of the primary.

  禁用或启用链式复制。当从节点从另一个从节点（非主节点）复制时，即产生链式复制。

* [Change Hostnames in a Replica Set](https://docs.mongodb.com/v4.2/tutorial/change-hostnames-in-a-replica-set/)
* [更改副本集中的主机名](https://docs.mongodb.com/v4.2/tutorial/change-hostnames-in-a-replica-set/)

  Update the replica set configuration to reflect changes in members’ hostnames.

  通过更新副本集配置以反映成员主机名的更改。

* [Configure a Secondary’s Sync Target](https://docs.mongodb.com/v4.2/tutorial/configure-replica-set-secondary-sync-target/)
* [配置从节点的同步目标](https://docs.mongodb.com/v4.2/tutorial/configure-replica-set-secondary-sync-target/)

  Specify the member that a secondary member synchronizes from.

  指定从节点要从哪个成员上同步数据。

  原文链接：[https://docs.mongodb.com/v4.2/administration/replica-set-maintenance/](https://docs.mongodb.com/v4.2/administration/replica-set-maintenance/)

  译者：桂陈

