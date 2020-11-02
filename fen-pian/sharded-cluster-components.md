# Sharded Cluster Components

# 分片集群组件

- On this page

- 内容概览

  - [Production Configuration](https://docs.mongodb.com/v4.2/core/sharded-cluster-components/#production-configuration)
  - [生产环境配置](https://docs.mongodb.com/v4.2/core/sharded-cluster-components/#production-configuration)
  - [Development Configuration](https://docs.mongodb.com/v4.2/core/sharded-cluster-components/#development-configuration)
  - [开发环境配置](https://docs.mongodb.com/v4.2/core/sharded-cluster-components/#development-configuration)
  
  A MongoDB [sharded cluster](https://docs.mongodb.com/v4.2/reference/glossary/#term-sharded-cluster) consists of the following components:
  
  一个MongoDB分片集群由以下组件组成：
  
  - [shard](https://docs.mongodb.com/v4.2/core/sharded-cluster-shards/): Each shard contains a subset of the sharded data. As of MongoDB 3.6, shards must be deployed as a [replica set](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set).
  - [shard](https://docs.mongodb.com/v4.2/core/sharded-cluster-shards/)：每个shard（分片）包含被分片的数据集中的一个子集。从MongoDB 3.6版本开始，每个分片必须部署为[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)架构。 
  - [mongos](https://docs.mongodb.com/v4.2/core/sharded-cluster-query-router/): The `mongos` acts as a query router, providing an interface between client applications and the sharded cluster.
  - [mongos](https://docs.mongodb.com/v4.2/core/sharded-cluster-query-router/)：`mongos`充当查询路由的角色，为客户端应用程序和分片集群间的通信提供一个接口。
  - [config servers](https://docs.mongodb.com/v4.2/core/sharded-cluster-config-servers/): Config servers（配置服务器）存储了分片集群的元数据和配置信息。从MongoDB 3.4版本开始，config servers必须部署为副本集架构 (CSRS)。
  
  
  
  ## Production Configuration
  
  ## 生产环境配置
  
  In a production cluster, ensure that data is redundant and that your systems are highly available. Consider the following for a production sharded cluster deployment:
  
  生产环境的分片集群须确保数据是冗余的，并且您的系统具有高可用性。关于生产环境下分片群集的部署，请考虑以下事项：
  
  - Deploy Config Servers as a 3 member [replica set](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)
  - 将配置服务器部署为拥有3个成员的[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)。
  - Deploy each Shard as a 3 member [replica set](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)
  - 将每个分片（Shard）部署为拥有3个成员的[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)。
  - Deploy one or more [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) routers
  - 部署一个或多个[`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)。
  
  ### Replica Set Distribution
  
  ### 分散部署副本集
  
  Where possible, consider deploying one member of each replica set in a site suitable for being a disaster recovery location.
  
  在可能的情况下，请考虑在适合作为灾难恢复位置的站点中，部署每个副本集的一个成员。
  
  NOTE
  
  注意
  
  Distributing replica set members across two data centers provides benefit over a single data center. In a two data center distribution,
  
  在两个数据中心分散部署副本集成员比在单个数据中心部署具备下述优势：
  
  - If one of the data centers goes down, the data is still available for reads unlike a single data center distribution.
  - 不同于单个数据中心部署，如果其中一个数据中心宕机，该数据仍然可用于读取。
  - If the data center with a minority of the members goes down, the replica set can still serve write operations as well as read operations.
  - 如果只有少数成员的数据中心宕机，副本集仍然可以提供读写服务。
  - However, if the data center with the majority of the members goes down, the replica set becomes read-only.
  - 如果具有大多数成员的数据中心宕机，副本集将只能提供读服务。
  
  If possible, distribute members across at least three data centers. For config server replica sets (CSRS), the best practice is to distribute across three (or more depending on the number of members) centers. If the cost of the third data center is prohibitive, one distribution possibility is to evenly distribute the data bearing members across the two data centers and store the remaining member in the cloud if your company policy allows.
  
  如果可能，请将成员分布到至少三个数据中心。对于配置服务器副本集(CSRS)，最佳实践将其分布在三个或更多（取决于成员的数量）的数据中心上。如果第三个数据中心的成本太高，一种可能的分布方式是将承载数据的成员均匀分布到两个数据中心，并在公司政策允许的情况下将其余成员存储在云中。
  
  ### Number of Shards
  
  ### 分片数量
  
  Sharding requires at least two shards to distribute sharded data. Single shard sharded clusters may be useful if you plan on enabling sharding in the near future, but do not need to at the time of deployment.
  
  分片集群需要至少两个分片来分发分片数据。如果您计划在不久的将来启用分片，那么单个分片的集群可能很有用，但在部署时不需要启用。
  
  ### Number of `mongos` and Distribution
  
  ### `mongos`数量和分布
  
  Deploying multiple [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) routers supports high availability and scalability. A common pattern is to place a [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) on each application server. Deploying one [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) router on each application server reduces network latency between the application and the router.
  
  部署多个[`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)实例可支持高可用性和可扩展性。一个常见的模式是在每个应用程序服务器上部署一个 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)实例。在每个应用服务器上部署一个 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)实例可以减少应用程序和mongos之间的网络延迟。
  
  Alternatively, you can place a [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) router on dedicated hosts. Large deployments benefit from this approach because it decouples the number of client application servers from the number of [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instances. This gives greater control over the number of connections the [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) instances serve.
  
  另外，您也可以在专用主机上部署 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)实例。大型部署可从此方法中受益，因为它使客户端应用程序服务器的数量与mongos节点的数量脱钩，可以更好地控制[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)实例服务的连接数。
  
  Installing [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instances on their own hosts allows these instances to use greater amounts of memory. Memory would not be shared with a [`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod) instance. It is possible to use primary shards to host [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) routers but be aware that memory contention may become an issue on large deployments.
  
  在自己的主机上安装 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)实例允许这些实例使用更多的内存。内存不会与[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)实例共享。您也可以使用主分片来托管 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)，但是要注意，在大型部署中，内存争抢可能会成为一个问题。
  
  There is no limit to the number of [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) routers you can have in a deployment. However, as [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) routers communicate frequently with your config servers, monitor config server performance closely as you increase the number of routers. If you see performance degradation, it may be beneficial to cap the number of [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) routers in your deployment.
  
  [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)的部署数量没有限制。但是，由于[`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)经常与配置服务器通信，当您增加mongos数量时，请密切关注配置服务器的性能。如果发现性能下降，最好限制您部署的[`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)数量。
  
  ![Diagram of a sample sharded cluster for production purposes.  Contains exactly 3 config servers, 2 or more ``mongos`` query routers, and at least 2 shards. The shards are replica sets.](https://docs.mongodb.com/v4.2/_images/sharded-cluster-production-architecture.bakedsvg.svg)
  
  ## Development Configuration
  
  ## 开发环境配置
  
  For testing and development, you can deploy a sharded cluster with a minimum number of components. These **non-production** clusters have the following components:
  
  为了进行测试和开发，您可以部署最少组件数量的分片集群。**非生产**群集具备以下组件：
  
  - A replica set [config server](https://docs.mongodb.com/v4.2/core/sharded-cluster-config-servers/#sharding-config-server) with one member.
  - 一个配置服务器，且其架构为拥有1个成员的[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)。
  - At least one shard as a single-member [replica set](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set).
  - 至少一个分片节点，且其架构拥有1个成员的[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)。
  - One [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos) instance.
  - 一个 [`mongos`](https://docs.mongodb.com/v4.2/reference/program/mongos/#bin.mongos)实例。
  
  ![Diagram of a sample sharded cluster for testing/development purposes only.  Contains only 1 config server, 1 ``mongos`` router, and at least 1 shard. The shard can be either a replica set or a standalone ``mongod`` instance.](https://docs.mongodb.com/v4.2/_images/sharded-cluster-test-architecture.bakedsvg.svg)
  
  click to enlarge
  
  单击放大
  
  WARNING
  
  警告
  
  Use the test cluster architecture for testing and development only.
  
  测试集群架构仅适用于测试和开发场景。
  
  SEE ALSO
  
  另请参阅
  
  [Deploy a Sharded Cluster](https://docs.mongodb.com/v4.2/tutorial/deploy-shard-cluster/)
  
  [部署分片集群](https://docs.mongodb.com/v4.2/tutorial/deploy-shard-cluster/)