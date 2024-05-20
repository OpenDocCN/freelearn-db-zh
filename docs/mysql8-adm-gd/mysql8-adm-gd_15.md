# 第十五章：MySQL 8 故障排除

在上一章中，我们学习了 MySQL 8 数据库、基准测试和最佳实践的一个重要方面。基准测试有助于比较当前数据库性能与预期性能矩阵。我们了解了什么是基准测试，以及可以用来查找 MySQL 8 服务器基准性能的工具。在本章的后面部分，我们学习了关于 memcached、复制、分区和索引的最佳实践。最佳实践有助于确保 MySQL 8 数据库的最佳配置。

在本章中，重点将放在理解在使用 MySQL 8 数据库时可能遇到的常见错误上。错误可能是服务器错误或客户端错误。我们将探讨一种确定问题发生的方法。我们还将学习错误的故障排除和解决技术。在本章的后面部分，我们将探讨这些技术适用的真实场景。以下是要涵盖的主题列表：

+   MySQL 8 常见问题

+   MySQL 8 服务器错误

+   MySQL 8 客户端错误

+   MySQL 8 故障排除方法

+   真实场景

# MySQL 8 常见问题

在故障排除时，首先要做的是找出导致问题的程序或设备。

以下是表明硬件或内核问题的症状：

+   键盘无法正常工作。可以通过按下*大写锁定*键来检查。如果*大写锁定*键上的灯不亮，那么键盘有问题。同样，鼠标不动表示鼠标有问题。

+   `ping`是一个操作系统命令，用于检查一台计算机从另一台计算机的可访问性。执行 ping 命令的计算机称为**本地计算机**，而被 ping 的计算机称为**远程计算机**。如果远程计算机不响应本地计算机的 ping，表示存在硬件或网络相关的问题。

+   如果除了 MySQL 之外的程序无法正常工作，可能表明操作系统内核程序有问题。

+   如果系统意外重启，可能表明操作系统或硬件有问题。在典型情况下，用户级程序不应该能够使系统崩溃。

要排除问题，可以执行以下一项或多项操作：

+   运行诊断工具检查硬件

+   确保相关的库文件是最新的

+   检查操作系统的更新、补丁或服务包的可用性

+   检查所有连接

`ECC 内存`是纠错码内存。它可以检测和纠正大多数常见的内部数据损坏问题。建议使用 ECC 内存以便在早期检测内存问题。

以下说明可能有助于进一步确定问题：

+   检查系统日志文件可能有助于发现问题的原因。如果 MySQL 出现问题，还必须检查 MySQL 日志文件。

+   可以使用特定于操作系统的命令来检查内存、文件描述符、磁盘空间或其他关键资源的问题。

+   如果一个有问题的运行进程即使我们执行了杀死它的命令仍不会死掉，那么操作系统内核中可能存在一个 bug。

+   如果硬件似乎没有问题，应该尝试确定可能导致问题的程序。使用特定于操作系统的命令，如 Windows 上的任务管理器，Linux 上的`ps`和`top`，或类似的程序，我们可以识别占用 CPU 或阻塞系统进程的程序。

+   即使键盘被锁定，也可以恢复对计算机的访问。可以通过从另一台计算机登录系统来实现。成功登录后执行`kbd_mode -a`命令。

MySQL 用户可以通过使用 MySQL 提供的多个渠道之一来报告问题。在检查了所有可能的替代方案之后，如果可以确定是 MySQL 服务器或 MySQL 客户端引起了问题，用户可以为邮件列表创建错误报告或联系 MySQL 支持团队。报告人必须提供有关错误、系统信息和行为以及预期行为的详细信息。报告人必须根据为什么似乎是 MySQL 错误的原因描述原因。如果程序失败，了解以下信息很有用：

+   使用`top`命令，检查所讨论的程序是否占用了所有的 CPU 时间。在这种情况下，我们应该允许程序运行一段时间，因为可能程序正在执行密集的计算指令。

+   观察 MySQL 服务器对客户端程序尝试连接时的响应。它停止响应了吗？服务器提供了任何输出吗？

+   如果发现 MySQL 服务器在`mysqld`程序中引起问题，请尝试使用`mysqladmin`程序连接以检查`mysqld`是否有响应。可以使用`mysqladmin -u root ping`或`mysqladmin -u root processlist`命令。

+   失败的程序是否发生了分段错误？

# 最常见的 MySQL 错误

本节提供了用户经常遇到的最常见的 MySQL 错误列表。

# 访问被拒绝

MySQL 提供了一个特权系统，用于验证从主机连接的用户，并将用户与数据库上的访问权限关联起来。权限包括`SELECT`、`INSERT`、`UPDATE`和`DELETE`，并能够识别匿名用户并授予 MySQL 特定功能的权限，例如`LOAD DATA INFILE`和管理操作。

访问被拒绝的错误可能是由许多原因引起的。在许多情况下，问题是由于客户端程序使用的 MySQL 帐户与 MySQL 服务器连接时获得了服务器的许可。

# 无法连接到[local] MySQL 服务器

在本节中，我们将重点关注遇到无法连接到 MySQL 服务器错误的情况。但在我们跳到特定错误的细节之前，有必要了解 MySQL 客户端如何连接到 MySQL 服务器。

