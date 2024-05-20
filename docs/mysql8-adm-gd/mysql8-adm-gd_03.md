# 第三章：MySQL 8-使用程序和实用程序

在上一章中，我们安装了 MySQL 8，并了解了安装 MySQL 8 的替代方法。我们还学习了如何迁移和升级到 MySQL 8。以下是上一章中解释的摘要主题：

+   MySQL 8 安装

+   安装后设置

+   MySQL 8 升级

+   MySQL 8 降级

在本章中，读者将了解 MySQL 8 中可用的各种程序和实用程序。读者还将了解如何在 MySQL 8 中使用程序和实用程序。读者将学习在 MySQL 8 中使用命令行程序的方法。读者将学习程序的语法以及它们如何使用特定选项来执行特定操作。以下是本章涵盖的主题摘要。

+   MySQL 8 程序概述

+   MySQL 8 命令行程序

+   MySQL 8 客户端程序

+   MySQL 8 管理程序

+   MySQL 8 环境变量

+   MySQL GUI 工具

# MySQL 8 程序概述

MySQL 安装中有各种不同的程序。本节将简要概述这些程序。接下来的章节将详细描述每个程序，并且描述将有自己的调用语法和选项来执行操作。

大多数 MySQL 发行版都将拥有所有这些程序，除了特定于平台的程序；例如，在 Windows 中不使用服务器启动脚本。RPM（Red-hat 软件包管理器）发行版非常专业化，并且是分发中所有程序的例外。RPM 发行版有何专业之处？嗯，它们为不同的操作有不同的程序；例如，一个程序将用于服务器，第二个程序将用于客户端，依此类推。如果您的安装中似乎缺少一个或多个程序，那就不用担心。请参阅第二章，*安装和升级 MySQL 8*，了解可用的分发类型以及其中包含的内容。也许您的分发不包括所有程序，您需要安装额外的软件包。

每个 MySQL 8 程序都有自己的选项，但大多数程序都有一个`--help`选项，可用于检索有关程序的所有选项的描述。例如，在命令行上尝试`mysql --help`（即您的 shell 或命令提示符）。

第一行的描述将包含有关已安装的 MySQL 的特定版本信息，以及操作系统和许可信息。下一行将以`Usage: mysql [OPTIONS] [database]`开头，即程序命令用法的语法，后面的行描述了可用的选项，以便根据使用说明使用它们。这只是我们将要查看的内容的一瞥：带有选项的程序详细信息，它们的用法和默认选项，以及在各种命令行程序、客户端程序、管理程序等中覆盖默认选项值。

有关在命令行中执行程序和指定程序选项的详细信息，请参阅*MySQL 8 命令行程序*部分*，*随后将列出安装、客户端和服务器启动以及其他实用程序。

# 简要介绍 MySQL 程序

让我们先从 MySQL 服务器程序开始！

`mysqld`是第一个程序，也被认为是 MySQL 安装的主要程序。它与几个脚本一起工作，帮助启动和停止服务器。以下是根据其操作范围划分的程序：

+   启动程序

+   安装/升级程序

+   客户端程序

+   管理和实用程序

# 启动程序

启动程序是在 MySQL 启动期间使用的程序，并根据配置启动所需的后台服务。

+   `mysqld`：这是 MySQL 服务器守护程序。所有其他客户端程序都使用这个服务器程序与数据库交互。除了维护之外，它必须始终处于启动和运行状态。

+   `mysqld_safe`：这是服务器启动程序脚本之一，尝试启动`mysqld`程序。

+   `mysql.server`：另一个服务器启动程序脚本，用于那些使用包含脚本的 V-style 运行目录的系统。它在特定运行级别启动系统服务。它调用`mysqld_safe`来启动 MySQL 服务器。

+   `mysqld_multi`：顾名思义，这是一个启动程序脚本，用于在系统上启动或停止多个 MySQL 服务器。

# 安装/升级程序

以下是安装和升级操作相关的程序及其各自的用法：

+   `comp_err`：这个程序用于从错误源文件编译错误消息文件，并在 MySQL 构建或安装操作期间使用。

+   `mysql_secure_installation`：这个程序用于更新安装 MySQL 时的安全配置，以启用安全性。

+   `mysql_ssl_rsa_setup`：顾名思义，这个程序用于生成`SSL`证书和密钥文件以及`RSA 密钥对`文件，如果这些文件丢失并且需要支持安全连接。

+   `mysql_tzinfo_to_sql`：这个程序获取主机系统时区信息数据库的内容（描述时区的文件），并将信息加载到 MySQL 的时区表中。

+   `mysql_upgrade`：顾名思义，用于升级操作。它检查是否存在任何不兼容性，并在必要时进行修复。它还根据 MySQL 新版本中的任何更改更新授权表。

# 客户端程序

客户端程序是常用于连接到 MySQL 数据库并执行不同查询操作的程序之一：

+   `mysql`：这是最常用的程序。它是一个用于直接执行 SQL 语句或使用批处理模式中的文件的交互式命令行工具。详细信息在下一个*MySQL 8 命令行程序*部分中介绍。

+   `mysqladmin`：这个程序负责执行各种管理操作，如创建或删除数据库、刷新表、重新加载授权表、重新打开日志文件等。该程序还用于从服务器检索信息，如版本、进程和状态。

+   `mysqlcheck`：这是用于维护表、执行分析、检查、修复和优化表的客户端程序。

+   `mysqldump`：这是客户端程序，将 MySQL 数据库转储到文本、SQL 或 XML 格式的文件中。它通常被称为数据库备份程序。

+   `mysqlimport`：这是客户端程序，使用`LOAD_DATA_INFILE`将文本文件导入到相应的表中。它也通常被称为数据导入程序。

+   `mysqlpump`：将 MySQL 数据库转储到 SQL 文件的客户端程序。

+   `mysqlshow`：显示数据库、表、列和索引信息的客户端。

+   `mysqlslap`：这是用于检查 MySQL 服务器客户端负载能力的客户端程序。该程序模拟多个客户端访问服务器。

# 管理和实用程序

以下是执行各种管理活动的程序。它们与一些辅助程序一起描述，这些辅助程序有助于管理操作：

+   `innochecksum`：用于`InnoDB`离线文件校验和的程序。

+   `myisam_ftdump`：提供`MyISAM`表中全文索引信息的实用程序。

+   `myisamchk`：用于检查、描述、修复和优化`MyISAM`表的程序。

+   `myisamlog`：处理`MyISAM`日志文件内容的实用程序。

+   `myisampack`：通过压缩生成更小的只读`MyISAM`表的实用程序。

+   `mysql_config_editor`：可以在加密和安全的登录路径文件`mylogin.cnf`中启用身份验证凭据存储的实用程序。

+   `mysqlbinlog`：可以读取二进制日志文件语句的实用程序。在服务器崩溃的情况下，执行二进制日志文件语句可以提供很大的帮助。

+   `mysqldumpslow`：可以读取和总结慢查询日志内容的实用程序。

# 环境变量

使用以下环境变量的 MySQL 客户端程序与 MySQL 服务器进行通信：

+   `MYSQL_UNIX_PORT`：此变量负责提供用于连接到本地主机的默认 Unix 套接字文件

+   `MYSQL_TCP_PORT`：此变量负责提供默认端口号，并在 TCP/IP 连接中使用

+   `MYSQL_PWD`：此变量负责提供默认密码。

+   `MYSQL_DEBUG`：此变量负责在调试操作期间提供调试跟踪选项

+   `TMPDIR`：此变量负责提供临时文件和表将被创建的目录

有关程序中环境变量的详细列表和用途，请参见*MySQL 8 环境变量*部分。使用`MYSQL_PWD`是不安全的。

# MySQL GUI 工具

由 Oracle 公司提供的 MySQL Workbench GUI 工具用于管理 MySQL 服务器和数据库，用于创建，执行和评估查询。它还用于从其他关系数据库管理系统迁移模式和数据以供与 MySQL 一起使用。还有其他 GUI 工具，包括 MySQL Notifier，MySQL for Excel，phpMyAdmin 等。

# MySQL 8 命令行程序

在上一节中，我们介绍了 MySQL 8 提供的各种类型的程序，并简要概述了它们的用法。

在本节中，我们将查看命令行程序，并学习如何从命令行执行程序。我们将详细了解选项的提供方式以及它们如何用于管理。

# 从命令行执行程序

从命令行（shell 或命令提示符）执行程序是 MySQL 中最常用的管理形式之一。许多程序已经添加了管理选项。

