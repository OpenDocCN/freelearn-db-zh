# 第八章：使用 MongoDB 进行日志记录和实时分析

在本书中，向您介绍了许多您已经了解的概念。您已经学会了如何将它们与 MongoDB 提供给我们的技术和工具结合使用。本章的目标是在一个真实的例子中应用这些技术。

我们将在本章中开发的真实例子解释如何使用 MongoDB 作为网络服务器日志数据的持久存储，更具体地说，是来自 Nginx 网络服务器的数据。通过这样做，我们将能够分析 Web 应用程序的流量数据。

我们将从分析 Nginx 日志格式开始，以定义对我们实验有用的信息。之后，我们将定义我们想要在 MongoDB 中执行的分析类型。最后，我们将设计我们的数据库模式，并通过使用代码在 MongoDB 中读写数据来实现。

在本章中，我们将考虑到每个生成此事件的主机都会消耗这些信息并将其发送到 MongoDB。我们的重点不在应用程序的架构或我们在示例中将生成的代码上。因此，亲爱的读者，如果您不同意这里显示的代码片段，请随意修改它们或自己创建一个新的。

也就是说，本章将涵盖：

+   日志数据分析

+   我们正在寻找的内容

+   设计模式

# 日志数据分析

访问日志经常被开发人员、系统管理员或任何在网络上保持服务的人忽视。但是，当我们需要及时了解每个请求在我们的网络服务器上发生了什么时，它是一个强大的工具。

访问日志保存有关服务器活动和性能的信息，还告诉我们有关可能问题。如今最常见的网络服务器是 Apache HTTPD 和 Nginx。这些网络服务器默认情况下有两种日志类型：错误日志和访问日志。

## 错误日志

正如其名称所示，错误日志是网络服务器在处理接收到的请求时发现的错误的存储位置。一般来说，这种类型的日志是可配置的，并且会根据预定义的严重级别写入消息。

## 访问日志

访问日志是存储所有接收和处理请求的地方。这将是我们研究的主要对象。

文件中记录的事件以预定义的布局记录，可以根据服务器管理者的愿望进行格式化。默认情况下，Apache HTTPD 和 Nginx 都有一个称为**combined**的格式。以这种格式生成的日志示例如下所示：

```sql
191.32.254.162 - - [29/Mar/2015:16:04:08 -0400] "GET /admin HTTP/1.1" 200 2529 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.104 Safari/537.36"

```

乍一看，一行日志中包含太多信息可能有点吓人。然而，如果我们看一下生成此日志的模式并尝试对其进行检查，我们会发现它并不难理解。

生成此行的 Nginx 网络服务器的模式如下所示：

```sql
$remote_addr - $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent"

```

我们将描述此模式的每个部分，以便您更好地理解：

+   `$remote_addr`：这是在网络服务器上执行请求的客户端的 IP 地址。在我们的示例中，该值对应于 191.32.254.162。

+   `$remote_user`：这是经过身份验证的用户，如果存在的话。当未识别经过身份验证的用户时，此字段将填写连字符。在我们的示例中，该值为`-`。

+   `[$time_local]`：这是请求在网络服务器上接收的时间，格式为`[day/month/year:hour:minute:second zone]`。

+   `"$request"`：这是客户端请求本身。也称为请求行。为了更好地理解，我们将分析示例中的请求行：`"GET /admin HTTP/1.1"`。首先，我们有客户端使用的 HTTP 动词。在这种情况下，它是一个`GET` HTTP 动词。接着，我们有客户端访问的资源。在这种情况下，访问的资源是`/admin`。最后，我们有客户端使用的协议。在这种情况下，是`HTTP/1.1`。

+   `$status`：这是 Web 服务器向客户端返回的 HTTP 状态码。该字段的可能值在 RFC 2616 中定义。在我们的示例中，Web 服务器向客户端返回状态码`200`。

### 注意