在 Unix 系统上，MySQL 客户端连接到`mysqld`服务器进程有两种不同的方式。以下是这两种方法的详细信息：

+   **TCP/IP 连接**：`mysqld`服务器进程在特定端口上监听客户端连接。MySQL 客户端使用指定的 TCP/IP 端口连接服务器。

+   **Unix 套接字文件**：在这种连接模式下，Unix 套接字文件用于通过文件系统（`/tmp/mysql.sock`）进行连接。

与 TCP/IP 相比，套接字文件连接速度更快，但只能在连接到同一台计算机上的服务器时使用。要使用 Unix 套接字文件，我们不指定主机名或应指定特殊主机名 localhost。

以下是 MySQL 客户端在 Windows 上连接到 MySQL 服务器的方式：

+   **TCP/IP 连接**：与 Unix 系统之前描述的一样，TCP/IP 连接在指定的端口号上运行。MySQL 客户端连接到 MySQL 服务器正在监听的端口。

+   **命名管道连接**：MySQL 服务器可以使用`--enable-named-pipe`选项启动。如果客户端在运行服务器的主机上运行，则客户端可以使用命名管道连接。**MySQL**是命名管道的默认名称。如果在连接到`mysqld`服务器进程时未提供主机名，则 MySQL 首先尝试连接到默认命名管道。如果无法连接到命名管道，则尝试连接到 TCP/IP 端口。在 Windows 上可以通过使用`.`作为主机名来强制使用命名管道。

MySQL 错误由预定义的唯一错误代码标识。相同的错误可能与不同的错误代码相关联。具有错误代码`2002`的无法连接到 MySQL 服务器错误表示三个问题之一。可能是 MySQL 服务器没有在系统上运行，或者提供的 Unix 套接字文件名不正确，或者提供的用于连接到服务器的 TCP/IP 端口号不正确。TCP/IP 端口可能被防火墙或端口阻止服务阻止。

错误代码`2003`也与无法连接到 MySQL 服务器相关联。它表示服务器拒绝了网络连接。应该检查 MySQL 服务器是否启用了网络连接，MySQL 服务器是否正在运行，并且服务器上是否配置了指定的网络端口。

以下命令可用于确保`mysqld`服务器进程正在运行：

```go
> ps xa | grep mysqld
```

如果`mysqld`服务器进程没有运行，我们应该启动服务器。如果服务器已经运行，应使用以下命令：

```go
> mysqladmin version
> mysqladmin variables
> mysqladmin -h `hostname` version variables
> mysqladmin -h `hostname` --port=3306 version 
> mysqladmin -h host_ip version
> mysqladmin --protocol=SOCKET --socket=/tmp/mysql.sock version
```

在上述命令中，`hostname`是运行 MySQL 服务器的计算机的主机名。`host_ip`是服务器机器的 IP 地址。

# 与 MySQL 服务器的连接丢失

与 MySQL 服务器的连接丢失错误可能是由本节中解释的三种可能原因之一引起的。

错误的一个潜在原因是网络连接出现问题。如果这是一个频繁出现的错误，应该检查网络条件。如果错误消息中包含**during query**，那么可以肯定是由于网络连接问题导致了错误。

`connection_timeout`系统变量定义了`mysqld`服务器在连接超时响应之前等待连接数据包的秒数。很少情况下，当客户端尝试与服务器进行初始连接并且`connection_timeout`值设置为几秒时，可能会发生此错误。在这种情况下，可以通过根据距离和连接速度增加`connection_timeout`值来解决问题。`SHOW GLOBAL STATUS LIKE`和`Aborted_connects`可用于确定我们是否更频繁地遇到此问题。可以肯定地说，如果错误消息包含**reading authorization packet**，增加`connection_timeout`值就是解决方案。

可能会因为**Binary Large OBject**（**BLOB**）值大于`max_allowed_packet`而出现问题。这可能会导致客户端与 MySQL 服务器的连接丢失错误。如果观察到`ER_NET_PACKET_TOO_LARGE`错误，则确认应增加`max_allowed_packet`值。

# 密码输入错误时失败

当客户端程序在没有密码值的情况下使用`--password`或`-p`选项调用时，MySQL 客户端会要求密码。以下是命令：

```go
> mysql -u user_name -p
Enter password:
```

在一些系统上，当在选项文件或命令行中指定密码时，密码可以正常工作。但是，在`Enter password:`提示符处交互输入时，密码无法正常工作。这是因为系统提供的用于读取密码的库将密码值限制为少量字符（通常为八个）。这是系统库的问题，而不是 MySQL 的问题。作为解决方法，将 MySQL 密码更改为八个或更少字符的值，或将密码存储在选项文件中。

# 主机 host_name 被阻止

如果`mysqld`服务器从中断的主机接收了太多连接请求，将出现以下错误：

```go
Host 'host_name' is blocked because of many connection errors.
Unblock with 'mysqladmin flush-hosts'
```

`max_connect_errors`系统变量确定允许的连续中断连接请求的次数。一旦有`max_connect_errors`次失败的请求而没有成功的连接，`mysqld`就会认为出现了问题，并阻止主机进一步连接，直到发出`FLUSH HOSTS`语句或`mysqladmin flush-hosts`命令。

默认情况下，`mysqld`在 100 个连接错误后会阻止主机。可以通过在服务器启动时设置`max_connect_errors`值来进行调整，如下所示：

