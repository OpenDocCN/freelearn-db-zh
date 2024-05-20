# 第六章：MySQL 8 存储引擎

在上一章中，我们学习了如何设置新系统、数据字典和系统数据库。提供了有关缓存技术、全球化、不同类型组件和插件配置以及对管理非常重要的几种类型日志文件的详细信息。

本章详细介绍了 MySQL 8 存储引擎。它详细解释了`InnoDB`存储引擎及其特性，并提供了有关自定义存储引擎创建以及如何使其可插拔以便安装在 MySQL 8 中的实用指南。本章将涵盖以下主题：

+   存储引擎概述

+   多种类型的存储引擎

+   `InnoDB`存储引擎

+   创建自定义存储引擎

# 存储引擎概述

存储引擎是 MySQL 组件，用于处理不同类型表格中使用的 SQL 操作。MySQL 存储引擎旨在管理不同类型环境中的不同类型任务。了解并选择最适合系统或应用需求的存储引擎非常重要。在接下来的章节中，我们将详细了解存储引擎的类型、默认存储引擎以及自定义存储引擎的创建。

让我们来看看为什么存储引擎是数据库中非常重要的组件，包括 MySQL 8。存储引擎与数据库引擎一起在不同环境中执行各种类型的任务。它们以语句的形式在数据库中对数据执行创建、读取、更新和删除操作。当您在创建表语句中提供`ENGINE`参数时，看起来很简单，但对于每个通过 SQL 语句发送的请求，需要对数据执行大量操作的配置。它远不止是持久化数据 - 引擎还负责存储限制、事务、锁定粒度/级别、多版本并发控制、地理空间数据类型、地理空间索引、B 树索引、T 树索引、`Hash`索引、全文搜索索引、聚集索引、数据缓存、索引缓存、压缩数据、加密数据、集群数据库、复制、外键、备份、查询缓存以及更新数据字典的统计信息。

# MySQL 存储引擎架构

MySQL 存储引擎的可插拔架构允许数据库专业人员为任何特定应用程序选择任何存储引擎。MySQL 存储引擎架构提供了一个简单的应用模型和 API，具有一致性，可以隔离数据库管理员和应用程序员免受存储级别的所有底层实现细节的影响。因此，应用程序始终在不同存储引擎的不同功能之上运行。它提供了标准的管理和支持服务，适用于所有底层存储引擎。

存储引擎在物理服务器级别上对持久化数据执行活动。这种模块化和高效的架构为任何特定应用程序的特定需求提供了解决方案，例如事务处理、高可用性情况或数据仓库，并且同时具有独立于底层存储引擎的接口和服务的优势。

数据库管理员和应用程序员通过连接器 API 和服务与 MySQL 数据库进行交互，这些 API 和服务位于存储引擎之上。MySQL 服务器架构使应用程序免受存储引擎的详细级别复杂性的影响，通过提供易于使用的 API，这些 API 在所有存储引擎上都是一致的和适用的。如果应用程序需要更改底层存储引擎，或者添加一个或多个存储引擎以支持应用程序的需求，那么不需要进行重大的编码或流程更改即可使事情正常运行。

# 多种类型的存储引擎

现在我们知道了存储引擎的重要性，以及从众多可用的存储引擎中选择使用哪些存储引擎的关键决策。让我们看看有哪些可用的存储引擎以及它们的规格。当您开始考虑存储引擎时，`InnoDB`是您首先想到的名字，对吧？

InnoDB 是 MySQL 8 中的默认和最通用的存储引擎，Oracle 建议将其用于表以及特殊用例。MySQL 服务器具有可插拔的存储引擎架构，可以从已经运行的 MySQL 服务器中加载和卸载存储引擎。

在 MySQL 8 中，识别服务器支持的存储引擎非常容易。我们只需进入 MySQL shell 或提示符，并使用`SHOW ENGINES`语句。在提示时输入该语句，结果将是一列引擎，包括 Engine、Support、Transactions、Savepoints 和 Comment。

支持列中的值 DEFAULT、YES 和 NO 表示存储引擎是否可用，并当前设置为默认存储引擎。

# InnoDB 存储引擎概述

`InnoDB`是 MySQL 8 中默认的、最通用的存储引擎，提供高可靠性和高性能。

如果您没有配置不同的默认存储引擎，那么在 MySQL 8 中发出不带`ENGINE =`子句的 SQL 语句`CREATE TABLE`将创建一个具有存储引擎`InnoDB`作为默认引擎的表。

`InnoDB`存储引擎提供的功能和优势将在*InnoDB 存储引擎*部分中进行解释。

# 自定义存储引擎

MySQL 5.1 和所有后续版本以及 MySQL 8 中的存储引擎架构都利用了灵活的存储引擎架构。

存储引擎可插拔架构提供了创建和添加新存储引擎的能力，而无需重新编译服务器，直接添加到正在运行的 MySQL 服务器。这种架构使得开发和部署新的存储引擎到 MySQL 8 变得非常容易。

我们将在即将到来的*创建自定义存储引擎*部分中使用 MySQL 存储引擎架构的可插拔特性来开发一个新的存储引擎。

# 多种类型的存储引擎

在这一部分，我们将更仔细地查看 MySQL 8 支持的广泛使用的存储引擎。但在查看它们之前，让我们看看存储引擎架构是如何可插拔的，并提供了灵活性，以便在同一模式或服务器中使用多个存储引擎。

以下是 MySQL 8 支持的存储引擎列表。

+   `InnoDB`：MySQL 8 的默认存储引擎。它是一个符合`ACID`（事务安全）的存储引擎，具有提交、回滚和崩溃恢复，用于保护用户数据和`引用完整性`约束以维护数据完整性，等等。

+   `MyISAM`：具有小占用空间的表的存储引擎。它具有表级锁定，因此主要用于只读或读最多的数据工作负载，例如数据仓库和 Web 配置。

+   `Memory`：以前被称为`HEAP`引擎的存储引擎。它将数据保存在 RAM 中，提供更快的数据访问，主要用于非关键数据环境的快速查找。

+   `CSV`：这种存储引擎使用文本文件和表中的逗号分隔值作为表。它们没有索引，主要用于以`CSV`格式导入和转储数据。

+   `存档`：这种存储引擎包括紧凑的、无索引的表，旨在存储和检索大量历史、归档或安全审计数据。

+   `黑洞`：这种存储引擎包括用于复制配置的表。查询总是返回一个空集。`DML` SQL 语句被发送到从服务器。它接受数据，但数据不会被存储，就像在 Unix 的`/dev/null`设备中使用一样。

+   `合并`：这种存储引擎提供了将一系列相似的`MyISAM`表逻辑分组并将它们称为一个对象的能力，而不是单独的表。

