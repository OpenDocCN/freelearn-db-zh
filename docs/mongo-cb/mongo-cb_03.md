# 第三章：编程语言驱动程序

在本章中，我们将涵盖以下配方：

+   使用 PyMongo 执行查询和插入操作

+   使用 PyMongo 执行更新和删除操作

+   使用 PyMongo 在 Mongo 中实现聚合

+   使用 PyMongo 在 Mongo 中执行 MapReduce

+   使用 Java 客户端执行查询和插入操作

+   使用 Java 客户端执行更新和删除操作

+   使用 Java 客户端在 Mongo 中实现聚合

+   使用 Java 客户端在 Mongo 中执行 MapReduce

# 介绍

到目前为止，我们已经在 shell 中使用 Mongo 执行了大部分操作。Mongo shell 是管理员执行管理任务和开发人员在编写应用程序逻辑之前通过查询数据快速测试事物的绝佳工具。然而，我们如何编写应用程序代码来允许我们在 MongoDB 中查询、插入、更新和删除（以及其他操作）数据？我们编写应用程序的编程语言应该有一个库。我们应该能够实例化一些东西或从程序中调用方法来执行一些操作在远程 Mongo 进程上。

除非有一个理解与远程服务器通信协议并能够传输我们需要的操作的桥梁，以便在 Mongo 服务器进程上执行并将结果传回客户端。简单地说，这个桥梁被称为**驱动程序**，也称为客户端库。驱动程序构成了 Mongo 的编程语言接口的支柱；如果没有它们，应用程序将负责使用服务器理解的低级协议与 Mongo 服务器通信。这不仅需要开发，还需要测试和维护。尽管通信协议是标准的，但不能有一个适用于所有语言的实现。各种编程语言需要有自己的实现，向所有语言公开类似的编程接口。我们将在本章中看到的客户端 API 的核心概念对所有语言都适用。

### 提示

