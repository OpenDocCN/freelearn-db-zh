# 前言

MongoDB 是一种面向文档的领先 NoSQL 数据库，提供线性可扩展性，因此成为高容量、高性能系统在所有业务领域的良好竞争者。它在易用性、高性能和丰富功能方面胜过大多数 NoSQL 解决方案。

本书提供了详细的配方，描述了如何使用 MongoDB 的不同功能。这些配方涵盖了从设置 MongoDB、了解其编程语言 API 和监控和管理，到一些高级主题，如云部署、与 Hadoop 集成，以及一些用于 MongoDB 的开源和专有工具。配方格式以简洁、可操作的形式呈现信息；这使您可以参考配方，以解决并了解手头的用例的详细信息，而无需阅读整本书。

# 本书涵盖的内容

第一章，“安装和启动服务器”，全都是关于启动 MongoDB。它将演示如何以独立模式、副本集模式和分片模式启动服务器，使用命令行或配置文件提供的启动选项。

第二章，“命令行操作和索引”，有简单的配方，用于在 Mongo shell 中执行 CRUD 操作，并在 shell 中创建各种类型的索引。

第三章，“编程语言驱动程序”，讨论了编程语言 API。虽然 Mongo 支持多种语言，但我们只会讨论如何使用驱动程序仅从 Java 和 Python 程序连接到 MongoDB 服务器。本章还探讨了 MongoDB 的线协议，用于服务器和编程语言客户端之间的通信。

第四章，“管理”，包含了许多用于管理或 MongoDB 部署的配方。本章涵盖了许多经常使用的管理任务，如查看集合和数据库的统计信息，查看和终止长时间运行的操作以及其他副本集和分片相关的管理。

第五章，“高级操作”，是第二章的延伸，我们将看一些稍微高级的功能，如实现服务器端脚本、地理空间搜索、GridFS、全文搜索，以及如何将 MongoDB 与外部全文搜索引擎集成。

第六章，“监控和备份”，告诉您有关管理和一些基本监控的所有内容。然而，MongoDB 提供了一流的监控和实时备份服务，MongoDB 监控服务（MMS）。在本章中，我们将看一些使用 MMS 进行监控和备份的配方。

第七章，“在云上部署 MongoDB”，涵盖了使用 MongoDB 服务提供商进行云部署的配方。我们将在 AWS 云上设置自己的 MongoDB 服务器，以及在 Docker 容器中运行 MongoDB。

第八章，“与 Hadoop 集成”，涵盖了将 MongoDB 与 Hadoop 集成的配方，以使用 Hadoop MapReduce API 在 MongoDB 数据文件中运行 MapReduce 作业并将结果写入其中。我们还将看到如何使用 AWS EMR 在云上运行我们的 MapReduce 作业，使用亚马逊的 Hadoop 集群 EMR 和 mongo-hadoop 连接器。

第九章，*开源和专有工具*，介绍了使用围绕 MongoDB 构建的框架和产品来提高开发人员的生产力，或者简化使用 Mongo 的一些日常工作。除非明确说明，本章中将要查看的产品/框架都是开源的。

附录，*参考概念*，为您提供了有关写入关注和读取偏好的一些额外信息。

# 您需要什么来阅读本书

用于尝试配方的 MongoDB 版本是 3.0.2。这些配方也适用于版本 2.6.*x*。如果有特定于版本 2.6.*x*的特殊功能，将在配方中明确说明。除非明确说明，所有命令都应在 Ubuntu Linux 上执行。

涉及 Java 编程的示例已在 Java 版本 1.7 上进行了测试和运行，Python 代码则使用 Python v2.7 运行（与 Python 3 兼容）。对于 MongoDB 驱动程序，您可以选择使用最新可用版本。

这些是相当常见的软件类型，它们的最低版本在不同的配方中使用。本书中的所有配方都将提到完成它所需的软件及其各自的版本。一些配方需要在 Windows 系统上进行测试，而另一些需要在 Linux 上进行测试。

# 这本书是为谁准备的

这本书是为对了解 MongoDB 并将其用作高性能和可扩展数据存储的管理员和开发人员设计的。它也适用于那些了解 MongoDB 基础知识并希望扩展知识的人。本书的受众预期至少具有一些 MongoDB 基础知识。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些这些样式的示例，以及它们的含义解释。

在本书中，您将找到一些区分不同信息类型的文本样式。以下是一些这些样式的示例和它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下："创建`/data/mongo/db 目录`（或您选择的任何目录）"。

代码块设置如下：

```go
   import com.mongodb.DB;
   import com.mongodb.DBCollection;
   import com.mongodb.DBObject;
   import com.mongodb.MongoClient;
```

任何命令行输入或输出都按如下方式编写：

```go
$ sudo apt-get install default-jdk

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中："由于我们想要启动一个免费的微实例，请在左侧勾选**仅免费层**复选框"。

### 注意

警告或重要提示会以以下方式显示在框中。

### 提示

提示和技巧会以这种方式出现。

# 读者反馈

我们的读者反馈总是受欢迎的。让我们知道你对这本书的想法——你喜欢什么，或者可能不喜欢什么。读者的反馈对我们开发能让你真正受益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在消息主题中提及书名。

如果您在某个专题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经是 Packt 书籍的自豪所有者，我们有很多事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误，我们将不胜感激地希望您向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表格**链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站上，或者添加到该标题的勘误列表中的任何现有勘误下的勘误部分。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有的勘误。

## 盗版

互联网上盗版版权材料是所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并附上涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值的内容的能力方面的帮助。

## 问题

如果您在书籍的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。