+   `联合`：这种存储引擎可以将许多独立的物理 MySQL 服务器链接成一个逻辑数据库。它非常适合数据仓库或分布式环境。

+   `示例`：这种存储引擎什么也不做，只是作为一个`存根`。它主要由开发人员使用，用来演示如何在 MySQL 源代码中开始编写新的存储引擎。

MySQL 不限制在整个服务器或模式上使用相同的存储引擎；相反，在表级别指定引擎使其根据数据类型和应用程序的用例变得灵活。

# 可插拔存储引擎架构

MySQL 服务器使用可插拔存储引擎架构，可以从已运行的 MySQL 服务器中加载和卸载存储引擎：

+   **安装存储引擎**：在服务器中使用存储引擎之前，必须使用`INSTALL PLUGIN` SQL 语句将存储引擎插件共享库加载到 MySQL 中。如果您创建了一个名为`MyExample`的`MYEXAMPLE`引擎插件，并且共享库的名称为`MyExample.so`，那么您需要使用以下语句加载它们：

```sql
 mysql> INSTALL PLUGIN MyExample SONAME 'MyExample.so';
```

要安装存储引擎，发出前述语句的用户必须对`mysql.plugin`表具有`INSERT`权限，并且插件文件必须存在于 MySQL 插件目录中。共享库也必须存在于`plugin_dir`变量中给出的 MySQL 服务器插件目录中。

+   **卸载存储引擎**：在卸载存储引擎之前，请确保没有表在使用该存储引擎。如果卸载了一个存储引擎，并且任何现有表需要该存储引擎，那么这些表将变得不可访问，并且只会存在于适用的磁盘上。如果您卸载了名为`MyExample`的`MYEXAMPLE`引擎插件，然后执行以下语句来卸载存储引擎：

```sql
 mysql> UNINSTALL PLUGIN MyExample ;
```

# 常见的数据库服务器层

MySQL 可插拔存储引擎负责在实际数据上执行 I/O 操作，并满足特定应用程序的需求，包括在需要时启用和强制执行所需的功能。使用特定或单一存储引擎更有可能导致更高的效率和更高的数据库性能，因为该引擎仅启用特定应用程序所需的功能，从而减少数据库的系统开销。

存储引擎支持以下独特的基础设施组件或键：

+   **并发性**：一些应用程序对锁级别（如行级锁）的要求比其他应用程序更细粒度。选择正确/错误的锁定策略以及多版本并发控制或快照读取功能都可能影响整体性能和由于锁定而产生的开销。

+   **事务支持**：存在非常明确定义的要求，比如`ACID`兼容性，如果应用程序需要事务，则还有更多要求。

+   **引用完整性**：服务器可以使用`DDL`定义的外键来强制关系数据库引用完整性，如果需要的话。

+   **物理存储**：这包括从表和索引的页面大小到在物理磁盘上存储数据所使用的格式等一切。

+   **索引支持**：这包括基于应用程序需求的索引策略，因为每个存储引擎都有自己的索引方法。

+   **内存缓存**：这是基于应用程序需求的缓存策略，因为每个存储引擎都有自己的缓存方法，以及所有存储引擎的通用内存缓存。

+   **性能辅助**：这涉及到大量插入处理、数据库检查点、多个 I/O 线程进行并行操作、线程并发性等。

+   **其他目标特性**：这可能包括对某些数据操作的安全限制、地理空间操作和其他类似特性的支持。

前述的基础设施组件都是为了支持特定应用程序需求的一组特定功能而设计的，因此非常重要的是要非常仔细地了解应用程序的需求，并选择正确的存储引擎，因为这可能会影响整个系统的效率和性能。

# 设置存储引擎

当使用`CREATE TABLE`语句创建新表时，可以使用`ENGINE`表选项指定要为表使用的引擎。如果不指定`ENGINE`表选项，则将使用默认的存储引擎。`InnoDB`是 MySQL 8.0 的默认引擎。您还可以使用`ALTER TABLE`语句将表从一个存储引擎转换为另一个存储引擎，如下例所示：

```sql
CREATE TABLE table1 (i1 INT) ENGINE = INNODB;
CREATE TABLE table3 (i3 INT) ENGINE = MEMORY;
ALTER TABLE table3 ENGINE = InnoDB;
```

可以通过设置`default_storage_engine`变量为当前会话设置默认存储引擎，如下例所示：

```sql
SET default_storage_engine=MEMORY;
```

使用`CREATE TEMPORARY TABLE`创建`TEMPORARY`表的默认存储引擎可以通过在启动或运行时设置`default_tmp_storage_engine`变量来单独设置。

# `MyISAM`存储引擎

`MyISAM`存储引擎使用占用空间小的表。它实现了表级锁定，因此主要用于只读或读取大部分数据负载的情况，例如数据仓库和 Web 配置。每个`MyISAM`表都存储在磁盘上的两个文件中。文件名以表名和其扩展类型开头，一个带有`.MYD`扩展名的数据文件，另一个带有`.MYI`扩展名的索引文件。

对于`MyISAM`引擎，有几个在`mysqld`中指定的启动选项可以改变`MyISAM`表的行为；例如：

```sql
--myisam-recover-options=mode
```

此选项将设置在`MyISAM`中崩溃表的自动恢复模式。

在`MyISAM`中需要用于键的空间，`MyISAM`表使用`B-Tree`索引，并且在`String`索引中使用空间压缩。如果一个字符串是索引的第一部分，那么还会进行前缀压缩，这样整体使索引文件大小更小。前缀压缩有助于处理许多具有相似前缀的字符串。通过在`MyISAM`表中使用表选项`PACK_KEYS=1`，前缀压缩也可以应用于数字，如果有许多具有相似前缀的数字。

在 MySQL 8.0 中，不支持对`MyISAM`表进行分区。

`MyISAM`表的一些重要特性如下：

+   存储的所有数据值都以低字节优先顺序存储，这使得数据独立于机器和操作系统

+   所有数值键值都以高字节优先顺序存储，这允许更好的索引压缩

+   `MyISAM`表的行数限制为*(2³²)²(1.844E+19)*。

+   `MyISAM`表的最大索引数限制为 64 个

+   `MyISAM`表的列索引最大数限制为 16 个

+   在`MyISAM`中支持并发插入，如果表在数据文件中间没有空闲块

+   `MyISAM`中也可以对`TEXT`和`BLOB`类型的列进行索引

+   在索引列中，允许`NULL`值

+   每一列都可以有不同的字符集

+   它还支持真正的 VARCHAR 类型列，其起始长度存储为 1 或 2 个字节，具有固定或动态行长度的 VARCHAR 列，以及任意长度的 UNIQUE 约束

