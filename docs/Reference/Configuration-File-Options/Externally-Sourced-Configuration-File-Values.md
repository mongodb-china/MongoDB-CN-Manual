# Externally Sourced Configuration File Values

On this page

- [Use the `__rest` Expansion Directive](https://docs.mongodb.com/master/reference/expansion-directives/#use-the-rest-expansion-directive)
- [Use the `__exec` Expansion Directive](https://docs.mongodb.com/master/reference/expansion-directives/#use-the-exec-expansion-directive)
- [Expansion Directives Reference](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-directives-reference)
- [Output the Configuration File with Resolved Expansion Directive Values](https://docs.mongodb.com/master/reference/expansion-directives/#output-the-configuration-file-with-resolved-expansion-directive-values)

*New in version 4.2.*

MongoDB supports using expansion directives in configuration files to load externally sourced values. Expansion directives can load values for specific [configuration file options](https://docs.mongodb.com/master/reference/configuration-options/#configuration-options) *or* load the entire configuration file. Expansion directives help obscure confidential information like security certificates and passwords.

copycopied

```
storage:
  dbPath: "/var/lib/mongo"
systemLog:
  destination: file
  path: "/var/log/mongodb/mongod.log"
net:
  bindIp:
    __exec: "python /home/user/getIPAddresses.py"
    type: "string"
    trim: "whitespace"
    digest: 85fed8997aac3f558e779625f2e51b4d142dff11184308dc6aca06cff26ee9ad
    digest_key: 68656c6c30303030307365637265746d796f6c64667269656e64
  tls:
    mode: requireTLS
    certificateKeyFile: "/etc/tls/mongod.pem"
    certificateKeyFilePassword:
      __rest: "https://myrestserver.example.net/api/config/myCertKeyFilePassword"
      type: "string"
      digest: b08519162ba332985ac18204851949611ef73835ec99067b85723e10113f5c26
      digest_key: 6d795365637265744b65795374756666
```

- If the configuration file includes the [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion, on Linux/macOS, the read access to the configuration file must be limited to the user running the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) process only.
- If the configuration file includes the [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) expansion, on Linux/macOS, the write access to the configuration file must be limited to the user running the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) process only.

To use expansion directives, you must specify the [`--configExpand`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) command-line option with the complete list of expansion directives used:

copycopied

```
mongod --config "/path/to/config/mongod.conf" --configExpand "rest,exec"
```

If you omit the [`--configExpand`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) option *or* if you do not specify the complete list of expansion directives used in the configuration file, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) returns an error and terminates. You can only specify the [`--configExpand`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) option on the command line.



## Use the `__rest` Expansion Directive

The [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion directive loads configuration file values from a `REST` endpoint. `__rest` supports loading specific values in the configuration file *or* loading the entire configuration file.

- Specific Value
- Full Configuration File

The following configuration file uses the [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion directive to load the setting [`net.tls.certificateKeyFilePassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFilePassword) value from an external `REST` endpoint:

copycopied

```
storage:
  dbPath: "/var/lib/mongo"
systemLog:
  destination: file
  path: "/var/log/mongodb/mongod.log"
net:
  bindIp: 192.51.100.24,127.0.0.1
  tls:
    mode: requireTLS
    certificateKeyFile: "/etc/tls/mongod.pem"
    certificateKeyFilePassword:
      __rest: "https://myrestserver.example.net/api/config/myCertKeyFilePassword"
      type: "string"
```

- File Permission

  If the configuration file includes the [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion, on Linux/macOS, the read access to the configuration file must be limited to the user running the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) process only.

- Expansion Parsing

  To parse the `__rest` blocks, start the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) with the [`--configExpand "rest"`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) option.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) issues a `GET` request against specified URL. If successful, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) replaces the value of `certificateKeyFilePassword` with the returned value. If the URL fails to resolve or if the `REST` endpoint returns an invalid value, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) throws an error and terminates.

IMPORTANT

The value returned by the specified `REST` endpoint **cannot** include any additional expansion directives. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) does **not** perform additional processing on the returned data and will terminate with an error code if the returned data includes additional expansion directives.



## Use the `__exec` Expansion Directive

The [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) expansion directive loads configuration file values from a shell or terminal command. `__exec` supports loading specific values in the configuration file *or* loading the entire configuration file.

- Specific Value
- Full Configuration File

The following example configuration file uses the [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) expansion directive to to load the setting [`net.tls.certificateKeyFilePassword`](https://docs.mongodb.com/master/reference/configuration-options/#net.tls.certificateKeyFilePassword) value from the output of a shell or terminal command:

copycopied

```
storage:
  dbPath: "/var/lib/mongo"
systemLog:
  destination: file
  path: "/var/log/mongodb/mongod.log"
net:
  bindIp: 192.51.100.24,127.0.0.1
  TLS:
    mode: requireTLS
    certificateKeyFile: "/etc/tls/mongod.pem"
    certificateKeyFilePassword:
      __exec: "python /home/myUserName/getPEMPassword.py"
      type: "string"
```

- File Permission

  If the configuration file includes the [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) expansion, on Linux/macOS, the write access to the configuration file must be limited to the user running the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) process only.

- Expansion Parsing

  To parse the `__exec` blocks, start the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) with the [`--configExpand "exec"`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) option.The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) attempts to execute the specified operation. If the command executes successfully, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) replaces the value of `certificateKeyFilePassword` with the returned value. If the command fails or returns an invalid value for the configuration file setting, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) throws an error and terminates.

IMPORTANT

The data returned by executing the specified `__exec` string **cannot** include any additional expansion directives. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) does **not** perform additional processing on the returned data and will terminate with an error code if the returned data includes additional expansion directives.



## Expansion Directives Reference

- `__rest`

  The [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion directive loads configuration file values from a `REST` endpoint. [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) supports loading specific values in the configuration file *or* loading the entire configuration file. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) then starts using the externally sourced values as part of its configuration.The [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion directive has the following syntax:To specify a `REST` endpoint for a specific configuration file setting or settings:copycopied`<some configuration file setting>:  __rest: "<string>"  type: "string"  trim: "none|whitespace"  digest: "<string>"  digest_key: "<string>" `To specify a `REST` endpoint for the entire configuration file:copycopied`__rest: "<string>" type: "yaml" trim: "none|whitespace" `If specifying the entire configuration file via `REST` endpoint, the expansion directive and its options **must** be the only values specified in the configuration file.[`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) takes the following fields:FieldTypeDescription[__rest](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-rest)string*Required* The URL against which the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) issues a `GET` request to retrieve the externally sourced value.For non-localhost `REST` endpoints (e.g. a `REST` endpoint hosted on a remote server), [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) requires encrypted (`https://`) URLs where both the host machine and the remote server support TLS 1.1 or later.If the `REST` endpoint specified in the URL requires authentication, encode credentials into the URL with the standard [RFC 3986 User Information](https://tools.ietf.org/html/rfc3986#section-3.2.1) format.For localhost `REST` endpoints (e.g. a `REST` endpoint listening on the host machine), [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) allows unencrypted (`http://`) URLs.IMPORTANTThe value returned by the specified `REST` endpoint **cannot** include any additional expansion directives. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) does **not** perform additional processing on the returned data and will terminate with an error code if the returned data includes additional expansion directives.`type`string*Optional* Controls how [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) parses the returned value from the specified URL.Possible values are:`string` (*Default*)Directs [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) to parse the returned data as a literal string. If specifying `string`, the entire [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) block and supporting options must be nested under the field for which you are loading externally sourced values.`yaml`Directs [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) to parse the returned data as a `yaml` formatted file. If specifying `yaml`, the [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) block must be the only content in the configuration file. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) replaces the configuration file contents with the `yaml` retrieved from the REST resource.`trim`string*Optional* Specify `whitespace` to direct [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) to trim any leading or trailing whitespace, specifically occurrences of `" "`, `"\r"`, `"\n"`, `"\t"`, `"\v"`, and `"\f"`. Defaults to `none`, or no trimming.[digest](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-rest-digest)string*Optional*. The SHA-256 digest of the expansion result.If specified, you must also specify the [digest_key](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-rest-digest-key).[digest_key](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-rest-digest-key)string*Optional*. The hexadecimal string representation of the secret used to calculate the SHA-256 [digest](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-rest-digest).If specified, you must also specify the [digest](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-rest-digest).NOTEIf the configuration file includes the [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion, on Linux/macOS, the read access to the configuration file must be limited to the user running the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) process only.To enable parsing of the `__rest` expansion directive, start the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) with the [`--configExpand "rest"`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) option.For examples, see [Use the __rest Expansion Directive](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-directive-rest).

- `__exec`

  The [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) expansion directive loads configuration file values from the output of a shell or terminal command. [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) supports loading specific values in the configuration file *or* loading the entire configuration file. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) then starts using the externally sourced values as part of its configuration.The [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) expansion directive has the following syntax:To specify a shell or terminal command for a specific configuration file setting or settings:copycopied`<some configuration file setting>:  __exec: "<string>"  type: "string"  trim: "none|whitespace" `To specify a a shell or terminal command for the entire configuration file:copycopied`__exec: "<string>" type: "yaml" trim: "none|whitespace" `If specifying the entire configuration file via a terminal or shell command, the expansion directive and its options **must** be the only values specified in the configuration file.[`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) takes the following fields:FieldTypeDescription`__exec`string*Required* The string which the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) executes on the terminal or shell to retrieve the externally sourced value.On Linux and OSX hosts, execution is handled via POSIX `popen()`. On Windows hosts, execution is handled via the process control API. `__exec` opens a read-only pipe as the same user that started the `mongod` or `mongos`.IMPORTANTThe data returned by executing the specified command **cannot** include any additional expansion directives. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) does **not** perform additional processing on the returned data and will terminate with an error code if the returned data includes additional expansion directives.`type`string*Optional* Controls how [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) parses the value returned by the executed command.Possible values are:`string` (*Default* )Directs [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) to parse the returned data as a literal string. If specifying `string`, the entire [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) block and supporting options must be nested under the field for which you are loading externally sourced values.`yaml`Directs [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) to parse the returned data as a `yaml` formatted file. If specifying `yaml`, the [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) block must be the only content in the configuration file. The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) replaces the configuration file contents with the `yaml` retrieved from the executed command.`trim`string*Optional* Specify `whitespace` to direct [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) to trim any leading or trailing whitespace, specifically occurrences of `" "`, `"\r"`, `"\n"`, `"\t"`, `"\v"`, and `"\f"`. Defaults to `none`, or no trimming.[digest](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-exec-digest)string*Optional*. The SHA-256 digest of the expansion result.If specified, you must also specify the [digest_key](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-exec-digest-key)[digest_key](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-exec-digest-key)string*Optional*. The hexadecimal string representation of the secret used to calculate the SHA-256 [digest](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-exec-digest).If specified, you must also specify the [digest](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-exec-digest)NOTEIf the configuration file includes the [`__exec`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__exec) expansion, on Linux/macOS, the write access to the configuration file must be limited to the user running the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) process only.To enable parsing of the `__exec` expansion directives, start the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) with the [`--configExpand "exec"`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) option.For examples, see [Use the __exec Expansion Directive](https://docs.mongodb.com/master/reference/expansion-directives/#expansion-directive-exec).



## Output the Configuration File with Resolved Expansion Directive Values

You can test the final output of a configuration file that specifies one or more expansion directives by starting the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) with the [`--outputConfig`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-outputconfig) option. A [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) started with [`--outputConfig`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-outputconfig) outputs the resolved YAML configuration document to `stdout` and halts. If any expansion directive specified in the configuration file returns additional expansion directives, the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) throws an error and terminates.

WARNING

The [`--outputConfig`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-outputconfig) option returns the resolved values for any field using an expansion directive. This includes any private or sensitive information previously obscured by using an external source for the configuration option.

For example, the following configuration file `mongod.conf` contains a [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion directive:

copycopied

```
storage:
  dbPath: "/var/lib/mongo"
systemLog:
  destination: file
  path: "/var/log/mongodb/mongod.log"
net:
  port:
    __rest: "https://mongoconf.example.net:8080/record/1"
    type: string
```

The string recorded at the specified URL is `20128`

If the configuration file includes the [`__rest`](https://docs.mongodb.com/master/reference/expansion-directives/#configexpansion.__rest) expansion, on Linux/macOS, the read access to the configuration file must be limited to the user running the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod)/[`mongos`](https://docs.mongodb.com/master/reference/program/mongos/#bin.mongos) process only.

Start the [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) with the [`--configExpand "rest"`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-configexpand) and [`--outputConfig`](https://docs.mongodb.com/master/reference/program/mongod/#cmdoption-mongod-outputconfig) options:

copycopied

```
mongod -f mongod.conf --configExpand rest --outputConfig
```

The [`mongod`](https://docs.mongodb.com/master/reference/program/mongod/#bin.mongod) outputs the following to `stdout` before terminating:

copycopied

```
config: mongod.conf
storage:
  dbPath: "/var/lib/mongo"
systemLog:
  destination: file
  path: "/var/log/mongodb/mongod.log"
net:
  port: 20128
outputConfig: true
```