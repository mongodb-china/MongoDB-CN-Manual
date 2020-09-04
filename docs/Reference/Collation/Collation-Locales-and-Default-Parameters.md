## 排序区域和默认参数

**在本页面**

- [行为](#行为)
- [支持的语言和语言环境](#支持)
- [排序默认参数](#参数)

*3.4版本新增.*

[Collation](https://docs.mongodb.com/master/reference/collation/)允许用户为字符串比较指定特定于语言的规则，比如字母大小写和重音符号的规则。

### 行为

一些排序区域有变体，它们使用特定于语言的规则。要指定语言环境变量，请使用以下语法:

```powershell
{ "locale" : "<locale code>@collation=<variant>" }
```

例如，使用中文排序的`unihan`变体:

```powershell
{ "locale" : "zh@collation=unihan" }
```

查看[排序规则页面](https://docs.mongodb.com/master/reference/collation/)获取排序行为和语法的完整描述。

### <span id="支持">支持的语言和语言环境</span>

MongoDB的[排序特性](https://docs.mongodb.com/master/reference/collation/)支持以下语言。下表列出了[ICU语言环境ID](http://userguide.icu-project.org/locale)所定义的受支持的语言和相关的语言环境。

| 语言                              | Locale        | Variants                             |
| :-------------------------------- | :------------ | :----------------------------------- |
| Afrikaans                         | `af`          |                                      |
| Albanian                          | `sq`          |                                      |
| Amharic                           | `am`          |                                      |
| Arabic                            | `ar`          | `compat`                             |
| Armenian                          | `hy`          |                                      |
| Assamese                          | `as`          |                                      |
| Azeri                             | `az`          | `search`                             |
| Bengali                           | `bn`          |                                      |
| Belarusian                        | `be`          |                                      |
| Bengali                           | `bn`          | `traditional`                        |
| Bosnian                           | `bs`          | `search`                             |
| Bosnian (Cyrillic)                | `bs_Cyrl`     |                                      |
| Bulgarian                         | `bg`          |                                      |
| Burmese                           | `my`          |                                      |
| Catalan                           | `ca`          | `search`                             |
| Cherokee                          | `chr`         |                                      |
| Chinese                           | `zh`          | `big5han``gb2312han``unihan``zhuyin` |
| Chinese (Traditional)             | `zh_Hant`     |                                      |
| Croatian                          | `hr`          | `search`                             |
| Czech                             | `cs`          | `search`                             |
| Danish                            | `da`          | `search`                             |
| Dutch                             | `nl`          |                                      |
| Dzongkha                          | `dz`          |                                      |
| English                           | `en`          |                                      |
| English (United States)           | `en_US`       |                                      |
| English (United States, Computer) | `en_US_POSIX` |                                      |
| Esperanto                         | `eo`          |                                      |
| Estonian                          | `et`          |                                      |
| Ewe                               | `ee`          |                                      |
| Faroese                           | `fo`          |                                      |
| Filipino                          | `fil`         |                                      |
| Finnish                           | `fi`          | `search``traditional`                |
| French                            | `fr`          |                                      |
| French (Canada)                   | `fr_CA`       |                                      |
| Galician                          | `gl`          | `search`                             |
| Georgian                          | `ka`          |                                      |
| German                            | `de`          | `search``eor``phonebook`             |
| German (Austria)                  | `de_AT`       | `phonebook`                          |
| Greek                             | `el`          |                                      |
| Gujarati                          | `gu`          |                                      |
| Hausa                             | `ha`          |                                      |
| Hawaiian                          | `haw`         |                                      |
| Hebrew                            | `he`          | `search`                             |
| Hindi                             | `hi`          |                                      |
| Hungarian                         | `hu`          |                                      |
| Icelandic                         | `is`          | `search`                             |
| Igbo                              | `ig`          |                                      |
| Inari Sami                        | `smn`         | `search`                             |
| Indonesian                        | `id`          |                                      |
| Irish                             | `ga`          |                                      |
| Italian                           | `it`          |                                      |
| Japanese                          | `ja`          | `unihan`                             |

| Language              | Locale    | Variants                   |
| :-------------------- | :-------- | :------------------------- |
| Kalaallisut           | `kl`      | `search`                   |
| Kannada               | `kn`      | `traditional`              |
| Kazakh                | `kk`      |                            |
| Khmer                 | `km`      |                            |
| Konkani               | `kok`     |                            |
| Korean                | `ko`      | `search``searchjl``unihan` |
| Kyrgyz                | `ky`      |                            |
| Lakota                | `lkt`     |                            |
| Lao                   | `lo`      |                            |
| Latvian               | `lv`      |                            |
| Lingala               | `ln`      | `phonetic`                 |
| Lithuanian            | `lt`      |                            |
| Lower Sorbian         | `dsb`     |                            |
| Luxembourgish         | `lb`      |                            |
| Macedonian            | `mk`      |                            |
| Malay                 | `ms`      |                            |
| Malayalam             | `ml`      |                            |
| Maltese               | `mt`      |                            |
| Marathi               | `mr`      |                            |
| Mongolian             | `mn`      |                            |
| Nepali                | `ne`      |                            |
| Northern Sami         | `se`      | `search`                   |
| Norwegian Bokmål      | `nb`      | `search`                   |
| Norwegian Nynorsk     | `nn`      | `search`                   |
| Oriya                 | `or`      |                            |
| Oromo                 | `om`      |                            |
| Pashto                | `ps`      |                            |
| Persian               | `fa`      |                            |
| Persian (Afghanistan) | `fa_AF`   |                            |
| Polish                | `pl`      |                            |
| Portuguese            | `pt`      |                            |
| Punjabi               | `pa`      |                            |
| Romanian              | `ro`      |                            |
| Russian               | `ru`      |                            |
| Serbian               | `sr`      |                            |
| Serbian (Latin)       | `sr_Latn` | `search`                   |
| Sinhala               | `si`      | `dictionary`               |
| Slovak                | `sk`      | `search`                   |
| Slovenian             | `sl`      |                            |
| Spanish               | `es`      | `search``traditional`      |
| Swahili               | `sw`      |                            |
| Swedish               | `sv`      | `search`                   |
| Tamil                 | `ta`      |                            |
| Telugu                | `te`      |                            |
| Thai                  | `th`      |                            |
| Tibetan               | `bo`      |                            |
| Tongan                | `to`      |                            |
| Turkish               | `tr`      | `search`                   |
| Ukrainian             | `uk`      |                            |
| Upper Sorbian         | `hsb`     |                            |
| Urdu                  | `ur`      |                            |
| Uyghur                | `ug`      |                            |
| Vietnamese            | `vi`      | `traditional`              |
| Walser                | `wae`     |                            |
| Welsh                 | `cy`      |                            |
| Yiddish               | `yi`      | `search`                   |
| Yoruba                | `yo`      |                            |
| Zulu                  | `zu`      |                            |

> 提示
>
> 要显式指定简单二进制比较，请将`locale`值指定为`simple`。

### <span id="参数">排序默认参数</span>

除了必需的**locale**参数外，一个排序文档还包含几个[可选参数](https://docs.mongodb.com/master/reference/collation/#coll-document-fields)。根据您使用的“locale”，默认参数可能会有所不同。查看[排序页面](https://docs.mongodb.com/master/reference/collation/)获取排序语法的完整描述。

以下默认参数在所有地区都是一致的:

- `caseLevel : false`
- `strength : 3`
- `numericOrdering : false`
- `maxVariable : punct`

下表显示了默认的排序规则参数，这些参数可能会因不同的地区而不同:

| 语言环境                    | caseFirst | 替换            | normalization | backwards |
| :-------------------------- | :-------- | :-------------- | :------------ | :-------- |
| `af`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sq`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `am`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ar`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ar@collation=compat`       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `hy`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `as`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `az`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `az@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `be`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `bn`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `bn@collation=traditional`  | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `bs`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `bs@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `bs_Cyrl`                   | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `bg`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `my`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ca`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ca@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `chr`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `zh`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `zh@collation=big5han`      | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `zh@collation=gb2312han`    | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `zh@collation=unihan`       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `zh@collation=zhuyin`       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `zh_Hant`                   | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `hr`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `hr@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `cs`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `cs@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `da`                        | `upper`   | `non-ignorable` | `FALSE`       | `FALSE`   |
| `da@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `nl`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `dz`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `en`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `en_US_POSIX`               | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `en_US`                     | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `eo`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `et`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ee`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `fo`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `fo@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `fil`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `fi`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `fi@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `fi@collation=traditional`  | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `fr`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `fr_CA`                     | `off`     | `non-ignorable` | `FALSE`       | `TRUE`    |
| `gl`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `gl@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ka`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `de`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `de@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `de@collation=phonebook`    | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `de@collation=eor`          | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `de_AT`                     | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `de_AT@collation=phonebook` | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `el`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `gu`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ha`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `haw`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `he`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `he@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `hi`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `hu`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `is`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `is@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ig`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `smn`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `smn@collation=search`      | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `id`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ga`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `it`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ja`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ja@collation=unihan`       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `kl`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `kl@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `kn`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `kn@collation=traditional`  | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `kk`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `km`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `kok`                       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ko`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ko@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ko@collation=searchjl`     | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ko@collation=unihan`       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ky`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `lkt`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `lo`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `lv`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ln`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ln@collation=phonetic`     | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `lt`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `dsb`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `lb`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `mk`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ms`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ml`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `mt`                        | `upper`   | `non-ignorable` | `FALSE`       | `FALSE`   |
| `mr`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `mn`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ne`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `se`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `se@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `nb`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `nb@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `nn`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `nn@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `or`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `om`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ps`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `fa`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `fa_AF`                     | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `pl`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `pt`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `pa`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ro`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ru`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sr`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sr_Latn`                   | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sr_Latn@collation=search`  | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `si`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `si@collation=dictionary`   | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `sk`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sk@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `sl`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `es`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `es@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `es@collation=traditional`  | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sw`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sv`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `sv@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `ta`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `te`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `th`                        | `off`     | shifted         | `TRUE`        | `FALSE`   |
| `bo`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `to`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `tr`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `tr@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `uk`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `hsb`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ur`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `ug`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `vi`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `vi@collation=traditional`  | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `wae`                       | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `cy`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |
| `yi`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `yi@collation=search`       | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `yo`                        | `off`     | `non-ignorable` | `TRUE`        | `FALSE`   |
| `zu`                        | `off`     | `non-ignorable` | `FALSE`       | `FALSE`   |