+   MyISAM 表存储格式：MyISAM 支持以下三种不同类型的存储格式：

+   静态表：MyISAM 存储引擎中表的默认格式，具有固定大小的列

+   动态表：顾名思义，包含可变大小列的格式，包括 VARCHAR、BLOB 或 TEXT

+   压缩表：用于在 MyISAM 存储引擎表中保存只读数据和压缩格式的表格格式

前两种格式，固定和动态，根据使用的列类型自动选择。压缩格式可以通过使用 myisampack 实用程序创建。

+   MyISAM 表问题：文件格式经过了广泛测试，但有些情况会导致数据库表损坏。让我们看看这些情况以及恢复这些表的方法。

在以下事件中可能会出现损坏的表：

+   如果 mysqld 进程在写入过程中被杀死

+   如果有意外的计算机关闭

+   如果有任何硬件故障

+   如果 MySQL 服务器和外部程序（如 myisamchk）同时修改表

+   MySQL 或 MyISAM 代码存在软件错误

使用 CHECK TABLE 语句检查表的健康状况，并尝试使用 REPAIR TABLE 语句修复任何损坏的 MyISAM 表。

MyISAM 表可能出现的问题是表没有被正确关闭。为了确定表是否被正确关闭，每个 MyISAM 索引文件在标头中保留一个计数器。在以下情况下，计数器可能不正确：

+   如果表在不发出 LOCK TABLES 和 FLUSH TABLES 的情况下被复制

+   MySQL 在更新期间最终关闭之前崩溃

+   mysqld 正在使用表，同时被另一个程序修改：myisamcheck --recover 或 myisamchk --update-state

# MEMORY 存储引擎

MEMORY 存储引擎，以前也称为 HEAP 引擎，将数据保存在 RAM 中，提供更快的数据访问。它主要用于快速查找非关键数据环境。它创建专用表，其中内容存储在内存中，但数据容易受到崩溃、停电和硬件问题的影响。因此，这些表用于临时工作区或在从其他表中提取数据后缓存只读数据。

您应该选择使用 MEMORY 还是 NDB Cluster。您应该检查应用程序是否需要重要的、高可用的或经常更新的数据，并考虑 NDB Cluster 是否是更好的选择。NDB Cluster 提供与 MEMORY 引擎相同的功能，但性能水平更高，并且具有 MEMORY 引擎不提供的其他功能。这些包括：

+   客户端之间的低争用通过多线程操作和行级锁定

+   包括写入的语句混合的可伸缩性

+   数据耐久性；它支持可选的磁盘支持操作

+   无共享架构，提供多主机操作而没有单点故障，为应用程序提供 99.999%的可用性

+   自动数据分布跨节点

+   支持可变长度数据类型，包括 BLOB 和 TEXT

MEMORY 表不支持分区

性能取决于服务器的繁忙程度以及单线程执行对更新处理期间的表锁开销的影响。在更新处理期间对表进行锁定会导致在 MEMORY 表上的多个会话的并发使用减慢。

**MEMORY 表特点**：表定义存储在 MySQL 数据字典中，并不在磁盘上创建任何文件。以下是表特性的亮点：

+   100%动态哈希用于插入，并且空间分配在小块中。

+   不需要额外的键空间、溢出区域或空闲列表的额外空间。通过将行放入链接列表中重用已删除的行来插入新记录。

+   固定长度行存储格式，`VARCHAR`，以固定长度存储。无法存储`BLOB`或`TEXT`列。

+   支持`AUTO_INCREMENT`列。

`MEMORY`存储引擎支持`HASH`和`BTREE`类型的索引。`MEMORY`表每个表最多有 64 个索引，每个索引最多有 16 列，最大键长度为 3,072 字节。`MEMORY`表也可以有`非唯一`键。

**用户创建和临时表**：服务器在处理查询时动态创建内部临时表。两种类型的表在存储转换上有所不同，其中`MEMORY`表不受转换的影响：

+   当内部临时表变得太大时，服务器会自动将其转换为磁盘存储

+   用户创建的`MEMORY`表不会被服务器转换

可以使用`--init-file`选项，使用`INSERT INTO ... SELECT`或`LOAD DATA INFILE`语句从任何持久性数据源加载数据，如果需要的话。

# CSV 存储引擎

该存储引擎以逗号分隔值的形式将数据存储在文本文件中。该引擎始终编译到 MySQL 服务器中，可以从 MySQL 分发的`storage/csv`目录中检查源代码。

服务器创建的数据文件以给定表和扩展名`.CSV`开头。数据文件是一个纯文本文件，以逗号分隔值格式包含数据。

MySQL 服务器创建一个与`CSV`表对应的元文件，该文件存储有关表状态和表中存在的行数的信息。元文件也与表名一起存储在以`.CSM`扩展名开头的位置。

+   **修复和检查**`CSV`**表**：存储引擎支持`CHECK`和`REPAIR`语句来验证并可能修复损坏的`CSV`表。您可以使用`CHECK TABLE`语句来验证或验证表，并使用`REPAIR TABLE`语句来修复从现有`CSV`数据文件复制有效行并用新复制/恢复的行替换现有文件的表。

在修复过程中，只有从`CSV`数据文件到第一个损坏的行的行被复制到新表或复制的数据文件中。损坏行后的其余行将从表中删除，包括有效行，因此建议您在进行修复之前对数据文件进行足够的备份。

`CSV`存储引擎不支持索引或分区，所有使用`CSV`存储引擎创建的表必须在所有列上具有`NOT NULL`属性。

# ARCHIVE 存储引擎

`ARCHIVE`存储引擎创建专用表，用于存储大量未索引数据，占用非常小的空间。

当创建`ARCHIVE`表时，它以表名开头，并以`.ARZ`扩展名结尾。在优化操作期间，可能会出现一个带有`.ARN`扩展名的文件。

引擎支持`AUTO_INCREMENT`列属性。它还支持`INSERT`、`REPLACE`、`SELECT`和`BLOB`列（除了空间数据类型），但不支持`DELETE`、`UPDATE`、`ORDER`或`BY`操作。

`ARCHIVE`存储引擎不支持分区：

+   **存储**：该引擎使用`zlib`进行无损数据压缩，并在插入时对行进行压缩。它支持`CHECK TABLE`操作。引擎使用几种插入类型：

+   `INSERT`语句将行发送到压缩缓冲区，并根据需要刷新缓冲区。压缩缓冲区中的插入受锁保护，只有在请求`SELECT`时才会发生刷新。

+   完成后可以看到一个批量缓冲区。只有在同时发生其他插入时才能看到。在加载任何正常插入时，刷新不会在`SELECT`时发生。

