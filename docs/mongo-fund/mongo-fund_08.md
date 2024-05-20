# 第八章：在 MongoDB 中编写 JavaScript 代码

概述

在本章中，您将学习如何使用 Node.js 驱动程序阅读、理解和创建简单的 MongoDB 应用程序。这些应用程序将帮助您以编程方式获取、更新和创建 MongoDB 集合中的数据，以及处理错误和用户输入。在本章结束时，您将能够创建一个简单的基于 MongoDB 的应用程序。

# 介绍

到目前为止，我们直接使用 mongo shell 与 MongoDB 数据库进行了交互。这些直接的交互快速、简单，是学习或实验 MongoDB 功能的绝佳方式。然而，在许多生产情况下，将是软件代替用户连接到数据库。MongoDB 是一个很好的存储和查询数据的地方，但通常，它最重要的用途是作为大规模应用程序的后端。这些应用程序通常在某些条件或用户界面触发后以编程方式写入、读取和更新数据。

要将您的软件与数据库连接，通常会使用一个库（通常由数据库创建者提供）称为驱动程序。这个驱动程序将帮助您连接、分析、读取和写入数据库，而无需为简单操作编写多行代码。它提供了常见用例的函数和抽象，以及用于处理从数据库中提取的数据的框架。MongoDB 为不同的编程语言提供了几种不同的驱动程序，其中最流行的（也是我们将在本章中探讨的）是 Node.js 驱动程序（有时称为 Node）。

要将这与现实生活联系起来，想想您的在线购物体验。第一次从网站购买产品时，您必须输入所有的账单和送货细节。然而，如果您已经注册了一个账户，第二次去结账时，所有的细节都已经保存在网站上。这是一个很好的体验，而且在许多网站上，这是通过 Web 应用程序查询后端数据库来实现的。MongoDB 是可以支持这些应用程序的一个这样的数据库。

MongoDB 取得如此出色的增长和采用的主要原因之一是其成功说服软件开发人员选择它作为其应用程序的数据库。其中很大一部分说服力来自于 MongoDB 与 Node 的良好集成。

Node.js 已经成为基于 Web 的应用程序的主要语言之一，我们将在本章后面学习。然而，现在知道 Node 和 MongoDB 集成的便利性对两种技术都非常有益就足够了。这种共生关系还导致了大量成功的 Node/MongoDB 实现，从小型移动应用到大规模 Web 应用。在展示 MongoDB 驱动程序时，选择 Node.js 是首选。

根据您的工作角色，您可能负责编写针对 MongoDB 运行的应用程序，或者期望偶尔编写一行代码。然而，无论您的编程水平或专业责任如何，了解应用程序如何使用驱动程序与 MongoDB 集成将非常有价值。大多数 MongoDB 生产查询是由应用程序而不是人运行的。无论您是数据分析师、前端开发人员还是数据库管理员，您的生产环境很可能会使用 MongoDB 驱动程序之一。

注意

在本章的整个持续时间内，包括的练习和活动都是对一个情景的迭代。数据和示例都基于名为`sample_mflix`的 MongoDB Atlas 示例数据库。

在本章的整个持续时间内，我们将按照一个基于理论情景的一系列练习。这是我们在第七章“聚合”中涵盖的情景的扩展。

在*第七章*“聚合”中构建场景的基础上，一个电影公司正在举办年度经典电影马拉松，并希望决定他们的放映计划应该是什么，他们需要满足特定标准的各种受欢迎的电影来满足他们的客户群。在探索数据并协助他们做出业务决策后，您为他们提供了新的见解。电影公司对您的建议感到满意，并决定让您参与他们的下一个项目。该项目涉及创建一个简单的 Node.js 应用程序，允许他们的员工查询电影数据库，而无需了解 MongoDB 并对应该在电影院放映哪些电影进行投票。在本章的过程中，您将创建此应用程序。

# 连接到驱动程序

在高层次上，使用 Node.js 驱动程序与 MongoDB 的过程类似于直接连接 shell。您将指定 MongoDB 服务器 URI、几个连接参数，并且可以对集合执行查询。这应该都很熟悉；主要区别在于这些指令将以 JavaScript 而不是 Bash 或 PowerShell 编写。

## Node.js 简介

由于本章的目标不是学习 Node.js 编程，我们将简要介绍基础知识，以确保我们可以创建我们的 MongoDB 应用程序。Node.js 中的**js**代表**JavaScript**，因为 JavaScript 是 Node.js 理解的编程语言。JavaScript 通常在浏览器中运行。但是，您可以将 Node.js 视为在计算机上执行 JavaScript 文件的引擎。

在本章的过程中，您将编写 JavaScript（`.js`）语法，并使用 Node.js 执行它。虽然您可以使用任何文本编辑器编写 JavaScript 文件，但建议使用可以帮助您进行语法高亮和格式化的应用程序，例如**Visual Studio Code**或**Sublime**。

首先，让我们看一些示例代码：

```js
// 1_Hello_World.js
var message = "Hello, Node!";
console.log(message);
```

让我们详细定义前面语法中的每个术语：

+   `var`关键字用于声明一个新变量；在本例中，变量名为`message`。

+   `=`符号将此变量的值设置为一个名为`Hello, Node!`的字符串。

+   在每个语句的末尾使用分号（`;`）。

+   `console.log(message)`是用于输出`message`值的函数。

如果您熟悉编程基础知识，您可能已经注意到我们不必将`message`变量显式声明为`string`。这是因为 JavaScript 是**动态类型**的，这意味着您不必显式指定变量类型（数字、字符串、布尔值等）。

如果您对编程基础知识不太熟悉，本章中的一些术语可能会使您感到困惑。因为这不是一本 JavaScript 编程书，这些概念不会被深入讨论。本章的目标是了解驱动程序如何与 MongoDB 交互；Node.js 的具体内容并不重要。尽管本章试图保持编程概念简单，但如果有什么复杂的地方，不要担心。

让我们尝试运行代码示例，将该代码保存到名为`1_Hello_World.js`的文件中，保存到我们当前的目录中，然后使用以下命令在我们的终端或命令提示符中运行该命令：

```js
> node 1_Hello_World.js
```

您将看到一个看起来像这样的输出：

```js
Section1> node 1_Hello_World.js
Hello, Node!
Section1>
```

如您所见，运行 Node.js 脚本非常简单，因为无需构建或编译，您可以编写代码并使用`node`调用它。

`var`关键字将信息存储在变量中，并在代码中稍后更改。但是，还有另一个关键字`const`，用于存储不会更改的信息。因此，在我们的示例中，我们可以用`const`关键字替换我们的`var`关键字。作为最佳实践，您可以将任何不会更改的内容声明为`const`：

```js
// 1_Hello_World.js
const message = "Hello, Node!";
console.log(message);
```

现在，让我们考虑函数和参数的结构。就像在 mongo shell 中的前几章查询的结构一样。首先，让我们考虑定义函数的以下示例：

```js
var printHello = function(parameter) {
    console.log("Hello, " + parameter);
}
printHello("World")
```

以下是我们将在本章后面遇到的一些代码类型的预览。您可能会注意到，尽管这是一个更复杂的代码片段，但与您在早期章节（*第四章*，*查询文档*，特别是）学到的 CRUD 操作有一些共同的元素，例如`find`命令的语法和 MongoDB URI：

