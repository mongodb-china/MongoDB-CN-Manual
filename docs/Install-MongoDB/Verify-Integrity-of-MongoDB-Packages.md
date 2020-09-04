# 验证MongoDB软件包的完整性

在本页面

- [验证Linux / macOS软件包](https://docs.mongodb.com/v4.2/tutorial/verify-mongodb-packages/#verify-linux-macos-packages)
- [验证Windows软件包](https://docs.mongodb.com/v4.2/tutorial/verify-mongodb-packages/#verify-windows-packages)

MongoDB版本团队对所有软件包进行数字签名，以证明特定的MongoDB软件包是有效且未更改的MongoDB版本。在安装MongoDB之前，您应该使用提供的PGP签名或SHA-256校验和来验证软件包。

通过检查文件的真实性和完整性以防止篡改，PGP签名提供了最有力的保证。

加密校验和仅验证文件完整性以防止网络传输错误。



## 验证的Linux / MacOS的包

### 使用PGP / GPG 

MongoDB使用不同的PGP密钥在每个发行分支上签名。自MongoDB 2.2起，每个发行分支的公钥文件都可以从[密钥服务器](https://www.mongodb.org/static/pgp/) 以文本`.asc`和二进制`.pub`格式下载。



### 1. 下载MongoDB安装文件。

根据您的环境从[MongoDB下载中心](https://www.mongodb.com/try/download?tck=docs_server)下载二进制文件。

例如，要通过shell下载macOS`4.2.8`发行版，请运行以下命令：

复制

```
curl -LO https://fastdl.mongodb.org/osx/mongodb-macos-x86_64-4.2.8.tgz
```



### 2. 下载公共签名文件。

复制

```
curl -LO https://fastdl.mongodb.org/osx/mongodb-macos-x86_64-4.2.8.tgz.sig
```



### 3. 下载然后导入密钥文件。

如果尚未下载并导入MongoDB 4.2公钥，请运行以下命令：

复制

```
curl -LO https://www.mongodb.org/static/pgp/server-4.2.asc
gpg --import server-4.2.asc
```

PGP应该返回以下响应：

复制

```
gpg: key 4B7C549A058F8B6B: "MongoDB 4.2 Release Signing Key <packaging@mongodb.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1
```



### 4. 验证MongoDB安装文件。

运行以下命令：

复制

```
gpg --verify mongodb-macos-x86_64-4.2.8.tgz.sig mongodb-macos-x86_64-4.2.8.tgz
```



GPG应该返回以下响应：

复制

```
gpg: Signature made Wed Jun  5 03:17:20 2019 EDT
gpg:                using RSA key 4B7C549A058F8B6B
gpg: Good signature from "MongoDB 4.2 Release Signing Key <packaging@mongodb.com>" [unknown]
```



如果软件包已正确签名，但是您当前不信任本地密钥`trustdb`，`gpg`则还会返回以下消息：

复制

```
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: E162 F504 A20C DF15 827F  718D 4B7C 549A 058F 8B6B
```



如果您收到以下错误消息，请确认您导入了正确的公钥：

复制

```
gpg: Can't check signature: public key not found
```





### 使用SHA-256



### 1. 下载MongoDB安装文件。

根据您的环境从[MongoDB下载中心](https://www.mongodb.com/try/download?tck=docs_server)下载二进制文件。

例如，要通过shell下载macOS`4.2.8`发行版，请输入以下命令：

复制

```
curl -LO https://fastdl.mongodb.org/osx/mongodb-macos-x86_64-4.2.8.tgz
```



### 2. 下载SHA256文件。

复制

```
curl -LO https://fastdl.mongodb.org/osx/mongodb-macos-x86_64-4.2.8.tgz.sha256
```



### 3. 使用SHA-256校验和验证MongoDB软件包文件。

计算软件包文件的校验和：

复制

```
shasum -c mongodb-macos-x86_64-4.2.8.tgz.sha256
```

如果校验和与下载的软件包匹配，它将返回以下内容：

复制

```
mongodb-macos-x86_64-4.2.8.tgz: OK
```





## 验证Windows软件包

这将根据其SHA256密钥验证MongoDB二进制文件。



### 1. 下载安装程序。

下载MongoDB `.msi`安装程序。例如，要下载最新版本的社区版MongoDB：

➤ [MongoDB社区版下载中心](https://www.mongodb.com/try/download/community?tck=docs_server)

1. 在**版本**下拉列表中，选择 `4.2.8 (current release)`。
2. 在**平台**下拉菜单中，选择**Windows**。
3. 在**Package**下拉列表中，选择**msi**。
4. 单击**下载，**然后将文件保存到下载文件夹中。



### 2. 获取公共签名文件。

获取您的MongoDB版本的公共签名文件。

例如，对于最新版本社区版MongoDB的SHA256签名：

1. 从https://fastdl.mongodb.org/win32/mongodb-win32-x86_64-2012plus-4.2.8-signed.msi.sha256复制内容。
2. 将内容保存到“ `mongodb-win32-x86_64-2012plus-4.2.8-signed.msi.sha256`”下载文件中的文件夹中。



### 3. 将签名文件与MongoDB installer hash进行比较。

要将签名文件与MongoDB二进制文件的哈希值进行比较，请调用以下Powershell脚本：

复制

```
$sigHash = (Get-Content $Env:HomePath\Downloads\mongodb-win32-x86_64-2012plus-4.2.8-signed.msi.sha256 | Out-String).SubString(0,64).ToUpper(); `
$fileHash = (Get-FileHash $Env:HomePath\Downloads\mongodb-win32-x86_64-2012plus-4.2.8-signed.msi).Hash.Trim(); `
echo $sigHash; echo $fileHash; `
$sigHash -eq $fileHash
```

复制

```
AF5AF79EFE540DCDDC2825A396C71FCFC4FEB463BC9CADDCCDE20AD126321CCC
AF5AF79EFE540DCDDC2825A396C71FCFC4FEB463BC9CADDCCDE20AD126321CCC
True
```

该命令输出三行：

- 您直接从MongoDB下载的`SHA256`哈希。
- 一个你从MongoDB下载的MongoDB二进制计算`SHA256`哈希值。
- 一个取决于哈希匹配的`True`或者`False`结果。

如果哈希匹配，则将验证MongoDB二进制文件。



←  [升级到MongoDB企业版（分片集群）](https://docs.mongodb.com/v4.2/tutorial/upgrade-to-enterprise-sharded-cluster/)<br/>[The mongo Shell](https://docs.mongodb.com/v4.2/mongo/) →



原文链接：https://docs.mongodb.com/v4.2/tutorial/verify-mongodb-packages/

译者：小芒果