+   **检索**：检索后，根据请求解压行，并且不使用任何行缓存。对于`SELECT`操作执行完整的表扫描：

+   `SELECT`检查当前有多少行可用，并且只读取该数量的行。它作为一次一致的读操作执行。

+   `SHOW TABLE STATUS`报告的行数对于`ARCHIVE`表始终是准确的。

+   使用`OPTIMIZE TABLE`或`REPAIR TABLE`操作以实现更好的压缩。

# BLACKHOLE 存储引擎

`BLACKHOLE`存储引擎充当黑洞。它接受数据但不存储数据，查询总是返回空结果。

服务器只有在创建`BLACKHOLE`表并且没有文件与该表关联时，才会在全局数据字典中添加表定义。

`BLACKHOLE`存储引擎支持各种**索引**，因此可以在表定义中包含相同的内容。

`BLACKHOLE`存储引擎不支持分区。

对表的插入不会存储任何数据，但如果为语句启用了二进制日志记录，则会记录并复制到从服务器。这种机制可用作过滤器或中继器。

`BLACKHOLE`存储引擎有以下可能的用途：

+   转储文件语法验证

+   使用启用或禁用二进制日志记录的`BLACKHOLE`性能比较的开销测量

+   它还可用于查找任何性能瓶颈，除了存储引擎本身

**自增列**：由于该引擎是一个无操作引擎，它不会增加任何字段值，但它对复制有影响，这可能非常重要。考虑以下情况：

1.  主服务器具有带有主键的自增字段的`BLOCKHOLE`表

1.  从服务器上存在相同的表，但使用`MyISAM`引擎

1.  在`INSERT`语句中插入到主服务器的表中，而不设置任何自增值或使用`SET INSERT_ID`语句

在上述情况下，主键列上的复制将失败，因为有重复条目。

# MERGE 存储引擎

`MERGE`存储引擎，也称为`MRG_MyISAM`引擎，是一组类似的表，可以作为一个表来使用。这里的“类似”意味着所有表具有相似的列数据类型和索引信息。不可能合并列顺序不同的表，或者在各自列中具有相同的数据类型，或者以不同的顺序进行索引。

以下是不会限制合并的表中的差异列表：

+   各自列和索引的名称可能不同。

+   表，列和索引之间的注释可能不同。

+   `AVG_ROW_LENGTH`，`MAX_ROWS`或`PACK_KEYS`表选项可能不同。

创建`MERGE`表时，MySQL 还会在磁盘上创建一个`.MRG`文件，其中包含正在使用的`MyISAM`表的名称。表的格式存储在 MySQL 数据字典中，底层表不需要在与`MERGE`表相同的数据库中。

必须具有对与`MERGE`表映射的`MyISAM`表的`SELECT`，`UPDATE`和`DELETE`权限，因此可以使用`SELECT`，`INSERT`，`UPDATE`和`DELETE`语句对`MERGE`表进行操作。

在`MERGE`表上执行`DROP TABLE`语句将仅删除`MERGE`的规范，对底层表不会产生影响。

使用`MERGE`表存在以下安全问题。如果用户可以访问`MyISAM`表`t1`，那么用户可以创建可以访问`t1`的`MERGE`表`m1`。现在，如果用户对表`t1`的权限被撤销，用户仍然可以通过使用表`m1`继续访问表`t1`。

# FEDERATED 存储引擎

`FEDERATED`存储引擎可以将许多独立的物理 MySQL 服务器链接成一个逻辑数据库，因此可以让您访问远程 MySQL 服务器的数据，而无需使用复制或集群技术。

当我们查询本地`FEDERATED`表时，会自动从远程联合表中提取数据，不需要将数据存储在本地表中。

`FEDERATED`存储引擎不是 MySQL 服务器的默认支持，但是使用`--federated`选项启动服务器将启用`FEDERATED`引擎选项。

创建`FEDERATED`表时，表定义与其他表相同，但关联数据的物理存储是在远程服务器上处理的。`FEDERATED`表包括以下两个元素：

+   一个**远程服务器**，其中包含一个由表定义和相关表数据组成的数据库表。这种类型的表可以是远程服务器支持的任何类型，包括`MyISAM`或`InnoDB`。

+   一个**本地服务器**，其中包含一个由远程服务器上相同的表定义组成的数据库表。表定义存储在数据字典中，本地服务器上没有关联的数据文件存储。相反，除了表定义之外，它还保留一个指向远程表本身的连接字符串。

当在`FEDERATED`表上执行 SQL 语句时，本地服务器和远程服务器之间的信息流如下：

1.  该引擎检查表的每一列，并构建一个适当的 SQL 语句，引用远程表。

1.  MySQL 客户端 API 用于将 SQL 语句发送到远程服务器。

1.  该语句由远程服务器处理，并且本地服务器检索相应的结果。

# EXAMPLE 存储引擎

`EXAMPLE`存储引擎只是一个存根引擎，其目的是在 MySQL 源代码中提供示例，以帮助开发人员编写新的存储引擎。

要使用`EXAMPLE`引擎源代码，请查看 MySQL 源代码分发下载的`storage/example`目录。

如果使用`EXAMPLE`引擎创建表，则不会创建文件。数据不能存储在`EXAMPLE`引擎中，并且返回空结果。

`EXAMPLE`存储引擎不支持索引和分区。

# InnoDB 存储引擎

`InnoDB`是最通用的存储引擎，也是 MySQL 8 中的默认引擎，提供高可靠性和高性能。

`InnoDB`存储引擎提供的主要优势如下：

+   其`DML`操作遵循`ACID`模型，并且事务具有提交、回滚和崩溃恢复功能，以保护用户数据

+   `Oracle-style`提供一致的读取和行级锁定，增加了多用户并发性能

+   每个`InnoDB`表都有一个主键索引，称为聚簇索引，它按顺序在磁盘上排列数据，以优化基于主键的查询，并在主键查找期间最小化 I/O

+   通过支持外键，插入、删除和更新都会进行检查，以确保跨不同表的一致性，以维护数据完整性

使用`InnoDB`表的主要优势如下：

+   如果服务器由于任何硬件或软件问题而崩溃，无论当时服务器正在处理什么更改，重新启动服务器后都不需要进行任何特殊操作。它具有崩溃恢复系统，可以处理在服务器崩溃期间提交的更改。它将转到这些更改并从处理中断的地方开始。

+   引擎具有自己的缓冲池，用于根据访问的数据将表和索引数据缓存到内存中。经常使用的数据直接从缓存内存中获取，因此可以加快处理速度。在专用服务器中，它占用分配的物理内存的 80％用于缓冲池。

+   使用外键设置将相关数据拆分到表中，强制执行引用完整性，防止在没有主表中相应数据的情况下向辅助表插入任何不相关的数据。

