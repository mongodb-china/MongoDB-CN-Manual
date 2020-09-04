# 在macOS上安装MongoDB社区版



在本页面

- [概述](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#overview)
- [注意事项](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#considerations)
- [安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#install-mongodb-community-edition)
- [附加信息](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#additional-information)



MONGODB ATLAS

[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是MongoDB公司提供的MongoDB云服务，无需安装开销，并提供免费的入门套餐。



## 概述

使用本教程可使用第三方`brew`包管理器在macOS上安装MongoDB 4.2社区版。



### MongoDB版本

本教程将安装MongoDB 4.2社区版。要安装其他版本的MongoDB，请使用此页面左上角的版本下拉菜单选择该版本的文档。



## 注意事项

### 平台支持

MongoDB 4.2 社区版支持macOS 10.12或更高版本。

有关更多信息，请参见[支持的平台](https://docs.mongodb.com/v4.2/administration/production-notes/#prod-notes-supported-platforms)。

### 生产注意事项

在生产环境中部署MongoDB之前，请考虑 [生产说明](https://docs.mongodb.com/v4.2/administration/production-notes/)文档，该文档提供了生产MongoDB部署的性能注意事项和配置建议。



## 安装MongoDB社区版[¶](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#install-mongodb-community-edition)

### 前提条件

如果您在OSX主机上安装了Homebrew `brew`软件包， *并且*以前已经使用了官方的 [MongoDB Homebrew Tap](https://github.com/mongodb/homebrew-brew)，请跳过前提条件并转到“ [过程”](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#install)步骤。



#### 安装XCode 

Apple的XCode包含所需的`brew`命令行工具，可在App Store上免费获得。确保您正在运行最新版本。



#### 安装Homebrew

OSX 默认不包括Homebrew`brew`软件包。按照 [官方说明进行](https://brew.sh/#install)安装`brew`。



#### 点击MongoDB Homebrew 

在终端上发出以下命令，以点击官方的 [MongoDB Homebrew Tap](https://github.com/mongodb/homebrew-brew)：

复制

```
brew tap mongodb/brew
```



### 过程

请按照以下步骤使用第三方`brew`程序包管理器安装MongoDB社区版。

在终端上，发出以下命令：

复制

```
brew install mongodb-community@4.2
```

提示

如果您以前安装了该公式的较旧版本，则可能会遇到ChecksumMismatchError。若要解决，请参阅 [ChecksumMismatchError故障排除](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#troubleshooting-checksumerror)。

除[二进制文件外](https://docs.mongodb.com/v4.2/reference/program/)，安装还会创建：

- [配置文件](https://docs.mongodb.com/v4.2/reference/configuration-options/) （`/usr/local/etc/mongod.conf`）
- （）[`log directory path`](https://docs.mongodb.com/v4.2/reference/configuration-options/#systemLog.path)`/usr/local/var/log/mongodb`
- （）[`data directory path`](https://docs.mongodb.com/v4.2/reference/configuration-options/#storage.dbPath)`/usr/local/var/mongodb`



### 运行MongoDB社区版

请按照以下步骤运行MongoDB社区版。这些说明假定您使用的是默认设置。

您可以使用`brew`来将MongoDB作为macOS服务运行，也可以作为后台进程手动运行MongoDB。建议将MongoDB作为macOS服务运行，因为这样做会自动设置正确的系统`ulimit`值（有关更多信息，请参阅 [ulimit设置](https://docs.mongodb.com/v4.2/reference/ulimit/#ulimit-settings)）。

- 要将MongoDB（即[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)进程）**作为macOS服务运行**，请发出以下命令：

  复制

  ```
  brew services start mongodb-community@4.2
  ```

  要停止[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)作为macOS服务运行，请根据需要使用以下命令：

  复制

  ```
brew services stop mongodb-community@4.2
  ```
  
  

- 要将MongoDB（即[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)进程）**作为后台进程手动**运行，请发出以下命令：

  复制

  ```
  mongod --config /usr/local/etc/mongod.conf --fork
  ```

  要停止[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)作为后台进程运行，请从**mongo** shell 连接到[`mongod`](https://docs.mongodb.com/v4.2/reference/program/mongod/#bin.mongod)，然后根据需要发出[`shutdown`](https://docs.mongodb.com/v4.2/reference/command/shutdown/#dbcmd.shutdown)命令。



两种方法都使用在安装过程中创建的`/usr/local/etc/mongod.conf`文件。您也可以将自己的MongoDB [配置选项](https://docs.mongodb.com/v4.2/reference/configuration-options/)添加到此文件。

MACOS阻止`MONGOD`打开

`mongod`安装后，macOS可能无法运行。如果在启动时收到安全错误，`mongod` 显示无法识别或验证开发人员，请执行以下操作以授予`mongod`运行权限：

- 打开*系统偏好设置*
- 选择“ *安全性和隐私”*窗格。
- 在*常规*选项卡下，单击关于`mongod`消息右侧的按钮，根据您的macOS版本标记为“始终**打开”** 或“ **始终允许”**。



要验证MongoDB是否正在运行，请在正在运行的进程中搜索`mongod`：

复制

```
ps aux | grep -v grep | grep mongod
```

您还可以查看日志文件以查看`mongod`进程的当前状态 ：`/usr/local/var/log/mongodb/mongo.log`。



### 连接和使用MongoDB

要开始使用MongoDB，请将[`mongo`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)shell 连接到正在运行的实例。在新终端上，发出以下命令：

复制

```
mongo
```

- MACOS阻止`MONGOD`打开

  `mongod`安装后，macOS可能无法运行。如果在启动时收到安全错误，`mongod` 显示无法识别或验证开发人员，请执行以下操作以授予`mongod`运行权限：

  - 打开*系统偏好设置*
  - 选择“ *安全性和隐私”*窗格。
  - 在*常规*选项卡下，单击关于`mongod`消息右侧的按钮，根据您的macOS版本标记为“始终**打开”** 或“ **始终允许”**。



有关CRUD（创建，读取，更新，删除）操作的信息，请参阅：

- [插入文档](https://docs.mongodb.com/v4.2/tutorial/insert-documents/)
- [查询文档](https://docs.mongodb.com/v4.2/tutorial/query-documents/)
- [更新文档](https://docs.mongodb.com/v4.2/tutorial/update-documents/)
- [删除文档](https://docs.mongodb.com/v4.2/tutorial/remove-documents/)



## 其他信息

### 默认为localhost绑定

默认情况下，MongoDB在启动时将[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)设置为 `127.0.0.1`，绑定到localhost网络接口。这意味着`mongod`只能接受来自同一计算机上运行的客户端的连接。除非将此值设置为有效的网络接口，否则远程客户端将无法连接到`mongod`，并且`mongod`不能初始化[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)。

可以配置以下值：

- 在MongoDB配置文件中使用[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)，或
- 通过命令行参数 [`--bind_ip`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-bind-ip)

警告

绑定到非本地主机（例如，可公共访问）的IP地址之前，请确保已保护群集免受未经授权的访问。有关安全建议的完整列表，请参阅“ [安全清单”](https://docs.mongodb.com/v4.2/administration/security-checklist/)。至少应考虑 [启用身份验证](https://docs.mongodb.com/v4.2/administration/security-checklist/#checklist-auth)并 [强化网络基础架构](https://docs.mongodb.com/v4.2/core/security-hardening/)。

有关配置的更多信息[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)，请参见 [IP绑定](https://docs.mongodb.com/v4.2/core/security-mongodb-configuration/)。



### 对ChecksumMismatchError进行故障排除[¶](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/#troubleshooting-checksummismatcherror)

如果您以前安装了该公式的较旧版本，则可能会遇到类似于以下内容的ChecksumMismatchError：

复制

```
Error: An exception occurred within a child process:

  ChecksumMismatchError: SHA256 mismatch

Expected: c7214ee7bda3cf9566e8776a8978706d9827c1b09017e17b66a5a4e0c0731e1f

  Actual: 6aa2e0c348e8abeec7931dced1f85d4bb161ef209c6af317fe530ea11bbac8f0

 Archive: /Users/kay/Library/Caches/Homebrew/downloads/a6696157a9852f392ec6323b4bb697b86312f0c345d390111bd51bb1cbd7e219--mongodb-macos-x86_64-4.2.0.tgz

To retry an incomplete download, remove the file above.
```

修复：

1. 删除下载的`.tgz`档案。
2. 点击公式。

复制

```
brew untap mongodb/brew && brew tap mongodb/brew
```

2. 重试安装。

   复制

   ```
   brew install mongodb-community@4.2
   ```



←  [Install MongoDB Community on Amazon Linux using .tgz Tarball](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-amazon-tarball/)<br/>[Install MongoDB Community on macOS using .tgz Tarball](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x-tarball/) →



原文链接：https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-os-x/

译者：小芒果
