# 第三章：服务器和客户端

概述

本章介绍了 MongoDB Atlas Cloud 服务的网络和数据库访问安全性。您将了解 MongoDB 客户端以及如何连接客户端到云数据库以运行 MongoDB 命令。您将使用 Atlas Cloud 安全配置创建和管理用户身份验证和授权，并为 MongoDB 数据库创建用户帐户。连接到 MongoDB 数据库后，您将探索用于 MongoDB 服务器命令的 Compass GUI 客户端。

# 介绍

我们已经在云中探索了 MongoDB 数据库的基础知识，并了解了 MongoDB 与其他数据库的不同之处。*第二章*，*文档和数据类型*解释了 MongoDB 中使用的数据结构。到目前为止，您已经知道如何连接到您的 MongoDB Atlas 控制台，以及如何使用数据浏览器浏览数据库。在本章中，您将继续探索 MongoDB 的世界，并连接和访问新的 MongoDB 数据库，发现其内部架构和命令。

在当今世界，互联网和云计算是现有和未来应用程序制定规则的主要驱动力。到目前为止，我们已经了解到 MongoDB Atlas 是 MongoDB 的强大云版本，为客户提供性能、安全性和灵活性。虽然云基础设施为用户提供了许多好处，但也增加了与存储在云中的数据相关的安全风险。网络安全事件经常在新闻中出现。2013 年，塔吉特公司成为大规模网络攻击的受害者，超过 1 亿客户的个人数据被盗。

MongoDB Atlas 服务的一个优势是许多安全功能默认启用，从而防止互联网攻击。因此，了解配置 Atlas 安全的基础知识非常重要。

考虑这样一个场景，您正在基于 MongoDB 的项目上工作。IT 部门的同事已经在 Atlas Cloud 中部署了一个新的 MongoDB 数据库，并向您发送了连接详细信息。然而，经过查看后，您发现由于网络和用户访问的安全规则，您无法连接到新的数据库。首先要配置的是为自己提供对新数据库的访问权限。您还需要确保未经授权的互联网访问将继续被禁用。

要配置对项目数据库的访问，有两个关键方面需要牢记：

+   **网络访问**：配置 IP 网络访问

+   **数据库访问**：配置用户和数据库角色

# 网络访问

在安装和运行数据库之后，第一步是能够成功连接到我们的数据库。网络访问是 Atlas Cloud 中部署的数据库可用的低级安全配置。

对于安装在笔记本电脑上的数据库，通常不需要配置任何网络安全性。连接是指向本地安装的数据库。然而，对于部署在云基础设施上的数据库，默认情况下启用了安全性并且需要进行配置。非常重要的是保护对数据库的访问，以防止未经授权的互联网访问。在学习如何在 MongoDB 中配置网络访问之前，让我们先了解一些其核心概念。

## 网络协议

**互联网协议**（**IP**）是一个有几十年历史的标准，**传输控制协议/互联网协议**（**TCP/IP**）是所有应用程序用来可靠地在互联网上传输数据包的传输协议。互联网上的每台计算机或设备都有其独特的 IP 地址或主机名。设备之间的通信是通过在网络数据包头中包括源 IP 地址和目标 IP 地址来实现的。

注意

网络数据包头是数据包开头的附加数据，包含有关数据包携带的数据的信息。这些信息包括源 IP、目标 IP、协议和其他信息。

MongoDB 在使用 TCP/IP 作为其传输数据的网络协议方面并没有例外。此外，目前有两个版本的 IP：IPv4 和 IPv6。Atlas Cloud 平台支持这两个版本。IPv4 定义了标准的 4 字节（32 位）地址，而 IPv6 定义了标准的 16 字节（128 位）地址。

IPv4 和 IPv6 都用于指定互联网上设备的完整地址。最新的标准 IPv6 旨在克服 IPv4 协议的限制。IP 地址有两部分：IP 网络和 IP 主机地址。子网掩码是一系列位（掩码），用于指示 IP 地址的网络和主机部分。网络地址是 IP 地址的前缀，而主机的地址是剩余部分（IP 地址的后缀）：

![图 3.1：IP 地址的图解表示](img/B15507_03_01.jpg)

图 3.1：IP 地址的图解表示

在*图 3.1*中，子网掩码 255.255.0.0（或二进制格式中的（1111 1111）。（1111 1111）。（0000 0000）（0000 0000））充当掩码，指示 IP 地址的网络和 IP 主机部分。IP 地址的网络部分（前缀）由 IPv4 地址的前 16 位 100.100 组成，而主机地址是地址的其余部分-20.50。

MongoDB Atlas 使用**无类别域间路由**（**CIDR**）表示法来指定 IP 地址，而不是 IP 子网掩码。CIDR 格式是一种更短的格式，用于描述 IP 网络和主机格式。此外，CIDR 比旧的 IP 子网掩码表示法更灵活。

以下是一个子网掩码及其等效 CIDR 表示法的示例：

![图 3.2：子网掩码及其 CIDR 表示法](img/B15507_03_02.jpg)

图 3.2：子网掩码及其 CIDR 表示法

它们都描述了相同的 IP 网络- 54.175.147.0（从左边的 24 位，或 3 个字节），和主机号-155。在这个网络中可能有 254 个主机（从 1 到 254）。

注意