+   如果内存或磁盘中存在损坏的数据，校验和机制会在我们使用之前提醒我们有损坏的数据。

+   更改缓冲区会自动优化`Insert`，`Update`和`Delete`。`InnoDB`还允许对同一表进行并发读写访问，并缓存数据更改以简化磁盘 I/O。

+   当从表中重复访问相同的数据行时，自适应哈希索引功能可以加快查找速度并提供性能优势。

+   允许在表和相关索引上进行压缩。

+   通过查询`INFORMATION_SCHEMA`或`Performance Schema`表，轻松监视存储引擎的内部工作和性能细节。

现在让我们看看存储引擎的每个区域，在这些区域中`InnoDB`被增强或优化以提供非常高效和增强的性能。

# ACID 模型

`ACID`模型是一组数据库设计原则，强调可靠性，这对于关键任务应用程序和业务数据至关重要。

MySQL 具有诸如`InnoDB`存储引擎之类的组件，严格遵循`ACID`模型。因此，即使在硬件故障或软件崩溃的特殊情况下，数据也是安全且不会损坏。

使用 MySQL 8，`InnoDB`支持原子`DDL`，确保即使在执行操作时服务器停止，`DDL`操作也会完全提交或回滚。现在`DDL`日志可以写入`mysql.innodb_ddl_log`配置以用于数据字典表，并启用`innodb_print_ddl_logs`配置选项以将`DDL`恢复日志打印到`stderr`。

# 多版本

InnoDB 是一种多版本存储引擎。这意味着它具有保留更改的旧版本行数据信息并支持事务特性（如并发性和回滚）的能力。信息存储在表空间、数据结构和命名回滚段中。

在内部，对于存储在数据库中的每一行，`InnoDB`创建三个字段：6 字节的`DB_TRX_ID`，7 字节的`DB_ROLL_PTR`（称为回滚指针）和 6 字节的`DB_ROW_ID`。有了这些字段，`InnoDB`创建了聚集索引，以保留数据库中更改的行数据信息。

# 架构

在本节中，我们将简要介绍`InnoDB`架构的主要组件：

+   缓冲池：主内存区域，用于缓存表和索引数据以加快处理速度

+   更改缓冲区：缓存对辅助索引页面的更改的特殊数据结构

+   自适应哈希索引：使内存数据库能够在具有平衡和适当组合的缓冲池内存和工作负载的系统上进行查找和操作

+   重做日志缓冲区：存储数据以便写入重做日志的内存区域

+   系统表空间：存储`doublewrite`缓冲区、撤销日志和更改缓冲区的存储区域，在 MySQL 8 数据字典信息之前存储

+   双写缓冲区：系统表空间中的存储区域，用于写入从缓冲池刷新的页面

+   **撤销日志**：与任何单个事务相关联的撤销日志记录的集合

+   **每个表的表空间**：添加到自己的数据文件的单表表空间

+   **通用表空间**：使用`CREATE TABLESPACE`语法创建的共享表空间

+   **撤销表空间**：一个或多个带有撤销日志的文件

+   **临时表空间**：用于非压缩临时表及其相关对象

+   **重做日志**：用于在崩溃恢复期间纠正不完整事务数据的基于磁盘的数据结构

在 MySQL 8 中，`InnoDB`存储引擎利用全局 MySQL 数据字典，而不是其自己的存储引擎特定数据字典。

# 锁定和事务模型

本节简要介绍了`InnoDB`使用的锁定和`InnoDB`实现的事务模型。`InnoDB`使用以下不同类型的锁定：

+   **共享和排他锁**：实现了两种标准的行级锁定。共享锁允许您将一行读取到不同的事务中；排他锁用于更新或删除一行，并且不允许您将该行读取到任何不同的事务中。

+   **意向锁**：表级锁，支持多粒度锁定，`InnoDB`实际上维护了行级锁和整个表级锁的共存。

+   **记录锁**：索引记录锁，防止任何其他事务插入、更新或删除记录。

+   **间隙锁**：锁定适用于索引记录之间的间隙（范围）。

+   **下一个键锁**：在前一个索引记录的间隙上组合索引记录锁和间隙锁。

+   **插入意向锁**：`INSERT`操作在插入行之前设置的一种间隙锁类型。

+   **AUTO-INC 锁**：用于插入具有`AUTO_INCREMENT`列的记录的特殊表级锁。

+   **空间索引的谓词锁**：对空间索引的锁定，使支持具有空间索引的表的隔离级别

遵循事务模型的目标是将传统的两阶段锁定与多版本数据库属性的最佳部分结合起来。执行行级锁定，并使用非锁定一致性读取运行查询。`InnoDB`负责事务隔离级别、自动提交、回滚和提交以及锁定读取。它允许根据需要进行非锁定一致性读取。`InnoDB`还使用一种机制来避免幻影行，并配置支持自动死锁检测。

# 配置

本节提供了有关`InnoDB`初始化启动中使用的配置和程序的简要信息，适用于不同的`InnoDB`组件：

+   `InnoDB` **启动配置**：包括指定启动选项、日志文件配置、存储考虑事项、系统表空间数据文件、撤销表空间、临时表空间、页面大小和内存配置

+   **用于只读操作的** `InnoDB`：使用`--innodb-read-only=1`选项，可以将 MySQL 实例配置为只读操作，当使用只读介质（如`CD`或`DVD`）时非常有用

+   `InnoDB` **缓冲池配置**：配置缓冲池大小、多个实例、刷新和监控

+   `InnoDB` **更改缓冲**：为辅助索引缓存配置更改缓冲选项

+   **`InnoDB`的线程并发性**：并发线程计数限制配置

+   **后台** `InnoDB` **I/O 线程的数量**：配置后台线程的数量，用于对数据页进行 I/O 读/写操作

+   在 Linux 上使用异步 I/O：在 Linux 上使用本机异步 I/O 子系统的配置

+   **`InnoDB`主线程 I/O 速率**：配置后台工作的主线程的整体 I/O 容量，负责多个任务

+   **自旋锁轮询**：配置自旋等待延迟周期，以控制多个线程之间频繁轮询以获取`mutexes`或`rw-locks`的最大延迟

+   `InnoDB` **清除调度**：为适用的可伸缩性配置清除线程。

+   **`InnoDB`的优化器统计信息**：配置持久和非持久的优化器统计参数。

+   **索引页的合并阈值**：配置`MERGE_THRESHOLD`以减少合并分裂行为。

+   **启用专用 MySQL 服务器的自动配置**：配置专用服务器选项`--innodb_dedicated_server`，以自动配置缓冲池大小和日志文件大小。

# 表空间

