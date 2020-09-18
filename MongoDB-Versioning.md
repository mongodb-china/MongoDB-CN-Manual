发行说明 > MongoDB 版本管理

# MongoDB Versioning[¶](https://docs.mongodb.com/v4.2/reference/versioning/#mongodb-versioning)MongoDB 版本管理

IMPORTANT 重要提示

Always upgrade to the latest stable revision of your release series.

始终升级到所发布系列的最新稳定版本。

MongoDB versioning has the form `X.Y.Z` where `X.Y` refers to either a release series or development series and `Z` refers to the revision/patch number.

MongoDB的版本管理按照`X.Y.Z`的形式，其中`X.Y`是发行版本序列号或者开发版本序列号，`Z`是版本号或者修订号。

- If `Y` is even, `X.Y` refers to a release series; for example, `4.0` release series and `4.2` release series. Release series are **stable** and suitable for production.
- 如果`Y`是偶数，则`X.Y`为发行版本序号；例如，`4.0`是一个发行版本序列号，`4.2`也是一个发行版本序列号。发行版本通常比较稳定，可用于生产环境。
- If `Y` is odd, `X.Y` refers to a development series; for example, `4.1` development series and `4.3` development series. Development series are **for testing only and not for production**.
- 如果`Y`是奇数, 则`X.Y`为开发版本；例如，`4.1`是一个开发版本序列号，`4.3`也是一个开发版本序列号。开发版本应该**仅用于测试，不能用于生产环境**。

For example, in MongoDB version `4.0.12`, `4.0` refers to the release series and `.12` refers to the revision.

例如，MongoDB版本号`4.0.12`，`4.0`是发行版本序列号，`.12`是此发行版本的修订号。

## New Releases新版本

Changes in the release series (e.g. `4.0` to `4.2`) generally mark the introduction of new features that may break backwards compatibility.

发行版本系列号的改变（如`4.0`变成`4.2`）通常标志着新的特性引入，这些新特性通常无法向后兼容。

## Patch Releases补丁发布

Changes to the revision number (e.g. `4.0.11` to `4.0.12`) generally mark the release of bug fixes and backwards-compatible changes.

修订号的改变（例如`4.0.11`到`4.0.12`）通常标志着bug的修复，并且可以向后兼容。

## Driver Versions驱动程序版本

The version numbering system for MongoDB differs from the system used for the MongoDB drivers.

MongoDB的版本编号系统与用于MongoDB驱动程序的版本编号系统不同。

←  [Release Notes for MongoDB 1.2.x](https://docs.mongodb.com/v4.2/release-notes/1.2/)[Technical Support](https://docs.mongodb.com/v4.2/support/) →

←  [MongoDB 1.2.x 发行版本说明](https://docs.mongodb.com/v4.2/release-notes/1.2/)[技术支持](https://docs.mongodb.com/v4.2/support/) →



https://docs.mongodb.com/v4.2/reference/versioning/

译者: 罗治军 

