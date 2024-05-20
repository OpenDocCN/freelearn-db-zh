# 前言

即使在今天，仍然很常见说计算机科学是一个年轻和新的领域。然而，当我们观察其他领域时，这种说法变得有些矛盾。与其他领域不同，计算机科学是一个不断以超出正常速度发展的学科。我敢说，计算机科学现在已经为医学和工程等其他领域的发展设定了进化的路径。在这种情况下，作为计算机科学学科领域的数据库系统不仅促进了其他领域的增长，而且还充分利用了许多技术领域的进化和进步，如计算机网络和计算机存储。

从形式上来说，自 20 世纪 60 年代以来，数据库系统一直是一个活跃的研究课题。从那时起，我们经历了几代，IT 行业中出现了一些大名鼎鼎的人物，并开始主导市场的趋势。

在 2000 年代，随着世界互联网接入的增长，形成了一个新的网络流量模式，社交网络的蓬勃发展，NoSQL 这个术语变得很普遍。许多人认为这是一个矛盾和有争议的话题，有些人认为这是一代新技术，是对过去十年经历的所有变化的回应。

MongoDB 就是其中之一。诞生于 21 世纪初，它成为了世界上最受欢迎的 NoSQL 数据库。不仅是世界上最受欢迎的数据库，自 2015 年 2 月以来，根据 DB-Engines 排名([`db-engines.com/en/`](http://db-engines.com/en/))，MongoDB 成为了第四受欢迎的数据库系统，超过了著名的 PostgreSQL 数据库。

然而，流行度不应该与采用混淆。尽管 DB-Engines 排名显示 MongoDB 在搜索引擎（如 Google）上负责一些流量，有工作搜索活动，并且在社交媒体上有相当大的活动，但我们无法确定有多少应用程序正在使用 MongoDB 作为数据源。事实上，这不仅仅是 MongoDB 的问题，而是每一种 NoSQL 技术都存在的问题。

好消息是采用 MongoDB 并不是一个很艰难的决定。它是开源的，所以你可以免费从 MongoDB Inc. ([`www.mongodb.com`](https://www.mongodb.com))下载，那里有广泛的文档。你也可以依靠一个庞大且不断增长的社区，他们像你一样，总是在书籍、博客和论坛上寻找新的东西；分享知识和发现；并合作推动 MongoDB 的发展。

《MongoDB 数据建模》的撰写目的是为您提供另一个研究和参考来源。在其中，我们将介绍用于创建可扩展数据模型的技术和模式。我们将介绍基本的数据库建模概念，并提供一个专注于 MongoDB 建模的概述。最后，您将看到一个实际的逐步示例，对一个现实问题进行建模。

主要来说，有一些 MongoDB 背景的数据库管理员将受益于《MongoDB 数据建模》。然而，从开发人员到所有下载了 MongoDB 的好奇者都会从中受益。

本书侧重于 MongoDB 3.0 版本。MongoDB 3.0 版本是社区期待已久的版本，被 MongoDB Inc.认为是迄今为止最重要的发布。这是因为在这个版本中，我们被介绍了新的、高度灵活的存储架构 WiredTiger。性能和可扩展性的增强意图加强 MongoDB 在数据库系统技术中的重要性，并将其定位为现代应用程序的标准数据库。

# 本书涵盖的内容

第一章 *介绍数据建模*，向您介绍了基本的数据建模概念和 NoSQL 领域。

第二章，“使用 MongoDB 进行数据建模”，为您提供了 MongoDB 的面向文档的架构概述，并向您展示了文档及其特征以及如何构建它。

第三章，“查询文档”，通过 MongoDB API 引导您查询文档，并向您展示查询如何影响我们的数据建模过程。

第四章，“索引”，解释了如何通过使用索引来改善查询的执行，并因此改变我们建模数据的方式。

第五章，“优化查询”，帮助您使用 MongoDB 的本机工具来优化您的查询。

第六章，“管理数据”，侧重于数据的维护。这将教会你在开始数据建模之前查看数据操作和管理的重要性。

第七章，“扩展”，向您展示了 MongoDB 的自动共享特性有多强大，以及我们如何认为我们的数据模型是分布式的。

第八章，“使用 MongoDB 进行日志记录和实时分析”，带您了解了一个真实问题示例的模式设计。

# 您需要为本书准备什么

要成功理解本书的每一章，您需要访问 MongoDB 3.0 实例。

您可以选择在何处以及如何运行它。我们知道有许多方法可以做到这一点。所以，选择一个。

要执行查询和命令，我建议您在 mongo shell 上执行此操作。每次我在 mongo shell 之外执行此操作时，我都会警告您。

在第八章，“使用 MongoDB 进行日志记录和实时分析”，您需要在计算机上安装 Node.js，并且应该可以访问您的 MongoDB 实例。

# 这本书是为谁准备的

本书假定您已经与 MongoDB 有过初次接触，并且具有一些 JavaScript 经验。本书适用于数据库管理员、开发人员或任何希望了解一些数据建模概念以及它们如何适用于 MongoDB 世界的人。它不会教您 JavaScript 或如何在计算机上安装 MongoDB。如果您是 MongoDB 初学者，您可以找到一些很好的 Packt Publishing 图书，这些图书将帮助您获得足够的经验，以更好地理解本书。

# 约定

在本书中，您将找到许多区分不同类型信息的文本样式。以下是这些样式的一些示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：“我们可以将关系存储在`Group`文档中。”

代码块设置如下：

```go
  collection.update({resource: resource, date: today},
    {$inc : {daily: 1}}, {upsert: true},
    function(error, result){
      assert.equal(error, null);
      assert.equal(1, result.result.n);
      console.log("Daily Hit logged");
      callback(result);
  });
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目将以粗体显示：

```go
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
```

任何命令行输入或输出都以以下方式编写：

```go
db.customers.find(
{"username": "johnclay"},
{_id: 1, username: 1, details: 1}
)

```

**新术语**和**重要单词**以粗体显示。

### 注意

警告或重要说明以这样的框出现。

### 提示

提示和技巧看起来像这样。

# 读者反馈

我们的读者的反馈总是受欢迎的。让我们知道您对本书的看法 - 您喜欢或不喜欢的内容。读者的反馈对我们很重要，因为它可以帮助我们开发您真正能够从中获益的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在消息主题中提及书名。

如果您对某个主题有专业知识，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的自豪所有者，我们有一些事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载示例代码文件，这适用于您购买的所有 Packt Publishing 图书。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便文件直接发送到您的邮箱。

## 勘误表

尽管我们已经非常注意确保内容的准确性，但错误是难免的。如果您在我们的书籍中发现错误——也许是文本或代码中的错误——我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击**勘误提交表格**链接，并输入您的勘误详情。一旦您的勘误被验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误部分的任何现有勘误列表中。

要查看先前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将出现在**勘误表**部分下。

## 盗版

互联网上盗版受版权保护的材料是一个持续存在的问题，涉及各种媒体。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并附上疑似盗版材料的链接。

我们感谢您帮助我们保护作者和为您提供有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
