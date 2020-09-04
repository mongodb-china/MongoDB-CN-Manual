# 入门

下方页面提供了在MongoDB Shell中进行查询的各种示例。有关使用MongoDB驱动程序的示例，请参阅“ [其他示例”](https://docs.mongodb.com/v4.2/tutorial/getting-started/#gs-additional-examples)部分中的链接。



## 示例

<iframe class="mws-root" allowfullscreen="" sandbox="allow-scripts allow-same-origin" width="100%" height="320" src="https://mws.mongodb.com/?version=4.2" style="box-sizing: border-box; border: 0px;"></iframe>

在 [shell](https://docs.mongodb.com/v4.2/tutorial/getting-started/#mongo-web-shell)内单击以进行连接。连接后，您可以在上面的 [shell](https://docs.mongodb.com/v4.2/tutorial/getting-started/#mongo-web-shell)中运行示例。

##### 切换数据库

在[shell中](https://docs.mongodb.com/v4.2/tutorial/getting-started/#mongo-web-shell)，`db`是指您当前的数据库。键入`db`以显示当前数据库。

复制

```
db
```

该操作应返回`test`，这是默认数据库。

要切换数据库，请键入 `use <db>`。例如，要切换到 `examples` 数据库：

复制

```
use examples
```

切换之前您无需创建数据库。当您第一次在数据库中存储数据时（例如在数据库中创建第一个集合），MongoDB会创建数据库。

要验证您的数据库现在是`examples`，在上面的[shell中](https://docs.mongodb.com/v4.2/tutorial/getting-started/#mongo-web-shell)键入`db`。

复制

```
db
```

要在数据库中创建集合，请参见下一个选项卡。

> 译者注：填充一个集合（插入）/选择所有文档/指定平等匹配/指定要返回的字段（投影）相关操作请到原文查看和复制代码。
>
> 链接：https://docs.mongodb.com/v4.2/tutorial/getting-started/



## 下一步

### 建立自己的部署

要设置自己的部署：

| MongoDB Atlas免费套餐集群 | MongoDB Atlas是一种快速，便捷，免费的MongoDB入门途径。要了解更多信息，请参阅 [Atlas入门](https://docs.atlas.mongodb.com/getting-started/)教程。 |
| ------------------------- | ------------------------------------------------------------ |
| 本地MongoDB安装           | 有关在本地安装MongoDB的更多信息，请参阅 [安装MongoDB](https://docs.mongodb.com/v4.2/installation/#tutorial-installation)。 |



### 其他示例

有关其他示例，包括MongoDB驱动程序特定的示例（Python，Java，Node.js等），请参阅：

| 查询文档示例 | [查询文档](https://docs.mongodb.com/v4.2/tutorial/query-documents/)  [查询嵌入/嵌套文档](https://docs.mongodb.com/v4.2/tutorial/query-embedded-documents/)  [查询数组](https://docs.mongodb.com/v4.2/tutorial/query-arrays/)  [查询嵌入式文档数组](https://docs.mongodb.com/v4.2/tutorial/query-array-of-documents/)  [从查询返回的项目字段](https://docs.mongodb.com/v4.2/tutorial/project-fields-from-query-results/)  [查询空字段或缺少字段](https://docs.mongodb.com/v4.2/tutorial/query-for-null-fields/) |
| ------------ | ------------------------------------------------------------ |
| 更新文档示例 | [更新文档](https://docs.mongodb.com/v4.2/tutorial/update-documents/) |
| 删除文档示例 | [删除文档](https://docs.mongodb.com/v4.2/tutorial/remove-documents/) |



### 其他主题

| 介绍                                                         | 开发者                                                       | 管理员                                                       | 参考                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| [MongoDB简介](https://docs.mongodb.com/v4.2/introduction/)  [安装指南](https://docs.mongodb.com/v4.2/installation/)  [数据库和集合](https://docs.mongodb.com/v4.2/core/databases-and-collections/)  [文档资料](https://docs.mongodb.com/v4.2/core/document/) | [CRUD操作](https://docs.mongodb.com/v4.2/crud/)  [聚合](https://docs.mongodb.com/v4.2/aggregation/)  [SQL到MongoDB](https://docs.mongodb.com/v4.2/reference/sql-comparison/)  [索引](https://docs.mongodb.com/v4.2/indexes/) | [生产须知](https://docs.mongodb.com/v4.2/administration/production-notes/)  [副本集](https://docs.mongodb.com/v4.2/replication/)  [分片集群](https://docs.mongodb.com/v4.2/sharding/)  [MongoDB安全](https://docs.mongodb.com/v4.2/security/) | [Shell方法](https://docs.mongodb.com/v4.2/reference/method/)  [查询运算符](https://docs.mongodb.com/v4.2/reference/operator/)  [参考](https://docs.mongodb.com/v4.2/reference/)  [词汇表](https://docs.mongodb.com/v4.2/reference/glossary/) |





原文链接：https://docs.mongodb.com/v4.2/tutorial/getting-started/

译者：小芒果
