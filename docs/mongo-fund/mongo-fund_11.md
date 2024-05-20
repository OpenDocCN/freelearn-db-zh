# 第十一章：MongoDB 中的备份和恢复

概述

在本章中，我们将详细研究如何将备份、样本和测试数据库加载到目标 MongoDB 实例中，同样重要的是，你将学会如何导出现有数据集以备份和恢复。到本章结束时，你将能够将 MongoDB 数据备份、导出、导入和恢复到现有服务器中。这使你能够从灾难中恢复数据，以及快速将已知信息加载到系统进行测试。

# 介绍

在之前的章节中，我们主要依赖预加载到 MongoDB Atlas 实例中的样本数据。除非你在进行新项目，否则这通常是数据库首次出现的方式。然而，当你被雇佣或转移到一个包含 MongoDB 数据库的不同项目时，它将包含在你开始之前创建的所有数据。

现在，如果你需要一个本地副本来测试你的应用程序或查询呢？直接对生产数据库运行查询通常是不安全或不可行的，因此将数据集复制到测试环境是非常常见的过程。同样，当创建一个新项目时，你可能希望将一些样本数据或测试数据加载到数据库中。在本章中，我们将研究迁移、导入或导出现有 MongoDB 服务器的程序，并设置一个包含现有数据的新数据库的程序。

注意

在本章中，包括的练习和活动都是对一个场景的迭代。数据和示例都基于名为`sample_mflix`的 MongoDB Atlas 示例数据库。

在本章期间，我们将按照一个理论场景进行一系列练习。这是对*第七章*“数据聚合”和*第八章*“在 MongoDB 中编写 JavaScript”中涵盖的场景的扩展。你可能还记得，一个电影院连锁要求你创建查询和程序，分析他们的数据库，以制作在促销季期间放映的电影列表。

在这些章节中，你已经构建了一些聚合，其输出是包含摘要数据的新集合。你还创建了一个应用程序，使用户可以以编程方式更新电影。公司对你的工作非常满意，他们决定将整个系统迁移到更重要、更好的硬件上。尽管系统管理员们对将现有的 MongoDB 实例迁移到新硬件上感到自信，但你决定最好手动测试该过程，以确保在需要时能够提供帮助。

# MongoDB 实用程序

mongo shell 不包括导出、导入、备份或恢复的功能。然而，MongoDB 已经创建了方法来实现这一点，因此不需要脚本工作或复杂的图形用户界面。为此，提供了几个实用程序脚本，可以用于批量将数据进出数据库。这些实用程序脚本包括：

+   `mongoimport`

+   `mongoexport`

+   `mongodump`

+   `mongorestore`

我们将在接下来的章节中详细介绍这些实用程序。正如它们的名称所示，这四个实用程序对应于导入文档、导出文档、备份数据库和恢复数据库。我们将从导出数据的主题开始。

# 导出 MongoDB 数据

在批量移动数据进出 MongoDB 时，最常见且通用的实用程序是`mongoexport`。这个命令很有用，因为它是从 MongoDB 中提取大量数据的主要方式之一。将 MongoDB 数据导出到 JSON 文件中，可以让你将其与其他应用程序或数据库一起使用，并与 MongoDB 之外的利益相关者共享数据。

重要的是要注意，`mongoexport` 必须在指定的单个数据库和集合上运行。 不能在整个数据库或多个集合上运行 `mongoexport`。 我们将在本章后面看到如何完成类似这样的更大范围的备份。 以下片段是 `mongoexport` 的示例：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@provendocs-fawxo.gcp.mongodb.net/sample_mflix –quiet --limit=10 --sort="{theaterId:1}" --collection=theaters --out=output.json
```

这个例子是一个更复杂的命令，其中包括一些可选参数并明确设置其他参数。 但是在实践中，您的导出命令可能会简单得多。 这里使用的结构和参数在下一节中有详细解释。

## 使用 mongoexport

学习 `mongoexport` 语法的最佳方法是逐个参数地构建命令。 所以让我们从最简单的导出开始：

```js
mongoexport –-collection=theaters
```

正如您所看到的，命令的最简单形式只需要一个参数：`–-collection`。 此参数是我们希望导出文档的集合。

如果执行此命令，可能会遇到一些令人困惑的结果，如下所示：

```js
2020-03-07-T13:16:09.152+1100 error connecting to db server: no reachable servers
```

我们得到这个结果是因为我们没有指定数据库或 URI。 在这种情况下，`mongoexport` 默认使用本地 MongoDB 的端口 27017 和默认数据库。 由于在上一章的示例和练习中我们一直在 Atlas 上运行我们的 MongoDB 服务器，让我们更新我们的命令以指定这些参数。

注意

您不能同时指定数据库和 URI；这是因为数据库是 URI 的一部分。 在本章中，我们将使用 URI 进行导出。

更新后的命令如下所示：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer.gcp.mongodb.net/sample_mflix --collection=theaters
```

现在您有一个有效的命令，可以针对 MongoDB Atlas 数据库运行它。 您将看到以下输出：

```js
2020-08-17T11:07:23.302+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/sample_mflix
{"_id":{"$oid":"59a47286cfa9a3a73e51e72c"},"theaterId":1000,"location":  {"address":{"street1":"340 W Market","city":"Bloomington","state":"MN","zipcode":"55425"},"geo":  {"type":"Point","coordinates":[-93.24565,44.85466]}}}
{"_id":{"$oid":"59a47286cfa9a3a73e51e72d"},"theaterId":1003,"location":  {"address":{"street1":"45235 Worth Ave.","city":"California","state":"MD","zipcode":"20619"},"geo":  {"type":"Point","coordinates":[-76.512016,38.29697]}}}
{"_id":{"$oid":"59a47286cfa9a3a73e51e72e"},"theaterId":1008,"location":  {"address":{"street1":"1621 E Monte Vista Ave","city":"Vacaville","state":"CA","zipcode":"95688"},"geo":  {"type":"Point","coordinates":[-121.96328,38.367649]}}}
{"_id":{"$oid":"59a47286cfa9a3a73e51e72f"},"theaterId":1004,"location":  {"address":{"street1":"5072 Pinnacle Sq","city":"Birmingham","state":"AL","zipcode":"35235"},"geo":  {"type":"Point","coordinates":[-86.642662,33.605438]}}}
```

在输出的末尾，您应该看到导出的记录数：

```js
{"_id":{"$oid":"59a47287cfa9a3a73e51ed46"},"theaterId":952,"location":  {"address":{"street1":"4620 Garth Rd","city":"Baytown","state":"TX","zipcode":"77521"},"geo":  {"type":"Point","coordinates":[-94.97554,29.774206]}}}
{"_id":{"$oid":"59a47287cfa9a3a73e51ed47"},"theaterId":953,"location":  {"address":{"street1":"10 McKenna Rd","city":"Arden","state":"NC","zipcode":"28704"},"geo":  {"type":"Point","coordinates":[-82.536293,35.442486]}}}
2020-08-17T11:07:24.992+1000    [########################]  sample_mflix.theaters  1564/1564  (100.0%)
2020-08-17T11:07:24.992+1000    exported 1564 records
```