```js
// 3_Full_Example.js
const Mongo = require('mongodb').MongoClient;
const server = 'mongodb+srv://username:password@server-  abcdef.gcp.mongodb.net/test?retryWrites=true&w=majority'
const myDB   = 'sample_mflix'
const myColl = 'movies';
const mongo = new Mongo(server);
mongo.connect(function(err) {
    console.log('Our driver has connected to MongoDB!');
    const database = mongo.db(myDB);
    const collection = database.collection(myColl);
    collection.find({title: 'Blacksmith Scene'}).each(function(err, doc) {
        if(doc) {
            console.log('Doc returned: ')
            console.log(doc);
        } else {
            mongo.close();
            return false;
        }
    })
})
```

开始时可能有点令人生畏，但随着我们深入探讨本章，这将变得更加熟悉。正如我们之前提到的，即使它们看起来有些不同，您应该能够从 mongo shell 中识别出一些元素。代码中映射到 mongo shell 元素的一些元素如下：

+   `collection`对象，就像 shell 中的`db.collection`。

+   在我们的`collection`之后使用`find`命令，就像在 shell 中一样。

+   我们`find`命令中的参数是一个文档过滤器，这正是我们在 shell 中使用的。

在 Node.js 中，函数声明是使用`function(parameter){…}`函数完成的，它允许我们创建可以多次运行的较小、可重用的代码片段，例如`find()`或`insertOne()`函数。定义函数很容易；您只需使用`function`关键字，后跟函数的名称、括号中的参数和大括号来定义此函数的实际逻辑。

这是定义函数的代码。请注意，有两种方法可以做到这一点：您可以将函数声明为变量，也可以将函数作为参数传递给另一个函数。我们将在本章后面详细介绍这一点：

```js
// 4_Define_Function.js
const newFunction = function(parameter1, parameter2) {
    // Function logic goes here.
    console.log(parameter1);
    console.log(parameter2);
}
```

## 获取 Node.js 的 MongoDB 驱动程序

安装 Node.js 的 MongoDB 驱动程序最简单的方法是使用`npm`。`npm`，或 node 包管理器，是一个用于添加、更新和管理 Node.js 程序中使用的不同包的包管理工具。在这种情况下，您要添加的包是 MongoDB 驱动程序，因此在存储脚本的目录中，在您的终端或命令提示符中运行以下命令：

```js
> npm install mongo --save
```

安装包后可能会看到一些输出，如下所示：

![图 8.1：使用 npm 安装 MongoDB 驱动程序](img/B15507_08_01.jpg)

图 8.1：使用 npm 安装 MongoDB 驱动程序

就这么简单。现在，让我们开始针对 MongoDB 进行编程。

## 数据库和集合对象

在使用 MongoDB 驱动程序时，您可以使用三个主要组件进行大多数操作。在后面的练习中，我们将看到它们如何组合在一起，但在那之前，让我们简要介绍每个组件及其目的。

`MongoClient`是您在代码中必须创建的第一个对象。这代表您与 MongoDB 服务器的连接。将其视为 mongo shell 的等价物；您传入数据库的 URL 和连接参数，它将为您创建一个连接供您使用。要使用`MongoClient`，您必须在脚本顶部导入模块：

```js
// First load the Driver module.
const Mongo = require('MongoDB').MongoClient;
// Then define our server.
const server = 'mongodb+srv://username:password@server-  abcdef.gcp.mongodb.net/test?retryWrites=true&w=majority';
// Create a new client.
const mongo = new Mongo(server);
// Connect to our server.
mongo.connect(function(err) {
    // Inside this block we are connected to MongoDB.
mongo.close(); // Close our connection at the end.
})
```

接下来是`database`对象。就像 mongo shell 一样，一旦建立连接，您可以针对服务器中的特定数据库运行命令。这个数据库对象还将确定您可以针对哪些集合运行查询：

```js
…
mongo.connect(function(err) {
    // Inside this block we are connected to MongoDB.
    // Create our database object.
    const database = mongo.db(«sample_mflix»);
    mongo.close(); // Close our connection at the end.
})
…
```

在（几乎）每个基于 MongoDB 的应用程序中使用的第三个基本对象是`collection`对象。正如在 mongo shell 中一样，大多数常见操作将针对单个集合运行：

```js
…
mongo.connect(function(err) {
    // Inside this block we are connected to MongoDB.
    // Create our database object.
    const database = mongo.db("sample_mflix");
    // Create our collection object
    const collection = database.collection("movies");
    mongo.close(); // Close our connection at the end.
})
…
```

`database`和`collection`对象表达了与直接连接 mongo shell 相同的概念。在本章中，`MongoClient`仅用于创建和存储与服务器的连接。

重要的是要注意，这些对象之间的关系是`MongoClient`对象可以创建多个`database`对象，而`database`对象可以创建多个用于运行查询的`collection`对象：

![图 8.2：驱动程序实体关系](img/B15507_08_02.jpg)

图 8.2：驱动程序实体关系

上图是对前面段落中描述的实体关系的可视化表示。这里有一个`MongoClient`对象对应多个`database`对象，每个`database`对象可能有多个用于运行查询的`collection`对象。

## 连接参数

在编写代码之前，了解如何建立到`MongoClient`的连接是很重要的。创建新客户端时只有两个参数：服务器的 URL 和任何额外的连接选项。如果需要创建客户端，连接选项是可选的，如下所示：

```js
const serverURL = 'mongodb+srv://username:password@server-  abcdef.gcp.mongodb.net/test';
const mongo = new Mongo(serverURL);
mongo.connect(function(err) {
    // Inside this block we are connected to MongoDB.
mongo.close(); // Close our connection at the end.
})
```

注意

```js
callback. We will cover these in detail later in this chapter. For now, it is enough to use this pattern without having a more in-depth understanding.
```

与 mongo shell 一样，`serverURL`支持所有 MongoDB URI 选项，这意味着您可以在连接字符串本身中指定配置，而不是在第二个可选参数中；例如：

```js
const serverURL = 'mongodb+srv://username:password@server-  abcdef.gcp.mongodb.net/test?retryWrites=true&w=majority';
```

为了简化这个字符串，可以在创建客户端时在第二个参数中指定许多这些 URI 选项（以及其他选项，例如 SSL 设置）；例如：

```js
const mongo = new Mongo(serverURL, {
     sslValidate: false
});
mongo.connect(function(err) {
     // Inside this block we are connected to MongoDB.
mongo.close(); // Close our connection at the end.
})
```

与 mongo shell 一样，有许多配置选项，包括 SSL、身份验证和写入关注选项。然而，大部分超出了本章的范围。

注意

请记住，您可以在 cloud.mongodb.com 的用户界面中找到 Atlas 的完整连接字符串。您可能希望复制此连接字符串，并在所有脚本中使用它作为`serverURL`。

让我们通过练习学习如何与 Node.js 驱动程序建立连接。

## 练习 8.01：使用 Node.js 驱动程序创建连接

在开始这个练习之前，回顾一下*介绍*部分中概述的电影公司。您可能还记得电影公司希望有一个 Node.js 应用程序，允许用户查询和更新电影数据库中的记录。为了实现这一点，您的应用程序首先需要建立与服务器的连接。可以通过执行以下步骤来完成：

