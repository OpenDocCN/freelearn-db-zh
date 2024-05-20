# 第二章：安装和升级 MySQL 8

在上一章中，我们提供了 MySQL 的概述以及 MySQL 8 的新功能、用例和限制。MySQL 在平台方面非常灵活，例如 RedHat、Fedora、Ubuntu、Debian、Solaris、Microsoft Windows 等。它支持与不同语言连接的 API，例如 C、C++、C#、PHP、Java、Ruby 等。对于任何编程平台，设置环境与必要的软件工具是最重要和单调的任务。但对于 MySQL 8 来说，情况并非如此，因为本章重点是如何设置 MySQL 8 的环境。

本章详细解释了 MySQL 8 的安装步骤以及必要的先决条件。提供了在各种平台上设置 MySQL 8 的单独安装步骤。本章还涵盖了升级到 MySQL 8 或从 MySQL 8 降级的方法。

本章将涵盖以下主题：

+   MySQL 8 安装过程

+   MySQL 8 的后安装设置

+   MySQL 8 升级

+   MySQL 8 降级

# MySQL 8 安装过程

本节将指导读者选择 MySQL 8 版本，从哪里获取 MySQL 8 以及如何安装 MySQL 8。它还解释了设置所需的后安装步骤。本章提供了有关如何升级或降级从 MySQL 8 的信息。

# 通用安装指南

MySQL 8 在许多操作系统上都有不同版本。MySQL 8 的发布以两种方式进行管理：

+   **开发发布**：这具有最新功能，但不建议在生产中使用

+   **通用发布**：这是一个稳定的发布，用户可以在生产中使用它

MySQL 8 的每个发布都遵循命名约定，表示其状态。每个发布名称由三个数字和一个可选后缀组成。例如，**mysql.8.1.2-rc**。数字的解释如下：

+   第一个数字（8）表示发布的主要版本。

+   第二个数字（1）表示发布的次要版本。主要和次要数字的组合描述了发布系列。

+   第三个数字（2）表示发布系列中的版本。每个错误修复发布都会增加它。

最新版本的发布是最适合使用的。示例中给出的后缀表示 MySQL 8 发布的稳定性。MySQL 8 发布遵循三个后缀：

+   **开发里程碑发布**（**dmr**）：MySQL 8 遵循里程碑模型，其中每个里程碑表示经过彻底测试的功能。

+   **发布候选人**（**rc**）：新功能可能会在此版本中发布，但目的是修复先前发布的功能中的错误。

+   **没有后缀**：这表示**通用可用性**（**GA**）或生产发布。此版本稳定，并通过了早期阶段。它可靠且适用于生产。

如前所述，每个发布之前都是 DMR，然后是 RC，最后是 GA 发布状态。现在，在决定安装 MySQL 8 版本之后，是时候选择分发格式了。

二进制分发被推荐用于通用用途。它以原生格式提供给许多平台。例如，Linux 的 RPM 软件包和 OS X 的 DMG 软件包。

# 下载 MySQL 8