使用指定的 URI，导出操作成功，并且您可以看到从 `theatres` 集合中的所有文档。 但是，将所有这些文档淹没在输出中并不是很有用。 您可以使用一些 shell 命令将此输出管道或附加到文件中，但是 `mongoexport` 命令在其语法中提供了另一个参数，用于自动输出到文件。 您可以在以下命令中看到此参数 (`--out`)：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer.gcp.mongodb.net/sample_mflix --collection=theaters --out=output.json
```

运行此命令后，您将看到以下输出：

```js
2020-08-17T11:11:44.499+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/sample_mflix
2020-08-17T11:11:45.634+1000    [........................]  sample_mflix.theaters  0/1564  (0.0%)
2020-08-17T11:11:45.694+1000    [########################]  sample_mflix.theaters  1564/1564  (100.0%)
2020-08-17T11:11:45.694+1000    exported 1564 records
```

现在，在该目录中创建了一个名为 `output.json` 的新文件。 如果您查看此文件，您可以看到我们从 theatres 集合中导出的文档。

参数 `uri`、`collection` 和 `out` 可以满足大多数导出用例。 一旦您的数据在磁盘上的文件中，就很容易将其与其他应用程序或脚本集成。

## mongoexport 选项

现在我们知道了 `mongoexport` 的三个最重要的选项。 但是，还有一些其他有用的选项可帮助从 MongoDB 导出数据。 以下是其中一些选项及其效果：

+   `--quiet`：此选项减少了在导出期间发送到命令行的输出量。

+   `--type`：这将影响文档在控制台中的打印方式，默认为 JSON。 例如，您可以通过指定 CSV 来以 **逗号分隔值** (**CSV**) 格式导出数据。

+   `--pretty`：这以良好格式的方式输出文档。

+   `--fields`：这指定要导出的文档中的键的逗号分隔列表，类似于导出级别的投影。

+   `--skip`：这类似于查询级别的跳过，跳过导出的文档。

+   `--sort`：这类似于查询级别的排序，按某些键对文档进行排序。

+   `--limit`：这类似于查询级别的限制，限制输出的文档数量。

以下是一个示例，其中使用了一些这些选项，本例中将排序的十个 `theatre` 文档输出到名为 `output.json` 的文件中。 此外，还使用了 `--quiet` 参数：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@provendocs-fawxo.gcp.mongodb.net/sample_mflix --quiet --limit=10 --sort="{theaterId:1}" --collection=theaters --out=output.json
```

由于我们使用了 `--quiet` 选项，因此将不会看到任何输出。

```js
> mongoexport --uri=mongodb+srv://testUser:testPassword@performancet uning.98afc.gcp.mongodb.net/sample_mflix --quiet --limit=10 --sort="{theaterId:1}" --collection=theaters --out=output.json
>
```

但是，如果我们查看`output.json`文件的内容，我们可以看到按 ID 排序的十个文档：

![图 11.1：output.json 文件的内容（已截断）](img/B15507_11_01.jpg)

图 11.1：output.json 文件的内容（已截断）

还有另一个选项可用于更高级的导出，那就是查询选项。查询选项允许您指定一个查询，使用与标准 MongoDB 查询相同的格式。只有匹配此查询的文档将被导出。将此选项与`--fields`、`--skip`和`--limit`等其他选项结合使用，可以定义一个完整的查询，并将其格式化输出，然后将其导出到文件中。

以下是使用查询选项返回特定文档子集的导出。在这种情况下，我们正在获取所有`theaterId`为`4`的电影院。

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@provendocs-fawxo.gcp.mongodb.net/sample_mflix --query="{theaterId: 4}" --collection=theaters
```

注意

在 MacOS 上，您可能需要用引号括起`theaterId`，例如：`--query="{\"theaterId\": 4}"`

现在我们将看到我们正在寻找的文档如下：

```js
2020-08-17T11:22:48.559+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/sample_mflix
{"_id":{"$oid":"59a47287cfa9a3a73e51eb78"},"theaterId":4,"location":  {"address":{"street1":"13513 Ridgedale Dr","city":"Hopkins","state":"MN","zipcode":"55305"},"geo":  {"type":"Point","coordinates":[-93.449539,44.969658]}}}
2020-08-17T11:22:48.893+1000    exported 1 record
```

让我们在下一个练习中使用这些选项。

## 练习 11.01：导出 MongoDB 数据

在开始本练习之前，让我们重新审视一下*介绍*部分中概述的电影公司的情景。假设您的客户（电影公司）将迁移其现有数据，您担心会丢失宝贵的信息。您决定的第一件事是将数据库中的文档导出为 JSON 文件，以防灾难发生，可以将其存储在廉价的云存储中。此外，您将为每个电影类别创建不同的导出。

注意

为了展示对`mongoexport`的了解，我们将不为每个类别创建一个导出，而只为单个类别创建一个。您还只会导出前三个文档。

在这个练习中，您将使用`mongoexport`创建一个名为`action_movies.json`的文件，其中包含按发行年份排序的三部动作电影。以下步骤将帮助您完成任务：

1.  调整您的导出并保存以备后用。创建一个名为`Exercise11.01.txt`的新文件，以存储您的导出命令。

1.  接下来，只需输入标准的`mongoexport`语法，包括 URI 和`movies`集合：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --collection=movies
```

1.  添加额外的参数以满足您的条件。首先，将您的导出输出到名为`action_movies.json`的文件中。使用`--out`参数如下：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --collection=movies --out=action_movies.json
```

1.  接下来，根据本练习的规范，添加您的排序条件以按发行年份对电影进行排序。您可以使用`--sort`来实现：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --collection=movies --out=action_movies.json --sort='{released: 1}'
```

1.  如果您在当前的中间阶段运行此命令，您将遇到以下错误：

```js
2020-08-17T11:25:51.911+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/sample_mflix
2020-08-17T11:25:52.581+1000    Failed: (OperationFailed) Executor error during find command :: caused by :: Sort operation used more than the maximum 33554432 bytes of RAM. Add an index, or specify a smaller limit.
```

这是因为 MongoDB 服务器正在尝试为我们排序大量文档。为了提高导出和导入的性能，您可以限制检索的文档数量，这样 MongoDB 就不必为您排序那么多文档。

1.  添加`--limit`参数以减少被排序的文档数量，并满足三个文档的条件：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --collection=movies --out=action_movies.json --sort='{released: 1}' --limit=3
```

最后，您需要添加查询参数以过滤掉不属于电影类型的任何文档。

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --collection=movies --out=action_movies.json --sort='{released : 1}' --limit=3 --query="{'genres': 'Action'}"
```

注意

在 MacOS 和 Linux 上，您可能需要更改参数中字符串周围的引号，例如在前面的查询中，您需要使用：`--query='{"genres": "Action"}'`

1.  完成命令后，从`Exercise11.01.txt`文件中将其复制到终端或命令提示符中运行：

```js
2020-08-18T12:35:42.514+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/sample_mflix
2020-08-18T12:35:42.906+1000    exported 3 records
```

到目前为止，输出看起来不错，但您需要检查输出文件以确保已导出正确的文档。在您刚刚执行命令的目录中，您应该看到新文件`action_movies.json`。打开此文件并查看其中的内容。

注意

为了提高输出的清晰度，已删除了剧情字段。

您应该看到以下文档：

![图 11.2：action_movies.json 文件的内容（为简洁起见已截断）](img/B15507_11_02.jpg)

图 11.2：action_movies.json 文件的内容（为简洁起见已截断）

这个练习说明了以强大和灵活的方式从 MongoDB 导出文档所需的基本知识。结合这里学到的参数，大多数基本导出现在都很容易。要掌握 MongoDB 中的数据导出，保持实验和学习是有帮助的。

# 将数据导入到 MongoDB

现在你知道如何将你的集合数据从 MongoDB 中导出并以易于使用的格式保存到磁盘上。但是假设你在磁盘上有这个文件，并且你想与拥有自己的 MongoDB 数据库的人分享？这种情况下，`mongoimport`就派上用场了。正如你可能从名称中猜到的那样，这个命令本质上是`mongoexport`的反向，并且它被设计为将`mongoexport`的输出作为`mongoimport`的输入。

然而，不仅可以使用从 MongoDB 导出的数据来使用`mongoimport`。该命令支持 JSON、CSV 和 TSV 格式，这意味着从其他应用程序提取的数据或手动创建的数据仍然可以轻松地添加到数据库中。通过支持这些广泛使用的文件格式，该命令成为将大量数据加载到 MongoDB 中的通用方式。

与`mongoexport`一样，`mongoimport`在指定的数据库中操作单个目标集合。这意味着如果你希望将数据导入多个集合，你必须将数据分开成单独的文件。

以下是一个复杂`mongoimport`的例子。我们将在下一节详细介绍语法。

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=oldData --file=old.csv --type=CSV --headerline --ignoreBlanks --drop
```

## 使用 mongoimport

以下是一个具有最少参数的`mongoimport`命令。这比前面的命令简单得多。

```js
mongoimport --db=imports --collection=contacts --file=contacts.json
```

这个例子看起来也与我们在前一节中看到的一些片段非常相似。它几乎与我们的`mongoexport`语法相同，只是不是提供一个使用`--out`创建新文件的位置，而是输入一个`--file`参数，指定我们希望加载的数据。我们的数据库和集合参数与`mongoexport`示例中提供的语法相同。

正如你可能猜到的，`mongoimport`与`mongoexport`共享的另一个相似之处是，默认情况下，它将针对本地计算机上运行的 MongoDB 数据库运行。我们使用相同的`--uri`参数来指定我们正在将数据加载到远程 MongoDB 服务器中——在这种情况下是 MongoDB Atlas。

注意

与`mongoexport`一样，`db`和`uri`参数是互斥的，因为数据库在`uri`中已经定义了。

当使用`--uri`参数时，`mongoimport`命令将如下所示：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/imports --collection=contacts --file=contacts.json
```

在你可以执行这个命令来导入你的 MongoDB 数据库之前，你需要一个包含有效数据的文件。现在让我们创建一个。创建可导入数据的最简单方法之一是运行`mongoexport`。然而，为了提高你导入文件的知识，我们将从头开始创建一个。

你可以开始创建一个名为`contacts.json`的文件。在文本编辑器中打开文件并创建一些非常简单的文档。在导入 JSON 文件时，文件中的每一行必须包含一个文档。

`contacts.json`文件应该如下所示：

```js
//contacts.json
{"name": "Aragorn","location": "New Zealand","job": "Park Ranger"}
{"name": "Frodo","location": "New Zealand","job": "Unemployed"}
{"name": "Ned Kelly","location": "Australia","job": "Outlaw"}
```

执行以下导入：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/imports --collection=contacts --file=contacts.json
```

这将导致以下输出：

```js
2020-08-17T20:10:38.892+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.g
cp.mongodb.net/imports
2020-08-17T20:10:39.150+1000    3 document(s) imported successfully. 0 document(s) failed to import. 
```

你还可以使用 JSON 数组格式的文件，这意味着你的导入文件包含许多不同的 JSON 文档的数组。在这种情况下，你必须在命令中指定`--jsonArray`选项。这个 JSON 数组结构现在应该对你来说非常熟悉，因为它与`mongoexport`的输出以及你从 MongoDB 查询中收到的结果匹配。例如，如果你的文件包含如下数组：

```js
[
    {
        "name": "Aragorn",
        "location": "New Zealand",
        "job": "Park Ranger"
    },
    {
        "name": "Frodo",
        "location": "New Zealand",
        "job": "Unemployed"
    },
    {
        "name": "Ned Kelly",
        "location": "Australia",
        "job": "Outlaw"
    }
]
```

你仍然可以使用`mongoimport`命令导入文件，并使用`--jsonArray`选项，如下所示：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/imports --collection=contacts --file=contacts.json --jsonArray
```

这将导致以下输出：

```js
2020-08-17T20:10:38.892+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.g
cp.mongodb.net/imports
2020-08-17T20:10:39.150+1000    3 document(s) imported successfully. 0 document(s) failed to import. 
```

注意

在上面的示例中，您会注意到可以为导入的文档提供`_id`值。如果没有提供`_id`，则将为文档生成一个。您必须确保您提供的`_id`尚未被使用；否则，`mongoimport`命令将抛出错误。

这两个导入向我们展示了将数据导入 MongoDB 数据库的简单方法，但让我们看看当事情出错时会发生什么。让我们修改文件以为我们的一些文档指定`_id`。

```js
[
    {
        "_id": 1,
        "name": "Aragorn",
        "location": "New Zealand",
        "job": "Park Ranger"
    },
    {
        "name": "Frodo",
        "location": "New Zealand",
        "job": "Unemployed"
    },
    {
        "_id": 2,
        "name": "Ned Kelly",
        "location": "Australia",
        "job": "Outlaw"
    }
]
```

执行一次，您应该可以得到无错误的输出。

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/imports --collection=contacts --file=contacts.json --jsonArray
```

您将看到以下输出：

```js
2020-08-17T20:12:12.164+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.g
cp.mongodb.net/imports
2020-08-17T20:12:12.404+1000    3 document(s) imported successfully. 0 document(s) failed to import.
```

现在，如果您重新运行相同的命令，您会看到错误，因为该`_id`值已经存在于您的集合中。

```js
2020-08-17T20:12:29.742+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.g
cp.mongodb.net/imports
2020-08-17T20:12:29.979+1000    continuing through error: E11000 duplicate key error collection: imp
orts.contacts index: _id_ dup key: { _id: 1 }
2020-08-17T20:12:29.979+1000    continuing through error: E11000 duplicate key error collection: imp
orts.contacts index: _id_ dup key: { _id: 2 }
2020-08-17T20:12:29.979+1000    1 document(s) imported successfully. 2 document(s) failed to import.
```

您可以在输出中看到错误。您可能还注意到，没有问题的文档仍然成功导入。如果您导入了一个包含一万个文档的文件，`mongoimport`不会因为单个文档而失败。

假设您确实想要更新此文档而不更改其`_id`。您无法使用此`mongoimport`命令，因为每次都会收到重复键错误。

您可以使用 mongo shell 登录 MongoDB 并在导入之前手动删除此文档，但这将是一个缓慢的方法。使用`mongoimport`，我们可以使用`--drop`选项在导入之前删除集合。这是确保文件中存在的内容也存在于集合中的好方法。

例如，假设在我们导入之前，我们的集合中有以下文档：

```js
MongoDB Enterprise PerformanceTuning-shard-0:PRIMARY> db.contacts.find({})
{ "_id" : ObjectId("5e0c1db3fa8335898940129ca8"), "name": "John Smith"}
{ "_id" : ObjectId("5e0c1db3fa8335898940129ca8"), "name": "Jane Doe"}
{ "_id" : ObjectId("5e0c1db3fa8335898940129ca8"), "name": "May Sue"}
```

现在，使用`--drop`运行以下`mongoimport`命令：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/imports --collection=contacts –-file=contacts.json --jsonArray --drop
2020-08-17T20:16:08.280+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.g
cp.mongodb.net/imports
2020-08-17T20:16:08.394+1000    dropping: imports.contacts
2020-08-17T20:16:08.670+1000    3 document(s) imported successfully. 0 document(s) failed to import.  
```

执行命令后，您将看到集合中有以下文档，可以使用 find 命令查看这些文档。

```js
db.contacts.find({})
```

您应该看到以下输出：

```js
{ "_id" : ObjectId("5f3a58e8fd0803fc3dec8cbf"), "name" : "Frodo", "location" : "New Zealand", "job" : "Unemployed" }
{ "_id" : 1, "name" : "Aragorn", "location" : "New Zealand", "job" : "Park Ranger" }
{ "_id" : 2, "name" : "Ned Kelly", "location" : "Australia", "job" : "Outlaw" }
```

在下一节中，我们将看看可以与`mongoimport`一起使用的选项。

## mongoimport 选项

现在我们知道了使用`mongoimport`的基本选项，包括`--uri`、`--collection`和`--file`参数。但是，就像我们上一节中的`mongoexport`一样，运行命令时可能还有几个其他选项。其中许多选项与`mongoexport`相同。以下列表描述了一些选项及其效果。

+   `--quiet`：这会减少导入的输出消息量。

+   `--drop`：在开始导入之前删除集合。

+   --jsonArray：仅适用于 JSON 类型，指定文件是否为 JSON 数组格式。

+   `--type`：可以是 JSON、CSV 或 TSV，用于指定要导入的文件类型，但默认类型为 JSON。

+   `--ignoreBlanks`：仅适用于 TSV 和 CSV，这将忽略导入文件中的空字段。

+   `--headerline`：仅适用于 TSV 和 CSV，这将假定导入文件的第一行是字段名称列表。

+   `--fields`：仅适用于 TSV 和 CSV，这将为 CSV 和 TSV 格式的文档指定逗号分隔的键列表。只有在没有标题行时才需要。

+   `--stopOnError`：如果指定，导入将在遇到第一个错误时停止。

以下是使用更多这些选项的示例，特别是带有标题行的 CSV 导入。我们还必须忽略空白，以便文档不会获得空白的`_id`值。

这是我们的`.csv`文件，名为`contacts.csv`：

```js
_id,name,location,job
1,Aragorn,New Zealand,Park Ranger
,Frodo,New Zealand,Unemployed
2,Ned Kelly,Australia,Outlaw
```

我们将使用以下命令导入 CSV：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/imports --collection=contacts --file=contacts.csv --drop --type=CSV --headerline --ignoreBlanks
2020-08-17T20:22:39.750+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/imports
2020-08-17T20:22:39.863+1000    dropping: imports.contacts
2020-08-17T20:22:40.132+1000    3 document(s) imported successfully. 0 document(s) failed to import.
```

上述命令导致我们的集合中有以下文档：

```js
MongoDB Enterprise atlas-nb3biv-shard-0:PRIMARY> db.contacts.find({})
{ "_id" : 2, "name" : "Ned Kelly", "location" : "Australia", "job" : "Outlaw" }
{ "_id" : 1, "name" : "Aragorn", "location" : "New Zealand", "job" : "Park Ranger" }
{ "_id" : ObjectId("5f3a5a6fc67ba81a6d4bcf69"), "name" : "Frodo", "location" : "New Zealand", "job" : "Unemployed" }
```

当然，这些只是您可能遇到的一些常见选项。文档中提供了完整的列表。熟悉这些选项是有用的，以防您需要对不同配置的 MongoDB 服务器运行更高级的导入。

## 练习 11.02：将数据加载到 MongoDB 中

在这种情况下，您已经成功地在本地机器上创建了客户数据的导出。您已经在不同版本的新服务器上设置了一个新服务器，并希望确保数据正确地导入到新配置中。此外，您还从另一个较旧的数据库中以 CSV 格式获得了一些数据文件，这些数据文件将迁移到新的 MongoDB 服务器。您希望确保这种不同的格式也能正确导入。考虑到这一点，您的目标是将两个文件（如下所示）导入到 Atlas 数据库中，并测试文档是否存在于正确的集合中。

在这个练习中，您将使用`mongoimport`将两个文件（`old.csv`和`new.json`）导入到两个单独的集合（`oldData`和`newData`），并使用 drop 来确保没有剩余的文档存在。

可以通过执行以下步骤来实现这个目标：

1.  微调您的导入并保存以备后用。创建一个名为`Exercise11.02.txt`的新文件来存储您的导出命令。

1.  创建包含要导入的数据的`old.csv`和`new.json`文件。可以从 GitHub 上下载文件[`packt.live/2LsgKS3`](https://packt.live/2LsgKS3)，或者将以下内容复制到当前目录中的相同文件中。

`old.csv`文件应如下所示：

```js
_id,title,year,genre
54234,The King of The Bracelets,1999,Fantasy
6521,Knife Runner,1977,Science Fiction
124124,Kingzilla,1543,Horror
64532,Casabianca,1942,Drama
23214,Skyhog Day,1882,Comedy
```

`new.json`文件应如下所示：

```js
[
    {"_id": 54234,"title": "The King of The Bracelets","year": 1999,"genre": "Fantasy"},
    {"_id": 6521, "title": "Knife Runner","year": 1977,"genre": "Science Fiction"},
    {"_id": 124124,"title": "Kingzilla","year": 1543,"genre": "Horror"},
    {"_id": 64532,"title": "Casabianca","year": 1942,"genre": "Drama"},
    {"_id": 23214,"title": "Skyhog Day","year": 1882,"genre": "Comedy"}
]
```

1.  将标准的`mongoimport`语法输入到您的`Exercise11.02.txt`文件中，只包括 URI、集合和文件位置。首先将您的数据导入到`"imports"`数据库中，先导入旧数据：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports --collection=oldData --file=old.csv
```

1.  现在，开始添加额外的参数以满足 CSV 文件的条件。指定`type=CSV`：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/ imports  --collection=oldData --file=old.csv --type=CSV
```

1.  接下来，因为旧数据中有标题行，所以使用`headerline`参数。

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=oldData --file=old.csv --type=CSV --headerline
```

1.  在本章的一些示例中，当您看到 CSV 导入时，使用了`--ignoreBlanks`参数来确保空字段不被导入。这是一个很好的做法，所以在这里也要添加它。

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=oldData --file=old.csv --type=CSV --headerline --ignoreBlanks
```

1.  最后，在这个练习中，您需要确保不要在现有数据之上进行导入，因为这可能会导致冲突。为了确保您的数据被干净地导入，请使用`--drop`参数如下：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=oldData --file=old.csv --type=CSV --headerline --ignoreBlanks --drop
```

1.  这应该是你进行 CSV 导入所需要的一切。通过复制现有命令到新行，然后删除 CSV 特定参数来开始编写 JSON 导入。

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=oldData --file=old.csv --drop
```

1.  现在，通过以下命令更改`file`和`collection`参数，将您的`new.json`文件导入到`newData`集合中：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --drop --collection=newData --file=new.json 
```

1.  您可以看到`new.json`文件中的数据是以 JSON 数组格式，因此添加匹配参数如下：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports --collection=newData --file=new.json --drop --jsonArray
```

1.  现在，您的`Exercise11.02.txt`文件中应该有以下两个命令。

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=newData --file=new.json --drop --jsonArray
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=oldData --file=old.csv --type=CSV --headerline --ignoreBlanks --drop
```

1.  使用以下命令运行您的`newData`导入：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=newData --file=new.json --drop --jsonArray
```

输出如下：

```js
2020-08-17T20:25:21.622+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/imports
2020-08-17T20:25:21.734+1000    dropping: imports.newData
2020-08-17T20:25:22.019+1000    5 document(s) imported successfully. 0 document(s) failed to import.
```

1.  现在，按照以下方式执行`oldData`导入：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports  --collection=oldData --file=old.csv --type=CSV --headerline --ignoreBlanks --drop
```

输出如下：

```js
2020-08-17T20:26:09.588+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/imports
2020-08-17T20:26:09.699+1000    dropping: imports.oldData
2020-08-17T20:26:09.958+1000    5 document(s) imported successfully. 0 document(s) failed to import. 
```

1.  通过运行以下命令来检查 MongoDB 中的两个新集合：

```js
show collections
```

输出如下：

![图 11.3：显示新集合](img/B15507_11_03.jpg)

图 11.3：显示新集合

首先，我们学会了如何从 MongoDB 服务器中导出数据。现在我们可以使用导入命令将外部数据输入到 MongoDB 中。通过结合这两个简单的命令，我们还可以在 MongoDB 的不同实例之间转移数据，或者在导入到 MongoDB 之前使用外部工具创建数据。

# 备份整个数据库

使用`mongoexport`，理论上我们可以获取整个 MongoDB 服务器，并提取每个数据库和集合中的所有数据。然而，我们必须一次处理一个集合，确保文件正确映射到原始数据库和集合。手动完成这个过程是可能的，但很困难。脚本可以可靠地完成整个 MongoDB 服务器的这项工作，即使有数百个集合。

幸运的是，除了`mongoimport`和`mongoexport`之外，MongoDB 工具包还提供了一个工具，用于导出整个数据库的内容。这个实用程序称为`mongodump`。这个命令创建了整个 MongoDB 实例的备份。您只需要提供 URI（或主机和端口号），`mongodump`命令就会完成剩下的工作。这个导出创建了一个二进制文件，可以使用`mongorestore`来恢复（这是下一节中介绍的命令）。通过结合使用`mongodump`和`mongorestore`，您可以可靠地备份、恢复和迁移 MongoDB 数据库，跨不同的硬件和软件配置。

## 使用 mongodump

以下是一个最简单形式的`mongodump`命令：

```js
mongodump
```

有趣的是，您可以运行`mongodump`而不使用任何参数。这是因为命令需要使用的唯一信息是您的 MongoDB 服务器的位置。如果没有指定 URI 或主机，它将尝试创建运行在您本地系统上的 MongoDB 服务器的备份。

我们可以使用`--uri`参数来指定 URI，以指定我们的 MongoDB 服务器的位置。

注意

与`mongoexport`一样，`--db/--host`和`--uri`参数是互斥的。

然而，如果我们确实有一个本地运行的 MongoDB 服务器，这是我们可能会收到的输出：

```js
2020-08-18T12:38:43.091+1000    writing imports.newData to 
2020-08-18T12:38:43.091+1000    writing imports.contacts to 
2020-08-18T12:38:43.091+1000    writing imports.oldData to 
2020-08-18T12:38:43.310+1000    done dumping imports.newData (5 documents)
2020-08-18T12:38:44.120+1000    done dumping imports.contacts (3 documents)
2020-08-18T12:38:44.120+1000    done dumping imports.oldData (5 documents)
```

在这个命令结束时，我们可以看到我们的目录中有一个新的文件夹，其中包含我们数据库的备份。默认情况下，`mongodump`会导出 MongoDB 服务器中的所有内容。但是，我们可以更加有选择地进行导出，我们将在下一节中看到一个例子。

## mongodump 选项

`mongodump`命令需要非常少的选项才能运行；在大多数情况下，您可能只使用`--uri`参数。但是，我们可以使用几个选项来充分利用这个实用程序命令。以下是一些最有用的选项列表。

+   `--quiet`：这会减少备份时的输出信息量。

+   `--out`：这允许您指定导出的不同位置，以便将其写入磁盘，默认情况下将在运行命令的相同目录中创建一个名为“dump”的目录。

+   `--db`：这允许您指定要备份的单个数据库，默认情况下将备份所有数据库。

+   `--collection`：这允许您指定要备份的单个集合，默认情况下将备份所有集合。

+   `--excludeCollection`：这允许您指定要从备份中排除的集合。

+   `--query`：这允许您指定一个查询文档，将备份的文档限制为仅匹配查询的文档。

+   `--gzip`：如果启用，导出的输出将是一个压缩文件，格式为`.gz`，而不是一个目录。

我们将看看如何创建一个单个数据库的备份，包括用户和角色，并将其保存到磁盘上的特定位置。因为我们正在进行单个数据库的备份，所以可以使用`--uri`来指定要使用的数据库。

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports --out="./backups"
      2020-08-18T12:39:51.457+1000    writing imports.newData to 
2020-08-18T12:39:51.457+1000    writing imports.contacts to 
2020-08-18T12:39:51.457+1000    writing imports.oldData to 
2020-08-18T12:39:51.697+1000    done dumping imports.newData (5 documents)
2020-08-18T12:39:52.472+1000    done dumping imports.contacts (3 documents)
2020-08-18T12:39:52.493+1000    done dumping imports.oldData (5 documents)
```

正如您在前面的截图中所看到的，只有我们指定的数据库中存在的集合被导出。如果您查看包含我们导出内容的文件夹，甚至可以看到这一点：

```js
╭─ ~/backups
╰─ ls
      imports/
╭─ ~/backups
╰─ ls imports 
      contacts.bson          contacts.metadata.json newData.bson 
      newData.metadata.json  oldData.bson           oldData.metadata.json 
```

您可以在导入目录中看到，对于转储中的每个集合，都创建了两个文件，一个包含我们数据的`.bson`文件，一个用于集合元数据的`.metadata.json`文件。所有`mongodump`的结果都将匹配这种格式。

接下来，使用您的`--query`参数来仅转储集合中的特定文档。您可以使用标准查询文档来指定您的集合。例如，在 Windows 上考虑以下命令：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/sample_mflix --collection="movies" --out="./backups" --query="{genres: 'Action'}"
```

在 MacOS/Linux 上，您将不得不修改引号如下：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/sample_mflix --collection="movies" --out="./backups" --query='{"genres": "Action"}'
```

输出如下：

```js
2020-08-18T12:57:06.533+1000    writing sample_mflix.movies to 
2020-08-18T12:57:07.258+1000    sample_mflix.movies  101
2020-08-18T12:57:09.109+1000    sample_mflix.movies  2539
2020-08-18T12:57:09.110+1000    done dumping sample_mflix.movies (2539 documents)
```

电影收藏中有超过 20,000 个文档，但我们只导出了`2539`个匹配的文档。

现在，执行相同的导出，但不使用`--query`参数：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net/sample_mflix --collection="movies" --out="./backups"
```

输出如下：

```js
2020-08-18T12:57:45.263+1000    writing sample_mflix.movies to 
2020-08-18T12:57:45.900+1000    [........................]  sample_mflix.movies  101/23531  (0.4%)
2020-08-18T12:57:48.891+1000    [........................]  sample_mflix.movies  101/23531  (0.4%)
2020-08-18T12:57:51.894+1000    [##########..............]  sample_mflix.movies  10564/23531  (44.9%
)
2020-08-18T12:57:54.895+1000    [##########..............]  sample_mflix.movies  10564/23531  (44.9%)
2020-08-18T12:57:57.550+1000    [########################]  sample_mflix.movies  23531/23531  (100.0%)
2020-08-18T12:57:57.550+1000    done dumping sample_mflix.movies (23531 documents)
```

我们可以在前面的输出中看到，如果没有`--query`参数，导出的文档数量会显著增加，这意味着我们已经将从我们的集合中导出的文档数量减少到仅与查询匹配的文档。

与我们之前学习的命令一样，这些选项只代表您可以提供给`mongodump`的参数的一小部分。通过组合和尝试这些选项，您将能够为您的 MongoDB 服务器创建一个强大的备份和快照解决方案。

通过使用`mongoimport`和`mongoexport`，您已经能够轻松地将特定集合导入和导出数据库。然而，作为 MongoDB 服务器的备份策略的一部分，您可能希望备份整个 MongoDB 数据库的状态。在下一个练习中，我们将仅创建`sample_mflix`数据库的转储，而不是创建我们的 MongoDB 服务器中可能有的许多不同数据库的更大的转储。

## 练习 11.03：备份 MongoDB

在这个练习中，您将使用`mongodump`来创建`sample_mflix`数据库的备份。将数据导出到名为`movies_backup`的文件夹中的`.gz`文件。

完成此练习，执行以下步骤：

1.  为了调整您的导入并将其保存以备后用，创建一个名为`Exercise11.03.txt`的新文件来存储您的`mongodump`命令。

1.  接下来，输入标准的`mongodump`语法，只设置`--uri`参数。记住，`--uri`包含目标数据库。

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix
```

1.  接下来，添加指定转储位置的参数。在这种情况下，那就是一个名为`movies_backup`的文件夹：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=movies_backup
```

1.  最后，为了自动将您的转储文件放入`.gz`文件中，使用`--gzip`参数并运行命令。

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=movies_backup --gzip
```

注意

因为这个命令将转储整个`sample_mflix`数据库，所以根据您的互联网连接速度，可能需要一点时间。

一旦命令执行，您应该看到类似以下截图的输出：

![图 11.4：执行`mongodump`命令后的输出](img/B15507_11_04.jpg)

图 11.4：执行`mongodump`命令后的输出

1.  检查您的转储目录。您可以看到所有的`mongodump`数据都已经写入了正确的目录。

```js
╰─ ls movies_backup
      sample_mflix/
╰─ ls movies_backup/sample_mflix
      comments.bson.gz                   comments.metadata.json.gz
      most_commented_movies.bson.gz       most_commented_movies.metadata.json.gz 
      movies.bson.gz                      movies.metadata.json.gz
      movies_top_romance.bson.gz          movies_top_romance.metadata.json.gz
      sessions.bson.gz                    sessions.metadata.json.gz
      theaters.bson.gz                    theaters.metadata.json.gz
      users.bson.gz                       users.metadata.json.gz
```

在本练习过程中，您已经学会了如何编写一个`mongodump`命令，可以正确地创建数据库的压缩备份。现在，您将能够将这种技术作为数据库迁移或备份策略的一部分。

# 恢复 MongoDB 数据库

在前一节中，我们学习了如何使用`mongodump`创建整个 MongoDB 数据库的备份。然而，除非我们拥有一种将它们加载回 MongoDB 服务器的方法，否则这些导出对我们的备份策略没有好处。通过将我们的导出放回数据库的命令是`mongorestore`。

与`mongoimport`允许我们将常用格式导入 MongoDB 不同，`mongorestore`仅用于导入`mongodump`的结果。这意味着它最常用于将大部分或全部数据库恢复到特定状态。`mongorestore`命令非常适合在灾难后恢复转储，或者将整个 MongoDB 实例迁移到新配置。

当与我们的其他命令结合使用时，应该清楚`mongorestore`完成了导入和导出的生命周期。通过这三个命令（`mongoimport`，`mongoexport`和`mongodump`），我们已经学会了可以导出集合级别的数据，导入集合级别的数据，导出服务器级别的数据，现在最后，通过`mongorestore`，我们可以导入服务器级别的信息。

## 使用`mongorestore`

与其他命令一样，让我们来看一个`mongorestore`命令的简单实现。

```js
mongorestore .\dump\
```

或者在 MacOS/Linux 上，您可以输入以下内容：

```js
mongorestore ./dump/
```

我们需要传递的唯一必需参数是要还原的转储位置。但是，正如您可能已经从我们的其他命令中猜到的那样，默认情况下，`mongorestore`会尝试将备份还原到本地系统。

注意

转储位置不需要`--parameter`格式，而是可以作为命令的最后一个值传递。

在这里，我们可以再次使用`--uri`参数指定 URI，以指定我们的 MongoDB 服务器的位置。

例如，假设我们确实有一个正在运行的本地 MongoDB 服务器。要完成还原，我们需要之前创建的转储文件。以下是基于*练习 11.03，备份 MongoDB*的转储命令：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports --out=./dump
```

如果我们现在使用`--drop`选项对此转储运行`mongorestore`，您可能会看到类似以下的输出：

![图 11.5：使用`--drop`选项运行`mongorestore`后的输出](img/B15507_11_05.jpg)

图 11.5：使用`--drop`选项运行`mongorestore`后的输出

正如您所期望的，此输出应该与`mongoimport`的输出最为相似，告诉我们从转储文件中恢复了多少文档和索引。如果您的用例是作为备份策略的一部分进行还原，那么这个简单的命令和最少的参数就是您所需要的。

默认情况下，`mongorestore`会还原目标转储中的每个数据库、集合和文档。如果您希望在还原时更加具体，有几个方便的选项可以让您只还原特定的集合，甚至在还原过程中重命名集合。这些选项的示例将在下一节中提供。

## mongorestore 选项

与`mongodump`一样，`mongorestore`命令可以只使用其基本参数（如`--uri`和转储文件的位置）来满足大多数用例。如果您希望执行更具体类型的还原，可以使用以下一些选项：

+   --quiet：这会减少转储的输出消息量。

+   `--drop`：类似于`mongoimport`，`--drop`选项将在还原之前删除要还原的集合，确保命令运行后不会保留旧数据。

+   `--dryRun`：这允许您查看运行`mongorestore`的输出，而不实际更改数据库中的信息，这是在执行潜在危险操作之前测试命令的绝佳方式。

+   `--stopOnError`：如果启用，一旦发生单个错误，进程就会停止。

+   `--nsInclude`：这个选项允许您定义应从转储文件中导入哪些命名空间（数据库和集合），而不是明确提供数据库和集合。我们将在本章后面看到这个选项的示例。

+   `--nsExclude`：这是`nsInclude`的补充选项，允许您提供一个在还原时不导入的命名空间模式。下一节将提供一个示例。

+   `--nsFrom`：使用与`nsInclude`和`nsExclude`中相同的命名空间模式，此参数可以与`--nsTo`一起使用，提供导出中的命名空间到还原备份中的新命名空间的映射。这允许您在还原过程中更改集合的名称。

现在，让我们看一些使用这些选项的示例。请注意，对于这些示例，我们使用的是前一节创建的转储文件。作为提醒，这是创建此转储文件所需的命令：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=dump
```

首先，假设您有一个从`sample_mflix`数据库创建的完整`mongodump`。以下是还原我们集合的子集所需的命令示例。您可能会注意到参数的格式是`{数据库}.{集合}`，但是您可以使用通配符（`*`）运算符来匹配所有值。在以下示例中，我们包括与命名空间`"sample_mflix.movies"`匹配的任何集合（仅`sample_mflix`数据库的 movies 集合）。

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net --drop --nsInclude="sample_mflix.movies" dump
```

当此命令完成运行时，您应该会看到类似以下的输出：

```js
2020-08-18T13:12:28.204+1000    [###################.....]  sample_mflix.movies  7.53MB/9.06MB  (83.2%)
2020-08-18T13:12:31.203+1000    [#######################.]  sample_mflix.movies  9.04MB/9.06MB  (99.7%)
2020-08-18T13:12:33.896+1000    [########################]  sample_mflix.movies  9.06MB/9.06MB  (100.0%)
2020-08-18T13:12:33.896+1000    no indexes to restore
2020-08-18T13:12:33.902+1000    finished restoring sample_mflix.movies (6017 documents, 0 failures)
2020-08-18T13:12:33.902+1000    6017 document(s) restored successfully. 0 document(s) failed to restore.
```

在输出中，您可以看到只有匹配的命名空间被恢复。现在让我们看一下`nsFrom`和`nsTo`参数如何被用来重命名集合，使用与前面示例相同的格式。我们将在`sample_mflix`数据库中将集合重命名为相同的集合名称，但在一个名为`backup`的新数据库中：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net --drop --nsFrom="sample_mflix.*" --nsTo="backup.*" dump
```

一旦执行此命令完成，最后几行应该类似于以下内容：

```js
2020-08-18T13:13:54.152+1000    [################........]    backup.movies  6.16MB/9.06MB  (68.0%)
2020-08-18T13:13:54.152+1000
2020-08-18T13:13:56.916+1000    [########################]  backup.comments  4.35MB/4.35MB  (100.0%)
2020-08-18T13:13:56.916+1000    no indexes to restore
2020-08-18T13:13:56.916+1000    finished restoring backup.comments (16017 documents, 0 failures)
2020-08-18T13:13:57.153+1000    [###################.....]  backup.movies  7.53MB/9.06MB  (83.1%)
2020-08-18T13:14:00.152+1000    [#######################.]  backup.movies  9.04MB/9.06MB  (99.7%)
2020-08-18T13:14:02.929+1000    [########################]  backup.movies  9.06MB/9.06MB  (100.0%)
2020-08-18T13:14:02.929+1000    no indexes to restore
2020-08-18T13:14:02.929+1000    finished restoring backup.movies (6017 documents, 0 failures)
2020-08-18T13:14:02.929+1000    23807 document(s) restored successfully. 0 document(s) failed to restore. 
```

现在，如果我们观察一下我们的 MongoDB 数据库中的集合，我们会发现`sample_mflix`集合也存在于名为`backup`的数据库中，例如：

```js
MongoDB Enterprise atlas-nb3biv-shard-0:PRIMARY> use backup
switched to db backup
MongoDB Enterprise atlas-nb3biv-shard-0:PRIMARY> show collections
comments
most_commented_movies
movies
movies_top_romance
sessions
theaters
users
```

最后，让我们快速看一下`dryRun`参数的工作原理。看一下以下命令：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlasServer-fawxo.gcp.mongodb.net --drop --nsFrom="imports.*" --nsTo="backup.*" --dryRun .\dump\
```

您将注意到有一个关于命令准备恢复的输出。但是，它不会加载任何数据。MongoDB 中的基础数据都没有改变。这是确保在执行命令之前，确保命令不会出错的绝佳方法。

`mongorestore`命令完成了我们的四个命令，即`mongoimport`，`mongoexport`，`mongodump`和`mongorestore`。虽然使用`mongorestore`很简单，但如果您的备份策略设置更复杂，您可能需要使用多个选项并参考文档。

## 练习 11.04：恢复 MongoDB 数据

在上一个练习中，您使用`mongodump`创建了`sample_mflix`数据库的备份。作为 MongoDB 服务器的备份策略的一部分，您现在需要将这些数据放回数据库。在这个练习中，假设您从导出的数据库和导入的数据库是不同的数据库。因此，为了向客户证明备份策略有效，您将使用`mongorestore`将该转储数据导入到不同的命名空间中。

注意

在完成此练习之前，您需要从*练习 11.03*，*备份 MongoDB*中创建一个转储。

在这个练习中，您将使用`mongorestore`从上一个练习中创建的`movies_backup`转储中恢复`sample_mflix`数据库，并将每个集合的命名空间更改为`backup_mflix`。

1.  调整您的导入并保存以备后用。创建一个名为`Exercise11.04.txt`的新文件来存储您的恢复命令。

1.  确保*练习 11.03*，*备份 MongoDB*中的`movies_backup`转储也在您当前的目录中。否则，您可以使用以下命令创建一个新的备份：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=./movies_backup --gzip
```

1.  接下来，只需输入标准的`mongorestore`语法，只提供 URI 和转储文件的位置。记住，URI 中包括目标数据库：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net ./movies_backup
```

1.  由于转储文件是以`gzip`格式，您还需要在恢复命令中添加`--gzip`参数，以便它可以解压数据。

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --gzip ./movies_backup
```

1.  为了确保恢复结果干净，使用您的`--drop`参数在尝试恢复之前删除相关集合：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --gzip --drop ./movies_backup
```

1.  现在，添加修改命名空间的参数。因为您正在恢复`sample_mflix`数据库的转储，所以`"sample_mflix"`将是您的`nsFrom`参数的值：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --nsFrom="sample_mflix.*" --gzip --drop ./movies_backup
```

1.  这种用例规定这些集合将被恢复到一个名为`backup_mflix`的数据库中。使用`nsTo`参数提供这个新的命名空间。

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --nsFrom="sample_mflix.*" --nsTo="backup_mflix.*" --gzip --drop ./movies_backup
```

1.  您的命令现在已经完成。将此代码复制并粘贴到您的终端或命令提示符中并运行。将会有大量输出来显示恢复的进度，但最后，您应该会看到如下输出：

```js
2020-08-18T13:18:08.862+1000    [####################....]  backup_mflix.movies  10.2MB/11.7MB  (86.7%)
2020-08-18T13:18:11.862+1000    [#####################...]  backup_mflix.movies  10.7MB/11.7MB  (90.8%)
2020-08-18T13:18:14.865+1000    [######################..]  backup_mflix.movies  11.1MB/11.7MB  (94.9%)
2020-08-18T13:18:17.866+1000    [#######################.]  backup_mflix.movies  11.6MB/11.7MB  (98.5%)
2020-08-18T13:18:20.217+1000    [########################]  backup_mflix.movies  11.7MB/11.7MB  (100.0%)
2020-08-18T13:18:20.217+1000    restoring indexes for collection backup_mflix.movies from metadata
2020-08-18T13:18:26.389+1000    finished restoring backup_mflix.movies (23531 documents, 0 failures)
2020-08-18T13:18:26.389+1000    75594 document(s) restored successfully. 0 document(s) failed to restore.
```

从输出中可以看出，恢复已完成，将每个现有集合恢复到名为`backup_mflix`的新数据库中。输出甚至会告诉您恢复的一部分写入了多少个文档。例如，`23541`个文档被恢复到`movies`集合中。

现在，如果您使用 mongo shell 登录到服务器，您应该能够看到您新恢复的`backup_mflix`数据库和相关集合，如下所示：

```js
MongoDB Enterprise atlas-nb3biv-shard-0:PRIMARY> use backup_mflix
switched to db backup_mflix
MongoDB Enterprise atlas-nb3biv-shard-0:PRIMARY> show collections
comments
most_commented_movies
movies
movies_top_romance
sessions
theaters
users
```

就是这样。你已经成功地将备份还原到了 MongoDB 服务器中。有了对`mongorestore`的工作知识，你现在可以高效地备份和迁移整个 MongoDB 数据库或服务器。正如本章前面提到的，你也许可以用`mongoimport`来完成同样的任务，但是能够使用`mongodump`和`mongorestore`会让你的任务变得更简单。

通过本章学到的四个关键命令（`mongoexport`，`mongoimport`，`mongodump`和`mongorestore`），你现在应该能够完成大部分与 MongoDB 一起工作时遇到的备份、迁移和还原任务。

## 活动 11.01：MongoDB 中的备份和还原

你的客户（电影公司）已经有了几个每晚运行的脚本，用于导出、导入、备份和还原数据。他们进行备份和导出是为了确保数据有冗余的副本。然而，由于他们对 MongoDB 的经验不足，这些命令并没有正确运行。为了解决这个问题，他们请求你帮助他们优化他们的备份策略。按照以下步骤完成这个活动：

注意

这个活动中的四个命令必须按正确的顺序运行，因为`import`和`restore`命令依赖于`export`和`dump`命令的输出。

1.  按`theaterId`字段，按`theaterId`排序，导出到名为`theaters.csv`的 CSV 文件中：

```js
mongoexport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --db=sample_mflix --collection=theaters --out="theaters.csv" --type=csv --sort='{theaterId: 1}'
```

1.  将`theaters.csv`文件导入到名为`theaters_import`的新集合中：

```js
mongoimport --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/imports --collection=theaters_import --file=theaters.csv
```

1.  将`theaters`集合以`gzip`格式备份到名为`backups`的文件夹中：

```js
mongodump --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net/sample_mflix --out=./backups –gz --nsExclude=theaters
```

1.  `sample_mflix_backup`：

```js
mongorestore --uri=mongodb+srv://USERNAME:PASSWORD@myAtlas-fawxo.gcp.mongodb.net --from="sample_mflix" --to="backup_mflix_backup" --drop ./backups
```

你的目标是接受客户提供的脚本，确定这些脚本有什么问题，并解决这些问题。你可以在自己的 MongoDB 服务器上测试这些脚本是否正确运行。

你可以通过几种方式完成这个目标，但要记住我们在整个章节中学到的东西，并尝试创建简单、易于使用的代码。以下步骤将帮助你完成这个任务：

1.  目标数据库被指定了两次，尝试移除多余的参数。

1.  重新运行`export`命令。我们缺少一个特定于 CSV 格式的选项。添加这个参数以确保我们导出`theaterId`和 location 字段。

现在看看`import`命令，你应该立即注意到一些缺少的参数，这些参数是 CSV 导入所需的。

1.  首先对于`dump`命令，有一个选项是不正确的；运行命令以获取提示。

1.  其次，`dump`命令中没有`nsInclude`选项，因为这是`mongorestore`的选项。用适用于`mongodump`的适当选项替换它。

1.  在`restore`命令中，有一些选项的名称不正确。修复这些名称。

1.  同样在`restore`命令中，从前一个命令中还原一个`gzip`格式的 dump。在还原命令中添加一个选项来支持这种格式。

1.  最后，在`restore`命令中，查看`nsFrom`和`nsTo`选项的值，并检查它们是否在正确的命名空间格式中。

为了测试你的结果，按顺序运行这四个命令（导出，导入，dump，还原）。

`mongoexport`命令的输出如下：

```js
2020-08-18T13:21:29.778+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.gcp.mongodb.net/sample_mflix
2020-08-18T13:21:30.891+1000    exported 1564 records
```

`mongoimport`命令的输出如下：

```js
2020-08-18T13:22:20.720+1000    connected to: mongodb+srv://[**REDACTED**]@performancetuning.98afc.g
cp.mongodb.net/imports
2020-08-18T13:22:22.817+1000    1564 document(s) imported successfully. 0 document(s) failed to import.
```

`mongodump`命令的输出如下：

![图 11.6 mongodump 命令的输出](img/B15507_11_06.jpg)

图 11.6 mongodump 命令的输出

`mongorestore`命令的输出开始如下：

![图 11.7：mongorestore 命令的输出开始](img/B15507_11_07.jpg)

图 11.7：mongodump 命令的输出开始

`mongorestore`命令的输出结束如下：

![图 11.8：mongorestore 命令的输出结束](img/B15507_11_08.jpg)

图 11.8：mongorestore 命令的输出结束

注意

此活动的解决方案可通过此链接找到。

# 摘要

在这一章中，我们涵盖了四个单独的命令。然而，这四个命令都作为 MongoDB 完整备份和恢复生命周期中的元素。通过结合这些基本命令和它们的高级选项，您现在应该能够确保您负责的任何 MongoDB 服务器在数据损坏、丢失或灾难发生时都能得到适当的快照、备份、导出和恢复。

您可能不负责备份 MongoDB 数据，但这些命令也可以用于各种实用程序。例如，将数据导出为 CSV 格式将非常方便，当尝试以电子表格的形式直观地探索信息，甚至向不熟悉文档模型的同事展示时。通过使用 `mongoimport`，您还可以减少导入非 MongoDB 格式数据所需的手动工作量，以及批量导入来自其他服务器的 MongoDB 数据。

下一章涵盖了数据可视化，这是一个非常重要的概念，可以将 MongoDB 信息转化为易于理解的结果，为业务问题提供洞察和清晰度，并将其整合到演示文稿中，以说服利益相关者对数据中难以解释的趋势产生兴趣。