# 执行 MySQL 程序

要执行 MySQL 程序，请输入程序名称，然后输入选项或其他参数，以告诉程序您希望它执行什么操作。以下是一些示例执行命令。这里`shell>`表示命令解释器。典型的提示符将是`c：>\`（对于使用`command.com`或`cmd.exe`作为命令解释器的 Windows 机器），`$`（对于使用`sh`，`ksh`或`bash`作为命令解释器的 Unix 机器），以及`％`（对于使用`csh`或`tcsh`作为命令解释器的 Mac 机器）。

```sql
shell> mysql --verbose --help
shell> mysql --user=root --password=******** mysampledb
shell> mysqldump -u root personnel
shell> mysqlshow --help 
```

有非选项参数，即没有任何前导破折号的参数，为程序提供补充信息。例如，如果您看到前面示例的第二行，它有一个带有数据库名称`mysampledb`的第三个非选项参数，因此命令`mysql --user=root --password=******** mysampledb`告诉`mysql`程序您要使用`mysampledb`作为数据库名称。

以单破折号或双破折号`（-，--）`开头的参数用于指定程序选项。指定程序选项表示程序将连接到服务器的连接类型或将影响操作模式。选项的语法将详细解释，参见*为程序指定选项*部分。

# 连接到 MySQL 服务器

在本节中，我们将解释如何建立与 MySQL 服务器的连接。我们将使用客户端程序连接到 MySQL 服务器。要连接到服务器程序，我们需要一些信息来指定 MySQL 帐户的“主机名”、“用户名”和“密码”，因为我们需要告诉客户端程序服务器运行在哪个主机上以及相关的用户名和密码。尽管这些选项都有默认选项值，但在必要时可以覆盖选项值。例如，考虑使用常见的客户端程序`mysql：`

```sql
shell> mysql
```

在前面的程序中，未指定任何选项，但将自动应用以下默认值：

+   主机名默认值为`localhost`

+   用户名默认值根据登录名（在 Windows 中为 ODBC 名称或在 Unix 中为登录名）

+   如果未与程序一起指定`-p`或`--password`选项，则不会向程序发送任何选项值

+   第一个非选项参数被视为`mysql`程序的默认数据库名称，如果未指定此类选项，则`mysql`不会选择任何默认数据库

应用于客户端程序`mysql`的原则也适用于其他客户端程序，如`mysqldump`、`mysqladmin`或`mysqlshow`。现在让我们看一个具有特定选项值参数的示例客户端程序连接：

```sql
shell> mysql --host=localhost --user=root --password=mypwd mysampledb
```

如您在前面的示例中看到的特定选项值，主机应被视为`localhost`，用户值提供为`myname`。密码也被指定，最后，指定了一个非选项参数，告诉程序使用`mysampledb`作为默认数据库名称。

# 为程序指定选项

在前面的部分中，我们已经看到程序选项如何根据客户端程序中指定的参数值改变操作模式。在这里，我们将看几种指定 MySQL 程序选项的方法。这些包括：

+   在程序名称后面提供命令行选项。这是提供选项的常见方式，但它仅适用于该时间执行程序的执行。

+   在程序开始执行之前读取的选项文件中提供选项。这是提供您希望程序每次执行时使用的选项的常见方式。

+   在环境变量中提供选项。通过使用这种方法，您还可以指定每次执行程序时要应用的选项。在一般实践中，使用选项文件通常用于此目的，但在某些情况下，在环境变量中指定选项值也非常有用；例如，在 Unix 系统上运行多个 MySQL 实例时。

MySQL 程序通过检查相关环境变量来确定首先给出哪些选项，然后处理选项文件，最后考虑命令行中的选项参数。因此，命令行选项具有最高优先级，环境变量具有最低优先级。但是，有一个例外适用，即数据目录中处理的`mysqld-auto.cnf`选项文件，因此它比命令行选项具有更高的优先级。

# 命令行选项

使用命令行程序选项时，请遵循以下规则：

+   选项后跟程序名称。

+   选项参数以单破折号或双破折号开头，具体取决于它们使用选项名称的简写形式还是长形式。

+   选项名称区分大小写。例如，`-V`和`-v`都是有效的，因为它们分别是`--verbose`和`--version`的简写形式，因此对程序表示不同的含义。

+   选项也可以带有后面跟着选项名称的值。例如，`-h localhost`或`--host=localhost`告诉客户端程序将 localhost 作为主机名。

+   使用短选项，值可以紧跟在选项字母后面，或者两者之间也可以用单个空格。唯一的例外是指定 MySQL 密码选项时。

+   使用长选项，值和名称可以用=号分隔。

+   在选项名称中可以互换使用(-)和(_)，例如`--skip-grant-table`或`--skip_grant_table.`。两者都是有效的，并且工作方式相同，但下划线不能用来代替前导破折号。

+   带有数字值的选项值可以使用 K、M 或 G 的后缀表示 1,024 的倍数，大小写不限。考虑以下示例，命令告诉`mysqladmin`程序对服务器进行 1,024 次 ping，并且每次 ping 后休眠`10`秒：

```sql
 shell> mysqladmin --count=1k --sleep=10 ping
```

+   对于文件名选项值，避免使用`~`元字符，因为它不会按预期解释。

+   包含空格的选项值在命令行指定值时必须用引号括起来。

# 修改程序选项

一些选项是`boolean`类型的，控制可以打开或关闭的行为。让我们考虑一个例子，`mysql`程序。它支持`--column-names`选项，用于控制在查询结果的第一行中显示列名。为了禁用列名，以下规范将适用于我们：

```sql
--disable-column-names
--skip-column-names
--column-names=0
```

如前面的例子所示，`=0 后缀` 和 `--skip` 和 `--disable 前缀` 有相同的效果。当使用`=1`后缀和`--enable`前缀时也适用。

如果使用`--loose 前缀`指定选项，并且指定的选项不存在，程序将发出警告而不是退出。

对于某些程序，`--maximum 前缀`可用于与选项名称一起指定限制。它也可以与环境变量一起使用。

# 使用文件修改选项

大多数 MySQL 程序可以从选项文件中读取启动选项，有时也称为**配置文件**。这是一种非常方便的方式，可以提供常用的选项，一旦指定，就无需每次执行程序时都指定它们。要检查程序是否读取选项文件，请使用`--help`选项。例如，考虑`mysqld`程序，如果它读取选项文件，应该使用`--verbose`和`--help`。帮助消息将指示它查找哪个选项文件以及哪个选项组：

使用`--no-defualts`选项的 MySQL 程序除了`.mylogin.cnf`之外不会读取任何选项文件。如果服务器程序启动时禁用了`persisted_globals_load`系统变量，则程序不会读取`mysqld-auto.cnf`文件。

大多数选项文件都是纯文本文件，可以由任何文本编辑器创建和编辑。这些文件中的例外包括`.mylogin.cnf`，它具有登录路径选项，由`mysql_config_editor`实用程序加密。

MySQL 按特定顺序检查 Windows 和 Unix 系统的选项文件，并遵循从读取全局选项开始的优先顺序，例如在 Windows 系统中：

+   `%PROGRAMDATA%\MySQL\MySQL Server 8.0\my.ini` 和 `%PROGRAMDATA%\MySQL\MySQL Server 8.0\my.cnf`

+   `%WINDIR%\my.ini` 和 `%WINDIR%\my.cnf`

+   `C:\my.ini` 和 `C:\my.cnf`

+   `BASEDIR\my.ini` 和 `BASEDIR\my.cnf`

+   `--defaults-extra-file` 指定的文件（如果有的话）

+   在`%APPDATA%\MySQL\.mylogin.cnf`中的登录路径选项（仅限客户端程序）

+   对于使用`SET_PERSIST`或`PERSIST_ONLY`（如果是服务器程序）持久化的系统变量在`DATADIR\mysql-auto.cnf`中

类似地，在 Unix 系统中，它遵循以下读取选项文件的优先顺序：

+   `/etc/my.cnf`

+   `/etc/mysql/my.cnf`

+   `SYSCONFDIR/my.cnf`

+   `$MYSQL_HOME/my.cnf`（仅限服务器程序）

+   使用`--defaults-extra-file`指定的文件（如果有的话）

+   `~/.my.cnf`用于特定用户的选项

+   `~/.mylogin.cnf`用于特定用户的登录路径选项（仅限客户端程序）

+   对于使用`SET_PERSIST`或`PERSIST_ONLY`（如果是服务器程序）在`DATADIR\mysql-auto.cnf`中持久化的系统变量

在前面的选项中，`~`指的是当前用户的主目录。

选项文件中的空行将被忽略，以及注释。可以使用`#`或`;`字符指定注释，`#`也可以在任何行的中间开始。

