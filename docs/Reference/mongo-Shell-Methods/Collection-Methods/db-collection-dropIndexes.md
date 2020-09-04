# [ ](#)db.collection.dropIndexes（）

[]()

在本页面

*   [定义](#definition)
*   [行为](#behavior)

## <span id="definition">定义</span>


*   `db.collection.`  `dropIndexes` ()

`_id`从集合中删除指定的一个或多个索引（字段中的索引除外 ）。

您可以使用该方法执行以下操作：

* `_id`从集合中删除除索引之外的所有内容。

  ```powershell
  db.collection.dropIndexes()
  ```

* 从集合中删除指定的索引。要指定索引，可以通过以下方法之一：

  * 索引规范文档（除非索引是 文本索引，在这种情况下，请使用索引名称删除）：

    ```powershell
    db.collection.dropIndexes( { a: 1, b: 1 } )
    ```

  * 索引名称：

    ```powershell
    db.collection.dropIndexes( "a_1_b_1" )
    ```
    > **建议**
    >
    > 若要获取索引的名称，请使用 `db.collection.getIndexes()`方法。

* 从集合中删除指定的索引。（从MongoDB 4.2开始可用）。要指定要删除的多个索引，请向该方法传递一个索引名称数组：

  ```powershell
  db.collection.dropIndexes( [ "a_1_b_1", "a_1", "a_1__id_-1" ] )
  ```

  如果索引名称数组包含不存在的索引，则该方法将出错，而不会删除任何指定的索引。
  
  > **建议**
  >
  > 若要获取索引的名称，请使用 `db.collection.getIndexes()`方法。

`db.collection.dropIndexes()`方法采用以下可选参数：

| 参数    | 类型                                   | 描述                                                         |
| ------- | -------------------------------------- | ------------------------------------------------------------ |
| indexes | string 或 document 或 array of strings | 可选的。指定要删除的一个或多个索引。<br />**要删除集合中除_id索引以外的所有索引**，请省略参数。<br />**要删除单个索引**，请指定索引名称，索引规范文档（除非索引是 文本索引）或索引名称的数组。要删除文本索引，请指定索引名称或索引名称的数组，而不是索引规范文档。<br />**要删除多个索引**（从MongoDB 4.2开始可用），请指定一个索引名称数组。 |

`db.collection.dropIndexes()`是围绕着一个包装 `dropIndexes`命令。

## <span id="behavior">行为</span>

### 只kill相关查询

从MongoDB 4.2开始，该`dropIndexes()`操作只会终止使用正在删除的索引的查询。这可能包括将索引视为查询计划一部分的 查询。

在MongoDB 4.2之前，在集合上删除索引将杀死该集合上所有打开的查询。

### 资源锁定

*在版本4.2中进行了更改。*

`db.collection.dropIndexes()`在操作期间获得对指定集合的排他锁。集合上的所有后续操作都必须等到`db.collection.dropIndexes()`释放锁为止。

在MongoDB 4.2之前的版本中，`db.collection.dropIndexes()`获得了对父数据库的排他锁，阻止了对数据库*及其*所有集合的所有操作，直到操作完成。

### 索引名称

如果给该方法传递了一个包含不存在的索引的索引名数组，则该方法将出错，而不会删除任何指定的索引。

### `_id`索引

您不能在`_id`字段上删除默认索引。

### 文本索引

要删除文本索引，请指定索引名称而不是索引规范文档。



译者：李冠飞

校对：