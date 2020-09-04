# 配置审计

在本页
*   [启用和配置审计输出](#enable-and-configure-audit-output)
*   [启用和配置审计输出](#enable-and-configure-audit-output)


MONGODB ATLAS中的审计:

MongoDB Atlas支持对所有M10更大的集群进行审计。
 
Atlas支持指定“[配置审计过滤器](../configure-audit-filters/)”中所述的JSON格式的审计过滤器， 并使用Atlas审计过滤器构建器来简化审计配置。

要了解更多信息，请参阅Atlas文档中的“[设置数据库审计](https://docs.atlas.mongodb.com/database-auditing)和[配置自定义审计过滤器](https://docs.atlas.mongodb.com/tutorial/auditing-custom-filter)”。


[MongoDB 企业版](https://www.mongodb.com/products/mongodb-enterprise-advanced?jmp=docs)支持[审计](../../core/auditing/#auditing)各种操作。

完整的审计解决方案必须涉及所有 [`mongod`](../../reference/program/mongod/#bin.mongod "bin.mongod")服务器 和 [`mongos`](../../reference/program/mongos/#bin.mongos "bin.mongos") 路由器过程。

审计工具可以将审计事件写入到控制台、[syslog](../../reference/glossary/#term-syslog)（Windows上不提供该 选项）、JSON文件或BSON文件。有关审计的操作和审计日志消息的详细信息，请参阅系统事件审计消息[系统事件审计消息](../../reference/audit-message/)。


## 启用和配置审计输出


使用该[`--auditDestination`](../../reference/program/mongod/#cmdoption-mongod-auditdestination)选项可以启用审计并指定在何处输出审计事件。


警告

对于分片群集，如果对[`mongos`](../../reference/program/mongos/#bin.mongos "bin.mongos")实例启用审计，则必须对群集中的所有[`mongod`](../../reference/program/mongod/#bin.mongod "bin.mongod")实例（即分片和配置服务器）启用审计。


### 输出到[Syslog](#output-to-syslog "Permalink to this headline")


要启用审计并将审计事件以JSON格式打印到syslog（在Windows上该选项不可用），请为[`--auditDestination`](../../reference/program/mongod/#cmdoption-mongod-auditdestination)设置为syslog。例如：

```sh
mongod --dbpath data/db --auditDestination syslog
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定 --bind_ip。有关更多信息，请参见 [Localhost绑定兼容性更改](../../release-notes/3.6-compatibility/#bind-ip-compatibility)。


重要

绑定到其他IP地址之前，请考虑[启用范围控制](../../administration/security-checklist/#checklist-auth)和其他
绑定到其他IP地址之前，请考虑启用[“安全性检查表”](../../administration/security-checklist/) 中列出的[访问控制](../../administration/security-checklist/#checklist-auth)和其他安全措施，以防止未经授权的访问。

警告

syslog消息限制可能导致审计消息被截断。审计系统不会在发生截断时检测到截断或错误。

您也可以在[配置文件](../../reference/configuration-options/)中指定以下选项：

```yml
storage:
   dbPath: data/db
auditLog:
   destination: syslog
```

### 输出到控制台

要启用审计并将审计事件打印到标准输出（即stdout），请为[`--auditDestination`](../../reference/program/mongod/#cmdoption-mongod-auditdestination)指定参数为'console'。例如：

```sh
mongod --dbpath data/db --auditDestination console
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定 --bind_ip。有关更多信息，请参见 [Localhost绑定兼容性更改](../../release-notes/3.6-compatibility/#bind-ip-compatibility)。

重要

绑定到其他IP地址之前，请考虑启用“安全性检查表”中列出的访问控制和其他安全措施，以防止未经授权的访问。

您也可以在[配置文件中](../../reference/configuration-options/)指定以下选项：

```yml
storage:
   dbPath: data/db
auditLog:
   destination: console
```

### 输出到JSON文件[¶](#output-to-json-file "Permalink to this headline")


要启用审计并将审计事件打印为BSON二进制格式的文件，请指定以下选项：


选项 &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;值 
------
[`--auditDestination`](../../reference/program/mongod/#cmdoption-mongod-auditdestination) &emsp;&emsp;`file`
------
[`--auditFormat`](../../reference/program/mongod/#cmdoption-mongod-auditformat)&emsp;&emsp;&emsp;&emsp;&emsp;`JSON`
------
[`--auditPath`](../../reference/program/mongod/#cmdoption-mongod-auditpath) &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;输出文件名，接受完整路径名或相对路径名。


例如，以下选项启用审计并将审计事件记录到相对路径'data/db/auditLog.json'的文件中：

```sh
mongod --dbpath data/db --auditDestination file --auditFormat JSON --auditPath data/db/auditLog.json
```

包括配置所需的其他选项。例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定--bind_ip参数。有关更多信息，请参见[Localhost绑定兼容性更改](../../release-notes/3.6-compatibility/#bind-ip-compatibility)。


重要：

绑定到其他IP地址之前，请考虑启用[“安全性检查表”](../../administration/security-checklist/)中列出的[访问控制](../../administration/security-checklist/#checklist-auth)和其他安全措施，以防止未经授权的访问。

审计文件与服务器日志文件同时旋转。

您也可以在[配置文件](../../reference/configuration-options/)中指定以下选项：

```yml
storage:
   dbPath: data/db
auditLog:
   destination: file
   format: JSON
   path: data/db/auditLog.json
```

注意

与以BSON格式打印到文件相比，以JSON格式打印审计事件到文件的性能降低服务器性能。


### 输出到BSON文件 [¶](#output-to-bson-file "Permalink to this headline")


要启用审计并将审计事件打印为BSON二进制格式的文件，请指定以下选项：  

选项 &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;值 
------
[`--auditDestination`](../../reference/program/mongod/#cmdoption-mongod-auditdestination) &emsp;&emsp;`file`
------
[`--auditFormat`](../../reference/program/mongod/#cmdoption-mongod-auditformat)&emsp;&emsp;&emsp;&emsp;&emsp;`BSON`
------
[`--auditPath`](../../reference/program/mongod/#cmdoption-mongod-auditpath) &emsp;&emsp;&emsp;&emsp;&emsp;&emsp; 输出文件名，接受完整路径名或相对路径名。


例如，以下选项启用审计并将审计事件记录到相对路径'data/db/auditLog.bson'的文件中：

```sh
mongod --dbpath data/db --auditDestination file --auditFormat BSON --auditPath data/db/auditLog.bson
```

例如，如果您希望远程客户端连接到您的部署，或者您的部署成员在不同的主机上运行，请指定`--bind_ip`。更多信息请查看[Localhost绑定兼容性更改](../../release-notes/3.6-compatibility/#bind-ip-compatibility)。


重要

绑定到其他IP地址之前，请考虑启用[“安全性检查表”](../../administration/security-checklist/)中列出的[访问控制](../../administration/security-checklist/#checklist-auth)和其他安全措施，以防止未经授权的访问。

审计文件与服务器日志文件同时旋转。

您也可以在[配置文件](../../reference/configuration-options/)中指定以下选项：

```yml
storage:
   dbPath: data/db
auditLog:
   destination: file
   format: BSON
   path: data/db/auditLog.bson
```


要查看文件的内容，请将文件传递给MongoDB实用程序 bsondump。例如，以下内容将审计日志转换为可读格式并输出到终端：

```sh
bsondump data/db/auditLog.bson
```

也可以看
[配置审计过滤器](../configure-audit-filters/)，[审计](../../core/auditing/)，[系统事件审计消息](../../reference/audit-message/)

原文链接：https://docs.mongodb.com/manual/tutorial/configure-auditing/

译者：谢伟成