1.  首先，在您当前的工作目录中，创建一个名为`Exercise8.01.js`的新 JavaScript 文件，并在您选择的文本编辑器（Visual Studio Code、Sublime 等）中打开它：

```js
> node Exercise8.01.js
```

1.  通过将以下行添加到文件顶部，将 MongoDB 驱动程序库（如本章前面所述）导入到您的脚本文件中：

```js
const MongoClient = require('mongodb').MongoClient;
```

注意

如果您在本章早期没有安装 npm MongoDB 库，现在应该运行`npm install mongo --save`在命令提示符或终端中进行安装。在与您的脚本相同的目录中运行此命令。

1.  创建一个包含您的 MongoDB 服务器的 URL 的新变量：

```js
const url = 'mongodb+srv://username:password@server-  abcdef.gcp.mongodb.net/test';
```

1.  创建一个名为`client`的新`MongoClient`对象，使用`url`变量：

```js
const client = new MongoClient(url);
```

1.  使用以下方式打开到 MongoDB 的连接`connect`函数：

```js
client.connect(function(err) {
     …
})
```

1.  在连接块中添加一个`console.log()`消息，以确认连接已打开：

```js
console.log('Connected to MongoDB with NodeJS!');
```

1.  最后，在连接块的末尾，使用以下语法关闭连接：

```js
client.close(); // Close our connection at the end.
```

您的完整脚本应如下所示：

```js
// Import MongoDB Driver module.
const MongoClient = require('mongodb').MongoClient;
// Create a new url variable.
const url = 'mongodb+srv://username:password@server-  abcdef.gcp.mongodb.net/test';
// Create a new MongoClient.
const client = new MongoClient(url);
// Open the connection using the .connect function.
client.connect(function(err) {
    // Within the connection block, add a console.log to confirm the       connection
    console.log('Connected to MongoDB with NodeJS!');
    client.close(); // Close our connection at the end.
})
```

使用`node Exercise8.01.js`执行代码后，将生成以下输出：

```js
Chapter8> node Excercise8.01.js
Connected to MongoDB with NodeJS!
Chapter8>
```

在这个练习中，您使用 Node.js 驱动程序建立了与服务器的连接。

# 执行简单查询

现在我们已经连接到 MongoDB，可以对数据库运行一些简单的查询。在 Node.js 驱动程序中运行查询与在 shell 中运行查询非常相似。到目前为止，您应该熟悉 shell 中的`find`命令：

```js
db.movies.findOne({})
```

以下是驱动程序中`find`命令的语法：

```js
collection.find({title: 'Blacksmith Scene'}).each(function(err, doc) { … }
```

正如您所看到的，一般结构与您在 mongo shell 中执行的`find`命令相同。在这里，我们从数据库对象中获取一个集合，然后针对该集合运行带有查询文档的`find`命令。这个过程本身很简单。主要的区别在于我们如何构造我们的命令以及如何处理驱动程序返回的结果。

在编写 Node.js 应用程序时，一个关键的问题是确保您的代码以一种易于修改、扩展或理解的方式编写，无论是将来您自己还是其他专业人士可能需要在应用程序上工作。

## 创建和执行 find 查询

将*Exercise 8.01*中的代码，*使用 Node.js 驱动程序创建连接*，作为参考，因为它已经包含了连接：

```js
const MongoClient = require('mongodb').MongoClient;
// Replace this variable with the connection string for your server, provided by   MongoDB Atlas.
const url = 'mongodb+srv://username:password@server-abcdef.gcp.mongodb.net/test';
const client = new MongoClient(url);
client.connect(function(err) {
    console.log('Connected to MongoDB with NodeJS!');
    // OUR CODE GOES BELOW HERE
    // AND ABOVE HERE
    client.close();
})
```

我们的查询逻辑将在这里添加：

```js
    // OUR CODE GOES BELOW HERE
    // AND ABOVE HERE
```

现在，我们已经连接到了 MongoDB 服务器。但是，还有两个重要的对象——`db`和`collection`。让我们创建我们的数据库对象（用于`sample_mflix`数据库），如下所示：

```js
    // OUR CODE GOES BELOW HERE
    const database = client.db("sample_mflix")
    // AND ABOVE HERE
```

现在我们有了我们的`database`对象。在 mongo shell 中发送查询时，您必须将文档作为命令的过滤器传递给您的文档。这在 Node.js 驱动程序中也是一样的。您可以直接传递文档。但是，建议将过滤器单独定义为变量，然后再分配一个值。您可以在以下代码片段中看到差异：

```js
// Defining filter first.
var filter = { title: 'Blacksmith Scene'};
database.collection("movies").find(filter).toArray(function(err, docs) { });
// Doing everything in a single line.
database.collection("movies").find({title: 'Blacksmith   Scene'}).toArray(function(err, docs) {});
```

与 mongo shell 一样，您可以将空文档作为参数传递以查找所有文档。您可能还注意到我们的`find`命令末尾有`toArray`。这是因为，默认情况下，`find`命令将返回一个游标。我们将在下一节中介绍游标，但与此同时，让我们看看这个完整脚本会是什么样子：

```js
const MongoClient = require('mongodb').MongoClient;
// Replace this variable with the connection string for your server, provided by   MongoDB Atlas.
const url = 'mongodb+srv://mike:password@myAtlas-  fawxo.gcp.mongodb.net/test?retryWrites=true&w=majority'
const client = new MongoClient(url);
client.connect(function(err) {
    console.log('Connected to MongoDB with NodeJS!');
    const database = client.db("sample_mflix");
    var filter = { title: 'Blacksmith Scene'};
    database.collection("movies").find(filter).toArray(function(err, docs) {
        console.log('Docs results:');
        console.log(docs);
     });
    client.close();
})
```

如果您将此修改后的脚本保存为`2_Simple_Find.js`并使用命令`node 2_Simple_Find.js`运行它，将会得到以下输出：

![图 8.3：上述片段的输出（为简洁起见而截断）](img/B15507_08_03.jpg)

图 8.3：上述片段的输出（为简洁起见而截断）

上述输出与通过 mongo shell 而不是驱动程序执行的 MongoDB 查询的输出非常相似。在通过驱动程序执行查询时，我们已经了解到，尽管语法可能与 mongo shell 不同，但查询及其输出中的基本元素是相同的。

## 使用游标和查询结果

在前面的示例中，我们使用`toArray`函数将我们的查询输出转换为一个可以用`console.log`输出的数组。当处理少量数据时，这是一种简单的处理结果的方法；然而，对于较大的结果集，您应该使用游标。您应该对游标有一定的了解，这是从*第五章*，*插入、更新和删除文档*中的 mongo shell 查询中得到的。在 mongo shell 中，您可以使用`it`命令来遍历游标。在 Node.js 中，有许多访问游标的方式，其中三种更常见的模式如下：

+   `toArray`：这将获取查询的所有结果并将它们放入一个单一的数组中。这很容易使用，但当您期望从查询中获得大量结果时，效率不是很高。在以下代码中，我们针对电影集合运行`find`命令，然后使用`toArray`将数组中的第一个元素记录到控制台中：

```js
database.collection("movies").find(filter).toArray(function(err, docsArray) {
    console.log('Docs results as an array:');
    console.log(docsArray[0]); // Print the first entry in the array.
 });
```

+   `each`：这将逐个遍历结果集中的每个文档。如果您想要检查或使用结果中的每个文档，这是一个很好的模式。在以下代码片段中，我们针对电影集合运行`find`命令，使用`each`记录返回的每个文档，直到没有文档为止：

