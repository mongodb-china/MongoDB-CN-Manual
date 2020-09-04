# 在Mac OS安装MongoDB企业版



在本页面

- [概述](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-os-x/#overview)
- [注意事项](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-os-x/#considerations)
- [安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-os-x/#install-mongodb-enterprise-edition)
- [运行MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-os-x/#run-mongodb-enterprise-edition)
- [附加信息](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-os-x/#additional-information)



MONGODB ATLAS

[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是MongoDB公司提供的MongoDB云服务，无需安装开销，并提供免费的入门套餐。



## 概述

使用本教程，可以使用下载的`.tgz`tarball 在macOS上手动安装MongoDB 4.2企业版 。

[MongoDB Enterprise Edition](https://www.mongodb.com/products/mongodb-enterprise-advanced?tck=docs_server) 在某些平台上可用，并且包含对与安全性和监视相关的多种功能的支持。



### MongoDB版本

本教程将安装MongoDB 4.2企业版。要安装其他版本的MongoDB企业版，请使用此页面左上角的版本下拉菜单选择该版本的文档。



## 注意事项

### 平台支持

MongoDB 4.2企业版支持macOS 10.12或更高版本。

有关更多信息，请参见[支持的平台](https://docs.mongodb.com/v4.2/administration/production-notes/#prod-notes-supported-platforms)。

### 生产注意事项

在生产环境中部署MongoDB之前，请考虑 [生产说明](https://docs.mongodb.com/v4.2/administration/production-notes/)文档，该文档提供了生产MongoDB部署的性能注意事项和配置建议。



## 安装MongoDB企业版

请按照以下步骤从 `.tgz`中手动安装MongoDB Enterprise Edition。



### 1. 下载压缩包。

从以下链接下载MongoDB企业版`tgz`tarball：

➤ [MongoDB的下载中心](https://www.mongodb.com/try/download/enterprise?tck=docs_server)

1. 在“ **版本”**下拉列表中，选择要下载的MongoDB版本。
2. 在**平台**下拉列表中，选择**macOS**。
3. 在**包**下拉列表中，选择**tgz**。
4. 点击**下载**。



### 2. 从下载的档案中提取文件。

复制

```
tar -zxvf mongodb-macos-x86_64-enterprise-4.2.8.tgz
```

如果您的网络浏览器在下载过程中自动将文件解压缩，则文件将以`.tar`结尾。



### 3. 确保二进制文件在`PATH`环境变量列出的目录中。

MongoDB二进制文件位于tarball`bin/`目录中。您可以：

- 将二进制文件复制到`PATH` 变量中列出的目录中，例如`/usr/local/bin`（根据需要更新 `/path/to/the/mongodb-directory/`安装目录）

  复制

  ```
  sudo cp /path/to/the/mongodb-directory/bin/* /usr/local/bin/
  ```

- 从`PATH`变量中列出的目录创建指向二进制文件的符号链接，例如`/usr/local/bin`（根据需要更新 `/path/to/the/mongodb-directory/`安装目录）：

  复制

  ```
  sudo ln -s  /path/to/the/mongodb-directory/bin/* /usr/local/bin/
  ```



## 运行MongoDB企业版

请按照以下步骤运行MongoDB企业版。这些说明假定您使用的是默认设置。

### 1. 创建数据目录。

首次启动MongoDB之前，必须创建该[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)进程将向其写入数据的目录。

例如，要创建`/usr/local/var/mongodb`目录：

复制

```
sudo mkdir -p /usr/local/var/mongodb
```

重要

从macOS 10.15 Catalina开始，Apple限制访问MongoDB默认`/data/db`数据目录。在macOS 10.15 Catalina上，您必须使用其他数据目录，例如 `/usr/local/var/mongodb`。



### 2. 创建日志目录。

您还必须创建该`mongod`进程将在其中写入其日志文件的目录：

例如，要创建`/usr/local/var/log/mongodb`目录：

复制

```
sudo mkdir -p /usr/local/var/log/mongodb
```



### 3. 设置数据和日志目录的权限。

确保正在运行的用户帐户[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)对这两个目录具有读写权限。如果您以自己的用户帐户运行[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)，并且刚刚在上面创建了两个目录，则用户应该已经可以访问它们。否则，您可以用`chown`来设置所有权，以替换适当的用户：

复制

```
sudo chown my_mongodb_user /usr/local/var/mongodb
sudo chown my_mongodb_user /usr/local/var/log/mongodb
```



### 4. 运行MongoDB。

要运行MongoDB，请在系统提示符下运行[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)过程，从上方提供`dbpath`和`logpath` 两个参数，并在后台`fork`该参数运行[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)。另外，您也可以选择在 [配置文件](https://docs.mongodb.com/v4.2/reference/configuration-options/)中存储`dbpath`，`logpath`，`fork`值和许多其他的参数。



#### 使用命令行参数运行`mongod`

在系统提示符下运行该[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)过程，直接在命令行上提供三个必需的参数：

复制

```
mongod --dbpath / usr / local / var / mongodb --logpath /usr/local/var/log/mongodb/mongo.log --fork
```



#### 使用配置文件运行`mongod`

在系统提示符下运行[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)过程，并使用`config`参数提供[配置文件](https://docs.mongodb.com/v4.2/reference/configuration-options/)的路径 ：

复制

```
mongod --config /usr/local/etc/mongod.conf
```

MACOS阻止`MONGOD`打开

`mongod`安装后，macOS可能无法运行。如果在启动时收到安全错误，`mongod` 显示无法识别或验证开发人员，请执行以下操作以授予`mongod`运行权限：

- 打开*系统偏好设置*
- 选择“ *安全性和隐私”*窗格。
- 在*常规*选项卡下，单击关于`mongod`消息右侧的按钮，根据您的macOS版本标记为“始终**打开”** 或“ **始终允许”**。



### 5. 验证MongoDB已成功启动。

验证MongoDB已成功启动：

复制

```
ps aux | grep -v grep | grep mongod
```

如果看不到`mongod`进程正在运行，请检查日志文件中是否有任何错误消息。



### 6. 开始使用MongoDB。

在相同的主机上启动[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell 作为[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)。您可以在不使用任何命令行选项的情况下运行[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell ，以使用默认端口*27017*连接到在*本地主机*上*运行的*[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)：

复制

```
mongo
```

- MACOS阻止`MONGOD`打开

  `mongod`安装后，macOS可能无法运行。如果在启动时收到安全错误，`mongod` 显示无法识别或验证开发人员，请执行以下操作以授予`mongod`运行权限：

  - 打开*系统偏好设置*
  - 选择“ *安全性和隐私”*窗格。
  - 在*常规*选项卡下，单击关于`mongod`消息右侧的按钮，根据您的macOS版本标记为“始终**打开”** 或“ **始终允许”**。

有关使用[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell 连接的更多信息，例如连接到[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)在其他主机和/或端口上运行的实例，请参阅[mongo Shell](https://docs.mongodb.com/v4.2/mongo/)。

为了帮助您开始使用MongoDB，MongoDB提供了各种驱动程序版本的[入门指南](https://docs.mongodb.com/v4.2/tutorial/getting-started/#getting-started)。有关可用版本，请参阅 [入门](https://docs.mongodb.com/v4.2/tutorial/getting-started/#getting-started)。



## 其他信息

### 默认为localhost绑定

默认情况下，MongoDB在启动时将[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)设置为 `127.0.0.1`，该绑定到localhost网络接口。这意味着`mongod`只能接受来自同一计算机上运行的客户端的连接。除非将此值设置为有效的网络接口，否则远程客户端将无法连接到`mongod`，并且`mongod`不能初始化[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)。

可以配置以下值：

- 在MongoDB配置文件中使用[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)，或
- 通过命令行参数 [`--bind_ip`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-bind-ip)



警告

绑定到非本地主机（例如，可公共访问）的IP地址之前，请确保已保护群集免受未经授权的访问。有关安全建议的完整列表，请参阅“ [安全清单”](https://docs.mongodb.com/v4.2/administration/security-checklist/)。至少应考虑 [启用身份验证](https://docs.mongodb.com/v4.2/administration/security-checklist/#checklist-auth)并 [强化网络基础架构](https://docs.mongodb.com/v4.2/core/security-hardening/)。

有关配置的更多信息[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)，请参见 [IP绑定](https://docs.mongodb.com/v4.2/core/security-mongodb-configuration/)。

←  [使用.tgz Tarball在Amazon Linux上安装MongoDB Enterprise](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-amazon-tarball/)<br/>[在Windows上安装MongoDB企业版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-windows/) →



原文链接：https://docs.mongodb.com/v4.2/tutorial/install-mongodb-enterprise-on-os-x/

译者：小芒果