要了解有关 RFC 2616 的更多信息，您可以访问 [`www.w3.org/Protocols/rfc2616/rfc2616.txt`](http://www.w3.org/Protocols/rfc2616/rfc2616.txt)。

+   `$body_bytes_sent`：这是发送给客户端的响应正文的字节长度。当没有正文时，该值将是一个连字符。我们必须注意，该值不包括请求头。在我们的示例中，该值为 2,529 字节。

+   `"$http_referer"`：这是客户端请求头中“Referer”中包含的值。该值表示所请求资源的引用位置。当直接执行资源访问时，该字段填充一个连字符。在我们的示例中，该值为`-`。

+   `"$http_user_agent"`：这是客户端在 User-Agent 头中发送的信息。通常，我们可以在该头中识别请求中使用的网络浏览器。在我们的示例中，该字段的值为`"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2272.104 Safari/537.36"`。

除此之外，我们还有更多可用于创建新日志格式的变量。其中，我们要强调：

+   `$request_time`：这表示请求处理的总时间

+   `$request_length`：这表示客户端响应的总长度，包括头部。

现在我们熟悉了 Web 服务器访问日志，我们将定义我们想要分析的内容，以了解需要记录哪些信息。

# 我们正在寻找的内容

从 Web 服务器访问日志中提取的信息非常丰富，为我们提供了无限的研究可能性。简单直接，我们可以通过计算访问日志的行数来统计 Web 服务器接收的请求数。但我们可以扩展我们的分析，尝试测量数据流量的平均值，例如。

最近，最广泛使用的服务之一是应用性能管理系统，也称为 **APM**。如今，这些服务通常作为软件即服务提供，并且主要目标是为我们提供应用程序性能和健康状况的视图。

APM 是基于从访问日志中提取的信息进行分析的一个很好的例子，因为 APM 生成的大部分信息都是基于访问日志的。

注意！我并不是说 APM 仅基于访问日志工作，但 APM 生成的大部分信息可以从访问日志中提取。好吗？

### 注意

要了解更多信息，请访问 [`en.wikipedia.org/wiki/Application_performance_management`](https://en.wikipedia.org/wiki/Application_performance_management)。

正如本章开头所说，我们并没有打算编写或创建整个系统，但我们将在实践中展示如何使用 MongoDB 保留访问日志信息以供可能的分析。

基于 APM，我们将结构化我们的示例，分析 Web 服务器资源吞吐量。只需使用 Web 服务器访问日志中包含的信息就可以进行此分析。为此，我们的访问日志需要哪些数据？我们应该使用组合格式吗？

## 测量 Web 服务器上的流量

我们的 Web 服务器吞吐量将根据一定时间段内的请求数进行估算，即一天内的请求数，一小时内的请求数，一分钟内的请求数或一秒内的请求数。每分钟的请求数是实时监控的一个非常合理的度量。

吞吐量是通过计算在我们的 Web 服务器中处理的请求来计算的。因此，不需要使用访问日志中的特定数据。然而，为了使我们的数据能够进行更丰富的进一步分析，我们将创建一个特定的日志格式，其中将收集请求信息，如 HTTP 状态码、请求时间和长度。

Apache HTTP 和 Nginx 都允许我们自定义访问日志或创建一个具有自定义格式的新文件。第二个选项似乎是完美的。在开始配置 Web 服务器之前，我们将使用先前解释的变量创建我们的日志格式。只需记住我们正在使用 Nginx Web 服务器。

```sql
$remote_addr [$time_local] "$request" $status $request_time $request_length

```

当我们定义了我们的日志格式后，我们可以配置我们的 Nginx Web 服务器。为此，让我们执行以下步骤：

1.  首先，在 Nginx 中定义这种新格式，我们需要编辑`nginx.conf`文件，在 HTTP 元素中添加一个新的条目，其中包含新的日志格式：

```sql
log_format custom_format '$remote_addr [$time_local] "$request" $status $request_time $request_length';
```

1.  现在我们需要在`nginx.conf`文件中添加另一个条目，定义新的自定义日志将写入哪个文件：

```sql
access_log /var/log/nginx/custom_access.log custom_format;
```

1.  要应用我们的更改，请在终端中执行以下命令重新加载 Nginx Web 服务器：

```sql
/usr/sbin/nginx reload

```

1.  重新加载 Nginx 服务器后，我们可以查看我们的新日志文件`/var/log/nginx/custom_access.log`，并检查行是否像以下行一样：

```sql
191.32.254.162 [29/Mar/2015:18:35:26 -0400] "GET / HTTP/1.1" 200 0.755 802
```

配置了日志格式，Web 服务器设置好了；现在是设计我们的模式的时候了。

# 设计模式

在本书中已经多次重复的一件事是了解我们的数据及其用途的重要性。现在，比以往任何时候，在这个实际练习中，我们将逐步设计我们的模式，考虑其中涉及的每个细节。

在前一节中，我们定义了我们想要从 Web 服务器访问日志中提取的数据集。下一步是列出与数据库客户端相关的要求。如前所述，一个进程将负责从日志文件中捕获信息并将其写入 MongoDB，另一个进程将读取已在数据库中持久化的信息。

一个值得关注的问题是在 MongoDB 中写入文档时的性能，因为重要的是要确保信息几乎实时生成。由于我们没有对每秒数据量的先前估计，我们将持乐观态度。让我们假设我们将一直从 Web 服务器到我们的 MongoDB 实例中获得大量数据。

考虑到这一要求，我们将担心随着时间的推移数据维度将增加。更多的事件意味着将插入更多的文档。这些是我们的系统良好运行的主要要求。

乍一看，我们可以想象这一切都是关于定义文档格式并将其持久化到集合中。但这样想忽视了众所周知的 MongoDB 模式灵活性。因此，我们将分析吞吐量问题，以便定义在哪里以及如何持久化信息。

## 捕获事件请求

Web 服务器吞吐量分析可能是最简单的任务。简单地说，事件数量的测量将给我们一个代表 Web 服务器吞吐量的数字。

因此，如果对于每个生成的事件都执行了写文档操作，并声明了此操作的时间，这是否意味着我们可以轻松获得吞吐量？是的！因此，表示我们可以分析吞吐量的最简单的 MongoDB 文档的方式如下：

```sql
{

 "_id" : ObjectId("5518ce5c322fa17f4243f85f"),
 "request_line" : "191.32.254.162 [29/Mar/2015:18:35:26 -0400] \"GET /media/catalog/product/image/2.jpg HTTP/1.1\" 200 0.000 867"

}

```

在文档集合中执行`count`方法时，我们将获得 Web 服务器的吞吐量值。假设我们有一个名为`events`的集合，为了找出吞吐量，我们必须在 mongod shell 中执行以下命令：

```sql
db.events.find({}).count()

```

这个命令返回到目前为止在我们的 Web 服务器上生成的事件总数。但是，这是我们想要的数字吗？不是。在没有将其放在一定时间段内的情况下，拥有事件的总数是没有意义的。如果我们不知道我们何时开始记录这些事件，甚至最后一个事件生成的时间，那么到目前为止由 Web 服务器处理的 10,000 个事件有什么用呢？

如果我们想在一定时间段内计算事件的数量，最简单的方法是包括一个代表事件创建日期的字段。这个文档的一个例子如下所示：

```sql
{

 "_id" : ObjectId("5518ce5c322fa17f4243f85f"),
 "request_line" : "191.32.254.162 [29/Mar/2015:18:35:26 -0400] \"GET /media/catalog/product/image/2.jpg HTTP/1.1\" 0.000 867",
 "date_created" : ISODate("2015-03-30T04:17:32.246Z")

}

```

因此，我们可以通过执行查询来检查在给定时间段内 Web 服务器上的总请求数。执行这个查询的最简单方法是使用聚合框架。在 mongod shell 上执行以下命令将返回每分钟的总请求数：

```sql
db.events.aggregate(
{
 $group: {
 _id: {
 request_time: {
 month: {
 $month: "$date_created"
 },
 day: {
 $dayOfMonth: "$date_created"
 },
 year: {
 $year: "$date_created"
 },
 hour: {
 $hour: "$date_created"
 },
 min: {
 $minute: "$date_created"
 }
 }
 },
 count: {
 $sum: 1
 }
 }
})

```

### 注意

聚合管道有其限制。如果命令结果返回一个超过 BSON 文档大小的单个文档，就会产生错误。自 MongoDB 的 2.6 版本以来，`aggregate`命令返回一个游标，因此可以返回任意大小的结果集。

您可以在 MongoDB 参考手册的[`docs.mongodb.org/manual/core/aggregation-pipeline-limits/`](http://docs.mongodb.org/manual/core/aggregation-pipeline-limits/)找到更多关于聚合管道限制的信息。

在命令管道中，我们定义了`$group`阶段来按天、月、年、小时和分钟对文档进行分组。并且使用`$sum`运算符来计算所有内容。通过这个`aggregate`命令的执行，我们将得到如下的文档：

```sql
{
 "_id": {
 "request_time": {
 "month": 3,
 "day": 30,
 "year": 2015,
 "hour": 4,
 "min": 48
 }
 },
 "count": 50
}
{
 "_id": {
 "request_time": {
 "month": 3,
 "day": 30,
 "year": 2015,
 "hour": 4,
 "min": 38
 }
 },
 "count": 13
}
{
 "_id": {
 "request_time": {
 "month": 3,
 "day": 30,
 "year": 2015,
 "hour": 4,
 "min": 17
 }
 },
 "count": 26
}

```

在这个输出中，可以知道 Web 服务器在一定时间段内收到了多少请求。这是由于`$group`运算符的行为，它根据一个或多个字段收集与查询匹配的文档，并收集文档组。我们将聚合管道的组阶段的每个部分，如月、日、年、小时和分钟，都带入了我们的`$date_created`字段。

如果您想知道在您的 Web 服务器上哪个资源的吞吐量最高，但没有一个选项符合这个要求。然而，这个问题的快速解决方案很容易实现。乍一看，最快的方法是拆解事件并创建一个更复杂的文档，就像下面的例子所示：

```sql
{
 "_id" : ObjectId("5519baca82d8285709606ce9"),
 "remote_address" : "191.32.254.162",
 "date_created" : ISODate("2015-03-29T18:35:25Z"),
 "http_method" : "GET",
 "resource" : "/media/catalog/product/cache/1/image/200x267/9df78eab33525d08d6e5fb8d27136e95/2/_/2.jpg",
 "http_version" : "HTTP/1.1",
 "status": 200,
 "request_time" : 0,
 "request_length" : 867
}

```

通过使用这种文档设计，可以通过聚合框架知道每分钟的资源吞吐量：

```sql
db.events.aggregate([
 {
 $group: {
 _id: "$resource",
 hits: {
 $sum: 1
 }
 }
 },
 {
 $project: {
 _id: 0,
 resource: "$_id",
 throughput: {
 $divide: [
 "$hits",
 1440
 ]
 }
 }
 },
 {
 $sort: {
 throughput: -1
 }
 }
])

```

在前面的管道中，第一步是按资源分组，并计算在整个一天内资源请求发生的次数。下一步是使用`$project`运算符，并且与`$divide`运算符一起使用给定资源中的点击次数，并通过 1,440 分钟进行平均计算，即一天或 24 小时的总分钟数。最后，我们按降序排序结果，以查看哪些资源具有更高的吞吐量。

为了保持清晰，我们将逐步执行管道并解释每个步骤的结果。在第一阶段的执行中，我们有以下结果：

```sql
db.events.aggregate([{$group: {_id: "$resource", hits: {$sum: 1}}}])

```

这个执行通过字段资源对事件集合文档进行分组，并计算使用`$sum`和值`1`时字段点击的发生次数。返回的结果如下所示：

```sql
{ "_id" : "/", "hits" : 5201 }
{ "_id" : "/legal/faq", "hits" : 1332 }
{ "_id" : "/legal/terms", "hits" : 3512 }

```

在管道的第二阶段，我们使用`$project`运算符，它将给出每分钟的点击次数：

```sql
db.documents.aggregate([
 {
 $group: {
 _id: "$resource",
 hits: {
 $sum: 1
 }
 }
 },
 {
 $project: {
 _id: 0,
 resource: "$_id",
 throughput: {
 $divide: [
 "$hits",
 1440
 ]
 }
 }
 }
])

```

以下是这个阶段的结果：

```sql
{ "resource" : "/", "throughput" : 3.6118055555555557 }
{ "resource" : "/legal/faq", "throughput" : 0.925 }
{ "resource" : "/legal/terms", "throughput" : 2.438888888888889 }

```

管道的最后阶段是按吞吐量降序排序结果：

```sql
db.documents.aggregate([
 {
 $group: {
 _id: "$resource",
 hits: {
 $sum: 1
 }
 }
 },
 {
 $project: {
 _id: 0,
 resource: "$_id",
 throughput: {
 $divide: [
 "$hits",
 1440
 ]
 }
 }
 },
 {
 $sort: {
 throughput: -1
 }
 }
])

```

产生的输出如下：

```sql
{ "resource" : "/", "throughput" : 3.6118055555555557 }
{ "resource" : "/legal/terms", "throughput" : 2.438888888888889 }
{ "resource" : "/legal/faq", "throughput" : 0.925 }

```

看起来我们成功地获得了我们文档的良好设计。现在我们可以提取所需的分析和其他分析，正如我们将在后面看到的那样，因此我们现在可以停下来。错了！我们将审查我们的要求，将它们与我们设计的模型进行比较，并尝试弄清楚是否这是最佳解决方案。

我们的愿望是知道每分钟对所有网络服务器资源的吞吐量。在我们设计的模型中，我们的网络服务器每个事件创建一个文档，通过使用聚合框架，我们可以计算我们分析所需的信息。

这个解决方案有什么问题？嗯，如果你认为是集合中的文档数量，那么你是对的。每个事件一个文档可能会生成巨大的集合，这取决于网络服务器的流量。显然，我们可以采用使用分片的策略，并将集合分布到许多主机上。但首先，我们将看看如何利用 MongoDB 中的模式灵活性来减少集合大小并优化查询。

## 一个文档解决方案

如果我们考虑到我们将有大量信息来创建分析，那么每个事件一个文档可能是有利的。但在我们试图解决的例子中，为每个 HTTP 请求持久化一个文档是昂贵的。

我们将利用 MongoDB 中的模式灵活性，这将帮助我们随着时间的推移增加文档。以下提案的主要目标是减少持久化文档的数量，同时优化我们集合中的读取和写入操作的查询。

我们正在寻找的文档应该能够为我们提供所有所需的信息，以便了解每分钟请求的资源吞吐量；因此我们可以有一个具有这种结构的文档：

+   一个带有资源的字段

+   一个带有事件日期的字段

+   事件发生的分钟和总点击数

以下文档实现了前面列表中描述的所有要求：

```sql
{
 "_id" : ObjectId("552005f5e2202a2f6001d7b0"),
 "resource" : "/",
 "date" : ISODate("2015-05-02T03:00:00Z"),
 "daily" : 215840,
 "minute" : {
 "0" : 90,

 "1" : 150,
 "2" : 143,
 ...
 "1349": 210
 }
}

```

有了这个文档设计，我们可以检索在某个资源上每分钟发生的事件数量。我们还可以通过每日字段知道一天内的总请求数，并使用这个来计算我们想要的任何内容，比如每分钟的请求或每小时的请求。

为了演示我们可以在这个集合上进行的写入和读取操作，我们将利用在 Node.js 平台上运行的 JavaScript 代码。因此，在继续之前，我们必须确保我们的机器上已经安装了 Node.js。

### 注意

如果您需要帮助，您可以在[`nodejs.org`](http://nodejs.org)找到更多信息。

我们应该做的第一件事是创建一个我们的应用程序将驻留的目录。在终端中，执行以下命令：

```sql
mkdir throughput_project

```

接下来我们转到我们创建的目录并启动项目：

```sql
cd throughput_project
npm init

```

回答向导提出的所有问题，以创建我们新项目的初始结构。目前，我们将根据我们给出的答案拥有一个基于`package.json`文件。

下一步是为我们的项目设置 MongoDB 驱动程序。我们可以通过编辑`package.json`文件来实现这一点，包括其依赖项的驱动程序引用，或者通过执行以下命令来实现：

```sql
npm install mongodb --save

```

上述命令将为我们的项目安装 MongoDB 驱动程序并将引用保存在`package.json`文件中。我们的文件应该是这样的：

```sql
{
  "name": "throughput_project",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Wilson da Rocha França",
  "license": "ISC",
  "dependencies": {
    "mongodb": "².0.25"
  }
}
```

最后一步是创建带有我们示例代码的`app.js`文件。以下是一个示例代码，向我们展示了如何在我们的网络服务器上计算事件的数量并将其记录在我们的集合中：

```sql
var fs = require('fs');
var util = require('util');
var mongo = require('mongodb').MongoClient;
var assert = require('assert');

// Connection URL
var url = 'mongodb://127.0.0.1:27017/monitoring;
// Create the date object and set hours, minutes,
// seconds and milliseconds to 00:00:00.000
var today = new Date();
today.setHours(0, 0, 0, 0);

var logDailyHit = function(db, resource, callback){
 // Get the events collection
  var collection = db.collection('events');
 // Update daily stats
  collection.update({resource: resource, date: today},
    {$inc : {daily: 1}}, {upsert: true},
    function(error, result){
      assert.equal(error, null);
      assert.equal(1, result.result.n);
      console.log("Daily Hit logged");
      callback(result);
  });
}

var logMinuteHit = function(db, resource, callback) {
 // Get the events collection
  var collection = db.collection('events');
 // Get current minute to update
  var currentDate = new Date();
  var minute = currentDate.getMinutes();
  var hour = currentDate.getHours();
 // We calculate the minute of the day
  var minuteOfDay = minute + (hour * 60);
  var minuteField = util.format('minute.%s', minuteOfDay);
 // Create a update object
  var update = {};
  var inc = {};
  inc[minuteField] = 1;
  update['$inc'] = inc;

 // Update minute stats
  collection.update({resource: resource, date: today},
    update, {upsert: true}, function(error, result){
      assert.equal(error, null);
      assert.equal(1, result.result.n);
      console.log("Minute Hit logged");
      callback(result);
  });
}

// Connect to MongoDB and log
mongo.connect(url, function(err, db) {
  assert.equal(null, err);
  console.log("Connected to server");
  var resource = "/";
  logDailyHit(db, resource, function() {
    logMinuteHit(db, resource, function(){
      db.close();
      console.log("Disconnected from server")
      });
    });
});
```

上面的示例代码非常简单。其中，我们有`logDailyHit`函数，负责记录事件并在文档`daily`字段中递增一个单位。第二个函数是`logMinuteHit`函数，负责记录事件的发生并递增代表当天当前分钟的文档`minute`字段。这两个函数都有一个更新查询，如果文档不存在，则具有值为`true`的`upsert`选项，那么将创建文档。

当我们执行以下命令时，将在资源`"/"`上记录一个事件。要运行代码，只需转到项目目录并执行以下命令：

```sql
node app.js

```

如果一切正常，运行命令后我们应该看到以下输出：

```sql
Connected to server
Daily Hit logged
Minute Hit logged
Disconnected from server

```

为了感受一下，我们将在 mongod shell 上执行`findOne`命令并观察结果：

```sql
db.events.findOne()
{
 "_id" : ObjectId("5520ade00175e1fb3361b860"),
 "resource" : "/",
 "date" : ISODate("2015-04-04T03:00:00Z"),
 "daily" : 383,
 "minute" : {
 "0" : 90,
 "1" : 150,
 "2" : 143
 }
}

```

除了之前的模型可以给我们的一切，这个模型还有一些优点。我们注意到的第一件事是，每当我们在 Web 服务器上注册一个新事件发生时，我们只会操作一个文档。另一个优点也在于，我们可以很容易地找到我们正在寻找的信息，给定一个特定的资源，因为我们在一个文档中有一整天的信息，这将导致我们在每次查询中操作更少的文档。

这个模式设计处理时间的方式将在我们考虑报告时给我们带来很多好处。无论是历史还是实时分析，都可以很容易地从这个集合中提取出文本和图形表示。

然而，与之前的方法一样，我们也将不得不处理一些限制。正如我们所见，我们在事件文档中同时增加了`daily`字段和`minute`字段，因为它们在 Web 服务器上发生。当某一天没有资源上报事件时，将创建一个新文档，因为我们在更新查询中使用了`upsert`选项。如果在给定的分钟内首次发生资源事件，同样的事情也会发生——`$inc`运算符将创建新的`minute`字段，并将`"1"`设置为值。这意味着我们的文档会随着时间的推移而增长，并且将超过 MongoDB 最初为其分配的大小。MongoDB 在文档分配的空间满时会自动执行重新分配操作。整天发生的这种重新分配操作直接影响了数据库的性能。

我们应该怎么办？就这样接受吗？不。我们可以通过添加一个预分配空间的过程来减少重新分配操作的影响。总之，我们将让应用程序负责创建一个包含一天内所有分钟的文档，并将每个字段初始化为值`0`。通过这样做，我们将避免 MongoDB 在一天内进行过多的重新分配操作。

### 注意

要了解更多关于记录分配策略的信息，请访问 MongoDB 参考用户手册[`docs.mongodb.org/manual/core/storage/#record-allocation-strategies`](http://docs.mongodb.org/manual/core/storage/#record-allocation-strategies)。

举个例子，我们可以在`app.js`文件中创建一个新的函数来预分配文档空间：

```sql
var fs = require('fs');
var util = require('util');
var mongo = require('mongodb').MongoClient;
var assert = require('assert');

// Connection URL
var url = 'mongodb://127.0.0.1:27017/monitoring';

var preAllocate = function(db, resource, callback){
 // Get the events collection
 var collection = db.collection('events');
 var now = new Date();
 now.setHours(0,0,0,0);
 // Create the minute document
 var minuteDoc = {};
 for(i = 0; i < 1440; i++){
 minuteDoc[i] = 0;
 }
 // Update minute stats
 collection.update(
 {resource: resource,
 date: now,
 daily: 0},
 {$set: {minute: minuteDoc}},
 {upsert: true}, function(error, result){
 assert.equal(error, null);
 assert.equal(1, result.result.n);
 console.log("Pre-allocated successfully!");
 callback(result);
 });
}

// Connect to MongoDB and log
mongo.connect(url, function(err, db) {
 assert.equal(null, err);
 console.log("Connected to server");
 var resource = "/";
 preAllocate(db, resource, function(){
 db.close();
 console.log("Disconnected from server")
 });
});

```

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载示例代码文件，用于您购买的所有 Packt Publishing 图书。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接发送到您的电子邮件。

要在当前日期为`"/"`资源预分配空间，只需运行以下命令：

```sql
node app.js

```

执行的输出类似于这样：

```sql
Connected to server
Pre-allocated successfully!
Disconnected from server

```

我们可以在 mongod shell 上运行`findOne`命令来检查新文档。由于创建的文档非常长，所以我们只展示其中的一部分：

```sql
db.events.findOne();
{
 "_id" : ObjectId("551fd893eb6efdc4e71260a0"),
 "daily" : 0,
 "date" : ISODate("2015-04-06T03:00:00Z"),
 "resource" : "/",
 "minute" : {
 "0" : 0,
 "1" : 0,
 "2" : 0,
 "3" : 0,
 ...
 "1439" : 0,
 }
}

```

建议我们在午夜前预先分配文档，以确保应用程序的顺利运行。如果我们安排创建这个文档并留有适当的安全边界，我们就不会有在午夜后发生事件的风险。

好了，现在问题解决了，我们可以回到引发我们文档重新设计的问题：数据的增长。

即使将我们收集的文档数量减少到每天每个事件一个文档，我们仍然可能会遇到存储空间的问题。当我们的网络服务器上有太多资源接收事件时，我们无法预测应用程序生命周期中会有多少新资源。为了解决这个问题，我们将使用两种不同的技术：TTL 索引和分片。

## TTL 索引

并不总是需要将所有日志信息永久存储在服务器上。限制存储在磁盘上的文件数量已经成为运维人员的标准做法。

同样的道理，我们可以限制我们需要存储在集合中的文档数量。为了实现这一点，我们可以在`date`字段上创建一个 TTL 索引，指定一个文档在集合中存在多长时间。只要记住，一旦创建了 TTL 索引，MongoDB 将自动从集合中删除过期的文档。

假设事件命中信息只在一年内有用。我们将在`date`字段上创建一个索引，属性为`expireAfterSeconds`，值为`31556926`，对应一年的秒数。

在 mongod shell 上执行以下命令，为我们的事件集合创建索引：

```sql
db.monitoring.createIndex({date: 1}, {expireAfterSeconds: 31556926})

```

如果索引不存在，输出应该是这样的：

```sql
{
 "createdCollectionAutomatically" : false,
 "numIndexesBefore" : 1,
 "numIndexesAfter" : 2,
 "ok" : 1
}

```

一旦完成这一步，我们的文档将根据日期字段在我们的集合中存活一年，之后 MongoDB 将自动删除它们。

## 分片

如果你是那些拥有无限资源并希望在磁盘上存储大量信息的人之一，那么缓解存储空间问题的一个解决方案是通过分片来分布数据。

正如我们之前所说，当我们选择分片键时，我们应该增加我们的努力，因为通过分片键，我们将确保我们的读写操作将由分片平均分布，也就是说，一个查询将针对集群上的一个分片或几个分片。

一旦我们完全控制了我们在网络服务器上拥有多少资源（或页面）以及这个数字将如何增长或减少，资源名称就成为一个很好的分片键选择。然而，如果我们有一个资源比其他资源有更多的请求（或事件），那么我们将有一个负载过重的分片。为了避免这种情况，我们将包括日期字段来组成分片键，这也将使我们在包含该字段的查询执行中获得更好的性能。

记住：我们的目标不是解释分片集群的设置。我们将向您介绍分片我们的集合的命令，考虑到您之前已经创建了分片集群。

为了使用我们选择的分片键对事件集合进行分片，我们将在 mongos shell 上执行以下命令：

```sql
mongos> sh.shardCollection("monitoring.events", {resource: 1, date: 1})

```

预期的输出是：

```sql
{ "collectionsharded" : "monitoring.events", "ok" : 1 }

```

### 提示

如果我们的事件集合中有任何文档，我们需要在分片集合之前创建一个索引，其中分片键是分片的前缀。要创建索引，请执行以下命令：

```sql
db.monitoring.createIndex({resource: 1, date: 1})
```

启用了分片的集合将使我们能够在事件集合中存储更多数据，并在数据增长时提高性能。

既然我们已经设计好了我们的文档并准备好我们的集合来接收大量数据，让我们执行一些查询！

## 查询报告

到目前为止，我们一直把精力集中在将数据存储在我们的数据库中。这并不意味着我们不关心读取操作。我们所做的一切都是通过概述我们应用程序的概况，并试图满足所有要求，为我们的数据库做好准备，迎接任何可能出现的情况。

因此，我们现在将说明我们有多少种可能性来查询我们的集合，以便基于存储的数据构建报告。

如果我们需要的是关于资源总点击量的实时信息，我们可以使用我们的每日字段来查询数据。有了这个字段，我们可以确定在一天中特定时间的资源总点击量，甚至可以根据一天中的分钟数确定资源的平均每分钟请求次数。

要查询基于当天时间的总点击量，我们将创建一个名为`getCurrentDayhits`的新函数，并且为了查询一天内每分钟的平均请求次数，我们将在`app.js`文件中创建`getCurrentMinuteStats`函数：

```sql
var fs = require('fs');
var util = require('util');
var mongo = require('mongodb').MongoClient;
var assert = require('assert');

// Connection URL
var url = 'mongodb://127.0.0.1:27017/monitoring';

var getCurrentDayhitStats = function(db, resource, callback){
 // Get the events collection
  var collection = db.collection('events');
  var now = new Date();
  now.setHours(0,0,0,0);
  collection.findOne({resource: "/", date: now},
    {daily: 1}, function(err, doc) {
    assert.equal(err, null);
    console.log("Document found.");
    console.dir(doc);
    callback(doc);
  });
}

var getCurrentMinuteStats = function(db, resource, callback){
 // Get the events collection
  var collection = db.collection('events');
  var now = new Date();
 // get hours and minutes and hold
  var hour = now.getHours()
  var minute = now.getMinutes();
 // calculate minute of the day to create field name
  var minuteOfDay = minute + (hour * 60);
  var minuteField = util.format('minute.%s', minuteOfDay);
 // set hour to zero to put on criteria
  now.setHours(0, 0, 0, 0);
 // create the project object and set minute of the day value
  var project = {};
  project[minuteField] = 1;
  collection.findOne({resource: "/", date: now},
    project, function(err, doc) {
    assert.equal(err, null);
    console.log("Document found.");
    console.dir(doc);
    callback(doc);
  });
}

// Connect to MongoDB and log
mongo.connect(url, function(err, db) {
  assert.equal(null, err);
  console.log("Connected to server");
  var resource = "/";
  getCurrentDayhitStats(db, resource, function(){
    getCurrentMinuteStats(db, resource, function(){
      db.close();
      console.log("Disconnected from server");
    });
  });
});
```

要看到魔术发生，我们应该在终端中运行以下命令：

```sql
node app.js

```

如果一切正常，输出应该是这样的：

```sql
Connected to server
Document found.
{ _id: 551fdacdeb6efdc4e71260a2, daily: 27450 }
Document found.
{ _id: 551fdacdeb6efdc4e71260a2, minute: { '183': 142 } }
Disconnected from server

```

另一个可能性是每天检索信息，计算资源每分钟的平均请求次数，或者在两个日期之间获取数据集以构建图表或表格。

以下代码有两个新函数，`getAverageRequestPerMinuteStats`，用于计算资源每分钟的平均请求次数，以及`getBetweenDatesDailyStats`，展示了如何在两个日期之间检索数据集。让我们看看`app.js`文件是什么样子的：

```sql
var fs = require('fs');
var util = require('util');
var mongo = require('mongodb').MongoClient;
var assert = require('assert');

// Connection URL
var url = 'mongodb://127.0.0.1:27017/monitoring';

var getAverageRequestPerMinuteStats = function(db, resource, callback){
 // Get the events collection
  var collection = db.collection('events');
  var now = new Date();
 // get hours and minutes and hold
  var hour = now.getHours()
  var minute = now.getMinutes();
 // calculate minute of the day to get the avg
  var minuteOfDay = minute + (hour * 60);
 // set hour to zero to put on criteria
  now.setHours(0, 0, 0, 0);
 // create the project object and set minute of the day value
  collection.findOne({resource: resource, date: now},
    {daily: 1}, function(err, doc) {
    assert.equal(err, null);
    console.log("The avg rpm is: "+doc.daily / minuteOfDay);
    console.dir(doc);
    callback(doc);
  });
}

var getBetweenDatesDailyStats = function(db, resource, dtFrom, dtTo, callback){
 // Get the events collection
  var collection = db.collection('events');
 // set hours for date parameters
  dtFrom.setHours(0,0,0,0);
  dtTo.setHours(0,0,0,0);
  collection.find({date:{$gte: dtFrom, $lte: dtTo}, resource: resource},
  {date: 1, daily: 1},{sort: [['date', 1]]}).toArray(function(err, docs) {
    assert.equal(err, null);
    console.log("Documents founded.");
    console.dir(docs);
    callback(docs);
  });
}

// Connect to MongoDB and log
mongo.connect(url, function(err, db) {
  assert.equal(null, err);
  console.log("Connected to server");
  var resource = "/";
  getAverageRequestPerMinuteStats(db, resource, function(){
    var now = new Date();
    var yesterday = new Date(now.getTime());
    yesterday.setDate(now.getDate() -1);
    getBetweenDatesDailyStats(db, resource, yesterday, now, function(){
      db.close();
      console.log("Disconnected from server");
    });

  });
});
```

正如你所看到的，有许多方法可以查询`events`集合中的数据。这些是一些非常简单的提取数据的例子，但它们是功能齐全且可靠的。

# 总结

本章向您展示了一个从零开始设计模式的过程的示例，以解决一个现实生活中的问题。我们从一个详细的问题及其要求开始，演变出了更好地利用可用资源的模式设计。基于这个问题的示例代码非常简单，但将为您终身学习提供基础。太棒了！

在这最后一章中，我们有机会在短短几页内回顾本书的前几章，并应用沿途介绍的概念。但是，正如你现在可能已经意识到的那样，MongoDB 是一个充满可能性的年轻数据库。它在社区中的采用率——包括你自己在内——随着每一次新发布而增加。因此，如果你发现自己面临一个新的挑战，意识到有不止一个解决方案，进行任何你认为必要或有用的测试。同事们也可以帮助，所以和他们交流。并且永远记住，一个好的设计是符合你需求的设计。
