# 在Windows上安装MongoDB社区版

在本页面

- [概述](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#overview)
- [注意事项](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#considerations)
- [安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#install-mongodb-community-edition)
- [将MongoDB社区版作为Windows服务运行](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#run-mongodb-community-edition-as-a-windows-service)
- [从命令解释器运行MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#run-mongodb-community-edition-from-the-command-interpreter)
- [其他注意事项](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#additional-considerations)



MONGODB ATLAS

[MongoDB Atlas](https://www.mongodb.com/cloud/atlas?tck=docs_server) 是MongoDB公司提供的MongoDB云服务，无需安装开销，并提供免费的入门套餐。



## 概述

使用本教程可以使用默认安装向导在Windows上安装MongoDB 4.2社区版。



### MongoDB版本

本教程将安装MongoDB 4.2社区版。要安装其他版本的MongoDB社区，请使用此页面左上角的版本下拉菜单选择该版本的文档。

### 安装方法

本教程使用默认安装向导在Windows上安装MongoDB。或者，您可以选择使用`msiexec.exe`命令行（`cmd.exe`）以无人参与的方式在Windows上安装MongoDB 。这对于希望使用自动化部署MongoDB的系统管理员很有用。

➤有关说明，请参阅[使用msiexec.exe在Windows上安装MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows-unattended/)。



## 注意事项

### 平台支持

MongoDB 4.2 社区版在[x86_64](https://docs.mongodb.com/v4.2/administration/production-notes/#prod-notes-supported-platforms-x86-64)架构上支持Windows 的以下 **64位**版本 ：

- Windows Server 2019
- Windows 10 / Windows Server 2016
- Windows 8.1 / Windows Server 2012 R2
- Windows 8 / Windows Server 2012
- Windows 7 / Windows Server 2008 R2

MongoDB仅支持这些平台的64位版本。

有关更多信息，请参见[支持的平台](https://docs.mongodb.com/v4.2/administration/production-notes/#prod-notes-supported-platforms)。



### 生产注意事项

在生产环境中部署MongoDB之前，请考虑 [生产说明](https://docs.mongodb.com/v4.2/administration/production-notes/)文档，该文档提供了生产MongoDB部署的性能注意事项和配置建议。



## 安装MongoDB社区版

### 前提条件

Windows 10之前的Windows版本上的用户必须在安装MongoDB之前安装以下更新：

➤ [Windows系统Universal C运行时更新](https://support.microsoft.com/en-us/help/2999226/update-for-universal-c-runtime-in-windows)

Windows 10，Server 2016和Server 2019上的用户不需要此更新。



### 程序

请按照以下步骤使用MongoDB安装程序向导安装MongoDB社区版。安装过程将同时安装MongoDB二进制文件和默认[配置文件](https://docs.mongodb.com/v4.2/reference/configuration-options/)  `<install directory>\bin\mongod.cfg`。



#### 1. 下载安装程序。

从以下链接下载MongoDB社区安装程序`.msi`：

➤ [MongoDB的下载中心](https://www.mongodb.com/try/download/community?tck=docs_server)

1. 在“ **版本”**下拉列表中，选择要下载的MongoDB版本。
2. 在**平台**下拉菜单中，选择**Windows**。
3. 在**Package**下拉列表中，选择**msi**。
4. 点击**下载**。



#### 2. 运行MongoDB安装程序。

例如，从Windows资源管理器/文件资源管理器中：

1. 转到下载MongoDB安装程序的目录（`.msi`文件）。默认情况下，这是您的`Downloads`目录。
2. 双击`.msi`文件。



#### 3. 遵循MongoDB社区版安装向导。

该向导将引导您完成MongoDB和MongoDB Compass的安装。

1. - **选择安装类型**

     您可以选择“ **完整”**（建议大多数用户使用）或“ **自定义”**安装类型。“ **完整**设置”选项会将MongoDB和MongoDB工具安装到默认位置。使用“ **自定义** 安装”选项可以指定要安装的可执行文件以及安装位置。
   
2. - **服务配置**

     从MongoDB 4.0开始，您可以在安装过程中将MongoDB设置为Windows服务，也可以仅安装二进制文件。
     
     - MongoDB服务
     - MongoDB
     
     以下内容将MongoDB安装并配置为Windows服务。
     从MongoDB 4.0开始，您可以在安装过程中将MongoDB配置和启动为Windows服务，并在成功安装后启动MongoDB服务。  
 ![Image of the MongoDB Installer wizard - Service Configuration.](https://docs.mongodb.com/v4.2/_images/windows-installer.png)
     - 选择“ **将MongoDB作为服务安装”**。
     
     - 选择以下任一项：
     
       - **以网络服务用户身份运行服务**（默认）
     
         这是Windows内置的Windows用户帐户
     
         **或者**
     
       - **以本地或域用户身份运行服务**
     
         - 对于现有的本地用户帐户，请为“ **帐户域”**指定一个句点（即`.`），并为该用户指定“ **帐户名”**和“ **帐户密码** ”。
         - 对于现有的域用户，请为该用户指定“ **帐户域”**，“ **帐户名称”**和“ **帐户密码** ”。
         - **服务名称**。指定服务名称。默认名称为`MongoDB`。如果您已经拥有使用指定名称的服务，则必须选择另一个名称。
         - **数据目录**。指定数据目录，它对应于 [`--dbpath`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-dbpath)。如果目录不存在，安装程序将创建该目录并设置对服务用户的目录访问权限。
         - **日志目录**。指定日志目录，它对应于 [`--logpath`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-logpath)。如果目录不存在，安装程序将创建该目录并设置对服务用户的目录访问权限。
   
3. - 对于Windows 8或更高版本，您可以让向导安装 [MongoDB Compass](https://www.mongodb.com/products/compass)。要安装Compass，请选择**Install MongoDB Compass**（默认）。

     注意
     
     安装脚本需要PowerShell 3.0或更高版本。如果您使用Windows 7，请取消单击 **Install MongoDB Compass**。您可以[从下载中心](https://www.mongodb.com/download-center/compass?tck=docs_server)手动[下载Compass](https://www.mongodb.com/download-center/compass?tck=docs_server)。
   
4. 准备就绪后，点击**安装**。

### 如果您将MongoDB安装为Windows服务

成功安装后将启动MongoDB服务[[1\]](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#cfg)。

要开始使用MongoDB，请将[`mongo.exe`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell 连接到正在运行的MongoDB实例。要么：

- 在Windows资源管理器/文件资源管理器中，转到目录`C:\Program Files\MongoDB\Server\4.2\bin\`，然后双击 [mongo.exe`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo)

- 或者，使用管理特权打开**命令解释器**并运行：

  复制

  ```
  “ C：\ Program Files \ MongoDB \ Server \ 4.2 \ bin \ mongo.exe”
  ```

有关CRUD（创建，读取，更新，删除）操作的信息，请参阅：

- [插入文档](https://docs.mongodb.com/v4.2/tutorial/insert-documents/)
- [查询文档](https://docs.mongodb.com/v4.2/tutorial/query-documents/)
- [更新文档](https://docs.mongodb.com/v4.2/tutorial/update-documents/)
- [删除文档](https://docs.mongodb.com/v4.2/tutorial/remove-documents/)

| [[1\]](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#id1) | 使用配置文件`<install directory>\bin\mongod.cfg`配置MongoDB实例 。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |



### 如果您没有将MongoDB安装为Windows服务[¶](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#if-you-did-not-install-mongodb-as-a-windows-service)

如果您仅安装了可执行文件而没有将MongoDB作为Windows服务安装，则必须手动启动MongoDB实例。

有关启动MongoDB实例的说明，请参阅[从命令解释器运行MongoDB社区版](https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/#run-mongodb-from-cmd)。



## 将社区版MongoDB作为Windows服务运行

从版本4.0开始，您可以在安装过程中将MongoDB安装和配置为 **Windows服务**，并在成功安装后启动MongoDB服务。使用配置文件 `<install directory>\bin\mongod.cfg`配置MongoDB 。



### 将社区版MongoDB作为Windows服务启动

要启动/重新启动MongoDB服务，请使用服务控制台：

1. 在服务控制台中，找到MongoDB服务。
2. 右键单击MongoDB服务，然后单击**启动**。

要开始使用MongoDB，请将[`mongo.exe`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell 连接到正在运行的MongoDB实例。要进行连接，请打开具有管理权限的**命令解释器**并运行：

复制

```
“ C：\ Program Files \ MongoDB \ Server \ 4.2 \ bin \ mongo.exe”
```

有关[`mongo.exe`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell的更多信息，例如连接到在不同主机和/或端口上运行的MongoDB实例，请参阅[mongo Shell](https://docs.mongodb.com/v4.2/mongo/)。

有关CRUD（创建，读取，更新，删除）操作的信息，请参阅

- [插入文档](https://docs.mongodb.com/v4.2/tutorial/insert-documents/)
- [查询文档](https://docs.mongodb.com/v4.2/tutorial/query-documents/)
- [更新文档](https://docs.mongodb.com/v4.2/tutorial/update-documents/)
- [删除文档](https://docs.mongodb.com/v4.2/tutorial/remove-documents/)



### 将社区版MongoDB作为Windows服务停止

要停止/暂停MongoDB服务，请使用服务控制台：

1. 在服务控制台中，找到MongoDB服务。
2. 右键单击MongoDB服务，然后单击“ **停止”**（或“ **暂停”**）。



### 将社区版MongoDB作为Windows服务删除

要删除MongoDB服务，请首先使用服务控制台停止该服务。然后以**管理员**身份打开[Windows命令提示符/解释器](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmd)（`cmd.exe`），然后运行以下命令：

复制

```
sc.exe delete MongoDB
```



## 从命令解释器运行MongoDB社区版

您可以从[Windows命令提示符/解释器](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmd)（`cmd.exe`）而不是作为服务运行MongoDB社区版。

以**管理员**身份打开[Windows命令提示符/解释器](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmd)（`cmd.exe`）。

重要

您必须以**管理员**身份打开命令解释器 。



### 1. 创建数据库目录。

创建MongoDB存储数据的[数据目录](https://docs.mongodb.com/v4.2/reference/glossary/#term-dbpath)。MongoDB的默认数据目录路径是 `\data\db` 启动MongoDB的驱动上的绝对路径  。

在**命令解释器中**，创建数据目录：

复制

```
cd C:\
md "\data\db"
```



### 2. 启动您的MongoDB数据库。

要启动MongoDB，请运行[`mongod.exe`](https://docs.mongodb.com/v4.2/reference/program/mongod.exe/#bin.mongod.exe)。

复制

```
"C:\Program Files\MongoDB\Server\4.2\bin\mongod.exe" --dbpath="c:\data\db"
```

该[`--dbpath`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-dbpath)选项指向您的数据库目录。

如果MongoDB数据库服务器正常运行，则 **命令解释器将**显示：

复制

```
[initandlisten] waiting for connections
```

重要

根据 Windows主机上的 [Windows Defender防火墙](https://docs.microsoft.com/en-us/windows/security/identity-protection/windows-firewall/windows-firewall-with-advanced-security)设置，Windows可能会显示“ **安全警报”**对话框，提示`C:\Program Files\MongoDB\Server\4.2\bin\mongod.exe`的“某些功能” 在网络上进行通信被阻止。要解决此问题：

1. 点击**专用网络，例如我的家庭或工作网络**。
2. 点击**允许访问**。

要了解有关安全性和MongoDB的更多信息，请参阅“ [安全性文档”](https://docs.mongodb.com/v4.2/security/)。



### 3. 连接到MongoDB。

要将[`mongo.exe`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell 连接到MongoDB实例，请打开另一个 具有管理权限的**命令解释器**，然后运行：

复制

```
"C:\Program Files\MongoDB\Server\4.2\bin\mongo.exe"
```

有关连接[`mongo.exe`](https://docs.mongodb.com/v4.2/reference/program/mongo/#bin.mongo) shell 的更多信息，例如连接到在不同主机和/或端口上运行的MongoDB实例，请参阅[The mongo Shell](https://docs.mongodb.com/v4.2/mongo/)。

有关CRUD（创建，读取，更新，删除）操作的信息，请参阅：

- [插入文档](https://docs.mongodb.com/v4.2/tutorial/insert-documents/)
- [查询文档](https://docs.mongodb.com/v4.2/tutorial/query-documents/)
- [更新文档](https://docs.mongodb.com/v4.2/tutorial/update-documents/)
- [删除文档](https://docs.mongodb.com/v4.2/tutorial/remove-documents/)



## 其他注意事项

### 默认为localhost绑定

默认情况下，MongoDB在启动时将[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)设置为 `127.0.0.1`，绑定到localhost网络接口。这意味着`mongod.exe`只能接受来自同一计算机上运行的客户端的连接。除非将此值设置为有效的网络接口，否则远程客户端将无法连接到`mongod.exe`，并且`mongod.exe`不能初始化[副本集](https://docs.mongodb.com/v4.2/reference/glossary/#term-replica-set)。

可以配置以下值：

- 在MongoDB配置文件中使用[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)，或
- 通过命令行参数 [`--bind_ip`](https://docs.mongodb.com/v4.2/reference/program/mongod/#cmdoption-mongod-bind-ip)



警告

绑定到非本地主机（例如，可公共访问）的IP地址之前，请确保已保护群集免受未经授权的访问。有关安全建议的完整列表，请参阅“ [安全清单”](https://docs.mongodb.com/v4.2/administration/security-checklist/)。至少应考虑 [启用身份验证](https://docs.mongodb.com/v4.2/administration/security-checklist/#checklist-auth)并 [强化网络基础架构](https://docs.mongodb.com/v4.2/core/security-hardening/)。

有关配置[`bindIp`](https://docs.mongodb.com/v4.2/reference/configuration-options/#net.bindIp)的更多信息，请参见 [IP绑定](https://docs.mongodb.com/v4.2/core/security-mongodb-configuration/)。



### 版本发布和 `.msi`

如果您使用Windows安装程序（`.msi`） 安装了MongoDB，`.msi`将在其[发行系列](https://docs.mongodb.com/v4.2/reference/versioning/#release-version-numbers)（例如4.2.1到4.2.2）中自动升级。

升级完整版本系列（例如4.0至4.2）需要重新安装。



### 将MongoDB二进制文件添加到系统路径

本教程中的所有命令行示例均作为MongoDB二进制文件的绝对路径提供。您可以将`C:\Program Files\MongoDB\Server\4.2\bin`添加到系统路径中，然后省略MongoDB二进制文件的完整路径。



原文链接：https://docs.mongodb.com/v4.2/tutorial/install-mongodb-on-windows/

译者：汪子豪

update：小芒果













