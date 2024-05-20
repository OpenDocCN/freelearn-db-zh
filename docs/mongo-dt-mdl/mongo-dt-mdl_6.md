# 第六章：管理数据

- 计划数据库操作是数据模型维护中最重要的阶段之一。在 MongoDB 中，根据数据的性质，我们可以通过功能或地理分组来隔离应用程序的操作。

在本章中，我们将回顾一些在第五章中已经介绍的概念，如读取偏好和写入关注。但这次我们将专注于理解这些功能如何帮助我们通过 MongoDB 部署分割操作，例如，分离读取和写入操作，或者考虑应用程序特性，通过副本集节点进行写入传播来确保信息一致性。

您还将了解如何通过探索特殊属性来支持高读/写吞吐量的集合，这对某些应用程序至关重要。

因此，在本章中，您将了解：

+   操作隔离

+   有限集合

+   - 数据自动过期

# 操作隔离

到目前为止，我们已经看到我们应用程序的查询如何影响了我们对文档设计的决策。然而，读取偏好和写入关注概念还有更多内容需要探讨。

MongoDB 为我们提供了一系列功能，允许我们通过功能或地理分组来隔离应用程序操作。在使用功能隔离时，我们可以指示负责报告生成的应用程序仅使用特定的 MongoDB 部署。地理隔离意味着我们可以针对距离 MongoDB 部署的地理距离来定位操作。

## - 优先考虑读操作

可以想象一旦构建了一个应用程序，营销或商业人员将要求提供应用程序数据的新报告，顺便说一句，这将是必不可少的报告。我们知道为了报告的目的而在我们的主数据库中构建和插入这样的应用程序是多么危险。除了与其他应用程序的数据并发性外，我们知道这种类型的应用程序可能通过进行复杂查询和操作大量数据来过载我们的数据库。

这就是为什么我们必须将处理大量数据并需要数据库更重的处理的操作定位到专用的 MongoDB 部署。我们将通过读取偏好使应用程序定位到正确的 MongoDB 部署，就像您在第五章中看到的那样，*优化查询*。

默认情况下，应用程序将始终从我们的副本集中读取第一个节点。这种行为确保应用程序始终读取最新的数据，从而确保数据的一致性。但是，如果意图是减少第一个节点的吞吐量，并且我们可以接受最终一致性，可以通过启用`secondary`或`secondaryPreferred`模式将读操作重定向到副本集中的辅助节点。

- 除了在主节点上减少吞吐量的功能之外，在次要节点中优先考虑读操作对于分布在多个数据中心的应用程序至关重要，因此我们在地理上分布了副本集。这是因为我们可以通过设置最近模式选择最近的节点或延迟最低的节点来执行读操作。

- 最后，通过使用`primaryPreferred`模式，我们可以大大提高数据库的可用性，允许读操作在任何副本集节点中执行。

但是，除了读取偏好规范，主要或次要，如果我们还可以指定将操作定位到哪个实例呢？例如，考虑一个分布在两个不同位置的副本集，每个实例都有不同类型的物理存储。除此之外，我们希望确保写操作将在至少一个具有**ssd**磁盘的每个数据中心的实例中执行。这是可能的吗？答案是*是*！

这是由于**标签集**。标签集是一个配置属性，可以控制副本集的写关注和读偏好。它们由一个包含零个或多个标签的文档组成。我们将把这个配置存储在副本集配置文档的`members[n].tags`字段中。

在读取偏好的情况下，标签集允许您为副本集的特定成员定位读取操作。当选择读取过程的副本集成员时，标签集值将被应用。

标签集只会影响读取偏好模式之一，即`primaryPreferred`、`secondary`、`secondaryPreferred`和`nearest`。标签集不会影响`primary`模式，这意味着它只会影响副本集次要成员的选择，除非与`nearest`模式结合使用，在这种情况下，最接近的节点或延迟最小的节点可以成为主节点。

在看如何进行此配置之前，您需要了解副本集成员是如何选择的。将执行操作的客户端驱动程序进行选择，或者在分片集群的情况下，选择是由**mongos**实例完成的。

因此，选择过程是这样进行的：

