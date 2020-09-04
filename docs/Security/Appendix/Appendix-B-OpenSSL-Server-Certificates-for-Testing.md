# 附录 B-用于测试的OpenSSL服务器证书


免责声明


此页面仅用于**测试目的**；证书仅用于**测试目的**。


以下教程提供了创建**测试**x.509证书的一些基本步骤：


- 请勿将这些证书用于生产环境。相反，请遵循您的安全策略。
- 有关OpenSSL的信息，请参考官方的OpenSSL文档。尽管本教程使用的是OpenSSL，但不应将本材料当作OpenSSL的权威参考。


## 前提条件


本页所描述的过程会使用**测试**的中间权限证书以及在[附录A - 用于测试的OpenSSL CA证书](https://docs.mongodb.com/manual/appendix/security/appendixA-openssl-ca/)中创建的秘钥  `mongodb-test-ia.crt` 和 `mongodb-test-ia.key` 。

## 过程[¶](https://docs.mongodb.com/manual/appendix/security/appendixB-openssl-server/#procedure)


以下过程概述了为MongoDB服务器创建**测试**证书的步骤。有关为MongoDB客户端创建**测试**证书的步骤，请参阅[附录C - 用于测试的OpenSSL客户端证书](https://docs.mongodb.com/manual/appendix/security/appendixC-openssl-client/)。


### A. 创建OpenSSL配置文件


1. 使用以下内容为您的服务器创建一个**测试**配置文件`openssl-test-server.cnf`： 

   ```
   # NOT FOR PRODUCTION USE. OpenSSL configuration file for testing.
   [ req ]
   default_bits = 4096
   default_keyfile = myTestServerCertificateKey.pem    ## The default private key file name.
   default_md = sha256
   distinguished_name = req_dn
   req_extensions = v3_req
   
   [ v3_req ]
   subjectKeyIdentifier  = hash
   basicConstraints = CA:FALSE
   keyUsage = critical, digitalSignature, keyEncipherment
   nsComment = "OpenSSL Generated Certificate for TESTING only.  NOT FOR PRODUCTION USE."
   extendedKeyUsage  = serverAuth, clientAuth
   subjectAltName = @alt_names

   [ alt_names ]
   DNS.1 =         ##TODO: Enter the DNS names. The DNS names should match the server names.
   DNS.2 =         ##TODO: Enter the DNS names. The DNS names should match the server names.
   IP.1 =          ##TODO: Enter the IP address. SAN matching by IP address is available starting in MongoDB 4.2
   IP.2 =          ##TODO: Enter the IP address. SAN matching by IP address is available starting in MongoDB 4.2
   
   [ req_dn ]
   countryName = Country Name (2 letter code)
   countryName_default = TestServerCertificateCountry
   countryName_min = 2
   countryName_max = 2
   
   stateOrProvinceName = State or Province Name (full name)
   stateOrProvinceName_default = TestServerCertificateState
   stateOrProvinceName_max = 64
   
   localityName = Locality Name (eg, city)
   localityName_default = TestServerCertificateLocality
   localityName_max = 64
   
   organizationName = Organization Name (eg, company)
   organizationName_default = TestServerCertificateOrg
   organizationName_max = 64
   
   organizationalUnitName = Organizational Unit Name (eg, section)
   organizationalUnitName_default = TestServerCertificateOrgUnit
   organizationalUnitName_max = 64
   
   commonName = Common Name (eg, YOUR name)
   commonName_max = 64
   ```
2. 在该`[alt_names]`部分中，输入适合MongoDB服务器的DNS名称和/或IP地址。您可以为MongoDB服务器指定多个DNS名称。
   对于OpenSSL SAN标识符，MongoDB支持：
   - DNS名称和/或
   - IP地址字段（从MongoDB 4.2开始)
3. *可选*。你可以更新默认的专有名称(DN)值


提示
  - 为至少一个下列属性指定一个非空值：组织 (`O`)、组织单元 (`OU`)或者域组件 (`DC`)
  - 为内部成员身份验证创建**测试**服务器证书，如果指定了下面的属性，则在成员证书之间必须完全匹配：组织 (`O`)、组织单元 (`OU`)、域组件 (`DC`)。
  有关内部成员身份验证要求的更多信息，请查阅[成员身份验证](https://docs.mongodb.com/manual/core/security-internal-authentication/#internal-auth-x509)。
### B. 为服务器生成测试PEM文件[¶](https://docs.mongodb.com/manual/appendix/security/appendixB-openssl-server/#b-generate-the-test-pem-file-for-server)


重要

在继续之前，请确保在配置文件`openssl-test-server.cnf`中的`[alt_names]`部分输入了适当的DNS名称。
1. 创建**测试**密钥文件`mongodb-test-server1.key`。
```
   openssl genrsa -out mongodb-test-server1.key 4096
```
2. 创建测试的证书签名请求`mongodb-test-server1.csr`。

   当要求提供专有名称值时，为您的测试证书输入适当的值：

   - 为以下属性中的至少一个指定一个非空值：组织(`O`)、组织单位(`OU`)或域组件(`DC`)。
   - 为内部成员身份验证创建**测试**服务器证书时，如果指定了以下属性，则这些属性必须在成员证书之间完全匹配：组织(`O`)、组织单位(`OU`)、域组件(`DC`)。
   
   复制
   ```
   openssl req -new -key mongodb-test-server1.key -out mongodb-test-server1.csr -config openssl-test-server.cnf
   ```


3. 创建**测试**服务器证书`mongodb-test-server1.crt`。

   ```
   openssl x509 -sha256 -req -days 365 -in mongodb-test-server1.csr -CA mongodb-test-ia.crt -CAkey mongodb-test-ia.key -CAcreateserial -out mongodb-test-server1.crt -extfile openssl-test-server.cnf -extensions v3_req
   ```

4. 为服务器创建**测试**PEM文件。
   
   ```
   cat mongodb-test-server1.crt mongodb-test-server1.key > test-server1.pem
   ```
   
   
   你可以使用**test**PEM文件为TLS/SSL**测试**配置一个[`mongod`](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)或一个[`mongos`](https://docs.mongodb.com/manual/reference/program/mongos/#bin.mongos)。例如：


**对于MongDB 4.2或更高版本**

```
mongod --tlsMode requireTLS --tlsCertificateKeyFile test-server1.pem  --tlsCAFile test-ca.pem
```


虽然仍然可以使用，但[`--sslMode`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-sslmode)、[`--sslPEMKeyFile`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-sslpemkeyfile)和[`--sslCAFile`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-sslcafile)在[MongoDB 4.2中已废弃](https://docs.mongodb.com/manual/release-notes/4.2/#tls)。


**对于MongoDB 4.0及更早的版本**

```
mongod --sslMode requireSSL --sslPEMKeyFile test-server1.pem  --sslCAFile test-ca.pem
```


### 在macOS系统中


如果你使用Keychain Access管理证书，创建一个pkcs-12而不是PEM文件添加到Keychain Access中。

```
openssl pkcs12 -export -out test-client.pfx -inkey mongodb-test-client.key -in mongodb-test-client.crt -certfile mongodb-test-ia.crt
```


将其添加到Keychain Access后，您无需指定证书密钥文件，就可以使用[`--tlsCertificateSelector`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-tlscertificateselector)来指定要使用的证书。如果CA文件也在Keychain Access中，也可省略`--tlsCAFile`。


**对于MongoDB 4.2或者更高版本** 

```
mongo --tls --tlsCertificateSelector subject="<TestClientCertificateCommonName>"
```


虽然仍然可以使用，[`--sslMode`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-sslmode)和[`--sslCertificateSelector`](https://docs.mongodb.com/manual/reference/program/mongod/#cmdoption-mongod-sslcertificateselector)在[MongoDB 4.2中已废弃](https://docs.mongodb.com/manual/release-notes/4.2/#tls)。


**对于MongoDB 4.0及更早版本**

```
mongo --ssl --sslCertificateSelector subject="<TestClientCertificateCommonName>"
```

 
要向Keychain Access添加证书，请参阅Keychain Access的官方文档。

另请参阅

- [附录A - 用于测试的OpenSSL CA证书](https://docs.mongodb.com/manual/appendix/security/appendixA-openssl-ca/#appendix-ca-certificate)
- [附录C - 用于测试的OpenSSL客户端证书](https://docs.mongodb.com/manual/appendix/security/appendixC-openssl-client/#appendix-client-certificate)
- [成员的x.509证书](https://docs.mongodb.com/manual/tutorial/configure-x509-member-authentication/#x509-member-certificate)


译者：谢伟成

