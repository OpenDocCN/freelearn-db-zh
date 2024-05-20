# 第六章：聚合

在第五章《多文档 ACID 事务》中，我们使用 Ruby 和 Python 的代码解决了新事务功能的两个用例。在本章中，我们将更深入地了解聚合框架，学习它如何有用。我们还将看看 MongoDB 支持的操作符。

为了了解这些信息，我们将使用聚合来处理以太坊区块链的交易数据。完整的源代码可在[`github.com/PacktPublishing/Mastering-MongoDB-4.x-Second-Edition`](https://github.com/PacktPublishing/Mastering-MongoDB-4.x-Second-Edition)上找到。

在本章中，我们将涵盖以下主题：

+   为什么要使用聚合？

+   不同的聚合操作符

+   限制

# 为什么要使用聚合？

聚合框架是由 MongoDB 在 2.2 版本中引入的（在开发分支中是 2.1 版本）。它作为 MapReduce 框架和直接查询数据库的替代方案。

使用聚合框架，我们可以在服务器上执行`GROUP BY`操作。因此，我们可以只投影结果集中需要的字段。使用`$match`和`$project`操作符，我们可以减少通过管道传递的数据量，从而加快数据处理速度。

自连接——也就是在同一集合内连接数据——也可以使用聚合框架来执行，正如我们将在我们的用例中看到的那样。

将聚合框架与仅使用 shell 或其他驱动程序提供的查询进行比较时，重要的是要记住两者都有用途。

对于选择和投影查询，几乎总是更好使用简单的查询，因为开发、测试和部署聚合框架操作的复杂性很难超过使用内置命令的简单性。查找具有`( db.books.find({price: 50} {price: 1, name: 1}) )`的文档，或者没有`( db.books.find({price: 50}) )`只投影一些字段，是简单且足够快速，不需要使用聚合框架。

另一方面，如果我们想使用 MongoDB 执行`GROUP BY`和自连接操作，可能需要使用聚合框架。在 MongoDB shell 中`group()`命令的最重要限制是结果集必须适合一个文档，这意味着它的大小不能超过 16MB。此外，任何`group()`命令的结果不能超过 20,000 个。最后，`group()`不适用于分片输入集合，这意味着当我们的数据量增加时，我们必须重新编写我们的查询。

与 MapReduce 相比，聚合框架在功能和灵活性上更有限。在聚合框架中，我们受到可用操作符的限制。但好的一面是，聚合框架的 API 比 MapReduce 更容易理解和使用。在性能方面，聚合框架在 MongoDB 早期版本中比 MapReduce 快得多，但在 MapReduce 性能改进后，似乎与最新版本持平。

最后，还有一种选择，就是使用数据库作为数据存储，并使用应用程序执行复杂操作。有时这可能会很快开发，但应该避免，因为最终可能会产生内存、网络和性能成本。

在下一节中，我们将在使用实际用例之前描述可用的操作符。

# 聚合操作符

在本节中，我们将学习如何使用聚合操作符。聚合操作符分为两类。在每个阶段中，我们使用**表达式操作符**来比较和处理值。在不同阶段之间，我们使用**聚合阶段操作符**来定义将从一个阶段传递到下一个阶段的数据，因为它被认为是以相同格式呈现的。

# 聚合阶段操作符

聚合管道由不同的阶段组成。这些阶段在数组中声明并按顺序执行，每个阶段的输出都是下一个阶段的输入。

`$out` 阶段必须是聚合管道中的最终阶段，通过替换或添加到现有文档将数据输出到输出集合：

+   `$group`：最常用于按标识符表达式分组，并应用累加器表达式。它输出每个不同组的一个文档。

+   `$project`：用于文档转换，每个输入文档输出一个文档。

+   `$match`：根据条件从输入中过滤文档。

+   `$lookup`：用于从输入中过滤文档。输入可以是同一数据库中另一个集合中的文档，由外部左连接选择。

+   `$out`：将此管道阶段的文档输出到输出集合，以替换或添加到已存在于集合中的文档。

+   `$limit`：根据预定义的条件限制传递到下一个聚合阶段的文档数量。

+   `$count`：返回管道阶段的文档数量。

+   `$skip`：跳过一定数量的文档，防止它们传递到管道的下一阶段。

+   `$sort`：根据条件对文档进行排序。

+   `$redact`：作为 `$project` 和 `$match` 的组合，这将从每个文档中选择的字段进行 redact，并将它们传递到管道的下一阶段。

+   `$unwind`：这将数组中的 *n* 个元素转换为 *n* 个文档，将每个文档映射到数组的一个元素。然后将这些文档传递到管道的下一阶段。

+   `$collStats`：返回有关视图或集合的统计信息。

+   `$indexStats`：返回集合索引的统计信息。

+   `$sample`：从输入中随机选择指定数量的文档。

+   `$facet`：在单个阶段内组合多个聚合管道。

+   `$bucket`：根据预定义的选择标准和桶边界将文档分割成桶。

+   `$bucketAuto`：根据预定义的选择标准将文档分割成桶，并尝试在桶之间均匀分布文档。

+   `$sortByCount`：根据表达式的值对传入的文档进行分组，并计算每个桶中的文档数量。

+   `$addFields`：这将向文档添加新字段，并输出与输入相同数量的文档，带有添加的字段。

+   `$replaceRoot`：用指定的字段替换输入文档的所有现有字段（包括 `standard _id` 字段）。

+   `$geoNear`：根据与指定字段的接近程度返回文档的有序列表。输出文档包括一个计算出的 `distance` 字段。

+   `$graphLookup`：递归搜索集合，并在每个输出文档中添加一个包含搜索结果的数组字段。

# 表达式运算符

在每个阶段中，我们可以定义一个或多个表达式运算符来应用我们的中间计算。本节将重点介绍这些表达式运算符。

# 表达式布尔运算符

布尔运算符用于将 `true` 或 `false` 的值传递到我们聚合管道的下一阶段。

我们也可以选择传递原始的 `integer`、`string` 或任何其他类型的值。

我们可以像在任何编程语言中一样使用 `$and`、`$or` 和 `$not` 运算符。

# 表达式比较运算符

比较运算符可以与布尔运算符结合使用，构建我们需要评估为 `true`/`false` 的表达式，以输出管道阶段的结果。

最常用的运算符如下：

+   `$eq ( equal )`

+   `$ne ( not equal)`

+   `$gt (greater than)`

+   `$gte (greater than or equal)`

+   `$lt`

+   `$lte`

所有上述运算符返回 `true` 或 `false` 的布尔值。

唯一不返回布尔值的运算符是`$cmp`，如果两个参数相等则返回`0`，如果第一个值大于第二个值则返回`1`，如果第二个值大于第一个值则返回`-1`。

# 集合表达式和数组运算符

与大多数编程语言一样，集合操作会忽略重复的条目和元素的顺序，将它们视为集合。结果的顺序是未指定的，并且重复的条目将在结果集中被去重。集合表达式不会递归应用于集合的元素，而只会应用于顶层。这意味着，如果一个集合包含，例如，一个嵌套数组，那么这个数组可能包含重复项，也可能不包含。

可用的集合运算符如下：

+   `$setEquals`：如果两个集合具有相同的不同元素，则为`true`

+   `$setIntersection`：返回所有输入集合的交集（即出现在所有输入集合中的文档）

+   `$setUnion`：返回所有输入集合的并集（即出现在所有输入集合中的至少一个文档）

+   `$setDifference`：返回出现在第一个输入集合中但不在第二个输入集合中的文档

+   `$setIsSubset`：如果第一个集合中的所有文档都出现在第二个集合中，则为`true`，即使这两个集合是相同的。

+   `$anyElementTrue`：如果集合中的任何元素求值为`true`，则为`true`

+   `$allElementsTrue`：如果集合中的所有元素求值为`true`，则为`true`

可用的数组运算符如下：

+   `$arrayElemAt`：返回数组索引位置的元素。

+   `$concatArrays`：返回一个连接的数组。

+   `$filter`：根据指定的条件返回数组的子集。

+   `$indexOfArray`：返回满足搜索条件的数组的索引。如果没有，则返回`-1`。

+   `$isArray`：如果输入是数组，则返回`true`；否则返回`false`。

+   `$range`：根据用户定义的输入输出包含一系列整数的数组。

+   `$reverseArray`：返回元素顺序相反的数组。

+   `$reduce`：根据指定的输入将数组的元素减少为单个值。

+   `$size`：返回数组中的项目数。

+   `$slice`：返回数组的子集。

+   `$zip`：返回合并的数组。

+   `$in`：如果指定的值在数组中，则返回`true`；否则返回`false`。

# 表达式日期运算符

日期运算符用于从日期字段中提取日期信息，当我们想要基于一周/月/年的统计数据计算时，使用管道：

+   `$dayOfYear` 用于获取一年中的日期，范围为 1 到 366（闰年）

+   `$dayOfMonth` 用于获取一个月中的日期，范围为 1 到 31

+   `$dayOfWeek` 用于获取一周中的日期，范围为 1 到 7，其中 1 代表星期日，7 代表星期六（使用英文星期几）

+   `$isoDayOfWeek` 返回 ISO 8601 日期格式中的星期几编号，范围为 1 到 7，其中 1 代表星期一，7 代表星期日

+   `$week` 是 0 到 53 范围内的周数，0 代表每年年初的部分周，53 代表有闰周的年份

+   `$isoWeek` 返回 ISO 8601 日期格式中的周数，范围为 1 到 53，1 代表包含星期四的年份的第一周，53 代表有闰周的年份

+   `$year`、`$month`、`$hour`、`$minute`、`$milliSecond` 返回日期的相关部分，从零开始编号，除了`$month`，它返回从 1 到 12 的值

+   `$isoWeekYear` 根据 ISO 8601 日期格式返回日期的年份，该日期是 ISO 8601 日期格式中最后一周结束的日期（例如，2016/1/1 仍然返回 2015）

+   `$second` 返回 0 到 60 的值，包括闰秒

+   `$dateToString` 将日期输入转换为字符串

# 表达式字符串运算符

与日期运算符一样，字符串运算符用于在我们想要将数据从管道的一个阶段转换到下一个阶段时使用。潜在的用例包括预处理文本字段以提取相关信息，以便在管道的后续阶段中使用：

+   `$concat`: 这用于连接字符串。

+   `$split`: 这用于根据分隔符拆分字符串。如果找不到分隔符，则返回原始字符串。

+   `$strcasecmp`: 这用于不区分大小写的字符串比较。如果字符串相等，则返回`0`，如果第一个字符串较大，则返回`1`；否则返回`-1`。

+   `$toLower`/`$toUpper`: 这用于将字符串转换为全小写或全大写。

+   `$indexOfBytes`: 这用于返回字符串中子字符串的第一个出现的字节位置。

+   `$strLenBytes`: 这是输入字符串的字节数。

+   `$substrBytes`: 这返回子字符串的指定字节。

代码点的等效方法（Unicode 中的一个值，不考虑其表示中的基础字节）如下：

+   `$indexOfCP`

+   `$strLenCP`

+   `$substrCP`

# 表达式算术运算符

在管道的每个阶段，我们可以应用一个或多个算术运算符来执行中间计算。这些运算符在以下列表中显示：

+   `$abs`: 这是绝对值。

+   `$add`: 这可以将数字或日期加上一个数字以得到一个新的日期。

+   `$ceil`/`$floor`: 这些分别是向上取整和向下取整函数。

+   `$divide`: 这用于由两个输入进行除法。

+   `$exp`: 这将自然数*e*提升到指定的指数幂。

+   `$pow`: 这将一个数字提升到指定的指数幂。

+   `$ln`/`$log`/`$log10`: 这些用于计算自然对数、自定义底数的对数或以十为底的对数。

+   `$mod`: 这是模值。

+   `$multiply`: 这用于将输入相乘。

+   `$sqrt`: 这是输入的平方根。

+   `$subtract`: 这是从第二个值中减去第一个值的结果。如果两个参数都是日期，则返回它们之间的差值。如果一个参数是日期（这个参数必须是第一个参数），另一个是数字，则返回结果日期。

+   `$trunc`: 这用于截断结果。

# 聚合累加器

累加器可能是最广泛使用的运算符，因为它们允许我们对我们组中的每个成员进行求和、平均值、获取标准偏差统计数据以及执行其他操作。以下是聚合累加器的列表：

+   `$sum`: 这是数值的总和。它会忽略非数值。

+   `$avg`: 这是数值的平均值。它会忽略非数值。

+   `$first`/`$last`: 这是通过管道阶段的第一个和最后一个值。它仅在组阶段中可用。

+   `$max`/`$min`: 这分别获取通过管道阶段的最大值和最小值。

+   `$push`: 这将一个新元素添加到输入数组的末尾。它仅在组阶段中可用。

+   `$addToSet`: 这将一个元素（仅当它不存在时）添加到数组中，有效地将其视为一个集合。它仅在组阶段中可用。

+   `$stdDevPop`/`$stdDevSamp`: 这些用于在`$project`或`$match`阶段获取总体/样本标准偏差。

这些累加器在组或项目管道阶段中都可用，除非另有说明。

# 条件表达式

表达式可以根据布尔真值测试将不同的数据输出到我们管道中的下一阶段：

```go
$cond
```

`$cond`短语将评估格式为`if...then...else`的表达式，并根据`if`语句的结果返回`then`语句或`else`分支的值。输入可以是三个命名参数或有序列表中的三个表达式。

```go
$ifNull
```

`$ifNull`短语将评估一个表达式，并在其不为 null 时返回第一个表达式，如果第一个表达式为 null，则返回第二个表达式。Null 可以是一个缺失的字段或一个具有未定义值的字段：

```go
$switch
```

类似于编程语言的`switch`语句，`$switch`将在评估为`true`时执行指定的表达式，并跳出控制流。

# 类型转换运算符

在 MongoDB 4.0 中引入的类型转换运算符允许我们将值转换为指定的类型。命令的通用语法如下：

```go
{
   $convert:
      {
         input: <expression>,
         to: <type expression>,
         onError: <expression>,  // Optional.
         onNull: <expression>    // Optional.
      } }

```

在此语法中，`input`和`to`（唯一的强制参数）可以是任何有效的表达式。在其最简单的形式中，我们可以，例如，有以下内容：

```go
$convert: { input: "true", to: "bool" } 
```

将值为`true`的字符串转换为布尔值`true`。

`onError`短语可以是任何有效的表达式，指定了在转换过程中 MongoDB 遇到错误时将返回的值，包括不支持的类型转换。其默认行为是抛出错误并停止处理。

`onNull`短语也可以是任何有效的表达式，指定了如果输入为 null 或缺失时 MongoDB 将返回的值。默认行为是返回 null。

MongoDB 还为最常见的`$convert`操作提供了一些辅助函数。这些函数如下：

+   `$toBool`

+   `$toDate`

+   `$toDecimal`

+   `$toDouble`

+   `$toInt`

+   `$toLong`

+   `$toObjectId`

+   `$toString`

这些更简单易用。我们可以将前面的示例重写为以下形式：

```go
{ $toBool: "true" }
```

# 其他操作符

有一些操作符并不常用，但在特定用例中可能很有用。其中最重要的列在以下部分中。

# 文本搜索

`$meta`运算符用于访问文本搜索元数据。

# 变量

`$map`运算符将子表达式应用于数组的每个元素，并返回结果值的数组。它接受命名参数。

`$let`运算符为子表达式的范围内定义变量，并返回子表达式的结果。它接受命名参数。

# 字面值

`$literal`运算符将返回一个不经解析的值。它用于聚合管道可能解释为表达式的值。例如，您可以将`$literal`表达式应用于以`$`开头的字符串，以避免解析为字段路径。

# 解析数据类型

`$type`运算符返回字段的`BSON`数据类型。

# 限制

聚合管道可以以以下三种不同的方式输出结果：

+   内联作为包含结果集的文档

+   在一个集合中

+   返回结果集的游标

内联结果受`BSON`最大文档大小 16 MB 的限制，这意味着我们只能在最终结果是固定大小时使用它。一个例子是从电子商务网站输出前五个最常订购商品的`ObjectId`。

与此相反的例子是输出前 1,000 个最常订购的商品，以及产品信息，包括描述和其他大小可变的字段。

如果我们想对数据进行进一步处理，将结果输出到集合是首选解决方案。我们可以将结果输出到新集合，或替换现有集合的内容。聚合输出结果只有在聚合命令成功后才会可见；否则，它将根本不可见。

输出集合不能是分片的或有上限的集合（截至 v3.4）。如果聚合输出违反索引（包括每个文档的唯一`ObjectId`上的内置索引）或文档验证规则，聚合将失败。

每个管道阶段可以有超过 16MB 限制的文档，因为这些由 MongoDB 在内部处理。然而，每个管道阶段只能使用最多 100MB 的内存。如果我们期望在我们的阶段中有更多的数据，我们应该将`allowDiskUse:`设置为`true`，以允许多余的数据溢出到磁盘，以换取性能。

`$graphLookup`运算符不支持超过 100MB 的数据集，并将忽略`allowDiskUse`上的任何设置。

# 聚合使用案例

在这个相当冗长的部分中，我们将使用聚合框架来处理以太坊区块链的数据。

使用我们的 Python 代码，我们已经从以太坊中提取了数据，并将其加载到我们的 MongoDB 数据库中。区块链与我们的数据库的关系如下图所示：

![](img/8ad5e8b5-bd05-430e-9e4f-06548aa820a9.png)

我们的数据驻留在两个集合中：**blocks**和**transactions**。

样本区块文档具有以下字段：

+   交易数量

+   承包内部交易的数量

+   区块哈希

+   父区块哈希

+   挖矿难度

+   使用的燃气

+   区块高度

以下代码显示了区块的输出数据：

```go
> db.blocks.findOne()
{
"_id" : ObjectId("595368fbcedea89d3f4fb0ca"),
"number_transactions" : 28,
"timestamp" : NumberLong("1498324744877"),
"gas_used" : 4694483,
"number_internal_transactions" : 4,
"block_hash" : "0x89d235c4e2e4e4978440f3cc1966f1ffb343b9b5cfec9e5cebc331fb810bded3",
"difficulty" : NumberLong("882071747513072"),
"block_height" : 3923788
}
```

样本交易文档具有以下字段：

+   交易哈希

+   它所属的区块高度

+   从哈希地址

+   到哈希地址

+   交易价值

+   交易费用

以下代码显示了交易的输出数据：

```go
> db.transactions.findOne()
{
"_id" : ObjectId("59535748cedea89997e8385a"),
"from" : "0x3c540be890df69eca5f0099bbedd5d667bd693f3",
"txfee" : 28594,
"timestamp" : ISODate("2017-06-06T11:23:10Z"),
"value" : 0,
"to" : "0x4b9e0d224dabcc96191cace2d367a8d8b75c9c81",
"txhash" : "0xf205991d937bcb60955733e760356070319d95131a2d9643e3c48f2dfca39e77",
"block" : 3923794
}
```

我们的数据库的样本数据可在 GitHub 上找到：[`github.com/PacktPublishing/Mastering-MongoDB-4.x-Second-Edition`](https://github.com/PacktPublishing/Mastering-MongoDB-4.x-Second-Edition)。

作为使用这种新型区块链技术的好奇开发人员，我们想要分析以太坊交易。我们特别希望做到以下几点：

+   找到交易发起的前十个地址

+   找到交易结束的前十个地址

+   找到每笔交易的平均值，并统计偏差

+   找到每笔交易所需的平均费用，并统计偏差

+   找到网络在一天中的哪个时间更活跃，根据交易的数量或价值

+   找到网络在一周中的哪一天更活跃，根据交易的数量或价值

我们找到了交易发起的前十个地址。为了计算这个指标，我们首先计算每个`from`字段的值为`1`的出现次数，然后按`from`字段的值对它们进行分组，并将它们输出到一个名为`count`的新字段中。

之后，我们按照`count`字段的值按降序（`-1`）排序，最后，我们将输出限制为通过管道的前十个文档。这些文档是我们正在寻找的前十个地址。

以下是一些示例 Python 代码：

```go
   def top_ten_addresses_from(self):
       pipeline = [
           {"$group": {"_id": "$from", "count": {"$sum": 1}}},
           {"$sort": SON([("count", -1)])},
           {"$limit": 10},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

前面代码的输出如下：

```go
{u'count': 38, u'_id': u'miningpoolhub_1'}
{u'count': 31, u'_id': u'Ethermine'}
{u'count': 30, u'_id': u'0x3c540be890df69eca5f0099bbedd5d667bd693f3'}
{u'count': 27, u'_id': u'0xb42b20ddbeabdc2a288be7ff847ff94fb48d2579'}
{u'count': 25, u'_id': u'ethfans.org'}
{u'count': 16, u'_id': u'Bittrex'}
{u'count': 8, u'_id': u'0x009735c1f7d06faaf9db5223c795e2d35080e826'}
{u'count': 8, u'_id': u'Oraclize'}
{u'count': 7, u'_id': u'0x1151314c646ce4e0efd76d1af4760ae66a9fe30f'}
{u'count': 7, u'_id': u'0x4d3ef0e8b49999de8fa4d531f07186cc3abe3d6e'}
```

现在我们找到了交易结束的前十个地址。就像我们对`from`所做的那样，对`to`地址的计算也完全相同，只是使用`to`字段而不是`from`进行分组，如下面的代码所示：

```go
   def top_ten_addresses_to(self):
       pipeline = [
           {"$group": {"_id": "$to", "count": {"$sum": 1}}},
           {"$sort": SON([("count", -1)])},
           {"$limit": 10},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

前面代码的输出如下：

```go
{u'count': 33, u'_id': u'0x6090a6e47849629b7245dfa1ca21d94cd15878ef'}
{u'count': 30, u'_id': u'0x4b9e0d224dabcc96191cace2d367a8d8b75c9c81'}
{u'count': 25, u'_id': u'0x69ea6b31ef305d6b99bb2d4c9d99456fa108b02a'}
{u'count': 23, u'_id': u'0xe94b04a0fed112f3664e45adb2b8915693dd5ff3'}
{u'count': 22, u'_id': u'0x8d12a197cb00d4747a1fe03395095ce2a5cc6819'}
{u'count': 18, u'_id': u'0x91337a300e0361bddb2e377dd4e88ccb7796663d'}
{u'count': 13, u'_id': u'0x1c3f580daeaac2f540c998c8ae3e4b18440f7c45'}
{u'count': 12, u'_id': u'0xeef274b28bd40b717f5fea9b806d1203daad0807'}
{u'count': 9, u'_id': u'0x96fc4553a00c117c5b0bed950dd625d1c16dc894'}
{u'count': 9, u'_id': u'0xd43d09ec1bc5e57c8f3d0c64020d403b04c7f783'}
```

让我们找到每笔交易的平均值，并统计标准偏差。在这个示例中，我们使用`$avg`和`$stdDevPop`操作符来计算`value`字段的统计数据。使用简单的`$group`操作，我们输出一个具有我们选择的 ID（这里是`value`）和`averageValues`的单个文档，如下面的代码所示：

```go
   def average_value_per_transaction(self):
       pipeline = [
           {"$group": {"_id": "value", "averageValues": {"$avg": "$value"}, "stdDevValues": {"$stdDevPop": "$value"}}},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

前面代码的输出如下：

```go
{u'averageValues': 5.227238976440972, u'_id': u'value', u'stdDevValues': 38.90322689649576}
```

让我们找到每笔交易所需的平均费用，返回有关偏差的统计数据。平均费用类似于平均值，只是将`$value`替换为`$txfee`，如下面的代码所示：

```go
   def average_fee_per_transaction(self):
       pipeline = [
           {"$group": {"_id": "value", "averageFees": {"$avg": "$txfee"}, "stdDevValues": {"$stdDevPop": "$txfee"}}},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

前面代码片段的输出如下：

```go
{u'_id': u'value', u'averageFees': 320842.0729166667, u'stdDevValues': 1798081.7305142984} 
```

我们找到网络在特定时间更活跃的时间。

为了找出交易最活跃的小时，我们使用`$hour`运算符从我们存储了`datetime`值并称为`timestamp`的`ISODate()`字段中提取`hour`字段，如下面的代码所示：

```go
   def active_hour_of_day_transactions(self):
       pipeline = [
           {"$group": {"_id": {"$hour": "$timestamp"}, "transactions": {"$sum": 1}}},
           {"$sort": SON([("transactions", -1)])},
           {"$limit": 1},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

输出如下：

```go
{u'_id': 11, u'transactions': 34} 
```

以下代码将计算一天中交易价值最高的小时的交易总值：

```go
  def active_hour_of_day_values(self):
 pipeline = [
 {"$group": {"_id": {"$hour": "$timestamp"}, "transaction_values": {"$sum": "$value"}}},
 {"$sort": SON([("transactions", -1)])},
 {"$limit": 1},
 ]
 result = self.collection.aggregate(pipeline)
 for res in result:
 print(res)
```

上述代码的输出如下：

```go
{u'transaction_values': 33.17773841, u'_id': 20} 
```

让我们找出网络活动最频繁的一天是一周中的哪一天，根据交易数量或交易价值。与一天中的小时一样，我们使用`$dayOfWeek`运算符从`ISODate()`对象中提取一周中的哪一天，如下面的代码所示。按照美国的惯例，星期天为一，星期六为七：

```go
   def active_day_of_week_transactions(self):
       pipeline = [
           {"$group": {"_id": {"$dayOfWeek": "$timestamp"}, "transactions": {"$sum": 1}}},
           {"$sort": SON([("transactions", -1)])},
           {"$limit": 1},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)

```

上述代码的输出如下：

```go
{u'_id': 3, u'transactions': 92} 
```

以下代码将计算一周中交易价值最高的一天的交易总值：

```go
  def active_day_of_week_values(self):
       pipeline = [
           {"$group": {"_id": {"$dayOfWeek": "$timestamp"}, "transaction_values": {"$sum": "$value"}}},
           {"$sort": SON([("transactions", -1)])},
           {"$limit": 1},
       ]
```

```go
       result = self.collection.aggregate(pipeline)
 for res in result:
 print(res)
```

上述代码的输出如下：

```go

 {u'transaction_values': 547.62439312, u'_id': 2} 
```

我们计算的聚合可以用以下图表描述：

![](img/e8fe830a-aef7-4dc6-b15a-60247711d69e.png)

在区块方面，我们想了解以下内容：

+   每个区块的平均交易数量，包括总体交易数量和合约内部交易的总体交易数量。

+   每个区块的平均燃气使用量。

+   每个交易到区块的平均燃气使用量。是否有机会在一个区块中提交我的智能合约？

+   每个区块的平均难度及其偏差。

+   每个区块的平均交易数量，总交易数量以及合约内部交易的平均交易数量。

通过对`number_transactions`字段进行平均，我们可以得到每个区块的交易数量，如下面的代码所示：

```go
   def average_number_transactions_total_block(self):
       pipeline = [
           {"$group": {"_id": "average_transactions_per_block", "count": {"$avg": "$number_transactions"}}},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

上述代码的输出如下：

```go
 {u'count': 39.458333333333336, u'_id': u'average_transactions_per_block'}

```

而使用以下代码，我们可以得到每个区块的内部交易的平均数量：

```go
  def average_number_transactions_internal_block(self):
       pipeline = [
           {"$group": {"_id": "average_transactions_internal_per_block", "count": {"$avg": "$number_internal_transactions"}}},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

上述代码的输出如下：

```go
{u'count': 8.0, u'_id': u'average_transactions_internal_per_block'}
```

每个区块使用的平均燃气量可以通过以下方式获得：

```go
def average_gas_block(self):
       pipeline = [
           {"$group": {"_id": "average_gas_used_per_block",
                       "count": {"$avg": "$gas_used"}}},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

输出如下：

```go
{u'count': 2563647.9166666665, u'_id': u'average_gas_used_per_block'} 
```

每个区块的平均难度及其偏差可以通过以下方式获得：

```go
  def average_difficulty_block(self):
       pipeline = [
           {"$group": {"_id": "average_difficulty_per_block",
                       "count": {"$avg": "$difficulty"}, "stddev": {"$stdDevPop": "$difficulty"}}},
       ]
       result = self.collection.aggregate(pipeline)
       for res in result:
           print(res)
```

输出如下：

```go
{u'count': 881676386932100.0, u'_id': u'average_difficulty_per_block', u'stddev': 446694674991.6385} 
```

我们的聚合描述如下模式：

![](img/1b5a0878-4238-4012-bca0-c395a951a7f9.png)

现在我们已经计算了基本统计数据，我们想要提高我们的水平，并了解有关我们的交易的更多信息。通过我们复杂的机器学习算法，我们已经确定了一些交易是欺诈或**首次代币发行**（ICO），或者两者兼而有之。

在这些文档中，我们已经在一个名为`tags`的数组中标记了这些属性，如下所示：

```go
{
 "_id" : ObjectId("59554977cedea8f696a416dd"),
 "to" : "0x4b9e0d224dabcc96191cace2d367a8d8b75c9c81",
 "txhash" : "0xf205991d937bcb60955733e760356070319d95131a2d9643e3c48f2dfca39e77",
 "from" : "0x3c540be890df69eca5f0099bbedd5d667bd693f3",
 "block" : 3923794,
 "txfee" : 28594,
 "timestamp" : ISODate("2017-06-10T09:59:35Z"),
 "tags" : [
 "scam",
 "ico"
 ],
 "value" : 0
 }
```

现在我们想要获取 2017 年 6 月的交易，移除`_id`字段，并根据我们已经识别的标签生成不同的文档。因此，在我们的示例中，我们将在我们的新集合`scam_ico_documents`中输出两个文档，以便进行单独处理。

通过聚合框架进行此操作的方式如下所示：

```go
def scam_or_ico_aggregation(self):
 pipeline = [
 {"$match": {"timestamp": {"$gte": datetime.datetime(2017,6,1), "$lte": datetime.datetime(2017,7,1)}}},
 {"$project": {
 "to": 1,
 "txhash": 1,
 "from": 1,
 "block": 1,
 "txfee": 1,
 "tags": 1,
 "value": 1,
 "report_period": "June 2017",
 "_id": 0,
 }

 },
 {"$unwind": "$tags"},
 {"$out": "scam_ico_documents"}
 ]
 result = self.collection.aggregate(pipeline)
 for res in result:
 print(res)
```

在聚合框架管道中，我们有以下四个不同的步骤：

1.  使用`$match`，我们只提取具有`timestamp`字段值为 2017 年 6 月 1 日的文档。

1.  使用`$project`，我们添加一个名为`report_period`的新字段，其值为`2017 年 6 月`，并通过将其值设置为`0`来移除`_id`字段。我们通过使用值`1`保持其余字段不变，如前面的代码所示。

1.  使用`$unwind`，我们在我们的`$tags`数组中为每个标签输出一个新文档。

1.  最后，使用`$out`，我们将所有文档输出到一个新的`scam_ico_documents`集合中。

由于我们使用了`$out`运算符，在命令行中将得不到任何结果。如果我们注释掉`{"$out": "scam_ico_documents"}`，我们将得到以下类似的文档：

```go
{u'from': u'miningpoolhub_1', u'tags': u'scam', u'report_period': u'June 2017', u'value': 0.52415349, u'to': u'0xdaf112bcbd38d231b1be4ae92a72a41aa2bb231d', u'txhash': u'0xe11ea11df4190bf06cbdaf19ae88a707766b007b3d9f35270cde37ceccba9a5c', u'txfee': 21.0, u'block': 3923785}
```

我们数据库中的最终结果将如下所示：

```go
{
 "_id" : ObjectId("5955533be9ec57bdb074074e"),
 "to" : "0x4b9e0d224dabcc96191cace2d367a8d8b75c9c81",
 "txhash" : "0xf205991d937bcb60955733e760356070319d95131a2d9643e3c48f2dfca39e77",
 "from" : "0x3c540be890df69eca5f0099bbedd5d667bd693f3",
 "block" : 3923794,
 "txfee" : 28594,
 "tags" : "scam",
 "value" : 0,
 "report_period" : "June 2017"
 }
```

现在，我们在`scam_ico_documents`集合中有了明确定义的文档，我们可以很容易地进行进一步的分析。这种分析的一个例子是在一些骗子上附加更多信息。幸运的是，我们的数据科学家已经提供了一些额外的信息，我们已经提取到一个新的集合`scam_details`中，它看起来是这样的：

```go
{
 "_id" : ObjectId("5955510e14ae9238fe76d7f0"),
 "scam_address" : "0x3c540be890df69eca5f0099bbedd5d667bd693f3",
 Email_address": example@scammer.com"
 }
```

现在，我们可以创建一个新的聚合管道作业，将我们的`scam_ico_documents`与`scam_details`集合连接起来，并将这些扩展结果输出到一个新的集合中，名为`scam_ico_documents_extended`，就像这样：

```go
def scam_add_information(self):
 client = MongoClient()
 db = client.mongo_book
 scam_collection = db.scam_ico_documents
 pipeline = [
 {"$lookup": {"from": "scam_details", "localField": "from", "foreignField": "scam_address", "as": "scam_details"}},
 {"$match": {"scam_details": { "$ne": [] }}},
 {"$out": "scam_ico_documents_extended"}
 ]
 result = scam_collection.aggregate(pipeline)
 for res in result:
 print(res)
```

在这里，我们使用以下三步聚合管道：

1.  使用`$lookup`命令，从`scam_details`集合和`scam_address`字段中的数据与我们的本地集合（`scam_ico_documents`）中的数据进行连接，基于本地集合属性`from`的值等于`scam_details`集合的`scam_address`字段中的值。如果它们相等，那么管道将在文档中添加一个名为`scam_details`的新字段。

1.  接下来，我们只匹配具有`scam_details`字段的文档，即与查找聚合框架步骤匹配的文档。

1.  最后，我们将这些文档输出到一个名为`scam_ico_documents_extended`的新集合中。

现在这些文档看起来是这样的：

```go
> db.scam_ico_documents_extended.findOne()
 {
 "_id" : ObjectId("5955533be9ec57bdb074074e"),
 "to" : "0x4b9e0d224dabcc96191cace2d367a8d8b75c9c81",
 "txhash" : "0xf205991d937bcb60955733e760356070319d95131a2d9643e3c48f2dfca39e77",
 "from" : "0x3c540be890df69eca5f0099bbedd5d667bd693f3",
 "block" : 3923794,
 "txfee" : 28594,
 "tags" : "scam",
 "value" : 0,
 "report_period" : "June 2017",
 "scam_details_data" : [
 {
 "_id" : ObjectId("5955510e14ae9238fe76d7f0"),
 "scam_address" : "0x3c540be890df69eca5f0099bbedd5d667bd693f3",
 email_address": example@scammer.com"
 }]}
```

使用聚合框架，我们已经确定了我们的数据，并且可以快速高效地处理它。

前面的步骤可以总结如下图所示：

![](img/0ce20f81-e8bb-4501-94a4-2a855c17c203.png)

# 总结

在本章中，我们深入探讨了聚合框架。我们讨论了为什么以及何时应该使用聚合，而不是简单地使用 MapReduce 或查询数据库。我们详细介绍了聚合的各种选项和功能。

我们讨论了聚合阶段和各种运算符，如布尔运算符、比较运算符、集合运算符、数组运算符、日期运算符、字符串运算符、表达式算术运算符、聚合累加器、条件表达式和变量，以及文字解析数据类型运算符。

使用以太坊用例，我们通过工作代码进行了聚合，并学习了如何解决工程问题。

最后，我们了解了聚合框架目前存在的限制以及何时应避免使用它。

在下一章中，我们将继续讨论索引的主题，并学习如何为我们的读写工作负载设计和实现高性能索引。
