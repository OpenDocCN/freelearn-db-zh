# 第二章：模式设计和数据建模

本章将重点讨论无模式数据库（如 MongoDB）的模式设计。尽管这听起来有些违反直觉，但在开发 MongoDB 时，我们应该考虑一些因素。我们将了解 MongoDB 支持的模式考虑因素和数据类型。我们还将学习如何通过连接 Ruby、Python 和 PHP 来为 MongoDB 准备文本搜索的数据。

在本章中，我们将涵盖以下主题：

+   关系模式设计

+   数据建模

+   为原子操作建模数据

+   建模关系

+   连接到 MongoDB

# 关系模式设计

在关系数据库中，我们的设计目标是避免异常和冗余。当我们在多个列中存储相同的信息时，异常可能会发生；我们更新其中一个列，但没有更新其他列，因此最终得到相互冲突的信息。当我们无法删除一行而不丢失我们可能需要的信息时，异常也可能发生，可能是在其他引用它的行中。数据冗余可能发生在我们的数据不在正常形式中，但在不同的表中具有重复数据。这可能导致数据不一致，并且难以维护。

在关系数据库中，我们使用正常形式来规范化我们的数据。从基本的**第一正常形式**（**1NF**）开始，到 2NF、3NF 和 BCNF，我们根据功能依赖关系对我们的数据进行建模，如果我们遵循规则，我们最终可能会得到比领域模型对象更多的表。

实际上，关系数据库建模通常是由我们拥有的数据结构驱动的。在遵循某种**模型-视图-控制器**（**MVC**）模式的 Web 应用程序中，我们将根据我们的模型来设计我们的数据库，这些模型是根据**统一建模语言**（**UML**）图表约定进行建模的。像**Django**的 ORM 或 Rails 的**Active Record**这样的抽象帮助应用程序开发人员将数据库结构抽象为对象模型。最终，很多时候，我们最终设计我们的数据库是基于可用数据的结构。因此，我们是根据我们可以得到的答案来设计的。

# MongoDB 模式设计

与关系数据库相比，在 MongoDB 中，我们必须基于我们特定于应用程序的数据访问模式进行建模。找出我们的用户将会有的问题对于设计我们的实体至关重要。与 RDBMS 相比，数据重复和去规范化更频繁地使用，并且有充分的理由。

MongoDB 使用的文档模型意味着每个文档可以容纳的信息量远远多于或少于下一个文档，即使在同一个集合中也是如此。再加上在嵌入文档级别上 MongoDB 可以进行丰富和详细的查询，这意味着我们可以自由设计我们的文档。当我们了解我们的数据访问模式时，我们可以估计哪些字段需要被嵌入，哪些可以拆分到不同的集合中。

# 读写比

读写比通常是 MongoDB 建模的重要考虑因素。在读取数据时，我们希望避免散布/聚集的情况，即我们必须向多个分片发出随机 I/O 请求才能获取应用程序所需的数据。

另一方面，在写入数据时，我们希望将写入分散到尽可能多的服务器上，以避免过载任何一个服务器。这些目标表面上看起来是相互冲突的，但一旦我们了解我们的访问模式，并结合应用程序设计考虑，比如使用副本集从辅助节点读取，它们可以结合起来。

# 数据建模

在本节中，我们将讨论 MongoDB 使用的不同数据类型，它们如何映射到编程语言使用的数据类型，以及我们如何使用 Ruby、Python 和 PHP 在 MongoDB 中建模数据关系。

# 数据类型

MongoDB 使用 BSON，这是一种用于 JSON 文档的二进制编码序列化。 BSON 扩展了 JSON 数据类型，例如提供了原生数据和二进制数据类型。