```js
database.collection("movies").find(filter).each(function(err, doc) {
    if(doc) {
        console.log('Current doc');
        console.log(doc);
    } else {
        client.close(); // Close our connection.
        return false;   // End the each loop.
    }
 });
```

当没有更多文档返回时，文档将等于`null`。因此，每次检查新文档时，检查文档是否存在（使用`if(doc)`）是很重要的。

+   `next`：这将允许你访问结果集中的下一个文档。如果你只想要一个单独的文档或结果的子集，而不必遍历整个结果，这是最好的模式。在下面的代码片段中，我们对电影集合运行了一个`find`命令，使用`next`获取返回的第一个文档，然后将该文档输出到控制台：

```js
database.collection("movies").find(filter).next(function(err, doc) {
    console.log("First doc in the cursor");
    console.log(doc);
 });
```

因为`next`一次只返回一个文档，在这个例子中，我们运行了三次来检查前三个文档。

在本章的示例、练习和活动中，我们将学习这三种方法是如何被使用的。然而，需要注意的是还有其他更高级的模式。

你也可以通过在`find(…)`之后放置这些命令来实现相同的`sort`和`limit`功能，这应该是你在 shell 中以前查询时熟悉的：

```js
database.collection("movies").find(filter).limit(5).sort([['title', 1]]).next   (function(err, doc) {…}
```

## 练习 8.02：构建一个 Node.js Driver 查询

在这个练习中，你将在*练习 8.01*的场景上进行构建，*使用 Node.js Driver 创建连接*，这允许你连接到 mongo 服务器。如果你要交付一个 Node.js 应用程序，允许电影院员工查询和对电影投票，你的脚本将需要根据给定的条件查询数据库，并以易于阅读的格式返回结果。对于这种情况，你必须获取以下查询的结果：

*查找两部浪漫类电影，只投影每部电影的标题。*

你可以通过执行以下步骤在 Node.js 中实现这一点：

1.  创建一个名为`Exercise8.02.js`的新的 JavaScript 文件。

1.  为了不必从头开始重写所有内容，将`Exercise8.01.js`的内容复制到你的新脚本中。否则，在你的新文件中重写连接代码。

1.  为了保持代码整洁，创建新的变量来存储`databaseName`和`collectionName`。记住，由于这些在整个脚本中不会改变，你必须使用`const`关键字将它们声明为常量：

```js
const databaseName = "sample_mflix";
const collectionName = "movies";
```

1.  现在，创建一个新的`const`来存储我们的查询文档；你应该熟悉从之前的章节中创建这些：

```js
const query = { genres: { $all: ["Romance"]} };
```

1.  定义好所有的变量后，创建我们的数据库对象：

```js
const database = client.db(databaseName);
```

现在，你可以使用以下语法发送你的查询。使用`each`模式，传递一个回调函数来处理每个文档。如果这看起来奇怪，不要担心；你将在接下来的部分中详细了解这个。记得使用`limit`只返回两个文档和`project`只输出`title`，因为它们是我们场景的要求：

```js
database.collection(collectionName).find(query).limit(2).project({title:   1}).each(function(err, doc) {
    if(doc) {

    } else {
        client.close(); // Close our connection.
        return false;   // End the each loop.
    }
 });
```

1.  在你的回调函数中，使用`console.log`输出我们的查询返回的每个文档：

```js
if(doc){
           console.log('Current doc');
           console.log(doc);
}  
```

你的最终代码应该像这样：

```js
const MongoClient = require('mongodb').MongoClient;
const url = 'mongodb+srv://username:password@server-  abcdef.gcp.mongodb.net/test';
const client = new MongoClient(url);
const databaseName = "sample_mflix";
const collectionName = "movies";
const query = { genres: { $all: ["Romance"]} };
// Open the connection using the .connect function.
client.connect(function(err) {
    // Within the connection block, add a console.log to confirm the       connection
    console.log('Connected to MongoDB with NodeJS!');
    const database = client.db(databaseName);
    database.collection(collectionName).find(query).limit(2).project({title:      1}).each(function(err, doc) {
        if(doc) {
            console.log('Current doc');
            console.log(doc);
        } else {
            client.close(); // Close our connection.
            return false;   // End the each loop.
        }
     });
})
```

1.  现在，使用`node Exercise8.02.js`运行脚本。你应该得到以下输出：

```js
Connected to MongoDB with NodeJS!
Our database connected alright!
Current doc
{ _id: 573a1390f29313caabcd548c, title: 'The Birth of a Nation' }
Current doc
{ _id: 573a1390f29313caabcd5b9a, title: "Hell's Hinges" }
```

在这个练习中，你构建了一个 Node.js 程序，对 MongoDB 执行查询，并将结果返回到控制台。虽然这是一个小步骤，我们可以很容易地在 mongo shell 中完成，但这个脚本将作为更高级和交互式的 Node.js 应用程序的基础。

# 在 Node.js 中的回调和错误处理

所以，我们已经成功打开了与 MongoDB 的连接并运行了一些简单的查询，但可能有一些代码元素看起来不太熟悉；例如，这里的语法：

```js
.each(function(err, doc) {
        if(doc) {
            console.log('Current doc');
            console.log(doc);
        } else {
            client.close(); // Close our connection.
            return false;   // End the each loop.
        }
     });
```

这就是所谓的`MongoClient`，一旦它完成了自己的内部逻辑，它应该执行我们作为第二个参数传递的函数中的代码。第二个参数被称为回调。回调是额外的函数（代码块），作为参数传递给另一个首先执行的函数。

回调允许您指定仅在函数完成后执行的逻辑。我们必须在 Node.js 中使用回调的原因是 Node.js 是异步的，这意味着当我们调用诸如`connect`之类的函数时，它不会阻塞执行。脚本中的下一个内容将被执行。这就是为什么我们使用回调的原因：确保我们的下一步等待连接完成。除了回调之外，还有其他现代模式可以用来替代回调，例如**promises**和**await/async**。但是，考虑到本书的范围，我们将只在本章中涵盖回调，并学习如何处理驱动程序返回的错误。

# Node.js 中的回调

回调通常在视觉上令人困惑且难以概念化；但是，从根本上讲，它们非常简单。回调是作为第二个函数的参数提供的函数，这允许两个函数按顺序运行。

不使用回调函数（或任何其他同步模式），两个函数将在彼此之后立即开始执行。使用驱动程序时，这会创建错误，因为第二个函数可能依赖于第一个函数在开始之前完成。例如，在连接建立之前，您无法查询数据。让我们来看一下回调的分解：

![图 8.4：回调的分解](img/B15507_08_04.jpg)

图 8.4：回调的分解

现在，将此与我们的`find`查询代码进行比较：

![图 8.5：MongoDB 回调的分解](img/B15507_08_05.jpg)

图 8.5：MongoDB 回调的分解

您可以看到，相同的结构存在，只是回调函数的参数不同。您可能想知道我们如何知道在特定回调中使用哪些参数。答案是，我们传递给回调函数的参数由我们提供回调函数的第一个函数确定。也许这是一个令人困惑的句子，但它的意思是：当将函数 fA 作为参数传递给第二个函数 fB 时，fA 的参数由 fB 提供。让我们再次检查我们的实际示例，以确保我们理解这一点：