```go
> mysqld_safe --max_connect_errors=10000
```

此值也可以在运行时设置，如下所示：

```go
mysql> SET GLOBAL max_connect_errors=10000;
```

如果针对特定主机收到`host_name`被阻止的错误，首先应检查主机的 TCP/IP 连接是否存在问题。如果网络存在问题，则增加`max_connect_errors`变量的值是无济于事的。

# 连接过多

此错误表示所有可用连接都用于其他客户端连接。`max_connections`是控制与服务器连接数的系统变量。最大连接数的默认值为 151。我们可以为`max_connections`系统变量设置大于 151 的值，以支持超过 151 个连接。

`mysqld`服务器进程实际上允许比`max_connections`（`max_connections + 1`）值多一个连接。额外的一个连接被保留给具有`CONNECTION_ADMIN`或`SUPER`特权的帐户。管理员可以通过具有`PROCESS`特权的访问来授予该特权。有了这个访问权限，管理员可以使用保留的连接连接到服务器。他们可以执行`SHOW PROCESSLIST`命令来诊断问题，即使最大客户端连接数已用完。

# 内存不足

如果`mysql`没有足够的内存来存储 MySQL 客户端程序发出的查询的整个请求，服务器会抛出以下错误：

```go
mysql: Out of memory at line 42, 'malloc.c'
mysql: needed 8136 byte (8k), memory in use: 12481367 bytes (12189k)
ERROR 2008: MySQL client ran out of memory
```

为了解决问题，我们必须首先检查查询是否正确。我们是否期望查询返回这么多行？如果不是，我们应该纠正查询并再次执行。如果查询是正确的且不需要更正，我们可以使用`--quick`选项连接`mysql`。使用`--quick`选项会导致`mysql_use_result()` C API 函数用于获取结果集。该函数会增加服务器的负载，减少客户端的负载。

# 数据包太大

通信数据包是以下内容之一：

+   MySQL 客户端发送到 MySQL 服务器的单个 SQL 语句

+   从 MySQL 服务器发送到 MySQL 客户端的单行

+   从复制主服务器发送到复制从服务器的二进制日志事件

1 GB 数据包大小是可以传输到 MySQL 8 服务器或客户端的最大可能数据包大小。如果接收到大于`max_allowed_packet`字节的数据包，MySQL 服务器或客户端会发出`ER_NET_PACKET_TOO_LARGE`错误并关闭连接。

MySQL 客户端程序的默认`max_allowed_packet`大小为 16 MB。可以使用以下命令来设置更大的值：

```go
> mysql --max_allowed_packet=32M
```

MySQL 服务器的默认值为 64 MB。值得注意的是，为此系统变量设置更大的值是没有害处的，因为额外的内存会根据需要分配。

# 表已满

表已满错误发生在以下条件之一：

+   磁盘已满

+   表已达到最大大小

MySQL 数据库中的实际最大表大小可以通过操作系统对文件大小的限制确定。

# 无法创建/写入文件

如果在执行查询时出现以下错误，则表示 MySQL 无法在结果集的临时目录中创建临时文件：

```go
Can't create/write to file '\\sqla3fe_0.ism'.
```

错误的可能解决方法是使用`--tmpdir`选项启动`mysqld`服务器。以下是命令：

```go
> mysqld --tmpdir C:/temp
```

作为替代，可以在 MySQL 配置文件的`[mysqld]`部分中指定如下：

```go
[mysqld]
tmpdir=C:/temp
```

# 命令不同步

如果客户端函数的调用顺序错误，则会收到命令不同步的错误。这意味着命令无法在客户端代码中执行。例如，如果我们执行`mysql_use_result()`并在执行`mysql_free_result()`之前尝试执行另一个查询，则可能会出现此错误。如果我们在调用`mysql_use_result()`或`mysql_store_result()`函数之间执行两个返回结果集的查询，也可能会发生这种情况。

# 忽略用户

在`mysqld`服务器启动时或服务器重新加载授权表时，如果在用户表中找到具有无效密码的帐户，则会收到以下错误：

```go
Found wrong password for user 'some_user'@'some_host'; ignoring user
```

由于 MySQL 权限系统忽略了该帐户，因此会出现问题。为了解决问题，我们应该为该帐户分配一个新的有效密码。

# 表 tbl_name 不存在

以下错误表示默认数据库中不存在指定的表：

```go
Table 'tbl_name' doesn't exist
Can't find file: 'tbl_name' (errno: 2)
```

在某些情况下，用户可能会错误地引用表。这是可能的，因为 MySQL 服务器使用目录和文件来存储数据库表。根据操作系统文件管理的不同，数据库和表名可能区分大小写。

对于不区分大小写的文件系统，例如 Windows，在查询中使用的指定表的引用必须使用相同的字母大小写。

# MySQL 8 服务器错误

本节重点介绍 MySQL 8 服务器错误。该部分描述了与 MySQL 服务器管理、表定义和 MySQL 8 服务器中已知问题相关的错误。

# 文件权限问题

如果在服务器启动时设置了`UMASK`或`UMASK_DIR`环境变量，则可能会出现文件权限问题。在表创建时，MySQL 服务器可能会发出以下错误消息：

```go
ERROR: Can't find file: 'path/with/file_name' (Errcode: 13)
```