本节提供了关于表空间和在`InnoDB`中执行的表空间相关操作的简要信息：

+   **调整`InnoDB`系统表空间的大小**：在启动/重新启动 MySQL 服务器时，增加和减少系统表空间的大小。

+   **更改`InnoDB`重做日志文件的数量或大小**：在启动/重新启动 MySQL 服务器之前，分别配置`my.cnf`中的`innodb_log_files_in_group`和`innodb_log_file_size`值。

+   **使用原始磁盘分区作为系统表空间的数据文件**：配置原始磁盘分区以用作系统表空间中的数据文件。

+   `InnoDB` **每表表空间**：默认启用了`innodb_file_per_table`功能，确保每个表和相关索引都存储在单独的`.idb`数据文件中。

+   **配置撤消表空间**：配置设置撤消表空间的数量，其中撤消日志驻留。

+   **截断撤消表空间**：配置`innodb_undo_log_truncate`以启用截断超过`innodb_max_undo_log_size`定义的最大限制的撤消表空间文件。

+   `InnoDB` **通用表空间**：使用`CREATE TABLESPACE`语句创建的共享表空间。它类似于系统表空间。

+   `InnoDB` **表空间加密**：支持以文件为基础的表空间存储的表的数据加密，使用`AES`分块加密算法。

# 表和索引

本节提供了关于`InnoDB`表和索引以及它们相关操作的简要信息：

+   **创建`InnoDB`表**：使用`CREATE TABLE`语句创建表。

+   **`InnoDB`表的物理行结构**：取决于表创建时指定的行格式。如果未指定，则使用默认的`DYNAMIC`。

+   **移动或复制`InnoDB`表**：将一些或所有`InnoDB`表移动或复制到不同的实例或服务器的不同技术。

+   **将表从`MyISAM`转换为`InnoDB`**：在将`MyISAM`表转换为`InnoDB`表时考虑指南和提示，但不支持分区表，这在 MySQL 8 中不受支持。

+   `InnoDB`中的`AUTO_INCREMENT` **处理**：使用`innodb_autoinc_lock_mode`参数配置`AUTO_INCREMENT`的模式为 0、1 和 2，分别为传统、连续或交错，其中交错是 MySQL 8 的默认模式。

+   **`InnoDB`表的限制**：表最多可以包含 1,017 列，最多可以包含 64 个次要索引，以及基于页面大小、表大小和数据行格式定义的其他限制。

+   **聚集和次要索引**：`InnoDB`使用称为聚集索引的特殊索引。其余的索引称为次要索引。

+   **`InnoDB`索引的物理结构**：对于空间索引，`InnoDB`使用专门的数据结构`R-tree`。对于其他索引，使用`B-tree`数据结构。

+   **排序索引构建**：在创建或重建索引进行插入时进行批量加载。它们被称为排序索引构建，并且不支持空间索引。

+   `InnoDB` `FULLTEXT` **索引**：为基于文本的列（`char`，`varchar`或`text`类型）创建。它们有助于加快查询和搜索操作的速度。

# INFORMATION_SCHEMA 表

本节提供了`InnoDB` `INFORMATION_SCHEMA`表的用法示例和相关信息。

它提供了有关`InnoDB`存储引擎不同方面的元数据、统计和状态信息。

可以通过在`INFORMATION_SCHEMA`数据库上执行`SHOW TABLES`语句来检索`InnoDB` `INFORMATION_SCHEMA`表的列表：

```sql
mysql> SHOW TABLES FROM INFORMATION_SCHEMA LIKE 'INNODB%';
```

+   **关于压缩的表**：`INNODB_CMP`和`INNODB_CMP_RESET`表提供了有关压缩操作次数和压缩相关信息所花费的时间。在压缩期间的内存分配在`INNODB_CMPMEM`和`INNODB_CMPMEM_RESET`表中提供。

+   **事务和锁信息**：`INNODB_TRX`包含当前执行的事务信息，`Performance Schema`表中的`data_locks`和`data_lock_waits`表提供有关锁的信息。

+   **模式对象表**：提供有关`InnoDB`模式对象的元数据信息。

+   `FULLTEXT` **索引表**：提供有关`FULLTEXT`索引的元数据信息。

+   **缓冲池表**：提供有关缓冲池中页面的状态信息和元数据。

+   **指标表**：提供性能和资源相关信息。

+   **临时表信息表**：提供有关当前在`InnoDB`实例中活动的所有用户和系统创建的临时表的元数据信息。

+   **检索`InnoDB`表空间元数据**：提供有关`InnoDB`实例中所有类型的表空间的元数据信息。

已添加了一个新视图`INNODB_TABLESPACES_BRIEF`，用于提供名称、路径、标志、空间和空间类型数据。

已添加了一个新表`INNODB_CACHED_INDEXES`，用于提供缓冲池中每个索引的索引页数。

# Memcached 插件

MySQL 8 为您提供了名为`daemon_memcached`的`InnoDB` memcached 插件，可以帮助我们轻松管理数据。它将自动从`InnoDB`表中存储和检索数据，并提供`get`、`set`和`incr`操作，通过跳过 SQL 解析来消除性能开销，从而加快数据操作。`memcached`插件使用集成的`memcached`守护程序，自动从`InnoDB`表中检索和存储数据，使 MySQL 服务器能够快速将数据发送到`键值`存储。

使用`InnoDB memcached`插件的主要好处如下：

+   直接访问`InnoDB`存储引擎，减少解析和规划 SQL 开销

+   `memcached`使用与 MySQL 服务器相同的进程空间，减少了网络开销

+   以`memcached`协议编写或请求的数据会透明地从`InnoDB`表中写入或查询，减少了必须经过 SQL 层开销的情况

+   通过自动在磁盘和内存之间传输，简化应用逻辑

+   MySQL 数据库存储数据，以防止损坏、崩溃或中断

+   在主服务器上使用`daemon_memcached`插件和 MySQL 复制结合，确保高可用性

+   使用`InnoDB`缓冲池缓存重复的数据请求，提供高速处理

+   由于数据存储在`InnoDB`表中，数据一致性会自动执行

`InnoDB memcached`插件支持多个获取操作（在单个`memcached`查询中获取多个键/值对）和范围查询。

# 创建自定义存储引擎

MySQL AB 在 MySQL 5.1 中引入了可插拔存储引擎架构，包括 MySQL 8 在内的所有后续版本都利用了灵活的存储引擎架构。

存储引擎可插拔架构提供了在不重新编译服务器的情况下创建和添加新存储引擎的能力，直接添加到运行中的 MySQL 服务器。这种架构使得开发和部署新的存储引擎到 MySQL 8 变得非常容易。

