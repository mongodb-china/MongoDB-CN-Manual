# [ ](#)聚合表达式中的变量

[]()

在本页面

*   [用户变量](#user-variables)

*   [系统变量](#system-variables)

[聚合表达式](meta-aggregation-quick-reference.html#aggregation-expressions)可以同时使用 user-defined 和系统变量。

变量可以容纳任何[BSON 类型数据](reference-bson-types.html)。要访问变量的 value，请使用带有前缀为 double 美元符号(`$$`)的变量 name 的 string。

如果变量 references 一个 object，要访问 object 中的特定字段，请使用点表示法; 即： `"$$<variable>.<field>"`。

[]()

[]()

## <span id="user-variables">用户变量</span>

用户变量名称可以包含 ascii 字符`[_a-zA-Z0-9]`和任何 non-ascii 字符。

用户变量名必须以小写的 ascii 字母`[a-z]`或 non-ascii 字符开头。

[]()

[]()

## <span id="system-variables">系统变量</span>

MongoDB 提供以下系统变量：

| 变量      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| `ROOT`    | References 根文档，即： top-level 文档，当前正在聚合管道阶段中处理。 |
| `CURRENT` | Reference 聚合管道阶段中正在处理的字段路径的开始。除非另有说明，否则所有阶段都以[CURRENT]()开头，与[ROOT]()相同。 <br/> [CURRENT]()可以修改。但是，由于`$<field>`等同于`$$CURRENT.<field>`，因此重新绑定[CURRENT]()会改变`$`访问的含义。 |
| `REMOVE`  | 一个变量，用于计算缺少的 value。允许条件排除字段。在`$projection`中，从输出中排除设置为变量[REMOVE]()的字段。 <br/>有关其用法的示例，请参阅[有条件地排除字段]()。 <br/> version 3.6 中的新内容。 |
| `DESCEND` | [$redact]()表达式的允许结果之一。                            |
| `PRUNE`   | [$redact]()表达式的允许结果之一。                            |
| `KEEP`    | [$redact]()表达式的允许结果之一。                            |

> **也可以看看**
>
> [$let]()，[$redact]()，[$map]()



译者：李冠飞

校对：