`UMASK`和`UMASK_DIR`系统变量的默认值分别为 0640 和 0750。如果这些环境变量的值以零开头，则表示 MySQL 服务器的值是八进制的。例如，八进制中的默认值 0640 和 0750 分别等于十进制的 415 和 488。

为了更改默认的`UMASK`值，我们应该启动`mysqld_safe`，如下所示：

```go
> UMASK=384 # = 600 in octal 
> export UMASK 
> mysqld_safe
```

MySQL 服务器创建数据库目录时的默认访问权限值为`0750`。我们可以设置`UMASK_DIR`变量来修改这种行为。如果设置了此值，新目录将以`UMASK`和`UMASK_DIR`值的组合作为访问权限值创建。

以下是提供所有新目录组访问权限的示例：

```go
> UMASK_DIR=504 # = 770 in octal 
> export UMASK_DIR 
> mysqld_safe &
```

# 重置根密码

如果在 MySQL 中从未设置根密码，则 MySQL 服务器连接为根用户时不需要密码。如果之前分配的密码被遗忘，可以进行重置。

以下是在 Windows 系统上重置`root @ localhost`帐户密码的说明：

1.  使用系统管理员凭据登录系统。

1.  如果 MySQL 服务器已经在运行，请停止服务器。如果 MySQL 服务器作为 Windows 服务运行，请按照开始菜单|控制面板|管理工具|服务找到**服务**。在服务中，找到 MySQL 服务并停止它。如果 MySQL 服务器没有作为 Windows 服务运行，请使用 Windows 任务管理器杀死 MySQL 服务器进程。

1.  一旦 MySQL 服务器停止，创建一个包含密码分配语句的单行文本文件，如下所示：

```go
 ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword';
```

1.  保存文件。例如，将文件保存为`C:\mysql-root-reset.txt`。

1.  按照开始菜单|运行|cmd 打开 Windows 命令提示符。

1.  在命令提示符中，使用`--init-file`选项启动 MySQL 服务器，如下所示：

```go
 C:\> cd "C:\Program Files\MySQL\MySQL Server 8.0\bin" 
        C:\> mysqld --init-file=C:\\mysql-root-reset.txt
```

1.  一旦 MySQL 服务器重新启动，删除`C:\mysql-root-reset.txt`文件。

以下是在类 Unix 系统上重置根用户密码的说明：

1.  使用与 MySQL 服务器运行的相同用户登录系统。通常是`mysql`用户。

1.  如果 MySQL 服务器已经运行，请停止服务器。为此，找到包含 MySQL 服务器进程 ID 的`.pid`文件。根据 Unix 发行版的不同，文件的实际位置和名称可能不同。通常的位置是`/var/lib/mysql/`，`/var/run/mysqld/`和`/usr/local/mysql/data/`。通常，文件名以`mysqld`或系统主机名开头，并具有`.pid`扩展名。可以通过向`mysqld`服务器进程发送正常的 kill 命令来停止 MySQL 服务器。可以使用以下命令和`.pid`文件的实际路径名：

```go
 > kill 'cat /mysql-data-directory/host_name.pid'
```

1.  一旦 MySQL 服务器停止，创建一个包含密码赋值语句的文本文件，如下所示：

```go
 ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword';
```

1.  保存文件。假设文件存储在`/home/me/mysql-reset-root`。由于文件包含 root 用户的密码，应确保其他用户无法读取它。如果我们没有使用适当的用户登录，我们应该确保用户有权限读取该文件。

1.  使用`--init-file`选项启动 MySQL 服务器，如下所示：

```go
 > mysqld --init-file=/home/me/mysql-reset-root &
```

1.  一旦服务器启动，删除`/home/me/mysql-reset-root`中的文件。

以下是重置 root 用户密码的通用说明：

1.  如果 MySQL 服务器正在运行，请停止服务器。一旦停止，使用`--skip-grant-tables`特权重新启动 MySQL 服务器。除了`--skip-grant-tables`，`--skip-networking`选项会自动启用，以防止远程连接。

1.  使用`mysql`客户端程序连接到 MySQL 服务器。由于服务器是使用`--skip-grant-tables`启动的，因此不需要密码：

```go
 > mysql
```

1.  在 MySQL 客户端本身中，要求服务器重新加载授予表。这将启用帐户管理语句：

```go
 mysql> FLUSH PRIVILEGES;
```

1.  使用以下命令更改`root @ localhost`帐户密码：

```go
 mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 
          'NewPassword';
```

1.  重新启动服务器并使用 root 用户和新设置的密码登录。

# MySQL 崩溃预防

作为标准发布实践，每个 MySQL 版本在发布之前都会在不同的平台上进行验证。假设 MySQL 可能有一些难以发现的错误。当我们遇到 MySQL 的问题时，如果我们尝试找出系统崩溃的原因，这将是有帮助的。首先要确定的是`mysqld`服务器进程是否崩溃，或者问题出现在 MySQL 客户端程序上。可以通过执行`mysqladmin version`命令来检查 MySQL 服务器运行了多长时间。以下是一个示例输出：