本课程的目标不是提供互联网网络标准的全面指南。有关更多详细信息，请参阅*理解 TCP/IP* ([`www.packtpub.com/networking-and-servers/understanding-tcpip`](https://www.packtpub.com/networking-and-servers/understanding-tcpip))，这是 TCP/IP 协议的清晰和全面指南。

### 公共 IP 地址与私有 IP 地址

如前所述，连接到互联网的任何设备都需要一个唯一的 IP 地址，以便与其他服务器通信。这些类型的 IP 地址称为**公共**IP 地址。除了公共 IP 地址，互联网标准还定义了一些保留供私人使用的 IP 地址，称为**私有**IP 地址。这些在企业环境中更常用，需要限制员工访问私人网络（内部网络）而不是让他们访问公共互联网。

以下表格描述了 IP 版本 4 可用的私有 IP 地址。

![图 3.3：IP4 的私有 IP 地址](img/B15507_03_03.jpg)

图 3.3：IP4 的私有 IP 地址

另一方面，公共 IP 地址在互联网上是唯一的，可以有与*图 3.3*中不同的任何值。

## 域名服务器

让我们考虑一个例子，IP 地址`52.206.222.245`是 MongoDB 网站的公共 IP 地址：

```js
C:\>ping mongodb.com
Pinging mongodb.com [52.206.222.245] with 32 bytes of data:
Reply from 52.206.222.245: bytes=32 time=241ms TTL=48
Reply from 52.206.222.245: bytes=32 time=242ms TTL=48
Reply from 52.206.222.245: bytes=32 time=243ms TTL=48
Ping statistics for 52.206.222.245:
    Packets: Sent = 3, Received = 3, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 241ms, Maximum = 250ms, Average = 244ms
```

正如你所看到的，我们使用名称`mongodb.com`来运行 ping 命令，而不是直接使用 MongoDB 网站的 IP 地址。`mongodb.com`，DNS 服务器会响应该主机和域名注册的公共 IP 地址：IP`54.175.147.155`。

## 传输控制协议

**传输控制协议**（**TCP**）是 IP 地址的一部分，定义了可以用于不同类型网络连接的套接字或端口。每个需要通过互联网通信的进程都使用 TCP 端口建立连接。

MongoDB 服务器的默认 TCP 端口是 27017。在 MongoDB Atlas 免费版中，无法更改默认 TCP 端口。这是 Atlas 免费版 M0 服务器的限制之一。但是，在本地安装时，可以在启动服务器时配置 TCP 监听器端口。

MongoDB Atlas Cloud 始终使用专门的网络加密协议 TLS（传输层安全）加密服务器和应用程序之间的网络通信。数据受到保护。

有几个重要的 TCP/IP 通信方面需要记住：

+   服务器始终在 TCP 端口 27017 上监听来自客户端的新连接。

+   客户端始终通过发送特殊的 TCP 数据包来初始化与服务器的连接。

+   如果配置了网络访问，客户端可以与数据库服务器建立 TCP 连接。

+   只有客户端通过了安全检查，服务器才会接受连接。

+   Atlas Cloud 中的数据库的网络通信始终是加密的。

+   一旦连接建立，客户端通过发送数据库命令和接收数据与服务器通信。

## Wire 协议

在内部，MongoDB 以一种称为**二进制 JSON**（**BSON**）的特殊二进制格式存储文档。我们在*第二章*，*文档和数据类型*中了解了 JSON 文档的结构。BSON 比 JSON 更有效地存储数据。因此，MongoDB 使用 BSON 来在文件中存储数据和在网络上传输数据。

Wire 协议是 MongoDB 将 BSON 数据封装为可以通过互联网发送的网络数据包的解决方案。Wire 协议定义了标准数据报文或数据包的格式，可以被 MongoDB 服务器和客户端理解。数据报文的结构由头部和主体组成，由 MongoDB 定义的简单但严格的格式。Wire 协议数据报文也封装在 TCP/IP 数据包中，如下图所示：

![图 3.4：封装的 Wire 协议数据报](img/B15507_03_04.jpg)

图 3.4：封装的 Wire 协议数据报

## 网络访问配置

Atlas 项目所有者或集群管理器可以从 Atlas Web 管理控制台修改网络访问。登录 Atlas 控制台后，您可以从 Atlas Web 控制台的`SECURITY`菜单中访问`Network Access`选项卡：

![图 3.5：MongoDB Atlas 控制台](img/B15507_03_05.jpg)

图 3.5：MongoDB Atlas 控制台

`Network Access`配置页面显示在页面的右侧。MongoDB Atlas 包括三种管理网络访问的方法，可以使用以下选项卡访问：

+   `IP 访问列表`

+   `对等`

+   `私有端点`

## IP 访问列表

`IP 访问列表`帮助 Atlas 管理员指定允许连接到 MongoDB 数据库的有效 IP 地址列表。要添加您的第一个 IP 地址，您可以单击页面中间的绿色按钮`ADD IP ADDRESS`：

注意

如果您已经添加了一个 IP 地址（或几个 IP 地址），则`+ ADD IP ADDRESS`按钮将显示在网络访问 IP 列表的右侧，如*图 3.6*所示。

![图 3.6：添加 IP 地址列表](img/B15507_03_06.jpg)

图 3.6：添加 IP 地址列表

当您单击`ADD IP ADDRESS`按钮（或`+ ADD IP ADDRESS`）时，将弹出一个窗口：

![图 3.7：添加新的 IP 访问列表条目](img/B15507_03_07.jpg)

图 3.7：添加新的 IP 访问列表条目

在`添加 IP 访问列表`表单中提供以下选项：

+   `添加当前 IP 地址`：这是用于简单部署的最常见方法。它允许您将自己的 IP 地址添加到 IP 访问列表中，如*图 3.7*所示。Atlas 会自动从 Web 管理控制台的当前会话中检测 IP 源地址，因此您无需记住 IP 地址。您的计算机很可能具有来自私有 IP 类的内部 IP 地址，例如 192.168.0.xx，这与 Atlas 检测到的地址有很大不同。这是因为 Atlas 始终检测网络网关的外部 IP 地址，而不是内部网络私有 IP 地址。私有 IP 地址在互联网上不可见。您可以通过在 Google 中搜索“我的 IP 是多少？”来验证您的外部 IP 地址。Google 搜索的结果应与 Atlas 中的地址匹配。

+   `允许从任何地方访问`：顾名思义，此选项通过禁用数据库的网络保护来启用从任何位置访问数据库，如*图 3.7*所示。特殊的 IP 类 0.0.0.0/0 被添加到 IP 访问列表中。

注意

不建议选择允许从任何地方访问的选项，因为这将禁用网络安全保护，并将我们的云数据库暴露给可能的攻击。

在向`IP 列表条目`字段添加自定义 IP 地址时，IP 地址需要以 CIDR 表示法表示，如本章介绍中所述。还可以在“注释”字段中输入简短的描述，如*图 3.8*所示。

![图 3.8：填写 IP 访问列表条目中的注释字段](img/B15507_03_08.jpg)

图 3.8：填写 IP 访问列表条目中的注释字段

注意

在当前版本的 Atlas 控制台中，无法将主机名或**完全限定域名**（**FQDN**）添加到 IP 访问列表中。只有 IP 地址被接受为有效条目。MongoDB Atlas 支持 IPv4 和 IPv6。例如，无法添加主机名，如*server01prd*或*server01prd.mongodb.com*（包括域），而应添加主机的公共 IP 地址。IP 地址可以通过 DNS 查找或只是 ping 主机名来获取。

## 临时访问

访问列表中的条目可以是永久的，也可以具有过期时间。临时条目在过期时会自动从列表中删除。如果要添加临时 IP 地址，请在“添加 IP 访问列表条目”表单中勾选“此条目是临时的，将在...被删除”选项，如*图 3.9*所示。您可以使用下拉菜单指定过期时间：

![图 3.9：添加临时 IP 访问列表条目](img/B15507_03_09.jpg)

图 3.9：添加临时 IP 访问列表条目

单击“确认”后，IP/主机地址将保存在访问列表中，并且网络配置将被激活。该过程通常在一分钟内完成，在此期间，条目状态将在几秒钟内显示为“待处理”，如*图 3.10*所示：

![图 3.10：显示待处理状态的网络访问窗口](img/B15507_03_10.jpg)

图 3.10：显示待处理状态的网络访问窗口

网络配置激活后，“状态”将显示为“活动”，如*图 3.11*所示：

![图 3.11：网络访问窗口](img/B15507_03_11.jpg)

图 3.11：网络访问窗口

注意

屏幕上会出现一条消息，提示用户可用 IP 地址列表，如*图 3.11*所示。

在 IP 保存在 IP 访问列表后，管理员可以修改条目。可以从“操作”选项卡中访问以下操作的权限，如*图 3.11*所示：

+   通过单击“删除”来删除 IP 访问列表中的现有条目。

+   通过单击“编辑”来编辑 IP 访问列表中的现有条目。

注意

您可以将多个 IP 地址添加到访问列表中。例如，如果您需要从办公室和家中访问云数据库，可以将两个 IP 地址都添加到访问列表中。然而，请注意，最多只能添加 200 个地址到列表中。

## 网络对等连接

网络对等连接是 Atlas Cloud 基础设施上控制网络访问的另一种方法，与 IP 访问列表不同。它使公司能够在本地公司网络和 Atlas 网络基础设施之间建立**虚拟专用云**（**VPC**）连接，如下所示：

+   私人 IP 网络用于配置客户私人网络和 MongoDB Atlas 服务器之间的 VPC。任何类型的私人 IP 都支持 VPC 网络对等连接。

+   所有云提供商都支持网络对等连接，例如 AWS 的、微软的或谷歌的云基础设施。

+   网络对等连接仅适用于大型实施（M10+），因此不适用于 Atlas 免费用户。

注意

网络对等连接和私有端点的详细信息超出了本入门课程的范围。

## 练习 3.01：启用网络访问

在本练习中，您将使用 Atlas Web 管理控制台为云中的新数据库启用网络访问。这是允许通过互联网进行网络连接所必需的。

该练习将指导您完成将您自己的 IP 地址添加到访问列表的步骤。结果，将允许从您的位置进行网络访问，并且您将能够使用在本地计算机上运行的客户端连接到 MongoDB 数据库。按照以下步骤完成此练习：

1.  转到[`cloud.mongodb.com`](http://cloud.mongodb.com)连接到 Atlas 控制台。

1.  使用您在注册 Atlas Cloud 时创建的用户名和密码登录到新的 MongoDB Atlas Web 界面：![图 3.12：MongoDB Atlas 登录页面](img/B15507_03_12.jpg)

图 3.12：MongoDB Atlas 登录页面

1.  从“安全”菜单中，点击“网络访问”选项卡：![图 3.13：网络访问窗口](img/B15507_03_13.jpg)

图 3.13：网络访问窗口

1.  在“IP 访问列表”选项卡中单击“添加 IP 地址”。

1.  在出现的“添加 IP 访问列表条目”窗口中，单击“添加当前 IP 地址”按钮：![图 3.14：IP 访问列表窗口](img/B15507_03_07.jpg)

图 3.14：IP 访问列表窗口

MongoDB Web 界面将自动检测您的外部 IP 地址，并将其反映在“IP 访问列表条目”字段中。

1.  在“注释”字段中键入`This is my IP Address`（这是可选的）：![图 3.15：在添加 IP 访问列表条目窗口中输入注释](img/B15507_03_08.jpg)

图 3.15：在添加 IP 访问列表条目窗口中输入注释

1.  单击“确认”按钮以保存新条目。Atlas 正在将新的 IP 访问列表规则部署到云系统

1.  IP 地址将出现在访问列表表中（作为活动状态）：![图 3.16：网络访问窗口](img/B15507_03_11.jpg)

图 3.16：网络访问窗口

注意

IP`100.100.10.10/32`是一个示例虚拟 IP 地址。在您的实际情况中，IP 地址将是您自己的公共 IP 地址，而且您的 ISP（互联网服务提供商）可能会分配给您一个动态 IP 地址，这是不固定的，并且可能在一段时间后更改。

我们已成功将当前公共 IP 地址列入 Atlas Cloud 控制台的白名单，以便允许来自我们公共 IP 地址的 TCP/IP 连接。如果您有多个位置，比如家里和办公室，可以在 Atlas 控制台的访问列表中添加多个 IP 地址。

# 数据库访问

部署在 Atlas 云上的 MongoDB 数据库默认启用了几个安全功能，例如用户访问控制。数据库访问控制验证用户的身份验证凭据，例如用户名和密码。因此，即使可以从任何地方访问网络，您仍需要进行身份验证才能成功连接到云中的 MongoDB 数据库。这是为了保护部署在云中的数据库免受未经授权的互联网访问。更重要的是，与其他安全功能相比，访问控制无法在云数据库中禁用，并且始终保持启用状态。

数据库访问涵盖数据库安全的以下方面：

+   数据库用户

+   数据库角色

与其他 MongoDB 安装相比，在 Atlas 云中管理用户帐户是在项目级别进行配置的。在一个 Atlas 项目中创建的用户将在该项目中创建的所有 MongoDB 数据库集群中共享。本章节涵盖了配置 Atlas 数据库安全（用户和角色）的基本方法。

注意

数据库访问仅涉及对部署在 Atlas 中的数据库服务的访问，而不涉及 Atlas 控制台本身。作为 Atlas 项目所有者，您始终可以连接到 Atlas Web 控制台以管理您的云数据库访问。如果需要向 Atlas 项目添加更多项目团队成员，则可以从 Atlas Web 应用程序的“项目”选项卡中进行操作。在本课程范围内，这些示例在连接为 Atlas 项目所有者时是相关的。

## 用户身份验证

验证用户身份是数据库安全的一个重要方面，也是为了保护数据完整性和机密性而必要的。这正是为什么所有部署在 Atlas 云中的 MongoDB 数据库在创建新的数据库会话之前都需要对用户进行身份验证的原因。因此，只有受信任的数据库用户才被授予对云数据库的访问权限。

数据库身份验证过程包括在连接之前验证用户身份的过程。

用户身份必须符合以下两个参数：

+   连接时必须提供有效的用户名。

+   用户的身份必须通过验证确认。

声明有效的用户名很简单。唯一的先决条件是用户名必须存在，这意味着用户名必须先前已创建，并且其帐户必须已激活。

### 用户名存储

用户需要在 Atlas 中声明后才能使用。用户名和密码可以存储在内部（数据库内部）或外部（数据库外部），如下所示：

+   **内部**：用户名存储在 MongoDB 数据库中的特殊集合中，位于 admin 数据库中。有一些限制。admin 数据库仅对系统管理员可访问。当用户尝试连接时，用户名必须存在于 admin 数据库中现有用户名列表中。

+   **外部**：用户名存储在外部系统中，例如**轻量目录访问协议（LDAP）**。例如，Microsoft Active Directory 是一个可以配置为 MongoDB 用户名身份验证的 LDAP 目录实现。

注意

LDAP 身份验证仅适用于更大的 Atlas 集群（M10+），并允许对许多数据库用户帐户进行企业特定配置。此配置不在本入门课程中涵盖。

### 用户名身份验证

身份验证是验证用户身份的过程。如果用户身份验证成功，则用户将被确认并信任访问数据库。否则，用户将被拒绝，并且将不被允许建立数据库连接。以下是一些身份验证机制，每种机制都具有不同的技术和安全级别。

**密码身份验证**

+   简单密码身份验证。用户需要提供正确的密码。数据库系统会根据声明的用户名验证密码。在互联网上安全验证用户密码的过程称为**握手**或**挑战-响应**。

+   密码由 MongoDB 数据库验证。在 LDAP 身份验证的情况下，密码会在外部进行验证。自 4.0 版本以来，MongoDB 有一种新的挑战-响应方法来验证密码，称为**Salted Challenge Response Authentication Mechanism**（**SCRAM**）。SCRAM 保证用户密码可以在互联网上安全验证，而无需传输或存储明文密码。这是因为在互联网的公共基础设施上传输明文密码被认为是极其不安全的。

+   在较旧版本的 MongoDB 中，使用了不同的挑战-响应方法。如果您将应用程序从 MongoDB 2.0 或 3.0 升级到最新版本，请验证 MongoDB 客户端与 MongoDB 4.0 或更高版本的兼容性。在撰写本文时，MongoDB 服务器的本地当前版本是 4.4 版本。

**X.509 证书认证**

+   这指的是使用加密证书进行用户身份验证，而不是简单密码。证书比密码更长，更安全。

+   X.509 证书是使用密码学标准**公钥基础设施**（**PKI**）创建的数字加密密钥。证书是以一对密钥（公钥-私钥）创建的。

+   这种方法还允许用户进行无密码身份验证，允许用户和应用程序使用私钥 X.509 证书进行连接。

### 在 Atlas 中配置身份验证

建议仅使用 Atlas Web 应用程序来创建和配置数据库用户。

Atlas 项目所有者可以通过 Atlas Web 界面向 Atlas 项目添加用户并配置用户的身份验证。Atlas 用户可以添加到相应 Atlas 项目中的所有数据库集群。单击 Atlas 应用程序中的`Database Access`即可提供身份验证设置。

这是 Atlas Web 应用程序的屏幕截图（[`cloud.mongodb.com`](http://cloud.mongodb.com)）：

![图 3.17：数据库访问窗口](img/B15507_03_17.jpg)

图 3.17：数据库访问窗口

在*图 3.17*中，您会注意到两个选项卡，`数据库用户`和`自定义角色`。让我们首先关注`数据库用户`的可用选项。当您单击`ADD NEW DATABASE USER`选项创建新用户时，将出现以下窗口：

![图 3.18：添加新数据库用户窗口](img/B15507_03_18.jpg)

图 3.18：添加新数据库用户窗口

注意

密码 SCRAM 身份验证是 Atlas M0 免费集群的唯一选项，该集群用于本课程的示例。其他身份验证方法选项，如证书和 AWS IAM，适用于更大的 Atlas M10+集群。

窗口中有两个字段，如*图 3.19*所示：

![图 3.19：添加新数据库用户窗口中的用户名和密码字段](img/B15507_03_19.jpg)

图 3.19：添加新数据库用户窗口中的用户名和密码字段

在第一个字段中，您可以输入新的数据库用户名。用户名不应包含空格或特殊字符。只允许 ASCII 字母、数字、连字符和下划线。

第二个字段是用户密码。管理员可以手动输入密码，也可以由 Atlas 应用程序生成。`自动生成安全密码`按钮会自动生成一个安全、复杂的密码。`SHOW`和`HIDE`选项将在屏幕上显示或隐藏密码输入。还有一个选项，可以通过单击`COPY`按钮将密码复制到剪贴板，如*图 3.19*所示。

### 临时用户

Atlas 管理员可以决定添加临时用户账户。临时用户账户是仅在有限期限内有效的账户。账户将在到期时间后由 Atlas 自动删除：

![图 3.20：在添加新用户窗口中的临时用户选项](img/B15507_03_20.jpg)

图 3.20：在添加新用户窗口中的临时用户选项

在上面的示例中，用户账户`my_user`被设置为在 1 天（24 小时）后自动过期。选择了“保存为临时用户”，并设置了规定的时间。

注意

从“内置角色或特权”下拉菜单中，管理员可以在创建新用户时分配数据库特权。默认情况下，分配的特权是“读写任何数据库”。数据库特权选项将在下一节中详细解释。

`添加用户`按钮完成了添加新用户的过程。一旦用户账户创建完成，它将出现在 MongoDB 用户列表中，如*图 3.21*所示。如果需要，可以更改或删除用户账户。用户账户的详细信息可以使用`编辑`或`删除`选项在`操作`选项卡中进行更改或删除：

![图 3.21：数据库访问窗口](img/B15507_03_21.jpg)

图 3.21：数据库访问窗口

注意

正如您在*图 3.21*中所看到的那样，`my_user`账户被设置为在 24 小时后自动过期（23:57）。用户账户将在到期时间后自动删除。

## 数据库特权和角色

数据库授权是数据库安全的一部分，涵盖了 MongoDB 数据库的特权和角色。一旦您成功验证用户并创建新的数据库会话，数据库特权和角色将分配给用户。数据库集合和对象的可访问性将根据分配给用户的数据库特权进行验证。

特权（或操作）是在 MongoDB 数据库中对特定数据库资源执行特定操作或操作的权利。例如，读取特权授予对特定数据库集合或视图进行查询的权利。

多个数据库特权可以被分组在一个角色中。MongoDB 中有一长串的数据库特权，每个特权都用于 MongoDB 中的不同功能。特权不是直接分配给用户，而是分配给角色，然后这些角色再分配给用户。因此，数据库中特权和角色的管理更容易理解：

![图 3.22：数据库特权的图示表示](img/B15507_03_22.jpg)

图 3.22：数据库特权的图示表示

角色可以具有全局或本地范围：

+   `GLOBAL`：此角色适用于所有 MongoDB 数据库和集合。

+   `Database`：此角色仅适用于特定数据库名称。

+   `Collection`：此角色仅适用于数据库中特定集合名称。它具有最严格的范围。

### 预定义角色

有一些预定义的数据库角色，对于每个角色，都有一系列特定的特权。例如，管理员角色包含了管理 MongoDB 数据库所需的所有特权。分配预定义角色是管理 MongoDB 数据库的最常见方式。

如果预定义的角色都不符合应用程序的安全要求，可以在 MongoDB 中定义自定义角色。以下角色在 Atlas 应用程序中预定义，并且可以在创建新数据库用户时分配：

+   `dbAdminAnyDatabase`、`readWriteAnyDatabase`和`clusterMonitor`。

注意

`Atlas admin`角色与 MongoDB 数据库`dbAdmin`角色不同。`Atlas admin`角色包括`dbAdmin`以及其他角色，并且仅在 Atlas Cloud 平台上可用。

+   **读写任何数据库**：此 Atlas 角色具有对任何数据库的读写权限，并适用于在一个 Atlas 项目账户中创建的所有数据库集群。

+   **仅读取任何数据库**：这是一个只读的 Atlas 角色，适用于在一个 Atlas 项目帐户中创建的所有数据库集群。

### 在 Atlas 中配置内置角色

在创建新用户时，分配内置角色的最简单方法是在创建新用户时。Atlas 提供了一个非常简单直观的界面来添加新的数据库用户。在创建新用户时会分配默认的`内置角色或权限`。然而，管理员可以为新用户分配不同的角色，或者可以编辑现有用户的权限。

注意

强烈建议仅使用 Atlas Web 界面来管理数据库角色和权限。Atlas 将自动禁用并回滚通过 Atlas Web 界面之外进行的任何更改数据库角色的更改。

Atlas 中的用户角色可以在“+添加新用户”窗口或“编辑”用户窗口中进行管理，如前一节所述：

![图 3.23：添加新数据库用户窗口](img/B15507_03_23.jpg)

图 3.23：添加新数据库用户窗口

默认情况下，在窗口中自动选择了内置的`读取和写入任何数据库`角色，如*图 3.23*所示。然而，管理员可以通过单击下拉菜单来分配不同的角色（例如`Atlas 管理员`），如*图 3.24*所示：

![图 3.24：在“添加新用户”窗口中选择角色](img/B15507_03_24.jpg)

图 3.24：在“添加新用户”窗口中选择角色

### 高级权限

有时，内置的 Atlas 数据库角色都不适合我们对数据库的访问需求。有时候，预期的数据库设计需要特殊的用户访问，或者应用程序需要实施特定的安全策略。

注意

稍后在本章中介绍的自定义角色比高级权限提供了���好的功能。始终建议创建自定义角色，并为角色分配单独的权限，而不是直接为用户分配特定的权限。

如果从下拉列表中选择`授予特定权限`，界面会发生变化：

![图 3.25：在“添加新用户”窗口中授予特定权限](img/B15507_03_25.jpg)

图 3.25：在“添加新用户”窗口中授予特定权限

如*图 3.25*所示，管理员可以快速为用户分配特定的 MongoDB 权限。这种高级功能将在本章后面的自定义角色中介绍。目前，让我们在以下练习中配置数据库访问。

## 练习 3.02：配置数据库访问

此练习的目标是为您的新 MongoDB 数据库启用数据库访问。您的数据库现在允许连接，并且正在请求用户名和密码验证。为了启用访问，您需要创建一个新用户，并为其授予适当的数据库权限。

创建一个用户名为`admindb`的管理员用户。

按照以下步骤完成此练习：

1.  重复*步骤* *1*、*2*和*3*从*练习 3.01*，*启用网络访问*，以登录到您的新 MongoDB Atlas Web 界面并选择`project 0`。

1.  从`安全`菜单中，选择`数据库访问`选项：![图 3.26：选择数据库访问选项](img/B15507_03_26.jpg)

图 3.26：选择数据库访问选项

1.  在`数据库用户`选项卡中单击`添加新数据库用户`以添加新的数据库用户。将打开“添加新用户”窗口。

1.  保持默认的身份验证方法，`密码`。

1.  提供用户名或输入`admindb`作为用户名。

1.  提供密码或单击`自动生成安全密码`以生成密码。单击`显示`以查看自动生成的密码：![图 3.27：添加新数据库用户窗口](img/B15507_03_27.jpg)

图 3.27：添加新数据库用户窗口

1.  单击`数据库用户权限`下拉菜单，并选择`Atlas 管理员`角色。

1.  单击`添加用户`。系统将对数据库应用更改：![图 3.28：新管理员用户详细信息](img/B15507_03_28.jpg)

图 3.28：新管理员用户详细信息

在*图 3.28*中，您可以看到已创建了一个名为`admindb`的新用户，其`认证方法`为`SCRAM`，`MongoDB 角色`（全局）设置为项目中所有数据库的`atlasAdmin@admin`。

新的数据库用户现在已在 Atlas 中配置和部署。

### 配置自定义角色

顾名思义，自定义角色是选定的数据库权限集合，不包括在任何内置的 Atlas 数据库角色中。例如，如果需要读取和更新权限，但没有删除和插入新文档的权限，则需要创建自定义角色，因为这种权限组合不是任何内置角色的一部分。

从“数据库访问”窗口中，单击应用程序中的第二个选项卡“自定义角色”。此选项用于创建和修改自定义 Atlas 角色。

注意

在分配给用户之前，自定义角色需要在 Atlas 中定义。

可以通过单击“添加新自定义角色”按钮来创建新的自定义角色。将出现新的自定义角色窗口：

![图 3.29：MongoDB 自定义角色](img/B15507_03_29.jpg)

图 3.29：MongoDB 自定义角色

可以根据以下类别选择操作：

+   **集合操作**：适用于集合数据库对象的操作

+   **数据库操作**：适用于数据库的操作

+   **全局操作**：适用于所有 Atlas 项目的全局操作

例如，数据库管理员只允许用户更新数据库集合。用户不能删除或插入新文档到集合中。这种特定的操作组合不包含在任何 Atlas 预定义角色中。

在一个复杂角色下可能定义许多集合/数据库/全局操作的组合。定义完成后，单击“添加自定义角色”按钮在 Atlas 中创建新角色。新角色将在列表中可见，如*图 3.30*所示：

![图 3.30：自定义角色列表](img/B15507_03_30.jpg)

图 3.30：自定义角色列表

注意

创建自定义角色后，它们将在 Atlas 中可见，并可以分配给数据库用户。可以从“添加/编辑”用户窗口中的“数据库权限”下拉列表中的“选择预定义的自定义角色”中分配新的自定义角色。

# 数据库客户端

在我们介绍 MongoDB 数据库不同类型的客户端之前，让我们先看一下简短的介绍，以澄清数据库客户端的基础知识。数据库客户端是一个旨在执行以下操作的软件应用程序：

+   连接到 MongoDB 数据库服务器

+   从数据库服务器请求信息

+   通过发送 MongoDB CRUD 请求修改数据

+   向数据库服务器发送其他数据库命令

与 MongoDB 数据库服务器的交互和兼容性至关重要。例如，客户端和服务器之间的兼容性差异（例如，不同版本）可能会产生意外结果或生成数据库或应用程序错误。这就是为什么客户端通���会经过特定版本的 MongoDB 数据库的兼容性测试和认证的原因。

让我们根据创建目的对 MongoDB 客户端进行分类：

+   **基本**：这是客户端的最简版本。通常随数据库软件一起提供，基本客户端提供一个交互式应用程序来与数据库服务器一起工作。

+   **数据导向**：这种类型的客户端旨在处理数据。通常提供**图形用户界面**（**GUI**）和辅助您高效查询、聚合和修改数据的工具。

+   **驱动程序**：这些驱动程序旨在提供 MongoDB 数据库与另一个软件系统（如通用编程语言）之间的接口。驱动程序的主要用途是在软件开发和应用部署中。

您现在已经完成了在 Atlas Cloud 中部署的新 MongoDB 数据库的所有配置更改。在之前的章节中已经介绍了在本地计算机上安装 MongoDB 客户端。如果需要，可以查看*第一章* *MongoDB 简介*，了解基本的 MongoDB 安装。下一步是使用本地 MongoDB 客户端连接到云中的新数据库。其次，将使用一组自定义的 Python 脚本进行数据迁移，因此您需要知道如何从 Python 连接到 Atlas 中的 MongoDB 数据库。下一节将讨论 MongoDB 中客户端连接的所有方面。

## 连接字符串

连接字符串到底是什么，为什么它很重要？连接字符串只不过是一种标识数据库服务地址及其参数的方法，以便客户端可以通过网络连接到服务器。它很重要，因为没有连接字符串，客户端将不知道如何连接到数据库服务。

数据库客户端，如用户和应用程序，需要形成一个有效的连接字符串，以便能够连接到数据库服务。此外，MongoDB 连接字符串遵循**统一资源标识符**（**URI**）格式，以将所有连接详细信息传递给数据库客户端。

以下是 MongoDB 连接字符串的一般格式：`mongodb+srv://user:pass@hostname:port/database_name?options`

连接字符串的各个元素在下表中描述：

![图 3.31：连接字符串的各个元素](img/B15507_03_31.jpg)

图 3.31：连接字符串的各个元素

注意

关于新前缀`mongodb+srv`以及如何使用 DNS SRV 记录来识别 MongoDB 服务的更多细节将在*第十章* *复制*中介绍。

现在让我们看一些连接字符串的例子，如下所示：

`mongodb+srv://guest:passwd123@atlas1-u7xxx.mongodb.net:27017/data1`

此连接字符串适合尝试使用以下参数进行数据库连接：

+   服务器在 Atlas Cloud 上运行（主机名为`mongodb.net`）。

+   数据库集群名称为`atlas1`。

+   尝试使用用户名`guest`和密码`passwd123`进行连接。

+   数据库服务在标准 TCP 端口`27017`上提供。

+   服务器上的默认数据库名称是`data1`。

虽然前面的连接字符串对 Atlas 数据库连接是有效的，但通常不建议在连接字符串中显示密码。以下是一个在连接时请求密码的例子：

`mongodb+srv://guest@atlas1-u7xxx.mongodb.net:27017/data1`

另一个例子如下：

`mongodb+srv://atlas1-u7xxx.mongodb.net:27017/data1 --username guest`

在这种情况下，使用`guest 用户名`尝试连接。但是，密码不是连接字符串的一部分，它将在连接时由服务器请求。

如果省略了数据库名称（或者无效），则会尝试连接到默认数据库，即 admin 数据库。此外，如果省略了 TCP 端口，它将尝试连接到默认的 TCP 端口 27017，如下例所示：

`mongodb+srv://guest@atlas1-u7xxx.mongodb.net`

对于非云数据库连接或传统的 MongoDB 连接，应使用简单的`mongodb`前缀。以下是一些非云连接字符串的例子：

`mongodb://localhost/data1`

在这个例子中，主机名是`localhost`，这意味着数据库服务器在与应用程序相同的计算机上运行，并尝试连接到数据库`data1`。以下是另一个在非默认 TCP 端口`5500`上进行远程网络连接的例子：

`mongodb://devsrv01.dev-domain-example.com:5500/data1`

如果连接字符串中未指定用户名，则尝试在没有用户名的情况下进行连接。这种类型的连接适用于没有授权模式（未配置用户安全）的数据库。授权模式始终针对云数据库进行配置。

注意

如果数据库服务配置为复制或分片集群，则 MongoDB 连接字符串可能会有所不同。有关 MongoDB 集群的连接字符串示例将在*第十章* *复制*中提供。

### Mongo Shell

连接到 MongoDB 数据库最简单的方法可能是使用 mongo shell。mongo shell 为 MongoDB 数据库提供了一个简单的终端模式客户端：

+   mongo shell 包含在所有 MongoDB 安装中。

+   它可以用于在终端模式下运行服务器交互命令。

+   它可以用于运行 JavaScript。

+   mongo shell 有自己的命令。

要启动 mongo shell，请在命令提示符中运行`mongo`命令，如下所示：

```js
C:\>mongo --help
MongoDB shell version v4.4.0
usage: mongo [options] [db address] [file names (ending in .js)]
db address can be:
  foo                   foo database on local machine
  192.168.0.5/foo       foo database on 192.168.0.5 machine
  192.168.0.5:9999/foo  foo database on 192.168.0.5 machine on port 9999
  mongodb://192.168.0.5:9999/foo  connection string URI can also be used
Options:
  --ipv6                               enable IPv6 support (disabled by
....
```

## 练习 3.03：使用 Mongo Shell 连接到云数据库

这个简单的练习将向您展示使用 mongo shell 连接到 Atlas 的步骤。在这个练习中，使用连接字符串中的`mongodb+srv`前缀。第一步是获取 Atlas Cloud 数据库的集群名称（DNS SRV 记录）：

1.  登录到您的新 MongoDB Atlas web 界面，使用您在注册 Atlas Cloud 时创建的用户名和密码：![图 3.32：MongoDB Atlas 登录页面](img/B15507_03_32.jpg)

图 3.32：MongoDB Atlas 登录页面

1.  点击`Atlas`项目菜单中的`Clusters`选项卡，如*图 3.33*所示。

1.  在`Clusters`菜单中点击`CONNECT`按钮。在 M0 免费版中，只有一个名为`Cluster0`的集群：![图 3.33：集群窗口](img/B15507_03_33.jpg)

图 3.33：集群窗口

1.  `连接到 Cluster0`窗口出现：![图 3.34：连接到 Cluster0 窗口](img/B15507_03_34.jpg)

图 3.34：连接到 Cluster0 窗口

1.  单击`使用 mongo shell 连接`。将出现以下窗口：![图 3.35：连接到 Cluster0 页面](img/B15507_03_35.jpg)

图 3.35：连接到 Cluster0 页面

1.  选择`我已安装 mongo shell`选项，并选择正确的 mongo shell 版本（在撰写本文时，最新的 mongo shell 版本是 4.4）。或者，如果您尚未安装 mongo shell，可以选择`我尚未安装 mongo shell`并安装 mongo shell。

1.  单击`复制`以将连接字符串复制到剪贴板。

1.  在您的操作系统中启动命令提示符窗口或终端。

1.  使用新的连接字符串命令行启动 mongo shell：

```js
C:\>mongo "mongodb+srv://cluster0.u7n6b.mongodb.net/test" --username admindb
```

以下详细信息将出现：

```js
MongoDB shell version v4.4.0
Enter password:
connecting to: mongodb://cluster0-shard-00-00.u7n6b.mongodb.net:27017,cluster0-
Implicit session: session { "id" : UUID("7407ce65-d9b6-4d92-87b2-754a844ae0e7") }
MongoDB server version: 4.2.8
WARNING: shell and server versions do not match
MongoDB Enterprise atlas-rzhbg7-shard-0:PRIMARY>
```

要作为*练习 3.02*中创建的`admindb`数据库用户连接到 Atlas 数据库时，当提示时提供`admindb`用户的密码并完成连接。

成功建立连接后，shell 提示将显示以下详细信息：

```js
MongoDB Enterprise atlas-rzhbg7-shard-0:PRIMARY>
```

具体细节如下：

+   `企业`：这指的是 MongoDB 企业版。

+   `atlas1-#####-shard-0`：这指的是 MongoDB 副本集名称。我们将在后面更详细地了解这个。

+   `PRIMARY>`：这指的是 MongoDB 实例的状态，即`PRIMARY`。

注意

您可能会看到一条消息，上面写着`警告：shell 和服务器版本不匹配`。这是因为 mongo shell 的最新版本是 4.4，而 M0 Atlas 云数据库的版本是 4.2.8。可以忽略此警告。

1.  输入`exit`退出 mongo shell。

在这个练习中，您使用 mongo shell 客户端连接到了一个云数据库。为了方便起见，您使用 Atlas 界面复制了我们 Atlas 集群的连接字符串。在实践中，开发人员已经提前准备好了数据库连接字符串，因此他们不需要每次连接数据库时都从 Atlas 应用程序中复制它。

## MongoDB Compass

MongoDB Compass 是 MongoDB 中数据可视化的图形工具。它与 MongoDB 服务器安装一起安装，因为 MongoDB Compass 是标准发行版的一部分。另外，MongoDB Compass 也可以单独下载和安装，而无需 MongoDB 服务器软件。

MongoDB Compass 的简单而强大的图形用户界面帮助您轻松查询和分析数据库中的数据。MongoDB Compass 具有一个查询构建器图形界面，大大简化了创建复杂的 JSON 数据库查询的工作。

MongoDB Compass 版本 1.23 如下截图所示：

![图 3.36：MongoDB Compass 连接到 Atlas 云](img/B15507_03_36.jpg)

图 3.36：MongoDB Compass 连接到 Atlas 云

以下是标准版本中最重要的 MongoDB Compass 功能：

+   轻松管理数据库连接

+   与数据、查询和 CRUD 的交互

+   高效的图形查询构建器

+   查询执行计划的管理

+   聚合构建器

+   集合索引的管理

+   模式分析

+   实时服务器统计信息

除了标准的 MongoDB Compass 版本外，在撰写本章时，还有其他两个版本的 MongoDB Compass 可供下载：

+   Compass 隔离：用于高度安全的环境。Compass 的隔离版本仅向连接的 MongoDB 服务器发起网络请求。

+   Compass 只读：顾名思义，Compass 的只读版本不会更改数据库中的任何数据，仅用于查询。

注意：

MongoDB Compass 社区版本现已停用。您可以使用免费的完整版本 MongoDB Compass，其中包括 MongoDB 模式分析等企业版功能。

## MongoDB 驱动程序

有一种误解，即 MongoDB 只是 JavaScript 堆栈的数据库。将 MongoDB 的能力减小并仅将其用于 JavaScript 应用程序是不恰当的。

MongoDB 是一个多平台数据库，具有灵活的数据模型，可用于任何类型的应用程序。此外，几乎每种编程语言都对 MongoDB 有很好的支持。

目前，最有用和最受欢迎的 MongoDB 客户端版本是驱动程序。MongoDB 驱动程序是数据库与软件开发世界之间的粘合剂。目前，对于最流行的编程语言，如 C/C++、C#、Java、Node 和 Python，都有许多驱动程序。

驱动程序 API 是软件库接口，它使得可以直接在编程语言结构中使用 MongoDB 数据库功能。例如，来自 MongoDB 的特定 BSON 数据类型被转换为可以在诸如 Python 之类的编程语言中使用的数据格式。

## 练习 3.04：使用 Python 驱动程序连接到 MongoDB 云数据库

商业决策通常基于数据分析。有时，为了获得有用的结果，开发人员使用诸如 Python 之类的编程语言来分析数据。Python 是一种强大的编程语言，但易于学习和实践。在这个练习中，您将从 Python 3 连接到 MongoDB 数据库。在使用 Python 连接到 MongoDB 之前，请注意以下几点：

+   您无需在计算机上本地安装 MongoDB 即可使用 Python 进行连接。

+   Python 库使用`pymongo`模块连接到 MongoDB。

+   `pymongo`模块适用于 Python 2 和 Python 3。但是，由于 Python 2 现在已经停止维护，强烈建议在新软件开发中使用 Python 3。

+   MongoDB 客户端是`pymongo` Python 库的一部分。

+   您还需要安装 DNSPython 模块，因为 Atlas 连接字符串是 DNS SRV 记录。因此，需要 DNSPython 模块来执行 DNS 查找。

按照以下步骤完成练习：

1.  验证 Python 版本是否为 3.6 或更高，方法如下：

```js
# Check Python version – 3.6+
# On Windows
C:\>python --version
Python 3.7.3
# On MacOS or Linux OS
$ python3 --version
```

注意

对于 macOS 或 Linux，Python shell 可以使用`python3`而不是`python`来启动。

1.  安装`pymongo`之前，请确保安装了 Python 软件包管理器`pip`：

```js
# Check PIP version
# On Windows
C:\>pip --version
pip 19.2.3 from C:\Python\Python37\site-packages\pip (python 3.7)
# On MacOS and Linux
$ pip3 --version
```

1.  安装`pymongo` `client`，如下：

```js
# Install PyMongo client on Windows
C:\>pip install pymongo
# Install PyMongo client on MacOS and Linux
$ pip3 install pymongo
# Example output (Windows OS)
C:\>pip install pymongo
Collecting pymongo
  Downloading https://files.pythonhosted.org/packages/c9/36/715c4ccace03a20cf7e8f15a670f651615744987af62fad8b48bea8f65f9/pymongo-3.9.0-cp37-cp37m-win_amd64.whl (351kB)
     358kB 133kB/s
Installing collected packages: pymongo
Successfully installed pymongo-3.9.0
```

1.  安装`dnspython`模块，如下：

```js
# Install dnspython on Windows OS
C:\> pip install dnspython
# Install dnspython on MacOS and Linux
$ pip3 install dnspython
# Example output (Windows OS)
C:\> pip install dnspython
Collecting dnspython
  Using cached https://files.pythonhosted.org/packages/ec/d3/3aa0e7213ef72b8585747aa0e271a9523e713813b9a20177ebe1e939deb0/dnspython-1.16.0-py2.py3-none-any.whl
Installing collected packages: dnspython
Successfully installed dnspython-1.16.0
```

现在你已经准备好了 Python 环境，下一步是获取你的云数据库的正确连接字符串。测试 MongoDB 连接以确认这一点。

1.  编辑连接字符串并添加你的数据库名称和密码。使用在*Exercise 3.02*，*Configuring Database Access*中创建的`admindb`用户名尝试连接：

```js
mongodb+srv://admindb:<password>@<server_link>/<database_name>
```

1.  用你的服务器链接替换`<server_link>`。

注意

例如，考虑以下情况，连接字符串如下：

`"mongodb+srv://admindb:xxxxxx@cluster0-u7xxx.mongodb.net/test?retryWrites=true&w=majority"`

这里，服务器链接可以快速识别为：`cluster0-u7xxx.mongodb.net`

1.  用你的数据库名称替换`<database_name>`，在这种情况下是`sample_mflix`。

1.  用`admindb`用户密码替换`<password>`。

注意

如果你想用不同的用户连接，而不是`admindb`，请用你的用户名替换`admindb`，用你的密码替换`<password>`。

1.  编辑一个 Python 测试脚本来测试你的连接并执行 Python 脚本。在 Windows 中，打开记事本文本编辑器，输入以下 Python 代码：

```js
# Python 3 script to test MongoDB connection
# MongoDB Atlas connection string needs to be edited with your connection
from pymongo import MongoClient
uri="mongodb+srv://admindb:xxxxxx@cluster0-u7xxx.mongodb.net/test?retryWrites=true&w=majority"
client = MongoClient(uri)
# switch to mflix database
mflix = client['sample_mflix']
# list collection names
print('Mflix Collections: ')
for name in mflix.list_collection_names(): 
  print(name)
```

注意：

不要忘记使用你的 Atlas 连接详细信息更新 URI。如果你使用本例中提供的 URI，那么你将收到连接错误。

1.  将文本脚本保存为`mongo4_atlas.py`，例如在`C:\Temp\mongo4_atlas.py`中。

1.  运行测试脚本。

在 Windows 的命令提示符中，键入：

```js
"python C:\Temp\mongo4_atlas.py"
```

在 macOS/Linux shell 提示符中，键入：

```js
"$ python3 ./mongo4_atlas.py " 
```

脚本的输出将显示数据库中的集合，如下所示：

```js
C:\>python C:\Temp\mongo4_atlas.py 
Mflix Collections: 
comments
users
theaters
sessions
movies
>>>
```

在这个练习中，你通过使用 Python 等编程语言在云中实际操作 MongoDB。在使用扩展的 Python 库方面，可能性是无限的；你可以创建 Web 应用程序，进行数据分析等。

# 服务器命令

MongoDB 是一个数据库服务器，它有客户端通过网络连接到服务器。数据库服务器管理数据库，而客户端被应用程序或用户用来从数据库查询数据。如果你想知道是否只有数据库（没有服务器），那么是的，有的。例如，Microsoft Access 就是一个没有数据库服务器的关系型数据库的例子。客户端-服务器架构的主要优势在于服务器整合了控制数据管理、用户安全和并行访问的并发性。

还有物理和逻辑结构的分离。数据库服务器管理数据库的物理结构，如存储和内存。另一方面，数据库客户端通常只能访问逻辑数据库结构，如集合、索引和视图。

本节将简要解释 MongoDB 4.4 中的物理和逻辑结构。

## 物理结构

数据库的物理结构由为 MongoDB 服务器分配的计算资源组成，例如处理器线程、内存分配和数据库文件存储。计算需求和调整是数据库管理的重要部分，特别是对于本地部署的数据库服务器。然而，在部署在 MongoDB Atlas 云上的数据库的情况下，数据库的物理结构对用户不可见。数据库由 MongoDB 在内部管理。因此，云用户可以专注于数据库利用和应用开发，而不是花时间在物理资源的数据库管理上，比如存储和内存。

如介绍所述，MongoDB Atlas 根据集群层大小分配物理资源。资源管理完全通过云 Atlas 应用程序进行。如果需要更多资源，集群可以扩展到更大的大小。

免费的 M0 集群没有专用资源（只有共享的 CPU 和内存）。但是，免费的 M0 集群是一个很好的数据库集群，因为它始终可用于学习和测试 MongoDB。

### 数据库文件

MongoDB 会在磁盘上自动创建许多类型的文件，如数据文件和日志文件。在 Atlas 云数据库的情况下，所有数据库文件都由 MongoDB 内部管理：

+   **数据文件：** 这些文件用于数据库集合和其他数据库对象。MongoDB 有一个可配置的数据文件存储引擎，WiredTiger 是一个高性能的存储引擎，自 MongoDB 3.0 版本以来就被引入。

+   **Oplog：** 这些文件用于集群成员之间的事务复制。我们将在*第十章*中详细学习这些。

+   **其他文件：** 这些文件包括配置文件、数据库日志和审计文件。

### 数据库指标

虽然云部署的数据库不涉及数据文件和内存管理，但有必要监视分配的云资源的利用情况。Atlas 资源监控提供了一个图形界面，显示性能指标。在 Atlas 中有许多可用的指标，如逻辑数据库指标、物理数据库指标和网络带宽。

此主题的内容超出了本书的范围。有关更多详细信息，您可以参考 MongoDB Atlas 文档，*监控和警报* ([`docs.atlas.mongodb.com/monitoring-alerts/`](https://docs.atlas.mongodb.com/monitoring-alerts/))。

## 逻辑结构

数据库的逻辑结构包括数据库、集合和其他数据库对象。以下图表示了 MongoDB 的主要逻辑结构：

![图 3.37：MongoDB 的逻辑结构](img/B15507_03_37.jpg)

图 3.37：MongoDB 的逻辑结构

**MongoDB 服务器：** 运行 MongoDB 服务器实例的物理或虚拟计算机。对于 MongoDB 集群，当客户端连接到 MongoDB 时，会有一组少量的 MongoDB 实例

**数据库：** MongoDB 集群包含许多数据库。每个数据库是 MongoDB 中的逻辑存储容器，用于数据库对象。在部署数据库时会创建一些系统数据库。系统数据库由 MongoDB 服务器内部用于数据库配置和安全，不能用于用户数据。

**对象：** 一个数据库包含以下对象：

+   JSON 文档的集合

+   索引

+   视图

MongoDB 中的基本逻辑实体是 JSON 文档。多个文档被分组在一个集合中，多个集合被分组在一个数据库中。在 MongoDB 版本 4 中，引入了更多的对象，如数据库视图，这为数据库增加了更多功能。我们将在*练习 3.05*中使用一个合适的示例来学习数据库视图对象的内容，*创建数据库视图对象*。

## 服务器命令

在客户端-服务器数据库服务器架构中，例如 MongoDB 服务器，客户端向数据库服务器发送请求，MongoDB 服务器在服务器端执行请求。因此，当服务器执行客户端请求时，不涉及客户端处理。一旦请求完成，服务器将执行结果或消息发送回客户端。

MongoDB 服务器有许多功能，但有几个不同的类别：

+   **CRUD 操作**：数据库**创建、读取、更新、删除**（**CRUD**）操作是修改数据文档的命令。

+   **数据库命令**：这些命令与数据查询和 CRUD 操作不同。数据库命令有其他功能，如数据库管理、安全和复制。

大多数数据库命令都是由 Atlas 在用户更改数据库配置时在后台执行的。例如，当 Atlas 项目所有者添加新用户时，Atlas 应用程序会在后台运行数据库命令，以在数据库中创建用户。然而，也可以从 MongoDB Shell 或 MongoDB Driver 中执行服务器命令。

一般来说，运行数据库命令的语法如下：

```js
>>> db.runCommand( { <db_command> } )
```

`db_command`是数据库命令。

例如，如果我们想要检索在 MongoDB 中正在执行的当前操作，我们可以使用以下语法运行命令：

```js
>>> db.runCommand( {currentOp: 1} )
```

服务器将返回一个 JSON 格式的文档，其中包含正在进行的操作。

一些数据库命令有自己的更短的语法，并且可以在没有一般`db.runCommand`语法的情况下运行。这是为了方便记住更常用的命令的语法。例如，列出当前数据库中所有集合的命令的语法是：

```js
>>> db.getCollectionNames()
```

对于部署在 Atlas Cloud 中的数据库，有一些数据库管理命令无法直接从 mongo shell 中执行。完整的命令列表可在 MongoDB Atlas 文档中找到，*M0/M2/M5 集群中不支持的命令*（[`docs.atlas.mongodb.com/reference/unsupported-commands/`](https://docs.atlas.mongodb.com/reference/unsupported-commands/)）。

## 练习 3.05：创建数据库视图对象

在这个练习中，您将练习数据库命令。练习的目标是从 mongo shell 终端创建一个新的数据库对象。您将创建一个数据库视图对象，仅显示三列：电影名称，发行年份和集合信息。您将使用 MongoDB 控制台执行所有数据库命令。

以下是执行此练习的步骤：

1.  使用 MongoDB 控制台的连接字符串连接到 Atlas 数据库。重复*练习 3.03*中的*步骤 1 到 9*，*使用 Mongo Shell 连接到云数据库*，使用 mongo shell 客户端进行连接。如果您已经为 Atlas 数据库准备好了连接字符串，请启动 mongo shell 并按照*练习 3.03*中的*步骤 8*描述的方式进行连接，*使用 Mongo Shell 连接到云数据库*。

1.  使用`use`数据库命令选择`mflix`电影数据库：

```js
>>> use sample_mflix
```

1.  使用`getCollectionNames`数据库命令列出现有的集合，以返回当前数据库中所有集合的列表：

```js
>>> db.getCollectionNames()
```

1.  从电影集合创建一个`short_movie_info`视图：

```js
db.createView(
   "short_movie_info",
   "movies",
   [ { $project: { "year": 1, "title":1, "plot":1}}]
)
```

注意

`$project`运算符用于从电影集合中选择仅三个字段（`year`，`title`和`plot`）。

1.  执行`createView`代码：

```js
MongoDB Enterprise Cluster0-shard-0:PRIMARY> db.createView(
...    "short_movie_info",
...    "movies",
...    [ { $project: { "year": 1, "title":1, "plot":1}}]
... )
```

响应`"ok" : 1`表示成功执行创建和查看数据库的命令，没有错误，如下代码输出所示：

```js
# Command Output
{
        "ok" : 1,
        "operationTime" : Timestamp(1569982200, 1),
        "$clusterTime" : {
                "clusterTime" : Timestamp(1569982200, 1),
                "signature" : {
                        "hash" : BinData(0,"brozBUoH099xryq5l439woGcL3o="),
                        "keyId" : NumberLong("6728292437866840066")
                }
        }
}
```

注意

输出的详细信息可能会根据服务器运行时的值而有所不同。

1.  验证视图是否已创建。视图只显示为一个集合：

```js
>>> db.getCollectionNames()
```

此命令返回一个包含集合列表中视图名称的数组。

1.  查询视图，如下：

```js
>>> db.short_movie_info.findOne()
```

视图数据库对象的行为与普通集合完全相同。您可以以与查询数据库集合相同的方式查询视图。您将运行一个简短的查询，只返回一个文档。

此查询的输出将只显示文档`id`，`plot`，`year`和`title`。完整的会话输出如下：

![图 3.38：会话输出](img/B15507_03_38.jpg)

图 3.38：会话输出

这是一个创建新数据库对象的示例，比如一个简单的视图。视图对于用户和开发人员来说非常有用，可以连接多个集合，并且可以限制 JSON 文档中的某些字段的可见性。一旦我们学习了更多关于 MongoDB 查询和聚合的知识，我们就可以应用所有这些技术来在数据库中创建更复杂的视图，从多个集合到使用聚合管道。

## 活动 3.01：管理您的数据库用户

假设您负责管理公司的 MongoDB 数据库，该数据库位于**亚马逊网络服务**（**AWS**）的 MongoDB Atlas 云基础设施中。最近，您收到通知，新开发人员 Mark 已加入团队。作为新团队成员，Mark 需要访问 MongoDB 电影数据库，用于一个新项目。

执行以下高级步骤以完成此活动：

1.  创建一个名为`dev_mflix`的新数据库，该数据库将用于开发。

1.  为开发人员创建一个名为`developers`的新自定义角色。

1.  向`developers`角色授予对`dev_mflix`数据库的读写权限。

1.  向`developers`角色授予对`sample_mflix`电影数据库的只读权限。

1.  为 Mark 创建一个新的数据库帐户。

1.  将`developers`自定义角色授予 Mark。

1.  通过使用 Mark 作为用户连接到数据库来验证帐户，并验证访问权限。

Mark 不应能够修改生产电影数据库，也不应该能够看到服务器上除`sample_mflix`和`dev_mflix`之外的任何其他数据库。

一旦 Mark 成功添加到 Atlas 项目中，您应该能够使用该帐户测试连接。使用以下命令使用 mongo shell 进行连接：

```js
C:\> mongo "mongodb+srv://cluster0.u7n##.mongodb.net/admin" --username Mark
```

注意

您的实际连接字符串不同，需要从 Atlas 连接窗口中复制，如本章所述。

这是输出终端的一个示例（来自 mongo shell）：

![图 3.39：连接 MongoDB Shell](img/B15507_03_39.jpg)

图 3.39：连接 MongoDB Shell

注意

此活动的解决方案可以通过此链接找到

# 摘要

在本章中，您学习了 Atlas 服务管理的基础知识。由于安全性是云计算的一个非常重要的方面，控制网络访问和数据库访问对于 Atlas 平台至关重要，您现在应该能够设置新用户并授予对数据库资源的权限。还详细探讨了数据库连接和 MongoDB 数据库命令。下一章将向您介绍 MongoDB 查询语法的世界。MongoDB NoSQL 语言是一种功能丰富且强大的数据库语言，与所有编程语言都非常好地集成在一起。