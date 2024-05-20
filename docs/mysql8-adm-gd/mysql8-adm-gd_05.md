# 第五章：MySQL 8 数据库管理

在上一章中，我们学习了 MySQL 8 数据类型，详细解释了可用的数据类型及其分类。每种数据类型都有各种属性，存储容量也因类型而异。上一章还为您提供了对 MySQL 8 数据类型的深入了解。现在是时候获得一些关于 MySQL 8 管理功能的实际知识了。了解更多关于 MySQL 8 管理功能的信息，如何为其进行配置等，这难道不是很有趣吗？对于管理员来说，详细了解 MySQL 8 的全球化工作原理、如何维护日志以及如何增强服务器的功能非常重要。现在，让我们从一些基本概念开始。

本章将涵盖以下主题：

+   MySQL 8 服务器管理

+   数据目录

+   系统数据库

+   在单台机器上运行多个实例

+   组件和插件管理

+   角色和权限

+   缓存技术

+   全球化

+   MySQL 8 服务器日志

# MySQL 8 服务器管理

MySQL 8 有许多可用的操作参数，其中所有必需的参数在安装过程中默认设置。安装后，您可以通过删除或添加特定参数设置行的注释符（`#`）来更改**选项文件**。用户还可以使用命令行参数或选项文件在运行时设置参数。

# 服务器选项和不同类型的变量

在本节中，我们将介绍 MySQL 8 启动时可用的**服务器选项**、**系统变量**和**状态变量**。

+   服务器选项：如前一章所述，MySQL 8 使用选项文件和命令行参数来设置启动参数。有关所有可用选项的详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/mysqld-option-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqld-option-tables.html)。`mysqld`接受许多命令选项。要获得简要摘要，请执行以下命令：

```sql
 mysqld --help
```

要查看完整列表，请使用以下命令：

```sql
 mysqld –verbose --help
```

+   **服务器系统变量**：MySQL 服务器管理许多系统变量。MySQL 为每个系统变量提供默认值。系统变量可以使用命令行设置，也可以在选项文件中定义。MySQL 8 具有在运行时更改这些变量的灵活性，无需服务器启动或停止。有关更多详细信息，请参阅：[`dev.mysql.com/doc/refman/8.0/en/server-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)。

+   **服务器状态变量**：MySQL 服务器使用许多状态变量来提供有关其操作的信息。有关更多详细信息，请参阅：[`dev.mysql.com/doc/refman/8.0/en/server-status-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/server-status-variables.html)。

# 服务器 SQL 模式

MySQL 8 提供了不同的模式，这些模式将影响 MySQL 支持和数据验证检查。此选项使用户更容易在不同环境中使用 MySQL。为了设置不同的模式，MySQL 提供了`sql_mode`系统变量，可以在全局或会话级别设置。详细了解模式的以下要点：

# 设置 SQL 模式

可以使用`--sql-mode="modes"`选项在启动时设置 SQL 模式。用户还可以在选项文件中定义此选项为`sql-mode="modes"`*。*您可以通过添加逗号分隔的值来定义多个节点。MySQL 8 默认使用以下模式：`ONLY_FULL_GROUP_BY`、`STRICT_TRANS_TABLES`、`NO_ZERO_IN_DATE`、`NO_ZERO_DATE`、`ERROR_FOR_DIVISION_BY_ZERO`*、*`NO_AUTO_CREATE_USER, NO_ENGINE_SUBSTITUTION`。要在运行时更改模式，请执行以下命令：

```sql
SET GLOBAL sql_mode = 'modes';
SET SESSION sql_mode = 'modes';
```

要检索这两个变量的值，请执行以下命令：

```sql
SELECT @@GLOBAL.sql_mode;
SELECT @@SESSION.sql_mode;
```

# 可用的 SQL 模式

此部分描述了所有可用的 SQL 模式。其中，前三个是最重要的 SQL 模式：

+   `ANSI`：此模式用于更改语法和行为，使其更接近标准 SQL。

+   `STRICT_TRANS_TABLES`：顾名思义，此模式与事务有关，主要用于事务存储引擎。当此模式对非事务表启用时，MySQL 8 将无效值转换为最接近的有效值，并将调整后的值插入列中。如果值丢失，则 MySQL 8 将插入与列数据类型相关的隐式默认值。在这种情况下，MySQL 8 将生成警告消息而不是错误消息，并继续执行语句而不中断。然而，在事务表的情况下，MySQL 8 会给出错误并中断执行。

+   `TRADITIONAL`：此模式通常表现得像传统的 SQL 数据库系统。它表示在将不正确的值插入列时产生错误而不是警告。

+   `ALLOW_INVALID_DATES`：此模式仅检查日期值的月份范围和日期范围。换句话说，月份范围必须在 1 到 12 之间，日期范围必须在 1 到 31 之间。此模式适用于`DATE`和`DATETIME`数据类型，而不适用于`timestamp`数据类型。

+   `ANSI_QUOTES`：用于将`"`视为标识符引用字符而不是字符串引用字符。当启用此模式时，您不能使用双引号引用字符串文字。

+   `ERROR_FOR_DIVISION_BY_ZERO`：用于处理除以零的情况。此模式的输出还取决于严格的 SQL 模式状态：

+   如果未启用此模式，除以零会插入`NULL`并且不会产生警告。

+   如果启用了此模式，除以零会插入`NULL`并产生警告。

+   如果启用了此模式和严格模式，除以零会产生错误，除非也给出了`IGNORE`。对于`INSERT IGNORE`和`UPDATE IGNORE`，除以零会插入`NULL`并产生警告。

+   `HIGH_NOT_PRECEDENCE`：此模式用于为`NOT`运算符设置高优先级。例如，当启用此模式时，表达式`NOT a BETWEEN b AND c`被解析为`NOT (a BETWEEN b AND c)`而不是`(NOT a) BETWEEN b AND c`。

+   `IGNORE_SPACE`：此模式适用于内置函数，而不适用于用户定义的函数或存储过程。

+   `NO_AUTO_CREATE_USER`：此模式用于防止通过自动创建新用户帐户的`GRANT`语句。

+   `NO_AUTO_VALUE_ON_ZERO`：此模式用于自动增量列。当找到 0 时，MySQL 为该字段创建一个新的序列号，这在加载转储时会造成问题。在重新加载转储之前启用此模式以解决此问题。

+   `NO_BACKSLASH_ESCAPES`：如果启用此模式，反斜杠将成为普通字符。

+   `NO_DIR_IN_CREATE`：此选项对于从属复制服务器非常有用，在表创建时会忽略`INDEX DIRECTORY`和`DATA DIRECTORY`指令。

+   `NO_ENGINE_SUBSTITUTION`：用于提供默认存储引擎的替换。当启用此模式并且所需的引擎不可用时，MySQL 会给出错误，表不会被创建。

+   `NO_FIELD_OPTIONS`：这表示，在`SHOW_CREATE_TABLE`的输出中不打印 MySQL 特定的列选项。

+   `NO_KEY_OPTIONS`：这表示，在`SHOW_CREATE_TABLE`的输出中不打印 MySQL 特定的索引选项。

+   `NO_TABLE_OPTIONS`：这表示，在`SHOW_CREATE_TABLE`的输出中不打印 MySQL 特定的表选项。

+   `NO_UNSIGNED_SUBTRACTION`：当启用此模式时，它确保减法结果必须是有符号值，即使操作数中的任何一个是无符号的。

+   `NO_ZERO_DATE`：此模式的效果取决于下面定义的严格模式：

+   如果未启用，允许使用 0000-00-00，MySQL 在插入时不会产生警告

+   如果启用此模式，则允许 0000-00-00，并且 MySQL 记录警告

+   如果同时启用此模式和严格模式，则不允许 0000-00-00，并且 MySQL 在插入时产生错误

+   `NO_ZERO_IN_DATE`：此模式的影响也取决于如下定义的严格模式：