在开发新的存储引擎时，需要注意为存储引擎工作的所有组件。这些包括安装处理程序、对表的操作（如创建、打开和关闭）、`DML`、索引等。

在本节中，我们将介绍如何可以在高层次基础上开始开发新的存储引擎，参考 MySQL 开发社区提供的文档。创建自定义存储引擎需要具备使用`C`和`CPP`进行开发的工作知识，以及使用`cmake`和`Visual Studio`进行编译。

# 创建存储引擎源文件

实现新存储引擎的最简单方法是通过复制和修改`EXAMPLE`存储引擎开始。文件`ha_example.cc`和`ha_example.h`可以在 MySQL 源分发的`storage/example`目录中找到。

在复制文件时，将名称从`ha_example.cc`和`ha_example.h`更改为适合您的存储引擎的名称，例如`ha_foo.cc`和`ha_foo.h`。

在复制和重命名文件后，必须将所有`EXAMPLE`和`example`的实例替换为您的存储引擎的名称。

# 添加特定于引擎的变量和参数

插件可以实现状态和系统变量，在本节中我们已经介绍了变量和参数的更改，以及适当的值和数据类型。

服务器插件接口使插件能够使用通用插件描述符的`status_vars`和`system_vars`成员公开状态和系统变量。

`status_vars`是通用插件描述符的成员。如果值不为 0，则指向一个`st_mysql_show_var`结构的数组，其中每个结构描述一个状态变量，后跟一个所有成员都设置为 0 的结构。`st_mysql_show_var`结构的定义如下：

```sql
struct st_mysql_show_var {   
  const char *name;   
  char *value;   
  enum enum_mysql_show_type type; 
};
```

插件安装后，插件名称和名称值用下划线连接，以形成`SHOW STATUS`语句显示的名称。

以下列表显示了允许的状态变量类型值以及相应的变量应该是什么：

+   `SHOW_BOOL`：这是一个指向`boolean`变量的指针

+   `SHOW_INT`：这是一个指向`integer`变量的指针

+   `SHOW_LONG`：这是一个指向长整型变量的指针

+   `SHOW_LONGLONG`：这是一个指向`longlong integer`变量的指针

+   `SHOW_CHAR`：这是一个`String`索引

+   `SHOW_CHAR_PTR`：这是一个指向`String`索引的指针

+   `SHOW_ARRAY`：这是一个指向另一个`st_mysql_show_var array`的指针

+   `SHOW_FUNC`：这是一个指向函数的指针

+   `SHOW_DOUBLE`：这是一个指向`double`的指针

所有会话和全局系统变量在使用之前都必须发布到`mysqld`。这是通过构建一个变量的`NULL`终止数组，并在插件公共接口中链接到它来实现的。

所有可变的和插件系统变量都存储在`HASH`结构中。

服务器命令行帮助文本的显示是通过编译所有相关变量的`DYNAMIC_ARRAY`，对其进行排序和迭代来显示每个选项。

在插件安装过程中，服务器处理命令行选项，插件成功加载后立即进行处理，但尚未调用插件初始化函数。

在`runtime`加载的插件不受任何配置选项的影响，必须具有可用的默认值。一旦安装，它们将在`mysqld`初始化时加载，并且可以在命令行或`my.cnf`中设置配置选项。

插件中的`thd`参数应被视为只读。

# 创建 handlerton

handlerton（处理程序单例的简称）定义了存储引擎。它包含指向应用于整个存储引擎的方法的方法指针，而不是在每个表上工作的方法。此类方法的示例包括处理提交和回滚操作的事务方法。

`EXAMPLE`存储引擎的示例如下：

```sql
handlerton example_hton= {
 "EXAMPLE", /* Name of the storage engine */
 SHOW_OPTION_YES, /* It should be displayed in options or not */
 "Example storage engine", /* Description of the storage engine */
 DB_TYPE_EXAMPLE_DB, /* Type of storage engine it should refer to */
 NULL, /* Initialize handlerton */
 0, /* slot  available */
 0, /* define savepoint size. */
 NULL, /* handle close_connection */
 NULL, /* handle savepoint */
 NULL, /* handle rollback to savepoint */
 NULL, /* handle release savepoint */
 NULL, /* handle commit */
 NULL, /* handle rollback */
 NULL, /* handle prepare */
 NULL, /* handle recover */
 NULL, /* handle commit_by_xid */
 NULL, /* handle rollback_by_xid */
 NULL, /* handle create_cursor_read_view */
 NULL, /* handle set_cursor_read_view */
 NULL, /* handle close_cursor_read_view */
 example_create_handler, /* Create a new handler instance */
 NULL, /* handle drop database */
 NULL, /* handle panic call */
 NULL, /* handle release temporary latches */
 NULL, /* Update relevant Statistics */
 NULL, /* Start Consistent Snapshot for reference */
 NULL, /* handle flush logs */
 NULL, /* handle show status */
 NULL, /* handle replication Report Sent to Binlog */
 HTON_CAN_RECREATE
};
```

有 30 个`handlerton`元素，其中只有少数是强制性的。

# 处理处理程序安装

这是创建新处理程序实例所需的存储引擎中的第一个方法调用。

在源文件中定义`handlerton`之前，必须在方法头中定义实例化方法。以下是`CSV`引擎显示实例化方法的示例：

```sql
static handler* tina_create_handler(TABLE *table);
```

如前面的示例所示，该方法接受一个指向表的指针。处理程序负责管理和返回处理程序对象。在方法头定义之后，使用方法指针在`create()` `handlerton`元素中命名方法。这将标识该方法负责在请求时生成新的处理程序实例。

以下示例显示了`MyISAM`存储引擎的实例化方法：

```sql
static handler *myisam_create_handler(TABLE *table)
 {
 return new ha_myisam(table);
 }
```

# 定义文件扩展名

存储引擎必须提供与给定表及其数据和索引相关的存储引擎使用的扩展名列表给 MySQL 服务器。

扩展应以空终止的字符串数组的形式给出，并且在调用[`custom-engine.html#custom-engine-api-reference-bas_ext bas_ext()`]方法时返回相同的内容，如下面的块所示：

```sql
const char **ha_tina::bas_ext() const
{
 return ha_tina_exts;
}
```

通过提供扩展信息，您还可以跳过实现`DROP TABLE`功能，因为 MySQL 服务器将通过关闭表并删除指定扩展名的所有文件来实现相同的功能。

# 创建表

在处理程序实例化之后，应该遵循创建表方法。存储引擎必须实现[`custom-engine.html#custom-engine-api-reference-create create()`]方法，如下面的块所示：

```sql
virtual int create(const char *name, TABLE *form, HA_CREATE_INFO *info)=0;
```

