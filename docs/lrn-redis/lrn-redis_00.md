# 前言

《学习 Redis》旨在成为开发人员、架构师、解决方案提供商、顾问、工程师以及计划学习、设计和构建企业解决方案并寻找一种灵活快速且能够扩展其能力的内存数据存储的指南和手册。

本书首先介绍了 NoSQL 不断发展的格局，通过易于理解的示例探索命令，然后在一些示例应用中使用 Redis 作为支撑。书的后面部分着重于管理 Redis 以提高性能和可伸缩性。

本书涵盖了设计和创建快速、灵活和并发应用的核心概念，但不是 Redis 官方文档指南的替代品。 

# 本书涵盖的内容

第一章，“NoSQL 简介”，涵盖了 NoSQL 的生态系统。它讨论了 NoSQL 格局的演变，并介绍了各种类型的 NoSQL 及其特点。

第二章，“开始使用 Redis”，涉及到 Redis 的世界。它还涵盖了在各种平台上安装 Redis 以及在 Java 中运行示例程序连接到 Redis 等领域。

第三章，“Redis 中的数据结构和通信协议”，涵盖了 Redis 中可用的数据结构和通信协议。它还涵盖了用户可以执行并感受其使用的示例。通过本章结束时，您应该对 Redis 的能力有了基本的了解。

第四章，“Redis 服务器中的功能”，从学习命令到 Redis 的各种内置功能。这些功能包括 Redis 中的消息传递、事务以及管道功能，它们之间有所不同。本章还向用户介绍了一种名为 LUA 的脚本语言。

第五章，“Redis 中的数据处理”，着重于 Redis 的深度数据处理能力。这包括主从安排、数据存储方式以及其提供的各种持久化数据选项。

第六章，“Web 应用中的 Redis”，是关于在 Web 应用中定位 Redis 的内容。为了增加趣味性，书中提供了一些示例应用，您可以从中获取有关 Redis 可用的广泛用例的想法。

第七章，“业务应用中的 Redis”，是关于在业务应用中定位 Redis 的内容。为了进一步扩展其在企业解决方案设计领域中的适用性，书中解释了一些示例应用，您可以从中看到其多功能性。

第八章，“集群”，讨论了集群能力，最终用户如何利用 Redis 中的各种集群模式，并相应地在其解决方案中使用这些模式。

第九章，“维护 Redis”，是关于在生产环境中维护 Redis 的内容。

# 本书需要什么

本书需要以下软件：

+   Redis

+   JDK 1.7

+   Jedis（Redis 的 Java 客户端）

+   Eclipse，用于开发的集成开发环境

# 本书适合对象

本书适用于开发人员、架构师、解决方案提供商、顾问和工程师。主要需要 Java 知识，但也可以被任何有一点编程背景的人理解。

除此之外，还有关于如何设计解决方案并在生产中维护它们的信息，不需要编码技能。

# 约定

在本书中，您会发现一些区分不同信息类型的文本样式。以下是这些样式的一些示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："以下代码是新的 Hello World 程序，现在称为 `HelloWorld2`："

代码块设置如下：

```go
package org.learningredis.chapter.two;

public class Helloworld2  {
  JedisWrapper jedisWrapper = null;
  public Helloworld2() {
    jedisWrapper = new JedisWrapper();
  }

  private void test() {
    jedisWrapper.set("MSG", "Hello world 2 ");

    String result = jedisWrapper.get("MSG");
    System.out.println("MSG : " + result);
  }

  public static void main(String[] args) {
    Helloworld2 helloworld2 = new Helloworld2();
    helloworld2.test();
  }
}
```

**新术语** 和 **重要单词** 以粗体显示。例如，屏幕上显示的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中："请注意命令提示符上显示的最后一行：**服务器现在已准备好在端口 6379 上接受连接**"。

### 注意

警告或重要说明会以这样的方式显示在一个框中。

### 提示

提示和技巧会以这样的方式显示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者的反馈对我们很重要，因为它有助于我们开发您真正能充分利用的标题。

要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在主题中提及书名。

如果您在某个专题上有专业知识，并且有兴趣撰写或为一本书做出贡献，请参阅我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经是 Packt 图书的自豪所有者，我们有很多事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从 [`www.packtpub.com`](http://www.packtpub.com) 的帐户中下载您购买的所有 Packt Publishing 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误确实会发生。如果您在我们的书中发现错误——也许是文本或代码中的错误——我们将不胜感激，如果您能向我们报告。通过这样做，您可以帮助其他读者避免挫折，并帮助我们改进本书的后续版本。如果您发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击 **勘误提交表格** 链接，并输入您的勘误详情。一旦您的勘误经过验证，您的提交将被接受，并且勘误将被上传到我们的网站或添加到该标题的勘误列表下的勘误部分。

要查看先前提交的勘误，请转到 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索框中输入书名。所需信息将显示在 **勘误** 部分下。

## 盗版

互联网上的盗版行为是跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可。如果您在互联网上发现我们作品的任何非法副本，请立即向我们提供位置地址或网站名称，以便我们采取补救措施。

请通过链接 `<copyright@packtpub.com>` 与我们联系，提供涉嫌盗版材料的链接。

我们感谢您帮助保护我们的作者和我们提供有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，可以通过 `<questions@packtpub.com>` 与我们联系，我们将尽力解决问题。