```js
database.collection(collectionName).find(query).limit(2).project({title: 1}).each   (function(err, doc) {
        if(doc) {
            console.log('Current doc');
            console.log(doc);
        } else {
            client.close(); // Close our connection.
            return false;   // End the each loop.
        }
     });
```

因此，我们的回调函数`function(err, doc) { … }`作为参数提供给驱动程序函数`each`。这意味着`each`将为结果集中的每个文档运行我们的回调函数，为每次执行传递`err`（错误）和`doc`（文档）参数。以下是相同的代码，但添加了一些日志以演示执行顺序：

```js
console.log('This will execute first.')
database.collection(collectionName).find(query).limit(2).project({title: 1}).each   (function(err, doc) {
console.log('This will execute last, once for each document in the result.')
        if(doc) {
        } else {
            client.close(); // Close our connection.
            return false;   // End the each loop.
        }
     });
console.log('This will execute second.');
```

如果我们使用`node 3_Callbacks.js`运行此代码，我们可以在输出中看到执行顺序：

```js
Connected to MongoDB with NodeJS!
This will execute first.
This will execute second.
This will execute last, once for each doc.
This will execute last, once for each doc.
This will execute last, once for each doc.
```

回调有时是复杂的模式，需要熟悉，并且越来越多地被更高级的 Node.js 模式（例如`promises`和`async/await`）所取代。熟悉这些模式的最佳方法是使用它们，因此如果您对它们还不太熟悉，不用担心。

## Node.js 中的基本错误处理

当我们检查回调时，您可能已经注意到我们尚未描述的参数：`err`。在 MongoDB 驱动程序中，大多数在 mongo shell 中可能返回错误的命令也可以在驱动程序中返回错误。在回调的情况下，`err`参数将始终存在；但是，如果没有错误，则`err`的值为`null`。在 NodeJS 中捕获异步代码中的错误的“错误优先”模式是标准做法。

例如，假设您创建了一个应用程序，将用户的电话号码输入客户数据库，两个不同的用户输入相同的电话号码。当您尝试运行插入时，MongoDB 将返回重复键错误。此时，作为 Node.js 应用程序的创建者，您有责任正确处理该错误。要检查查询中的任何错误，我们可以检查`err`是否不为`null`。您可以使用以下语法轻松检查：

```js
database.collection(collectionName).find(query).limit(2).project({title: 1}).each   (function(err, doc) {
        if(err) {
            console.log('Error in query.');
            console.log(err);
            client.close();
            return false;
        }
        else if(doc) {
            console.log('Current doc');
            console.log(doc);
        } else {
            client.close(); // Close our connection.
            return false;   // End the each loop.
        }
     });
```

您可能会注意到，这与我们在使用`each`时检查是否有更多文档时使用的语法相同。类似于我们检查查询的错误，我们的客户端中的`connect`函数也会向我们的`callback`函数提供一个错误，这在运行任何进一步的逻辑之前应该进行检查：

```js
// Open the connection using the .connect function.
client.connect(function(err) {
    if(err) {
        console.log('Error connecting!');
        console.log(err);
        client.close();
    } else {
        // Within the connection block, add a console.log to confirm the           connection
        console.log('Connected to MongoDB with NodeJS!');
        client.close(); // Close our connection at the end.
    }
})
```

注意

在尝试使用参数之前，最好使用回调来检查传递的参数。在`find`命令的情况下，这意味着检查是否有错误并检查是否返回了文档。在针对 MongoDB 编写代码时，最好验证从数据库返回的所有内容，并记录错误以进行调试。

但我们不仅可以在回调中验证代码的准确性。我们还可以检查非回调函数，以确保一切顺利，例如当我们创建我们的`database`对象时：

```js
    const database = client.db(databaseName);
    if(database) {
        console.log('Our database connected alright!');
    }
```

根据您尝试使用 MongoDB 实现的目标，您的错误处理可能像前面的示例那样简单，或者您可能需要更复杂的逻辑。但是，在本章的范围内，我们只会看一下基本的错误处理。

## 练习 8.03：使用 Node.js 驱动程序进行错误处理和回调

在*Exercise 8.02*中，*构建 Node.js 驱动程序查询*，您创建了一个成功连接到 MongoDB 服务器并生成查询结果的脚本。在这个练习中，您将向您的代码添加错误处理——这意味着如果出现任何问题，它可以帮助您识别或修复问题。您将通过修改查询来测试此处理，以使其失败。您可以通过以下步骤在 Node.js 中实现这一点：

1.  创建一个名为`Exercise8.03.js`的新 JavaScript 文件。

1.  为了不必从头开始重写所有内容，将`Exercise8.02.js`的内容复制到您的新脚本中。否则，在新文件中重写连接和查询代码。

1.  在连接回调中，检查`err`参数。如果您有错误，请确保使用`console.log`输出它：

```js
client.connect(function(err) {
    if(err) {
        console.log('Failed to connect.');
        console.log(err);
        return false;
    }
    // Within the connection block, add a console.log to confirm the       connection
    console.log('Connected to MongoDB with NodeJS!');
```

1.  在运行查询之前添加一些错误检查，以确保数据库对象已成功创建。如果您有错误，请使用`console.log`输出它。使用`!`语法来检查某些东西是否不存在：

```js
    const database = client.db(databaseName);
    if(!database) {
        console.log('Database object doesn't exist!');
        return false;
    }
```

1.  在`each`回调中，检查`err`参数，确保每个文档都没有错误地返回：

```js
    database.collection(collectionName).find(query).limit(2).project({title: 1}).each(function(err, doc) {
        if(err) {
            console.log('Query error.');
            console.log(err);
            client.close();
            return false;
        }
        if(doc) {
            console.log('Current doc');
            console.log(doc);
        } else {
            client.close(); // Close our connection.
            return false;   // End the each loop.
        }
     });
```

此时，您的整个代码应该如下所示：

```js
const MongoClient = require('mongodb').MongoClient;
const url = 'mongodb+srv://username:password@server-  fawxo.gcp.mongodb.net/test?retryWrites=true&w=majority';
const client = new MongoClient(url);
const databaseName = "sample_mflix";
const collectionName = "movies";
const query = { genres: { $all: ["Romance"]} };
// Open the connection using the .connect function.
client.connect(function(err) {
    if(err) {
        console.log('Failed to connect.');
        console.log(err);
        return false;
    }
    // Within the connection block, add a console.log to confirm the       connection
    console.log('Connected to MongoDB with NodeJS!');
    const database = client.db(databaseName);
    if(!database) {
        console.log('Database object doesn't exist!');
        return false;
    }
    database.collection(collectionName).find(query).limit(2).project({title:      1}).each(function(err, doc) {
        if(err) {
            console.log('Query error.');
            console.log(err);
            client.close();
            return false;
        }
        if(doc) {
            console.log('Current doc');
            console.log(doc);
        } else {
            client.close(); // Close our connection.
            return false;   // End the each loop.
        }
     });
})
```

1.  在添加错误之前，使用 node `Exercise8.03.js`运行脚本。您应该会得到以下输出：

```js
Connected to MongoDB with NodeJS!
Current doc
{ _id: 573a1390f29313caabcd548c, title: 'The Birth of a Nation' }
Current doc
{ _id: 573a1390f29313caabcd5b9a, title: "Hell's Hinges" }
```

