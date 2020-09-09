# db.collection.drop\(\)

在本页面

* [定义](db-collection-drop.md#definition)
* [行为](db-collection-drop.md#behavior)
* [例子](db-collection-drop.md#examples)

## 定义

* `db.collection.drop()`
  * 从数据库中删除集合或视图。该方法还会删除与已删除集合关联的所有索引。该方法在下降命令周围提供 wrapper。

db.collection.drop\(\)的形式如下：

_在版本4.0中更改：_`db.collection.drop()`接受选项文档。

```text
db.collection.drop()
```

`db.collection.drop()`接受具有以下字段的可选文档：

| 字段 | 描述 |
| :--- | :--- |
| writeConcern | 可选的。表示操作的写关注点的 文档`db.collection.drop()`。省略使用默认的写关注。 当分片群集上发出，`mongos`转换 写入关注的的 `drop`命令及其助手 `db.collection.drop()`来`"majority"`。 _版本4.0中的新功能。_ |

|  |  |
| :--- | :--- |
| 返回： | `true`成功删除集合时。   `false`当不存在要收集的集合时。 |

## 行为

* 该`db.collection.drop()`方法和`drop`命令为在删除的集合上打开的任何 变更流创建一个无效事件。
* 从MongoDB 4.0.2开始，删除集合将删除其关联的区域/标签范围。

### 资源锁定

_在版本4.2中进行了更改。_

`db.collection.drop()`在操作期间获得对指定集合的排他锁。集合上的所有后续操作都必须等到`db.collection.drop()`释放锁为止。

在MongoDB 4.2之前的版本中，`db.collection.drop()`获得了对父数据库的排他锁，阻止了对数据库_及其_所有集合的所有操作，直到操作完成。

## 例子

### 使用默认写入问题删除集合

以下操作将`students`集合拖放到当前数据库中。

```text
db.students.drop()
```

### 使用Write Concern 删除一个集合`w: "majority"`

_在版本4.0中更改：_`db.collection.drop()`接受选项文档。

以下操作将`students`集合拖放到当前数据库中。该操作使用`"majority"`写关注点：

```text
db.students.drop( { writeConcern: { w: "majority" } } )
```

译者：李冠飞

校对：