# group

`group`是要设置选项的程序或组的名称。它们不区分大小写。一旦将组行添加到选项文件中，所有后续行都适用于命名组，直到指定另一个组行或选项文件结束。

# opt_name

这类似于命令行中的`--opt_name`，打开了命名优化。

# opt_name=value

这类似于命令行中的`--opt_name`，但是在值的位置上，您可以指定带有空格的值，而在命令行中则不行。

# 包含指令

可以在选项文件中使用`!include`指令包含另一个选项文件，使用`!includedir`搜索特定目录以检查选项文件。例如，`!include /home/dev/myopt.cnf`和`!includedir /home/dev`，用于目录。MySQL 不考虑的唯一一件事是在目录搜索期间的任何顺序。

在 Windows 系统上，要在`!includedir`指令中使用的任何选项文件必须以`.ini`或`.cnf`扩展名结尾，在 Unix 系统中，它们必须以`.cnf`结尾。

# 影响选项文件处理的命令行选项

大多数 MySQL 程序支持选项文件。由于它们会影响选项文件的处理，因此必须在命令行中给出，而不是作为选项文件的一部分。

为了使它们正常工作，必须在其他选项之前给出。以下是一些例外情况：`--print-defaults`可能会在`--login-path`、`--defaults-file`或`defaults-extra-file`之后立即使用。

`--no-defaults`和`--print-defaults`也用于修改选项文件处理。

# 使用选项设置程序变量

许多 MySQL 程序具有我们可以在运行时操作中使用`SET`语句设置的内部变量，也可以使用我们指定选项值的相同语法。这将在程序启动时起作用。例如，如果我们使用选项值语法，那么我们必须这样指定：`shell> mysql --max_allowed_packet=16M`。要使用`SET`指定运行时选项，我们可以这样指定：`mysql> SET GLOBAL max_allowed_packet=16*1024*1024`。

要查看选项变量语法是否正确，可以转到`mysql`并使用以下内容：`mysql> show variables like 'max%'`。

# 设置环境变量

在命令提示符中，我们可以设置环境变量，这将影响程序的运行时执行，或者可以设置为永久影响未来的执行。它们可以在启动文件中设置，也可以使用系统提供的界面进行设置。可以在*MySQL 8 环境变量*部分详细列出可以影响 MySQL 程序的环境变量。

要指定环境变量的值，语法将基于底层命令解释器。对于 Windows 系统，可以使用以下语法设置用户变量：

```sql
SET  USER=your_user_name
```

对于 Unix 系统，这取决于 shell，因此如果使用`sh`、`ksh`、`zsh`或`bash`，则需要使用以下语法：

```sql
MYSQL_TCP_PORT=3306 
export MYSQL_TCP_PORT
```

如果使用`csh`或`tcsh`，请使用`setenv`，这将使 shell 变量可用于执行环境：

```sql
setenv MYSQL_TCP_PORT 3306
```

执行的设置环境变量的命令将立即影响正在执行的程序，但如果要使环境变量持久存在，您需要在系统提供的界面上指定它，或者可以在命令处理器在启动时使用的启动文件中设置它。在 Windows 上，可以从控制面板选项设置环境变量，在 Unix 上，可以根据您使用的命令行处理器进行设置。对于`bash`，您需要将值放在`.bashrc`或`.bash_profile`中，对于`tcsh`，请使用`.tcshrc`。

# 服务器和服务器启动程序

MySQL 提供了特定的程序，您需要首先执行这些程序才能使 MySQL 正常工作。在接下来的部分中，我们将看看可以根据您的需求使用多个选项的服务器程序和相关启动程序。

# mysqld - MySQL 服务器程序

MySQL 服务器是一个守护程序。所有其他程序都通过该服务器连接到数据库，因此它应该始终运行。守护程序通常从名为`mysqld_safe`的脚本启动。需要该程序脚本，因为它设置适当的环境变量并使用所需的参数`-option`值执行`mysqld`程序。

# 选项

以下是详细介绍各种命令行可用选项的选项：

+   -？，-I，--help：显示程序的使用信息。

+   -# debuglevel，--debug=debuglevel：设置指定的调试级别。

+   -b directory，--basedir=directory：指定用于确定所有其他相关目录的基本目录。

+   --big-tables：用于允许大型结果集。它们被保存为临时结果文件。

+   --bind-address=ip-number：指定服务器将绑定到的 IP 地址。

+   -h directory，--datadir=directory：指定存储数据库数据文件的目录。

+   -l [logfile]，--log[=logfile]：添加各种日志信息，包括连接和错误信息。如果未提供参数，则使用`hostname.log`作为日志文件，这里的`hostname`是服务器机器的名称。

+   --log-isam[=logfile]：在日志中添加对数据（ISAM）文件的更改。如果未提供参数，则使用`isam.log`作为日志文件，此选项生成的日志只能使用`theisamlog`实用程序读取和操作。

+   --log-update[=number]：记录数据库更新信息。日志文件被命名为`hostname.num`，其中`hostname`是服务器机器的名称，`num`是选项的参数，如果未指定参数，则生成唯一数字。

+   -L=language，--language=language：指定服务器的语言（英语、法语、德语等）。

+   -n，--new：启用新例程（可能是不安全的例程）。

+   -S，--skip-new：禁用/启用新例程（可能是不安全的例程）。

+   -O variable=value，--set-variable variable=value：指定并设置变量的值

+   --pid-file=file：获取运行服务器的**进程 ID**（**PID**）的文件名。文件的默认值是`hostname.pid`，其中`hostname`是服务器机器的名称。

+   -P port，--port=port：指定网络端口号。

+   --secure：启用网络安全检查。但这会降低数据库性能。

+   --skip-name-resolve：指定仅使用 IP 号码（而不是名称）进行连接。这会增加网络性能。

+   --skip-networking：禁用网络连接，仅允许本地访问。

+   --skip-thread-priority：为所有线程提供相同的优先级。

+   -Sg：禁用访问检查。这允许所有用户对所有数据库拥有完全访问权限。

+   -Sl：指定不执行线程锁定。

+   --use-locking：启用线程锁定。

+   --socket=file：指定 Unix 套接字的文件名。

+   `-T`，`--exit-info`：用于在关闭服务器时显示调试信息。

+   `-v`，`-V`和`--version`：显示服务器的版本信息。

# mysqld_safe - MySQL 服务器启动脚本

这是在基于 Unix 的系统中启动 MySQL 服务器的最推荐方式，因为它增加了一些安全功能，例如在运行时将信息记录到错误日志中，如果出现错误，则重新启动服务器。

在某些 Unix 平台上，从 RPM 或 Debian 软件包安装的 MySQL 包括`systemd`支持来管理 MySQL 的启动和关闭操作，因此在这些系统上不安装`mysqld_safe`。

`mysqld_safe`尝试执行`mysqld`并覆盖默认行为以指定您想要执行的服务器的名称。还可以使用`--ledir`指定目录的选项，以便`mysqld_safe`将在该目录中查找服务器。`mysqld_safe`中的大多数选项也在`mysqld`中可用，如果指定的选项对`mysqld_safe`是未知的，则会传递给`mysqld`。`mysqld_safe`从选项文件中读取`mysqld`、服务器和`mysqld_safe`部分中的所有选项。为了向后兼容，`mysqld_safe`还读取`safe_mysqld`部分，但是您应该将这样的部分重命名为当前的`mysqld_safe`。

如前所述，`mysqld`和`mysqld_safe`中都指定了非常常见的选项，因此以下选项列表中排除了一些选项：

```sql
--core-file-size=size
```

指定`mysqld`应该创建的核心文件的大小。

```sql
--ledir=dir_name
```

如果`mysqld`无法找到服务器，则使用此选项指定服务器所在的目录的路径名。此选项仅可在命令行上使用，而不可在选项文件中使用。在使用`systemd`的平台上，该值应在`MYSQLD_OPTS`的值中给出。

```sql
--mysqld-safe-log-timestamps
```

此选项是指定`mysqld_safe`生成的日志输出中的`timestamps`格式。

```sql
--mysqld=prog_name
```

指定包含您想要启动的`ledir`目录中的服务器程序名称。如果`mysqld_safe`找不到服务器，请使用`--ledir`选项指定服务器所在的`pathname`目录。此选项仅在命令行上接受，而不是从选项文件中接受。

