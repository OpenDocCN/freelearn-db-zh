# 第四章：高级查询

在上一章中，我们学习了如何以安全的方式使用 mongo shell 进行脚本编写、管理和开发。在本章中，我们将更深入地使用来自 Ruby，Python 和 PHP 的驱动程序和流行框架与 MongoDB 一起使用：**超文本预处理器**（**PHP**）。

我们还将展示使用这些语言的最佳实践以及 MongoDB 在数据库级别支持的各种比较和更新运算符，这些运算符可以通过 Ruby，Python 和 PHP 访问。

在本章中，我们将学习以下主题：

+   MongoDB 操作

+   使用 Ruby，Mongoid，Python，PyMODM，PHP 和 Doctrine 进行 CRUD

+   比较运算符

+   更改流

# MongoDB CRUD 操作

在本节中，我们将分别使用官方 MongoDB 驱动程序和每种语言的一些流行框架，使用 Ruby，Python 和 PHP 进行 CRUD 操作。

# 使用 Ruby 驱动程序进行 CRUD

在第三章中，*MongoDB CRUD 操作*，我们介绍了如何使用驱动程序和 ODM 从 Ruby，Python 和 PHP 连接到 MongoDB。在本章中，我们将探索使用官方驱动程序和最常用的 ODM 框架进行`create`，`read`，`update`和`delete`操作。

# 创建文档

使用第二章中描述的过程，*模式设计和数据建模*，我们假设我们有一个`@collection`实例变量，指向`mongo_book`数据库中`127.0.0.1:27017`默认数据库中的`books`集合：

```sql
@collection = Mongo::Client.new([ '127.0.0.1:27017' ], :database => 'mongo_book').database[:books]
```

我们插入一个具有我们定义的单个文档，如下所示：

```sql
document = { isbn: '101', name: 'Mastering MongoDB', price: 30}
```

这可以通过一行代码执行如下：

```sql
result = @collection.insert_one(document)
```

生成的对象是`Mongo::Operation::Result`类，其内容与我们在 shell 中看到的内容相似，如下面的代码所示：

```sql
{"n"=>1, "ok"=>1.0}
```

这里，`n`是受影响的文档数量；`1`表示我们插入了一个对象，`ok`表示`1`（`true`）。

在一步中创建多个文档与此类似。对于具有`isbn 102`和`103`的两个文档，并且使用`insert_many`而不是`insert_one`，我们有以下代码：

```sql
documents = [ { isbn: '102', name: 'MongoDB in 7 years', price: 50 },
            { isbn: '103', name: 'MongoDB for experts', price: 40 } ]
result = @collection.insert_many(documents)
```

生成的对象现在是`Mongo::BulkWrite::Result`类，这意味着使用了`BulkWrite`接口以提高性能。

主要区别在于我们现在有一个属性`inserted_ids`，它将返回从`BSON::ObjectId`类中插入的对象的`ObjectId`。

# 读取

查找文档的工作方式与创建它们的方式相同，即在集合级别：

```sql
@collection.find( { isbn: '101' } )
```

可以链接多个搜索条件，并且相当于 SQL 中的`AND`运算符：

```sql
@collection.find( { isbn: '101', name: 'Mastering MongoDB' } )
```

mongo-ruby-driver API 提供了几个查询选项来增强查询；最常用的查询选项列在以下表中：

| **选项** | **描述** |
| --- | --- |
| `allow_partial_results` | 这是用于分片集群。如果一个分片关闭，它允许查询从打开的分片返回结果，可能只得到部分结果。 |
| `batch_size(Integer)` | 这可以改变游标从 MongoDB 获取的批量大小。这是在每个`GETMORE`操作（例如，在 mongo shell 上输入）上完成的。 |
| `comment(String)` | 使用此命令，我们可以为了文档目的在我们的查询中添加注释。 |
| `hint(Hash)` | 我们可以使用`hint()`强制使用索引。 |
| `limit(Integer)` | 我们可以将结果集限制为`Integer`指定的文档数量。 |
| `max_scan(Integer)` | 我们可以限制将被扫描的文档数量。这将返回不完整的结果，并且在执行我们希望保证不会花费很长时间的操作时非常有用，例如当我们连接到我们的生产数据库时。 |
| `no_cursor_timeout` | 如果我们不指定此参数，MongoDB 将在 600 秒后关闭任何不活动的游标。使用此参数，我们的游标将永远不会关闭。 |
| `projection(Hash)` | 我们可以使用这个参数来获取或排除结果中的特定属性。这将减少通过网络的数据传输。例如，`client[:books].find.projection(:price => 1)`。 |
| `read(Hash)` | 我们可以指定一个读取偏好，仅应用于此查询：`client[:books].find.read(:mode => :secondary_preferred)`。 |
| `show_disk_loc(Boolean)` | 如果我们想要查找结果在磁盘上的实际位置，应该使用此选项。 |
| `skip(Integer)` | 这可以用于跳过指定数量的文档。对于结果的分页很有用。 |
| `snapshot` | 这可以用于以快照模式执行我们的查询。当我们需要更严格的一致性时，这是很有用的。 |
| `sort(Hash)` | 我们可以使用这个来对结果进行排序，例如，`client[:books].find.sort(:name => -1)`。 |

除了查询选项之外，mongo-ruby-driver 还提供了一些辅助函数，可以在方法调用级别进行链接，如下所示：

+   `.count`：前面查询的总计数

+   `.distinct(:field_name)`：通过 `:field_name` 区分前面查询的结果

`Find()` 返回一个包含结果集的游标，我们可以像其他对象一样在 Ruby 中使用 `.each` 进行迭代：

```sql
result = @collection.find({ isbn: '101' })
result.each do |doc|
  puts doc.inspect
end
```

我们的 `books` 集合的输出如下：

```sql
{"_id"=>BSON::ObjectId('592149c4aabac953a3a1e31e'), "isbn"=>"101", "name"=>"Mastering MongoDB", "price"=>30.0, "published"=>2017-06-25 00:00:00 UTC}
```

# 在 find() 中链接操作

`find()` 默认使用 `AND` 运算符来匹配多个字段。如果我们想使用 `OR` 运算符，我们的查询需要如下所示：

```sql
result = @collection.find('$or' => [{ isbn: '101' }, { isbn: '102' }]).to_a
puts result
```

前面代码的输出如下：

```sql
{"_id"=>BSON::ObjectId('592149c4aabac953a3a1e31e'), "isbn"=>"101", "name"=>"Mastering MongoDB", "price"=>30.0, "published"=>2017-06-25 00:00:00 UTC}{"_id"=>BSON::ObjectId('59214bc1aabac954263b24e0'), "isbn"=>"102", "name"=>"MongoDB in 7 years", "price"=>50.0, "published"=>2017-06-26 00:00:00 UTC}
```

在前面的示例中，我们也可以使用 `$and` 而不是 `$or`：

```sql
result = @collection.find('$and' => [{ isbn: '101' }, { isbn: '102' }]).to_a
puts result
```

当然，这将不返回任何结果，因为没有文档可以同时具有 `isbn 101` 和 `102`。

一个有趣且难以发现的 bug 是如果我们多次定义相同的键，就像以下代码中一样：

```sql
result = @collection.find({ isbn: '101', isbn: '102' })
puts result
{"_id"=>BSON::ObjectId('59214bc1aabac954263b24e0'), "isbn"=>"102", "name"=>"MongoDB in 7 years", "price"=>50.0, "published"=>2017-06-26 00:00:00 UTC}
```

相反的顺序将导致返回带有 `isbn 101` 的文档：

```sql
result = @collection.find({ isbn: '102', isbn: '101' })
puts result
{"_id"=>BSON::ObjectId('592149c4aabac953a3a1e31e'), "isbn"=>"101", "name"=>"Mastering MongoDB", "price"=>30.0, "published"=>2017-06-25 00:00:00 UTC}
```