```go
C:\Program Files\MySQL\MySQL Server 8.0\bin>mysqladmin version -u root -p
Enter password: *****
mysqladmin Ver 8.0.3-rc for Win64 on x86_64 (MySQL Community Server (GPL))
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Server version 8.0.3-rc-log
Protocol version 10
Connection localhost via TCP/IP
TCP port 3306
Uptime: 9 days 4 hours 4 min 52 sec

Threads: 2 Questions: 4 Slow queries: 0 Opens: 93 Flush tables: 2 Open tables: 69 Queries per second avg: 0.000
```

`resolve_stack_dump`是一个实用程序，用于将数字堆栈转储解析为符号。为了分析`mysqld`服务器进程死机的根本原因，我们在堆栈跟踪错误日志中找到。可以使用`resolve_stack_dump`程序解决这个问题。必须注意的是，错误日志中找到的变量值可能不准确。

损坏的数据或索引文件可能会导致 MySQL 服务器崩溃。这些文件在执行每个 SQL 语句之前和客户端被通知结果之前使用`write()`系统在磁盘上进行更新。这意味着即使在`mysqld`崩溃的情况下，数据文件中的内容也是安全的。未刷新的数据在磁盘上的写入由操作系统负责。`--flush`选项可以与`mysqld`一起使用，以强制 MySQL 在每个 SQL 语句执行后将所有内容刷新到磁盘上。

以下是 MySQL 损坏表的原因之一：

+   如果数据文件或索引文件崩溃，则其中包含损坏的数据。

+   MySQL 服务器进程中的一个错误导致服务器在更新过程中崩溃。

+   外部程序在没有表锁定的情况下与`mysqld`同时操纵数据和索引文件。

+   在更新过程中，MySQL 服务器进程被终止。

+   许多`mysqld`服务器正在系统上运行。这些服务器使用相同的数据目录。系统没有良好的文件系统锁定，或者外部锁定被禁用。

+   可能会发现数据存储代码中存在错误。我们可以尝试通过在已修复的表的副本上使用`ALTER TABLE`来更改存储引擎。

# 处理 MySQL 磁盘满

本节重点介绍 MySQL 对磁盘满错误和超出配额错误的响应。它更相关于`MyISAM`表中的写入。它可以应用于二进制日志文件和索引文件的写入。它排除了应被视为事件的行和记录的引用。

当磁盘满时，MySQL 执行以下操作：

+   MySQL 确保有足够的空间可用来写入当前行。

+   MySQL 服务器每 10 分钟在日志文件中写入一条条目。它会警告磁盘满的情况。

应采取以下措施来解决问题：

+   应该释放磁盘空间，以确保有足够的空间来插入所有记录。

+   我们可以执行`mysqladmin kill`命令来中止线程。下次检查磁盘时，线程将被中止。

+   可能会有一些线程在等待导致磁盘满的表。在多个被锁定的线程中，杀死等待磁盘满条件的线程将使其他线程继续运行。

+   `REPAIR TABLE`或`OPTIMIZE TABLE`语句是前述条件的例外。其他例外包括在`LOAD DATA INFILE`或`ALTER TABLE`语句之后批量创建的索引。这些 SQL 语句可能会创建大量的临时文件。这可能会给系统的其余部分带来大问题。

# MySQL 临时文件存储

`TMPDIR`环境变量的值在 Unix 上被 MySQL 用作存储临时文件的目录路径名。如果未设置`TMPDIR`，MySQL 会使用系统默认值，如`/tmp`、`/var/tmp`或`/usr/tmp`。

MySQL 在 Windows 上会检查`TMPDIR`、`TEMP`和`TMP`环境变量的值。如果 MySQL 找到已设置的变量，它将使用该值，并不会检查剩余的值。如果这三个变量都未设置，MySQL 将使用系统默认值，即`C:\windows\temp\`。

如果文件系统中的临时文件目录太小，我们可以使用`mysqld --tmpdir`选项来指定具有足够空间的文件系统上的目录。对于复制，在从服务器上，我们可以使用`--slave-load-tmpdir`并指定在复制`LOAD DATA INFILE`语句期间保存临时文件的目录。可以使用`--tmpdir`选项以轮询方式设置多个路径的列表。在 Unix 系统上，路径可以用冒号字符(:)分隔，而在 Windows 上，可以用分号字符(;)分隔路径。

为了有效地分配负载，多个临时目录路径应该属于不同的物理磁盘，而不是同一磁盘的不同分区。

对于作为复制从服务器工作的 MySQL 服务器，我们必须注意设置`--slave-load-tmpdir`选项，以免指向基于内存的文件系统中的目录，或者指向在服务器或服务器主机重新启动时清除的目录。为了复制临时表或`LOAD DATA INFILE`操作，复制从服务器需要在机器重新启动时其临时文件。如果临时文件目录中的文件丢失，复制将失败。

当`mysqld`服务器进程终止时，MySQL 会负责删除临时文件。在类 Unix 平台上，可以在打开文件后取消链接文件。这样做的一个主要缺点是该文件名不会出现在目录列表中。还有可能我们看不到占用文件系统的大文件。

`ibtmp1`是`InnoDB`存储引擎用来存储临时表的表空间文件的名称。该文件位于 MySQL 的数据目录中。如果我们想要指定不同的文件名和位置，可以在服务器启动时使用`innodb_temp_data_file_path`选项。

如果`ALTER TABLE`操作在`InnoDB`表上使用`ALGORITHM=COPY`技术，存储引擎会在相同目录中创建原始表的临时副本。临时表的文件名以`#sql-`前缀开头。它们只在执行`ALTER TABLE`操作时短暂出现。