1.  修改查询以确保产生错误：

```js
const query = { genres: { $thisIsNotAnOperator: ["Romance"]} };
```

1.  使用 node `Exercise8.03.js`运行脚本。您应该会得到以下输出：![图 8.6：脚本运行后的输出（为简洁起见进行了截断）](img/B15507_08_06.jpg)

图 8.6：脚本运行后的输出（为简洁起见进行了截断）

在这个练习中，您扩展了您的 Node.js 应用程序，以便在 Node.js 环境中运行 MongoDB 查询时捕获和处理可能遇到的错误。这将使您能够创建更健壮、容错和可扩展的应用程序。

# 高级查询

在上一节中，我们连接到了 MongoDB 服务器，查询了一些数据，输出了它，并处理了我们遇到的任何错误。但是，如果应用程序或脚本只能执行读取操作，那么它的实用性将受到限制。在本节中，我们将在 MongoDB 驱动程序中应用`write`和`update`操作。此外，我们将研究如何使用函数语法为我们的最终应用程序创建可重用的代码块。

## 使用 Node.js 驱动程序插入数据

与 mongo shell 类似，我们可以使用`insertOne`或`insertMany`函数将数据写入我们的集合。这些函数在集合对象上调用。我们需要将单个文档传递给这些函数，或者在`insertMany`的情况下，需要传递文档数组。以下是一个包含如何使用带有回调的`insertOne`和`insertMany`的代码片段。到目前为止，您应该能够认识到这是一个不完整的代码片段。要执行以下代码，您需要添加我们在本章前面学到的基本连接逻辑。现在这应该看起来非常熟悉：

```js
    database.collection(collectionName).insertOne({Hello:      "World"}, function(err, result) {
        // Handle result.
    })
    database.collection(collectionName).insertMany([{Hello: "World"},       {Hello: "Mongo"}], function(err, result) {
        // Handle result.
    })
```

与`find`一样，我们将回调传递给这些函数以处理操作的结果。插入操作将返回一个错误（可能为`null`）和一个结果，其中详细说明了插入操作的执行方式。例如，如果我们要构建在先前练习的结果之上，并记录`insertMany`操作的结果，那么将产生以下输出：

```js
    database.collection(collectionName).insertOne({Hello: "World"},       function(err, result) {
        console.log(result.result);
   client.close();
    })
```

我们可能会在输出中看到一个像*图 8.7*那样的`result`对象。

注意

我们只输出了整个`result`对象的一个子集，其中包含有关我们操作的更多信息。例如，我们正在记录`result.result`，这是整个`result`对象中的一个子文档。这仅适用于本示例的范围。在其他用例中，您可能需要更多关于操作结果的信息：

![图 8.7：显示整个结果对象的子集的输出](img/B15507_08_07.jpg)

图 8.7：显示整个结果对象的子集的输出

## 使用 Node.js 驱动程序更新和删除数据

使用驱动程序更新和删除文档遵循与`insert`函数相同的模式，其中`collection`对象通过回调传递，检查错误，并分析操作的结果。所有这些函数都将返回一个结果文档。但是，在这三个操作之间，结果文档中包含的格式和信息可能会有所不同。让我们看一些示例。

以下是一些示例代码的示例（也建立在我们之前的连接代码之上），用于更新文档。我们可以使用`updateOne`或`updateMany`：

```js
    database.collection(collectionName).updateOne({Hello: "World"}, {$set: {Hello       : "Earth"}}, function(err, result) {
        console.log(result.modifiedCount);
        client.close();
    })
```

如果我们运行这段代码，我们的输出结果可能如下所示：

```js
Connected to MongoDB with NodeJS!
1
```

现在，让我们看一个删除文档的示例。与我们的其他函数一样，我们可以使用`deleteOne`或`deleteMany`。请记住，此代码片段作为我们为*Exercise 8.03*创建的较大代码的一部分存在，*Node.js 驱动程序中的错误处理和回调*：

```js
    database.collection(collectionName).deleteOne({Hello: "Earth"}, function(err, result) {
        console.log(result.deletedCount);
        client.close();
    })
```

如果我们运行这段代码，我们的输出将如下所示：

```js
Connected to MongoDB with NodeJS!
1
```

正如您所看到的，所有这些操作都遵循类似的模式，并且在结构上非常接近您将发送到 mongo shell 的相同命令。主要区别在于回调，我们可以在操作结果上运行自定义逻辑。

## 编写可重用的函数

到目前为止，在我们的示例和练习中，我们总是执行单个操作并输出结果。但是，在更大，更复杂的应用程序中，您将希望在同一程序中运行许多不同的操作，具体取决于上下文。例如，在您的应用程序中，您可能希望多次运行相同的查询并比较各自的结果，或者您可能希望根据第一个查询的输出修改第二个查询。

这就是我们将创建自己的函数的地方。您已经编写了一些函数用作回调，但在这种情况下，我们将编写可以随时调用的函数，无论是用于实用程序还是保持代码清晰和分离。让我们看一个例子。

让我们通过以下代码片段更好地理解这一点，该代码片段运行了三个非常相似的查询。这些查询之间唯一的区别是每个查询中的一个参数（评分）：

```js
database.collection(collectionName).find({name: "Matthew"}).each(function(err,   doc) {});
database.collection(collectionName).find({name: "Mark"}).each(function(err, doc)   {});
database.collection(collectionName).find({name: "Luke"}).each(function(err, doc)   {})
```

让我们尝试用一个函数简化和清理这段代码。我们使用与变量相同的语法声明一个新函数。因为这个函数不会改变，我们可以将它声明为`const`。对于函数的值，我们可以使用我们在之前的示例中（本章早期的*回调*部分的示例）已经熟悉的语法：

```js
const findByName = function(name) {

}
```

现在，让我们在花括号之间为这个函数添加逻辑：

```js
const findByName = function(name) {
    database.collection(collectionName).find({name:       name}).each(function(err, doc) {})
}
```

但是有些地方不太对。我们在创建数据库对象之前引用了数据库对象。我们将不得不将该对象作为参数传递给这个函数，所以让我们调整我们的函数来做到这一点：

```js
const findByName = function(name, database) {
    database.collection(collectionName).find({name: name}).each(function(err,       doc) {})
}
```

现在，我们可以用三个函数调用来替换我们的三个查询：

```js
const findByName = function(name, database) {
    database.collection(collectionName).find({name: name}).each(function(err, doc       ) {})
}
findByName("Matthew", database);
findByName("Mark", database);
findByName("Luke", database);
```

在本章中，为了简单起见，我们不会过多地讨论创建模块化、功能性的代码。但是，如果您想进一步改进这段代码，您可以使用数组和`for`循环来为每个值运行函数，而不必调用它三次。

## 练习 8.04：使用 Node.js 驱动程序更新数据

考虑*介绍*部分的情景，您已经从起点取得了相当大的进展。您的最终应用程序需要能够通过运行更新操作向电影添加投票。但是，您还没有准备好添加这个逻辑。不过，为了证明您能够做到这一点，编写一个脚本，更新数据库中的几个不同文档，并创建一个可重用的函数来实现这一点。在这个练习中，您需要更新`chapter8_Exercise4`集合中的以下名称。您将使用这个唯一的集合来确保在更新期间不会损坏其他活动的数据：

