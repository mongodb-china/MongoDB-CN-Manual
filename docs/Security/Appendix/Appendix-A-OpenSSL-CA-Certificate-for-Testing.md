# 附录 A - 用于测试的 OpenSSl CA 证书


免责声明

提供此页面仅用于**测试目的**，证书仅用于**测试目的**。

以下教程提供了一些有关创建**测试** x.509证书的准则 ：

* 请勿将这些证书用于生产。相反，请遵循您的安全策略。
* 有关 OpenSSL 的信息，请参考官方的 OpenSSL 文档。尽管本教程使用的是 OpenSSL，但不应将本材料当作 OpenSSL 的权威参考。

## 程序

以下过程概述了创建**测试** CA PEM 文件的步骤。该过程同时创建 CA PEM 文件和中间授权证书以及用于签署服务器/客户端**测试**证书的密钥文件。

### A.创建OpenSSL配置文件

1. 创建具有以下内容的配置文件 `openssl-test-ca.cnf`：
   ```
   # NOT FOR PRODUCTION USE. OpenSSL configuration file for testing.
   # 不用于生产用途。用于测试的OpenSSL配置文件。
   
   # For the CA policy
   # 对于CA策略
   
   [ policy_match ]
   countryName = match
   stateOrProvinceName = match
   organizationName = match
   organizationalUnitName = optional
   commonName = supplied
   emailAddress = optional
   
   [ req ]
   default_bits = 4096
   default_keyfile = myTestCertificateKey.pem    ## The default private key file name. 
                                                 ## 默认私钥文件名
   default_md = sha256                           ## Use SHA-256 for Signatures
                                                 ## 使用SHA-256签名
   distinguished_name = req_dn
   req_extensions = v3_req
   x509_extensions = v3_ca # The extentions to add to the self signed cert
   
   [ v3_req ]
   subjectKeyIdentifier  = hash
   basicConstraints = CA:FALSE
   keyUsage = critical, digitalSignature, keyEncipherment
   nsComment = "OpenSSL Generated Certificate for TESTING only.  NOT FOR PRODUCTION USE."
   extendedKeyUsage  = serverAuth, clientAuth
   
   [ req_dn ]
   countryName = Country Name (2 letter code)
   countryName_default =
   countryName_min = 2
   countryName_max = 2
   
   stateOrProvinceName = State or Province Name (full name)
   stateOrProvinceName_default = TestCertificateStateName
   stateOrProvinceName_max = 64
   
   localityName = Locality Name (eg, city)
   localityName_default = TestCertificateLocalityName
   localityName_max = 64
   
   organizationName = Organization Name (eg, company)
   organizationName_default = TestCertificateOrgName
   organizationName_max = 64
   
   organizationalUnitName = Organizational Unit Name (eg, section)
   organizationalUnitName_default = TestCertificateOrgUnitName
   organizationalUnitName_max = 64
   
   commonName = Common Name (eg, YOUR name)
   commonName_max = 64
   
   [ v3_ca ]
   # Extensions for a typical CA
   # 典型CA的扩展
   subjectKeyIdentifier=hash
   basicConstraints = critical,CA:true
   authorityKeyIdentifier=keyid:always,issuer:always
   ```
   2. 可选。您可以更新默认专有名称(DN)值。

### B. 生成测试 CA PEM 文件


1. 创建**测试** CA 密钥文件 `mongodb-test-ca.key`。
   复制

   ```
   openssl genrsa -out mongodb-test-ia.key 4096
   ```
   提示:

   此私钥用于为CA生成有效证书。尽管此私钥与本附录中的所有文件一样，仅用于**测试**目的，但您应遵循良好的安全惯例并保护此密钥文件。
2. `mongod-test-ca.crt`使用生成的密钥文件创建CA证书。当要求提供专有名称值时，为您的**测试**CA证书输入适当的值。
   复制

   ```
   openssl req -new -x509 -days 1826 -key mongodb-test-ca.key -out mongodb-test-ca.crt -config openssl-test-ca.cnf
   ```
3. 创建中间证书的私钥
   ```
   openssl genrsa -out mongodb-test-ia.key 4096
   ```
   提示: 

   此私钥用于为中间机构生成有效的证书。尽管此私钥与本附录中的所有文件一样，仅用于**测试**目的，但您应遵循良好的安全惯例并保护此密钥文件。
4. 为中间证书创建证书签名请求。当要求提供专有名称值时，请为您的**测试**中间机构证书输入适当的值。

   ```
   openssl req -new -key mongodb-test-ia.key -out mongodb-test-ia.csr -config openssl-test-ca.cnf
   ```
5. 创建中间证书`mongodb-test-ia.crt`。
   复制
   ```
   openssl x509 -sha256 -req -days 730 -in mongodb-test-ia.csr -CA mongodb-test-ca.crt -CAkey mongodb-test-ca.key -set_serial 01 -out mongodb-test-ia.crt -extfile openssl-test-ca.cnf -extensions v3_ca
   ```
6. 从测试的 CA 证书 `mongod-test-ca.crt` 和测试的中间证书 `mongodb-test-ia.crt` 创建测试的 CA PEM 文件。

   你可以使用测试的 PEM 文件为 TLS/SSL 测试配置[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)， [`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)或者[`mongo`](https://docs.mongodb.com/manual/reference/program/mongo/#bin.mongo)。
   您可以使用**测试**中间权限来为服务器和客户端签署**测试**证书。单个机构必须为客户端和服务器都颁发证书。

也可以看看

* [附录B-用于测试的OpenSSL服务器证书](https://docs.mongodb.com/manual/appendix/security/appendixB-openssl-server/#appendix-server-certificate)

- [附录C-用于测试的OpenSSL客户端证书](https://docs.mongodb.com/manual/appendix/security/appendixC-openssl-client/#appendix-client-certificate) 


译者：谢伟成