如果使用`ALGORITHM=INPLACE`方法通过`ALTER TABLE` SQL 语句重建`InnoDB`表，则`InnoDB`存储引擎会在与原始表相同的目录中创建原始表的中间副本。中间表的文件名以`#sql-ib`前缀开头。它们只在执行`ALTER TABLE`操作时短暂出现。

`innodb_tmpdir`选项不能应用于中间表文件。这些中间文件始终在与原始表相同的目录中创建和存储。

使用`ALGORITHM=INPLACE`方法重建`InnoDB`表的`ALTER TABLE` SQL 语句会在默认的 MySQL 临时目录中创建临时排序文件。默认临时目录由 Unix 上的`$TMPDIR`，Windows 上的`%TEMP%`或`--tmpdir`选项指定的目录表示。如果临时目录不足以存储这样的文件，可能需要重新配置`tmpdir`。作为替代方案，可以使用`innodb_tmpdir`选项为在线`InnoDB` `ALTER TABLE`语句定义另一个临时目录。`innodb_tmpdir`选项可以在运行时使用`SET GLOBAL`或`SET SESSION`语句进行配置。

在复制环境中，如果所有服务器具有相同的操作系统环境，则应考虑复制`innodb_tmpdir`配置。在其他情况下，`innodb_tmpdir`设置复制可能会导致在线`ALTER TABLE`操作执行失败。如果操作环境不同，建议为每台服务器单独配置`innodb_tmpdir`。

# MySQL Unix 套接字文件

MySQL 服务器使用`/tmp/mysql.sock`作为与本地客户端通信的 Unix 套接字文件的默认位置。根据不同的发行格式，如 RPMs 的`/var/lib/mysql`，可能会有所不同。

在几个 Unix 版本上，可以删除存储在`/tmp`目录和其他类似目录中的文件。如果套接字文件存储在文件系统上的这种目录中，可能会导致问题。

可以保护`/tmp`目录，以确保文件只能由所有者或 root 超级用户删除。这在几乎每个版本的 Unix 上都是可能的。可以在以 root 用户登录时设置`/tmp`目录的粘滞位。以下是执行相同操作的命令：

```go
chmod +t /tmp
```

使用`ls -ld /tmp`命令，还可以检查粘滞位是否已设置。如果最后一个权限字符是`t`，则设置了该位。粘滞位用于定义 Unix 系统中的文件权限。

还有一种替代方法，即更改 Unix 套接字文件的位置。如果更改 Unix 套接字文件的位置，必须确保客户端程序也知道文件的新位置。以下是实现这一点的方法：

+   可以在全局或本地选项文件中设置路径，如下所示：

```go
 [mysqld]
 socket=/path/to/socket

 [client]
 socket=/path/to/socket
```

+   +   我们还可以在命令行上为`mysqld_safe`指定`--socket`选项，并在运行客户端程序时也可以这样做。

+   `MYSQL_UNIX_PORT`环境变量可以设置为 Unix 套接字文件的路径。

+   MySQL 也可以重新从源代码编译，以便将不同的 Unix 套接字文件位置作为默认值使用。

使用以下命令，可以确保新的套接字位置有效：

```go
mysqladmin --socket=/path/to/socket version
```

# 时区问题

MySQL 服务器必须告知用户当前的时区，如果我们在使用`SELECT NOW()`时返回的是 UTC 时间而不是用户当前的时区。如果`UNIX_TIMESTAMP()`返回错误的数值也适用。这应该针对运行服务器的环境进行设置，例如`mysqld_safe`或`mysql.server`。

我们还可以使用`--timezone=timezone_name`选项与`mysqld_safe`一起设置服务器时区。也可以在启动`mysqld`之前将值分配给`TZ`环境变量来设置时区。

`--timezone`或`TZ`的允许值列表取决于系统。

# MySQL 8 客户端错误

本节重点介绍 MySQL 8 客户端出现的错误。MySQL 客户端的工作是连接到 MySQL 服务器，以执行 SQL 查询并从 MySQL 8 数据库获取结果。本节列出了与查询执行相关的错误。

# 字符串搜索中的区分大小写

字符串搜索使用非二进制字符串的比较操作数的逻辑顺序，例如`CHAR`，`VARCHAR`和`TEXT`。二进制字符串的比较，如`BINARY`，`VARBINARY`和`BLOB`使用操作数中字节的数值。这基本上意味着对于字母字符，比较将区分大小写。

将非二进制字符串与二进制字符串进行比较将被视为二进制字符串之间的比较。

比较操作，如`>=`, `>`, `=`, `<`, `<=`, `sorting`和`grouping`取决于每个字符的排序值。具有相似排序值的字符被视为相同字符。以 e 和é为例。这些字符在提供的逻辑顺序中具有相同的排序值。这些被视为相等。

`utf8mb4`和`utf8mb4_0900_ai_ci`分别是默认的字符集和排序规则。默认情况下，非二进制字符串比较是不区分大小写的。这意味着如果我们使用`col_name LIKE 'a%'`搜索，我们将得到所有以 A 或 a 开头的列值。要使其区分大小写，我们必须确保其中一个操作数具有二进制或区分大小写的排序规则。例如，如果将列与字符串进行比较，并且两者都具有`utf8mb4`字符集，则可以使用`COLLATE`运算符来使其中一个操作数具有`utf8mb4_0900_as_cs`或`utf8mb4_bin`排序规则。以下是一个示例：

