# 一对一嵌套关系模型

在本页面

* [概述](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#overview)
* [嵌套文档模式](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#embedded-document-pattern)
* [子集模式](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#subset-pattern)

## [概述¶](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#overview)

这章节使用嵌套文档的模型描述了具有一对一关系的数据实体。把有关联的数据嵌套在单个文档中，可以减少读操作的次数。通常来说，如果你按嵌套文档模式来设计你的数据结构，那么在一次读取操作里，你的应用程序会接收文档所有的信息。

## [嵌套文档模式](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#embedded-document-pattern)

考虑下面 patron 和 address 映射关系的示例。这个示例说明嵌套文档优于引用：你需要在一个数据实体的内部查看另一个数据实体的信息。数据实体 `patron` 和 数据实体`address` 的关系是一对一的，一个 `address` 数据实体属于一个 `patron` 数据实体。

在标准化数据模型里，`address` 文档包含了 `patron`文档的引用。

```text
// patron document
// patron 文档
{
   _id: "joe",
   name: "Joe Bookreader"
}

// address document
// address 文档
{
   patron_id: "joe", // reference to patron document // patron文档的引用
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}
```

如果以引用的方式频繁读取`name` 和 `address` 的数据，那么你的应用程序需要查询多次才能获取信息。更好的方式应该把 `address` 实体嵌套到 `patron` 实体内，像下面这个示例。

```text
{
   _id: "joe",
   name: "Joe Bookreader",
   address: {
              street: "123 Fake Street",
              city: "Faketon",
              state: "MA",
              zip: "12345"
            }
}
```

在嵌套文档模型里，你的应用程序查询一次就能获取 `patron` 的完整信息。

## [子集模式](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#subset-pattern)

嵌套文档模型有一个潜在问题是：当文档包含了应用程序不需要的字段时，它会导致文档过大。这些冗余的数据会造成服务器的额外开销从而降低读的性能。相反，你可以把频繁被访问的数据子集放在单独的数据库中，以子集模式去方式去获取。

考虑一个应用会呈现电影的信息。数据库包含了一个 `movie` 集合，`movie` 的模式如下。

```text
{
  "_id": 1,
  "title": "The Arrival of a Train",
  "year": 1896,
  "runtime": 1,
  "released": ISODate("01-25-1896"),
  "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
  "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
  "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
  "lastupdated": ISODate("2015-08-15T10:06:53"),
  "type": "movie",
  "directors": [ "Auguste Lumière", "Louis Lumière" ],
  "imdb": {
    "rating": 7.3,
    "votes": 5043,
    "id": 12
  },
  "countries": [ "France" ],
  "genres": [ "Documentary", "Short" ],
  "tomatoes": {
    "viewer": {
      "rating": 3.7,
      "numReviews": 59
    },
    "lastUpdated": ISODate("2020-01-09T00:02:53")
  }
}
```

目前，在展示一部电影的简介时，`movie` 集合包含了应用程序不需要的几个的字段，比如像 `fullplot` 和 `rating` 的值。并不是要把电影的所有的数据都存储在单个集合里，你可以把单个集合分离成两个集合：

* `movie` 集合包含了一部电影的基本信息。应用程序会默认加载这个文档数据。

  ```text
  // movie collection
  // movie 集合

  {
    "_id": 1,
    "title": "The Arrival of a Train",
    "year": 1896,
    "runtime": 1,
    "released": ISODate("1896-01-25"),
    "type": "movie",
    "directors": [ "Auguste Lumière", "Louis Lumière" ],
    "countries": [ "France" ],
    "genres": [ "Documentary", "Short" ],
  }
  ```

* `movie_details` 集合包含了每部电影额外的，较少访问的数据。

  ```text
  // movie_details collection
  // movie_details 集合

  {
    "_id": 156,
    "movie_id": 1, // reference to the movie collection
    "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
    "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
    "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
    "lastupdated": ISODate("2015-08-15T10:06:53"),
    "imdb": {
      "rating": 7.3,
      "votes": 5043,
      "id": 12
    },
    "tomatoes": {
      "viewer": {
        "rating": 3.7,
        "numReviews": 59
      },
      "lastUpdated": ISODate("2020-01-29T00:02:53")
    }
  }
  ```

这种方法提高了读取性能，因为它要求应用程序读取更少的数据来满足最常见的需求。如果需要那些较少被访问的数据，应用程序会调用其他的数据库。

> 提示：
>
> 当考虑在哪里分离你的数据时，在文档中频繁被访问的部分应该被分离出来，因为它会被应用程序首先加载。

另请参阅

了解如何使用子集模式到一对多关系集合模型中，参阅 [Model One-to-Many Relationships with Embedded Documents](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-many-relationships-between-documents/#data-modeling-example-one-to-many)

### 子集模式的权衡

使用包含频繁被访问数据的小文档会减少工作集的大小。这些较小的文档集会提升了读取性能，并为应用程序提供更多内存可用。

然而，理解你的应用程序及其加载数据的方式是重要的。如果分离数据不当，你的应用程序会经常需要多次访问数据库和依赖 `JOIN` 操作才能获取应用程序需要的全部数据。

另外，你的数据被分离成很多个集合，会增加数据库的维护成本。因为它会使数据存储和数据查询变得困难。

原文链接：[https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/)

译者：朱俊豪

