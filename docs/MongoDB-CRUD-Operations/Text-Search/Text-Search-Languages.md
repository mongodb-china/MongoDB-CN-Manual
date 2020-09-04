
# 文本搜索语言
[文本索引](https://docs.mongodb.com/master/core/index-text/#index-feature-text) 和[`$text`](https://docs.mongodb.com/master/reference/operator/query/text/#op._S_text) 运算符可用于下列语言，并接受两个字母的ISO 639-1语言代码或语言名称的长形式:

| 语言名称     | ISO 639-1(双字母代码) |
| :----------- | :-------------------- |
| `danish`     | `da`                  |
| `dutch`      | `nl`                  |
| `english`    | `en`                  |
| `finnish`    | `fi`                  |
| `french`     | `fr`                  |
| `german`     | `de`                  |
| `hungarian`  | `hu`                  |
| `italian`    | `it`                  |
| `norwegian`  | `nb`                  |
| `portuguese` | `pt`                  |
| `romanian`   | `ro`                  |
| `russian`    | `ru`                  |
| `spanish`    | `es`                  |
| `swedish`    | `sv`                  |
| `turkish`    | `tr`                  |

> **[success] Note**
>
> 如果指定语言值为**“none”**，则文本搜索使用简单的标记化，不包含停止词列表和词干分析。

另看：

[Specify a Language for Text Index](https://docs.mongodb.com/manual/tutorial/specify-language-for-text-index/)


译者：杨帅

校对：杨帅