+   如果未启用，允许具有零部分的日期，并且 MySQL 在插入时不会产生警告

+   如果启用此模式，则允许具有零部分的日期并产生警告

+   如果启用此模式和严格模式，则不允许具有零部分的日期，并且 MySQL 在插入时产生错误

+   `ONLY_FULL_GROUP_BY`：如果启用此模式，MySQL 将拒绝查询，其中`select`列表，`order by`列表和`HAVING`条件引用非聚合列。

+   `PAD_CHAR_TO_FULL_LENGTH`：此模式适用于数据类型设置为`CHAR`的列。启用此模式时，MySQL 通过填充以获取列值的完整长度。

+   `PIPES_AS_CONCAT`：当启用此模式时，`| |`将被视为字符串连接运算符，而不是`OR`。

+   `REAL_AS_FLOAT`：默认情况下，MySQL 8 将`REAL`视为`DOUBLE`的同义词，但当启用此标志时，MySQL 将`REAL`视为`FLOAT`的同义词。

+   `STRICT_ALL_TABLES`：在此模式下，无效的数据值将被拒绝。

+   `TIME_TRUNCATE_FRACTIONAL`：此模式指示是否允许对`TIME`，`DATE`和`TIMESTAMP`列进行截断。默认行为是对值进行四舍五入而不是截断。

# 组合 SQL 模式

MySQL 8 还提供了一些特殊模式，作为模式值的组合：

+   `ANSI`：它包括`REAL_AS_FLOAT`，`PIPES_AS_CONCAT`，`ANSI_QUOTES`，`IGNORE_SPACE`和`ONLY_FULL_GROUP_BY`模式的影响。

+   `DB2`：它包括`PIPES_AS_CONCAT`，`ANSI_QUOTES`，`IGNORE_SPACE`，`NO_KEY_OPTIONS`，`NO_TABLE_OPTIONS`和`NO_FIELD_OPTIONS`模式的影响。

+   `MAXDB`：它包括`PIPES_AS_CONCAT`，`ANSI_QUOTES`，`IGNORE_SPACE`，`NO_KEY_OPTIONS`，`NO_TABLE_OPTIONS`，`NO_FIELD_OPTIONS`和`NO_AUTO_CREATE_USER`的影响。

+   `MSSQL`：它包括`PIPES_AS_CONCAT`，`ANSI_QUOTES`，`IGNORE_SPACE`，`NO_KEY_OPTIONS`，`NO_TABLE_OPTIONS`和`NO_FIELD_OPTIONS`的影响。

+   `MYSQL323`：它包括`MYSQL323`和`HIGH_NOT_PRECEDENCE`模式的影响。

+   `MYSQL40`：它包括`MYSQL40`和`HIGH_NOT_PRECEDENCE`模式的影响。

+   `ORACLE`：它包括`PIPES_AS_CONCAT`，`ANSI_QUOTES`，`IGNORE_SPACE`，`NO_KEY_OPTIONS`，`NO_TABLE_OPTIONS`，`NO_FIELD_OPTIONS`和`NO_AUTO_CREATE_USER`模式的影响。

+   `POSTGRESQL`：它包括`PIPES_AS_CONCAT`，`ANSI_QUOTES`，`IGNORE_SPACE`，`NO_KEY_OPTIONS`，`NO_TABLE_OPTIONS`和`NO_FIELD_OPTIONS`模式的影响。

+   `TRADITIONAL`：它包括`STRICT_TRANS_TABLES`，`STRICT_ALL_TABLES`，`NO_ZERO_IN_DATE`，`NO_ZERO_DATE`，`ERROR_FOR_DIVISION_BY_ZERO`，`NO_AUTO_CREATE_USER`和`NO_ENGINE_SUBSTITUTION`模式的影响。

# 严格的 SQL 模式

**严格模式**用于管理*无效数据*或*丢失数据*。如果未启用严格模式，则 MySQL 将通过调整值和生成警告消息来管理插入和更新操作。我们可以通过启用`INSERT IGNORE`或`UPDATE IGNORE`选项在严格模式下执行相同的操作。让我们以一个键插入的例子来说明，其中键值超过了最大限制。如果启用了严格模式，MySQL 会产生错误并停止执行，而在相反的情况下，它会通过截断允许键值。同样，在`SELECT`语句的情况下，如果数据没有更改，MySQL 仍会产生错误，在严格模式下，如果存在无效值，则会生成警告消息。如果启用了`STRICT_ALL_TABLES`或`STRICT_TRANS_TABLES`选项，则严格模式生效。这两个选项在事务表的情况下行为类似，在非事务表的情况下行为不同。

+   **对于事务表**：如果启用了任一模式，则 MySQL 将在出现无效或缺少值的情况下产生错误并中止语句执行。

+   **对于非事务表**：当表是非事务性的时，MySQL 的行为将取决于以下因素：

+   `STRICT_ALL_TABLES`: 在这种情况下，将生成错误并停止执行。但仍然存在部分数据更新的可能性。为了避免这种错误情况，使用单行语句，如果在第一行插入/更新期间发生错误，将中止执行。

+   `STRICT_TRANS_TABLES`: 此选项提供了将无效值转换为最接近有效值的灵活性。在缺少值的情况下，MySQL 将数据类型的默认值插入到列中。在这里，MySQL 生成警告消息并继续执行。

严格模式影响对零的除法、零日期和日期中的零的处理，如前面的点中所述，使用`ERROR_FOR_DIVISION_BY_ZERO`、`NO_ZERO_DATE`和`NO_ZERO_IN_DATE`模式。

SQL 模式将应用于以下 SQL 语句：

```sql
ALTER TABLE
CREATE TABLE
CREATE TABLE ... SELECT
DELETE (both single table and multiple table)
INSERT
LOAD DATA
LOAD XML
SELECT SLEEP()
UPDATE (both single table and multiple table)
```

您可以访问：[`dev.mysql.com/doc/refman/8.0/en/sql-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html) 以获取 MySQL 中严格 SQL 模式相关错误的详细列表。

# IGNORE 关键字

MySQL 提供了一个可选的`IGNORE`关键字，用于语句执行。`IGNORE`关键字用于将错误降级为警告，并适用于多个语句。对于多行语句，`IGNORE`关键字允许您跳过特定行，而不是中止。以下语句支持`IGNORE`关键字：

+   `CREATE TABLE ... SELECT`: 单独的`CREATE`和`SELECT`语句不支持此关键字，但是当我们使用`SELECT`语句插入表时，具有唯一键值的行将被丢弃。

+   `DELETE`: 如果此语句执行`IGNORE`选项，MySQL 将避免执行期间发生的错误。

+   `INSERT`: 在行插入期间，此关键字将处理唯一键中的重复值和数据转换问题。MySQL 将在列中插入最接近的可能值并忽略错误。

+   `LOAD DATA`和`LOAD XML`: 在加载数据时，如果发现重复，该语句将丢弃它并继续插入剩余数据，如果定义了`IGNORE`关键字。

+   `UPDATE`: 在语句执行期间，如果唯一键发生重复键冲突，MySQL 将使用最接近的识别值更新列。

`IGNORE`关键字也适用于一些特定的错误，列在这里：[`dev.mysql.com/doc/refman/8.0/en/sql-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html)。

# IPv6 支持

MySQL 8 提供了对**IPv6**的支持，具有以下功能：

+   MySQL 服务器将接受来自具有 IPv6 连接性的客户端的 TCP/IP 连接

+   MySQL 8 帐户名称允许 IPv6 地址，这使得 DBA 可以为连接到服务器的客户端指定特权，使用 IPv6

+   IPv6 功能使字符串和内部 IPv6 地址格式之间的转换成为可能，并检查这些值是否表示有效的 IPv6 地址

# 服务器端帮助

