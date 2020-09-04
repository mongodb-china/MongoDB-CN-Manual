# Convert Command-Line Options to YAML

Starting in MongoDB 4.2, [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) and [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) accept `--outputConfig` command-line option to output the configuration used by the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance.

You can use this option to convert command-line options to YAML configuration.

## Examples

### Convert `mongod` Command-Line Options to YAML

Consider the following [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) invocation that uses the command-line options:

```
mongod --shardsvr --replSet myShard  --dbpath /var/lib/mongodb --bind_ip localhost,My-Example-Hostname --fork --logpath /var/log/mongodb/mongod.log --clusterAuthMode x509 --tlsMode requireTLS  --tlsCAFile /path/to/my/CA/file  --tlsCertificateKeyFile /path/to/my/certificate/file --tlsClusterFile /path/to/my/cluster/membership/file
```

Include the [`--outputConfig`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-outputconfig) command-line option to generate the corresponding YAML file.

copycopied

```
mongod --shardsvr --replSet myShard  --dbpath /var/lib/mongodb --bind_ip localhost,My-Example-Hostname --fork --logpath /var/log/mongodb/mongod.log --clusterAuthMode x509 --tlsMode requireTLS  --tlsCAFile /path/to/my/CA/file  --tlsCertificateKeyFile /path/to/my/certificate/file --tlsClusterFile /path/to/my/cluster/membership/file --outputConfig
```

The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) outputs the following YAML to `stdout` and exits:

copycopied

```
net:
  bindIp: localhost,My-Example-Hostname
  tls:
    CAFile: /path/to/my/CA/file
    certificateKeyFile: /path/to/my/certificate/file
    clusterFile: /path/to/my/cluster/membership/file
    mode: requireTLS
outputConfig: true
processManagement:
  fork: true
replication:
  replSet: myShard
security:
  clusterAuthMode: x509
sharding:
  clusterRole: shardsvr
storage:
  dbPath: /var/lib/mongodb
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
```

To create a configuration file, copy the generated content into a file and delete the `outputConfig` setting from the YAML.

### Convert `mongos` Command-Line Options to YAML

Consider the following [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) invocation that uses the command-line options:

```
mongos --configdb myCSRS/cfg1.example.net:27019,cfg2.example.net:27019 --bind_ip localhost,My-Example-MONGOS-Hostname --fork --logpath /var/log/mongodb/mongos.log --clusterAuthMode x509 --tlsMode requireTLS  --tlsCAFile /path/to/my/CA/file  --tlsCertificateKeyFile /path/to/my/certificate/file --tlsClusterFile /path/to/my/cluster/membership/file
```

Include the [`--outputConfig`](https://docs.mongodb.com/master/reference/program/mongos/#cmdoption-mongos-outputconfig) command-line option to generate the corresponding YAML for the [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) instance:

copycopied

```
mongos --configdb myCSRS/cfg1.example.net:27019,cfg2.example.net:27019 --bind_ip localhost,My-Example-MONGOS-Hostname --fork --logpath /var/log/mongodb/mongos.log --clusterAuthMode x509 --tlsMode requireTLS  --tlsCAFile /path/to/my/CA/file  --tlsCertificateKeyFile /path/to/my/certificate/file --tlsClusterFile /path/to/my/cluster/membership/file --outputConfig
```

The [`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) outputs the following YAML to `stdout` and exits:

copycopied

```
net:
  bindIp: localhost,My-Example-MONGOS-Hostname
  tls:
    CAFile: /path/to/my/CA/file
    certificateKeyFile: /path/to/my/certificate/file
    clusterFile: /path/to/my/cluster/membership/file
    mode: requireTLS
outputConfig: true
processManagement:
  fork: true
security:
  clusterAuthMode: x509
sharding:
  configDB: myCSRS/cfg1.example.net:27019,cfg2.example.net:27019
systemLog:
  destination: file
  path: /var/log/mongodb/mongos.log
```

To create a configuration file, copy the generated content into a file and delete the `outputConfig` setting from the YAML.