```sql
--open-files-limit=count
```

`mysqld`可以打开的文件数。

```sql
--plugin-dir=dir_name
```

指定插件目录的路径和名称。

```sql
--timezone=timezone
```

这个选项是将`timezone`环境变量设置为给定的选项值，具体取决于操作系统的时区规范格式。

```sql
--user={ username | user_id }
```

以您拥有用户名的名称运行`mysqld`服务器。指定`user_name`或指定数字`user ID`作为`user_id`。

# mysql.server - MySQL 服务器启动脚本

这是在 Unix/类 Unix 系统的 MySQL 发行版上使用的服务器启动脚本。它使用`mysqld_safe`来启动 MySQL 服务器程序。该程序还用于包含脚本的`V-style`运行目录的系统。它在特定运行级别启动系统服务。

```sql
basedir=dir_name
```

指定 MySQL 安装目录的路径。

```sql
datadir=data_dir
```

指定 MySQL 数据目录的路径。

```sql
pid-file=file_name
```

指定服务器写入其进程 ID 的路径名和文件名。

```sql
service-startup-timeout=seconds
```

指定等待服务器启动确认的秒数。如果服务器在此时间内未启动，`mysql.server`将以错误指示退出。该选项的默认值为 900 秒，值为 0 表示根本不等待启动，提供负值表示永远等待（不应该有超时）。

# mysqld_multi - 管理多个 MySQL 服务器

`mysqld_multi`用于管理在不同 Unix 套接字文件和 TCP/IP 端口上监听连接的多个`mysqld`进程。它还可以启动或停止服务器并报告其当前状态。

`mysqld_multi`在`my.cnf`或提供的文件中搜索名为`mysqldN`的组，作为`--default-file`选项。这里 N 可以是任何正数。这个数字被称为选项组号，即`GNR`。组号将选项组彼此分开，并且在`mysqld_multi`的参数中用于指定要启动、停止或请求状态的服务器。此组中的选项与用于启动`mysqld`的`[mysqld]`组中使用的选项相同。

要执行`mysqld_multi`，使用以下语法：

```sql
shell> mysqld_multi [options] {start|stop|reload|report} [GNR[,GNR] ...]
```

在上述语法中，start、stop、reload（停止和重新启动）和 report 指的是要执行的操作。根据指定的`GNR`列表值，您可以对单个或多个服务器执行有针对性的操作。

请确保所有服务器的数据目录对启动特定`mysqld`进程的 Unix 帐户完全可访问。除非您确切知道要做什么，否则不要使用 root 帐户。

# 安装程序

本节讨论的程序在安装过程中或升级 MySQL 时使用，因此在对程序进行任何修改之前，请确保您正确理解它。

# comp_err - 编译 MySQL 错误消息文件

这将创建`errmsg.sys`文件，`mysqld`用于识别错误消息并显示单独的错误代码。当构建 MySQL 时，通常会自动运行`comp_err`。`errmsg.sys`文件是从 MySQL 发行版中的文本文件`sql/share/errmsg-utf8.txt`编译而成。它还生成`sql_state.h`、`mysqld_ername.h`和`mysqld_error.h`头文件。

`comp_err`有几个选项，并且可以在前一个命令中使用`--help`选项检索。

# mysql_secure_installation - 改进 MySQL 安装安全性

该程序有办法启用和改进 MySQL 安装的安全性，包括：

+   为`root`帐户设置密码。

+   删除可以在`localhost`之外访问的`root`帐户。

+   删除匿名帐户。

+   删除允许任何人访问以`test_`开头的数据库的测试数据库权限。

+   如果您希望在本地 MySQL 服务器上进行正常使用，请执行`mysql_secure_installation`命令而不带任何参数。它将要求您进一步检查要执行的操作。

+   `validate_password`插件可用于加强密码检查。如果插件尚未安装，它将要求您安装，一旦安装并启用，它可以验证密码。

使用以下语法执行`mysql_secure_installation`：

```sql
shell> mysql_secure_installation [options]
```

您可以在这里使用`--help`选项，并在需要时检索其他选项的列表。

# mysql_ssl_rsa_setup - 创建 SSL/RSA 文件

正如其名称所示，此程序用于生成 SSL 证书和密钥文件以及 RSA 密钥对文件，如果这些文件丢失并且需要文件来支持安全连接。如果现有文件已过期，`mysql_ssl_rsa_setup`也可以用于创建新文件。

`mysqlssl_rsa_setup`使用`Openssl`命令，因此在使用它时，有必要在您的机器上安装`OpenSSL`。为了自动生成这些文件，服务器可以使用使用`OpenSSL`编译的 MySQL 发行版，这是生成`SSL`和`RSA`文件的另一种方式。

执行`mysql_ssl_rsa_setup`，如下所示：

```sql
shell> mysql_ssl_rsa_setup [options]
```

在这里使用`--help`选项，并在需要时检索其他选项的列表。

使用`ssl``mysql_ssl_rsa_setup`降低了`ssl`的障碍，并使生成所需文件变得更容易，但生成的文件是自签名的，这并不是非常安全的。您可以考虑从相应的机构获取 CA 证书。

# mysql_tzinfo_to_sql - 加载时区表