*Ned Stark to Greg Stark, Robb Stark to Bob Stark, and Bran Stark to Brad Stark.*

您可以通过执行以下步骤在 Node.js 中实现这一点：

1.  首先，确保正确的文档存在以进行更新。直接使用 mongo shell 连接到服务器，并执行以下代码片段来检查这些文档：

```js
db.chapter8_Exercise4.find({ $or: [{name: "Ned Stark"}, {name: "Robb Stark"}, {name: "Bran Stark"}]});
```

1.  如果前面查询的结果为空，请使用此片段添加要更新的文档：

```js
db.chapter8_Exercise4.insert([{name: "Ned Stark"}, {name: "Bran Stark"}, {name: "Robb Stark"}]);
```

1.  现在，要创建脚本，请退出 mongo shell 连接，并创建一个名为`Exercise8.04.js`的新 JavaScript 文件。这样您就不必从头开始重写所有内容，只需将`Exercise8.03.js`的内容复制到新脚本中。否则，请在新文件中重写连接代码。如果您从*Exercise 8.03*，*使用 Node.js 驱动程序处理错误和回调*中复制了代码，则删除查找查询的代码。

1.  将集合从电影更改为`chapter8_Exercise4`：

```js
const collectionName = "chapter8_Exercise4";
```

1.  在脚本开始之前，在连接之前，创建一个名为`updateName`的新函数。这个函数将以数据库对象、客户端对象以及`oldName`和`newName`作为参数：

```js
const updateName = function(client, database, oldName, newName) {
}
```

1.  在`updateName`函数中，添加运行更新命令的代码，该命令将更新包含名为`oldName`的字段的文档，并将该值更新为`newName`：

```js
const updateName = function(client, database, oldName, newName) {
    database.collection(collectionName).updateOne({name: oldName}, {$set: {name: newName}}, function(err, result) {
        if(err) {
            console.log('Error updating');
            console.log(err);
            client.close();
            return false;
        }
        console.log('Updated documents #:');
        console.log(result.modifiedCount);
        client.close();
    })
};
```

1.  现在，在连接回调中，运行您的新函数三次，分别为要更新的三个名称运行一次：

```js
    updateName(client, database, "Ned Stark", "Greg Stark");
    updateName(client, database, "Robb Stark", "Bob Stark");
    updateName(client, database, "Bran Stark", "Brad Stark");
```

1.  此时，您的整个代码应该如下所示：

```js
const MongoClient = require('mongodb').MongoClient;
const url = 'mongodb+srv://mike:password@myAtlas-fawxo.gcp.mongodb.net/test?retryWrites=true&w=majority';
const client = new MongoClient(url);
const databaseName = "sample_mflix";
const collectionName = "chapter8_Exercise4";
const updateName = function(client, database, oldName, newName) {
    database.collection(collectionName).updateOne({name: oldName}, {$set: {name: newName}}, function(err, result) {
        if(err) {
            console.log('Error updating');
            console.log(err);
            client.close();
            return false;
        }
        console.log('Updated documents #:');
        console.log(result.modifiedCount);
        client.close();
    })
};
// Open the connection using the .connect function.
client.connect(function(err) {
    if(err) {
        console.log('Failed to connect.');
        console.log(err);
        return false;
    }
    // Within the connection block, add a console.log to confirm the connection
    console.log('Connected to MongoDB with NodeJS!');
    const database = client.db(databaseName);
    if(!database) {
        console.log('Database object doesn't exist!');
        return false;
    }

    updateName(client, database, "Ned Stark", "Greg Stark");
    updateName(client, database, "Robb Stark", "Bob Stark");
    updateName(client, database, "Bran Stark", "Brad Stark");
})
```

1.  使用`node Exercise8.04.js`运行脚本。您应该会得到以下输出：

```js
Connected to MongoDB with NodeJS!
Updated documents #:
1
Updated documents #:
1
Updated documents #:
1
```

在过去的四个部分中，您已经学会了如何创建一个连接到 MongoDB 的 Node.js 脚本，运行易于使用的函数进行查询，并处理我们可能遇到的任何错误。这为您搭建了一个基础，可以用它来构建许多脚本，以使用您的 MongoDB 数据库执行复杂的逻辑。然而，在我们迄今为止的示例中，我们的查询参数总是硬编码到我们的脚本中，这意味着我们的每个脚本只能满足特定的用例。

这并不理想。像 Node.js 驱动程序这样的强大之处之一是能够拥有一个解决大量问题的单个应用程序。为了扩大我们脚本的范围，我们将接受用户输入来创建动态查询，能够解决用户的问题，而无需重写和分发我们程序的新版本。在本节中，我们将学习如何接受用户输入、处理它，并从中构建动态查询。

注意

在大多数大型、生产就绪的应用程序中，用户输入将以**图形用户界面**（**GUI**）的形式出现。这些 GUI 将简单的用户选择转换为复杂的、相关的查询。然而，构建 GUI 是非常棘手的，超出了本书的范围。

## 从命令行读取输入

在本节中，我们将从命令行获取输入。幸运的是，Node.js 为我们提供了一些简单的方法来从命令行读取输入并在我们的代码中使用它。Node.js 提供了一个名为 `readline` 的模块，它允许我们向用户请求输入、接受输入，然后使用它。您可以通过在文件顶部添加以下行来将 `readline` 加载到您的脚本中。在使用 `readline` 时，您必须始终创建一个接口：

```js
const readline = require('readline');
const interface = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
});
```

现在，我们可以要求用户输入一些内容。 `readline` 为我们提供了多种处理输入的方式。然而，现在最简单的方法是使用 `question` 函数，就像这里的例子一样：

```js
interface.question('Hello, what is your name? ', (input) => {
    console.log(`Hello, ${input}`);
    interface.close();
  });
```

注意

`${input}` 语法允许我们在字符串中嵌入一个变量。在使用时，请确保使用反引号，```js (if you're not sure where to find this, on a standard QWERTY keyboard, it shares a key with the ~ symbol, to the left of the `1` key.)

If we run this example, we will get an output resembling this:

```

Chapter_8> node example.js

你好，你叫什么名字？ 迈克尔

你好，迈克尔

```js

If you want to create a longer prompt, it is better to use `console.log` to output the bulk of your output, and then provide just a smaller question for the `readline`. For example, say we have a long message that we send before we ask for user input. We can define it as a variable and log it before we ask our question:

```

const question = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum?"

interface.question(question, (input) => {

console.log(`你好， ${input}`);

interface.close();

});

```js

In this way, it is easy to modify and reuse our messages across multiple inputs.

Note

There are many different libraries and modules for handling inputs in Node.js. However, to keep things simple, we'll use `readline` in this chapter.

## Creating an Interactive Loop

So, we have an easy way of asking a user a question and accepting some input from them. However, our application won't be very useful if we have to run it from the command line every time we want to use it. It would be much more useful if we could run the program once, and execute many runs of it based on different inputs.

To accomplish this, we can create an interactive loop, meaning the application will keep asking for input until an exit condition is met. To make sure we keep looping, we can place our prompt in a function that calls itself, which will keep running the code inside its block until the exit condition stated becomes `true`. This will provide a much better experience for users of our code. Here is an example of an interactive loop using our aforementioned `readline`:

```