```go
col_name COLLATE utf8mb4_0900_as_cs LIKE 'a%' 
col_name LIKE 'a%' COLLATE utf8mb4_0900_as_cs 
col_name COLLATE utf8mb4_bin LIKE 'a%' 
col_name LIKE 'a%' COLLATE utf8mb4_bin
```

为了将非二进制区分大小写字符串比较更改为不区分大小写，我们应该使用`COLLATE`来命名一个不区分大小写的排序规则。以下是`COLLATE`如何将比较更改为区分大小写的示例：

```go
mysql> SET NAMES 'utf8mb4'; 
mysql> SET @s1 = 'MySQL' COLLATE utf8mb4_bin, @s2 = 'mysql' COLLATE utf8mb4_bin; mysql> SELECT @s1 = @s2;
+-----------+ 
| @s1 = @s2 | 
+-----------+ 
|         0 | 
+-----------+ 
mysql> SELECT @s1 COLLATE utf8mb4_0900_ai_ci = @s2; 
+--------------------------------------+ 
| @s1 COLLATE utf8mb4_0900_ai_ci = @s2 | 
+--------------------------------------+ 
|                                    1 | 
+--------------------------------------+
```

# DATE 列的问题

在 MySQL 中，`DATE`值的默认格式是`YYYY-MM-DD`。标准 SQL 不允许任何其他格式。这是在`UPDATE`表达式和`SELECT`语句的`WHERE`子句中应该使用的格式。以下是日期格式的示例：

```go
SELECT * FROM table_name WHERE date_col >= '2011-06-02';
```

当将常量字符串与`DATE`，`TIME`，`DATETIME`或`TIMESTAMP`使用`<`, `<=`, `=`, `>=`, `>`,或`BETWEEN`运算符进行比较时，MySQL 将字符串转换为内部长整数值。MySQL 这样做是为了实现更快的比较。然而，以下例外情况适用于此转换：

+   比较两列

+   将`DATE`，`TIME`，`DATETIME`或`TIMESTAMP`列与表达式进行比较

+   使用除列出的方法之外的比较方法，如`IN`或`STRCMP()`

在这些例外情况下，通过将对象转换为字符串值并执行字符串比较来进行比较。

# NULL 值的问题

`NULL`值经常让新程序员感到困惑。`NULL`值在字符串的情况下被错误地解释为空字符串`''`。这是不正确的。以下是完全不同的语句的示例：

```go
mysql> INSERT INTO my_table (phone) VALUES (NULL); 
mysql> INSERT INTO my_table (phone) VALUES ('');
```

在上面的例子中，两个语句都将值插入到同一列（phone 列）中。第一个语句插入一个`NULL`值，而第二个语句插入一个空字符串。第一个值可以被认为是电话号码未知，而第二个值表示该人已知没有电话，因此没有电话号码。

当`NULL`值与任何其他值进行比较时，它总是评估为假。包含`NULL`值的表达式总是返回`NULL`值。以下示例返回一个`NULL`值：

```go
mysql> SELECT NULL, 1+NULL, CONCAT('Invisible',NULL);
```

如果 SQL 语句的目的是搜索`NULL`列值，我们不能使用`expression = NULL`。以下是一个示例，返回零行，因为`expression = NULL`始终为假：

```go
mysql> SELECT * FROM my_table WHERE phone = NULL;
```

要进行`NULL`值比较，应该使用`IS NULL`。以下示例演示了`IS NULL`的使用：

```go
mysql> SELECT * FROM my_table WHERE phone IS NULL; 
mysql> SELECT * FROM my_table WHERE phone = '';
```

# MySQL 8 故障排除方法

在本章的这一部分，我们将专注于 MySQL 8 的故障排除方法。我们为什么需要排除 MySQL 8 的故障？排除故障的原因如下：

+   更快地执行 SQL 查询

+   性能增强

+   资源的有效利用

主要的资源集包括 CPU、磁盘 IO、内存和网络。有两种方法来衡量 MySQL 的性能：

+   在查询集中的方法中，重要的是要衡量查询的执行速度。

+   在资源集中的方法中，查询使用更少的资源是很重要的。

让我们深入了解如何排除 MySQL 问题。

# 分析查询

`EXPLAIN`是提供 MySQL 执行 SQL 语句信息的 SQL 语句。`EXPLAIN`语句与`INSERT`、`UPDATE`、`REPLACE`、`DELETE`和`SELECT`语句一起使用。`EXPLAIN`语句的输出是对`SELECT`语句中提到或使用的每个表的信息行。输出按照 MySQL 在执行语句时读取这些表的顺序列出。所有连接都使用嵌套循环连接方法解析。在嵌套循环连接方法中，MySQL 从列表中的第一个表中读取一行，然后在列表中的第二个表中找到匹配的行，然后是第三个表，依此类推。一旦处理完列表中的所有表，MySQL 处理所选列的结果，并通过表的列表回溯，直到找到具有更多匹配行的表。它从这个表中读取下一行。这样的过程继续进行。

以下是来自`EXPLAIN`输出的列：