这是因为在 Ruby 哈希中，默认情况下，除了最后一个之外，所有重复的键都会被静默忽略。这可能不会发生在前面示例中所显示的简单形式中，但如果我们以编程方式创建键，这种情况很容易发生。

# 嵌套操作

在 mongo-ruby-driver 中访问嵌入式文档就像使用点表示法一样简单：

```sql
result = @collection.find({'meta.authors': 'alex giamas'}).to_a
puts result
"_id"=>BSON::ObjectId('593c24443c8ca55b969c4c54'), "isbn"=>"201", "name"=>"Mastering MongoDB, 2nd Edition", "meta"=>{"authors"=>"alex giamas"}}
```

我们需要用引号 (`''`) 括起键名来访问嵌入对象，就像我们需要对以 `$` 开头的操作一样，比如 `'$set'`。

# 更新

使用 mongo-ruby-driver 更新文档是通过查找它们进行链接的。使用我们的示例 `books` 集合，我们可以执行以下操作：

```sql
@collection.update_one( { 'isbn': 101}, { '$set' => { name: 'Mastering MongoDB, 2nd Edition' } } )
```

这将找到具有 `isbn 101` 的文档，并将其名称更改为 `Mastering MongoDB, 2nd Edition`。

类似于 `update_one`，我们可以使用 `update_many` 来更新通过方法的第一个参数检索到的多个文档。

如果我们不使用 `$set` 运算符，文档的内容将被新文档替换。

假设 Ruby 版本 >=2.2，键可以是带引号或不带引号的；但是，以 `$` 开头的键需要按如下方式带引号：

```sql
@collection.update( { isbn: '101'}, { "$set": { name: "Mastering MongoDB, 2nd edition" } } )
```

更新后的对象将包含有关操作的信息，包括以下方法：

+   `ok?`：一个布尔值，显示操作是否成功

+   `matched_count`：匹配查询的文档数量

+   `modified_count`：受影响的文档数量（已更新）

+   `upserted_count`：如果操作包括 `$set`，则插入的文档数量

+   `upserted_id`：如果有的话，插入文档的唯一 `ObjectId`

修改字段大小恒定的更新将是 *原地* 进行的；这意味着它们不会将文档从物理位置上移动。这包括对 `Integer` 和 `Date` 字段进行的操作，如 `$inc` 和 `$set`。

可能会导致文档大小增加的更新可能会导致文档从磁盘上的物理位置移动到文件末尾的新位置。在这种情况下，查询可能会错过或多次返回文档。为了避免这种情况，我们可以在查询时使用`$snapshot: true`。

# 删除

删除文档的工作方式与查找文档类似。我们需要找到文档，然后应用删除操作。

例如，对于我们之前使用的`books`集合，我们可以发出以下代码：

```sql
@collection.find( { isbn: '101' } ).delete_one
```

这将删除单个文档。在我们的情况下，由于每个文档的`isbn`都是唯一的，这是预期的。如果我们的`find()`子句匹配了多个文档，那么`delete_one`将只删除`find()`返回的第一个文档，这可能是我们想要的，也可能不是。

如果我们使用`delete_one`与匹配多个文档的查询，结果可能会出乎意料。

如果我们想要删除与我们的`find()`查询匹配的所有文档，我们必须使用`delete_many`，如下所示：

```sql
@collection.find( { price: { $gte: 30 } ).delete_many
```

在前面的示例中，我们正在删除所有价格大于或等于`30`的书籍。

# 批量操作

我们可以使用`BulkWrite` API 进行批量操作。在我们之前插入多个文档的示例中，操作如下：

```sql
@collection.bulk_write([ { insertMany: documents
                     }],
                   ordered: true)
```

`BulkWrite` API 可以接受以下参数：

+   `insertOne`

+   `updateOne`

+   `updateMany`

+   `replaceOne`

+   `deleteOne`

+   `deleteMany`

这些命令的一个版本将`insert`/`update`/`replace`/`delete`单个文档，即使我们指定的过滤器匹配多个文档。在这种情况下，为了避免意外行为，重要的是要有一个匹配单个文档的过滤器。

在`bulk_write`命令的第一个参数中包含多个操作也是可能的，也是一个完全有效的用例。这允许我们在有相互依赖的操作并且我们想要根据业务逻辑批量处理它们的情况下按照逻辑顺序发出命令。任何错误都将停止`ordered:true`批量写入，我们将需要手动回滚我们的操作。一个值得注意的例外是`writeConcern`错误，例如，请求我们的副本集成员中的大多数确认我们的写入。在这种情况下，批量写入将继续进行，我们可以在`writeConcernErrors`结果字段中观察到错误：

```sql
old_book = @collection.findOne(name: 'MongoDB for experts')
new_book = { isbn: 201, name: 'MongoDB for experts, 2nd Edition', price: 55 }
@collection.bulk_write([ {deleteOne: old_book}, { insertOne: new_book
                     }],
                   ordered: true)
```

在前面的示例中，我们确保在添加新的（更昂贵的）版本我们的`MongoDB for experts`书之前删除了原始书籍。

`BulkWrite`可以批处理最多 1,000 个操作。如果我们的命令中有超过 1,000 个基础操作，这些操作将被分成数千个块。如果可能的话，最好尽量将写操作保持在一个批次中，以避免意外行为。

# Mongoid 中的 CRUD

