## 默认的MongoDB端口

下表列出了MongoDB使用的默认TCP端口:

| Default Port | Description                                                  |
| ------------ | ------------------------------------------------------------ |
| `27017`      | [` mongod `](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)和[` mongos `](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos)实例的默认端口。您可以使用[` port `](https://docs.mongodb.com/master/reference/configuring-options/#net.port)或[`——port `](https://docs.mongodb.com/master/reference/program/mongod/# cmdop-mongod -port)来更改该端口。 |
| `27018`      | 默认端口为[` mongod `](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)运行[**——shardsvr**](https://docs.mongodb.com/master/reference/program/mongod/ # cmdoption-mongod-shardsvr)命令行选项或 **shardsvr** 值[**clusterRole**](https://docs.mongodb.com/master/reference/configuration-options/ # sharding.clusterRole)设置在配置文件中。 |
| `27019`      | 默认端口为[`mongod `](https://docs.mongodb.com/master/reference/program/mongod/ bin.mongod)运行[**——configsvr**](https://docs.mongodb.com/master/reference/program/mongod/ # cmdoption-mongod-configsvr)命令行选项或 **configsvr** 值[**clusterRole**](https://docs.mongodb.com/master/reference/configuration-options/ # sharding.clusterRole)设置在配置文件中。 |