MySQL 8 提供了`HELP`语句，以从 MySQL 参考手册中获取信息。为了管理这些信息，MySQL 使用系统数据库的几个表。为了初始化这些表，MySQL 提供了`fill_help_tables.sql`脚本。此脚本可在[`dev.mysql.com/doc/index-other.html`](https://dev.mysql.com/doc/index-other.html)下载并解压缩后，执行以下命令，以调用`HELP`函数：

```sql
mysql -u root mysql < fill_help_tables.sql
```

在安装过程中发生内容初始化。在升级的情况下，将执行上述命令。

# 服务器关闭过程

服务器关闭过程执行以下步骤：

1.  关闭过程已启动：有几种方法可以初始化关闭过程。执行`mysqladmin shutdown`命令，可以在任何平台上执行。还有一些特定于系统的方法来初始化关闭过程；例如，基于 Unix 的系统在接收到**SIGTERM**信号时将开始关闭。同样，基于 Windows 的系统将在服务管理器告知它们时开始关闭。

1.  如果需要，服务器将创建一个关闭线程：根据关闭初始化过程，服务器将决定是否创建新线程。如果客户端请求，将创建一个新线程。如果收到信号，则服务器可能会创建一个线程，或者自行处理。如果服务器尝试为关闭过程创建一个单独的线程，并且发生错误，则会在错误日志中产生以下消息：

```sql
 Error: Can't create thread to kill server
```

1.  服务器停止接受新连接：当关闭活动启动时，服务器将停止接受新的连接请求，使用网络接口的处理程序。服务器将使用 Windows 功能（如命名管道、TCP/IP 端口、Unix 套接字文件以及 Windows 上的共享内存）来监听新的连接请求。

1.  服务器终止当前活动：一旦关闭过程启动，服务器将开始与客户端断开连接。在正常情况下，连接线程将很快终止，但正在工作或处于进行中的活动阶段的线程将需要很长时间才能终止。因此，如果一个线程正在执行打开的事务，并且在执行过程中被回滚，那么用户可能只会得到部分更新的数据。另一方面，如果线程正在处理事务，服务器将等待直到事务完成。此外，用户可以通过执行`KILL QUERY`或`KILL CONNECTION`语句终止正在进行的事务。

1.  服务器关闭或关闭存储引擎：在此阶段，服务器刷新缓存并关闭所有打开的表。在这里，存储引擎执行所有必要的表操作。`InnoDB`刷新其缓冲池，将当前 LSN 写入表空间并终止其线程。`MyISAM`刷新挂起的索引。

1.  服务器退出：在此阶段，服务器将向管理进程提供以下值：

+   0 = 成功终止（未重新启动）

+   1 = 未成功终止（未重新启动）

+   2 = 未成功终止（已重新启动）

# 数据目录

数据目录是 MySQL 8 存储自身管理的所有信息的位置。数据目录的每个子目录代表一个数据库目录及其相关数据。所有 MySQL 安装都具有以下标准数据库：

+   `sys`目录：表示 sys 模式，其中包含用于性能模式信息解释的对象。

+   `performance schema`目录：此目录用于观察 MySQL 服务器在运行时的内部执行。

+   `mysql`目录：与 MySQL 系统数据库相关的目录，其中包含数据字典表和系统表。一旦 MySQL 服务器运行，它包含 MySQL 服务器所需的信息。

# 系统数据库

系统数据库主要包含存储对象元数据和其他操作目的的系统表的数据字典表。系统数据库包含许多系统表。我们将在接下来的部分中了解更多信息。

# 数据字典表

数据字典表包含有关数据对象的元数据。该目录中的表是不可见的，并且不会被一般的 SQL 查询（如`SELECT`、`SHOW TABLES`、`INFORMATION_SCHEMA.TABLES`等）读取。MySQL 主要使用`INFORMATION_SCHEMA`选项公开元数据。

# 授予系统表

这些表用于管理和提供用户、数据库和相关权限的授权信息。MySQL 8 使用授权表作为事务表，而不是非事务表（例如`MyISAM`），因此对事务的所有操作要么完成，要么失败；不会出现部分情况。

# 对象信息系统表

这些表包含与存储程序、组件和服务器端插件相关的信息。以下主要表用于存储信息：

+   **组件**: 作为服务器的注册表。MySQL 8 服务器在启动时加载此表列出的所有组件。

+   **Func**: 这个表包含与所有**用户定义函数**（**UDF**）相关的信息。MySQL 8 在服务器启动时加载此表中列出的所有 UDF。

+   **插件**: 包含与服务器端插件相关的信息。MySQL 8 服务器在启动时加载所有可用的插件。

# 日志系统表

这些表对记录和使用 csv 存储引擎很有用。例如，`general_log`和`slow_log`函数。

# 服务器端帮助系统表

这些表用于存储帮助信息。在这个类别中有以下表：

+   `help_category`: 提供关于帮助类别的信息

+   `help_keyword`: 提供与帮助主题相关的关键字

+   `help_relation`: 用于帮助关键字和主题之间的映射

+   `help_topic`: 帮助主题内容

# 时区系统表

这些表用于存储时区信息。在这个类别中有以下表：

+   `time_zone`: 提供时区 ID 以及它们是否使用闰秒

+   `time_zone_leap_second`: 当闰秒发生时会派上用场

+   `time_zone_name`: 用于时区 ID 和名称之间的映射

+   `time_zone_transition`和`time_zone_transition_type`: 时区描述

# 复制系统表

这些表对支持复制功能很有用。当配置为以下表中所述时，它有助于存储复制相关信息。在这个类别中有以下表：

+   `gtid_executed`: 用于创建存储 GTID 值的表

+   `ndb_binlog_index`: 为 MySQL 集群复制提供二进制日志信息

+   `slave_master_info`、`slave_relay_log_info`和`slave_worker_info`: 用于在从服务器上存储复制信息

# 优化器系统表

这些表对优化器很有用。在这个类别中有以下表：

+   `innodb_index_stats`和`innodb_table_stats`: 用于获取`InnoDB`持久优化器统计信息

+   `server_cost`: 包含了对一般服务器操作的优化器成本估算。

+   `engine_cost`: 包含特定存储引擎操作的估算

# 其他杂项系统表

不属于上述类别的表属于这个类别。在这个类别中有以下表：

+   `servers`: 被`FEDERATED`存储引擎使用

+   `innodb_dynamic_metadata`: 被`InnoDB`存储引擎用于存储快速变化的表元数据，如自增计数器值和索引树损坏标志

您可以在以下链接了解更多关于不同系统表的信息：[`dev.mysql.com/doc/refman/8.0/en/system-database.html`](https://dev.mysql.com/doc/refman/8.0/en/system-database.html)。

# 在单台机器上运行多个实例

可能会有一些情况需要在一台机器上安装多个实例。这可能是为了检查两个不同版本的性能，或者可能需要在不同的 MySQL 实例上管理两个单独的数据库。原因可能是任何，但是 MySQL 允许用户通过提供不同的配置值在同一台机器上执行多个实例。MySQL 8 允许用户使用命令行、选项文件或设置环境变量来配置参数。MySQL 8 用于此的主要资源是数据目录，对于两个实例，它必须是唯一的。我们可以使用`--datadir=dir_name`函数来定义相同的值。除了数据目录，我们还将为以下选项配置唯一的值：

+   `--port=port_num`

+   `--socket={file_name|pipe_name}`

+   `--shared-memory-base-name=name`

+   `--pid-file=file_name`

+   `--general_log_file=file_name`

+   `--log-bin[=file_name]`

+   ``--slow_query_log_file=file_name``

+   `--log-error[=file_name]`

+   `--tmpdir=dir_name`

# 设置多个数据目录

如上所述，每个 MySQL 实例必须有一个单独的数据目录。用户可以使用以下方法定义单独的目录：

+   **创建新的数据目录**：在这种方法中，我们必须遵循第二章中定义的相同过程，*安装和升级 MySQL*。对于 Microsoft Windows，当我们从 Zip 存档安装 MySQL 8 时，将其数据目录复制到要设置新实例的位置。在 MSI 软件包的情况下，连同数据目录一起，在安装目录下创建一个原始的`template`数据目录，命名为 data。安装完成后，复制数据目录以设置额外的实例。

+   **复制现有数据目录**：在这种方法中，我们将现有实例的数据目录复制到新实例的数据目录。要复制现有目录，请执行以下步骤：

1.  停止现有的 MySQL 实例。确保它被干净地关闭，以便磁盘中没有未决的更改。

1.  将数据目录复制到新位置。

1.  将现有实例使用的`my.cnf`或`my.ini`选项文件复制到新位置。

1.  根据新实例修改新选项。确保所有唯一的配置都正确完成。

1.  使用新的选项文件启动新实例。

# 在 Windows 上运行多个 MySQL 实例

用户可以通过使用命令行和传递值或通过窗口服务在单个 Windows 机器上运行多个 MySQL 实例。

+   **在 Windows 命令行上启动多个 MySQL 实例：**要使用命令行执行多个实例，我们可以在运行时指定选项，也可以在选项文件中设置它。选项文件是启动实例的更好选择，因为无需在启动时每次指定参数。要设置或配置选项文件，请按照[第二章](https://cdp.packtpub.com/mysql_8_administrator___s_guide/wp-admin/post.php?post=121&action=edit#post_26)中描述的相同步骤，*安装和升级 MySQL*。

+   **在 Windows 服务上启动多个 MySQL 实例：**要在 Windows 上启动多个实例作为服务，我们必须指定具有唯一名称的不同服务。如第二章中所述，*安装和升级 MySQL*，使用`–install`或`--install-manual`选项将 MySQL 定义为 Windows 服务。以下选项可用于将多个 MySQL 实例定义为 Windows 服务：

+   **方法 1**：为实例创建两个单独的选项文件，并在其中定义`mysqld`组。例如，使用函数`C:\my-opts1.cnf`。以下是相同代码供您参考：

```sql
 [mysqld]
 basedir = C:/mysql-5.5.5
 port = 3307
 enable-named-pipe
 socket = mypipe1
```

我们也可以使用`C:\my-opts2.cnf`函数来做同样的事情。以下代码描述了该过程：

```sql
 [mysqld]
 basedir = C:/mysql-8.0.1
 port = 3308
 enable-named-pipe
 socket = mypipe2
```

您可以使用以下命令安装 MySQL8 服务：

```sql
 C:\> C:\mysql-5.5.5\bin\mysqld --install mysqld1 --
                defaults-file=C:\my-opts1.cnf
 C:\> C:\mysql-8.0.1\bin\mysqld --install mysqld2 --
                defaults-file=C:\my-opts2.cnf
```

+   +   **方法 2**：为两个服务创建一个公共选项文件`C:\my.cnf`：

```sql
 # options for mysqld1 service
 [mysqld1]
 basedir = C:/mysql-5.5.5
 port = 3307
 enable-named-pipe
 socket = mypipe1

 # options for mysqld2 service
 [mysqld2]
 basedir = C:/mysql-8.0.1
 port = 3308
 enable-named-pipe
 socket = mypipe2
```

+   执行以下命令安装 MySQL 服务：

```sql
 C:\> C:\mysql-5.5.9\bin\mysqld --install mysqld1
 C:\> C:\mysql-8.0.4\bin\mysqld --install mysqld2
```

+   要启动 MySQL 服务，请执行以下命令：

```sql
 C:\> NET START mysqld1
 C:\> NET START mysqld2
```

# 组件和插件管理

MySQL 服务器支持基于组件的结构，以扩展服务器功能。MySQL 8 使用`INSTALL COMPONENT`和`UNINSTALL COMPONENT` SQL 语句在运行时加载和卸载组件。MySQL 8 将组件详细信息管理到`mysql.component`系统表中。因此，每次安装新组件时，MySQL 8 服务器都会执行以下任务：

+   将组件加载到服务器中以立即可用

+   将服务注册的组件加载到`mysql.component`系统表中。

当我们卸载任何组件时，MySQL 服务器将执行相同的步骤，但顺序相反。要查看可用的组件，请执行以下查询：

```sql
SELECT * FROM mysql.component;
```

# MySQL 8 服务器插件

MySQL 8 服务器具有插件 API，可用于创建服务器组件。使用 MySQL 8，您可以在运行时或启动时灵活安装插件。在接下来的主题中，我们将了解 MySQL 8 服务器插件的生命周期。

# 安装插件

插件的加载因其类型和特性而异。为了更清楚地了解这一点，让我们来看看以下内容：

+   **内置插件**：服务器知道内置插件并在启动时自动加载它们。用户可以通过任何激活状态来改变插件的状态，这将在下一节中讨论。

+   **在`mysql.plugin`系统表中注册的插件**：在启动时，MySQL 8 服务器将加载在`mysql.plugin`表中注册的所有插件。如果服务器使用`--skip-grant-tables`选项启动，则服务器将不加载那里列出的插件。

+   **使用命令行选项命名的插件**：MySQL 8 提供`--plugin-load`、`--plugin-load-add`和`--early-plugin-load`选项，用于在命令行加载插件。`--plugin-load`和`--plugin-load-add`选项在安装内置插件后在服务器启动时加载插件。但是，我们可以使用`--early-plugin-load`选项在初始化内置插件和存储引擎之前加载插件。

+   **使用`INSTALL PLUGIN`语句安装的插件**：这是一个永久的插件注册选项，它将在`mysql.plugin`表中注册插件信息。它还将加载插件库中的所有可用插件。

# 激活插件

要控制插件的状态（如激活或停用），MySQL 8 提供以下选项：

+   `--plugin_name=OFF`：禁用指定的插件。一些内置插件，如`asmysql_native_password`插件，不受此命令影响。

+   `--plugin_name[=ON]`：此命令启用指定的插件。如果在启动时插件初始化失败，MySQL 8 将以禁用插件的状态启动。

+   `--plugin_name=FORCE`：这与上述命令相同，只是服务器不会启动。这意味着如果在启动时提到了插件，它会强制服务器与插件一起启动。

+   `--plugin_name=FORCE_PLUS_PERMANENT`：与`FORCE`选项相同，但另外防止插件在运行时被卸载。

# 卸载插件

MySQL 8 使用`UNINSTALL PLUGIN`语句卸载插件，而不考虑它是在运行时还是在启动时安装的。但是，此语句不允许我们卸载内置插件和通过`--plugin_name=FORCE_PLUS_PERMANENT`选项安装的插件。此语句只是卸载插件并将其从`mysql.plugin`表中删除，因此需要`mysql.plugin`表上的额外*delete*权限。

# 获取已安装插件的信息

有多种方法可以获取有关已安装插件的信息。以下是其中一些，供您参考：

+   `INFORMATION_SCHEMA.PLUGINS`表包含插件的详细信息，如`PLUGIN_NAME`、`PLUGIN_VERSION`、`PLUGIN_STATUS`、`PLUGIN_TYPE`、`PLUGIN_LIBRARY`等等。该表的每一行都代表有关插件的信息：

```sql
 SELECT * FROM information_schema.PLUGINS;
```

+   `SHOW PLUGINS`语句显示了每个单独插件的名称、状态、类型、库和许可证详情。如果库的值为`NULL`，则表示它是一个内置插件，因此无法卸载。

```sql
 SHOW PLUGINS;
```

+   `mysql.plugin`表包含了所有通过`INSTALL PLUGIN`函数注册的插件的详细信息。

# 角色和权限

简而言之，*角色*是一组权限。在 MySQL 8 中创建角色，您必须具有全局的`CREATE ROLE`或`CREATE USER`权限。MySQL 8 提供了各种权限，可附加到角色和用户上。有关可用权限的更多详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/privileges-provided.html`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html)。

现在，让我们举个例子来理解角色创建和权限分配的作用。假设我们已经在当前数据库中创建了一个`hr_employee`表，并且我们想要将这个表的访问权限赋予`hrdepartment`角色。这个困境可以通过使用以下代码来解决：

```sql
CREATE ROLE hrdepartment;
grant all on hr_employee to hrdepartment;
```

上述代码将帮助我们创建`hrdepartment`角色并授予它所有必要的访问权限。这个主题将在第十一章中详细介绍，*安全*。

# 缓存技术

缓存是一种用于提高性能的机制。MySQL 使用多种策略在缓冲区中缓存信息。MySQL 8 在存储引擎级别使用缓存来处理其操作。它还在准备好的语句和存储程序中应用缓存以提高性能。MySQL 8 引入了各种系统级变量来管理缓存，例如`binlog_stmt_cache_size`、`daemon_memcached_enable_binlog`、`daemon_memcached_w_batch_size`、`host_cache_size`等等。我们将在第十二章中详细介绍缓存，*优化 MySQL 8*。

# 全球化

全球化是一个功能，为应用程序提供多语言支持，比如启用本地语言的使用。用我们自己的母语理解消息要比其他语言容易得多，对吧？为了实现这一点，全球化就出现了。使用全球化，用户可以将数据存储、检索和更新为多种语言。在全球化中有一些参数需要考虑。我们将在接下来的章节中详细讨论它们。

# 字符集

在详细讨论字符集之前，需要了解字符集实际上是什么，以及它的相关术语，对吧？让我们从术语本身开始；字符集是一组符号和编码。与字符集相关的另一个重要术语是**校对规则**，用于比较字符。让我们举一个简单的例子来理解字符集和校对规则。考虑两个字母，*P*和*Q*，并为每个分配一个数字，使得*P=1*和*Q=2*。现在，假设*P*是一个符号，1 是它的编码。在这里，这两个字母及它们的编码的组合被称为字符集。现在假设我们想要比较这些值；最简单的方法是参考编码值。由于 1 小于 2，我们可以说*P*小于*Q*，这就是校对规则。这是一个简单的例子来理解字符集和校对规则，但在现实生活中，我们有许多字符，包括特殊字符，同样的校对规则也有许多规则。

# 字符集支持

MySQL 8 支持多种字符集，具有各种排序规则。字符集可以在列、表、数据库或服务器级别定义。我们可以在`InnoDB`、`MyISAM`和`Memory`存储引擎中使用字符集。要检查 MySQL 8 的所有可用字符集，请执行以下命令：

```sql
mysql> show character set;
+----------+---------------------------------+---------------------+--------+
| Charset | Description | Default collation | Maxlen |
+----------+---------------------------------+---------------------+--------+
| armscii8 | ARMSCII-8 Armenian | armscii8_general_ci | 1 |
| ascii | US ASCII | ascii_general_ci | 1 |
| big5 | Big5 Traditional Chinese | big5_chinese_ci | 2 | .........
.........
+----------+---------------------------------+---------------------+--------+
41 rows in set (0.01 sec)
```

同样，要查看字符的排序规则，请执行以下命令：

```sql
mysql> SHOW COLLATION WHERE Charset = 'ascii';
+------------------+---------+----+---------+----------+---------+---------------+
| Collation | Charset | Id | Default | Compiled | Sortlen | Pad_attribute |
+------------------+---------+----+---------+----------+---------+---------------+
| ascii_bin | ascii | 65 | | Yes | 1 | PAD SPACE |
| ascii_general_ci | ascii | 11 | Yes | Yes | 1 | PAD SPACE |
+------------------+---------+----+---------+----------+---------+---------------+
2 rows in set (0.00 sec)
```

排序规则将具有以下三个特征：

+   两个不同的字符集合不能具有相同的排序规则。

+   每个字符集都有一个默认的排序规则。如上所示，`show character set`命令显示了字符集的默认排序规则。

+   排序规则遵循预定义的命名约定，稍后将对其进行解释。

+   字符集合：**repertoire**是数据集中的字符集合。任何字符串表达式都将具有 repertoire 属性，并且将属于以下值之一：

+   ASCII：包含 Unicode 范围 U+0000 到 U+007F 的字符的表达式。

+   UNICODE：包含 Unicode 范围 U+0000 到 U+10FFFF 的字符的表达式。这包括**基本多语言平面**（**BMP**）范围（U+0000 到 U+FFFF）和 BMP 范围之外的补充字符（U+01000 到 U+10FFFF）。

从这两个值的范围中，我们可以确定 ASCII 是 UNICODE 范围的子集，我们可以安全地将 ASCII 值转换为 UNICODE 值而不会丢失数据。Repertoire 主要用于将表达式从一个字符集转换为另一个字符集。在某些转换情况下，MySQL 8 会抛出类似“illegal mix of collations”的错误；为了处理这些情况，需要 repertoire。要了解其用法，请考虑以下示例：

```sql
CREATE TABLE employee (
 firstname CHAR(10) CHARACTER SET latin1,
 lastname CHAR(10) CHARACTER SET ascii
);

INSERT INTO employee VALUES ('Mona',' Singh');

select concat(firstname,lastname) from employee;
+----------------------------+
| concat(firstname,lastname) |
+----------------------------+
| Mona Singh |
+----------------------------+
1 row in set (0.00 sec)
```

+   **用于元数据的 UTF-8**：元数据是关于数据的数据。在数据库方面，我们可以说描述数据库对象的任何内容都称为**元数据**。例如：列名，用户名等。MySQL 遵循以下两条元数据规则：

+   包括所有语言中的所有字符以用于元数据；这使用户可以使用自己的语言作为列名和表名。

+   为所有元数据管理一个共同的字符集合。否则，`INFORMATION_SCHEMA`中的表的`SHOW`和`SELECT`语句将无法正常工作。

为了遵循上述规则，MySQL 8 将元数据存储为 Unicode 格式。请注意，MySQL 函数（如`USER()`、`CURRENT_USER()`、`SESSION_USER()`、`SYSTEM_USER()`、`DATABASE()`和`VERSION()`）默认使用 UTF-8 字符集。MySQL 8 服务器已定义`character_set_system`来指定元数据的字符集。确保在 Unicode 中存储元数据并不意味着列标题和`DESCRIBE`函数将以元数据字符集的形式返回值。它将根据`character_set_results`系统变量工作。

# 添加字符集

本节介绍如何在 MySQL 8 中添加字符集。此方法可能因字符类型而异，可能简单或复杂。在 MySQL 8 中添加字符集需要以下四个步骤：

1.  将`<charset>`元素添加到`sql/share/charsets/Index.xml`文件中的`MYSET`。有关语法，请参考已定义的其他字符集文件。

1.  在此步骤中，简单字符集和复杂字符集的处理方式不同。对于简单字符集，创建一个配置文件`MYSET.xml`，描述字符集属性，放在`sql/share/charsets`目录中。对于复杂字符集，需要 C 源文件。例如，在 strings 目录中创建`ctype-MYSET.c`类型。对于每个`<collation>`元素，提供`ctype-MYSET.c`文件。

1.  修改配置信息：

1.  编辑`mysys/charset-def.c`，并*注册*新字符集的排序规则。将这些行添加到**declaration**部分：

```sql
 #ifdef HAVE_CHARSET_MYSET
 extern CHARSET_INFO my_charset_MYSET_general_ci;
 extern CHARSET_INFO my_charset_MYSET_bin;
 #endif
```

将这些行添加到**registration**部分：

```sql
 #ifdef HAVE_CHARSET_MYSET
 add_compiled_collation(&my_charset_MYSET_general_ci);
 add_compiled_collation(&my_charset_MYSET_bin);
 #endif
```

1.  1.  如果字符集使用`ctype-MYSET.c`，请编辑`strings/CMakeLists.txt`并将`ctype-MYSET.c`添加到`STRINGS_SOURCES`变量的定义中。

1.  编辑`cmake/character_sets.cmake`进行以下更改：

+   按字母顺序将`MYSET`添加到`CHARSETS_AVAILABLE`的值中。

+   按字母顺序将`MYSET`添加到`CHARSETS_COMPLEX`的值中。即使对于简单的字符集，也需要这样做，否则`CMake`将无法识别`DDEFAULT_CHARSET=MYSET`。

1.  重新配置、重新编译和测试。

# 配置字符集

MySQL 8 提供了`--character-set-server`和`--collation-server`选项来配置字符集。默认字符集已从`latin1`更改为`UTF8`。`UTF8`是主导字符集，尽管在 MySQL 的先前版本中它不是默认字符集。随着这些全球性变化的接受，字符集和排序规则

现在基于`UTF8`；一个常见的原因是因为`UTF8`支持大约 21 种不同的语言，这使得系统提供多语言支持。在配置排序规则之前，请参考[`dev.mysql.com/doc/refman/8.0/en/show-collation.html`](https://dev.mysql.com/doc/refman/8.0/en/show-collation.html)上提供的排序规则列表。

# 语言选择

MySQL 8 默认使用英语语言的错误消息，但允许用户选择其他几种语言。例如，俄语、西班牙语、瑞典语等。MySQL 8 使用`lc_messages_dir`和`lc_messages`两个系统变量来管理错误消息的语言，并具有以下属性：

+   `lc_messages_dir`：这是一个系统变量，在服务器启动时设置。它是全局变量，因此通常由所有客户端在运行时使用。

+   `lc_messages`：此变量在全局和会话级别上都被使用。允许个别用户使用不同的语言来显示错误消息。例如，如果在服务器启动时设置了`en_US`，但如果要使用法语，则执行以下命令：

```sql
 SET lc_messages = 'fr_FR';
```

MySQL 8 服务器遵循以下三条错误消息文件规则：

+   MySQL 8 将在由两个系统变量`lc_messages_dir`和`lc_messages`构成的位置找到文件。例如，如果使用以下命令启动 MySQL 8，则`mysqld`将将区域设置`nl_NL`映射到荷兰语，并在`/usr/share/mysql/dutch`目录中搜索错误文件。MySQL 8 将所有语言文件存储在`MySQL8 Base Directory/share/mysql/LANGUAGE`目录中。默认情况下，语言文件位于 MySQL 基目录下的`share/mysql/LANGUAGE`目录中。

```sql
 mysqld --lc_messages_dir=/usr/share/mysql --lc_messages=nl_NL
```

+   如果目录下不存在消息文件，则 MySQL 8 将忽略`lc_messages`变量的值，并将`lc_messages_dir`变量的值视为要查找的位置。

+   如果 MySQL 8 服务器找不到消息文件，则它会在错误日志文件中显示一条消息，并对消息使用英语。

# MySQL8 的时区设置

MySQL 8 服务器以三种不同的方式管理时区：

+   系统时区：这由`system_time_zone`系统变量管理，可以通过`--timezone=timezone_name`或在执行 mysqld 之前使用`TZ`环境变量来设置。

+   服务器当前时区：这由`time_zone`系统变量管理。`time_zone`变量的默认值是`SYSTEM`，这意味着服务器时区与系统时区相同。MySQL 8 允许用户在启动时通过在选项文件中指定`default-time-zone='*timezone*'`来设置`time_zone`全局变量的值，并在运行时使用以下命令：

```sql
 mysql> SET GLOBAL time_zone = timezone;
```

+   预连接时区：这由`time_zone`变量管理，特定于连接到 MySQL 8 服务器的客户端。此变量从全局`time_zone`变量获取其初始值，但 MySQL 8 允许用户通过执行以下命令在运行时更改它：

```sql
 mysql> SET time_zone = timezone;
```

此会话变量影响区域特定值的显示和存储。例如，由`NOW()`和`CURTIME()`函数返回的值。另一方面，此变量不会影响以 UTC 格式显示和存储的值，例如`UTC_TIMESTAMP()`函数。

# 区域设置支持

MySQL 8 使用`lc_time_names`系统变量来控制语言，这将影响显示的日期、月份名称和缩写。`DATE_FORMAT()`、`DAYNAME()`和`MONTHNAME()`函数的输出取决于`lc_time_names`变量的值。首先浮现在脑海中的问题是，这些区域设置是在哪里定义的，我们如何获取它们？不用担心，参考[`www.iana.org/assignments/language-subtag-registry`](http://www.iana.org/assignments/language-subtag-registry)。所有区域设置都由**互联网编号分配机构**（**IANA**）以语言和地区缩写定义。默认情况下，MySQL 8 将`en_US`设置为系统变量的区域设置。用户可以在服务器启动时设置值，或者如果具有`SYSTEM_VARIABLES_ADMIN`或`SUPER`特权，则可以设置`GLOBAL`。MySQL 8 允许用户检查和设置其连接的区域设置。执行以下命令在您的工作站上检查区域设置：

```sql
mysql> SET NAMES 'utf8';
Query OK, 0 rows affected (0.09 sec)

mysql> SELECT @@lc_time_names;
+-----------------+
| @@lc_time_names |
+-----------------+
| en_US |
+-----------------+
1 row in set (0.00 sec)

mysql> SELECT DAYNAME('2010-01-01'), MONTHNAME('2010-01-01');
+-----------------------+-------------------------+
| DAYNAME('2010-01-01') | MONTHNAME('2010-01-01') |
+-----------------------+-------------------------+
| Friday | January |
+-----------------------+-------------------------+
1 row in set (0.00 sec)

mysql> SELECT DATE_FORMAT('2010-01-01','%W %a %M %b');
+-----------------------------------------+
| DATE_FORMAT('2010-01-01','%W %a %M %b') |
+-----------------------------------------+
| Friday Fri January Jan |
+-----------------------------------------+
1 row in set (0.00 sec)

mysql> SET lc_time_names = 'nl_NL';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@lc_time_names;
+-----------------+
| @@lc_time_names |
+-----------------+
| nl_NL |
+-----------------+
1 row in set (0.00 sec)

mysql> SELECT DAYNAME('2010-01-01'), MONTHNAME('2010-01-01');
+-----------------------+-------------------------+
| DAYNAME('2010-01-01') | MONTHNAME('2010-01-01') |
+-----------------------+-------------------------+
| vrijdag | januari |
+-----------------------+-------------------------+
1 row in set (0.00 sec)

mysql> SELECT DATE_FORMAT('2010-01-01','%W %a %M %b');
+-----------------------------------------+
| DATE_FORMAT('2010-01-01','%W %a %M %b') |
+-----------------------------------------+
| vrijdag vr januari jan |
+-----------------------------------------+
1 row in set (0.00 sec)</strong>
```

# MySQL 8 服务器日志

MySQL 8 服务器提供了以下不同类型的日志，使用户能够跟踪服务器在各种情况下的活动：

| **日志类型** | **写入日志的信息** |
| --- | --- |
| 错误日志 | 启动、运行或停止`mysqld`时遇到的问题 |
| 通用查询日志 | 已建立的客户端连接和从客户端接收到的语句 |
| 二进制日志 | 更改数据的语句（也用于复制） |
| 中继日志 | 从复制主服务器接收到的数据更改 |
| 慢查询日志 | 执行时间超过`long_query_time`秒的查询 |
| DDL 日志（元数据日志） | DDL 语句执行的元数据操作 |

您可以在[`dev.mysql.com/doc/refman/8.0/en/server-logs.html`](https://dev.mysql.com/doc/refman/8.0/en/server-logs.html)了解有关不同类型日志的更多信息。

MySQL 8 不会生成 MySQL 8 中的日志，除非在 Windows 中的错误日志中启用。默认情况下，MySQL 8 将所有日志存储在数据目录下的文件中。当我们谈论文件时，会有很多问题涌入我们的脑海，对吧？例如；文件的大小是多少？会生成多少个文件？我们如何刷新日志文件？MySQL 8 提供了各种配置来管理日志文件；我们将在本章的后面部分看到所有这些配置。另一个重要的问题是我们在哪里存储日志？在表中还是在文件中？以下是一些描述表与文件相比的优点的要点：

+   如果日志存储在表中，则其内容可通过 SQL 语句访问。这意味着用户可以执行带有所需条件的选择查询，以获得特定的输出。

+   任何远程用户都可以连接到数据库并获取日志的详细信息。

+   日志条目由标准格式管理。您可以使用以下命令检查日志表的结构：

通用日志的代码：

```sql
SHOW CREATE TABLE mysql.general_log;
```

慢查询日志的代码：

```sql
SHOW CREATE TABLE mysql.slow_log;
```

# 错误日志

此日志用于记录从 MySQL 8 启动到结束期间发生的错误、警告和注释等诊断消息。MySQL 8 为用户提供了各种配置和组件，以便根据其要求生成日志文件。当我们开始写入文件时，会有一些基本问题涌入脑海；我们要写什么？我们如何写？我们要写到哪里？让我们从第一个问题开始。MySQL 8 使用`log_error_verbosity`系统变量，并分配以下过滤选项来决定应将哪种类型的消息写入错误日志文件：

+   ``仅错误``

+   `错误和警告`

+   `错误，警告和注释`

要在目的地位置写入 MySQL 使用以下格式，其中时间戳取决于`log_timestamps`系统变量：

```sql
timestamp thread_id [severity] message 
```

写入日志文件后，首先要考虑的问题是，我们如何刷新这些日志？为此，MySQL 8 提供了三种方法：`FLUSH ERROR LOGS`，`FLUSH LOGS`或`mysqladmin flush-logs`。这些命令将关闭并重新打开正在写入的日志文件。当我们谈论如何写入以及在哪里写入时，有很多事情要理解。

# 组件配置

MySQL 8 使用`log_error_services`系统变量来控制错误日志组件。它允许用户通过分号分隔的方式定义多个组件以进行执行。在这里，组件将按照定义的顺序执行。用户可以在以下约束条件下更改此变量的值：

+   安装组件：要启用任何日志组件，我们必须首先使用此命令安装它，然后通过在`log_error_services`系统变量中列出该组件来使用该组件。按照以下命令添加`log_sink_syseventlog`组件：

```sql
 INSTALL COMPONENT 'file://component_log_sink_syseventlog';
 SET GLOBAL log_error_services = 'log_filter_internal; 
          log_sink_syseventlog';
```

执行安装命令后，MySQL 8 将注册该组件到`mysql.component`系统表中，以便在每次启动时加载。

+   卸载组件：要禁用任何日志组件，首先从`log_error_services`系统变量列表中删除它，然后使用此命令卸载它。执行以下命令以卸载组件：

```sql
 UNINSTALL COMPONENT 'file://component_log_sink_syseventlog';
```

要在每次启动时启用错误日志组件，请在`my.cnf`文件中定义它，或使用`SET_PERSIST`。当我们在`my.cnf`中定义它时，它将从下一次重新启动开始生效，而`SET_PERSIST`将立即生效。使用以下命令进行`SET_PERSIST`：

```sql
 SET PERSIST log_error_services = 'log_filter_internal; 
          log_sink_internal; 
          log_sink_json'; 
```

MySQL 8 还允许用户将错误日志写入系统日志：对于 Microsoft，请考虑事件日志，对于基于 Unix 的系统，请考虑 syslog。要将错误日志记录到系统`logfibf`中，配置`log_filter_internal`和系统日志写入器`log_sink_syseventlog`组件，并按照上述说明执行相同的指令。另一种方法是将 JSON 字符串写入日志文件配置`log_sink_json`组件。关于 JSON 写入器的一个有趣的点是，它将通过添加 NN（两位数）来管理文件命名约定。例如，将文件名视为`file_name.00.json`，`file_name.01.json`等。

# 默认错误日志目的地配置

错误日志可以写入日志文件或控制台。本节描述了如何在不同环境中配置错误日志的目的地。

# Windows 上的默认错误日志目的地

+   `--console`：如果给出此选项，则控制台将被视为默认目的地。在定义了两者的情况下，`--console`优先于`--log-error`。如果默认位置是控制台，那么 MySQL 8 服务器将`log_error`变量的值设置为`stderror`。

+   `--log-error`：如果未给出此选项，或者给出但未命名文件，则默认文件名为`host_name.err`，并且该文件将在数据目录中创建，除非指定了`--pid-fileoption`。如果在`–pid-file`选项中指定了文件名，则命名约定将是数据目录中带有`.err`后缀的**PID**文件基本名称。

# Unix 和类 Unix 系统上的默认错误日志目的地

在 Unix 系统中，Microsoft Windows 中提到的所有上述情况将由`–log_error`选项管理。

+   `--log-error`：如果未给出此选项，则默认目的地是控制台。如果未给出文件名，则与 Windows 一样，它将在数据目录中创建一个名为`host_name.err`的文件。用户可以在`mysqld`或`mysqld_safe`部分的选项文件中指定`–log-error`。

# 一般查询日志

一般查询日志是一个通用日志，用于记录`mysqld`执行的所有操作。在此日志中，文件语句按接收顺序编写，但执行顺序可能与接收顺序不同。它从客户端连接开始记录，并持续到断开连接。除了 SQL 命令，它还记录了`connection_type`，即协议客户端连接的方式，例如 TCP/IP、SSL、Socket 等。由于它记录了`mysqld`执行的大部分操作，当我们想要查找客户端发生了什么错误时，它非常有用。

默认情况下，此日志被禁用。我们可以使用**`--general_log[={0|1}]`**命令来启用它。当我们不指定任何参数或将 1 定义为参数时，表示启用一般查询日志，而 0 表示禁用日志。此外，我们可以使用`--general_log_file=file_name`命令指定日志文件名。如果命令未指定文件名，则 MySQL 8 将考虑默认名称为`host_name.log`。设置日志文件名对日志记录没有影响，如果日志目的地值不包含`FILE`。服务器重新启动和日志刷新不会导致生成新的一般查询日志文件；您必须使用`rename`（对于 Microsoft Windows）或`mv`（对于 Linux）命令来创建新文件。MySQL 8 提供了第二种在运行时重命名文件的方法，方法是使用以下命令禁用日志：

```sql
SET GLOBAL general_log = 'OFF';
```

禁用日志后，使用`ON`选项重命名日志文件并再次启用日志。同样，要在特定连接的运行时启用或禁用日志，请使用会话`sql_log_off`变量和`ON`或`OFF`选项。另一个选项是与一般日志文件对齐的，即`--log-output`。通过使用此选项，我们可以指定日志输出的目的地；这并不意味着日志已启用。

此命令提供了以下三种不同的选项：

+   `TABLE`：记录到表中

+   `FILE`：记录到文件中

+   `NONE`：不记录到表或文件中。如果存在`NONE`，则优先于任何其他指定符。

如果省略了`--log-output`选项，则默认值为文件。

# 二进制日志

二进制日志是一个文件，其中包含描述数据库事件的所有事件，例如表创建、数据更新和表中的删除。它不用于`SELECT`和`SHOW`语句，因为它不会更新任何数据。二进制日志写入会稍微降低数据库操作的性能，但它使用户能够使用复制设置和操作还原。二进制日志的主要目的是：

1.  **用于主从架构的复制**：基于二进制文件的复制，主服务器执行插入和更新操作，这些操作在二进制日志文件中反映出来。现在，从节点被配置为读取这些二进制文件，并且相同的事件在从服务器的二进制文件中执行，以便将数据复制到从服务器上。

1.  **数据恢复操作**：一旦备份被还原到数据库中，二进制日志的事件将被记录，并以重新执行的形式执行这些事件，从而使数据库从备份点更新到最新状态。

二进制日志默认启用，这表明 log_bin 系统变量设置为 ON。要禁用此日志，请在启动时使用`--skip-log-bin`或`--disable-log-bin`选项。要删除所有二进制日志文件，请使用 RESET MASTER 语句，或者使用`PURGE BINARY LOGS`删除其中的一部分。MySQL 8 服务器使用以下三种日志格式将信息记录到二进制日志文件中：

1.  **基于语句的日志记录**：通过使用`--binlog-format=STATEMENT`命令启动服务器来使用此格式。这主要是 SQL 语句的传播。

1.  **基于行的日志记录**：在服务器启动时使用`--binlog-format=ROW`启用基于行的日志记录。此格式指示行受到的影响。这是默认选项。

1.  **混合日志记录**：使用`--binlog-format=MIXED`选项启动 MySQL 8 以启用混合日志记录。在此模式下，默认情况下可用语句基础日志记录，并且在某些情况下 MySQL 8 将自动切换到基于行的日志记录。

MySQL 8 允许用户在全局和会话范围内在运行时更改格式。全局格式适用于所有客户端，而会话格式适用于单个客户端。以下分别设置全局和会话范围的格式：

```sql
mysql> SET GLOBAL binlog_format = 'STATEMENT';
mysql> SET SESSION binlog_format = 'STATEMENT';
```

有两种特殊情况下我们无法更改格式：

+   在存储过程或函数中

+   在设置为行格式并且临时表处于打开状态的情况下

MySQL 8 具有`--binlog-row-event-max-size`变量，用于以字节为单位控制二进制日志文件的大小。将此变量的值分配为 256 的倍数；此选项的默认值为 8192。MySQL 8 的各个存储引擎都有其自己的日志记录能力。如果存储引擎支持基于行的日志记录，则称为**行日志**能力，如果存储引擎支持基于语句的日志记录，则称为**语句日志**能力。有关存储引擎日志记录能力的更多信息，请参考下表。

| 存储引擎 | 支持行日志记录 | 支持语句日志记录 |
| --- | --- | --- |
| `ARCHIVE` | 是 | 是 |
| `BLACKHOLE` | 是 | 是 |
| `CSV` | 是 | 是 |
| `EXAMPLE` | 是 | 否 |
| `FEDERATED` | 是 | 是 |
| `HEAP` | 是 | 是 |
| `InnoDB` | 是 | 当事务隔离级别为`REPEATABLE`、`READ`或`SERIALIZABLE`时为是；否则为否。 |
| `MyISAM` | 是 | 是 |
| `MERGE` | 是 | 是 |
| `NDB` | 是 | 否 |

如本节所述，二进制日志将根据语句类型（安全、不安全或二进制注入）、日志格式（`ROW`、`STATEMENT`或`MIXED`）以及存储引擎的日志功能（行可用、语句可用、两者都可用或两者都不可用）进行工作。要了解二进制日志记录的所有可能情况，请参考此链接中给出的表格：[`dev.mysql.com/doc/refman/8.0/en/binary-log-mixed.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-log-mixed.html)。

# 慢查询日志

慢查询日志用于记录执行时间长的 SQL 语句。MySQL 8 为慢查询的时间配置定义了以下两个系统变量：

+   `long_query_time`：用于定义查询执行的理想时间。如果 SQL 语句的执行时间超过此时间，则被视为慢查询，并将语句记录到日志文件中。默认值为 10 秒。

+   `min_examined_row_limit`：执行每个查询所需的最短时间。默认值为 0 秒。

MySQL 8 不会将获取锁的初始时间计入执行时间，并且在所有锁释放并完成查询执行后将慢查询日志返回到文件中。当启动 MySQL 8 时，默认情况下禁用慢查询日志；要启动此日志，请使用`slow_query_log[={0|1}]`命令，其中`0`表示禁用慢查询日志，1 或无参数用于启用它。要记录不使用索引的管理语句和查询，请使用**`log_slow_admin_statements`**和**`log_queries_not_using_indexes`**变量。这里，管理语句包括`ALTER TABLE`、`ANALYZE TABLE`、`CHECK TABLE`、`CREATE INDEX`、`DROP INDEX`、`OPTIMIZE TABLE`和`REPAIR TABLE`。MySQL 8 允许用户使用`--slow_query_log_file=file_name`命令指定日志文件的名称。如果未指定文件名，则 MySQL 8 将在数据目录中使用`host_name-slow.log`命名约定创建文件。要将最少的信息写入此日志文件，请使用`--log-short-format`选项。

上述所有描述的参数由 MySQL 8 按以下顺序控制：

1.  查询必须不是管理语句，或者`log_slow_admin_statements`必须已启用

1.  查询必须至少花费`long_query_time`秒，或者启用了`log_queries_not_using_indexes`，并且查询必须没有使用索引进行行查找

1.  查询必须至少检查`min_examined_row_limit`行

1.  查询不应根据`log_throttle_queries_not_using_indexes`设置被抑制

`--log-output`选项也适用于此日志文件，并具有与通用日志相同的实现和效果。

# DDL 日志

正如名称所示，此日志文件用于记录所有与 DDL 语句执行相关的详细信息。MySQL 8 使用此日志文件来从在元数据操作执行期间发生的崩溃中恢复。让我们举一个例子来理解不同的情况：

+   **删除表 t1，t2**：我们必须确保 t1 和 t2 表都被删除

当我们执行任何 DDL 语句时，这些操作的记录将被写入 MySQL 8 数据目录下的`ddl_log.log`文件中。该文件是一个二进制文件，不是人类可读的格式。用户不允许更新此日志文件的内容。在 MySQL 服务器的正常执行中不需要记录元数据语句；只有在需要时才启用它。

# 服务器日志维护

为了维护日志文件，我们必须定期清理以管理磁盘空间。对于基于 RPM 的 Linux 系统，`mysql-log-rotate`脚本会自动提供。对于其他系统，没有这样的脚本，因此我们必须自己安装一个简短的脚本来管理日志文件。MySQL 8 提供了`expire_logs_days`系统变量，用于管理二进制日志文件。使用此变量，二进制日志文件将在指定期限后自动删除。

此变量的默认值为 30 天；您可以通过配置更改其值。二进制日志文件在服务器启动时或日志刷新时删除。在复制的情况下，您还可以使用`binlog_expire_logs_seconds`系统变量来管理主服务器和从服务器的日志。日志刷新执行以下任务：

+   如果启用了一般查询日志或慢查询日志到日志文件，服务器将关闭并重新打开查询日志文件

+   如果启用了二进制日志记录，服务器将关闭当前的二进制日志文件，并打开下一个序列号的新日志文件

+   如果服务器是使用`--log-error`选项启动的，以导致错误日志被写入文件，服务器将关闭并重新打开日志文件

在生成新的日志文件之前备份或重命名旧的日志文件，可以在 Unix 系统中使用`mv`（移动）命令，在 Windows 中使用`rename`函数。对于一般查询和慢查询日志文件，可以通过使用以下命令禁用日志来重命名文件：

```sql
SET GLOBAL general_log = 'OFF';
```

重命名日志文件后，使用以下命令启用日志：

```sql
SET GLOBAL general_log = 'ON';
```

# 总结

这对于任何 MySQL 8 用户来说都是一个有趣的章节，不是吗？在本章中，我们了解了 MySQL 8 如何管理不同的日志文件，以及在什么时候使用哪个日志文件。同时，我们还涵盖了许多管理功能，例如全球化、系统数据数据库和组件和插件配置，并解释了如何在单台机器上运行多个实例。本章的后半部分涵盖了日志维护。

接下来，我们将为您提供有关存储引擎的信息，例如不同类型的存储引擎是什么，哪种适合您的应用程序，以及如何为 MySQL 8 创建自定义存储引擎。