在本节中，我们将使用 Mongoid 执行`create`、`read`、`update`和`delete`操作。所有这些代码也可以在 GitHub 上找到：[`github.com/agiamas/mastering-mongodb/tree/master/chapter_4`](https://github.com/agiamas/mastering-mongodb/tree/master/chapter_4)。

# 读取

回到第二章*模式设计和数据建模*中，我们描述了如何安装、连接和设置模型，包括对 Mongoid 的继承。在这里，我们将介绍 CRUD 的最常见用例。

使用类似于**Active Record**（**AR**）的 DSL 来查找文档。与使用关系数据库的 AR 一样，Mongoid 将一个类分配给一个 MongoDB 集合（表），并将任何对象实例分配给一个文档（关系数据库中的行）：

```sql
Book.find('592149c4aabac953a3a1e31e')
```

这将通过`ObjectId`查找文档并返回具有`isbn 101`的文档，与通过名称属性进行的查询一样：

```sql
Book.where(name: 'Mastering MongoDB')
```

与通过属性动态生成的 AR 查询类似，我们可以使用辅助方法：

```sql
Book.find_by(name: 'Mastering MongoDB')
```

这通过属性名称查询，相当于之前的查询。

我们应该启用`QueryCache`以避免多次命中数据库相同的查询，如下所示：

```sql
Mongoid::QueryCache.enabled = true
```

这可以添加在我们想要启用的任何代码块中，或者在 Mongoid 的初始化程序中。

# 范围查询

我们可以使用类方法在 Mongoid 中范围查询，如下所示：

```sql
Class Book
...
  def self.premium
     where(price: {'$gt': 20'})
   end
End
```

然后我们将使用这个查询：

```sql
Book.premium
```

它将查询价格大于 20 的书籍。

# 创建、更新和删除

用于创建文档的 Ruby 接口类似于活动记录：

```sql
Book.where(isbn: 202, name: 'Mastering MongoDB, 3rd Edition').create
```

如果创建失败，这将返回错误。

我们可以使用感叹号版本来强制引发异常，如果保存文档失败：

```sql
Book.where(isbn: 202, name: 'Mastering MongoDB, 3rd Edition').create!
```

截至 Mongoid 版本 6.x，不支持`BulkWrite` API。解决方法是使用 mongo-ruby-driver API，它不会使用`mongoid.yml`配置或自定义验证。否则，您可以使用`insert_many([array_of_documents])`，它将逐个插入文档。

要更新文档，我们可以使用`update`或`update_all`。使用`update`将仅更新查询部分检索到的第一个文档，而`update_all`将更新所有文档：

```sql
Book.where(isbn: 202).update(name: 'Mastering MongoDB, THIRD Edition')
Book.where(price: { '$gt': 20 }).update_all(price_range: 'premium')
```

删除文档类似于创建文档，提供`delete`以跳过回调，以及如果我们想要执行受影响文档中的任何可用回调，则使用`destroy`。

`delete_all`和`destroy_all`是用于多个文档的便捷方法。

如果可能的话，应该避免使用`destroy_all`，因为它将加载所有文档到内存中以执行回调，因此可能会占用大量内存。

# 使用 Python 驱动程序进行 CRUD

PyMongo 是 MongoDB 官方支持的 Python 驱动程序。在本节中，我们将使用 PyMongo 在 MongoDB 中创建、读取、更新和删除文档。

# 创建和删除

Python 驱动程序提供了与 Ruby 和 PHP 一样的 CRUD 方法。在第二章“模式设计和数据建模”之后，指向我们的`books`集合的`books`变量，我们将编写以下代码块：

```sql
from pymongo import MongoClient
from pprint import pprint

>>> book = {
 'isbn': '301',
 'name': 'Python and MongoDB',
 'price': 60
}
>>> insert_result = books.insert_one(book)
>>> pprint(insert_result)

<pymongo.results.InsertOneResult object at 0x104bf3370>

>>> result = list(books.find())
>>> pprint(result)

[{u'_id': ObjectId('592149c4aabac953a3a1e31e'),
 u'isbn': u'101',
 u'name': u'Mastering MongoDB',
 u'price': 30.0,
 u'published': datetime.datetime(2017, 6, 25, 0, 0)},
{u'_id': ObjectId('59214bc1aabac954263b24e0'),
 u'isbn': u'102',
 u'name': u'MongoDB in 7 years',
 u'price': 50.0,
 u'published': datetime.datetime(2017, 6, 26, 0, 0)},
{u'_id': ObjectId('593c24443c8ca55b969c4c54'),
 u'isbn': u'201',
 u'meta': {u'authors': u'alex giamas'},
 u'name': u'Mastering MongoDB, 2nd Edition'},
{u'_id': ObjectId('594061a9aabac94b7c858d3d'),
 u'isbn': u'301',
 u'name': u'Python and MongoDB',
 u'price': 60}]
```

在前面的示例中，我们使用`insert_one()`来插入单个文档，我们可以使用 Python 字典表示法来定义它；然后我们可以查询它以获取集合中的所有文档。

`insert_one`的结果对象和`insert_many`的结果对象有两个感兴趣的字段：

+   `Acknowledged`：如果插入成功则为`true`，如果插入失败则为`false`，或者写关注为`0`（即插即忘写）。

+   `inserted_id`对于`insert_one`：写入文档的`ObjectId`和`inserted_ids`对于`insert_many`：写入文档的`ObjectIds`数组。

我们使用`pprint`库对`find()`结果进行漂亮打印。通过使用以下代码来迭代结果集的内置方式：

```sql
for document in results:
   print(document)
```

删除文档的工作方式与创建它们类似。我们可以使用`delete_one`来删除第一个实例，或者使用`delete_many`来删除匹配查询的所有实例：

```sql
>>> result = books.delete_many({ "isbn": "101" })
>>> print(result.deleted_count)
1
```

`deleted_count`实例告诉我们删除了多少个文档；在我们的案例中，它是`1`，即使我们使用了`delete_many`方法。

要从集合中删除所有文档，我们可以传入空文档`{}`。

要删除集合，我们可以使用`drop()`：

```sql
>>> books.delete_many({})
>>> books.drop()
```

# 查找文档

要根据顶级属性查找文档，我们可以简单地使用字典：

```sql
>>> books.find({"name": "Mastering MongoDB"})

[{u'_id': ObjectId('592149c4aabac953a3a1e31e'),
 u'isbn': u'101',
 u'name': u'Mastering MongoDB',
 u'price': 30.0,
 u'published': datetime.datetime(2017, 6, 25, 0, 0)}]
```

要在嵌入文档中查找文档，我们可以使用点表示法。在下面的示例中，我们使用`meta.authors`来访问`meta`文档内的`authors`嵌入文档：

```sql
>>> result = list(books.find({"meta.authors": {"$regex": "aLEx", "$options": "i"}}))
>>> pprint(result)

[{u'_id': ObjectId('593c24443c8ca55b969c4c54'),
 u'isbn': u'201',
 u'meta': {u'authors': u'alex giamas'},
 u'name': u'Mastering MongoDB, 2nd Edition'}]
```

在此示例中，我们使用正则表达式来匹配`aLEx`，它是不区分大小写的，在`meta.authors`嵌入文档中提到字符串的每个文档中。PyMongo 在 MongoDB 文档中称之为正则表达式查询的`$regex`表示法。第二个参数是`$regex`的选项参数，我们将在本章后面的“使用正则表达式”部分详细解释。

还支持比较运算符，完整列表将在本章后面的“比较运算符”部分中给出：

```sql
>>> result = list(books.find({ "price": {  "$gt":40 } }))
>>> pprint(result)

[{u'_id': ObjectId('594061a9aabac94b7c858d3d'),
 u'isbn': u'301',
 u'name': u'Python and MongoDB',
 u'price': 60}]
```

在我们的查询中添加多个字典会导致逻辑`AND`查询：

```sql
>>> result = list(books.find({"name": "Mastering MongoDB", "isbn": "101"}))
>>> pprint(result)

[{u'_id': ObjectId('592149c4aabac953a3a1e31e'),
 u'isbn': u'101',
 u'name': u'Mastering MongoDB',
 u'price': 30.0,
 u'published': datetime.datetime(2017, 6, 25, 0, 0)}]
```

对于同时具有`isbn=101`和`name=Mastering MongoDB`的书籍，要使用`$or`和`$and`等逻辑运算符，我们必须使用以下语法：

```sql
>>> result = list(books.find({"$or": [{"isbn": "101"}, {"isbn": "102"}]}))
>>> pprint(result)

[{u'_id': ObjectId('592149c4aabac953a3a1e31e'),
 u'isbn': u'101',
 u'name': u'Mastering MongoDB',
 u'price': 30.0,
 u'published': datetime.datetime(2017, 6, 25, 0, 0)},
{u'_id': ObjectId('59214bc1aabac954263b24e0'),
 u'isbn': u'102',
 u'name': u'MongoDB in 7 years',
 u'price': 50.0,
 u'published': datetime.datetime(2017, 6, 26, 0, 0)}]
```

对于具有`isbn`为`101`或`102`的书籍，如果我们想要结合`AND`和`OR`运算符，我们必须使用`$and`运算符，如下所示：

```sql
>>> result = list(books.find({"$or": [{"$and": [{"name": "Mastering MongoDB", "isbn": "101"}]}, {"$and": [{"name": "MongoDB in 7 years", "isbn": "102"}]}]}))
>>> pprint(result)
[{u'_id': ObjectId('592149c4aabac953a3a1e31e'),
 u'isbn': u'101',
 u'name': u'Mastering MongoDB',
 u'price': 30.0,
 u'published': datetime.datetime(2017, 6, 25, 0, 0)},
{u'_id': ObjectId('59214bc1aabac954263b24e0'),
 u'isbn': u'102',
 u'name': u'MongoDB in 7 years',
 u'price': 50.0,
 u'published': datetime.datetime(2017, 6, 26, 0, 0)}]
```

对于两个查询之间的`OR`结果，请考虑以下内容：

+   第一个查询是要求具有`isbn=101 AND name=Mastering MongoDB`的文档

+   第二个查询是要求在 7 年内具有`isbn=102 AND name=MongoDB`的文档。

+   结果是这两个数据集的并集

# 更新文档

在下面的代码块中，您可以看到使用`update_one`辅助方法更新单个文档的示例。

此操作在搜索阶段匹配一个文档，并根据要应用于匹配文档的操作修改一个文档：

```sql
>>> result = books.update_one({"isbn": "101"}, {"$set": {"price": 100}})
>>> print(result.matched_count)
1
>>> print(result.modified_count)
1
```

类似于插入文档时，更新文档时，我们可以使用`update_one`或`update_many`：

+   这里的第一个参数是匹配将要更新的文档的过滤文档

+   第二个参数是要应用于匹配文档的操作

+   第三个（可选）参数是使用`upsert=false`（默认值）或`true`，用于在找不到文档时创建新文档

另一个有趣的参数是`bypass_document_validation=false`（默认值）或`true`，这是可选的。这将忽略集合中文档的验证（如果有的话）。

结果对象将具有`matched_count`，表示匹配过滤查询的文档数量，以及`modified_count`，表示受`update`部分影响的文档数量。

在我们的示例中，我们通过`$set`更新运算符为具有`isbn=101`的第一本书设置`price=100`。所有更新运算符的列表将在本章后面的*更新运算符*部分中显示。

如果我们不使用更新运算符作为第二个参数，匹配文档的内容将完全被新文档替换。

# 使用 PyMODM 进行 CRUD

PyMODM 是一个核心 ODM，提供简单且可扩展的功能。它由 MongoDB 的工程师开发和维护，他们可以获得最新稳定版本的快速更新和支持。

在第二章*模式设计和数据建模*中，我们探讨了如何定义不同的模型并连接到 MongoDB。使用 PyMODM 进行 CRUD，与使用低级驱动程序相比，更简单。

# 创建文档

可以使用单行代码创建一个新的`user`对象，如第二章*模式设计和数据建模*中所定义的：

```sql
>>> user = User('alexgiamas@packt.com', 'Alex', 'Giamas').save()
```

在这个例子中，我们按照`user`模型中定义的顺序使用位置参数来为`user`模型属性赋值。

我们也可以使用关键字参数或两者的混合，如下所示：

```sql
>>> user = User(email='alexgiamas@packt.com', 'Alex', last_name='Giamas').save()
```

可以通过将用户数组传递给`bulk_create()`来进行批量保存：

```sql
>>> users = [ user1, user2,...,userN]
>>>  User.bulk_create(users)
```

# 更新文档

我们可以通过直接访问属性并再次调用`save()`来修改文档：

```sql
>>> user.first_name = 'Alexandros'
>>> user.save()
```

如果我们想要更新一个或多个文档，我们必须使用`raw()`来过滤将受影响的文档，并链接`update()`来设置新值：

```sql
>>> User.objects.raw({'first_name': {'$exists': True}})
              .update({'$set': {'updated_at': datetime.datetime.now()}})
```

在上面的示例中，我们搜索所有具有名字的`User`文档，并设置一个新字段`updated_at`为当前时间戳。`raw()`方法的结果是`QuerySet`，这是 PyMODM 中用于处理查询和批量处理文档的类。

# 删除文档

删除 API 与更新 API 类似-通过使用`QuerySet`查找受影响的文档，然后链接`.delete()`方法来删除它们：

```sql
>>> User.objects.raw({'first_name': {'$exists': True}}).delete()
```

在撰写本书时（2018 年 12 月），`BulkWrite`API 仍不受支持，相关的票号为 PYMODM-43。例如`bulk_create()`方法将在幕后向数据库发出多个命令。

# 查询文档

查询是使用`QuerySet`进行的，如在`update`和`delete`操作之前所述。

一些可用的便利方法包括以下内容：

+   `all()`

+   `count()`

+   `first()`

+   `exclude(*fields)` 从结果中排除一些字段

+   `only(*fields)` 只包括结果中的一些字段（可以链接以获取字段的并集）

+   ``limit(limit)``

+   `order_by(ordering)`

+   `reverse()` 如果我们想要反转`order_by()`的顺序

+   `skip(number)`

+   `values()` 返回 Python 字典实例而不是模型实例

通过使用`raw()`，我们可以使用与前面 PyMongo 部分中描述的相同查询，同时利用 ODM 层提供的灵活性和便利方法。

# 使用 PHP 驱动程序进行 CRUD

在 PHP 中，有一个名为`mongo-php-library`的新驱动程序，应该代替已弃用的 MongoClient。总体架构在第二章*模式设计和数据建模*中有解释。在这里，我们将介绍 API 的更多细节以及如何使用它执行 CRUD 操作。

# 创建和删除

以下命令将插入一个包含两个键/值对数组的单个`$document`：

```sql
$document = array( "isbn" => "401", "name" => "MongoDB and PHP" );
$result = $collection->insertOne($document);
var_dump($result);
```

`var_dump($result)`命令的输出如下所示：

```sql
MongoDB\InsertOneResult Object
(
   [writeResult:MongoDB\InsertOneResult:private] => MongoDB\Driver\WriteResult Object
       (
           [nInserted] => 1
           [nMatched] => 0
           [nModified] => 0
           [nRemoved] => 0
           [nUpserted] => 0
           [upsertedIds] => Array
               (
               )

           [writeErrors] => Array
               (
               )

           [writeConcernError] =>
           [writeConcern] => MongoDB\Driver\WriteConcern Object
               (
               )

       )

   [insertedId:MongoDB\InsertOneResult:private] => MongoDB\BSON\ObjectID Object
       (
           [oid] => 5941ac50aabac9d16f6da142
       )

   [isAcknowledged:MongoDB\InsertOneResult:private] => 1
)
```

这个相当冗长的输出包含了我们可能需要的所有信息。我们可以获取插入文档的`ObjectId`；通过以`n`为前缀的字段获取`inserted`，`matched`，`modified`，`removed`和`upserted`文档的数量；以及关于`writeError`或`writeConcernError`的信息。

如果我们想要获取信息，`$result`对象中还有一些便利方法：

+   `$result->getInsertedCount()`: 获取插入对象的数量

+   `$result->getInsertedId()`: 获取插入文档的`ObjectId`

我们还可以使用`->insertMany()`方法一次插入多个文档，如下所示：

```sql
$documentAlpha = array( "isbn" => "402", "name" => "MongoDB and PHP, 2nd Edition" );$documentBeta  = array( "isbn" => "403", "name" => "MongoDB and PHP, revisited" );
$result = $collection->insertMany([$documentAlpha, $documentBeta]);

print_r($result);
```

结果如下所示：

```sql
(
   [writeResult:MongoDB\InsertManyResult:private] => MongoDB\Driver\WriteResult Object
       (
           [nInserted] => 2
           [nMatched] => 0
           [nModified] => 0
           [nRemoved] => 0
           [nUpserted] => 0
           [upsertedIds] => Array
               (
               )

           [writeErrors] => Array
               (
               )

           [writeConcernError] =>
           [writeConcern] => MongoDB\Driver\WriteConcern Object
               (
               )

       )

   [insertedIds:MongoDB\InsertManyResult:private] => Array
       (
           [0] => MongoDB\BSON\ObjectID Object
               (
                   [oid] => 5941ae85aabac9d1d16c63a2
               )

           [1] => MongoDB\BSON\ObjectID Object
               (
                   [oid] => 5941ae85aabac9d1d16c63a3
               )

       )

   [isAcknowledged:MongoDB\InsertManyResult:private] => 1
)
```

再次，`$result->getInsertedCount()`将返回`2`，而`$result->getInsertedIds()`将返回一个包含两个新创建的`ObjectIds`的数组：

```sql
array(2) {
 [0]=>
 object(MongoDB\BSON\ObjectID)#13 (1) {
   ["oid"]=>
   string(24) "5941ae85aabac9d1d16c63a2"
 }
 [1]=>
 object(MongoDB\BSON\ObjectID)#14 (1) {
   ["oid"]=>
   string(24) "5941ae85aabac9d1d16c63a3"
 }
}
```

删除文档与插入文档类似，但是使用`deleteOne()`和`deleteMany()`方法；`deleteMany()`的示例如下所示：

```sql
$deleteQuery = array( "isbn" => "401");
$deleteResult = $collection->deleteMany($deleteQuery);
print($deleteResult->getDeletedCount());
```

以下代码块显示了输出：

```sql
MongoDB\DeleteResult Object
(
   [writeResult:MongoDB\DeleteResult:private] => MongoDB\Driver\WriteResult Object
       (
           [nInserted] => 0
           [nMatched] => 0
           [nModified] => 0
           [nRemoved] => 2
           [nUpserted] => 0
           [upsertedIds] => Array
               (
               )

           [writeErrors] => Array
               (
               )

           [writeConcernError] =>
           [writeConcern] => MongoDB\Driver\WriteConcern Object
               (
               )

       )

   [isAcknowledged:MongoDB\DeleteResult:private] => 1
)
2
```

在这个例子中，我们使用`->getDeletedCount()`来获取受影响文档的数量，这个数量在输出的最后一行打印出来。

# BulkWrite

新的 PHP 驱动程序支持`BulkWrite`接口，以最小化对 MongoDB 的网络调用：

```sql
$manager = new MongoDB\Driver\Manager('mongodb://localhost:27017');
$bulk = new MongoDB\Driver\BulkWrite(array("ordered" => true));
$bulk->insert(array( "isbn" => "401", "name" => "MongoDB and PHP" ));
$bulk->insert(array( "isbn" => "402", "name" => "MongoDB and PHP, 2nd Edition" ));
$bulk->update(array("isbn" => "402"), array('$set' => array("price" => 15)));
$bulk->insert(array( "isbn" => "403", "name" => "MongoDB and PHP, revisited" ));

$result = $manager->executeBulkWrite('mongo_book.books', $bulk);
print_r($result);
```

结果如下所示：

```sql
MongoDB\Driver\WriteResult Object
(
   [nInserted] => 3
   [nMatched] => 1
   [nModified] => 1
   [nRemoved] => 0
   [nUpserted] => 0
   [upsertedIds] => Array
       (
       )

   [writeErrors] => Array
       (
       )

   [writeConcernError] =>
   [writeConcern] => MongoDB\Driver\WriteConcern Object
       (
       )

)
```

在上面的例子中，我们按顺序执行了两次插入，一次更新和第三次插入。`WriteResult`对象包含了总共三个插入文档和一个修改文档。

与简单的创建/删除查询相比的主要区别是，`executeBulkWrite()`是`MongoDB\Driver\Manager`类的一个方法，我们在第一行实例化它。

# 读取

查询接口类似于插入和删除，使用`findOne()`和`find()`方法来检索查询的第一个结果或所有结果：

```sql
$document = $collection->findOne( array("isbn" => "401") );
$cursor = $collection->find( array( "name" => new MongoDB\BSON\Regex("mongo", "i") ) );
```

在第二个例子中，我们使用正则表达式搜索具有值`mongo`（不区分大小写）的键名。

使用`.`符号可以查询嵌入文档，就像我们在本章前面检查的其他语言一样：

```sql
$cursor = $collection->find( array('meta.price' => 50) );
```

我们这样做是为了查询`meta`键字段内嵌的`price`文档。

与 Ruby 和 Python 类似，在 PHP 中，我们可以使用比较运算符进行查询，如下面的代码所示：

```sql
$cursor = $collection->find( array( 'price' => array('$gte'=> 60) ) );
```

PHP 驱动程序支持的比较运算符的完整列表可在本章末尾找到。

使用多个键值对进行查询是隐式的`AND`，而使用`$or`，`$in`，`$nin`或`AND`（`$and`）与`$or`组合的查询可以通过嵌套查询实现：

```sql
$cursor = $collection->find( array( '$or' => array(
                                            array("price" => array( '$gte' => 60)),
                                            array("price" => array( '$lte' => 20))
                                   )));
```

这将找到`price>=60 OR price<=20`的文档。

# 更新文档

更新文档与`->updateOne() OR ->updateMany()`方法具有类似的接口。

第一个参数是用于查找文档的查询，第二个参数将更新我们的文档。

我们可以使用本章末尾解释的任何更新操作符来进行原地更新，或者指定一个新文档来完全替换查询中的文档：

```sql
$result = $collection->updateOne(  array( "isbn" => "401"),
   array( '$set' => array( "price" => 39 ) )
);
```

我们可以使用单引号或双引号作为键名，但如果我们有以`$`开头的特殊操作符，我们需要使用单引号。我们可以使用`array( "key" => "value" )`或`["key" => "value"]`。在本书中，我们更喜欢更明确的`array()`表示法。

`->getMatchedCount()`和`->getModifiedCount()`方法将返回查询部分匹配的文档数量或从查询中修改的文档数量。如果新值与文档的现有值相同，则不会计为修改。

# 使用 Doctrine 进行 CRUD

继续我们在第二章*模式设计和数据建模*中的 Doctrine 示例，我们将对这些模型进行 CRUD 操作。

# 创建、更新和删除

创建文档是一个两步过程。首先，我们创建我们的文档并设置属性值：

```sql
$book = new Book();
$book->setName('MongoDB with Doctrine');
$book->setPrice(45);
```

接着，我们要求 Doctrine 在下一次`flush()`调用中保存`$book`：

```sql
$dm->persist($book);
```

我们可以通过手动调用`flush()`来强制保存，如下所示：

```sql
$dm->flush();
```

在这个例子中，`$dm`是一个`DocumentManager`对象，我们用它来连接到我们的 MongoDB 实例，如下所示：

```sql
$dm = DocumentManager::create(new Connection(), $config);
```

更新文档就像给属性赋值一样简单：

```sql
$book->price = 39;
$book->persist($book);
```

这将以新价格`39`保存我们的`MongoDB with Doctrine`书。

在原地更新文档时使用`QueryBuilder`接口。

Doctrine 提供了几个围绕原子更新的辅助方法，列举如下：

+   `set($name, $value, $atomic = true)`

+   `setNewObj($newObj)`

+   `inc($name, $value)`

+   `unsetField($field)`

+   `push($field, $value)`

+   `pushAll($field, array $valueArray)`

+   `addToSet($field, $value)`

+   `addManyToSet($field, array $values)`

+   `popFirst($field)`

+   `popLast($field)`

+   `pull($field, $value)`

+   `pullAll($field, array $valueArray)`

`update`默认会更新查询找到的第一个文档。如果我们想要更改多个文档，我们需要使用`->updateMany()`：

```sql
$dm->createQueryBuilder('Book')
   ->updateMany()
   ->field('price')->set(69)
   ->field('name')->equals('MongoDB with Doctrine')
   ->getQuery()
   ->execute();
```

在上面的例子中，我们将书名为`'MongoDB with Doctrine'`的书的价格设置为`69`。Doctrine 中的比较运算符列表在以下*读取*部分中可用。

我们可以链接多个比较运算符，得到一个`AND`查询，也可以链接多个辅助方法，得到对多个字段的更新。

删除文档与创建文档类似，如下面的代码块所示：

```sql
$dm->remove($book);
```

最好使用`QueryBuilder`接口来删除多个文档，我们将在下一节中进一步探讨：

```sql
$qb = $dm->createQueryBuilder('Book');
$qb->remove()
   ->field('price')->equals(50)
   ->getQuery()
   ->execute();
```

# 读取

Doctrine 为 MongoDB 提供了一个`QueryBuilder`接口来构建查询。鉴于我们已经在第二章*模式设计和数据建模*中描述了我们的模型，我们可以这样做来获取一个名为`$db`的`QueryBuilder`接口的实例，获取默认的查找所有查询，并执行它，如下所示：

```sql
$qb = $dm->createQueryBuilder('Book');
$query = $qb->getQuery();
$books = $query->execute();
```

`$books`变量现在包含了一个可迭代的懒加载数据加载游标，用于遍历我们的结果集。

在`QueryBuilder`对象上使用`$qb->eagerCursor(true)`将返回一个急切的游标，一旦我们开始遍历结果，就会从 MongoDB 中获取所有数据。

这里列出了一些用于查询的辅助方法：

+   `->getSingleResult()`: 这相当于`findOne()`。

+   `->select('name')`: 这将仅返回我们的`books`集合中`'key'`属性的值。`ObjectId`将始终被返回。

+   `->hint('book_name_idx')`: 强制查询使用此索引。我们将在第七章 *索引*中更多地了解索引。

+   `->distinct('name')`: 这将按名称返回不同的结果。

+   `->limit(10)`: 这将返回前`10`个结果。

+   `->sort('name', 'desc')`: 这将按名称排序（如`desc`或`asc`）。

当从 MongoDB 获取文档时，Doctrine 使用了水合概念。水合定义了查询的结果模式。例如，我们可以配置水合以返回对象集合、单个标量值或表示不同记录的数组数组。使用身份映射，它将在内存中缓存 MongoDB 结果，并在访问数据库之前查询此映射。通过使用`->hydration(false)`可以在每个查询中禁用水合，或者可以在全局范围内使用第二章*模式设计和数据建模*中解释的配置。

我们还可以通过在`$qb`上使用`->refresh()`来强制 Doctrine 从 MongoDB 中的查询刷新身份映射中的数据。

我们可以使用的比较运算符如下：

+   `where($javascript)`

+   `in($values)`

+   `notIn($values)`

+   `equals($value)`

+   `notEqual($value)`

+   `gt($value)`

+   `gte($value)`

+   `lt($value)`

+   `lte($value)`

+   `range($start, $end)`

+   `size($size)`

+   `exists($bool)`

+   `type($type)`

+   `all($values)`

+   `mod($mod)`

+   `addOr($expr)`

+   `addAnd($expr)`

+   `references($document)`

+   `includesReferenceTo($document)`

考虑以下查询作为示例：

```sql
$qb = $dm->createQueryBuilder('Book')
                ->field('price')->lt(30);
```

这将返回所有价格低于 30 的书籍。

`addAnd()`可能看起来多余，因为在 Doctrine 中链接多个查询表达式隐式是`AND`，但如果我们想要执行`AND ( (A OR B), (C OR D) )`，其中`A`、`B`、`C`和`D`是独立的表达式，它是有用的。

要嵌套多个`OR`运算符与外部`AND`查询，以及其他同样复杂的情况，需要使用`->expr()`将嵌套的`OR`作为表达式进行评估：

```sql
$expression = $qb->expr()->field('name')->equals('MongoDB with Doctrine')
```

`$expression`是一个独立的表达式，可以与`$qb->addOr($expression)`一起使用，类似地使用`addAnd()`。

# 最佳实践

使用 Doctrine 与 MongoDB 的一些最佳实践如下：

+   不要使用不必要的级联。

+   不要使用不必要的生命周期事件。

+   不要在类、字段、表或列名称中使用特殊字符，因为 Doctrine 目前不支持 Unicode。

+   在模型的构造函数中初始化集合引用。

+   尽可能限制对象之间的关系。避免模型之间的双向关联，并消除不需要的关联。这有助于性能、松散耦合，并产生更简单、更易维护的代码。

# 比较运算符

以下是 MongoDB 支持的所有比较运算符的列表：

| **名称** | **描述** |
| --- | --- |
| `$eq` | 匹配等于指定值的值 |
| `$gt` | 匹配大于指定值的值 |
| `$gte` | 匹配大于或等于指定值的值 |
| `$lt` | 匹配小于指定值的值 |
| `$lt` | 匹配小于指定值的值 |
| `$ne` | 匹配所有不等于指定值的值 |
| `$in` | 匹配数组中指定的任何值 |
| `$nin` | 匹配数组中未指定的值 |

# 更新运算符

以下是 MongoDB 支持的所有更新运算符的列表：

| **名称** | **描述** |
| --- | --- |
| `$inc` | 将字段的值增加指定的数量 |
| `$mul` | 将字段的值乘以指定的数量 |
| `$rename` | 重命名字段。 |
| `$setOnInsert` | 如果更新导致文档插入，则设置字段的值。它不影响更新操作和修改现有文档。 |
| `$set` | 设置文档中字段的值。 |
| `$unset` | 从文档中删除指定的字段。 |
| `$min` | 仅在指定值小于现有字段值时更新字段。 |
| `$max` | 仅在指定值大于现有字段值时更新字段。 |
| `$currentDate` | 将字段的值设置为当前日期，可以是日期或时间戳。 |

# 智能查询

在查询 MongoDB 时，我们必须考虑几个因素。以下是一些使用正则表达式、查询结果、游标以及删除文档的最佳实践。

# 使用正则表达式

MongoDB 提供了丰富的接口，用于使用正则表达式进行查询。在其最简单的形式中，我们可以通过修改查询字符串来使用正则表达式：

```sql
> db.books.find({"name": /mongo/})
```

这是为了在我们的`books`集合中搜索包含`mongo`名称的书籍。这相当于 SQL 的`LIKE`查询。

MongoDB 使用**Perl Compatible Regular Expression**（**PCRE**）版本 8.39，并支持 UTF-8。

在查询时，我们还可以使用一些选项：

| **选项** | **描述** |
| --- | --- |
| `i` | 此选项查询不区分大小写。 |
| `m` | 对于包含锚点（即`^`表示开头，`$`表示结尾）的模式，此选项将在每行的开头或结尾匹配多行值的字符串。如果模式不包含锚点，或者字符串值没有换行符（例如`\n`），则`m`选项不起作用。 |

在我们的先前示例中，如果我们想搜索`mongo`、`Mongo`、`MONGO`以及任何其他不区分大小写的变体，我们需要使用`i`选项，如下所示：

```sql
> db.books.find({"name": /mongo/i})
```

或者，我们可以使用`$regex`运算符，这样可以提供更多的灵活性。

使用`$regex`进行相同的查询将写成如下形式：

```sql
> db.books.find({'name': { '$regex': /mongo/ } })
> db.books.find({'name': { '$regex': /mongo/i } })
```

通过使用`$regex`运算符，我们还可以使用以下两个选项：

| **选项** | **描述** |
| --- | --- |
| `x` | 扩展功能，忽略`$regex`模式中的所有空白字符，除非它们已被转义或包含在字符类中。此外，它还忽略未转义的井号（`#`、`£`）字符及其后的字符，以便在复杂模式中包含注释。这仅适用于数据字符；空白字符永远不会出现在模式中特殊字符序列中。`x`选项不影响对 VT 字符的处理。 |
| `s` | 此选项允许点字符（即`.`）匹配所有字符，包括换行符。 |

使用正则表达式扩展匹配文档会使我们的查询执行速度变慢。

使用正则表达式的索引只能在我们的正则表达式查询字符串的开头进行使用；也就是说，以`^`或`\A`开头的正则表达式。如果我们只想使用`starts with`正则表达式进行查询，我们应该避免编写更长的正则表达式，即使它们会匹配相同的字符串。

以以下代码块为例：

```sql
> db.books.find({'name': { '$regex': /mongo/ } })
> db.books.find({'name': { '$regex': /^mongo.*/ } })
```

这两个查询都将匹配以`mongo`开头的名称值（区分大小写），但第一个查询将更快，因为它在每个名称值中的第六个字符时就会停止匹配。

# 查询结果和游标

MongoDB 不支持事务意味着我们在 RDBMS 中认为理所当然的几个语义工作方式不同。

如前所述，更新可能会修改文档的大小。修改大小可能会导致 MongoDB 将文档移动到存储文件末尾的新位置。

当我们有多个线程查询和更新单个集合时，可能会导致一个文档在结果集中出现多次。

这将发生在以下情况下：

+   线程`A`开始查询集合并匹配文档`A1`。

+   线程`B`更新文档`A1`，增加其大小，并迫使 MongoDB 将其移动到存储文件末尾的不同物理位置。

+   线程`A`仍在查询集合。它到达集合末尾，并再次找到文档`A1`，其新值如下图所示：

![](img/91064db2-8b5e-4ad4-9d26-03e08cca5006.png)

这很少见，但在生产中可能会发生；如果我们无法在应用层保护免受这种情况，我们可以使用`snapshot()`来防止它发生。

`snapshot()`由官方驱动程序和 shell 支持，通过将其附加到返回游标的操作中：

```sql
> db.books.find().snapshot()
```

`$snapshot`不能与分片集合一起使用。`$snapshot`必须在查询返回第一个文档之前应用。快照不能与`hint()`或`sort()`操作符一起使用。

我们可以通过使用`hint({id :1})`来模拟`snapshot()`的行为，从而强制查询引擎使用`id`索引，就像`$snapshot`操作符一样。

如果我们的查询在一个字段的唯一索引上运行，而这个字段的值在查询期间不会被修改，我们应该使用这个查询来获得相同的查询行为。即使如此，`snapshot()`也无法保护我们免受插入或删除发生在查询中间的影响。`$snapshot`操作符将遍历每个集合在`id`字段上具有的内置索引，使其固有地缓慢。这应该只作为最后的手段使用。

如果我们想要在进行`update`、`insert`或`delete`多个文档时，不让其他线程在操作进行时看到操作的结果，我们可以使用`$isolated`操作符：

```sql
> db.books.remove( { price: { $gt: 30 }, $isolated: 1 } )
```

在这个例子中，查询`books`集合的线程将看到价格大于`30`的所有书籍，或者根本看不到任何书籍。隔离操作符将在整个查询期间为集合获取独占写锁，无论存储引擎支持什么，都会导致这个集合的争用。

隔离操作仍然不是事务；它们不提供`原子性（"全有或全无"）`。因此，如果它们在中途失败，我们需要手动回滚操作，使我们的数据库处于一致状态。

再次强调，这应该是最后的手段，只在使得多个线程随时看到不一致信息变得至关重要的情况下使用。

# 删除操作的存储考虑

在 MongoDB 中删除文档不会回收其使用的磁盘空间。如果我们的 MongoDB 使用了 10GB 的磁盘空间，我们删除了所有文档，我们仍然会使用 10GB。在幕后发生的是，MongoDB 会将这些文档标记为已删除，并可能使用空间来存储新文档。

这将导致我们的磁盘有未使用的空间，但不会为操作系统释放。如果我们想要收回它，我们可以使用`compact()`来收回任何未使用的空间：

```sql
> db.books.compact()
```

或者，我们可以使用`--repair`选项启动`mongod`服务器。

更好的选择是启用压缩，这在 3.0 版本中可用，且仅适用于 WiredTiger 存储引擎。我们可以使用 snappy 或 zlib 算法来压缩我们的文档大小。这将再次不会防止存储空洞，但如果我们的磁盘空间紧张，这比修复和压缩的繁重操作更可取。

存储压缩使用更少的磁盘空间，但以 CPU 使用为代价，但这种权衡大多是值得的。

在运行可能导致灾难性数据丢失的操作之前，始终进行备份。修复或压缩将在单个线程中运行，阻塞整个数据库的其他操作。在生产系统中，始终先在从库上执行这些操作；然后切换主从角色，并压缩原主服务器，现在的从服务器实例。

# 变更流

变更流功能在 3.6 版本中引入，并在 4.0 版本中增强，使其成为监听数据库变更的安全有效的方式。

# 介绍

改变流解决的基本问题是应用程序需要立即对基础数据的变化做出反应。现代 Web 应用程序需要对数据变化做出反应，并在不重新加载整个页面的情况下刷新页面视图。这是前端框架（如 Angular、React 和 Vue.js）正在解决的问题之一。当用户执行操作时，前端框架将异步地向服务器提交请求，并根据服务器的响应刷新页面的相关片段。

考虑到多用户 Web 应用程序，存在数据库更改可能是由另一个用户的操作引起的情况。例如，在项目管理看板中，用户 A 可能正在查看看板，而另一个用户 B 可能正在将一个工单的状态从“待办”更改为“进行中”。

用户 A 的视图需要实时更新，以反映用户 B 所做的更改，无需刷新页面。目前已经有三种方法来解决这个问题，如下所示：

+   最简单的方法是每隔 X 秒轮询数据库并确定是否有变化。通常，此代码将需要使用某种状态、时间戳或版本号，以避免多次获取相同的变化。这种方法简单，但效率低，因为它无法随着大量用户的增加而扩展。成千上万的用户同时轮询数据库将导致数据库锁定率高。

+   为了克服第一种方法带来的问题，已经实现了数据库和应用程序级触发器。数据库触发器依赖于底层数据库对数据库更改做出响应执行一些代码。然而，主要的缺点与第一种方法类似，即我们向数据库添加的触发器越多，我们的数据库就会变得越慢。它也与数据库耦合，而不是应用程序代码库的一部分。

+   最后，我们可以使用数据库事务或复制日志来查询最新的更改并对其做出反应。这是前面提到的三种方法中最有效和可扩展的方法，因为它不会对数据库造成压力。数据库会将写入此日志，通常是仅追加的，我们的后台任务会按顺序读取日志中的条目。这种方法的缺点是它是最复杂的实现方法，如果没有正确实现，可能会导致严重的错误。

改变流提供了一种解决这个问题的方式，这种方式对开发人员友好，易于实现和维护。它基于 oplog，本质上是 MongoDB 的操作日志，包含服务器范围内所有数据库中发生的每个操作。这样，开发人员就不必处理服务器范围内的 oplog 或可追溯的游标，这些通常不会从 MongoDB 特定语言驱动程序中公开或易于处理。此外，开发人员也不必解密和理解为 MongoDB 的利益而设计和构建的任何内部 oplog 数据结构。

改变流在安全方面还有其他优势：用户只能在具有读取权限的集合、数据库或部署上创建改变流。

改变流也是幂等的设计。即使应用程序无法获取绝对最新的改变流事件通知 ID，它也可以从先前已知的 ID 开始应用，并最终达到相同的状态。

最后，更改流是可恢复的。每个更改流响应文档都包括一个恢复令牌。如果应用程序与数据库不同步，它可以将最新的恢复令牌发送回数据库，并从那里继续处理。这个令牌需要在应用程序中持久化，因为 MongoDB 驱动程序不会保留应用程序的故障和重新启动。它只会在瞬态网络故障和 MongoDB 副本集选举的情况下保持状态并重试。

# 设置

更改流可以针对集合、数据库或整个部署（如副本集或分片集群）进行打开。更改流不会对系统集合或 admin、config 和 local 数据库中的任何集合的更改做出反应。

更改流需要 WiredTiger 存储引擎和副本集协议版本 1（pv1）。从 MongoDB 4.0 开始，pv1 是唯一受支持的版本。更改流与使用加密存储的部署兼容。

# 使用更改流

要使用更改流，我们需要连接到我们的副本集。副本集是使用更改流的先决条件。由于更改流内部使用 oplog，没有 oplog 是不可能工作的。更改流还将输出在副本集设置中不会回滚的文档，因此它们需要遵循大多数读取关注。无论如何，使用副本集进行本地开发和测试是一个好习惯，因为这是生产的推荐部署方式。例如，我们将在名为`streams`的数据库中使用`signals`集合。

我们将使用以下示例 Python 代码：

```sql
from pymongo import MongoClient

class MongoExamples:
   def __init__(self):
       self.client = MongoClient('localhost', 27017)
       db = self.client.streams
       self.signals = db.signals
   # a basic watch on signals collection
   def change_books(self):
       with self.client.watch() as stream:
           for change in stream:
               print(change)
def main():
   MongoExamples().change_books()
if __name__ == '__main__':
   main()
```

我们可以打开一个终端并使用`python change_streams.py`运行它。

然后，在另一个终端中，我们使用以下代码连接到我们的 MongoDB 副本集：

```sql
> mongo
> use streams
> db.signals.insert({value: 114.3, signal:1})
```

回到我们的第一个终端窗口，我们现在可以观察到输出类似于以下代码块：

```sql
{'_id': {'_data': '825BB7A25E0000000129295A1004A34408FB07864F8F960BF14453DFB98546645F696400645BB7A25EE10ED33145BCF7A70004'}, 'operationType': 'insert', 'clusterTime': Timestamp(1538761310, 1), 'fullDocument': {'_id': ObjectId('5bb7a25ee10ed33145bcf7a7'), 'value': 114.3, 'signal': 1.0}, 'ns': {'db': 'streams', 'coll': 'signals'}, 'documentKey': {'_id': ObjectId('5bb7a25ee10ed33145bcf7a7')}}
```

这里发生的是，我们已经打开了一个游标，监视整个`streams`数据库的更改。我们数据库中的每个数据更新都将被记录并输出到控制台。

例如，如果我们回到 mongo shell，我们可以发出以下代码：

```sql
> db.a_random_collection.insert({test: 'bar'})
```

Python 代码输出应该类似于以下代码：

```sql
{'_id': {'_data': '825BB7A3770000000229295A10044AB37F707D104634B646CC5810A40EF246645F696400645BB7A377E10ED33145BCF7A80004'}, 'operationType': 'insert', 'clusterTime': Timestamp(1538761591, 2), 'fullDocument': {'_id': ObjectId('5bb7a377e10ed33145bcf7a8'), 'test': 'bar'}, 'ns': {'db': 'streams', 'coll': 'a_random_collection'}, 'documentKey': {'_id': ObjectId('5bb7a377e10ed33145bcf7a8')}}
```

这意味着我们会收到关于数据库中所有集合的每个数据更新的通知。

然后，我们可以将我们的代码的第 11 行更改为以下内容：

```sql
> with self.signals.watch() as stream:
```

这将导致只观察`signals`集合，这应该是最常见的用例。

PyMongo 的`watch`命令可以采用几个参数，如下所示：

```sql
watch(pipeline=None, full_document='default', resume_after=None, max_await_time_ms=None, batch_size=None, collation=None, start_at_operation_time=None, session=None)
```

最重要的参数如下：

+   `Pipeline`：这是一个可选参数，我们可以使用它来定义要在每个与`watch()`匹配的文档上执行的聚合管道。因为更改流本身使用聚合管道，我们可以将事件附加到它。我们可以使用的聚合管道事件如下：

```sql
$match
$project
$addFields
$replaceRoot
$redact
```

+   `Full_document`：这是一个可选参数，我们可以通过将其设置为`'updateLookup'`来使用，以使更改流返回描述文档更改的增量和在部分更新的情况下从更改发生后的一段时间内更改的整个文档的副本。

+   `Start_at_operation_time`：这是一个可选参数，我们可以使用它来仅观察在指定时间戳之后或之后发生的更改。

+   `Session`：这是一个可选参数，如果我们的驱动程序支持传递`ClientSession`对象以侦听更新。

更改流响应文档的大小不能超过 16 MB。这是 MongoDB 对 BSON 文档的全局限制，更改流必须遵循此规则。

# 规范

以下文档显示了更改事件响应可能包括或不包括的所有可能字段，具体取决于实际发生的更改：

```sql
{  _id : { <BSON Object> },
  "operationType" : "<operation>",
  "fullDocument" : { <document> },
  "ns" : {
     "db" : "<database>",
     "coll" : "<collection"
  },
  "documentKey" : { "_id" : <ObjectId> },
  "updateDescription" : {
     "updatedFields" : { <document> },
     "removedFields" : [ "<field>", ... ]
  }
  "clusterTime" : <Timestamp>,
  "txnNumber" : <NumberLong>,
  "lsid" : {
     "id" : <UUID>,
     "uid" : <BinData>
  }
}
```

最重要的字段如下：

| `fullDocument` | 这是文档的新状态，可以包括以下内容：

+   如果是删除操作，则该字段被省略，因为文档已不再存在。

+   如果是插入或替换操作，则将是文档的新值。

+   如果是更新操作，并且我们已启用`'updateLookup'`，那么它将具有更新操作修改的文档的最近主要提交的版本。

|

| `operationType` | 这是操作的类型；它可以是`insert`、`delete`、`replace`、`update`或`invalidate`中的任何一个。 |
| --- | --- |
| `documentKey` | 这是操作影响的文档的`ObjectID`。 |
| `updateDescription.updatedFields / removedFields` | 这是一个文档或相应的键数组，分别显示了更新或删除操作更新或删除的数据。 |
| `txnNumber` | 这是事务号。仅当操作是多文档 ACID 事务的一部分时才适用。 |
| `lsid` | 这是事务的会话标识符。仅当操作是多文档 ACID 事务的一部分时才适用。 |

# 重要说明

在使用分片数据库时，更改流需要针对 MongoDB 服务器打开。在使用副本集时，更改流只能针对承载数据的实例打开。从 4.0.2 版本开始，每个更改流将打开一个新连接。如果我们想要并行地有大量的更改流，我们需要增加连接池（根据 SERVER-32946 JIRA MongoDB 票）以避免严重的性能下降。

# 生产建议

更改流是 MongoDB 数据库的一个相当新的添加。因此，生产部署的以下建议可能会在以后的版本中更改。这些是 MongoDB 和专家架构师在撰写本章时推荐的指南。

# 副本集

更改流只会处理已写入大多数处理数据成员的事件。如果我们失去了存储数据服务器的大多数，或者我们依赖仲裁者来建立多数，它将暂停。

使事件无效，例如删除或重命名集合，将关闭更改流。在使事件无效后关闭更改流后，我们无法恢复更改流。

由于更改流依赖于 oplog 大小，我们需要确保 oplog 大小足够大，以容纳事件，直到应用程序处理完毕。

# 分片集群

除了副本集的考虑之外，对于分片集群还有一些需要牢记的事项。它们如下：

+   更改流在集群中的每个分片上执行，并且速度将取决于最慢的分片。

+   为了避免为孤立文档创建更改流事件，如果我们在分片下有多文档更新，我们需要使用 ACID 兼容事务的新功能。

在对未分片集合进行分片（即从副本集迁移至分片）时，更改流通知文档的`documentKey`将包括`_id`，直到更改流追上第一个分块迁移。

# 摘要

在本章中，我们通过使用官方驱动程序和 ODM，使用 Ruby、Python 和 PHP，讨论了使用这些语言进行高级查询的概念。

使用 Ruby 和 Mongoid ODM，Python 和 PyMODM ODM，以及 PHP 和 Doctrine ODM，我们通过代码示例探讨了如何`创建`、`读取`、`更新`和`删除`文档。

我们还讨论了性能和最佳实践的批处理操作。我们提供了 MongoDB 使用的比较和更新操作符的详尽列表。

最后，我们讨论了智能查询，查询中的游标工作原理，删除时应考虑的存储性能，以及如何使用正则表达式。

在下一章中，我们将学习聚合框架，使用一个涉及从以太坊区块链处理交易数据的完整用例。