前面显示的方法应该创建所有必要的文件，但不会打开表。MySQL 服务器将单独调用打开表。

`*name`参数用于传递表的名称，`*form`参数用于传递`TABLE`结构。表结构定义了表，并匹配`tablename.frm`的内容。存储引擎不得修改`tablename.frm`文件，否则将导致错误或不可预测的问题。

`*info`参数是包含有关`CREATE TABLE`语句的信息的结构。它用于创建表，结构在`handler.h`文件中定义。以下是参考结构：

```sql
typedef struct st_ha_create_information
{
 CHARSET_INFO *table_charset, *default_table_charset; /* charset in table */
 LEX_STRING connect_string; /* connection string */
 const char *comment,*password; /* storing comments and password values */
 const char *data_file_name, *index_file_name; /* data and index file names */
 const char *alias; /* value pointer for alias */
 ulonglong max_rows,min_rows;
 ulonglong auto_increment_value;
 ulong table_options;
 ulong avg_row_length;
 ulong raid_chunksize;
 ulong used_fields;
 SQL_LIST merge_list;
 enum db_type db_type; /* value for db_type */
 enum row_type row_type; /* value for row_type */
 uint null_bits; /* NULL bits specified at start of record */
 uint options; /* OR of HA_CREATE_ options specification */
 uint raid_type,raid_chunks; /* raid type and chunks info */
 uint merge_insert_method;
 uint extra_size; /* length of extra data segments */
 bool table_existed; /* 1 in create if table existed */
 bool frm_only; /* 1 if no ha_create_table() */
 bool varchar; /* 1 if table has a VARCHAR */
} HA_CREATE_INFO;
```

存储引擎可以忽略`*info`和`*form`的内容，因为只有在存储引擎使用时才真正需要创建和初始化数据文件。

# 打开表

在对任何表执行任何读取或写入操作之前，MySQL 服务器调用[`custom-engine.html#custom-engine-api-reference-open handler::open()`]方法来打开表索引和数据文件：

```sql
int open(const char *name, int mode, int test_if_locked);
```

第一个参数是要打开的表的名称。第二个参数是要执行的文件操作。这些值在`handler.h`中定义：`O_RDONLY - 只读打开`，`O_RDWR - 读/写打开`。

最终选项决定处理程序在打开之前是否应检查表上的锁定。可以选择以下选项：

```sql
#define HA_OPEN_ABORT_IF_LOCKED 0 /* default */
#define HA_OPEN_WAIT_IF_LOCKED 1 /* wait if table is locked */
#define HA_OPEN_IGNORE_IF_LOCKED 2 /* ignore if locked */
#define HA_OPEN_TMP_TABLE 4 /* Table is a temp table */
#define HA_OPEN_DELAY_KEY_WRITE 8 /* Don't update index */
#define HA_OPEN_ABORT_IF_CRASHED 16
#define HA_OPEN_FOR_REPAIR 32 /* open even if crashed with repair */
```

典型的存储引擎将实现某种形式的共享访问控制，以防止在多线程环境中发生文件损坏。例如，查看`sql/example/ha_tina.cc`中的`get_share()`和`free_share()`方法来实现文件锁定。

# 实现基本表扫描

最基本的存储引擎实现了只读级别的表扫描，并且可能用于支持 SQL 查询，以请求从 MySQL 之外填充的日志和其他数据文件中获取信息。

方法的实现是创建高级存储引擎的第一步。以下显示了在`CSV`引擎的九行表扫描期间进行的方法调用：

```sql
ha_tina::store_lock
ha_tina::external_lock
ha_tina::info
ha_tina::rnd_init
ha_tina::extra - ENUM HA_EXTRA_CACHE Cache record in HA_rrnd()
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::rnd_next
ha_tina::extra - ENUM HA_EXTRA_NO_CACHE End caching of records (def)
ha_tina::external_lock
ha_tina::extra - ENUM HA_EXTRA_RESET Reset database to after open
```

可以实施以下方法来处理特定操作：

+   **实现** `store_lock()`: 此方法可以修改锁级别，忽略或为多个表添加锁

+   **实现** `external_lock()`: 当发出`LOCK TABLES`语句时调用此方法

+   **实现** `rnd_init()`: 此方法用于在表扫描中在表的开始处重置计数器和指针

+   **实现** `info(uinf flag)`: 此方法用于向优化器提供额外的表信息

+   **实现** `extra()`: 此方法用于向存储引擎提供额外的提示信息

+   **实现** `rnd_next()`: 此方法在扫描每一行直到达到`EOF`或满足搜索条件时调用

# 关闭表

当 MySQL 服务器完成与表的所有请求操作后，它将调用`custom-engine.html#custom-engine-api-reference-close close()`方法。它将关闭文件指针并释放所有相关资源。

使用共享访问方法的存储引擎在`CSV`引擎中可见。其他示例引擎必须从共享结构中删除相同的内容，如下所示：

```sql
int ha_tina::close(void)
 {
 DBUG_ENTER("ha_tina::close");
 DBUG_RETURN(free_share(share));
 }
```

存储引擎使用自己的共享管理系统。它们应使用所需的方法，以便从其处理程序中打开的相应表的共享中删除处理程序实例。

如果您的存储引擎编译为共享对象，在加载期间如果出现错误，例如`undefined symbol: _ZTI7handler`，则请确保使用与服务器相同的标志编译和链接您的扩展。此错误的常见原因是 LDFLAGS 缺少*-fno-rtti*选项。

# 高级自定义存储引擎的参考

我们已经详细介绍了前面的各个部分，为自定义存储引擎组件和所需的更改提供了高级信息。要在自定义存储引擎中实现`INSERT`、`UPDATE`、`DELETE`、索引等，需要具备使用`C/CPP`进行开发以及使用`cmake`和`Visual Studio`进行编译的工作知识。有关自定义存储引擎的高级开发，请参阅[`dev.mysql.com/doc/internals/en/custom-engine.html`](https://dev.mysql.com/doc/internals/en/custom-engine.html)中提供的详细信息

# 摘要

到目前为止，您已经了解了 MySQL 8 中可用的不同数据库引擎，以及我们为什么应该关注存储引擎和 MySQL 8 中可用的存储引擎选项。我们已经详细介绍了`InnoDB`存储引擎以及`InnoDB`存储引擎中已经提供的重要功能。现在，您实际上可以根据系统要求创建自定义存储引擎，并将其插入到 MySQL 8 中。选择适合您系统的存储引擎是一个重要方面，我们已经详细介绍了这一点。

在下一章中，您将了解 MySQL 8 中索引的工作原理，与索引相关的新功能，不同类型的索引以及如何在表中使用索引。除此之外，还将提供比较以及深入了解各种索引实现方式。