要从官方网站获取 MySQL 8，请参考以下网址：[`dev.mysql.com/downloads/`](http://dev.mysql.com/downloads/)。MySQL 还提供了一个镜像网站：[`dev.mysql.com/downloads/mirrors.html`](http://dev.mysql.com/downloads/mirrors.html)。当您到达下载页面时，您可以在页面底部看到版本选择选项卡，其中显示了两个选项卡：

+   **通用可用性**（**GA**）发布

+   开发发布

根据前一节，从列表中选择合适的版本，然后单击下载按钮。

# 验证软件包的完整性

这是下载软件包可用并准备安装的阶段。这是一个可选步骤，但我们建议这样做，以避免在安装过程中出现错误。有三种不同的方法可用于检查完整性：

+   使用 MD5 校验和

+   使用加密签名

+   使用 RPM 完整性验证机制

# 使用 MD5 校验和

这是检查完整性的最简单方法，需要很少的工作。MySQL 下载页面本身提供了一个 MD5 校验和，对于每个 MySQL 产品都是唯一的。下载 MySQL 8 后，我们只需确保下载文件的校验和与下载页面上提供的校验和匹配即可。有许多工具可用于不同操作系统来比较校验和。在这里，我们提供了一个使用命令行和一个名为 winMD5Sum 的图形工具的 MD5 校验和示例，用于 Windows 操作系统。

执行以下步骤以执行命令行：

1.  从[`www.fourmilab.ch/md5/`](http://www.fourmilab.ch/md5/)下载实用程序

1.  在`E:\Softwares`位置下解压文件

1.  转到命令行并执行以下命令：

```go
 E:\Softwares\md5>md5.exe 
        E:\Softwares\mysql-installer-community-5.7.19.0.msi
 2578BFC3C30273CEE42D77583B8596B5 
        E:\Softwares\mysql-installer-community-5.7.19.0.msi
```

执行以下步骤以执行图形工具：

1.  打开链接：[`www.nullriver.com/index/products/winmd5sum`](http://www.nullriver.com/index/products/winmd5sum)。

1.  点击页面上的下载 WinMD5Sum 选项。它将下载`winMD5Sum.exe`到您的计算机上。

1.  运行下载的`Install-winMD5Sum.exe`并将其安装在本地计算机上。

1.  安装成功后，打开 winMD5Sum 工具。这将打开一个对话框，在对话框中您必须选择下载的`MySQL.msi`文件。

1.  点击计算按钮。这将计算已下载文件的 MD5 校验和。

1.  在比较文本框中输入 MySQL 下载页面上提供的 MD5 校验和，然后点击比较按钮。

# 使用加密签名

这种完整性验证技术需要一个公共的 GPG 构建密钥。该密钥可以从[`pgp.mit.edu/`](http://pgp.mit.edu/) URL 获取。下载构建密钥后，您必须执行以下步骤：

1.  导入构建密钥

1.  从 MySQL 网站下载所需的 MySQL 8 软件包及其相关签名

确保 MySQL 软件包名称及其下载的签名文件名称相同。这两个文件必须放在一个共同的存储位置下。

1.  现在，是时候执行以下命令进行验证了：

```go
 cmd> gpg --verify package_name.asc
```

对于 Microsoft Windows，还有一些 GUI 工具可用于完整性检查。其中最流行的之一是`Gpg4win`。要在 Linux 上执行相同的检查，我们有可用的命令，因为 RPM 软件包本身包含 GPG 签名和 MD5 校验和。执行以下命令以验证软件包：

```go
cmd> rpm --checksig package_name.rpm
```

这种验证技术比 MD5 校验更可靠，但非常复杂，需要更多的完整性检查工作。

# 在 Microsoft Windows 上安装 MySQL 8

MySQL 提供 32 位和 64 位版本。有多种安装 MySQL 8 在 Microsoft Windows 上的方法。最常见的方法是使用安装程序，在本地系统上安装和配置 MySQL 8。

在安装 MySQL Community 8.0 服务器之前，请确保系统上已安装了 Microsoft Visual C++ 2015 可再发行包。

MySQL 8 可以作为标准应用程序运行，也可以作为 Windows 服务运行。使用服务，用户可以使用 Windows 服务管理工具控制和测量操作。每个平台都有三种主要的分发格式：

+   **安装程序分发**：这包括 MySQL 8 服务器以及其他产品，如 MySQL Workbench、MySQL for Excel 和 MySQL Notifier。安装程序还可用于将产品升级到其他版本。

+   **源分发**：顾名思义，这包含所有源代码以及所有支持的文件。需要 Visual Studio 编译器才能使其可执行。

+   **二进制分发**：此分发以 ZIP 文件格式提供。它包含除安装程序之外的所有必需文件。用户必须将文件解压缩到所选目录中。

# Windows 特定注意事项

在 Microsoft Windows 上安装 MySQL 8 之前，请考虑以下几点：

+   **防病毒软件**：众所周知，防病毒软件使用指纹技术，将快速更改的文件视为潜在的安全风险。在 MySQL 8 中，有一些目录包含 MySQL 8 相关数据和临时表信息，并且经常更新。因此，防病毒软件有可能将这些文件视为垃圾邮件。这也会影响性能。

防病毒软件提供配置以排除一些目录，因此建议排除 MySQL 8 数据目录和临时目录。默认情况下，MySQL 8 将临时数据存储到 Microsoft Windows 临时目录中。要更改 MySQL 8 中的此默认配置，请参考`my.ini`文件的`tempdir`参数。

+   **大表支持**：在 NTFS 或任何新文件系统上使用 MySQL 8 以支持大小超过 4 GB 的大表。对于这些更大的表，用户必须在创建表时定义`MAX_ROWS`和`AVG_ROW_LENGTH`属性。

# MySQL 8 安装布局

微软 Windows 默认情况下，将`C:\Program Files`目录视为 MySQL 8 安装目录。然而，在安装时我们可以选择目录。无论安装位置如何，安装后的子目录结构保持不变。对于 Microsoft Window 布局，请参考以下表格：

| **目录** | **目录内容** | **备注** |
| --- | --- | --- |
| `bin` | [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)服务器，客户端和实用程序 |   |
| `%PROGRAMDATA%\MySQL\MySQL Server 8.0\` | 日志文件，数据库 | Windows 系统变量`%PROGRAMDATA%`默认为`C:\ProgramData` |
| `examples` | 示例程序和脚本 |   |
| `include` | 包括（`header`）文件 |   |
| `lib` | 库 |   |
| `share` | 包括错误消息，字符集文件，示例配置文件，用于数据库安装的 SQL 等各种支持文件 |  |

您可以在[`dev.mysql.com/doc/refman/8.0/en/windows-installation-layout.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-installation-layout.html)了解更多关于此主题的信息。

# 选择正确的安装包

在 Windows 上安装 MySQL 8 时，有多种软件包格式可供选择。MySQL 提供了使用**程序数据库**（**pdb**）文件调试安装过程的功能。这些文件以 ZIP 分发方式提供：

+   **安装程序包**：这是一个基于向导的过程，易于使用。安装程序包仅适用于 32 位，但也可以在 64 位配置上安装 MySQL 8。它不包含 MYSQL 的调试组件；我们必须单独下载 ZIP 文件。安装程序包有两种不同的格式：

+   **Web Community**：顾名思义，这是用于 Web 安装的。这意味着需要互联网才能使用 Web 社区进行安装。其大小约为 19 MB。其名称定义为通过附加版本的 MySQL-installer-community。

+   **社区**：此软件包格式用于离线安装。其大小约为 301 MB。其名称定义为通过附加版本的 MySQL-installer-web-community。

安装程序是 MySQL 产品安装和升级的最常见方式。

+   **Noinstall Archives**：这是一个包含不完整安装包文件的手动安装过程。由于这是一个手动过程，因此没有 GUI 可用。用户必须手动安装和配置 MySQL 8 和其他产品（如果需要）。与安装程序不同，它以 ZIP 格式为 32 位和 64 位配置提供两个不同的文件。

# MySQL 8 安装程序

MySQL 8 安装程序主要用于简化安装过程以及在 Windows 平台上运行的 MySQL 产品的管理。在产品列表中，我们可以考虑：

+   MySQL 服务器

+   MySQL 应用程序

+   MySQL 连接器

+   文档和示例

MySQL 8 安装程序有两个版本：

+   **社区版**：可在[`dev.mysql.com/downloads/installer/`](http://dev.mysql.com/downloads/installer/)下载。如前所述，安装程序提供 Web 社区和社区包格式。

+   **商业版**：请参阅[`edelivery.oracle.com/`](https://edelivery.oracle.com/)下载商业版。商业版包含社区版中可用的所有产品以及以下产品：

+   Workbench SE/EE

+   MySQL 企业备份

+   MySQL 企业防火墙

# 初始设置信息

如前所述，安装程序将引导用户通过向导。一旦我们在主机机器上启动安装程序，它将检测已安装的 MySQL 产品，并将其列入要管理的产品列表。以下是安装程序初始设置中所需的步骤：

1.  **MySQL 安装程序许可和支持身份验证**：这是用户必须在开始 MySQL 8 安装之前接受许可协议的步骤。接受条款后，用户可以添加、更新或删除 MySQL 产品。在商业版中，需要凭据来解除捆绑产品，并且必须与用户在支持站点的 Oracle 帐户匹配。

1.  **选择设置类型**：这是用户必须选择要安装的 MySQL 产品的步骤。安装程序还提供了预定义设置的选项，其中包含一组 MySQL 产品。因此，您可以根据自己的需求选择一个设置类型。以下是安装程序中可用的一些设置。

1.  **开发者默认**：这将安装在下载时选择的 MySQL 8 服务器版本：

+   MySQL 服务器

+   MySQL shell

+   MySQL 路由器

+   MySQL Workbench

+   MySQL for Visual Studio

+   MySQL for Excel

+   MySQL 通知程序

+   MySQL 连接器

+   MySQL 实用程序

+   MySQL 文档

+   MySQL 示例和示例

1.  **仅服务器**：这只安装 MySQL 服务器。

1.  **仅客户端**：这与开发者默认设置类型相同，只是不包含 MySQL 8 服务器或任何特定于客户端的包。

1.  **完整**：这将安装 MySQL 的所有可用产品，如`mysql-server`，`mysql-client`，`mysqladmin`等。

1.  **自定义**：此选项仅安装用户从目录中选择的产品。在这里，用户可以自由选择所需的产品，而不是安装完整的产品包。

1.  **路径冲突**：当托管系统已经包含一个 MySQL 产品，并且用户试图在相同路径上安装该 MySQL 产品的不同版本时，安装程序将在向导中显示路径冲突错误。安装程序使用户能够以以下方式处理路径冲突：

+   使用向导中的浏览按钮选择不同的位置

+   通过自定义选择选择不同的设置类型或版本

+   通过继续到下一步覆盖现有文件夹

+   取消向导步骤，删除现有产品，然后重新启动安装程序

1.  **检查要求**：每个 MySQL 产品都附有一个`package-rules.xml`文件，其中包含所有先决条件软件列表。在初始设置期间，安装程序将检查所需软件的可用性，并在缺少要求时提示用户更新主机。

1.  **MySQL 安装程序配置文件**：安装程序配置文件位于`C:\Program Files`。以下是配置文件的详细信息：

| **文件或文件夹** | **描述** | **文件夹层次结构** |
| --- | --- | --- |
| Windows 的 MySQL 安装程序 | 此文件夹包含运行 MySQL 安装程序和[MySQLinstallerConsole.exe](https://dev.mysql.com/doc/refman/8.0/en/MySQLInstallerConsole.html)所需的所有文件，这是一个具有类似功能的命令行程序。 | `C:\Program Files (x86)` |
| `Templates` | `Templates`文件夹中有每个 MySQL 服务器版本的一个文件。`Templates`文件包含用于动态计算一些值的键和公式。 | `C:\ProgramData\MySQL\MySQL installer for Windows\Manifest` |
| `package-rules.xml` | 此文件包含要安装的每个产品的先决条件。 | `C:\ProgramData\MySQL\MySQL installer for Windows\Manifest` |
| `produts.xml` | `products`文件（或产品目录）包含可供下载的所有产品的列表。 | `C:\ProgramData\MySQL\MySQL installer for Windows\Manifest` |
| `Product Cache` | `Product Cache`文件夹包含与完整包或随后下载的所有独立`MSI`文件捆绑在一起。 | `C:\ProgramData\MySQL\MySQL installer for Windows` |

参考：[`dev.mysql.com/doc/refman/8.0/en/mysql-installer-setup.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-installer-setup.html)

# 安装工作流程

MySQL 安装程序遵循每个产品的工作流程：

1.  **产品下载**：安装程序将所有必需的产品 MSI 文件下载到`Product Cache`文件夹中。

1.  **产品安装**：安装程序通过 Ready | Install | Installing | Complete 来管理每个产品的状态。

1.  **产品配置**：此阶段使用逐步配置过程来配置产品。安装程序将状态从 Ready | Configure 更改。

1.  **安装完成**：完成安装后，用户可以开始使用该应用程序。

# InnoDB 集群沙盒测试设置

在 MySQL 8 中，使用安装程序有两种高可用性实现选项：

+   **独立的 MySQL 服务器/经典的 MySQL 复制**（**默认**）：此选项手动配置多个服务器，或使用最新版本的 MySQL Shell 配置`InnoDB`集群。

+   **InnoDB 集群沙盒测试设置**（**仅用于测试**）：这也被称为沙盒集群。此选项允许您仅在本地系统上为测试目的创建`InnoDB`集群。MySQL 安装程序工具栏提供了一些`InnoDB`集群中实例的配置。

集群节点在不同的端口上运行。配置完成后，单击摘要选项卡以获取每个集群的端口详细信息。

# 服务器配置

MySQL 安装程序执行一些基本配置，包括：

+   安装程序将为 MySQL 8 服务器创建一个`my.ini`配置文件。文件内容将根据安装过程中选择的选项确定。

+   默认情况下，安装程序将为 MySQL 8 服务器添加 Windows 服务。

+   安装程序将提供 MySQL 8 服务器的默认安装和数据路径。

+   安装程序将为 MySQL 8 服务器创建一些带有角色和权限的用户帐户。它可以创建一个具有有限特权的 Windows 用户到 MySQL 8 服务器。

+   使用显示高级选项，MySQL 安装程序允许定义日志选项的自定义路径。例如，您可以配置单独的错误日志路径，显示查询日志等。

MySQL 8 需要以下服务器配置：

+   **服务器配置类型**：根据服务器配置类型，系统资源将分配给 MySQL 8 服务器。

+   **开发**：通过将主机视为个人工作站，它配置 MySQL 8 以使用最少量的内存。

+   **服务器**：作为服务器，机器上还运行着一些其他应用程序，因此它将配置中等数量的内存。

+   **专用**：对于专用于 MySQL 8 服务器的专用机器，此选项配置了可用内存的最大使用。

+   **连接性**：此选项指示与 MySQL 8 服务器的连接。以下选项可用：

+   **TCP/IP**：此选项启用与 MySQL 8 的 TCP/IP 连接。用户可以定义端口号以及网络访问端口的防火墙设置。

+   **命名管道**：此选项允许您为连接定义管道名称。

+   **共享内存**：这允许您为 MySQL 8 服务器定义内存名称。

+   **高级配置**：此配置启用了额外的日志记录功能，将管理日志记录在单独的文件中。用户可以配置单独文件的路径。例如，配置二进制日志的自定义路径。

+   **MySQL 企业防火墙**：此选项仅适用于商业版。勾选**启用企业防火墙**选项以启用防火墙。

+   **帐户和角色**：帐户和角色用于管理用户的访问权限。在安装过程中，MySQL 安装程序允许您设置根帐户密码和用户帐户。

+   **根帐户密码**：在安装过程中需要输入根密码。安装程序将检查密码强度，并在违反预定义策略时发出警告。

+   **MySQL 用户帐户**：这是一个可选步骤，其中定义了一个新的 MySQL 用户帐户，并具有现有用户角色。预定义角色有其自己的权限。

+   **Windows 服务**：MySQL 8 服务可以以下两种方式进行配置：

+   **配置为 Windows 服务**：这是安装过程中默认选择的选项。它进一步提供了两个选项：

+   **系统启动时启动服务**：此选项默认选中，将在系统启动时自动启动 MySQL 8 服务。

+   **作为 Window 服务运行**：此选项允许将用户帐户与 MySQL 8 服务关联。默认情况下选择系统帐户，其中服务被视为网络服务。使用自定义用户，首先使用 Microsoft Windows 中的“本地安全策略”为用户设置权限。

+   **配置为可执行程序**：在安装过程中取消选择 Windows 服务选项。

+   **插件和扩展**：此步骤适用于新安装。如果用户希望从较旧的 MySQL 版本升级，则用户需要在 MySQL 安装程序中选择重新配置选项。

+   **高级选项**：要启用此选项，请在**类型和网络**步骤中选中**显示高级配置**复选框。此选项允许用户定义特定路径的日志文件，例如错误日志、常规日志、慢查询日志和二进制日志。

+   **应用服务器配置**：用户在 MySQL 安装程序中完成所有配置后，单击“执行”按钮使其可用。安装完成后，单击“完成”按钮，MySQL 安装程序和所有已安装的 MySQL 产品将在 Windows“开始”菜单中可用。

# MySQL 安装程序产品目录和仪表板

本节包含有关**MySQL 安装程序**如何处理产品目录和管理仪表板的详细信息。

**产品目录**是一个组件，其中包含所有发布的支持 Microsoft Windows 的 MySQL 产品列表。MySQL 安装程序每天更新目录，并提供手动更新目录的选项。产品目录执行以下操作来管理列表：

+   定期更新可用产品列表

+   检查在主机中安装的产品的更新

产品目录列出了所有开发、一般或任何次要版本中可用的产品。

**MySQL 安装程序仪表板**提供了在主机工作站中管理 MySQL 产品安装的功能。以下是使用仪表板管理产品的方法：

+   MySQL 安装程序提供了一个配置，可以在特定时间间隔更新目录。用户可以通过配置启用或禁用自动更新。仪表板在产品级别显示一个特殊图标，表示其新版本已经可用。

+   用户可以使用以下操作来管理产品：

+   **添加**：用于下载和安装一个或多个产品。

+   **修改**：用于在已安装产品中添加或删除功能。

+   **升级**：用于升级产品。确保在**可升级产品**窗格中产品级别的复选框被选中。

+   **删除**：用于从已填充列表中卸载产品。

+   仪表板提供了重新配置功能，用户可以更改已配置的选项和值。应用更改后，MySQL 安装程序将停止 MySQL 8 服务器，然后重新启动以使更改生效。

+   仪表板提供了下载产品目录而不进行升级的功能。**此时不更新**复选框可用于在不下载的情况下检查与产品相关的当前更改。要执行此功能，请选择复选框，然后单击**目录**链接。

# MySQL 安装程序控制台

MySQL 安装程序包括`**M**ySQLinstallerConsole.exe`文件，它提供了使用命令提示符执行命令的功能。这个功能在安装 MySQL 安装程序时默认安装。有一些命令可用于管理 MySQL 产品。要查看这些命令的详细信息，请执行`help`命令。

# 使用 ZIP 文件安装 MySQL 8

要使用 ZIP 存档安装 MySQL 8，请执行以下步骤：

1.  **提取安装存档**：在此步骤中，选择一个特定目录来提取已安装的 MySQL 8 存档。Microsoft Windows 默认将其安装到`c:\mysql`位置。在安装过程中，请确保登录用户具有管理员权限。

确保在安装 MySQL 作为 Windows 服务时没有运行任何 MySQL 服务。

在执行命令时，首先输入-- install，然后指定--default –file 选项；否则，`mysqld.exe`文件将启动 MySQL 8 服务器。

1.  **创建选项文件**：选项文件是用户可以配置与 MySQL 8 相关命令的地方。MySQL 8 在每次服务器启动时都会引用这个文件。在 Microsoft Windows 中，当 MySQL 8 服务器启动时，它会在 Windows 目录和 MySQL 8 基本目录中搜索选项文件。主要使用`my.ini`和`my.cnf`文件作为选项文件，它们是以纯文本形式存在的。

使用 Windows 操作系统：

1.  要获取`Windows`目录，请参考`WINDIR`环境变量。

1.  `my.ini`位于 MySQL 服务器安装的默认位置。

1.  选项文件中的路径名使用正斜杠而不是反斜杠指定。如果要使用反斜杠，则需要双反斜杠。

如前所述，选项文件与普通文本文件相同；用户可以使用任何文本编辑器修改它。考虑一个例子，MySQL 8 安装目录和数据目录位于不同位置。在这种情况下，用户必须在`mysqld`部分的选项文件中提及两个目录的位置：

```go
 [mysqld]
 # set basedir to your installation path
 basedir=E:\\mysql
 # set datadir to the location of your data directory
 datadir=E:\\mydata\\data
```

请记住，ZIP 存档不会初始化数据目录。要初始化数据目录并在 MySQL 8 系统数据库中填充表，用户必须执行`initialize`命令。这个过程将在后面的后安装部分中介绍。

1.  **选择服务器类型**：在 MySQL 8 中，为 Microsoft Windows 提供了以下两种服务器：

| **二进制** | **描述** |
| --- | --- |
| [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) | 带有命名管道支持的优化二进制文件 |
| [**mysqld-debug**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) | 类似于[mysqld](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html)，但是编译时带有完整的调试和自动内存分配检查 |

MySQL 8 在所有 Microsoft Windows 平台上都支持 TCP/IP 和管道。但默认选项是正常模式，因为管道选项会影响性能。它会减慢整体性能。您可以在此处了解更多信息：[`dev.mysql.com/doc/refman/8.0/en/windows-select-server.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-select-server.html)。

1.  **启动 MySQL 8 服务器**：此步骤描述了如何首次启动 MySQL 8 服务器。可以使用命令行或作为 Windows 服务启动。要使用命令行启动它，请执行以下命令。假设 MySQL 8 安装在`E:\MySQL\MySQL Server 8`文件夹下：

```go
 E:\> “E:\MySQL\MySQL Server 8\bin\mysqld”
```

在执行上述命令后，用户可以看到一系列消息，帮助用户识别错误（如果存在）。命令提示符中的最后两行显示如下。它们表明 MySQL 8 服务已启动，并准备好接受服务器-客户端请求：

```go
 mysqld: ready for connections
 Version: '8.0.4' socket: '' port: 3306
```

用户可以省略控制台以获取错误日志，因为 MySQL 8 会在数据目录下的单独日志文件中维护日志，扩展名为`.err`。启动 MySQL 8 服务器时，用户必须每次使用命令提示符执行相同的命令。要停止 MySQL 8 服务器，请执行以下命令：

```go
 E:\> “E:\MySQL\MySQL Server 8\bin\mysqladmin” -u root shutdown
```

最初安装 MySQL 8 时，根帐户密码未设置。首次启动 MySQL 8 服务器后，用户必须手动设置密码。设置密码的步骤在后续安装后的部分中有详细描述。

`mysqld`和`mysqladmin`的路径可能会根据 MySQL 8 的安装位置而有所不同。如果在 MySQL 8 服务器的根用户上设置了密码，则使用-p 选项执行命令并提供密码以成功执行。

1.  设置 MySQL 8 的环境变量。如前所述，我们已经用 MySQL 8 服务器安装的完整路径编写了命令。为了简化这个命令，在 Windows 10 中，我们必须通过以下步骤定义 MySQL 8 的环境变量：

1.  右键单击**我的电脑**，选择**属性**。

1.  点击“高级系统设置”。

1.  点击右下方的“环境变量”按钮。

1.  转到“系统变量”部分，找到“PATH”变量。

1.  选择“PATH”变量，点击“编辑”按钮。

1.  一个新的对话框打开；点击“新建”按钮，输入 MySQL 8 服务器安装的路径，直到`bin`文件位置。例如，`E:\MySQL\MySQL Server 8\bin`

1.  在 Windows 中按下所有相关对话框中的“确定”按钮以应用此更改。

应用环境变量后，用户可以在命令提示符中执行任何 MySQL 命令，而无需提供 MySQL 8 的完整路径。

1.  **将 MySQL 8 作为 Windows 服务启动**：建议将 MySQL 8 作为 Windows 服务使用，因为它将在 Windows 启动时启动，并在 Windows 停止时停止。在这种情况下，无需显式启动 MySQL 8 服务。使用 Microsoft Windows 服务实用程序来管理 MySQL 8 服务。要将 MySQL 服务器安装为服务，请使用以下命令：

```go
 E:\> “E:\MySQL\MySQL Server 8\bin\mysqld” --install
```

此命令有以下选项用于附加参数：

+   如果我们没有定义服务名称，那么该命令将考虑默认服务为**MySQL**。

+   使用`-- default -file`参数来指定包含服务名称的选项文件名

+   用户还可以使用`--local-service`选项，后面跟着服务名称

使用以下命令将 MySQL 服务设置为 Windows 服务，并引用文件`my-opts.conf`作为 MySQL 8 配置的选项文件：

```go
E:\> “E:\MySQL\MySQL Server 8\bin\mysqld” --install MySQL --defaults-file=E:\my-opts.cnf
```

到目前为止，我们已经讨论了 MySQL 8 服务作为 Windows 服务，但是还有一个带有`--install-manual`选项的命令可用于启动 MySQL 8 而不使用 Windows 服务。要删除 MySQL 8 服务，请使用`--remove`命令，如下所示：

```go
E:\> “E:\MySQL\MySQL Server 8\bin\mysqld” --install-manual
E:\> "E:\MySQL\MySQL Server 8\bin\mysqld" --remove
```

# 在 Linux 上安装 MySQL 8

对于 Linux 上的 MySQL 8 安装，有各种解决方案可用。用户可以根据自己的需求选择任何发行版。以下是不同的发行版，其中有三种进行了详细描述：

+   使用`Yum`仓库进行安装

+   使用`APT`仓库进行安装

+   使用`SLES`仓库进行安装

+   使用`RPM`软件包进行安装

+   使用`Debian`软件包进行安装

+   使用 Docker 进行安装

+   使用本地软件仓库进行安装

+   使用 Juju 进行安装

# 使用 Yum 仓库进行安装

执行以下步骤以使用`Yum`仓库安装 MySQL 8:

1.  从[`dev.mysql.com/downloads/repo/yum/`](http://dev.mysql.com/downloads/repo/yum/)链接下载`Yum`仓库。

1.  选择适合您安装的 MySQL 软件包。

1.  执行安装命令将 MySQL `Yum`仓库添加到您的`local`仓库中：

```go
 shell> sudo yum localinstall package_name.rpm
 shell> yum repolist enabled | grep "mysql.*-community.*"
```

用实际的 RPM 软件包名称替换`package_name`。安装后，执行第二个命令以检查`Yum`仓库是否已正确安装。

1.  **选择发布系列**：MySQL Yum 仓库包含各种发布系列供安装。执行以下命令以检查可用系列的列表：

```go
 shell> yum repolist all | grep mysql
```

在 MySQL Yum 仓库中，默认情况下启用了最新的 GA 系列进行安装，但除此之外，其他开发系列也以禁用状态可用。对于开发系列的安装，请执行以下两个命令以禁用 GA 发布并启用所需的开发系列：

```go
 shell> sudo yum-config-manager --disable mysql57-community
 shell> sudo yum-config-manager --enable mysql80-community
```

定义发布系列的另一种方式是在仓库文件中使用手动条目。例如，将以下条目添加到`/etc/yum.repos.d/mysql-community.repo`文件中：

```go
 mysql80-community]
 name=MySQL 8.0 Community Server
 baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/6/$basearch/
 enabled=1
```

在这里，`enabled=1`表示启用此系列，`enabled=0`表示禁用此系列。Yum 一次只允许一个启用的子仓库用于一个发布；如果启用了多个发布系列，则 Yum 仓库将只选择最新的系列。在保存配置文件中的更改后，执行以下命令以检查是否已选择正确的子仓库：

```go
 shell> yum repolist enabled | grep mysql
```

1.  **安装 MySQL 8**：执行以下命令：

```go
 shell> sudo yum install mysql-community-server
```

它将安装带有所有依赖项的 MySQL 8 服务器，例如客户端、客户端的字符集和所需的库。

1.  **启动 MySQL 8 服务器**：第一个命令将启动 MySQL 服务，第二个命令显示 MySQL 服务的当前状态：

```go
 shell> sudo service mysqld start 
 shell> sudo service mysqld status
```

在初始启动期间，执行以下任务：

+   服务器已初始化

+   SSL 证书和密钥文件在数据目录中生成

+   插件名称`validate_password_plugin`已安装并已启用

+   创建了一个超级账户

# 使用 RPM 软件包进行安装

RPM 软件包可从 Yum 仓库和 SLES 仓库获取，用于 MySQL 8。这是 MySQL 8 安装的推荐方式。RPM 软件包遵循`packagename-version-distribution-arch.rpm`的语法，其中 distribution 和 arch 分别表示 Linux 发行版和处理器。`RPM`软件包是所有必需软件包的捆绑包，它们彼此依赖。RPM 软件包遵循与 Yum 仓库安装中讨论的相同步骤。在基于 RPM 的系统中，MySQL 服务不会自动启动。要手动启动它，请执行以下命令：

```go
shell> sudo service mysqld start
```

与其他安装一样，RPM 软件包安装还会在系统上创建文件和目录，路径如下：

| **文件或资源** | **位置** |
| --- | --- |
| 客户端程序和脚本 | `/usr/bin` |
| - [**mysqld**](https://dev.mysql.com/doc/refman/8.0/en/mysqld.html) 服务器 | `/usr/sbin` |
| - 配置文件 | `/etc/my.cnf` |
| - 数据目录 | `/var/lib/mysql` |
| - 错误日志文件 | 对于 RHEL、Oracle Linux、CentOS 或 Fedora 平台：`/var/log/mysqld.log` 对于 SLES：`/var/log/mysql/mysqld.log` |
| - [secure_file_priv](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_secure_file_priv) 的值 | `/var/lib/mysql-files` |
| - System V `init` 脚本 | 对于 RHEL、Oracle Linux、CentOS 或 Fedora 平台：`/etc/init.d/mysqld` 对于 SLES：`/etc/init.d/mysql` |
| - Systemd 服务 | 对于 RHEL、Oracle Linux、CentOS 或 Fedora 平台：`mysqld` 对于 SLES：`mysql` |
| - Pid 文件 | `/var/run/mysql/mysqld.pid` |
| - `Socket` | `/var/lib/mysql/mysql.sock` |
| - 密钥环目录 | `/var/lib/mysql-keyring` |
| - Unix 手册页 | `/usr/share/man` |
| - 包括（`header`）文件 | `/usr/include/mysql` |
| - 库 | `/usr/lib/mysql` |
| - 其他支持文件（例如错误消息和字符集文件） | `/usr/share/mysql` |

# 使用 Debian 包进行安装

MySQL APT 仓库或 MySQL Developer Zone 的下载区域提供了 `Debian` 包。每个 MySQL 组件都有自己的 `Debian` 包进行安装，但也准备了一个 tarball 捆绑包，将不同的 `Debian` 包组合成一个单独的捆绑包。tarball 命名约定为 `mysql-server_MVER-DVER_CPU.deb-bundle.tar`，其中 `MVER` 表示 MySQL 版本，`DVER` 表示 Linux 发行版版本，CPU 表示处理器。要使用 `Debian` 包安装 MySQL 8，请执行以下步骤：

1.  从 MySQL 站点下载所需的 tar 包。

1.  使用以下命令解压包：

```go
 shell> tar -xvf mysql-server_MVER-DVER_CPU.deb-bundle.tar
```

1.  可能需要 `Libaio1` 库，因此执行库安装命令：

```go
 shell> sudo apt-get install libaio1
```

1.  执行以下命令对 MySQL 服务器进行预配置：

```go
 shell> sudo dpkg-preconfigure mysql-community-server_*.deb
```

这是在安装前实施的进程，适用于一组 `Debian` 包和其他使用 `debconf` 的配置脚本来检查系统的包。在此过程中，系统将要求输入 MySQL 8 安装的 root 用户密码。

1.  安装 MySQL 8 所需的依赖项：

```go
 shell>sudo apt-get -f install
```

MySQL 8 配置文件在 `Debian` 包中的以下路径下可用：

+   配置文件存储在 `/etc/mysql` 下

+   数据目录存储在 `/var/lib/mysql` 下

+   二进制文件、库和头文件存储在 `/user/bin` 和 `user/sbin` 下

# MySQL 8 的后安装设置

后安装是描述用户在 MySQL 8 安装后必须执行的基本步骤或配置的过程。

# 数据目录初始化

在前面的部分中，我们已经看到了 MySQL 8 安装的不同方法。其中一些方法将自动为 MySQL 8 创建数据目录。对于通用二进制分发和源分发，必须创建数据目录。数据目录初始化可以通过以下两个命令之一执行：

```go
E:\> bin\mysqld –-initialize 
E:\> bin\mysqld --initialize-insecure
```

可以根据用户的需求选择这两个命令中的任何一个来生成一个随机的初始密码。无论数据目录初始化的平台如何，用户都可以使用这两个命令。初始化是一种生成随机初始 root 密码的方法。在 `initialize-insecure` 的情况下，不会生成密码。

另一个选项是使用命令行参数指定安装目录和数据目录，如下命令所示：

```go
E:\> bin\mysqld --initialize --basedir E:\mysql --datadir :\mydata\data
```

用户还可以在名为 `Option` 文件的单独文件中指定这些目录，该文件位于 `mysqld` 参数下。此配置在本章的 `Option` 文件部分中有详细描述。当用户执行这两个命令中的任何一个时，`mysqld` 将执行以下步骤：

1.  它检查数据目录的存在

1.  MySQL 8 服务器将创建系统数据库及其表、授予表、帮助表和时区表

1.  它使用`InnoDB`表的数据结构初始化系统表空间

1.  将创建`root@localhost`超级用户账户和其他保留账户

# 保护初始的 MySQL 账户

本节解释了在工作站上首次执行 MySQL 8 服务器时如何为 root 账户分配密码。当用户在 Windows 中使用安装程序或在 Linux 中使用`Debian`软件包安装 MySQL 8 时，安装过程提供了输入密码并将该密码分配给 root 账户的选项。但是使用`RPM`软件包时，将为 root 账户生成随机密码，该密码在安装过程中写入服务器日志文件。初始 root 账户可能有也可能没有密码。要在初始阶段分配密码，请使用以下任一程序：

+   如果 root 账户有随机密码：

1.  查看服务器日志文件以获取自动生成的密码。

1.  通过执行以下命令连接到 MySQL 8 服务器，使用自动生成的密码：

```go
 shell> mysql -u root -p
 Enter password: (enter the random root password here) 
```

+   为`root`账户设置新密码：

```go
 mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'newPassword';
```

+   如果 root 账户没有密码：

1.  无需密码连接到 MySQL 8 服务器：

```go
 mysql -u root
```

2. 为`root`账户设置新密码：

```go
 mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'newPassword';
```

分配新密码后，每当您想要连接到 MySQL 8 时，都必须使用新密码。

# MySQL 8 服务的启动和故障排除

本节解释了如何启动 MySQL 8 服务器以及如何在启动过程中解决问题。在 Linux 系统上成功安装 MySQL 8 服务器后，执行以下命令启动 MySQL 8 服务：

```go
shell> sudo service mysqld start
```

要详细检查服务是否已启动，请参考日志文件。您可以使用 status 命令来检查 MySQL 8 服务的状态：

```go
shell> sudo service mysqld status
```

在收到服务启动/运行消息后，您可以使用以下命令连接到 MySQL 8：

```go
shell> mysql -uroot -p
```

上述命令会提示输入密码，因此输入密码并按下*Enter*键。MySQL 提示将显示在那里，您可以输入`mysql`命令进行执行。在执行上述命令时，可能会出现一些常见问题，因此在这里我们将提出以下故障排除建议：

1.  查看`log`文件以找到服务启动期间发生的确切错误。如前一节所述，`error`文件和`log`文件位于数据目录下。它们的命名约定为`host_name.err`和`host_name.log`。通过阅读文件的最后几行，您可以确定在上次执行的命令期间发生的问题。

1.  检查所需的端口/套接字是否可用。以下错误将表明所需的端口和套接字不可用，意味着它们正在被其他程序使用。要识别这一点，通过禁用服务来跟踪所有问题。另一个原因是防火墙设置阻止了对所需端口的访问，因此修改防火墙设置并允许对所需端口的访问：

+   **无法启动服务器**：在 TCP/IP 端口上绑定：地址已在使用中

+   **无法启动服务器**：在 Unix 套接字上绑定

1.  建议将特定参数定义到`Option`文件中。如果参数未定义到文件中，则 MySQL 8 将考虑默认参数，因此在使用之前，请参考 MySQL 8 提供的所有可用参数。

1.  验证`data`目录路径和权限是否正确定义。该目录用作 MySQL 8 的当前目录。要查找数据目录的当前设置路径，请使用`mysqld`命令执行`--verbose`和`--help`命令。如果`data`目录位于 MySQL 安装目录以外的其他位置，则使用`mysqld`命令的`--datadir`选项。对于权限，您将收到错误代码 13，表示权限被拒绝的错误。要解决此问题，请更改所需文件和文件夹的权限。另一种方法是使用 root 用户登录，但这在所有情况下都不可能，因此建议首先采用第一种方法来解决权限问题。

# 执行命令以测试服务器

执行上述步骤后，现在您的 MySQL 8 服务已启动，并已连接到指定用户。现在，通过执行以下基本命令来检查您的 MySQL 8 服务器是否正常工作：

```go
shell> bin/mysqladmin version
```

该命令列出了与安装的 MySQL 服务器相关的所有信息，包括其版本详细信息、协议版本等。连接到 MySQL 8 后，执行以下命令以检查是否已正确从服务器检索到信息：

```go
mysql>mysqlshow
mysql>mysqlshow mysql
```

第一个命令显示服务器中可用的数据库列表。列表可能因系统而异，但`mysql`和`information_schema`必须在列表中可用。第二个命令列出了在`mysql`数据库下创建的所有表。

# 升级 MySQL 8

在以前的 MySQL 版本中，数据字典存储在基于文件的系统中，而在 MySQL 8 中，它存储在数据字典结构中。因此，升级过程将文件式结构移动到数据字典结构中。从 MYSQL 5.7 GA 版本（即从 5.7.9 或更高版本）可以升级到 MySQL 8。对于 5.7 的非 GA 版本，无法进行升级。在开始升级过程之前，需要了解以下几点。

# 升级方法

有两种升级方法，它们通过其实施方法进行区分。让我们详细讨论这些方法。

# MySQL 的就地升级

顾名思义，这是一个过程，我们将 MySQL 的现有旧版本包替换为更新版本。在开始之前，请确保旧服务器已停止，并在替换包后，使用 MySQL 升级重新启动现有数据目录上的 MySQL 8 服务器。

执行以下步骤进行就地升级：

1.  使用以下命令对加密的`InnoDB`表空间进行主密钥旋转：

```go
 ALTER INSTANCE ROTATE INNODB MASTER KEY;
```

1.  使用`innodb_fast_shutdown`命令配置关闭参数：

```go
 SET GLOBAL innodb_fast_shutdown = 1; -- fast shutdown
 SET GLOBAL innodb_fast_shutdown = 0; -- slow shutdown
```

1.  使用以下命令关闭旧的 MySQL 版本：

```go
 mysqladmin -u root -p shutdown
```

1.  这是用户将旧包替换为新 MySQL 8 包的升级过程。

1.  使用现有的`data`目录启动 MySQL 8 服务器。

在服务器启动时，它将检查`data`字典表。如果不存在，则服务器将在`data`目录中创建表，并使用其正常启动顺序填充元数据和进程。如果这些步骤成功执行，服务器将通过创建`backup_metadata_57`目录来执行清理。此外，服务器将事件和 proc 表重命名为`event_backup_57`和`proc_backup_57`。如果此步骤失败，则服务器将恢复所有更改。

1.  成功完成 MySQL 8 启动后，执行`mysql_upgrade`：

```go
 mysql_upgrade -u root -p
```

1.  升级后，关闭并重新启动服务器，以检查是否已应用所有更改。

# MySQL 8 的逻辑升级

导出或获取旧的 MySQL 版本的转储。安装新的 MySQL 8 版本，并使用 MySQL 升级将转储文件加载到新的 MySQL 8 版本。执行以下步骤来应用逻辑升级：

1.  使用`mysqldump`命令导出数据：

```go
 mysqldump -u root -p --add-drop-table --routines --events --all-databases   --
          force > data-for-upgrade.sql
```

`--routines`[ ](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html#option_mysqldump_routines)和`--events`选项用于在转储文件中包含存储例程和事件，并明确定义这些选项以产生效果。

1.  关闭旧的 MySQL 服务器。

1.  安装新版本的 MySQL 8。

1.  初始化`data`目录：

```go
 mysqld --initialize --datadir=/path/to/8.0-datadir
```

1.  使用新的`data`目录启动 MySQL 8 服务器：

```go
 mysqld_safe --user=mysql --datadir=/path/to/8.0-datadir 
```

1.  将 SQL 转储文件加载到新的`MySQL`数据库中：

```go
 mysql -u root -p --force < data-for-upgrade.sql
```

1.  使用以下命令升级 MySQL：

```go
 mysql_upgrade -u root -p
```

# MySQL 5.7 的升级先决条件

在开始升级之前，执行以下检查以避免后期失败：

1.  执行以下命令以检查是否存在绝对数据类型或函数：

```go
 mysqlcheck -u root -p--all-databases--check-upgrade
```

1.  使用以下命令检查本机分区支持：

```go
 SELECT TABLE_SCHEMA, TABLE_NAME
 FROM INFORMATION_SCHEMA.TABLES
 WHERE ENGINE NOT IN ('innodb', 'ndbcluster')
 AND CREATE_OPTIONS LIKE '%partitioned%';
```

此命令将列出使用不支持本机分区的存储引擎的表。在执行前述查询后，如果发现任何表，则删除表上的分区并更改存储引擎，如以下命令所示：

```go
 ALTER TABLE table_name ENGINE = INNODB;
 ALTER TABLE table_name REMOVE PARTITIONING;
```

1.  确保 MySQL 5.7 中不包含在 MySQL 8 中用作数据字典的任何表。

1.  使用以下代码检查外键约束名称是否不超过 64 个字符：

```go
 SELECT CONSTRAINT_SCHEMA, TABLE_NAME, CONSTRAINT_NAME
 FROM INFORMATION_SCHEMA.REFERENTIAL_CONSTRAINTS
 WHERE LENGTH(CONSTRAINT_NAME) > 64;
```

1.  确保 MySQL 5.7 不包含 MySQL 8 中不可用的功能，例如：

+   如果表使用了 MySQL 8 不支持的存储引擎，则已更改为支持的存储引擎

+   使用 MySQL 8 中不可用的选项或变量进行配置更改

# MySQL 8 降级

降级是从升级的反向过程，我们将从 MySQL 的较高版本降级到 MySQL 的较低版本。在本节中，我们将介绍如何从 MySQL 8 降级到 MySQL 5.7。不支持版本跳过的降级意味着不支持从 MySQL 8 降级到 MySQL 5.6。在支持版本跳过的同一系列中，意味着您可以跳过 MySQL 8.y 版本，从 MySQL 8.z 降级到 MySQL 8.x。首先，我们将解释在开始降级之前需要理解的一些基本要点。

# 降级方法

原地降级意味着关闭 MySQL 8 的新版本，用旧版本的 MySQL 替换其二进制文件或软件包。重新启动旧版本意味着在现有数据目录中的 MySQL 5.7。此降级方法在同一系列的 GA 版本之间得到支持。执行以下步骤进行原地降级：

1.  关闭较新版本的 MySQL 8。

1.  关闭后，从`data`目录中删除`InnoDB`重做日志文件，以避免降级问题：

```go
 rm ib_logfile*
```

1.  将旧版本的 MySQL 定位到新版本的二进制文件或软件包的位置。

1.  通过指定`data`目录启动降级版本的 MySQL，使用以下命令：

```go
 mysqld_safe --user=mysql --datadir=/path/to/existing-datadir
```

1.  执行`mysql_upgrade`命令：

```go
 mysql_upgrade -u root -p
```

1.  关闭并重新启动 MySQL 服务器，以检查所有更改是否已应用。

对于基于 APT、SLES 和 Yum 存储库安装的 MySQL 安装，不支持原地降级

# 逻辑降级

使用新版本中的`mysqldump`对所有表进行转储。安装新版本的 MySQL 8，并将旧版本的转储加载到新数据库中。此降级也支持在同一 GA 发布系列和发布级别中。使用*逻辑降级*方法支持 MySQL 8.0 到 5.7 的降级。

要执行逻辑降级，请按照以下步骤操作：

1.  使用以下代码对数据库进行转储：

```go
 mysqldump -u root -p --add-drop-table --routines --events
 --all-databases --force > data-for-downgrade.sql

```

1.  关闭 MySQL 服务器，如下所示：

```go
 mysqladmin -u root -p shutdown
```

1.  将新的`data`目录初始化为旧的 MySQL 版本，使用以下代码：

```go
 mysqld --initialize --user=mysql
```

1.  使用新的`data`目录启动旧版 MySQL，使用以下代码：

```go
 mysqld_safe --user=mysql --datadir=/path/to/new-datadir
```

1.  将转储加载到旧的 MySQL 服务器中，如下所示：

```go
 mysql -u root -p --force < data-for-upgrade.sql
```

1.  执行`mysql_upgrade`：

```go
 mysql_upgrade -u root -p
```

1.  重新启动服务器以应用所有更改，使用以下代码：

```go
 mysqladmin -u root -p shutdown
 mysqld_safe --user=mysql --datadir=/path/to/new-datadir
```

# 降级前需要手动更改

本节描述了在降级之前用户需要手动执行的一些更改：

+   **系统表更改**：MySQL 5.7 为系统表管理单独的表空间，而在 MySQL 8 中，系统表迁移到一个名为`mysql.ibd`的单个表空间文件中。因此，在降级到 MySQL 5.7 之前，使用以下命令将系统表移回单独的表空间文件：

```go
 ALTER TABLE mysql.columns_priv TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.component TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.db TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.default_roles TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.engine_cost TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.func TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.general_log TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.global_grants TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.gtid_executed TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.help_category TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.help_keyword TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.help_relation TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.help_topic TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.innodb_index_stats TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.innodb_table_stats TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.plugin TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.procs_priv TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.proxies_priv TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.role_edges TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.server_cost TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.servers TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.slave_master_info TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.slave_relay_log_info TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.slave_worker_info TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.slow_log TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.tables_priv TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.time_zone TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.time_zone_leap_second TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.time_zone_name TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.time_zone_transition TABLESPACE=innodb_file_per_table; 
        ALTER TABLE mysql.time_zone_transition_type TABLESPACE=innodb_file_per_table;          
        ALTER TABLE mysql.user TABLESPACE=innodb_file_per_table;
```

在 MySQL 8.0.2 中，六个系统表的存储引擎从 MyISAM 更改为 InnoDB。它们的名称分别是`columns_priv`、`db`、`procs_priv`、`tables_priv`和`user`。因此，在降级之前，通过执行以下命令更改这些表的存储引擎。对于其余的表也应用相同的命令：

```go
 ALTER TABLE mysql.columns_priv ENGINE='MyISAM' 
          STATS_PERSISTENT=DEFAULT
```

在 MySQL 8.0.2 中，通过添加两个表来更改了`mysql.usertable`，因此，在降级到 MySQL 5.7 之前，从表中删除这些列：

```go
 ALTER TABLE mysql.user drop Create_role_priv;
 ALTER TABLE mysql.user drop Drop_role_priv;
```

+   **InnoDB 更改**：在进行原地降级之前，使用`innodb_fast_shutdown`选项关闭 MySQL。使用`innodb_fast_shutdown=0`关闭服务器。建议在进行原地降级时删除重做日志。

# 摘要

选择适当的软件及其版本进行开发是一个重要的阶段，对吧？在本章中，我们了解了如何通过了解其版本模式来选择 MySQL 8 的适当版本。我们还学习了在 Microsoft Windows 中使用安装程序和命令行安装 MySQL 8 的执行步骤。对于 Linux 平台，我们使用 Yum 存储库、RPM 包和 Debian 包安装了 MySQL 8。安装后描述了开始使用 MySQL 8 的基本配置。最后，我们解释了如何通过执行步骤从 MySQL 8 升级和降级。

在下一章中，我们将学习有关 MySQL 8 可用的各种程序和实用程序。主要关注如何在 MySQL 8 中使用这些程序以及命令行执行。