+   `id`**：这表示查询中的`SELECT`的顺序号。它也被称为`SELECT`标识符。当行属于其他行的联合结果时，该值可能为`NULL`。输出在表列中显示`<unionM, N>`。这意味着该行是 ID 值`M`和`N`的联合。

+   `select_type`：此输出列指示`SELECT`语句的类型。可能的值列表包括`SIMPLE`、`PRIMARY`、`UNION`、`DEPENDENT UNION`、`UNION RESULT`、`SUBQUERY`、`DEPENDENT SUBQUERY`、`DERIVED`、`MATERIALIZED`、`UNCACHEABLE SUBQUERY`和`UNCACHEABLE UNION`。

+   `table`：此列指示输出中提到的表的名称。它可以具有诸如`<unionM, N>`、`<derivedN>`和`<subqueryN>`之类的值。

+   `partitions`：这标识查询匹配记录的分区。对于非分区表，该值为`NULL`。

+   `类型`：这表示`JOIN`的类型。

+   `possible_keys`：此输出列指示 MySQL 可能选择用于获取表中行的索引。如果没有匹配的索引，返回值将为`NULL`。

+   `key`：此输出列指示 MySQL 实际用于从表中获取行的关键索引。

+   `ref`：`ref`输出列指示用于与键输出列中提到的索引进行比较以选择表行的列或常量。

+   `rows`：行输出列指示为成功执行查询需要检查的行数。

在`EXPLAIN`中有以下类型的连接：

+   `system`：这意味着表只有一行。这是`const`连接类型的特殊情况。

+   `const`：这意味着表至少有一行匹配。这一行在查询开始时被读取。由于只找到一行匹配，优化器的其余部分将这一行中的列值视为常量。由于 const 表只被读取一次，所以非常快。当`PRIMARY KEY`或`UNIQUE`索引的所有部分与常量值进行比较时，使用 const。以下是一个使用`tbl_name`作为 const 表的示例：

```go
 mysql> SELECT * FROM tbl_name WHERE primary_key=1; 
 mysql> SELECT * FROM tbl_name WHERE primary_key_part1=1 AND primary_key_part2=2;
```

+   `ref`：对于前面表的每个行组合，从`ref`表中读取所有具有匹配索引值的行。如果连接只使用了键的最左前缀，则使用`ref`。

# 现实世界的场景

MySQL 查询优化是指提高查询执行时间。例如，当一个查询性能不佳时，意味着查询执行时间比预期的时间长。查询执行时间很重要，但还有其他指标用于衡量性能。本节解释了应该测量什么以及如何尽可能精确地进行测量。

以下问题出现了：为什么我们应该优化查询？如果只需要百分之一秒，真的需要优化吗？是的，除非查询很少执行，否则确实需要优化。我们应该优化最昂贵的查询。

让我们讨论一个实时的例子。在某个应用程序中，我们有一个基于复杂查询生成的报告，花费了太多时间。执行时间是以分钟计算的。为了优化这样一个复杂的查询，我们考虑了以下方法：

1.  **使用`EXPLAIN`分析查询计划**：MySQL 提供了两种分析查询性能的方法。一种是`EXPLAIN`方法，我们已经在本章的前一部分学习过。另一个工具是`SHOW STATUS`。通常，我们应该优先使用`EXPLAIN`来理解`SELECT`查询的查询计划。在报告查询的情况下，我们将一些非`SELECT`查询转换为`SELECT`查询。这有助于我们理解非`SELECT`查询的查询执行计划。例如，通过在`UPDATE`查询中使用`WHERE`子句，我们可以将其转换为`SELECT`查询。我们还可以找到表上缺少的索引。

1.  `SHOW STATUS`：`SHOW STATUS`语句输出 MySQL 的内部计数器。这些计数器在每次查询执行时由 MySQL 递增。借助这些计数器，我们可以了解服务器的聚合操作类型。它还有助于指示每个单独查询所做的工作。

以下是对 MySQL 服务器变量执行的测量：

+   `Select_`：每次执行`SELECT`查询时，此计数器会递增。此计数器还可用于确定是否执行了表扫描。

+   `Key_read`：此变量提供了关于键索引使用情况的额外信息。

+   `Last_query_cost`：这个值表示上次执行的查询有多昂贵。

以下是执行查询优化的步骤：

1.  多次执行查询以确保返回相同的结果。

1.  执行`SHOW STATUS`。保存输出。

1.  执行查询。

1.  执行`SHOW STATUS`以观察与上一次执行的差异。

1.  如果需要，执行`EXPLAIN`。

应该分析以下参数以优化查询性能：

+   表索引

+   排序

+   整体性能

+   行级操作

+   磁盘 I/O 操作

# 总结

在这本书的最后一章中，我们学习了数据库的一个重要方面：解决我们在使用 MySQL 服务器或客户端时可能遇到的错误。我们从理解故障排除开始讨论。我们讨论了初步诊断错误的不同方法。我们了解了常见的 MySQL 错误以及错误消息的含义。我们还学习了如何修复这些错误。我们还了解了 MySQL 服务器和客户端的错误以及这些错误的修复方法。在本章的后半部分，我们学习了 MySQL 故障排除方法，并看了一个真实的案例。对于最后一章来说，这是相当重要的内容，是吧？这本书就到这里了。