该程序获取`hostsystem`区域信息数据库（描述时区的文件）的内容，并将信息加载到 MySQL 的时区表中。如果系统没有区域信息数据库，您可以从[`dev.mysql.com/downloads/timezones.html`](https://dev.mysql.com/downloads/timezones.html)下载包含`POSIX`标准`timezone_2017c_posix_sql.zip`和非`POSIX`标准`timezone_2017c_leaps_sql.zip`的可下载包。

`mysql_tzinfo_to_sql`可以以以下不同的方式执行：

```sql
shell> mysql_tzinfo_to_sql tz_dir
shell> mysql_tzinfo_to_sql tz_file tz_name
shell> mysql_tzinfo_to_sql --leap tz_file
```

运行`mysql_tzinfo_to_sql`程序后，强烈建议重新启动服务器，以便它不使用任何先前缓存的`时区`数据。

# mysql_upgrade - 检查和升级 MySQL 表

顾名思义，这用于升级操作。它检查任何不兼容性，如果有必要进行修复，并且还会更新 MySQL 新版本中的授权表中的任何更改。它还更新系统表，以便您可以利用在较新版本中添加的任何新特权或兼容性。

在执行升级之前，始终备份当前的 MySQL 安装。

如果`mysql_upgrade`发现表可能存在不兼容性，它将执行表检查并尝试修复表，如果无法修复，它将要求手动修复表。

每次升级 MySQL 时都应执行`mysql_upgrade`。它直接与 MySQL 服务器通信，并发送所需的 SQL 语句以执行升级。

运行`mysql_upgrade`后，我们应该重新启动服务器。如果对系统表进行了任何更改，需要在运行之前确保服务器正在运行。

使用以下语法执行`mysql_upgrade`：

```sql
shell> mysql_upgrade [options]
```

您可以在此处使用`--help`选项，并在需要时检索其他选项的列表。

# MySQL 8 客户端程序

MySQL 8 客户端程序是通常用于连接到 MySQL 数据库并执行不同查询操作的程序。

以下子节中详细介绍了程序信息，包括`mysql` - 带有许多命令和相关选项和配置的命令行工具，用于`logging`，`mysqlcheck`，`mysqldump`，`mysqlimport`，`mysqlsh`，`mysqladmin`等等。

# mysql - 命令行工具

这是最常用的程序。命令行工具用于直接执行 SQL 语句或以批处理模式使用文件。它支持交互和非交互模式。在本节中，我们将查看`mysql`命令行以及各种选项，命令，日志记录和其他相关程序。

# mysql 选项

`mysql`是一个长期提供的命令行工具，因此有很多选项可以完成您的工作。以下是带有格式和描述的选项表：

| **格式** | **描述** |
| --- | --- |
| `--auto-rehash` | 启用自动重新散列 |
| `--auto-vertical-output` | 启用自动垂直结果集显示 |
| `--batch` | 不使用历史文件 |
| `--binary-as-hex` | 以十六进制表示法显示二进制值 |
| `--binary-mode` | 禁用 \r\n - 到 - \n 的转换和将 \0 视为查询结束 |
| `--bind-address` | 使用指定的网络接口连接到 MySQL 服务器 |
| `--character-sets-dir` | 安装字符集的目录 |
| `--column-names` | 在结果中写入列名 |
| `--column-type-info` | 显示结果集元数据 |
| `--comments` | 确定是否保留或剥离发送到服务器的语句中的注释 |
| `--compress` | 压缩客户端和服务器之间发送的所有信息 |
| `--connect-expired-password` | 指示服务器客户端可以处理过期密码沙箱模式 |
| `--connect_timeout` | 连接超时前的秒数 |
| `--database` | 要使用的数据库 |
| `--debug` | 写入调试日志；仅在 MySQL 构建时支持调试 |
| `--debug-check` | 在程序退出时打印调试信息 |
| `--debug-info` | 在程序退出时打印调试信息、内存和 CPU 统计信息 |
| `--default-auth` | 要使用的身份验证插件 |
| `--default-character-set` | 指定默认字符集 |
| `--defaults-extra-file` | 除了通常的选项文件之外，读取命名的选项文件 |
| `--defaults-file` | 仅读取命名的选项文件 |
| `--defaults-group-suffix` | 选项组后缀值 |
| `--delimiter` | 设置语句分隔符 |
| `--enable-cleartext-plugin` | 启用明文身份验证插件 |
| `--execute` | 执行语句并退出 |
| `--force` | 即使发生 SQL 错误也继续 |
| `--get-server-public-key` | 包含 RSA 公钥的文件路径名 |
| `--help` | 显示帮助消息并退出 |
| `--histignore` | 指定要忽略记录的语句模式 |
| `--host` | 连接到给定主机上的 MySQL 服务器 |
| `--html` | 生成 HTML 输出 |
| `--ignore-spaces` | 忽略函数名后的空格 |
| `--init-command` | 连接后要执行的 SQL 语句 |
| `--line-numbers` | 为错误写入行号 |
| `--local-infile` | 启用或禁用`LOAD DATA INFILE`的`LOCAL`功能 |
| `--login-path` | 从.mylogin.cnf 中读取登录路径选项 |
| `--max_allowed_packet` | 发送到服务器或从服务器接收的最大数据包长度 |
| `--max_join_size` | 使用`--safe-updates`时连接中的行的自动限制 |
| `--named-commands` | 启用命名的`mysql`命令 |
| `--net_buffer_length` | TCP/IP 和套接字通信的缓冲区大小 |
| `--no-auto-rehash` | 禁用自动重新哈希 |
| `--no-beep` | 发生错误时不发出哔声 |
| `--no-defaults` | 不读取任何选项文件 |
| `--one-database` | 忽略除了命令行上指定的默认数据库之外的语句 |
| `--pager` | 用于分页查询输出的给定命令 |
| `--password` | 连接到服务器时使用的密码 |
| `--pipe` | 在 Windows 上，使用命名管道连接到服务器 |
| `--plugin-dir` | 插件安装的目录 |
| `--port` | 用于连接的 TCP/IP 端口号 |
| `--print-defaults` | 打印默认选项 |
| `--prompt` | 将提示设置为指定的格式 |
| `--protocol` | 要使用的连接协议 |
| `--quick` | 不缓存每个查询结果 |
| `--raw` | 写入列值而不进行转义转换 |
| `--reconnect` | 如果与服务器的连接丢失，自动尝试重新连接 |
| `--i-am-a-dummy, --safe-updates` | 仅允许指定键值的`UPDATE`和`DELETE`语句 |
| `--secure-auth` | 不以旧（4.1 之前）格式将密码发送到服务器 |
| `--select_limit` | 使用`--safe-updates`时`SELECT`语句的自动限制 |
| `--server-public-key-path` | 包含 RSA 公钥的文件路径名 |
| `--shared-memory-base-name` | 用于共享内存连接的共享内存名称 |
| `--show-warnings` | 如果有的话，在每个语句后显示警告 |
| `--sigint-ignore` | 忽略`SIGINT`信号（通常是键入*Control*+*C*的结果） |
| `--silent` | 静默模式 |
| `--skip-auto-rehash` | 禁用自动重新哈希 |
| `--skip-column-names` | 在结果中不写入列名 |
| `--skip-line-numbers` | 跳过错误的行号 |
| `--skip-named-commands` | 禁用命名的 mysql 命令 |
| `--skip-pager` | 禁用分页 |
| `--skip-reconnect` | 禁用重新连接 |
| `--socket` | 对于本地主机的连接，要使用的 Unix 套接字文件或 Windows 命名管道 |
| `--ssl-ca` | 包含受信任的 SSL CA 列表的文件路径 |
| `--ssl-capath` | 包含 PEM 格式的受信任 SSL CA 证书的目录路径 |
| `--ssl-cert` | 包含 PEM 格式 X509 证书的文件路径 |
| `--ssl-cipher` | 用于连接加密的允许密码列表 |
| `--ssl-crl` | 包含证书吊销列表的文件路径 |
| `--ssl-crlpath` | 包含证书吊销列表文件的目录路径 |
| `--ssl-key` | 包含 PEM 格式 X509 密钥的文件路径 |
| `--ssl-mode` | 与服务器连接的安全状态 |
| `--syslog` | 将交互式语句记录到 syslog |
| `--table` | 以表格格式显示输出 |
| `--tee` | 将输出的副本附加到指定文件 |
| `--tls-version` | 加密连接允许的协议 |
| `--unbuffered` | 在每次查询后刷新缓冲区 |
| `--user` | 连接到服务器时要使用的 MySQL 用户名 |
| `--verbose` | 详细模式 |
| `--version` | 显示版本信息并退出 |
| `--vertical` | 垂直打印查询输出行（每个列值一行） |
| `--wait` | 如果无法建立连接，则等待并重试，而不是中止 |
| `--xml` | 生成 XML 输出 |

参考：[`dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-command-options.html)

要获取有关各个选项的更多信息，请将该选项与`--help`选项一起使用。

# mysql 命令

您发出的每个 SQL 语句都将发送到服务器以进行执行。`mysql`本身也有一系列命令。要获取所有这些命令的列表，请在`mysql>`提示处键入`\h`或`\help`。

每个命令都有长格式和短格式，可以使用； 除了短格式不能在多行注释中使用。长格式不区分大小写，但短格式命令区分大小写。

# help [arg], \h [arg],\? [arg], ? [arg]

help `arg[]`命令用于显示帮助消息，并列出`mysql`中所有可用的命令。

# charset charset_name, \C charset_name

要更改默认字符集，请发出`SET_NAMES`语句。

# clear, \c

要清除命令行中的当前输出或先前的查询结果。

# connect [db_name host_name], \r [db_name host_name]

通过提供`数据库`和`host_name`参数来重新连接服务器。

# edit, \e

要编辑当前提供的输入语句。

# exit, \q

退出`mysql`命令行。

# prompt [str], \R [str]

要指定一个字符串并使用`mysql`提示重新配置它。

# quit, \q

退出`mysql`命令行。

# status, \s

这用于检查当前正在使用的服务器连接的状态。

# use db_name, \u db_name

要指定使用提供的`db_name`作为默认数据库。

# mysql logging

`mysql`程序可以按以下类型进行记录。

在 Unix 系统上，它将日志写入默认名称为`.mysql_history`的历史文件中。

在所有平台上，如果提供了`--syslog`选项，它将将语句写入系统日志实现。在 Unix 上，它是`syslog`，在 Windows 上，它是事件日志，在 Linux 发行版上，它通常会写入`/var/log/message`文件。

# mysql 服务器端帮助

要从`mysql`获取服务器端帮助，使用以下语法。

```sql
mysql> help search_string
```

如果在 help 命令之后提供任何参数，`mysql`将使用该参数来搜索字符串，以从 MySQL 参考手册内容中访问服务器端帮助。如果没有匹配搜索的字符串，搜索操作将失败，并显示如下：

```sql
mysql> help me
Nothing found
Please try to run 'help contents' fro a list of all accessible topics
```

如果`search_string`匹配主题的多个内容，则显示匹配主题项的列表。还可以将主题用作`search_string`，并查找主题的条目。它还包含通配符字符`％`和`_`，这些字符对`LIKE`运算符执行的匹配操作具有相同的含义。

# 从文本文件执行 sql

`mysql`客户端通常用于交互式使用，但也允许您从文件中执行 SQL 语句。为此，请创建一个包含需要执行的多个语句的`text_file`，如下所示：

```sql
shell> mysql db_name < text_file
```

如果将`USE db_name`保留为`text_file`的第一条语句，则可以在命令行中省略指定`db_name`：

```sql
shell> mysql < text_file
```

如果已经使用`mysql`连接，则使用 source 或`\.`命令：

```sql
mysql> source file_name
```

通过使用`--verbose`选项，每个语句在产生结果之前都会显示出来。

# mysqladmin - 用于管理 MySQL 服务器的客户端

`mysqladmin`是用于管理操作的客户端。它可以用于检查服务器的配置、连接状态、删除和创建数据库等。

`mysqladmin`命令的执行语法如下：

```sql
shell> mysqladmin [options] command [command-arg] [command [command-org]] ...
```

`mysqladmin`支持大量的程序命令，从`create db_name`创建名为`db_name`的新数据库，`debug`获取调试信息，`drop db_name`删除数据库，`flush-xxxx`，其中`xxxx`可以替换为 logs、hosts、privileges、status、tables、threads 等。`kill id`杀死服务器线程或多个线程，`password new_password`设置新密码，`ping`检查服务器的可用性，`shutdown`停止服务器，`start-slave`在从服务器上启动复制，`stop-slave`停止从服务器上的复制，`variables`显示服务器系统变量及其相应的值。

`mysqladmin status`命令显示了有关服务器的配置、连接状态、删除和创建数据库等信息。

除了命令列表之外，还有一些在从服务器检索特定信息时非常有用的选项。可以使用`mysqladmin --help`命令检索此类信息。

# mysqlcheck - 表维护程序

该程序用于表维护。它检查、修复、优化或分析表。该程序可能会耗费时间，特别是对于大型表。`mysql_upgrade`使用`mysqlcheck`命令来检查和修复所有表。必须运行`mysqld`服务器才能使用`mysqlcheck`命令。

`mysqlcheck`以用户方便的方式使用`CHECK TABLE`、`REPAIR TABLE`、`ANALYZE TABLE`和`OPTIMIZE TABLE`。如果`mysqlcheck`修复表失败，则需要手动修复表。

以下是`mysqlcheck`命令的执行语法：

```sql
shell> mysqlcheck [options] db_name [tbl_name ...]
shell> mysqlcheck [options] --databases db_name ...
shell> mysqlcheck [options] --all-databases
```

`mysqlcheck`具有一个特殊功能，即检查表的默认行为。可以通过重命名二进制文件来更改它，例如通过创建`mysqlcheck`的副本并为`mysqlcheck`添加符号链接，将`mysqlcheck`重命名为`mysqlrepair`程序，之后`mysqlrepair`可以修复表。这也适用于`mysqlanalyze`和`mysqloptimize`选项，使它们成为`mysqlcheck`命令的默认操作。

与其他管理程序类似，该程序也有许多选项可用于获取特定信息，通过使用`mysqlcheck --help`命令，可以检索到选项列表。

# mysqldump - 数据库备份程序

该程序是一个实用程序，用于通过生成一组 SQL 语句进行逻辑备份，以便重新生成原始数据库表数据和对象定义。它可以备份一个或多个数据库，也可以转移到另一个 SQL 服务器。它还可以以不同格式生成数据输出，如 CSV、XML 或其他分隔文本文件。

# 性能和可扩展性

不应将其视为备份大量数据的快速或可扩展的解决方案。备份需要一些时间，恢复数据可能非常慢，因为 SQL 语句涉及索引创建、插入的磁盘 I/O 等。它可以逐行检索和转储表数据行；否则，它可以获取整个表并将其缓冲在内存中以进行转储。

以下是`mysqldump`的执行语法：

```sql
shell> mysqldump [options] db_name [tbl_name ...] 
shell> mysqldump [options] --databases db_name ...
shell> mysqldump [options] --all-databases
```

`mysqldump`命令有超过 25 个选项可用于修改其操作，可以通过使用`mysqldump --help`命令来检索。根据您的需求，可以在此命令中使用特定的修改选项，例如调试选项、帮助选项、连接选项、DDL 选项等。

# mysqlimport - 数据导入程序

此客户端程序提供了`LOAD_DATA_INFILE` SQL 语句的`CLI`接口。大多数选项对应于`LOAD_DATA_INFILE`语法的子句。

`mysqlimport`的执行语法如下：

```sql
shell> mysqlimport [options] db_name textfile1 [textfile2 ...]
```

命令的选项可以根据您的需求在 CLI 上或在选项文件的`[mysqlimport]`或`[client]`组中指定。它提供了检索和修改数据导入操作的选项，例如使用不同的分隔符格式、调试、强制文件路径、提供默认值、忽略和锁定表等，可以通过使用`mysqlimport --help`命令来检索。

# mysqlpump - 数据库备份程序

该程序是一个实用程序，用于通过生成一组 SQL 语句来进行逻辑备份，以便执行以重现原始数据库表数据和对象定义。它可以备份一个或多个数据库，也可以转移到另一个 SQL 服务器。

`mysqlpump`具有以下重要功能：

+   数据库和数据库对象的并行处理，加快了转储处理速度

+   更多控制要转储的数据库对象和数据库

+   将用户帐户数据转储到帐户管理语句，而不是插入数据到系统数据库

+   创建压缩输出的能力

+   显示进度指示器

+   转储文件重新加载；对于`InnoDB`表，在插入行后添加索引

`mysqlpump`转储了所有数据库，如下代码中明确指定的：

```sql
shell> mysqlpump --all-databases
```

要转储多个数据库，请指定`--databases`，然后是要转储的数据库名称。您还可以指定`--exclude-databases=`，然后是不要转储的数据库名称。有许多选项可用于转储数据库或对象，例如用于指定排除或包含数据库选项的选项，适用于对象（如表、触发器、例程、事件、用户等），如果它们支持多个选项条目：

```sql
shell> mysqlpump --include-databases=db1,db2 --exclude-tables=db1.t1,db2.t2
```

上述命令转储了数据库`db1`和`db2`，但它将从数据库`db1`中排除表`t1`，从数据库`db2`中排除表`t2`。

`mysqlpump`使用并行处理来实现并发处理，可以在数据库之间或数据库内进行。`--default-parallelism=N`指定程序创建的队列中使用的默认线程数，`N`的默认值为`2`。`--parallel-schemas=[N:]db_list`根据提供的数据库名称列表设置处理队列。因此，可以控制额外的队列和线程数。

`mysqlpump`默认不转储`performance_schema`、`nbdinfo`或`sys`，但可以通过指定`--include-databases`选项来执行，同样它也不转储`INFORMATION_SCHEMA`。

# mysqlsh - MySQL Shell

MySQL 的高级命令行客户端和编辑器是非常著名的 MySQL Shell。它具有使用 Python 和 JavaScript 进行脚本编写的功能。当使用 X 协议连接到 MySQL 服务器时，X DevAPI 可以处理文档和关系数据。它包括 AdminAPI，可以与`InnoDB`集群一起使用。

MySQL Shell 有许多与之相关的选项，但重要的选项如下：

```sql
--port=port_num, -P port_num
```

要使用的 TCP/IP 端口号为`port_num`。默认端口为`33060`。

```sql
--node
```

使用 X 协议创建到单个服务器的节点会话，在 8.0.3 中已弃用。

```sql
--js
```

启动 JavaScript 模式。

```sql
--py
```

启动 Python 模式。

```sql
--sql
```

启动 SQL 模式。

```sql
--sqlc
```

在 ClassicSession 中启动 SQL 模式。

```sql
--sqln
```

在 NodeSession 中启动 SQL 模式。

```sql
--sqlx
```

通过创建 X 协议连接启动 SQL 模式。

```sql
--ssl* 
```

以 `--ssl` 开头的选项指定使用 SSL 连接服务器，并查找证书和 SSL 密钥。它的工作方式与 MySQL 服务器相同，并接受 SSL 选项：`--ssl-crl`、`--ssl-crlpath`、`--ssl-mode`、`--ssl-ca`、`--ssl-capath`、`--ssl-cert`、`--ssl-cipher`、`--ssl-key`、`--tls-version`。

使用 `mysqlsh --help` 命令可以获取未列出的其他选项。

# mysqlshow - 显示数据库、表和列信息

这是主要用于快速检查哪些数据库、它们的表、列和索引是否存在的客户端。它提供了一些 SQL `SHOW` 语句的接口。

该命令的执行语法如下：

```sql
shell> mysqlshow [options] [db_name [tbl_name [col_name]]]
```

通过执行上述命令，您将获得有关您具有权限的数据库、表或列的信息：

+   如果没有给出数据库，则显示数据库列表

+   如果没有给出表，则显示数据库中所有匹配的表的列表

+   如果没有给出列，则显示表中所有匹配的列和列类型的列表

在前面的命令执行中，如果使用了 SQL 通配符字符（`*`、`?`、`%`、`_`），则会显示与通配符匹配的名称。如果给出了 `*` 和 `?` 通配符字符，则会将它们转换为 SQL `%` 和 `_` 通配符字符。当尝试显示具有表或列名中的 `_` 的列时，它可能会造成混淆，它只显示与模式匹配的名称，但可以通过在命令行的末尾作为单独的参数添加额外的 `%` 来轻松解决。

该程序有许多选项，可以通过使用特定选项参数来获取所需的信息。以下是一些重要的选项：

```sql
--character-sets-dir=dir_name
```

指定安装字符集的目录名称。

```sql
--compress, -C
```

如果客户端和服务器都支持压缩，则发送的所有信息都将被压缩。

```sql
--enable-cleartext-plugin
```

启用 `cleartext` 认证插件。

```sql
--get--server-public-key
```

从服务器请求 RSA 公钥，用于基于密钥对的密码交换。此外，如果客户端使用安全连接连接到服务器，则不需要 RSA 密码交换，并且将被忽略。

```sql
--keys, -k
```

显示表索引。

```sql
--ssl*
```

指定以 `--ssl` 开头的选项，使用证书和 SSL 密钥连接到服务器的 SSL 连接。

使用 `mysqlshow --help` 命令可以获取这里未列出的许多其他选项。

# mysqlslap - 负载仿真客户端

这是用于检查 MySQL 服务器的客户端负载能力的诊断客户端程序。该程序模拟多个客户端访问服务器。

执行语法如下：

```sql
shell> mysqlslap [options]
```

有许多选项可用于修改执行中的命令，其中一些选项，如 `--create` 或 `--query`，提供了指定具有特定分隔符的 SQL 语句或文件的方法。

`mysqlslap` 在三个不同的阶段运行：

1.  创建表、模式和可选的存储程序或数据进行测试。它仅使用单个客户端连接。

1.  运行负载测试。它使用多个客户端连接。

1.  清理（如果之前指定了，删除表，并断开连接）。它使用单个客户端连接。

例如，使用 20 个客户端和每个客户端 100 次查询创建我们自己的查询语句将如下所示 `CLI:`

```sql
mysqlslap --delimiter=";" --create="CREATE TABLE t (i int);INSERT INTO t VALUES (21)" --query="SELECT * FROM t" --concurrency=20 --iterations=100
```

`mysqlslap` 也可以添加或创建自己的查询语句，如下面的代码块所示：

```sql
mysqlslap --concurrency=7 --iterations=20 --number-int-cols=2 --number-char-cols=2 --auto-generate-sql
```

在这里，`mysqlslap` 将使用两个 `INT` 列和两个 `VARCHAR` 列的表构建一个语句，并七个客户端对每个表查询 20 次。它还支持指定用于分别创建和查询的语句文件，并运行负载测试。它提供了许多这样的负载测试执行的变化选项，您可以通过在命令行上执行 `mysqlslap --help` 命令来检查。

# MySQL 8 管理程序

本节描述了不同的管理程序以及一些实用程序，这些程序将有助于进行管理操作，如执行校验和、压缩和提取等。

# ibdsdi - InnoDB 表空间 SDI 提取实用程序

这是一个实用程序，从`InnoDB`表空间文件中提取序列化的字典信息。SDI 数据中的序列化字典信息将始终存在于所有持久的`InnoDB`表空间文件中。它可以在文件表空间文件和通用表空间文件、系统表空间文件和数据字典表空间文件上运行，但不支持使用临时表空间或撤消表空间。

`ibd2sdi`可以在服务器离线或运行时使用。它从指定的表空间中读取 SDI 的未提交数据，并撤消日志和重做日志，这些日志是不可访问的。

`idb2sdi`的执行将如下命令行所示：

```sql
shell> ibd2sdi [options] file_name1 [file_name2 file_name3 ...] 
```

`ibd2sdi`还支持多个表空间，但不像`InnoDB`系统表空间那样一次运行多个表空间。指定每个文件将按如下方式工作：

```sql
shell> ibd2sdi ibdata1 ibdata2
```

`ibd2sdi`以 JSON 格式输出 SDI 数据。

该程序有许多选项可供使用，可以使用`ibd2sdi --help`命令检索。

# innochecksum - 离线 InnoDB 文件校验和实用程序

这是一个用于`InnoDB`文件的校验和实用程序。它读取`InnoDB`表空间文件，为它们计算校验和，并将其与存储的校验和值进行比较。如果比较失败，它将报告带有损坏页面详细信息的错误。它是为了验证停电后的完整性而开发的，但也可以在复制文件后使用。在运行`InnoDB`时，如果发现任何损坏的页面，它非常有用。它将关闭正在运行的服务器，因此为了避免由于损坏的页面而导致任何生产问题，需要注意。

如果表空间文件是打开的，则无法使用`innochecksum`。命令的执行语法将如下所示：

```sql
shell> innochecksum [options] file_name
```

`innochecksum`命令还有一些选项，用于显示正在验证的页面的信息，并可以使用`innochecksum --help`命令检索。

# myisam_ftdump - 显示全文索引实用程序

这是一个用于显示`MyISAM`表和`FULLTEXT`索引信息的实用程序。它将扫描和转储整个索引，这可能是一个缓慢和冗长的过程。如果服务器已经在运行，则需要确保首先插入`FLUSH TABLES`语句。

`myisam_ftdump`命令的执行将如下代码块所示：

```sql
shell> myisam_ftdump [options] <table_name> <index_num>
```

在前面的示例中，`table_name`应该是带有`.MYI`索引扩展名的`MyISAM`表的名称。

假设测试数据库有一个名为 mytexttable 的表，其定义如下：

```sql
CREATE TABLE mytexttable ( id INT NOT NULL, txt TEXT NOT NULL, PRIMARY KEY (id), FULLTEXT (txt) ) ENGINE=MyISAM;
```

在`FULLTEXT`索引上创建的 id 为 0，在`txt`上为 1。如果工作目录是测试数据库目录，则执行`myisam_ftdump`如下：

```sql
shell> myisam_ftdump mytexttable 1
```

`myisam_ftdump`也可以用来按出现频率生成索引条目列表，如下所示（Windows 中的第一行，Unix 系统中的第二行）：

```sql
shell> myisam_ftdump -c mytexttable 1 | sort /R 
shell> myisam_ftdump -c mytexttable 1 | sort -r

```

`myisam_ftdump`还有一些选项，可以使用`myisam_ftdump --help`命令检索。

# myisamchk - MyISAM 表维护实用程序

`myisamchk`是一个命令行工具，用于获取关于数据库表的信息，检查、修复和优化非分区的`MyISAM`表。它适用于`MyISAM`表。

`CHECK_TABLE`和`REPAIR_TABLE`语句也可用于检查和修复`MyISAM`表。

`myisamchk`命令的执行如下代码块所示：

```sql
shell> myisamchk [OPTIONS] tables[.MYI | .MYD]
```

在运行`myisamchk`命令之前，必须确保没有其他程序在使用表。否则，它将显示警告消息，提示：警告：客户端正在使用或未正确关闭表。为了有效地做到这一点，关闭 MySQL 服务器或锁定`myisamchk`正在使用的所有表。

该程序有许多选项可用于执行表维护。

# myisamlog - 显示 MyISAM 日志文件内容

该程序是用于处理 MyISAM 日志文件内容的实用程序。启动 MySQL 服务器时，使用`--log-isam=log_file`选项。

执行`myisamlog`命令的语法如下代码块所示：

```sql
shell> myisamlog [options] [file_name [tbl_name] ...]

```

默认操作标记为更新，如果进行恢复，则所有写入、更新和删除都会被执行，只计算错误。`myisam.log`是默认的日志文件名。

该程序具有一些用于指定偏移量、恢复、打开多个文件等的选项，可以使用`myisamlog --help`命令检索。

# myisampack - 生成压缩的只读 MyISAM 表

这是一个压缩`MyISAM`表的实用程序。它分别压缩表中的每一列，并将数据文件压缩约 40%至 70%。

MySQL 最好使用`mmap()`函数对压缩表进行内存映射；否则，它将使用正常的读/写文件操作。

`myisampack`不支持分区表。

一旦表被打包，它们就变成了只读。

停止服务器，然后进行压缩表是执行压缩操作的安全方式。

`myisampack`的执行语法如下命令行中的块：

```sql
shell> myisampack [options] file_name ...
```

指定索引文件名时，可以带有或不带有`.MYI`文件，并且如果不在数据库目录中，还可以添加`pathname`。

使用`myisampack`压缩表后，应使用`myisampack -rq`重新构建压缩表的索引。它支持一些常见选项，如版本控制、调试等，以及特定的压缩检查，如`--test`、`--backup`、`--join=big_tbl_name`、`--silent`等。如果您想进行详细检查，可以在命令行上执行`myisampack --verbose --help`命令。

# mysql_config_editor - MySQL 配置实用程序

这是一个实用程序，用于在加密的登录路径文件中存储和更新认证凭据，文件名为`.mylogin.cnf`。

有关详细信息，请使用`mysql_config_editor --verbose --help`命令在命令行上执行。

# mysqlbinlog - 用于处理二进制日志文件的实用程序

该程序是用于处理服务器的二进制日志文件的实用程序。二进制日志文件包含描述对数据库内容的修改的事件数据。服务器以二进制格式将此类内容写入文件。为了将它们转换为可读（文本）格式，使用`mysqlbinlog`实用程序。

`mysqlbinlog`还可以用于显示由从服务器在复制设置期间写入的中继日志文件的内容，因为中继日志文件和二进制日志文件的格式相同。

程序语法的执行如下代码块所示：

```sql
shell> mysqlbinlog [options] log_file ...
```

有几个选项可以修改`mysqlbinlog`的输出格式和用法。它们列如下：

+   它可以转换为包含字节位置、事件时间戳和发生的事件类型的十六进制转储格式

+   它还提供了一种行事件显示格式，以伪 SQL 语句的形式显示数据修改信息

+   它还用于通过提供所需的选项值（如备份文件的路径和备份中的类型或格式）来备份二进制日志文件的内容

+   当使用`mysqlbinlog`连接到 MySQL 服务器时，它会提供一个特定的服务器 ID 来标识自己，并从服务器请求二进制日志文件

该程序具有一些常见选项，未列出，但可以使用`mysqlbinlog --help`命令检索。

# mysqldumpslow - 汇总慢查询日志文件。

该程序是一个实用程序，用于读取慢查询日志文件的日志内容，其中包含执行时间较长的查询。它解析 MySQL 慢查询日志文件并打印出查询内容的摘要。

它通常将类似的查询分组在一起，除了特定的数字或字符串数据值。它会将这些值抽象化，并分别使用 `N` 和 `S` 显示摘要输出。`-n` 和 `-a` 选项用于修改值的抽象行为。

在命令行语法中执行程序的方式如下所示：

```sql
shell> mysqldumpslow [options] [log_file ...]
```

您可以通过使用一些选项来修改输出，例如限制结果的数量 `-t N`，其中 `N` 是要显示的查询结果的数量。`-s` 代表按查询时间、锁定时间或行数排序类型，`-r` 代表反转排序顺序。

# MySQL 8 环境变量

在本节中，我们将查看直接或间接用于不同 MySQL 程序的环境变量的数量，并使用环境变量改变它们的行为。

命令行提供的选项优先于选项文件和环境变量中指定的值，同样，选项中的值优先于环境变量；因此，在大多数情况下，最好使用选项文件而不是环境变量来修改行为。

以下是环境变量和变量描述的列表：

+   `CXX`: 用于运行 CMake 的 C++ 编译器的名称

+   `CC`: 用于运行 CMake 的 C 编译器的名称

+   `DBI_USER`: Perl DBI 的默认用户名

+   `DBI_TRACE`: Perl DBI 的跟踪选项

+   `HOME`: `mysql` 历史文件的默认路径是 `$HOME/.mysql_history`

+   `LD_RUN_PATH`: 用于指定 `libmysqlclient.so` 的位置。

+   `LIBMYSQL_ENABLE_CLEARTEXT_PLUGIN`: 启用 `mysql_clear_password` 认证插件

+   `LIBMYSQL_PLUGIN_DIR`: 查找客户端插件的目录

+   `LIBMYSQL_PLUGINS`: 预加载的客户端插件

+   `MYSQL_DEBUG`: 调试时的跟踪选项

+   `MYSQL_GROUP_SUFFIX`: 可选的组后缀值（例如指定 `--defaults-group-suffix`）

+   `MYSQL_HISTFILE`: `mysql` 历史文件的路径；如果设置了此变量，其值将覆盖 `$HOME/.mysql_history` 的默认值

+   `MYSQL_HISTIGNORE`：指定不应将其记录到 `$HOME/.mysql_history` 或 `orsyslog if --syslog` 的语句模式

+   `MYSQL_HOME`: 服务器特定的 `my.cnf` 文件所在目录的路径

+   `MYSQL_HOST`: 服务器特定的 `my.cnf` 文件所在目录的路径

+   `MYSQL_PWD`: 连接到 `mysqld` 时的默认密码；使用此选项是不安全的

+   `MYSQL_TCP_PORT`：默认的 TCP/IP 端口号

+   `MYSQL_TEST_LOGIN_FILE`：`.mylogin.cnf` 登录路径文件的名称

+   `PATH`: 用于 shell 查找 MySQL 程序

+   `PKG_CONFIG_PATH`: `mysqlclient.pc` `pkg-config` 文件的位置

+   `TMPDIR`: 创建临时文件的目录

+   `TZ`: 这应该设置为您的本地时区

+   `UMASK`: 创建文件时的用户文件创建模式

+   `UMASK_DIR`: 创建目录时的用户目录创建模式

+   `USER`: 连接到 `mysqld` 时 Windows 上的默认用户名

`MYSQL_TEST_LOGIN_FILE` 是由 `mysql_config_editor` 创建的登录路径文件的路径名。

`UMASK` 和 `UMASK_DIR` 变量用作模式而不是掩码。

如果使用 pkg-config 构建 MySQL 程序，则必须设置 `PKG_CONFIG_PATH`。

# MySQL GUI 工具

有许多 MySQL GUI 工具可用于执行各种操作，从创建数据库到执行日常管理任务。

# MySQL Workbench

MySQL Workbench 是一个与 MySQL 服务器和数据库一起工作的图形工具。它完全支持 MySQL 版本 5.1 及以上。在本节中，我们将简要讨论 MySQL Workbench 的功能。

MySQL Workbench 提供的五个主要功能如下：

+   **SQL 开发**：创建和管理数据库连接，并配置连接参数。它使用内置的 SQL 编辑器执行 SQL 语句，并替换了之前提供的独立应用程序查询浏览器。

+   **数据建模**：这将以图形方式创建数据库模式的模型，并在两个不同的模式之间进行反向和正向工程，以及在实时数据库上进行。它提供了一个全面的表编辑器，易于使用，用于编辑表、列、触发器、索引、选项、插入、分区、例程、视图和权限。

+   **服务器管理**：这将创建、维护和管理服务器实例。

+   **数据迁移**：这允许从 PostgreSQL、SQLite、Sybase ASE、Microsoft SQL Server 和其他关系数据库管理系统对象、表和数据迁移到 MySQL。它还便于从早期版本迁移到最新版本。

+   **MySQL 企业支持**：这为产品提供企业级支持；例如，MySQL 企业备份，MySQL 审计。

MySQL Workbench 有两个不同的版本，商业版和社区版。社区版是免费提供的。商业版提供额外的功能，如数据库文档生成。

# MySQL Notifier

MySQL Notifier 是一个简单的工具，用于监视和调整本地/远程 MySQL 服务器实例的状态。它是一个放置在系统托盘中的指示器。它是与 MySQL 安装程序一起安装的。

MySQL Notifier 充当快速启动器，其中列出的操作可以很容易地从系统托盘中进行操作和监视，并且它会根据指定的间隔进行监视，并在状态更改时通知。

# MySQL Notifier 的使用

MySQL Notifier 停留在系统托盘中，并提供了一个点击选项，用于 MySQL 的状态和维护。以下是 MySQL Notifier 的重要用途。

+   MySQL Notifier 提供 MySQL 服务器实例的启动、停止和重启。

+   MySQL Notifier 配置 MySQL 服务器服务，并自动检测并添加新的 MySQL 服务器服务。

+   MySQL Notifier 监视本地和远程 MySQL 实例。

# 摘要

在本章中，我们深入研究了用于 MySQL 服务器几乎所有活动的命令，从安装、服务器启动、客户端程序到管理程序，以及用于数据库管理的各种不同目的的多个实用程序。本章还提供了制作数据库备份和导入数据库的工作知识，包括具体表或不包括具体表。

下一章将详细介绍 MySQL 8 数据类型。它将根据其内容类型对数据类型进行分类，并详细描述每个类别的属性和在表和列设计期间应该牢记的存储级别细节。
