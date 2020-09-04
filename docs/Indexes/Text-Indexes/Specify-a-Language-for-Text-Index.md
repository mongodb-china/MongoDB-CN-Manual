# 为文本索引指定语言

**在本页面**

- [指定`text`索引的默认语言](#语言)
- [用多种语言为集合创建文本索引](#索引)

本教程描述了如何[指定与文本索引关联的默认语言](https://docs.mongodb.com/master/tutorial/specify-language-for-text-index/#specify-default-language-text-index)，以及如何为[包含不同语言文档的集合创建文本索引](https://docs.mongodb.com/master/tutorial/specify-language-for-text-index/#select-from-multiple-languages-for-text-index)。

## <span id="语言">指定`text`索引的默认语言</span>

与索引数据相关联的默认语言决定了解析词根(即：词干分析)和忽略停止词的规则。索引数据的默认语言是英语。

要指定不同的语言，请在创建文本索引时使用`default_language`选项。有关`default_language`可用的语言，请参阅[文本搜索语言](https://docs.mongodb.com/master/reference/text-search-languages/#text-search-languages)。

下面的示例为`quotes`集合在内容字段上创建了一个文本索引，并将`default_language`设置为西班牙语:

```powershell
db.quotes.createIndex(
   { content : "text" },
   { default_language: "spanish" }
)
```

## <span id="索引">用多种语言为集合创建文本索引</span>

### 指定文档内的索引语言

如果集合包含使用不同语言的文档或嵌入文档，则在文档或嵌入文档中包含名为`language`的字段，并将该文档或嵌入文档的语言指定为其值。

构建`text`索引时，MongoDB将为该文档或嵌入式文档使用指定的语言：

- 文档中指定的语言将覆盖`text`索引的默认语言。
- 嵌入式文档中的指定语言将覆盖附件文档中指定的语言或索引的默认语言。

有关支持的语言列表，请参见[文本搜索](https://docs.mongodb.com/master/reference/text-search-languages/#text-search-languages)语言。

例如，一个集合`quotes`包含多语言文档，根据需要包括`language`文档和/或嵌入文档中的字段：

```powershell
{
   _id: 1,
   language: "portuguese",
   original: "A sorte protege os audazes.",
   translation:
     [
        {
           language: "english",
           quote: "Fortune favors the bold."
        },
        {
           language: "spanish",
           quote: "La suerte protege a los audaces."
        }
    ]
}
{
   _id: 2,
   language: "spanish",
   original: "Nada hay más surrealista que la realidad.",
   translation:
      [
        {
          language: "english",
          quote: "There is nothing more surreal than reality."
        },
        {
          language: "french",
          quote: "Il n'y a rien de plus surréaliste que la réalité."
        }
      ]
}
{
   _id: 3,
   original: "is this a dagger which I see before me.",
   translation:
   {
      language: "spanish",
      quote: "Es este un puñal que veo delante de mí."
   }
}
```

如果您使用默认的英语语言在`quote`字段上创建了一个文本索引。

```powershell
db.quotes.createIndex( { original: "text", "translation.quote": "text" } )
```

然后，对于包含该`language` 字段的文档和嵌入文档，`text`索引使用该语言来解析词干和其他语言特征。

对于不包含该`language`字段的嵌入式文档，

- 如果封闭的文档包含该`language`字段，则索引将文档的语言用于嵌入式文档。
- 否则，索引将为嵌入文档使用默认语言。

对于不包含该`language`字段的文档，索引使用默认语言，即英语。

### 使用任何字段来指定文档的语言

要使用非语言名称的字段，请在创建索引时包含`language_override`选项。

例如，下面的命令使用`idioma`作为字段名而不是`language`:

```powershell
db.quotes.createIndex( { quote : "text" },
                       { language_override: "idioma" } )
```

`quotes`集合的文档可以在`idioma`字段中指定一种语言：

```powershell
{ _id: 1, idioma: "portuguese", quote: "A sorte protege os audazes" }
{ _id: 2, idioma: "spanish", quote: "Nada hay más surrealista que la realidad." }
{ _id: 3, idioma: "english", quote: "is this a dagger which I see before me" }
```



译者：杨帅