const askName = function() {

interface.question("你好，你叫什么名字？", (input) => {

if(input === "exit") {

return interface.close(); // 将终止循环。

}

console.log(`你好，${input}`);

askName();

});

}

askName(); // 第一次运行。

```js

Note the exit condition here:

```

if(input === "exit") {

return interface.close(); // 将终止循环。

}

```js

It is vitally important to ensure that, in any loop, you have an exit condition, as this allows users to quit the application. Otherwise, they will be stuck in a loop forever, and it may consume the resources of your computer.

Note

When writing loops in your code, it is possible that you could accidentally create an infinite loop without an exit condition. If this does happen, you may have to kill your shell or Terminal. You can try *Ctrl+C*, or *Cmd+C* on a macOS, to exit.

If you were to run the preceding example, you would be able to answer the question multiple times before exiting; for example:

```

Chapter_8> node examples.js

你好，你叫什么名字？迈克

你好，迈克

你好，你叫什么名字？约翰

你好，约翰

你好，你叫什么名字？埃德

你好，埃德

你好，你叫什么名字？退出

```js

## Exercise 8.05: Handling Inputs in Node.js

For this exercise, you're going to create a small Node.js application that allows you to ask users their name. You can think of this as a rudimentary login system. This application should run in an interactive loop; the options for the user are as follows:

*   `login` (*Ask and store the user's name*)
*   `who` (*Output the user's name*)
*   `exit` (*End the application*)

Create this application by performing the following steps:

1.  Create a new JavaScript file called `Exercise8.05.js`.
2.  Import the `readline` module:

```

const readline = require('readline');

const interface = readline.createInterface({

input: process.stdin,

output: process.stdout,

});

```js

3.  Define the choice and user variables.
4.  Now, define a new function called `login` that takes user as parameters. The function first asks for a user and stores it in a variable:

```

const login = function() {

interface.question("你好，你叫什么名字？", (name) => {

user = name;

prompt();

});

}

```js

5.  Create a new function called `who` that outputs `user`:

```

const who = function () {

console.log(`用户是 ${user}`);

prompt();

}

```js

6.  Create an input loop with the condition that choice is not equal to exit:

```

const prompt = function() {

interface.question("登录，who 或 退出？", (input) => {

if(input === "exit") {

return interface.close(); // 将终止循环。

}

提示();

});

}

```js

7.  After that, use the if keyword to check whether their choice matches "`login`". If we find a match, then run the `login` function:

```

if(input === "login") {

login();

}

```js

8.  Then, use the if keyword to check whether their choice matches "`who`". If we find a match, then print out the `user` variable:

```

if(input === "who") {

who();

}

```js

Your final code should look something like this:

```

const readline = require('readline');

const interface = readline.createInterface({

input: process.stdin,

output: process.stdout,

});

var choice;

var user;

var cinema;

const login = function() {

interface.question("你好，你叫什么名字？", (name) => {

user = name;

prompt();

});

}

const who = function () {

console.log(`用户是 ${user}`)

prompt();

}

const prompt = function() {

interface.question("登录，who 或 退出？", (input) => {

if(input === "exit") {

return interface.close(); // 将终止循环。

}

if(input === "login") {

login();

}

if(input === "who") {

who();

}

});

}

prompt();

```js

9.  Run the code using `node Exercise8.05.js` and enter some input. Now, you should be able to interact with the application. The following is an example:

```

Chapter_8> node .\Exercise8.06.js

登录，who 或 退出？登录

你好，你叫什么名字？迈克尔

登录，who 或 退出？who

用户是 Michael

登录，是谁还是退出？退出

```

在这个练习中，您创建了一个基本的交互式应用程序，使用 Node.js 让用户从三个输入中进行选择，并相应地输出结果。

## 活动 8.01：创建一个简单的 Node.js 应用程序

您已被一家电影公司聘请，创建一个应用程序，允许客户列出所选类别中评分最高的电影。客户应该能够提供一个类别，并在命名的命令行列表中提供响应。他们还需要提供他们最喜欢的电影的详细信息，以便在收藏字段中捕获。最后，完成所有这些后，客户应该能够`退出`应用程序，如下所示：

+   `"列表"`：询问用户一个流派，然后查询该流派中排名前五的电影，输出`ID`、`标题`和`favourite`字段。

+   `"favourite"`：询问用户一个电影 ID，然后更新该电影的收藏字段。

+   `"退出"`：退出交互循环和应用程序。

此活动旨在创建一个小型的 Node.js 应用程序，向用户公开一个交互式输入循环。在此循环中，用户可以通过流派查询数据库中的信息，并通过 ID 更新记录。您还需要确保处理可能出现的用户输入错误。

您可以通过多种方式完成此目标，但请记住我们在本章中学到的内容，并尝试创建简单、易于使用的代码。

以下高级步骤将帮助您完成此任务：

1.  导入`readline`和 MongoDB 库。

1.  创建您的`readline`接口。

1.  声明您将需要的任何变量。

1.  创建一个名为列表的函数，它将为给定流派获取排名前五的最高评分电影，返回`标题`、`收藏`和`ID`字段。

注意

您将需要在此函数中询问类别。查看*练习 8.05*，*在 Node.js 中处理输入*中的登录方法，以获取更多信息。

1.  创建一个名为`favourite`的函数，它将通过标题更新文档，并向文档添加一个名为`favourite`的键，其值为`true`。（提示：在此函数中，您将需要使用与列表函数相同的方法询问标题。）

1.  创建 MongoDB 连接、数据库和集合。

1.  基于用户输入创建一个交互式 while 循环。如果您不确定如何做到这一点，请参考我们在*练习 8.05*，*在 Node.js 中处理输入*中的提示函数。

1.  在交互循环中，使用 if 条件来检查输入。如果找到有效输入，则运行相关函数。

1.  请记住，您需要通过每个函数传递数据库和客户端对象，包括每次调用`prompt()`。要测试您的输出，请运行以下命令：

`列表`

`恐怖`

`favourite`

`列表`

`退出`

预期输出如下：

注意

您可能会注意到输出中标题`Nosferatu`出现了两次。如果查看`_id`值，您会发现这实际上是两部具有相同标题的不同电影。在 MongoDB 中，您可能有许多不同的文档，它们在字段中共享相同的值。

![图 8.8：最终输出（为简洁起见截断）](img/B15507_08_08.jpg)

图 8.8：最终输出（为简洁起见截断）

注意

此活动的解决方案可通过此链接找到。

# 总结

在本章中，我们已经介绍了创建使用 Node.js 驱动程序的 MongoDB 应用程序所必需的基本概念。使用这些基础知识，可以创建大量脚本来执行对数据库的查询和操作。我们甚至学会了处理错误和创建交互式应用程序。

尽管您可能不需要在日常工作职责中编写或阅读这些应用程序，但对这些应用程序是如何构建的有深入的理解，可以让您独特地了解 MongoDB 开发以及您的同行可能如何与您的 MongoDB 数据交互。

然而，如果你想增加对于 MongoDB 的 Node.js 驱动的专业知识，这只是个开始。有许多不同的模式、库和最佳实践可以用来开发针对 MongoDB 的 Node.js 应用程序。这只是你 Node.js 之旅的开始。

在下一章中，我们将深入探讨如何提高 MongoDB 交互的性能，并创建高效的索引来加快查询速度。我们还将介绍另一个有用的功能，即使用 `explain` 并且如何最好地解释其输出。