Mongo 支持所有主要编程语言，并得到 MongoDB Inc 的支持。社区还支持大量的编程语言。您可以访问[`docs.mongodb.org/ecosystem/drivers/community-supported-drivers/`](http://docs.mongodb.org/ecosystem/drivers/community-supported-drivers/)查看 Mongo 支持的各种平台。

# 使用 PyMongo 执行查询和插入操作

这个配方是关于使用 PyMongo 执行基本查询和“插入”操作的。这与我们在本书前面使用 Mongo shell 所做的类似。

## 准备工作

要执行简单的查询，我们需要一个正在运行的服务器。我们需要一个简单的单节点。有关如何启动服务器的说明，请参阅第一章中的*安装单节点 MongoDB*配方，*安装和启动服务器*。我们将操作的数据需要导入数据库。有关导入数据的步骤，请参阅第二章中的*创建测试数据*配方，*命令行操作和索引*。主机操作系统上必须安装 Python 2.7 或更高版本，以及 MongoDB 的 Python 客户端 PyMongo。如何在主机操作系统上安装 PyMongo，请参阅第一章中的早期配方*使用 Python 客户端连接到单节点*，*安装和启动服务器*。此外，在这个配方中，我们将执行“插入”操作并提供写关注。

## 如何做…

让我们从 Python shell 中查询 Mongo 开始。这将与我们在 mongo shell 中所做的完全相同，只是这是用 Python 编程语言而不是我们在 mongo shell 中使用的 JavaScript。我们可以使用这里将看到的基础知识来编写在 Python 上运行并使用 mongo 作为数据存储的大型生产系统。

让我们从操作系统的命令提示符开始启动 Python shell。所有这些步骤都与主机操作系统无关。执行以下步骤：

1.  在 shell 中输入以下内容，Python shell 应该启动：

```go
$ python
Python 2.7.6 (default, Mar 22 2014, 22:59:56)
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>

```

1.  然后，导入`pymongo`包，并创建客户端如下：

```go
>>> import pymongo
>>> client = pymongo.MongoClient('localhost', 27017)
The following is an alternative way to connect
>>> client = pymongo.MongoClient('mongodb://localhost:27017')

```

1.  这样做效果很好，可以实现相同的结果。现在我们有了客户端，下一步是获取我们将执行操作的数据库。这与一些编程语言不同，在那里我们有一个`getDatabase()`方法来获取数据库的实例。我们将获取一个对数据库对象的引用，我们将在其上执行操作，在这种情况下是`test`。我们将以以下方式执行此操作：

```go
>>> db = client.test
Another alternative is 
>>> db = client['test']

```

1.  我们将查询`postalCodes`集合。我们将将结果限制为 10 项。

```go
>>> postCodes = db.postalCodes.find().limit(10)

```

1.  遍历结果。注意`for`语句后的缩进。以下片段应该打印出返回的 10 个文档：

```go
>>> for postCode in postCodes:
 print 'City: ', postCode['city'], ', State: ', postCode['state'], ', Pin Code: ', postCode['pincode']

```

1.  要查找一个文档，请执行以下操作：

```go
>>> postCode = db.postalCodes.find_one()

```

1.  按以下方式打印返回结果的`state`和`city`：

```go
>>> print 'City: ', postCode['city'], ', State: ', postCode['state'], ', Pin Code: ', postCode['pincode']

```

1.  让我们查询古吉拉特邦前 10 个城市，按城市名称排序，并且额外选择`city`、`state`和`pincode`。在 Python shell 中执行以下查询：

```go
>>> cursor = db.postalCodes.find({'state':'Gujarat'}, {'_id':0, 'city':1, 'state':1, 'pincode':1}).sort('city', pymongo.ASCENDING).limit(10)

```

前面游标的结果可以以与我们在第 5 步中打印结果相同的方式打印出来。

1.  让我们对我们查询的数据进行排序。我们希望按州的降序和城市的升序进行排序。我们将编写以下查询：

```go
>>> city = db.postalCodes.find().sort([('state', pymongo.DESCENDING),('city',pymongo.ASCENDING)]).limit(5)

```

1.  通过迭代这个游标，应该在控制台上打印出五个结果。参考第 5 步，了解如何迭代返回的游标以打印结果。

1.  因此，我们已经玩了一会儿，找到了文档，并涵盖了 Python 中与查询 MongoDB 有关的基本操作。现在，让我们稍微了解一下`insert`操作。我们将使用一个测试集合来执行这些操作，而不会干扰我们的邮政编码测试数据。我们将使用`pymongoTest`集合来实现这个目的，并按以下方式向其中添加文档：

```go
>>> for i in range(1, 21):
 db.pymongoTest.insert_one({'i':i})

```

1.  `insert`可以接受一个字典对象列表并执行批量插入。因此，现在类似以下的`insert`是完全有效的：

```go
>>> db.pythonTest.insert_many([{'name':'John'}, {'name':'Mark'}])

```

对返回值有什么猜测吗？在单个文档插入的情况下，返回值是新创建文档的`_id`的值。在这种情况下，它是一个 ID 列表。

## 它是如何工作的...

在第 2 步，我们实例化客户端并获取对`MongoClient`对象的引用，该对象将用于访问数据库。有几种方法可以获取这个引用。第一个选项更方便，除非你的数据库名称中有一些特殊字符，比如连字符(-)。例如，如果名称是`db-test`，我们将不得不使用`[]`运算符来访问数据库。使用任何一种替代方案，我们现在都有了一个指向`db`变量中测试数据库的对象。在 Python 中获取`client`和`db`实例之后，我们查询以自然顺序从第 3 步的集合中找到前 10 个文档。语法与在 shell 中执行此查询的方式完全相同。第 4 步只是简单地打印出结果，这种情况下有 10 个结果。通常，如果您需要在 Python 解释器中使用类名或该类的实例获得特定类的即时帮助，只需执行`dir(<class_name>)`或`dir(<object of a class>)`，这将给出模块中定义的属性和函数的列表。例如，`dir('pymongo.MongoClient')`或`dir(client)`，其中`client`是持有`pymongo.MongoClient`实例引用的变量，可以用来获取所有支持的属性和函数的列表。`help`函数更具信息性，打印出模块的文档，并且在需要即时帮助时是一个很好的参考来源。尝试输入`help('pymongo.MongoClient')`或`help(client)`。

在第 3 和第 4 步，我们查询`postalCodes`集合，将结果限制为前 10 个结果，并将它们打印出来。返回的对象是`pymongo.cursor.Cursor`类的一个实例。下一步使用`find_one()`函数从集合中获取一个文档。这相当于在 shell 中调用集合的`findOne()`方法。这个函数返回的值是一个内置对象，`dict`。

在第 6 步，我们执行另一个`find`来查询数据。在第 8 步，我们传递了两个 Python 字典。第一个字典是查询，类似于我们在 mongo shell 中使用的查询参数。第二个字典用于提供要在结果中返回的字段。对于一个字段，值为 1 表示该值将被选择并返回在结果中。这与在关系数据库中使用`select`语句并显式提供要选择的几组列是相同的。`_id`字段默认被选择，除非在选择器`dict`对象中明确设置为零。这里提供的选择器是`{'_id':0, 'city':1, 'state':1, 'pincode':1}`，它选择了城市、州和邮政编码，并抑制了`_id`字段。我们也有一个排序方法。这个方法有两种格式，如下所示：

```go
sort(sort_field, sort_direction)
sort([(sort_field, sort_direction)…(sort_field, sort_direction)])
```

第一个用于只按一个字段排序。第二种表示接受一个排序字段和排序方向的对列表，并且用于按多个字段排序。我们在第 8 步的查询中使用了第一种形式，在第 9 步的查询中使用了第二种形式，因为我们首先按州名排序，然后按城市排序。

如果我们看一下我们调用`sort`的方式，它是在`Cursor`实例上调用的。同样，`limit`函数也是在`Cursor`类上的。评估是延迟的，直到进行迭代以从游标中检索结果为止。在这个时间点之前，`Cursor`对象不会在服务器上进行评估。

在第 11 步，我们在集合中插入一个文档 20 次。每次插入，正如我们在 Python shell 中看到的那样，都会返回一个生成的`_id`字段。在插入的语法方面，它与我们在 shell 中执行的操作完全相同。传递给插入的参数是一个`dict`类型的对象。

在第 12 步中，我们将一个文档列表传递给集合进行插入。这被称为批量插入操作，它可以在一次调用中向服务器插入多个文档。在这种情况下，返回值是一个 ID 列表，每个插入的文档都有一个 ID，并且顺序与输入列表中的顺序相同。然而，由于 MongoDB 不支持事务，每次插入都是相互独立的，一个插入的失败不会自动回滚整个操作。

添加插入多个文档的功能需要另一个参数来控制行为。在给定的列表中，如果其中一个插入失败，剩余的插入是否应该继续，还是在遇到第一个错误时停止插入？控制此行为的参数名称是`continue_on_error`，其默认值为`False`，即遇到第一个错误时停止插入。如果此值为`True`，并且在插入过程中发生多个错误，那么只有最新的错误将可用，因此默认选项为`False`是明智的。让我们看几个例子。在 Python shell 中，执行以下操作：

```go
>>> db.contOnError.drop()
>>> db.contOnError.insert([{'_id':1}, {'_id':1}, {'_id':2}, {'_id':2}])
>>> db.contOnError.count()

```

我们将得到的计数是`1`，这是第一个具有`_id`字段为`1`的文档。一旦找到另一个具有相同`_id`字段值的文档，即在本例中为`1`，就会抛出错误并停止批量插入。现在执行以下`insert`操作：

```go
>>> db.contOnError.drop()
>>> db.contOnError.insert([{'_id':1}, {'_id':1}, {'_id':2}, {'_id':2}], continue_on_error=True)
>>> db.contOnError.count()

```

在这里，我们传递了一个额外的参数`continue_on_error`，其值为`True`。这样做可以确保`insert`操作会在中间的`insert`操作失败时继续进行下一个文档的插入。第二个具有`_id:1`的插入失败，但在另一个具有`_id:2`的插入失败之前，下一个插入会继续进行（因为已经存在一个具有此`_id`的文档）。此外，报告的错误是最后一个失败的错误，即具有`_id:2`的错误。

## 另请参阅

下一个配方，*使用 PyMongo 执行更新和删除操作*，接着介绍了更新、删除和原子查找操作。

# 使用 PyMongo 执行更新和删除操作

在上一个配方中，我们看到了如何使用 PyMongo 在 MongoDB 中执行`find`和`insert`操作。在本配方中，我们将看到如何在 Python 中执行更新和删除操作。我们还将看到原子查找和更新/删除是什么，以及如何执行它们。最后，我们将重新审视查找操作，并了解`cursor`对象的一些有趣的功能。

## 准备工作

如果您已经看过并完成了上一个配方，那么您就可以开始了。如果没有，建议您在继续本配方之前先完成那个配方。此外，如果您不确定读取偏好和写入关注是什么，请参考本书的附录中的两个配方，*用于查询的读取偏好*和*写入关注及其重要性*。

在我们开始之前，让我们定义一个小函数，它通过游标迭代并在控制台上显示游标的结果。每当我们想要在`pymongoTests`集合上显示查询结果时，我们将使用这个函数。以下是函数体：

```go
>>> def showResults(cursor):
 if cursor.count() != 0:
 for e in cursor:
 print e
 else:
 print 'No documents found'

```

您可以参考上一个配方中的步骤 1 和步骤 2，了解如何连接到 MongoDB 服务器以及用于在该数据库上执行 CRUD 操作的`db`对象。此外，还可以参考上一个配方中的第 8 步，了解如何在`pymongoTest`集合中插入所需的测试数据。一旦数据存在，您可以在 Python shell 中执行以下操作来确认该集合中的数据：

```go
>>> showResults(db.pymongoTest.find())

```

对于食谱的一部分，人们也应该知道如何启动一个副本集实例。有关副本集的更多细节以及如何启动副本集，请参阅第一章中的*作为副本集的一部分启动多个实例*和*在 shell 中连接到副本集以查询和插入数据*食谱。

## 操作步骤…

我们将从 Python shell 中运行以下命令开始：

1.  如果`i`字段的值大于 10，则我们将设置一个名为`gtTen`的字段，并指定一个布尔值`True`。让我们执行以下更新：

```go
>>>result = db.pymongoTest.update_one({'i':{'$gt':10}}, {'$set':{'gtTen':True}})
>>> print result.raw_result
{u'n': 1, u'nModified': 0, u'ok': 1, 'updatedExisting': True}

```

1.  查询集合，通过执行以下操作查看其数据，并检查已更新的数据：

```go
>>> showResults(db.pymongoTest.find())

```

1.  显示的结果证实只有一个文档被更新。现在我们将再次执行相同的更新，但这一次，我们将更新所有与提供的查询匹配的文档。在 Python shell 中执行以下更新。注意响应中的 n 的值，这次是`10`。

```go
>>> result = db.pymongoTest.update_many({'i':{'$gt':10}},{'$set':{'gtTen':True}})
print result.raw_result
{u'n': 10, u'nModified': 9, u'ok': 1, 'updatedExisting': True}

```

1.  再次执行我们在步骤 2 中进行的操作，查看`pymongoTest`集合中的内容，并验证已更新的文档。

1.  让我们看看如何执行`upsert`操作。Upserts 是更新加插入，如果文档存在则更新文档，就像更新操作一样，否则插入一个新文档。我们将看一个例子。考虑对集合中不存在的文档进行以下更新：

```go
>>> db.pymongoTest.update_one({'i':21},{'$set':{'gtTen':True}})

```

1.  这里的更新不会更新任何内容，并且返回更新的文档数为零。但是，如果我们想要更新一个文档（如果存在），否则插入一个新文档并原子性地应用更新，那么我们执行一个`upsert`操作。在这种情况下，`upsert`操作执行如下。请注意返回结果中提到的`upsert`，新插入文档的`ObjectId`和`updatedExisting`值，这个值是`False`：

```go
>>>result = db.pymongoTest.update_one({'i':21},{'$set':{'gtTen':True}}, upsert=True)
>>> print result.raw_result
{u'n': 1,
 u'nModified': 0,
 u'ok': 1,
 'updatedExisting': False,
 u'upserted': ObjectId('557bd3a618292418c38b046d')}

```

1.  让我们看看如何使用`remove`方法从集合中删除文档：

```go
>>>result = db.pymongoTest.delete_one({'i':21})
>>> print result.raw_result
{u'n': 1, u'ok': 1}

```

1.  如果我们查看前面响应中`n`的值，我们可以看到它是`1`。这意味着已删除一个文档。

1.  要从集合中删除多个文档，我们使用`delete_many`方法：

```go
>>>result = db.pymongoTest.delete_many({'i':{'$gt': 10}})
>>> print result.raw_result
{u'n': 10, u'ok': 1}

```

1.  我们现在将看一下查找和修改操作。我们可以将这些操作视为查找文档并更新/删除它的一种方式，这两种操作都是原子性执行的。操作完成后，返回的文档要么是更新操作之前的文档，要么是更新操作之后的文档。（在`remove`的情况下，操作后将没有文档。）在没有这种操作的情况下，我们无法保证在多个客户端连接可能对同一文档执行类似操作的情况下的原子性。以下是如何在 Python 中执行此查找和修改操作的示例：

```go
>>> db.pymongoTest.find_one_and_update({'i':20}, {'$set':{'inWords':'Twenty'}})
{u'_id': ObjectId('557bdb070640fd0a0a935c22'), u'i': 20}

```

### 提示

前面的结果告诉我们，在应用更新之前返回的文档是之前的文档。

1.  执行以下`find`方法来查询并查看我们在上一步中更新的文档。结果文档将包含在`Words`字段中新增的内容：

```go
>>> db.pymongoTest.find_one({'i':20})
{u'i': 20, u'_id': ObjectId('557bdb070640fd0a0a935c22'), u'inWords': u'Twenty'}

```

1.  我们将再次执行`find`和`modify`操作，但这一次，我们返回更新后的文档，而不是在步骤 9 中看到的更新前的文档。在 Python shell 中执行以下操作：

```go
>>> db.pymongoTest.find_one_and_update({'i':19}, {'$set':{'inWords':'Nineteen'}}, new=True)
{u'_id': ObjectId('557bdb070640fd0a0a935c21'), u'i': 19, u'inWords': u'Nineteen'}

```

1.  我们在上一个食谱中看到了如何使用 PyMongo 进行查询。在这里，我们将继续进行查询操作。我们看到`sort`和`limit`函数是如何链接到 find 操作的。对`postalCodes`集合的调用原型如下：

```go
db.postalCode.find(..).limit(..).sort(..)

```

1.  有一种实现相同结果的替代方法。在 Python shell 中执行以下查询：

```go
>>>cursor = db.postalCodes.find({'state':'Gujarat'}, {'_id':0, 'city':1, 'state':1, 'pincode':1}, limit=10, sort=[('city', pymongo.ASCENDING)])

```

1.  使用已定义的`showResult`函数打印前面的游标。

## 工作原理…

让我们看看在这个示例中我们做了什么；我们从在步骤 1 中更新集合中的文档开始。但是，默认情况下，更新操作只更新第一个匹配的文档，其余匹配的文档不会被更新。在步骤 2 中，我们添加了一个名为`multi`的参数，其值为`True`，以便在同一更新操作中更新多个文档。请注意，所有这些文档不会作为一个事务的一部分被原子更新。在 Python shell 中查看更新后，我们看到与在 Mongo shell 中所做的操作非常相似。如果我们想要为更新操作命名参数，那么提供为查询使用的文档的参数名称称为`spec`，而更新文档的参数名称称为`document`。例如，以下更新是有效的：

```go
>>> db.pymongoTest.update_one(spec={'i':{'$gt':10}},document= {'$set':{'gtTen':True}})

```

在第 6 步，我们进行了`upsert`（更新加插入）操作。我们只有一个额外的参数`upsert`，其值为`True`。但是，在`upsert`的情况下到底发生了什么呢？Mongo 尝试更新匹配提供条件的文档，如果找到一个，则这将是一个常规更新。但是，在这种情况下（第 6 步的`upsert`），未找到文档。服务器将以原子方式将作为查询条件的文档（第一个参数）插入到集合中，然后对其进行更新。

在第 7 和第 9 步，我们看到了`remove`操作。第一个变体接受一个查询，并删除匹配的文档。第二个变体在第 9 步中删除了所有匹配的文档。

在第 10 到 12 步，我们执行了`find`和`modify`操作。这些操作的要点非常简单。我们没有提到的是`find_one_and_replace()`方法，顾名思义，可以用来搜索文档并完全替换它。

在这个示例中看到的所有操作都是针对连接到独立实例的客户端的。如果您连接到一个副本集，客户端将以不同的方式实例化。我们也知道，默认情况下不允许查询副本节点的数据。我们需要在连接到副本节点的 mongo shell 中显式执行`rs.slaveOk()`来查询它。在 Python 客户端中也是以类似的方式执行。如果我们连接到副本节点，不能默认查询它，但是我们指定我们可以查询副本节点的方式略有不同。从 PyMongo 3.0 开始，我们现在可以在初始化`MongoClient`时传递`ReadPreference`。这主要是因为，从 PyMongo 3.0 开始，`pymongo.MongoClient()`是连接到独立实例、副本集或分片集群的唯一方式。可用的读取偏好包括`PRIMARY`、`SECONDARY`、`PRIMARY_PREFERRED`、`SECONDARY_PREFERRED`和`NEAREST`。

```go
>> client = pymongo.MongoClient('localhost', 27017, readPreference='secondaryPreferred')
>> print cl.read_preference
SecondaryPreferred(tag_sets=None)

```

除了客户端之外，PyMongo 还允许您在数据库或集合级别设置读取偏好。

默认情况下，未显式设置读取偏好的初始化客户端的`read_preference`为`PRIMARY`（值为零）。但是，如果我们现在从先前初始化的客户端获取数据库对象，则读取偏好将为`NEAREST`（值为`4`）。

```go
>>> db = client.test
>>> db.read_preference
Primary()
>>>

```

设置读取偏好就像做以下操作一样简单：

```go
>>>db =  client.get_database('test', read_preference=ReadPreference.SECONDARY)

```

同样，由于读取偏好从客户端继承到数据库对象，它也从数据库对象继承到集合对象。这将作为针对该集合执行的所有查询的默认值，除非在`find`操作中显式指定读取偏好。

因此，`db.pymongoTest.find_one()`将使用读取偏好作为`SECONDARY`（因为我们刚刚在数据库对象级别将其设置为`SECONDARY`）。

现在，我们将通过尝试在 Python 驱动程序中完成基本操作来结束。这些操作与我们在 mongo shell 中进行的一些常见操作相似，例如获取所有数据库名称，获取数据库中集合的列表，并在集合上创建索引。

在 shell 中，我们显示`dbs`以显示连接的 mongo 实例中的所有数据库名称。从 Python 客户端，我们对客户端实例执行以下操作：

```go
>>> client.database_names()
[u'local', u'test']

```

同样，要查看集合的列表，我们在 mongo shell 中执行`show collections`；在 Python 中，我们对数据库对象执行以下操作：

```go
>>> db.collection_names()
[u'system.indexes', u'writeConcernTest', u'pymongoTest']

```

现在对于`index`操作；我们首先查看`pymongoTest`集合中存在的所有索引。在 Python shell 中执行以下操作以查看集合上的索引：

```go
>>> db.pymongoTest.index_information()
{u'_id_': {u'key': [(u'_id', 1)], u'ns': u'test.pymongoTest', u'v': 1}}

```

现在，我们将按照以下方式在`pymongoTest`集合上对键`x`创建一个升序排序的索引：

```go
>>>from pymongo import IndexModel, ASCENDING
>>> myindex = IndexModel([("x", ASCENDING)], name='Index_on_X')
>>>db.pymongoTest.create_indexes([myindex])
 ['Index_on_X']

```

我们可以再次列出索引以确认索引的创建：

```go
>>> db.pymongoTest.index_information()
{u'Index_on_X': {u'key': [(u'x', 1)], u'ns': u'test.pymongoTest', u'v': 1},
 u'_id_': {u'key': [(u'_id', 1)], u'ns': u'test.pymongoTest', u'v': 1}}

```

我们可以看到索引已经创建。删除索引也很简单，如下所示：

```go
db.pymongoTest.drop_index('Index_on_X')

```

另一个参数叫做`CursorType.TAILABLE`，用于表示`find`返回的游标是一个可追加的游标。解释可追加游标并提供更多细节不在这个配方的范围内，将在第五章的*创建和追加一个被截断的集合游标在 MongoDB 中*配方中进行解释。

# 使用 PyMongo 在 Mongo 中实现聚合

我们已经在之前的配方中看到了 PyMongo 使用 Python 的客户端接口来连接 Mongo。在这个配方中，我们将使用邮政编码集合并使用 PyMongo 运行一个聚合示例。这个配方的目的不是解释聚合，而是展示如何使用 PyMongo 实现聚合。在这个配方中，我们将根据州名对数据进行聚合，并获取出现次数最多的五个州名。我们将使用`$project`、`$group`、`$sort`和`$limit`操作符进行这个过程。

## 准备工作

要执行聚合操作，我们需要运行一个服务器。一个简单的单节点就足够了。请参考第一章中的*安装单节点 MongoDB*配方，了解如何启动服务器的说明。我们将要操作的数据需要导入到数据库中。导入数据的步骤在第二章的*创建测试数据*配方中有提到。此外，请参考第一章中的*使用 Python 客户端连接到单节点*配方，了解如何为您的主机操作系统安装 PyMongo。由于这是在 Python 中实现聚合的一种方式，假设读者已经了解了 MongoDB 中的聚合框架。

## 操作步骤如下：

1.  通过在命令提示符中输入以下内容来打开 Python 终端：

```go
$ Python

```

1.  一旦 Python shell 打开，按照以下方式导入`pymongo`：

```go
>>> import pymongo

```

1.  创建`MongoClient`的实例如下：

```go
>>> client = pymongo.MongoClient('mongodb://localhost:27017')

```

1.  按照以下方式获取测试数据库的对象：

```go
>>> db = client.test

```

1.  现在，我们按照以下步骤在`postalCodes`集合上执行聚合操作：

```go
result = db.postalCodes.aggregate(
 [
 {'$project':{'state':1, '_id':0}},
 {'$group':{'_id':'$state', 'count':{'$sum':1}}},
 {'$sort':{'count':-1}},
 {'$limit':5}
 ]
 )

```

1.  输入以下内容以查看结果：

```go
>>>for r in result:
print r
{u'count': 6446, u'_id': u'Maharashtra'}
{u'count': 4684, u'_id': u'Kerala'}
{u'count': 3784, u'_id': u'Tamil Nadu'}
{u'count': 3550, u'_id': u'Andhra Pradesh'}
{u'count': 3204, u'_id': u'Karnataka'}

```

## 工作原理如下：

这些步骤非常简单。我们已经连接到运行在本地主机上的数据库，并创建了一个数据库对象。我们在集合上调用的聚合操作使用了`aggregate`函数，这与我们在 shell 中调用聚合的方式非常相似。返回值中的对象`result`是一个游标，它在迭代时返回一个`dict`类型的对象。这个`dict`包含两个键，分别是州名和它们出现次数的计数。在第 6 步中，我们只是简单地迭代游标（result）以获取每个结果。

# 在 PyMongo 中执行 MapReduce

在我们之前的配方*使用 PyMongo 在 Mongo 中实现聚合*中，我们看到了如何使用 PyMongo 在 Mongo 中执行聚合操作。在这个配方中，我们将处理与聚合操作相同的用例，但是我们将使用 MapReduce。我们的目的是根据州名对数据进行聚合，并获取出现次数最多的五个州名。

编程语言驱动程序为我们提供了一个接口，用于在服务器上调用用 JavaScript 编写的 map reduce 作业。

## 准备工作

要执行 map reduce 操作，我们需要启动并运行一个服务器。一个简单的单节点就是我们需要的。请参考第一章中的*安装单节点 MongoDB*配方，了解如何启动服务器的说明。我们将要操作的数据需要导入到数据库中。导入数据的步骤在第二章的*创建测试数据*配方中有提到。另外，请参考第一章中的*使用 Python 客户端连接到单节点*配方，了解如何为您的主机操作系统安装 PyMongo。

## 操作步骤如下：

1.  通过在命令提示符上输入以下内容打开 Python 终端：

```go
>>>python

```

1.  一旦 Python shell 打开，导入`bson`包，如下所示：

```go
>>> import bson

```

1.  导入`pymongo`包，如下所示：

```go
>>> import pymongo

```

1.  创建一个`MongoClient`对象，如下所示：

```go
>>> client = pymongo.MongoClient('mongodb://localhost:27017')

```

1.  获取测试数据库的对象，如下所示：

```go
>>> db = client.test

```

1.  编写以下`mapper`函数：

```go
>>>  mapper = bson.Code('''function() {emit(this.state, 1)}''')

```

1.  编写以下`reducer`函数：

```go
>>>  reducer = bson.Code('''function(key, values){return Array.sum(values)}''')

```

1.  调用 map reduce；结果将被发送到`pymr_out`集合：

```go
>>>  db.postalCodes.map_reduce(map=mapper, reduce=reducer, out='pymr_out')

```

1.  验证结果如下：

```go
>>>  c = db.pymr_out.find(sort=[('value', pymongo.DESCENDING)], limit=5)
>>> for elem in c:
...     print elem
...
{u'_id': u'Maharashtra', u'value': 6446.0}
{u'_id': u'Kerala', u'value': 4684.0}
{u'_id': u'Tamil Nadu', u'value': 3784.0}
{u'_id': u'Andhra Pradesh', u'value': 3550.0}
{u'_id': u'Karnataka', u'value': 3204.0}
>>>

```

## 工作原理

除了常规的`pymongo`导入外，这里我们还导入了`bson`包。这就是我们有`Code`类的地方；它是我们用于 JavaScript `map`和`reduce`函数的`Python`对象。通过将 JavaScript 函数体作为构造函数参数来实例化它。

一旦实例化了两个`Code`类的实例，一个用于`map`，另一个用于`reduce`，我们所做的就是在集合上调用`map_reduce`函数。在这种情况下，我们传递了三个参数：两个`map`和`reduce`函数的`Code`实例，参数名称分别为`map`和`reduce`，以及一个字符串值，用于提供结果写入的输出集合的名称。

我们不会在这里解释 map reduce JavaScript 函数，但它非常简单，它所做的就是将键作为州名，并将值作为特定州名出现的次数。使用的结果文档，将州名作为`_id`字段，另一个名为 value 的字段，它是给定在`_id`字段中出现的特定州名的次数的总和，添加到输出集合`pymr_out`中。例如，在整个集合中，州`马哈拉施特拉邦`出现了`6446`次，因此马哈拉施特拉邦的文档是`{u'_id': u'马哈拉施特拉邦', u'value': 6446.0}`。要验证结果是否正确，您可以在 mongo shell 中执行以下查询，并查看结果是否确实为`6446`：

```go
> db.postalCodes.count({state:'Maharashtra'})
6446

```

我们还没有完成，因为要求是找到集合中出现次数最多的五个州；我们只有州和它们的出现次数，所以最后一步是按值字段对文档进行排序，即州名出现的次数，按降序排序，并将结果限制为五个文档。

## 另请参阅

请参考第八章中的*与 Hadoop 集成*，了解在 MongoDB 中使用 Hadoop 连接器执行 map reduce 作业的不同示例。这允许我们在诸如 Java、Python 等语言中编写`map`和`reduce`函数。

# 使用 Java 客户端执行查询和插入操作

在这个示例中，我们将使用 Java 客户端执行 MongoDB 的查询和`insert`操作。与 Python 编程语言不同，Java 代码片段无法从交互式解释器中执行，因此我们将已经实现了一些单元测试用例，其相关代码片段将被展示和解释。

## 准备工作

对于这个示例，我们将启动一个独立的实例。请参考第一章中的*安装单节点 MongoDB*示例，了解如何启动服务器的说明。

下一步是从 Packt 网站下载 Java 项目`mongo-cookbook-javadriver`。这个示例使用一个 JUnit 测试用例来测试 Java 客户端的各种功能。在整个过程中，我们将使用一些最常见的 API 调用，因此学会如何使用它们。

## 如何做…

执行测试用例，可以在类似 Eclipse 的 IDE 中导入项目并执行测试用例，也可以使用 Maven 从命令提示符中执行测试用例。

我们将执行此示例的测试用例是`com.packtpub.mongo.cookbook.MongoDriverQueryAndInsertTest`。

1.  如果您正在使用 IDE，请打开此测试类并将其执行为 JUnit 测试用例。

1.  如果您计划使用 Maven 来执行此测试用例，请转到命令提示符，更改项目根目录下的目录，并执行以下命令来执行此单个测试用例：

```go
$ mvn -Dtest=com.packtpub.mongo.cookbook.MongoDriverQueryAndInsertTest test

```

如果 Java SDK 和 Maven 已经正确设置，并且 MongoDB 服务器已经启动并监听端口`27017`以接收连接，那么一切都应该正常执行，测试用例应该成功。

## 工作原理…

我们现在将打开我们执行的测试类，并查看`test`方法中的一些重要的 API 调用。我们的`test`类的超类是`com.packtpub.mongo.cookbook.AbstractMongoTest`。

我们首先看一下这个类中的`getClient`方法。已创建的`client`实例是`com.mongodb.MongoClient`类型的实例。对于这个类有几个重载的构造函数；然而，我们使用以下方法来实例化客户端：

```go
MongoClient client = new MongoClient("localhost:27017");
```

另一个要看的方法是在同一个抽象类中的`getJavaDriverTestDatabase`，它可以获取数据库实例。这个实例类似于 shell 中的隐式变量`db`。在 Java 中，这个类是`com.mongodb.DB`类型的实例。我们通过在客户端实例上调用`getDB()`方法来获取这个`DB`类的实例。在我们的情况下，我们想要`javaDriverTest`数据库的`DB`实例，我们可以通过以下方式获取：

```go
getClient().getDB("javaDriverTest");
```

一旦我们获得了`com.mongodb.DB`的实例，我们就可以使用它来获得`com.mongodb.DBCollection`的实例，这将用于执行各种操作——在集合上进行`find`和`insert`操作。抽象测试类中的`getJavaTestCollection`方法返回`DBCollection`的一个实例。我们通过在`com.mongodb.DB`上调用`getCollection()`方法来获取`javaTest`集合的这个类的一个实例，如下所示：

```go
getJavaDriverTestDatabase().getCollection("javaTest")
```

一旦我们获得了`DBCollection`的实例，我们现在可以对其执行操作。在这个示例的范围内，它仅限于`find`和`insert`操作。

现在，我们打开主测试用例类`com.packtpub.mongo.cookbook.MongoDriverQueryAndInsertTest`。在 IDE 或文本编辑器中打开这个类。我们将查看这个类中的方法。我们将首先查看的方法是`findOneDocument`。在这里，我们感兴趣的行是查询`_id`值为`3`的文档的行：`collection.findOne(new BasicDBObject("_id", 3))`。

该方法返回一个`com.mongodb.DBObject`的实例，它是一个键值映射，返回文档的字段作为键和相应键的值。例如，要从返回的`DBObject`实例中获取`_id`的值，我们在返回的结果上调用`result.get("_id")`。

我们要检查的下一个方法是`getDocumentsFromTestCollection`。这个测试用例在集合上执行了一个`find`操作，并获取其中的所有文档。`collection.find()`调用在`DBCollection`的实例上执行`find`操作。`find`操作的返回值是`com.mongodb.DBCursor`。值得注意的一点是，调用`find`操作本身并不执行查询，而只是返回`DBCursor`的实例。这是一个不消耗服务器端资源的廉价操作。实际查询只有在`DBCursor`实例上调用`hasNext`或`next`方法时才在服务器端执行。`hasNext()`方法用于检查是否有更多的结果，`next()`方法用于导航到结果中的下一个`DBObject`实例。对`DBCursor`实例返回的示例用法是遍历结果：

```go
while(cursor.hasNext()) {
  DBObject object = cursor.next();
  //Some operation on the returned object to get the fields and
  //values in the document
}
```

现在，我们来看一下`withLimitAndSkip`和`withQueryProjectionAndSort`两个方法。这些方法向我们展示了如何对结果进行排序、限制数量和跳过初始结果的数量。正如我们所看到的，排序、限制和跳过方法是链接在一起的：

```go
DBCursor cursor = collection
        .find(null)
        .sort(new BasicDBObject("_id", -1))
        .limit(2)
        .skip(1);
```

所有这些方法都返回`DBCursor`的实例本身，允许我们链接调用。这些方法在`DBCursor`类中定义，根据它们在实例中执行的操作更改某些状态，并在方法的末尾返回相同的实例。

请记住，实际操作只有在`DBCursor`上调用`hasNext`或`next`方法时才在服务器上执行。在服务器上执行查询后调用任何方法，如`sort`、`limit`和`skip`，都会抛出`java.lang.IllegalStateException`。

我们使用了`find`方法的两种变体。一个接受一个参数用于执行查询，另一个接受两个参数——第一个用于查询，第二个是另一个`DBObject`，用于投影，将仅返回结果中文档的一组选定字段。

例如，测试用例的`withQueryProjectionAndSort`方法中的以下查询选择所有文档作为第一个参数为`null`，返回的`DBCursor`将包含只包含一个名为`value`的字段的文档：

```go
DBCursor cursor = collection
      .find(null, new BasicDBObject("value", 1).append("_id", 0))
      .sort(new BasicDBObject("_id", 1));
```

`_id`字段必须显式设置为`0`，否则默认情况下将返回它。

最后，我们来看一下测试用例中的另外两个方法，`insertDataTest`和`insertTestDataWithWriteConcern`。在这两种方法中，我们使用了`insert`方法的几种变体。所有`insert`方法都在`DBCollection`实例上调用，并返回一个`com.mongodb.WriteResult`的实例。结果可以用于通过调用`getLastError()`方法获取写操作期间发生的错误，使用`getN()`方法获取插入的文档数量，以及操作的写关注。有关方法的更多详细信息，请参阅 MongoDB API 的 Javadoc。我们执行的两个插入操作如下：

```go
collection.insert(new BasicDBObject("value", "Hello World"));

collection.insert(new BasicDBObject("value", "Hello World"), WriteConcern.JOURNALED);
```

这两个方法都接受一个`DBObject`实例作为要插入的文档的第一个参数。第二个方法允许我们提供用于`write`操作的写关注。`DBCollection`类中还有允许批量插入的`insert`方法。有关`insert`方法的各种重载版本的更多细节，请参考 Javadocs。

## 另请参阅…

MongoDB 驱动当前版本的 Javadocs 可以在[`api.mongodb.org/java/current/`](https://api.mongodb.org/java/current/)找到。

# 使用 Java 客户端执行更新和删除操作

在上一个配方中，我们看到了如何使用 Java 客户端在 MongoDB 中执行`find`和`insert`操作；在这个配方中，我们将看到 Java 客户端中更新和删除的工作原理。

## 准备工作

对于这个配方，我们将启动一个独立的实例。请参考第一章中的*安装单节点 MongoDB*配方，了解如何启动服务器的说明。

下一步是从 Packt 网站下载 Java 项目`mongo-cookbook-javadriver`。这个配方使用一个 JUnit 测试用例来测试 Java 客户端的各种功能。在整个过程中，我们将使用一些最常见的 API 调用，并学会使用它们。

## 如何操作…

要执行测试用例，可以在类似 Eclipse 的 IDE 中导入项目并执行测试用例，或者使用 Maven 从命令提示符中执行测试用例。

我们将为这个配方执行的测试用例是`com.packtpub.mongo.cookbook.MongoDriverUpdateAndDeleteTest`。

1.  如果您使用的是 IDE，请打开这个测试类并将其作为 JUnit 测试用例执行。

1.  如果您计划使用 Maven 执行此测试用例，请转到命令提示符，更改到项目的根目录，并执行以下命令来执行这个单个测试用例：

```go
$ mvn -Dtest=com.packtpub.mongo.cookbook.MongoDriverUpdateAndDeleteTest test

```

如果 Java SDK 和 Maven 已经正确设置，并且 MongoDB 服务器正在运行并监听端口`27017`以接收连接，则一切都应该正常执行。

## 工作原理…

我们使用`setupUpdateTestData()`方法为配方创建了测试数据。在这里，我们只是将文档放在`javaDriverTest`数据库的`javaTest`集合中。我们向这个集合添加了 20 个文档，`i`的值范围从 1 到 20。这些测试数据在不同的测试用例方法中用于创建测试数据。

现在让我们看看这个类中的方法。我们首先看一下`basicUpdateTest()`。在这个方法中，我们首先创建测试数据，然后执行以下更新：

```go
collection.update(
  new BasicDBObject("i", new BasicDBObject("$gt", 10)),
  new BasicDBObject("$set", new BasicDBObject("gtTen", true)));
```

这里的`update`方法接受两个参数。第一个是用于选择更新的符合条件的文档的查询，第二个参数是实际的更新。第一个参数由于嵌套的`BasicDBObject`实例看起来很混乱；然而，它是`{'i' : {'$gt' : 10}}`条件，第二个参数是更新，`{'$set' : {'gtTen' : true}}`。更新的结果是`com.mongodb.WriteResult`的一个实例。`WriteResult`的实例告诉我们更新的文档数量，执行`write`操作时发生的错误以及用于更新的写关注。更多细节请参考`WriteConcern`类的 Javadocs。默认情况下，此更新仅更新第一个匹配的文档，如果有多个文档匹配查询，则只更新第一个匹配的文档。

我们将要看的下一个方法是`multiUpdateTest`，它将更新给定查询的所有匹配文档，而不是第一个匹配的文档。我们使用的方法是在集合实例上使用`updateMulti`。`updateMulti`方法是更新多个文档的便捷方式。以下是我们调用的更新多个文档的方法：

```go
collection.updateMulti(new BasicDBObject("i",
    new BasicDBObject("$gt", 10)),
    new BasicDBObject("$set", new BasicDBObject("gtTen", true)));
```

我们接下来做的操作是删除文档。删除文档的测试用例方法是`deleteTest()`。文档被删除如下：

```go
collection.remove(new BasicDBObject(
      "i", new BasicDBObject("$gt", 10)),
      WriteConcern.JOURNALED);
```

我们有两个参数。第一个是查询，匹配的文档将从集合中删除。请注意，默认情况下，所有匹配的文档都将被删除，不像更新，更新默认情况下只会删除第一个匹配的文档。第二个参数是用于`remove`操作的写关注。

请注意，当服务器在 32 位机器上启动时，默认情况下禁用了日志记录。在这些机器上使用写关注可能会导致操作失败，并出现以下异常：

```go
com.mongodb.CommandFailureException: { "serverUsed" : "localhost/127.0.0.1:27017" , "connectionId" : 5 , "n" : 0 , "badGLE" : { "getlasterror" : 1 , "j" : true} , "ok" : 0.0 , "errmsg" : "cannot use 'j' option when a host does not have journaling enabled" , "code" : 2}

```

这将需要服务器使用`--journal`选项启动。在 64 位机器上，默认情况下启用了日志记录，因此这是不必要的。

接下来我们将看看`findAndModify`操作。执行此操作的测试用例方法是`findAndModifyTest`。以下代码行用于执行此操作：

```go
DBObject old = collection.findAndModify(
    new BasicDBObject("i", 10),
    new BasicDBObject("i", 100));
```

该操作是查询将找到匹配的文档，然后更新它们。该操作的返回类型是在应用更新之前的`DBObject`实例。`findAndModify`操作的一个重要特性是`find`和`update`操作是原子执行的。

前面的方法是`findAndModify`操作的简单版本。有一个重载版本的方法，签名如下：

```go
DBObject findAndModify(DBObject query, DBObject fields, DBObject sort,boolean remove, DBObject update, boolean returnNew, boolean upsert)

```

让我们看看以下表中这些参数是什么：

| 参数 | 描述 |
| --- | --- |
| `query` | 这是用于查询文档的查询。这是将被更新/删除的文档。 |
| `fields` | `find`方法支持选择结果文档中需要选择的字段的投影。此处的参数执行相同的操作，从结果文档中选择固定的字段集。 |
| `sort` | 如果你还没有注意到，让我告诉你，该方法只能对一个文档执行这个原子操作，并且只返回一个文档。`sort`函数可以用于查询选择多个文档并且只选择第一个文档进行操作的情况。在选择要更新的第一个文档之前，`sort`函数被应用于结果。 |
| `remove` | 这是一个布尔标志，指示是删除还是更新文档。如果该值为`true`，则将删除文档。 |
| `update` | 与前面的属性不同，这不是一个布尔值，而是一个`DBObject`实例，它将告诉更新需要什么。请注意，`remove`布尔标志优先于此参数；如果`remove`属性为`true`，即使提供了更新，更新也不会发生。 |
| `returnNew` | 查找操作返回一个文档，但是哪一个？在执行更新之前的文档还是在执行更新之后的文档？当给定为`true`时，这个布尔标志在更新执行后返回文档。 |
| `upsert` | 这又是一个布尔标志，当为`true`时执行`upsert`。这仅在预期操作为更新时才相关。 |

此操作还有更多的重载方法。请参考`com.mongodb.DBCollection`的 Javadocs 以获取更多方法。我们使用的`findAndModify`方法最终调用了我们讨论的方法，其中字段和排序参数为空，其余参数`remove`，`returnNew`和`upsert`都为`false`。

最后，我们来看看 MongoDB 的 Java API 中的查询构建器支持。

mongo 中的所有查询都是`DBObject`实例，其中可能包含更多嵌套的`DBObject`实例。对于小查询来说很简单，但对于更复杂的查询来说会变得很丑陋。考虑一个相对简单的查询，我们想查询`i > 10`和`i < 15`的文档。这个 mongo 查询是`{$and:[{i:{$gt:10}}`，`{i:{$lt:15}}]}`。在 Java 中编写这个查询意味着使用`BasicDBObject`实例，这甚至更加痛苦，如下所示：

```go
    DBObject query = new BasicDBObject("$and",
      new BasicDBObject[] {
        new BasicDBObject("i", new BasicDBObject("$gt", 10)),
        new BasicDBObject("i", new BasicDBObject("$lt", 15))
      });
```

然而，值得庆幸的是，有一个名为`com.mongodb.QueryBuilder`的类，这是一个用于构建复杂查询的实用类。前面的查询是使用查询构建器构建的，如下所示：

```go
DBObject query = QueryBuilder.start("i").greaterThan(10).and("i").lessThan(15).get();
```

在编写查询时，这样做错误的可能性较小，而且阅读起来也很容易。`com.mongodb.QueryBuilder`类中有很多方法，我鼓励您阅读该类的 Javadocs。基本思想是使用`start`方法和键开始构建，然后链接方法调用以添加不同的条件，当添加各种条件完成时，使用`get()`方法构建查询，该方法返回`DBObject`。请参考测试类中的`queryBuilderSample`方法，了解 MongoDB Java API 的查询构建器支持的示例用法。

## 另请参阅

还有一些使用 GridFS 和地理空间索引的操作。我们将在高级查询章节中看到如何在 Java 应用程序中使用它们的小样本。请参考第五章中的*高级操作*了解这样的配方。

当前版本的 MongoDB 驱动程序的 Javadocs 可以在[`api.mongodb.org/java/current/`](https://api.mongodb.org/java/current/)找到。

# 使用 Java 客户端在 Mongo 中实现聚合

这个配方的目的不是解释聚合，而是向您展示如何使用 Java 客户端从 Java 程序实现聚合。在这个配方中，我们将根据州名对数据进行聚合，并获取出现在其中的文档数量最多的五个州名。我们将使用`$project`、`$group`、`$sort`和`$limit`操作符进行处理。

## 准备工作

用于此配方的测试类是`com.packtpub.mongo.cookbook.MongoAggregationTest`。要执行聚合操作，我们需要一个正在运行的服务器。一个简单的单节点就是我们需要的。请参考第一章中的*安装单节点 MongoDB*配方，了解如何启动服务器的说明。我们将操作的数据需要导入到数据库中。有关如何导入数据的步骤，请参阅第二章中的*创建测试数据*配方，*命令行操作和索引*。下一步是从 Packt 网站下载 Java 项目`mongo-cookbook-javadriver`。虽然可以使用 Maven 执行测试用例，但将项目导入 IDE 并执行测试用例类更加方便。假设您熟悉 Java 编程语言，并且习惯使用要导入的项目的 IDE。

## 操作步骤

要执行测试用例，可以将项目导入类似 Eclipse 的 IDE 中并执行测试用例，或者使用 Maven 从命令提示符中执行测试用例。

1.  如果您使用的是 IDE，请打开测试类并将其执行为 JUnit 测试用例。

1.  如果您计划使用 Maven 执行此测试用例，请转到命令提示符，更改项目根目录下的目录，并执行以下命令以执行此单个测试用例：

```go
$ mvn -Dtest=com.packtpub.mongo.cookbook.MongoAggregationTesttest

```

如果 Java SDK 和 Maven 设置正确，并且 MongoDB 服务器正在运行并监听端口`27017`以进行传入连接，那么一切都应该正常执行。

## 工作原理

我们测试类中用于聚合功能的方法是`aggregationTest()`。聚合操作是在 Java 客户端上使用`DBCollection`类中定义的`aggregate()`方法对 MongoDB 执行的。该方法具有以下签名：

```go
AggregationOutput aggregate(firstOp, additionalOps)
```

只有第一个参数是必需的，它形成管道中的第一个操作。第二个参数是`varagrs`参数（零个或多个值的可变数量的参数），允许更多的管道操作符。所有这些参数都是`com.mongodb.DBObject`类型。如果在执行聚合命令时发生任何异常，聚合操作将抛出`com.mongodb.MongoException`，并带有异常的原因。

返回类型`com.mongodb.AggregationOutput`用于获取聚合操作的结果。从开发人员的角度来看，我们更感兴趣的是这个实例的`results`字段，可以使用返回对象的`results()`方法来访问。`results()`方法返回一个`Iterable<DBObject>`类型的对象，可以迭代以获取聚合的结果。

让我们看看我们如何在测试类中实现了聚合管道：

```go
AggregationOutput output = collection.aggregate(
    //{'$project':{'state':1, '_id':0}},
    new BasicDBObject("$project", new BasicDBObject("state", 1).append("_id", 0)),
    //{'$group':{'_id':'$state', 'count':{'$sum':1}}}
    new BasicDBObject("$group", new BasicDBObject("_id", "$state")
      .append("count", new BasicDBObject("$sum", 1))),
    //{'$sort':{'count':-1}}
    new BasicDBObject("$sort", new BasicDBObject("count", -1)),
    //{'$limit':5}
    new BasicDBObject("$limit", 5)
);
```

在以下顺序中，管道中有四个步骤：`$project`操作，然后是`$group`，`$sort`，然后是`$limit`。

最后两个操作看起来效率低下，我们首先对所有元素进行排序，然后只取前五个元素。在这种情况下，MongoDB 服务器足够智能，可以在排序时考虑限制操作，只需维护前五个结果而不是对所有结果进行排序。

对于 MongoDB 2.6 版本，聚合结果可以返回一个游标。尽管前面的代码仍然有效，但`AggregationResult`对象不再是获取操作结果的唯一方式。我们可以使用`com.mongodb.Cursor`来迭代结果。此外，前面的格式现在已被弃用，而是采用了接受管道操作符列表而不是操作符`varargs`的格式。请参考`com.mongodb.DBCollection`类的 Javadocs，并查看各种重载的`aggregate()`方法。

# 在 Java 客户端中执行 Mongo 中的 MapReduce

在我们之前的食谱中，*使用 Java 客户端在 Mongo 中实现聚合*，我们看到了如何使用 Java 客户端在 Mongo 中执行聚合操作。在这个食谱中，我们将处理与聚合操作相同的用例，但我们将使用 MapReduce。目的是根据州名对数据进行聚合，并获取出现在其中的文档数量最多的五个州名。

如果有人不知道如何从编程语言客户端为 Mongo 编写 MapReduce 代码，并且是第一次看到它，你可能会惊讶地看到它实际上是如何完成的。你可能想象过，你会在编写代码的编程语言中（在这种情况下是 Java）编写`map`和`reduce`函数，然后使用它来执行 map reduce。然而，我们需要记住，MapReduce 作业在 mongo 服务器上运行，并执行 JavaScript 函数。因此，无论编程语言驱动程序如何，map reduce 函数都是用 JavaScript 编写的。编程语言驱动程序只是让我们调用和执行服务器上用 JavaScript 编写的 map reduce 函数的手段。

## 准备工作

用于此示例的测试类是`com.packtpub.mongo.cookbook.MongoMapReduceTest`。要执行 map reduce 操作，我们需要运行一个服务器。我们只需要一个简单的单节点。有关如何启动服务器的说明，请参阅第一章中的*安装单节点 MongoDB*配方，*安装和启动服务器*。我们将操作的数据需要导入数据库。有关导入数据的步骤，请参阅第二章中的*创建测试数据*配方，*命令行操作和索引*。下一步是从 Packt 网站下载 Java 项目`mongo-cookbook-javadriver`。虽然可以使用 Maven 执行测试用例，但将项目导入 IDE 并执行测试用例类更加方便。假设您熟悉 Java 编程语言，并且习惯使用将要导入项目的 IDE。

## 如何做…

要执行测试用例，可以在类似 Eclipse 的 IDE 中导入项目并执行测试用例，也可以使用 Maven 从命令提示符中执行测试用例。

1.  如果使用 IDE，打开测试类并将其作为 JUnit 测试用例执行。

1.  如果您计划使用 Maven 执行此测试用例，请转到命令提示符，将目录更改为项目的根目录，并执行以下命令以执行此单个测试用例：

```go
$ mvn -Dtest=com.packtpub.mongo.cookbook.MongoMapReduceTesttest

```

如果 Java SDK 和 Maven 设置正确，并且 MongoDB 服务器正常运行并监听端口`27017`以接收连接，则一切都应该能够正常执行。

## 工作原理…

我们 map reduce 测试的测试用例方法是`mapReduceTest()`。

可以使用`DBCollection`类中定义的`mapReduce()`方法在 Java 客户端中对 Mongo 进行 map reduce 操作。有很多重载版本，您可以参考`com.mongodb.DBCollection`类的 Javadocs，了解此方法的各种用法。我们使用的是`collection.mapReduce(mapper, reducer, output collection, query)`。

该方法接受以下四个参数：

+   `mapper`函数的类型为 String，是在 mongo 数据库服务器上执行的 JavaScript 代码

+   `reducer`函数的类型为 String，是在 mongo 数据库服务器上执行的 JavaScript 代码

+   map reduce 执行的输出将写入的集合的名称

+   服务器将执行的查询，此查询的结果将作为 map reduce 作业执行的输入

假设读者对 shell 中的 map reduce 操作很熟悉，我们不会解释在测试用例方法中使用的 map reduce JavaScript 函数。它所做的就是将键作为州的名称和值进行发射，这些值是特定州名出现的次数。此结果将添加到输出集合`javaMROutput`中。例如，在整个集合中，州`Maharashtra`出现了`6446`次；因此，`Maharashtra`州的文档是`{'_id': 'Maharashtra', 'value': 6446}`。要确认这是否是真实值，可以在 mongo shell 中执行以下查询，并查看结果是否确实为`6446`：

```go
> db.postalCodes.count({state:'Maharashtra'})
6446

```

我们还没有完成，因为要找到集合中出现次数最多的五个州；我们只有州和它们的出现次数，所以最后一步是按照`value`字段对文档进行排序，这个字段是州名出现的次数，按降序排列，并将结果限制为五个文档。

## 另请参阅

参考第八章，“与 Hadoop 集成”中有关在 MongoDB 中使用 Hadoop 连接器执行 Map Reduce 作业的不同方法。这使我们能够使用 Java、Python 等语言编写`Map`和`Reduce`函数。