与协议缓冲区相比，BSON 允许更灵活的模式，但以空间效率为代价。总的来说，BSON 在编码/解码操作中是空间高效、易于遍历和时间高效的，如下表所示。（请参阅 MongoDB 文档[`docs.mongodb.com/manual/reference/bson-types/`](https://docs.mongodb.com/manual/reference/bson-types/)）：

| **类型** | **数字** | **别名** | **备注** |
| --- | --- | --- | --- |
| 双精度 | 1 | `double` |  |
| String | 2 | `string` |  |
| 对象 | 3 | `object` |  |
| 数组 | 4 | `array` |  |
| 二进制数据 | 5 | `binData` |  |
| ObjectID | 7 | `objectId` |  |
| 布尔 | 8 | `bool` |  |
| 日期 | 9 | `date` |  |
| 空 | 10 | `null` |  |
| 正则表达式 | 11 | `regex` |  |
| JavaScript | 13 | `javascript` |  |
| JavaScript（带作用域） | 15 | `javascriptWithScope` |  |
| 32 位整数 | 16 | `int` |  |
| 时间戳 | 17 | `timestamp` |  |
| 64 位整数 | 18 | `long` |  |
| Decimal128 | 19 | `decimal` | 3.4 版中的新功能 |
| 最小键 | -1 | `minKey` |  |
| 最大键 | 127 | `maxKey` |  |
| 未定义 | 6 | `undefined` | 已弃用 |
| DBPointer | 12 | `dbPointer` | 已弃用 |
| 符号 | 14 | `symbol` | 已弃用 |

在 MongoDB 中，我们可以在给定字段的文档中具有不同值类型，并且在使用`$type`运算符进行查询时，我们对它们进行区分。

例如，如果我们在 GBP 中有一个 32 位整数和`double`数据类型的`balance`字段，如果`balance`中有便士或没有，我们可以轻松查询所有帐户，这些帐户具有任何以下查询中显示的四舍五入的`balance`：

```sql
db.account.find( { "balance" : { $type : 16 } } );
db.account.find( { "balance" : { $type : "integer" } } );
```

我们将在以下部分比较不同的数据类型。

# 比较不同的数据类型

由于 MongoDB 的性质，在同一字段中具有不同数据类型的对象是完全可以接受的。这可能是意外发生的，也可能是有意为之（即，在字段中有空值和实际值）。

不同类型数据的排序顺序，从高到低，如下所示：

1.  内部类型的最大键

1.  正则表达式

1.  时间戳

1.  日期

1.  布尔值

1.  ObjectID

1.  二进制数据

1.  数组

1.  对象

1.  符号，字符串

1.  数字（`int`，`long`，`double`）

1.  空

1.  内部类型的最小键

不存在的字段会按照在相应字段中具有`null`的方式进行排序。比较数组比较字段更复杂。比较的升序（或`<`）将比较每个数组的最小元素。比较的降序（或`>`）将比较每个数组的最大元素。

例如，查看以下情景：

```sql
> db.types.find()
{ "_id" : ObjectId("5908d58455454e2de6519c49"), "a" : [ 1, 2, 3 ] }
{ "_id" : ObjectId("5908d59d55454e2de6519c4a"), "a" : [ 2, 5 ] }
```

按升序排列，如下所示：

```sql
> db.types.find().sort({a:1})
{ "_id" : ObjectId("5908d58455454e2de6519c49"), "a" : [ 1, 2, 3 ] }
{ "_id" : ObjectId("5908d59d55454e2de6519c4a"), "a" : [ 2, 5 ] }
```

然而，按降序排列，如下所示：

```sql
> db.types.find().sort({a:-1})
{ "_id" : ObjectId("5908d59d55454e2de6519c4a"), "a" : [ 2, 5 ] }
{ "_id" : ObjectId("5908d58455454e2de6519c49"), "a" : [ 1, 2, 3 ] }
```

当比较数组与单个数字值时，也是如下示例所示。插入一个整数值为`4`的新文档的操作如下：

```sql
> db.types.insert({"a":4})
WriteResult({ "nInserted" : 1 })
```

以下示例显示了降序`sort`的代码片段：

```sql
> db.types.find().sort({a:-1})
{ "_id" : ObjectId("5908d59d55454e2de6519c4a"), "a" : [ 2, 5 ] }
{ "_id" : ObjectId("5908d73c55454e2de6519c4c"), "a" : 4 }
{ "_id" : ObjectId("5908d58455454e2de6519c49"), "a" : [ 1, 2, 3 ] }
```

以下示例是升序`sort`的代码片段：

```sql
> db.types.find().sort({a:1})
{ "_id" : ObjectId("5908d58455454e2de6519c49"), "a" : [ 1, 2, 3 ] }
{ "_id" : ObjectId("5908d59d55454e2de6519c4a"), "a" : [ 2, 5 ] }
{ "_id" : ObjectId("5908d73c55454e2de6519c4c"), "a" : 4 }
```

在每种情况下，我们都突出显示了要比较的值。

我们将在以下部分了解数据类型。

# 日期类型

日期以毫秒为单位存储，从 1970 年 1 月 1 日（纪元时间）开始生效。它们是 64 位有符号整数，允许在 1970 年之前和之后的 135 百万年范围内。负日期值表示 1970 年 1 月 1 日之前的日期。BSON 规范将`date`类型称为 UTC`DateTime`。

MongoDB 中的日期存储在 UTC 中。与一些关系数据库中的`timestamp`带有`timezone`数据类型不同。需要根据本地时间访问和修改时间戳的应用程序应该将`timezone`偏移量与日期一起存储，并在应用程序级别上偏移日期。

在 MongoDB shell 中，可以使用以下 JavaScript 格式来完成：

```sql
var now = new Date();
db.page_views.save({date: now,
 offset: now.getTimezoneOffset()});
```

然后您需要应用保存的偏移量来重建原始本地时间，就像以下示例中所示：

```sql
var record = db.page_views.findOne();
var localNow = new Date( record.date.getTime() - ( record.offset * 60000 ) );
```

在下一节中，我们将介绍`ObjectId`。

# ObjectId

`ObjectId`是 MongoDB 的特殊数据类型。每个文档从创建到销毁都有一个`_id`字段。它是集合中每个文档的主键，并且必须是唯一的。如果我们在`create`语句中省略了这个字段，它将自动分配一个`ObjectId`。

擅自更改`ObjectId`是不可取的，但我们可以小心使用它来达到我们的目的。

`ObjectId`具有以下区别：

+   它有 12 个字节

+   它是有序的

+   按 _id 排序将按每个文档的创建时间进行排序

+   存储创建时间可以通过在 shell 中使用`.getTimeStamp()`来访问

`ObjectId`的结构如下：

+   一个 4 字节的值，表示自 Unix 纪元以来的秒数

+   一个 3 字节的机器标识符

+   一个 2 字节的进程 ID

+   一个 3 字节的计数器，从一个随机值开始

下图显示了 ObjectID 的结构：

![](img/28e23c3f-309a-4e34-b437-cfbce6e41960.png)

按其结构，`ObjectId`对于所有目的都是唯一的；但是，由于这是在客户端生成的，您应该检查底层库的源代码，以验证实现是否符合规范。

在下一节中，我们将学习有关建模原子操作的数据。

# 建模原子操作的数据

MongoDB 正在放宽许多在关系型数据库中找到的典型**原子性、一致性、隔离性和持久性**（**ACID**）约束。在没有事务的情况下，有时很难在操作中保持状态一致，特别是在发生故障时。

幸运的是，一些操作在文档级别上是原子的：

+   `update()`

+   `findandmodify()`

+   `remove()`

这些都是针对单个文档的原子（全部或无）。

这意味着，如果我们在同一文档中嵌入信息，我们可以确保它们始终同步。

一个示例是库存应用程序，每个库存中的物品都有一个文档，我们需要统计库存中剩余的可用物品数量，购物车中已放置的物品数量，并将这些数据用于计算总可用物品数量。

对于`total_available = 5`，`available_now = 3`，`shopping_cart_count = 2`，这个用例可能如下所示：`{available_now : 3, Shopping_cart_by: ["userA", "userB"] }`

当有人将商品放入购物车时，我们可以发出原子更新，将他们的用户 ID 添加到`shopping_cart_by`字段中，并同时将`available_now`字段减少一个。

此操作将在文档级别上保证是原子的。如果我们需要在同一集合中更新多个文档，更新操作可能会成功完成，而不修改我们打算修改的所有文档。这可能是因为该操作不能保证跨多个文档更新是原子的。

这种模式在某些情况下有所帮助，但并非所有情况都适用。在许多情况下，我们需要对所有文档或甚至集合应用多个更新，要么全部成功，要么全部失败。

一个典型的例子是两个账户之间的银行转账。我们想要从用户 A 那里减去 x 英镑，然后将 x 添加到用户 B 那里。如果我们无法完成这两个步骤中的任何一个，我们将返回到两个余额的原始状态。

这种模式的细节超出了本书的范围，但大致上，想法是实现一个手工编码的两阶段**提交**协议。该协议应该为每个转账创建一个新的事务条目，并在该事务中的每个可能状态（如初始、挂起、应用、完成、取消中、已取消）中创建一个新的事务条目，并根据每个事务留下的状态，对其应用适当的回滚函数。

如果您发现自己不得不在一个旨在避免它们的数据库中实现事务，请退一步，重新思考为什么需要这样做。

# 写隔离

我们可以节约地使用`$isolated`来隔离对多个文档的写入，以防其他写入者或读取者对这些文档进行操作。在前面的例子中，我们可以使用`$isolated`来更新多个文档，并确保在其他人有机会进行双倍花费并耗尽资金源账户之前，我们更新两个余额。

然而，这不会给我们带来原子性，即全有或全无的方法。因此，如果更新只部分修改了两个账户，我们仍然需要检测并取消处于挂起状态的任何修改。

`$isolated`在整个集合上使用独占锁，无论使用哪种存储引擎。这意味着在使用它时会有严重的速度惩罚，特别是对于 WiredTiger 文档级别的锁定语义。

`$isolated`在分片集群中不起作用，当我们决定从副本集转到分片部署时可能会成为一个问题。

# 读取隔离和一致性

在传统的关系数据库管理系统定义中，MongoDB 的读取操作将被描述为*读取未提交*。这意味着，默认情况下，读取可能会获取到最终不会持久到磁盘上的值，例如，数据丢失或副本集回滚操作。

特别是，在使用默认写入行为更新多个文档时，缺乏隔离可能会导致以下问题：

+   读取可能会错过在更新操作期间更新的文档

+   非串行化操作

+   读取操作不是即时的

这些可以通过使用`$isolated`运算符来解决，但会带来严重的性能惩罚。

在某些情况下，不使用`.snapshot()`的游标查询可能会得到不一致的结果。如果查询的结果游标获取了一个文档，而在查询仍在获取结果时该文档接收到更新，并且由于填充不足，最终位于磁盘上的不同物理位置，超出了查询结果游标的位置。`.snapshot()`是这种边缘情况的解决方案，但有以下限制：

+   它不适用于分片

+   它不适用于使用`sort()`或`hint()`来强制使用索引

+   它仍然不会提供即时读取行为

如果我们的集合大部分是静态数据，我们可以在查询字段中使用唯一索引来模拟`snapshot()`，并且仍然能够对其应用`sort()`。

总的来说，我们需要在应用程序级别应用保障措施，以确保我们不会得到意外的结果。

从版本 3.4 开始，MongoDB 提供了可线性化的读关注。通过从副本集的主要成员和大多数写关注中使用线性化的读关注，我们可以确保多个线程可以读取和写入单个文档，就好像单个线程在依次执行这些操作一样。在关系型数据库管理系统中，这被认为是一个线性化的调度，MongoDB 称之为实时顺序。

# 建模关系

在接下来的章节中，我们将解释如何将关系数据库管理系统理论中的关系转换为 MongoDB 的文档集合层次结构。我们还将研究如何在 MongoDB 中为文本搜索建模我们的数据。

# 一对一

从关系数据库世界来看，我们通过它们的关系来识别对象。一个一对一的关系可能是一个人和一个地址。在关系数据库中对其进行建模很可能需要两个表：一个**Person**表和一个**Address**表，**Address**表中有一个`person_id`外键，如下图所示：

![](img/b045cf93-073f-43f2-aeb7-d0831e4e7b68.png)

在 MongoDB 中，完美的类比是两个集合，`Person`和`Address`，如下代码所示：

```sql
> db.Person.findOne()
{
"_id" : ObjectId("590a530e3e37d79acac26a41"), "name" : "alex"
}
> db.Address.findOne()
{
"_id" : ObjectId("590a537f3e37d79acac26a42"),
"person_id" : ObjectId("590a530e3e37d79acac26a41"),
"address" : "N29DD"
}
```

现在，我们可以像在关系数据库中一样使用相同的模式从`address`中查找`Person`，如下例所示：

```sql
> db.Person.find({"_id": db.Address.findOne({"address":"N29DD"}).person_id})
{
"_id" : ObjectId("590a530e3e37d79acac26a41"), "name" : "alex"
}
```

这种模式在关系世界中是众所周知的，并且有效。

在 MongoDB 中，我们不必遵循这种模式，因为有更适合模型这些关系的方式。

在 MongoDB 中，我们通常会通过嵌入来建模一对一或一对多的关系。如果一个人有两个地址，那么同样的例子将如下所示：

```sql
{ "_id" : ObjectId("590a55863e37d79acac26a43"), "name" : "alex", "address" : [ "N29DD", "SW1E5ND" ] }
```

使用嵌入数组，我们可以访问此用户拥有的每个`address`。嵌入查询丰富而灵活，因此我们可以在每个文档中存储更多信息，如下例所示：

```sql
{ "_id" : ObjectId("590a56743e37d79acac26a44"),
"name" : "alex",
"address" : [ { "description" : "home", "postcode" : "N29DD" },
{ "description" : "work", "postcode" : "SW1E5ND" } ] }
```

这种方法的优点如下：

+   无需跨不同集合进行两次查询

+   它可以利用原子更新来确保文档中的更新对于其他读取此文档的读者来说是全有或全无的

+   它可以在多个嵌套级别中嵌入属性，创建复杂的结构

最显著的缺点是文档的最大大小为 16 MB，因此这种方法不能用于任意数量的属性。在嵌入数组中存储数百个元素也会降低性能。

# 一对多和多对多

当关系的*多*方的元素数量可以无限增长时，最好使用引用。引用可以有两种形式：

1.  从关系的*一*方，存储多边元素的数组，如下例所示：

```sql
> db.Person.findOne()
{ "_id" : ObjectId("590a530e3e37d79acac26a41"), "name" : "alex", addresses:
[ ObjectID('590a56743e37d79acac26a44'),
ObjectID('590a56743e37d79acac26a46'),
ObjectID('590a56743e37d79acac26a54') ] }
```

1.  这样我们可以从一方获取`addresses`数组，然后使用`in`查询获取多方的所有文档，如下例所示：

```sql
> person = db.Person.findOne({"name":"mary"})
> addresses = db.Addresses.find({_id: {$in: person.addresses} })
```

将这种一对多转换为多对多就像在关系的两端（即`Person`和`Address`集合）都存储这个数组一样容易。

1.  从关系的多方，存储对一方的引用，如下例所示：

```sql
> db.Address.find()
{ "_id" : ObjectId("590a55863e37d79acac26a44"), "person":  ObjectId("590a530e3e37d79acac26a41"), "address" : [ "N29DD" ] }
{ "_id" : ObjectId("590a55863e37d79acac26a46"), "person":  ObjectId("590a530e3e37d79acac26a41"), "address" : [ "SW1E5ND" ] }
{ "_id" : ObjectId("590a55863e37d79acac26a54"), "person":  ObjectId("590a530e3e37d79acac26a41"), "address" : [ "N225QG" ] }
> person = db.Person.findOne({"name":"alex"})
> addresses = db.Addresses.find({"person": person._id})
```

正如我们所看到的，无论哪种设计，我们都需要对数据库进行两次查询以获取信息。第二种方法的优势在于它不会让任何文档无限增长，因此它可以用于一对多是一对数百万的情况。

# 为关键字搜索建模数据

在许多应用程序中，搜索文档中的关键字是一个常见的操作。如果这是一个核心操作，使用专门的搜索存储，如**Elasticsearch**是有意义的；然而，直到规模要求转移到不同的解决方案之前，MongoDB 可以有效地使用。

关键字搜索的基本需求是能够搜索整个文档中的关键字。例如，在`products`集合中的文档，如下例所示：

```sql
{ name : "Macbook Pro late 2016 15in" ,
  manufacturer : "Apple" ,
  price: 2000 ,
  keywords : [ "Macbook Pro late 2016 15in", "2000", "Apple", "macbook", "laptop", "computer" ]
 }
```

我们可以在`keywords`字段中创建多键索引，如下例所示：

```sql
> db.products.createIndex( { keywords: 1 } )
```

现在我们可以在`keywords`字段中搜索任何名称、制造商、价格，以及我们设置的任何自定义关键字。这不是一种高效或灵活的方法，因为我们需要保持关键字列表同步，我们不能使用词干处理，也不能对结果进行排名（更像是过滤而不是搜索）。这种方法的唯一优点是它实现起来稍微快一些。

自 2.4 版本以来，MongoDB 就有了特殊的文本索引类型。它可以在一个或多个字段中声明，并支持词干处理、标记化、精确短语(`" "`)、否定(`-`)和加权结果。

在三个字段上声明具有自定义`权重`的索引如下例所示：

```sql
db.products.createIndex({
 name: "text",
 manufacturer: "text",
 price: "text"
 },
 {
 weights: { name: 10,
 manufacturer: 5,
 price: 1 },
 name: "ProductIndex"
 })
```

在这个例子中，`name`比`price`重要的程度是`10`倍，但比`manufacturer`只重要两倍。

可以使用通配符声明`text`索引，匹配与模式匹配的所有字段，如下例所示：

```sql
db.collection.createIndex( { "$**": "text" } )
```

这在我们有非结构化数据并且可能不知道它们将带有哪些字段时非常有用。我们可以像处理任何其他索引一样，通过名称删除索引。

然而，最大的优势是，除了所有的功能之外，所有的记录都是由数据库完成的。

在下一节中，我们将学习如何连接到 MongoDB。

# 连接到 MongoDB

有两种连接到 MongoDB 的方式。第一种是使用您的编程语言的驱动程序。第二种是使用 ODM 层以透明的方式将模型对象映射到 MongoDB。在本节中，我们将涵盖使用 Web 应用程序开发中最流行的三种语言：Ruby、Python 和 PHP 的两种方式。

# 使用 Ruby 连接

Ruby 是第一批得到 MongoDB 官方驱动程序支持的语言之一。在 GitHub 上，官方的 MongoDB Ruby 驱动程序是连接到 MongoDB 实例的推荐方式。执行以下步骤使用 Ruby 连接 MongoDB：

1.  安装就像将其添加到 Gemfile 一样简单，如下例所示：

```sql
gem 'mongo', '~> 2.6'
```

您需要安装 Ruby，然后从[`rvm.io/rvm/install`](https://rvm.io/rvm/install)安装 RVM，最后运行`gem install bundler`。

1.  然后，在我们的类中，我们可以连接到数据库，如下例所示：

```sql
require 'mongo'
client = Mongo::Client.new([ '127.0.0.1:27017' ], database: 'test')
```

1.  这是可能的最简单的例子：连接到我们的`localhost`中名为`test`的单个数据库实例。在大多数情况下，我们至少会有一个副本集要连接，如下面的代码片段所示：

```sql
client_host = ['server1_hostname:server1_ip, server2_hostname:server2_ip']
 client_options = {
  database: 'YOUR_DATABASE_NAME',
  replica_set: 'REPLICA_SET_NAME',
  user: 'YOUR_USERNAME',
  password: 'YOUR_PASSWORD'
 }
client = Mongo::Client.new(client_host, client_options)
```

1.  `client_host`服务器正在为客户端驱动程序提供服务器以尝试连接。一旦连接，驱动程序将根据主/次读取或写入配置确定要连接的服务器。`replica_set`属性需要匹配`REPLICA_SET_NAME`才能连接。

1.  `user`和`password`是可选的，但在任何 MongoDB 实例中都强烈建议使用。在`mongod.conf`文件中默认启用身份验证是一个良好的做法，我们将在第八章中了解更多信息，*监控、备份和安全*。

1.  连接到分片集群与连接到副本集类似，唯一的区别是，我们需要连接到充当 MongoDB 路由器的 MongoDB 进程，而不是提供服务器主机/端口。

# Mongoid ODM

使用低级驱动程序连接到 MongoDB 数据库通常不是最有效的方法。低级驱动程序提供的所有灵活性都抵消了更长的开发时间和用于将我们的模型与数据库粘合在一起的代码。

ODM 可以是这些问题的答案。就像 ORM 一样，ODM 弥合了我们的模型和数据库之间的差距。在 Rails 中，作为 Ruby 最广泛使用的 MVC 框架的 Mongoid 可以用于以类似于 Active Record 的方式对我们的数据进行建模。

安装`gem`类似于 Mongo Ruby 驱动程序，通过在 Gemfile 中添加一个文件，如下面的代码所示：

```sql
gem 'mongoid', '~> 7.0'
```

根据 Rails 的版本，我们可能还需要将以下内容添加到`application.rb`中：

```sql
config.generators do |g|
g.orm :mongoid
end
```

通过配置文件`mongoid.yml`连接到数据库，配置选项以语义缩进的键值对形式传递。其结构类似于用于关系数据库的`database.yml`。

我们可以通过`mongoid.yml`文件传递的一些选项如下表所示：

| **选项值** | **描述** |
| --- | --- |
| `Database` | 数据库名称。 |
| `Hosts` | 我们的数据库主机。 |
| `Write`/`w` | 写入关注（默认为 1）。 |
| `Auth_mech` | 认证机制。有效选项包括：`:scram`、`:mongodb_cr`、`:mongodb_x509`和`:plain`。3.0 的默认选项是`:scram`，而 2.4 和 2.6 的默认选项是`:plain`。 |
| `Auth_source` | 我们认证机制的认证源。 |
| `Min_pool_size`/`max_pool_size` | 连接的最小和最大池大小。 |
| `SSL`、`ssl_cert`、`ssl_key`、`ssl_key_pass_phrase`、`ssl_verify` | 一组关于与数据库的 SSL 连接的选项。 |
| `Include_root_in_json` | 在 JSON 序列化中包含根模型名称。 |
| `Include_type_for_serialization` | 在序列化 MongoDB 对象时包含`_type`字段。 |
| `Use_activesupport_time_zone` | 在服务器和客户端之间转换时间戳时使用 active support 的时区。 |

下一步是修改我们的模型以存储在 MongoDB 中。这就像在模型声明中包含一行代码那样简单，如下例所示：

```sql
class Person
  include Mongoid::Document
 End
```

我们还可以使用以下代码：

```sql
include Mongoid::Timestamps
```

我们用它来生成类似于 Active Record 的 `created_at` 和 `updated_at` 字段。在我们的模型中，数据字段不需要按类型声明，但这样做是个好习惯。支持的数据类型如下：

+   `Array`

+   `BigDecimal`

+   `Boolean`

+   `Date`

+   `DateTime`

+   `Float`

+   `Hash`

+   `Integer`

+   `BSON::ObjectId`

+   `BSON::Binary`

+   `Range`

+   `Regexp`

+   `String`

+   `Symbol`

+   `Time`

+   `TimeWithZone`

如果字段的类型未定义，字段将被转换为对象并存储在数据库中。这样稍微快一些，但不支持所有类型。如果我们尝试使用 `BigDecimal`、`Date`、`DateTime` 或 `Range`，将会收到错误信息。

# 使用 Mongoid 模型进行继承

以下代码是使用 Mongoid 模型进行继承的示例：

```sql
class Canvas
  include Mongoid::Document
  field :name, type: String
  embeds_many :shapes
end

class Shape
  include Mongoid::Document
  field :x, type: Integer
  field :y, type: Integer
  embedded_in :canvas
end

class Circle < Shape
  field :radius, type: Float
end

class Rectangle < Shape
  field :width, type: Float
  field :height, type: Float
end
```

现在，我们有一个具有许多嵌入的 `Shape` 对象的 `Canvas` 类。Mongoid 将自动创建一个字段，即 `_type`，以区分父节点和子节点字段。在从字段继承文档的情况下，关系、验证和作用域会复制到其子文档中，但反之则不会。

`embeds_many` 和 `embedded_in` 对将创建嵌入式子文档以存储关系。如果我们想通过引用 `ObjectId` 来存储这些关系，可以通过将它们替换为 `has_many` 和 `belongs_to` 来实现。

# 使用 Python 进行连接

与 Ruby 和 Rails 相媲美的是 Python 和 Django。类似于 Mongoid，还有 MongoEngine 和官方的 MongoDB 低级驱动程序 PyMongo。

使用 `pip` 或 `easy_install` 安装 PyMongo，如下代码所示：

```sql
python -m pip install pymongo
python -m easy_install pymongo
```

然后，在我们的类中，我们可以连接到数据库，如下例所示：

```sql
>>> from pymongo import MongoClient
>>> client = MongoClient()
```

连接到副本集需要一组种子服务器，客户端可以找出集合中的主、从或仲裁节点，如下例所示：

```sql
client = pymongo.MongoClient('mongodb://user:passwd@node1:p1,node2:p2/?replicaSet=rsname')
```

使用连接字符串 URL，我们可以在单个字符串中传递用户名、密码和 `replicaSet` 名称。连接字符串 URL 的一些最有趣的选项在下一节中。

连接到分片需要 MongoDB 路由器的服务器主机和 IP，这是 MongoDB 进程。

# PyMODM ODM

与 Ruby 的 Mongoid 类似，PyMODM 是 Python 的 ODM，紧随 Django 内置的 ORM。通过 `pip` 安装 `pymodm`，如下代码所示：

```sql
pip install pymodm
```

然后我们需要编辑 `settings.py`，将数据库 `ENGINE` 替换为 `dummy` 数据库，如下代码所示：

```sql
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.dummy'
    }
}
```

然后我们在 `settings.py` 的任何位置添加我们的连接字符串，如下代码所示：

```sql
from pymodm import connect
connect("mongodb://localhost:27017/myDatabase", alias="MyApplication")
```

在这里，我们必须使用具有以下结构的连接字符串：

```sql
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

选项必须是`name=value`对，每对之间用`&`分隔。一些有趣的对如下表所示：

| **名称** | **描述** |
| --- | --- |
| `minPoolSize`/`maxPoolSize` | 连接的最小和最大池大小。 |
| `w` | 写关注选项。 |
| `wtimeoutMS` | 写关注操作的超时时间。 |
| `Journal` | 日志选项。 |
| `readPreference` | 用于副本集的读取偏好。可用选项包括：`primary`、`primaryPreferred`、`secondary`、`secondaryPreferred`、`nearest`。 |
| `maxStalenessSeconds` | 指定从主服务器滞后的数据可以在客户端停止使用之前的秒数。 |
| `SSL` | 使用 SSL 连接到数据库。 |
| `authSource` | 与用户名一起使用，指定与用户凭据关联的数据库。当我们使用外部认证机制时，LDAP 或 Kerberos 应该是 `$external`。 |
| `authMechanism` | 可用于连接的身份验证机制。MongoDB 的可用选项有：**SCRAM-SHA-1**，**MONGODB-CR**，**MONGODB-X.509**。MongoDB 企业版（付费版本）提供了两个更多的选项：**GSSAPI**（Kerberos），**PLAIN**（**LDAP SASL**） |

模型类需要继承自`MongoModel`。以下代码显示了一个示例类的样子：

```sql
from pymodm import MongoModel, fields
class User(MongoModel):
    email = fields.EmailField(primary_key=True)
    first_name = fields.CharField()
    last_name = fields.CharField()
```

这里有一个`User`类，有`first_name`，`last_name`和`email`字段，其中`email`是主要字段。

# PyMODM 模型的继承

在 MongoDB 中处理一对一和一对多关系可以使用引用或嵌入。下面的例子展示了两种方式，即用户模型的引用和评论模型的嵌入：

```sql
from pymodm import EmbeddedMongoModel, MongoModel, fields

class Comment(EmbeddedMongoModel):
    author = fields.ReferenceField(User)
    content = fields.CharField()

class Post(MongoModel):
    title = fields.CharField()
    author = fields.ReferenceField(User)
    revised_on = fields.DateTimeField()
    content = fields.CharField()
    comments = fields.EmbeddedDocumentListField(Comment)
```

类似于 Ruby 的 Mongoid，我们可以根据设计决定将关系定义为嵌入式或引用式。

# 使用 PHP 连接

两年前，MongoDB PHP 驱动程序从头开始重写，以支持 PHP 5、PHP 7 和 HHVM 架构。当前的架构如下图所示：

![](img/0e79b102-ba4d-4275-9559-dcb782394764.png)

目前，我们对所有三种架构都有官方驱动程序，完全支持底层功能。

安装是一个两步过程。首先，我们需要安装 MongoDB 扩展。这个扩展依赖于我们安装的 PHP（或 HHVM）的版本，可以使用 macOS 中的`brew`来完成。以下示例是使用 PHP 7.0：

```sql
brew install php70-mongodb
```

然后，像下面的例子一样使用`composer`（PHP 中广泛使用的依赖管理器）：

```sql
composer require mongodb/mongodb
```

可以通过使用连接字符串 URL 或通过传递一个选项数组来连接到数据库。

使用连接字符串 URL，我们有以下代码：

```sql
$client = new MongoDB\Client($uri = 'mongodb://127.0.0.1/', array $uriOptions = [], array $driverOptions = [])
```

例如，要使用 SSL 身份验证连接到副本集，我们使用以下代码：

```sql
$client = new MongoDB\Client('mongodb://myUsername:myPassword@rs1.example.com,rs2.example.com/?ssl=true&replicaSet=myReplicaSet&authSource=admin');
```

或者我们可以使用`$uriOptions`参数来传递参数，而不使用连接字符串 URL，如下面的代码所示：

```sql
$client = new MongoDB\Client(
 'mongodb://rs1.example.com,rs2.example.com/'
 [
 'username' => 'myUsername',
 'password' => 'myPassword',
 'ssl' => true,
 'replicaSet' => 'myReplicaSet',
 'authSource' => 'admin',
 ],
);
```

可用的`$uriOptions`和连接字符串 URL 选项与用于 Ruby 和 Python 的选项类似。

# Doctrine ODM

**Laravel**是 PHP 中最广泛使用的 MVC 框架之一，类似于 Python 和 Ruby 世界中的 Django 和 Rails 的架构。我们将通过配置我们的模型使用 Laravel，Doctrine 和 MongoDB。本节假设 Doctrine 已安装并与 Laravel 5.x 一起使用。

Doctrine 实体是**Plain Old PHP Objects**（**POPO**），与**Eloquent**不同，Laravel 的默认 ORM 不需要继承`Model`类。Doctrine 使用**Data Mapper Pattern**，而 Eloquent 使用 Active Record。跳过`get()`和`set()`方法，一个简单的类将如下所示：

```sql
use Doctrine\ORM\Mapping AS ORM;
use Doctrine\Common\Collections\ArrayCollection;
/**
* @ORM\Entity
* @ORM\Table(name="scientist")
*/
class Scientist
{
   /**
    * @ORM\Id
    * @ORM\GeneratedValue
    * @ORM\Column(type="integer")
    */
   protected $id;
   /**
    * @ORM\Column(type="string")
    */
   protected $firstname;
   /**
    * @ORM\Column(type="string")
    */
   protected $lastname;
   /**
   * @ORM\OneToMany(targetEntity="Theory", mappedBy="scientist", cascade={"persist"})
   * @var ArrayCollection|Theory[]
   */
   protected $theories;
   /**
   * @param $firstname
   * @param $lastname
   */
   public function __construct($firstname, $lastname)
   {
       $this->firstname = $firstname;
       $this->lastname  = $lastname;
       $this->theories = new ArrayCollection;
   }
...
   public function addTheory(Theory $theory)
   {
       if(!$this->theories->contains($theory)) {
           $theory->setScientist($this);
           $this->theories->add($theory);
       }
   }
```

这个基于 POPO 的模型使用注释来定义需要在 MongoDB 中持久化的字段类型。例如，`@ORM\Column(type="string")`定义了 MongoDB 中的一个字段，`string`类型的`firstname`和`lastname`作为属性名称，在相应的行中。

这里有一整套可用的注释：[`doctrine2.readthedocs.io/en/latest/reference/annotations-reference.html`](https://doctrine2.readthedocs.io/en/latest/reference/annotations-reference.html)。

如果我们想要将 POPO 结构与注释分开，我们也可以使用 YAML 或 XML 来定义它们，而不是在我们的 POPO 模型类中使用注释。

# Doctrine 的继承

可以通过注释、YAML 或 XML 来建模一对一和一对多关系。使用注释，我们可以在我们的文档中定义多个嵌入的子文档，如下例所示：

```sql
/** @Document */
class User
{
   // ...
   /** @EmbedMany(targetDocument="Phonenumber") */
   private $phonenumbers = array();
   // ...
}
/** @EmbeddedDocument */
class Phonenumber
{
   // ...
}
```

在这里，一个`User`文档嵌入了许多`phonenumbers`。`@EmbedOne()`将嵌入一个子文档，用于建模一对一关系。

引用与嵌入类似，如下例所示：

```sql
/** @Document */
class User
{
   // ...
   /**
    * @ReferenceMany(targetDocument="Account")
    */
   private $accounts = array();
   // ...
}
/** @Document */
class Account
{
   // ...
}
```

`@ReferenceMany()`和`@ReferenceOne()`用于通过引用到单独的集合来建模一对多和一对一关系。

# 摘要

在本章中，我们学习了关系数据库和 MongoDB 的模式设计，以及如何从不同的起点开始实现相同的目标。

在 MongoDB 中，我们必须考虑读写比例，用户在最常见情况下可能会遇到的问题，以及关系之间的基数。

我们学习了关于原子操作以及如何构建查询，以便在没有事务开销的情况下具有 ACID 属性。

我们还了解了 MongoDB 的数据类型，它们如何进行比较，以及一些特殊的数据类型，比如`ObjectId`，它可以被数据库和我们自己利用。

从建模简单的一对一关系开始，我们经历了一对多关系和多对多关系建模，而无需像在关系数据库中那样使用中间表，可以使用引用或嵌入文档。

我们学习了如何为关键字搜索建模数据，这是大多数应用程序在 Web 环境中需要支持的功能之一。

最后，我们探讨了在三种最流行的 Web 编程语言中使用 MongoDB 的不同用例。我们看到了使用官方驱动程序和 Mongoid ODM 的 Ruby 的示例。然后我们探讨了如何使用官方驱动程序和 PyMODM ODM 连接 Python，最后，我们通过使用官方驱动程序和 Doctrine ODM 在 PHP 中的示例进行了工作。

对于所有这些语言（以及许多其他语言），都有官方驱动程序提供支持和完全访问底层数据库操作功能，还有**对象数据建模**框架，用于轻松建模我们的数据和快速开发。

在下一章中，我们将深入探讨 MongoDB shell 以及我们可以使用它实现的操作。我们还将掌握使用驱动程序对我们的文档进行 CRUD 操作。