1.  创建主要和次要成员的列表。

1.  如果指定了标签集，则不符合规范的成员将被跳过。

1.  确定最接近应用程序的客户端。

1.  创建其他副本集成员的列表，考虑其他成员之间的延迟。此延迟可以在通过`secondaryAcceptableLatencyMS`属性执行写操作时定义。在分片集群的情况下，可以通过`--localThreshold`或`localPingThresholdMs`选项进行设置。如果没有设置这些配置中的任何一个，那么默认值将为 15 毫秒。

### 提示

您可以在 MongoDB 手册参考中找到有关此配置的更多信息[`docs.mongodb.org/manual/reference/configuration-options/#replication.localPingThresholdMs`](http://docs.mongodb.org/manual/reference/configuration-options/#replication.localPingThresholdMs)。

1.  将随机选择要执行操作的主机，并执行读操作。

标签集配置与任何其他 MongoDB 配置一样简单。与往常一样，我们使用文档来创建配置，并且如前所述，标签集是副本集配置文档的一个字段。可以通过在副本集成员上运行`conf()`方法来检索此配置文档。

### 提示

您可以在 MongoDB 文档中找到有关`conf()`方法的更多信息[`docs.mongodb.org/manual/reference/method/rs.conf/#rs.conf`](http://docs.mongodb.org/manual/reference/method/rs.conf/#rs.conf)。

以下文件显示了在`rs1`的 mongod shell 上执行`rs.conf()`命令后，读操作的标签集示例，这是我们副本集的主节点。

```sql
rs1:PRIMARY> rs.conf()
{ // This is the replica set configuration document

 "_id" : "rs1",
 "version" : 4,
 "members" : [
 {
 "_id" : 0,
 "host" : "172.17.0.2:27017"
 },
 {
 "_id" : 1,
 "host" : "172.17.0.3:27017"
 },
 {
 "_id" : 2,
 "host" : "172.17.0.4:27017"
 }
 ]
}

```

要为副本集的每个节点创建标签集配置，我们必须在主要的 mongod shell 中执行以下命令序列：

首先，我们将获取副本集配置文档并将其存储在`cfg`变量中：

```sql
rs1:PRIMARY> cfg = rs.conf()
{
 "_id" : "rs1",
 "version" : 4,
 "members" : [
 {
 "_id" : 0,
 "host" : "172.17.0.7:27017"
 },
 {
 "_id" : 1,
 "host" : "172.17.0.5:27017"
 },
 {
 "_id" : 2,
 "host" : "172.17.0.6:27017"
 }
 ]
}

```

然后，通过使用`cfg`变量，我们将为我们的三个副本集成员中的每一个设置一个文档作为`members[n].tags`字段的新值：

```sql
rs1:PRIMARY> cfg.members[0].tags = {"media": "ssd", "application": "main"}
rs1:PRIMARY> cfg.members[1].tags = {"media": "ssd", "application": "main"}
rs1:PRIMARY> cfg.members[2].tags = {"media": "ssd", "application": "report"}

```

最后，我们调用`reconfig()`方法，传入存储在`cfg`变量中的新配置文档以重新配置我们的副本集：

```sql
rs1:PRIMARY> rs.reconfig(cfg)

```

如果一切正确，我们必须在 mongod shell 中看到这个输出：

```sql
{ "ok" : 1 }

```

要检查配置，我们可以重新执行命令`rs.conf()`。这将返回以下内容：

```sql
rs1:PRIMARY> cfg = rs.conf()
{
 "_id" : "rs1",
 "version" : 5,
 "members" : [
 {
 "_id" : 0,
 "host" : "172.17.0.7:27017",
 "tags" : {
 "application" : "main",
 "media" : "ssd"
 }
 },
 {
 "_id" : 1,
 "host" : "172.17.0.5:27017",
 "tags" : {
 "application" : "main",
 "media" : "ssd"
 }
 },
 {
 "_id" : 2,
 "host" : "172.17.0.6:27017",
 "tags" : {
 "application" : "report",
 "media" : "ssd"
 }
 }
 ]
}

```

现在，考虑以下`customer`集合：

```sql
{
 "_id": ObjectId("54bf0d719a5bc523007bb78f"),
 "username": "customer1",
 "email": "customer1@customer.com",
 "password": "1185031ff57bfdaae7812dd705383c74",
 "followedSellers": [
 "seller3",
 "seller1"
 ]
}
{
 "_id": ObjectId("54bf0d719a5bc523007bb790"),
 "username": "customer2",
 "email": "customer2@customer.com",
 "password": "6362e1832398e7d8e83d3582a3b0c1ef",
 "followedSellers": [
 "seller2",
 "seller4"
 ]
}
{
 "_id": ObjectId("54bf0d719a5bc523007bb791"),
 "username": "customer3",
 "email": "customer3@customer.com",
 "password": "f2394e387b49e2fdda1b4c8a6c58ae4b",
 "followedSellers": [
 "seller2",
 "seller4"
 ]
}
{
 "_id": ObjectId("54bf0d719a5bc523007bb792"),
 "username": "customer4",
 "email": "customer4@customer.com",
 "password": "10619c6751a0169653355bb92119822a",
 "followedSellers": [
 "seller1",
 "seller2"
 ]
}
{
 "_id": ObjectId("54bf0d719a5bc523007bb793"),
 "username": "customer5",
 "email": "customer5@customer.com",
 "password": "30c25cf1d31cbccbd2d7f2100ffbc6b5",
 "followedSellers": [
 "seller2",
 "seller4"
 ]
}

```

接下来的读操作将使用我们副本集实例中创建的标签：

```sql
db.customers.find(
 {username: "customer5"}
).readPref(
 {
 tags: [{application: "report", media: "ssd"}]
 }
)
db.customers.find(
 {username: "customer5"}
).readPref(
 {
 tags: [{application: "main", media: "ssd"}]
 }
)

```

前面的配置是*按应用操作分离*的一个例子。我们创建了标签集，标记了应用的性质以及将要读取的媒体类型。

正如我们之前所看到的，当我们需要在地理上分离我们的应用时，标签集非常有用。假设我们在两个不同的数据中心中有 MongoDB 应用程序和副本集的实例。让我们通过在副本集主节点 mongod shell 上运行以下序列来创建标签，这些标签将指示我们的实例位于哪个数据中心。首先，我们将获取副本集配置文档并将其存储在`cfg`变量中：

```sql
rs1:PRIMARY> cfg = rs.conf()

```

然后，通过使用`cfg`变量，我们将为我们的三个副本集成员中的每一个设置一个文档作为`members[n].tags`字段的新值：

```sql
rs1:PRIMARY> cfg.members[0].tags = {"media": "ssd", "application": "main", "datacenter": "A"}
rs1:PRIMARY> cfg.members[1].tags = {"media": "ssd", "application": "main", "datacenter": "B"}
rs1:PRIMARY> cfg.members[2].tags = {"media": "ssd", "application": "report", "datacenter": "A"}

```

最后，我们调用`reconfig()`方法，传入存储在`cfg`变量中的新配置文档以重新配置我们的副本集：

```sql
rs1:PRIMARY> rs.reconfig(cfg)

```

如果一切正确，我们将在 mongod shell 中看到这个输出：

```sql
{ "ok" : 1 }

```

我们的配置结果可以通过执行命令`rs.conf()`来检查：

```sql
rs1:PRIMARY> rs.conf()
{
 "_id" : "rs1",
 "version" : 6,
 "members" : [
 {
 "_id" : 0,
 "host" : "172.17.0.7:27017",
 "tags" : {
 "application" : "main",
 "datacenter" : "A",
 "media" : "ssd"
 }
 },
 {
 "_id" : 1,
 "host" : "172.17.0.5:27017",
 "tags" : {
 "application" : "main",
 "datacenter" : "B",
 "media" : "ssd"
 }
 },
 {
 "_id" : 2,
 "host" : "172.17.0.6:27017",
 "tags" : {
 "application" : "report",
 "datacenter" : "A",
 "media" : "ssd"
 }
 }
 ]
}

```

为了将读操作定位到特定的数据中心，我们必须在查询中指定一个新的标签。以下查询将使用标签，并且每个查询将在自己的数据中心中执行：

```sql
db.customers.find(
 {username: "customer5"}
).readPref(
 {tags: [{application: "main", media: "ssd", datacenter: "A"}]}
) // It will be executed in the replica set' instance 0 
db.customers.find(
 {username: "customer5"}
).readPref(
 {tags: [{application: "report", media: "ssd", datacenter: "A"}]}
) //It will be executed in the replica set's instance 2 
db.customers.find(
 {username: "customer5"}
).readPref(
 {tags: [{application: "main", media: "ssd", datacenter: "B"}]}
) //It will be executed in the replica set's instance 1

```

在写操作中，标签集不用于选择可用于写入的副本集成员。尽管可以通过创建自定义写关注来在写操作中使用标签集。

让我们回到本节开头提出的要求。我们如何确保写操作将分布在地理区域的至少两个实例上？通过在副本集主节点 mongod shell 上运行以下命令序列，我们将配置一个具有五个实例的副本集：

```sql
rs1:PRIMARY> cfg = rs.conf()
rs1:PRIMARY> cfg.members[0].tags = {"riodc": "rack1"}
rs1:PRIMARY> cfg.members[1].tags = {"riodc": "rack2"}
rs1:PRIMARY> cfg.members[2].tags = {"riodc": "rack3"}
rs1:PRIMARY> cfg.members[3].tags = {"spdc": "rack1"}
rs1:PRIMARY> cfg.members[4].tags = {"spdc": "rack2"}
rs1:PRIMARY> rs.reconfig(cfg)

```

标签`riodc`和`spdc`表示我们的实例所在的地理位置。

现在，让我们创建一个自定义的`writeConcern` MultipleDC，使用`getLastErrorModes`属性。这将确保写操作将分布到至少一个位置成员。

为此，我们将执行前面的序列，其中我们在副本集配置文档的`settings`字段上设置了一个代表我们自定义写关注的文档：

```sql
rs1:PRIMARY> cfg = rs.conf()
rs1:PRIMARY> cfg.settings = {getLastErrorModes: {MultipleDC: {"riodc": 1, "spdc":1}}}

```

mongod shell 中的输出应该是这样的：

```sql
{
 "getLastErrorModes" : {
 "MultipleDC" : {
 "riodc" : 1,
 "spdc" : 1
 }
 }
}

```

然后我们调用`reconfig()`方法，传入新的配置：

```sql
rs1:PRIMARY> rs.reconfig(cfg)

```

如果执行成功，在 mongod shell 中的输出将是这样的文档：

```sql
{ "ok" : 1 }

```

从这一刻起，我们可以使用`writeConcern` MultipleDC 来确保写操作将在每个显示的数据中心的至少一个节点中执行，如下所示：

```sql
db.customers.insert(
 {
 username: "customer6", 
 email: "customer6@customer.com",
 password: "1185031ff57bfdaae7812dd705383c74", 
 followedSellers: ["seller1", "seller3"]
 }, 
 {
 writeConcern: {w: "MultipleDC"} 
 }
)

```

回到我们的要求，如果我们希望写操作至少在每个数据中心的两个实例中执行，我们必须按以下方式配置：

```sql
rs1:PRIMARY> cfg = rs.conf()
rs1:PRIMARY> cfg.settings = {getLastErrorModes: {MultipleDC: {"riodc": 2, "spdc":2}}}
rs1:PRIMARY> rs.reconfig(cfg)

```

并且，满足我们的要求，我们可以创建一个名为`ssd`的`writeConcern` MultipleDC。这将确保写操作将发生在至少一个具有这种类型磁盘的实例中：

```sql
rs1:PRIMARY> cfg = rs.conf()
rs1:PRIMARY> cfg.members[0].tags = {"riodc": "rack1", "ssd": "ok"}
rs1:PRIMARY> cfg.members[3].tags = {"spdc": "rack1", "ssd": "ok"}
rs1:PRIMARY> rs.reconfig(cfg)
rs1:PRIMARY> cfg.settings = {getLastErrorModes: {MultipleDC: {"riodc": 2, "spdc":2}, ssd: {"ssd": 1}}}
rs1:PRIMARY> rs.reconfig(cfg)

```

在下面的查询中，我们看到使用`writeConcern` MultipleDC 需要写操作至少出现在具有`ssd`的一个实例中：

```sql
db.customers.insert(
 {
 username: "customer6", 
 email: "customer6@customer.com", 
 password: "1185031ff57bfdaae7812dd705383c74", 
 followedSellers: ["seller1", "seller3"]
 }, 
 {
 writeConcern: {w: "ssd"} 
 }
)

```

在我们的数据库中进行操作分离并不是一项简单的任务。但是，对于数据库的管理和维护非常有用。这种任务的早期实施需要对我们的数据模型有很好的了解，因为数据库所在的存储的细节非常重要。

在下一节中，我们将看到如何为需要高吞吐量和快速响应时间的应用程序规划集合。

### 提示

如果您想了解如何配置副本集标签集，可以访问 MongoDB 参考手册[`docs.mongodb.org/manual/tutorial/configure-replica-set-tag-sets/#replica-set-configuration-tag-sets`](http://docs.mongodb.org/manual/tutorial/configure-replica-set-tag-sets/#replica-set-configuration-tag-sets)。

# 固定大小集合

非功能性需求通常与应用程序的响应时间有关。特别是在当今时代，我们一直连接到新闻源，希望最新信息能在最短的响应时间内可用。

MongoDB 有一种特殊类型的集合，满足非功能性需求，即固定大小的集合。固定大小的集合支持高读写吞吐量。这是因为文档按其自然顺序插入，无需索引执行写操作。

MongoDB 保证了自然插入顺序，将数据写入磁盘。因此，在文档的生命周期中不允许增加文档大小的更新。一旦集合达到最大大小，MongoDB 会自动清理旧文档，以便插入新文档。

一个非常常见的用例是应用程序日志的持久性。MongoDB 本身使用副本集操作日志`oplog.rs`作为固定大小集合。在第八章*使用 MongoDB 进行日志记录和实时分析*中，您将看到另一个实际示例。

MongoDB 的另一个非常常见的用途是作为发布者/订阅者系统，特别是如果我们使用可追溯的游标。可追溯的游标是即使客户端读取了所有返回的记录，仍然保持打开状态的游标。因此，当新文档插入集合时，游标将其返回给客户端。

以下命令创建`ordersQueue`集合：

```sql
db.createCollection("ordersQueue",{capped: true, size: 10000})

```

我们使用`util`命令`createCollection`创建了我们的固定大小集合，传递给它名称`ordersQueue`和一个带有`capped`属性值为`true`和`size`值为`10000`的集合。如果`size`属性小于 4,096，MongoDB 会调整为 4,096 字节。另一方面，如果大于 4,096，MongoDB 会提高大小并调整为 256 的倍数。

可选地，我们可以使用`max`属性设置集合可以拥有的最大文档数量：

```sql
db.createCollection(
 "ordersQueue",
 {capped: true, size: 10000, max: 5000}
)

```

### 注意

如果我们需要将集合转换为固定大小集合，应该使用`convertToCapped`方法如下：

```sql
db.runCommand(
 {"convertToCapped": " ordersQueue ", size: 100000}
)

```

正如我们已经看到的，MongoDB 按自然顺序保留文档，换句话说，按照它们插入 MongoDB 的顺序。考虑以下文档，如`ordersQueue`集合中所示插入：

```sql
{
 "_id" : ObjectId("54d97db16840a9a7c089fa30"), 
 "orderId" : "order_1", 
 "time" : 1423539633910 
}
{
 "_id" : ObjectId("54d97db66840a9a7c089fa31"), 
 "orderId" : "order_2", 
 "time" : 1423539638006 
}
{
 "_id" : ObjectId("54d97dba6840a9a7c089fa32"), 
 "orderId" : "order_3", 
 "time" : 1423539642022 
}
{
 "_id" : ObjectId("54d97dbe6840a9a7c089fa33"), 
 "orderId" : "order_4", 
 "time" : 1423539646015 
}
{
 "_id" : ObjectId("54d97dcf6840a9a7c089fa34"), 
 "orderId" : "order_5", 
 "time" : 1423539663559 
}

```

查询`db.ordersQueue.find()`产生以下结果：

```sql
{ 
 "_id" : ObjectId("54d97db16840a9a7c089fa30"), 
 "orderId" : "order_1", 
 "time" : 1423539633910 
}
{ 
 "_id" : ObjectId("54d97db66840a9a7c089fa31"), 
 "orderId" : "order_2", 
 "time" : 1423539638006 
}
{ 
 "_id" : ObjectId("54d97dba6840a9a7c089fa32"), 
 "orderId" : "order_3", 
 "time" : 1423539642022 
}
{ 
 "_id" : ObjectId("54d97dbe6840a9a7c089fa33"), 
 "orderId" : "order_4", 
 "time" : 1423539646015 
}
{ 
 "_id" : ObjectId("54d97dcf6840a9a7c089fa34"), 
 "orderId" : "order_5", 
 "time" : 1423539663559 
}

```

如果我们像以下查询中所示使用`$natural`操作符，将得到与前面输出中相同的结果：

```sql
db.ordersQueue.find().sort({$natural: 1})

```

但是，如果我们需要最后插入的文档先返回，我们必须在`$natural`操作符上执行带有`-1`值的命令：

```sql
db.ordersQueue.find().sort({$natural: -1})

```

在创建固定大小集合时，我们必须小心：

+   我们不能对固定大小集合进行分片。

+   我们不能在固定大小集合中更新文档；否则，文档会增大。如果需要在固定大小集合中更新文档，则必须确保大小保持不变。为了更好的性能，在更新时应创建索引以避免集合扫描。

+   我们无法在封顶集合中删除文档。

当我们具有高读/写吞吐量作为非功能性要求，或者需要按字节大小或文档数量限制集合大小时，封顶集合是一个很好的工具。

尽管如此，如果我们需要根据时间范围自动使数据过期，我们应该使用**生存时间**（TTL）函数。

# 数据自动过期

正如您在第四章中已经看到的，MongoDB 为我们提供了一种索引类型，可以帮助我们在一定时间后或特定日期之后从集合中删除数据。

实际上，TTL 是在 mongod 实例上执行的后台线程，它会查找索引上具有日期类型字段的文档，并将其删除。

考虑一个名为`customers`的集合，其中包含以下文档：

```sql
{ 
 "_id" : ObjectId("5498da405d0ffdd8a07a87ba"), 
 "username" : "customer1", 
 "email" : "customer1@customer.com", 
 "password" : "b1c5098d0c6074db325b0b9dddb068e1", "accountConfirmationExpireAt" : ISODate("2015-01-11T20:27:02.138Z") 
}

```

为了在 360 秒后使该集合中的文档过期，我们应该创建以下索引：

```sql
db.customers.createIndex(
 {accountConfirmationExpireAt: 1}, 
 {expireAfterSeconds: 3600}
)

```

为了在 2015-01-11 20:27:02 准确地使文档过期，我们应该创建以下索引：

```sql
db.customers.createIndex(
 {accountConfirmationExpireAt: 1}, 
 {expireAfterSeconds: 0}
)

```

在使用 TTL 函数时，我们必须格外小心，并牢记以下几点：

+   我们无法在封顶集合上创建 TTL 索引，因为 MongoDB 无法从集合中删除文档。

+   TTL 索引不能具有作为另一个索引一部分的字段。

+   索引字段应为日期或日期类型的数组。

+   尽管在每个副本集节点中都有后台线程，可以在具有 TTL 索引时删除文档，但它只会从主节点中删除它们。复制过程将从副本集的辅助节点中删除文档。

# 总结

在本章中，您看到了除了根据我们的查询来思考架构设计之外，还要考虑规划操作和维护来创建我们的集合。

您学会了如何使用标签集来处理数据中心感知操作，以及为什么通过创建封顶集合来限制我们集合中存储的文档数量。同样，您还了解了 TTL 索引在实际用例中的用处。

在下一章中，您将看到如何通过创建分片来扩展我们的 MongoDB 实例。
