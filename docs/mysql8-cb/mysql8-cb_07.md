# 第七章：备份

在本章中，我们将介绍以下内容：

+   使用 mysqldump 进行备份

+   使用 mysqlpump 进行备份

+   使用 mydumper 进行备份

+   使用平面文件进行备份

+   使用 XtraBackup 进行备份

+   备份实例

+   二进制日志备份

# 介绍

设置数据库后，下一个重要的事情是设置备份。在本章中，您将学习如何设置各种类型的备份。执行备份有两种主要方式。一种是逻辑备份，它将所有数据库、表结构、数据和存储例程导出为一组 SQL 语句，可以再次执行以重新创建数据库的状态。另一种类型是物理备份，它包含系统上数据库用于存储所有数据库实体的所有文件：

+   **逻辑备份工具**：`mysqldump`、`mysqlpump`和`mydumper`（未随 MySQL 一起提供）

+   **物理备份工具**：XtraBackup（未随 MySQL 一起提供）和平面文件备份

对于时间点恢复，备份应能够提供备份所涉及的二进制日志位置。这称为**一致备份**。

强烈建议从从属机器上的 filer 进行备份。

# 使用 mysqldump 进行备份

`mysqldump`是一个广泛使用的逻辑备份工具。它提供了各种选项，可以包括或排除数据库，选择要备份的特定数据，仅备份架构而不包括数据，或者仅备份存储例程而不包括其他内容等。

# 如何做...

`mysqldump`实用程序随`mysql`二进制文件一起提供，因此您无需单独安装它。本节涵盖了大多数生产场景。

语法如下：

```sql
shell> mysqldump [options]
```

在选项中，您可以指定用户名、密码和主机名以连接到数据库，如下所示：

```sql
--user <user_name> --password <password>
or
-u <user_name> -p<password>
```

在本章中，每个示例中都没有提到`--user`和`--password`，以便读者专注于其他重要选项。

# 所有数据库的完整备份

可以通过以下方式完成：

```sql
shell> mysqldump --all-databases > dump.sql
```

`--all-databases`选项备份所有数据库和所有表。`>`运算符将输出重定向到`dump.sql`文件。在 MySQL 8 之前，存储过程和事件存储在`mysql.proc`和`mysql.event`表中。从 MySQL 8 开始，相应对象的定义存储在`数据字典`表中，但这些表不会被转储。要在使用`--all-databases`进行转储时包括存储例程和事件，请使用`--routines`和`--events`选项。

包括例程和事件：

```sql
shell> mysqldump --all-databases --routines --events > dump.sql
```

您可以打开`dump.sql`文件查看其结构。前几行是转储时的会话变量。接下来是`CREATE DATABASE`语句，然后是`USE DATABASE`命令。接下来是`DROP TABLE IF EXISTS`语句，然后是`CREATE TABLE`；然后我们有实际的`INSERT`语句插入数据。由于数据存储为 SQL 语句，因此称为**逻辑备份**。

您会注意到，当您恢复转储时，`DROP TABLE`语句将在创建表之前清除所有表。

# 时间点恢复

为了获得时间点恢复，您应该指定`--single-transaction`和`--master-data`。

`--single-transaction`选项通过将事务隔离模式更改为`REPEATABLE READ`并在进行备份之前执行`START TRANSACTION`来提供一致的备份。仅在使用事务表（如`InnoDB`）时才有用，因为它会在不阻止任何应用程序的情况下转储发出`START TRANSACTION`时数据库的一致状态。

`--master-data`选项将服务器的二进制日志坐标打印到`dump`文件中。如果`--master-data=2`，它将作为注释打印。这还使用`FLUSH TABLES WITH READ LOCK`语句来获取二进制日志的快照。正如在第五章“事务”中所解释的那样，在存在任何长时间运行的事务时，这可能非常危险：

```sql
shell> mysqldump --all-databases --routines --events --single-transaction --master-data > dump.sql
```

# 转储主二进制坐标

备份始终在从服务器上进行。要获取备份时主服务器的二进制日志坐标，可以使用`--dump-slave`选项。如果要从主服务器获取二进制日志备份，请使用此选项。否则，请使用`--master-data`选项：

```sql
shell> mysqldump --all-databases --routines --events --single-transaction --dump-slave > dump.sql
```

输出将如下所示：

```sql
--
-- Position to start replication or point-in-time recovery from (the master of this slave)
--
CHANGE MASTER TO MASTER_LOG_FILE='centos7-bin.000001', MASTER_LOG_POS=463;
```

# 特定数据库和表

要仅备份特定数据库，请执行以下操作：

```sql
shell> mysqldump --databases employees > employees_backup.sql
```

要仅备份特定表，请执行以下操作：

```sql
shell> mysqldump --databases employees --tables employees > employees_backup.sql
```

# 忽略表

要忽略某些表，可以使用`--ignore-table=database.table`选项。要指定要忽略的多个表，请多次使用该指令：

```sql
shell> mysqldump --databases employees --ignore-table=employees.salary > employees_backup.sql
```

# 特定行

`mysqldump`可帮助您过滤备份的数据。假设您要备份 2000 年后加入的员工的备份：

```sql
shell> mysqldump --databases employees --tables employees --databases employees --tables employees  --where="hire_date>'2000-01-01'" > employees_after_2000.sql
```

您可以使用`LIMIT`子句来限制结果：

```sql
shell> mysqldump --databases employees --tables employees --databases employees --tables employees  --where="hire_date >= '2000-01-01' LIMIT 10" > employees_after_2000_limit_10.sql
```

# 从远程服务器备份

有时，您可能无法访问数据库服务器的 SSH（例如云实例，如 Amazon RDS）。在这种情况下，您可以使用`mysqldump`从远程服务器备份到本地服务器。为此，您需要使用`--hostname`选项提到`hostname`。确保用户具有适当的权限以连接和执行备份：

```sql
shell> mysqldump --all-databases --routines --events --triggers --hostname <remote_hostname> > dump.sql
```

# 备份以重建具有不同模式的另一个服务器

可能会出现这样的情况，您希望在另一台服务器上具有不同的模式。在这种情况下，您必须转储和还原模式，根据需要更改模式，然后转储和还原数据。根据您拥有的数据量，更改带有数据的模式可能需要很长时间。请注意，此方法仅在修改后的模式与插入兼容时才有效。修改后的表可以有额外的列，但应该具有原始表中的所有列。

# 仅模式，无数据

您可以使用`--no-data`仅转储模式：

```sql
shell> mysqldump --all-databases --routines --events --triggers --no-data > schema.sql
```

# 仅数据，无模式

您可以使用以下选项仅获取数据转储，而不包括模式。

`--complete-insert`将在`INSERT`语句中打印列名，这将在修改后的表中有额外列时有所帮助：

```sql
shell> mysqldump --all-databases --no-create-db --no-create-info --complete-insert > data.sql
```

# 备份以与其他服务器合并数据

您可以以任何一种方式备份以替换旧数据或在冲突发生时保留旧数据。

# 使用新数据替换

假设您希望将生产数据库中的数据还原到已经存在一些数据的开发机器。如果要将生产中的数据与开发中的数据合并，可以使用`--replace`选项，该选项将使用`REPLACE INTO`语句而不是`INSERT`语句。您还应该包括`--skip-add-drop-table`选项，该选项不会将`DROP TABLE`语句写入`dump`文件。如果表的数量和结构相同，还可以包括`--no-create-info`选项，该选项将跳过`dump`文件中的`CREATE TABLE`语句：

```sql
shell> mysqldump --databases employees --skip-add-drop-table --no-create-info --replace > to_development.sql
```

如果生产环境中有一些额外的表，那么在还原时上述转储将失败，因为开发服务器上不存在该表。在这种情况下，您不应添加`--no-create-info`选项，并在还原时使用`force`选项。否则，还原将在`CREATE TABLE`时失败，说表已经存在。不幸的是，`mysqldump`没有提供`CREATE TABLE IF NOT EXISTS`选项。

# 忽略数据

您可以在写入`dump`文件时使用`INSERT IGNORE`语句代替`REPLACE`。这将保留服务器上的现有数据并插入新数据。

# 使用 mysqlpump 进行备份

`mysqlpump`是一个与`mysqldump`非常相似的程序，具有一些额外的功能。

# 如何做...

有很多方法可以做到这一点。让我们详细看看每种方法。

# 并行处理

通过指定线程数（基于 CPU 数量）可以加快转储过程。例如，使用八个线程进行完整备份：

```sql
shell> mysqlpump --default-parallelism=8 > full_backup.sql
```

您甚至可以为每个数据库指定线程数。在我们的情况下，`employees`数据库与`company`数据库相比非常大。因此，您可以为`employees`生成四个线程，并为`company`数据库生成两个线程：

```sql
shell> mysqlpump -u root --password --parallel-schemas=4:employees --default-parallelism=2 > full_backup.sql
Dump progress: 0/6 tables, 250/331145 rows
Dump progress: 0/34 tables, 494484/3954504 rows
Dump progress: 0/42 tables, 1035414/3954504 rows
Dump progress: 0/45 tables, 1586055/3958016 rows
Dump progress: 0/45 tables, 2208364/3958016 rows
Dump progress: 0/45 tables, 2846864/3958016 rows
Dump progress: 0/45 tables, 3594614/3958016 rows
Dump completed in 6957
```

另一个例子是将线程分配给`db1`和`db2`的三个线程，`db3`和`db4`的两个线程，以及其余数据库的四个线程：

```sql
shell> mysqlpump --parallel-schemas=3:db1,db2 --parallel-schemas=2:db3,db4 --default-parallelism=4 > full_backup.sql
```

您会注意到有一个进度条，可以帮助您估计时间。

# 使用正则表达式排除/包含数据库对象

备份以`prod`结尾的所有数据库：

```sql
shell> mysqlpump --include-databases=%prod --result-file=db_prod.sql
```

假设某些数据库中有一些测试表，您希望将它们从备份中排除；您可以使用`--exclude-tables`选项指定，该选项将在所有数据库中排除名称为`test`的表：

```sql
shell> mysqlpump --exclude-tables=test --result-file=backup_excluding_test.sql
```

每个包含和排除选项的值都是适当对象类型的名称的逗号分隔列表。通配符字符允许在对象名称中使用：

+   `%`匹配零个或多个字符的任何序列

+   `_`匹配任何单个字符

除了数据库和表之外，您还可以包括或排除触发器、例程、事件和用户，例如，`--include-routines`、`--include-events`和`--exclude-triggers`。

要了解更多关于包含和排除选项的信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/mysqlpump.html#mysqlpump-filtering`](https://dev.mysql.com/doc/refman/8.0/en/mysqlpump.html#mysqlpump-filtering)。

# 备份用户

在`mysqldump`中，您将不会在`CREATE USER`或`GRANT`语句中获得用户的备份；相反，您必须备份`mysql.user`表。使用`mysqlpump`，您可以将用户帐户作为帐户管理语句（`CREATE USER`和`GRANT`）而不是插入到`mysql`系统数据库中：

```sql
shell> mysqlpump --exclude-databases=% --users > users_backup.sql
```

您还可以通过指定`--exclude-users`选项来排除一些用户：

```sql
shell> mysqlpump --exclude-databases=% --exclude-users=root --users > users_backup.sql
```

# 压缩备份

您可以压缩备份以减少磁盘空间和网络带宽。您可以使用`--compress-output=lz4`或`--compress-output=zlib`。

请注意，您应该具有适当的解压缩实用程序：

```sql
shell> mysqlpump -u root -pxxxx --compress-output=lz4 > dump.lz4
```

要解压缩，请执行此操作：

```sql
shell> lz4_decompress dump.lz4 dump.sql
```

使用`zlib`执行此操作：

```sql
shell> mysqlpump -u root -pxxxx --compress-output=zlib > dump.zlib
```

要解压缩，请执行此操作：

```sql
shell> zlib_decompress dump.zlib dump.sql
```

# 更快的重新加载

您会注意到在输出中，从`CREATE TABLE`语句中省略了次要索引。这将加快恢复过程。索引将使用`ALTER TABLE`语句在`INSERT`的末尾添加。

索引将在第十三章，*性能调整*中进行讨论。

以前，可以在`mysql`系统数据库中转储所有表。从 MySQL 8 开始，`mysqldump`和`mysqlpump`仅转储该数据库中的非“数据字典”表。

# 使用 mydumper 进行备份

`mydumper`是一个类似于`mysqlpump`的逻辑备份工具。

`mydumper`相对于`mysqldump`具有以下优点：

+   并行性（因此，速度）和性能（避免昂贵的字符集转换例程，并且整体代码效率高）。

+   一致性。它在所有线程中保持快照，提供准确的主从日志位置等。 `mysqlpump`不能保证一致性。

+   更容易管理输出（为表和转储的元数据分别创建文件，并且很容易查看/解析数据）。 `mysqlpump`将所有内容写入一个文件，这限制了加载选择性数据库对象的选项。

+   使用正则表达式包含和排除数据库对象。

+   终止长时间运行的事务以阻止备份和所有后续查询的选项。

`mydumper`是一个开源备份工具，您需要单独安装。本节将介绍 Debian 和 Red Hat 系统上的安装步骤以及`mydumper`的使用。

# 如何做...

让我们从安装开始，然后我们将学习与备份相关的许多事项，这些事项在本食谱中列出的每个子部分中都有。

# 安装

安装先决条件：

在 Ubuntu/Debain 上：

```sql
shell> sudo apt-get install libglib2.0-dev libmysqlclient-dev zlib1g-dev libpcre3-dev cmake git
```

在 Red Hat/CentOS/Fedora 上：

```sql
shell> yum install glib2-devel mysql-devel zlib-devel pcre-devel cmake gcc-c++ git
shell> cd /opt
shell> git clone https://github.com/maxbube/mydumper.git
shell> cd mydumper
shell> cmake .

shell> make
Scanning dependencies of target mydumper
[ 25%] Building C object CMakeFiles/mydumper.dir/mydumper.c.o
[ 50%] Building C object CMakeFiles/mydumper.dir/server_detect.c.o
[ 75%] Building C object CMakeFiles/mydumper.dir/g_unix_signal.c.o

shell> make install
[ 75%] Built target mydumper
[100%] Built target myloader
Linking C executable CMakeFiles/CMakeRelink.dir/mydumper
Linking C executable CMakeFiles/CMakeRelink.dir/myloader
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/mydumper
-- Installing: /usr/local/bin/myloader
```

或者，您可以使用 YUM 或 APT，在此处找到发布版本：[`github.com/maxbube/mydumper/releases`](https://github.com/maxbube/mydumper/releases)

```sql
#YUM
shell> sudo yum install -y "https://github.com/maxbube/mydumper/releases/download/v0.9.3/mydumper-0.9.3-41.el7.x86_64.rpm"

#APT
shell> wget "https://github.com/maxbube/mydumper/releases/download/v0.9.3/mydumper_0.9.3-41.jessie_amd64.deb"

shell> sudo dpkg -i mydumper_0.9.3-41.jessie_amd64.deb
shell> sudo apt-get install -f
```

# 完整备份

以下命令将所有数据库备份到`/backups`文件夹中：

```sql
shell> mydumper -u root --password=<password> --outputdir /backups
```

在`/backups`文件夹中创建了多个文件。每个数据库都有其`CREATE DATABASE`语句，格式为`<database_name>-schema-create.sql`，每个表都有自己的模式和数据文件。模式文件存储为`<database_name>.<table>-schema.sql`，数据文件存储为`<database_name>.<table>.sql`。

视图存储为`<database_name>.<table>-schema-view.sql`。存储的例程，触发器和事件存储为`<database_name>-schema-post.sql`（如果目录未创建，请使用`sudo mkdir –pv /backups`）：

```sql
shell> ls -lhtr /backups/company*
-rw-r--r-- 1 root root 69 Aug 13 10:11 /backups/company-schema-create.sql
-rw-r--r-- 1 root root 180 Aug 13 10:11 /backups/company.payments.sql
-rw-r--r-- 1 root root 239 Aug 13 10:11 /backups/company.new_customers.sql
-rw-r--r-- 1 root root 238 Aug 13 10:11 /backups/company.payments-schema.sql
-rw-r--r-- 1 root root 303 Aug 13 10:11 /backups/company.new_customers-schema.sql
-rw-r--r-- 1 root root 324 Aug 13 10:11 /backups/company.customers-schema.sql
```

如果有任何超过 60 秒的查询，`mydumper`将以以下错误失败：

```sql
** (mydumper:18754): CRITICAL **: There are queries in PROCESSLIST running longer than 60s, aborting dump,
 use --long-query-guard to change the guard value, kill queries (--kill-long-queries) or use  different server for dump

```

为了避免这种情况，您可以传递`--kill-long-queries`选项或将`--long-query-guard`设置为更高的值。

`--kill-long-queries`选项会杀死所有大于 60 秒或由`--long-query-guard`设置的值的查询。请注意，由于错误（[`bugs.launchpad.net/mydumper/+bug/1713201`](https://bugs.launchpad.net/mydumper/+bug/1713201)），`--kill-long-queries`也会杀死复制线程：

```sql
shell> sudo mydumper --kill-long-queries --outputdir /backups** (mydumper:18915): WARNING **: Using trx_consistency_only, binlog coordinates will not be accurate if you are writing to non transactional tables.
** (mydumper:18915): WARNING **: Killed a query that was running for 368s
```

# 一致备份

备份目录中的元数据文件包含一致备份的二进制日志坐标。

在主服务器上，它捕获二进制日志位置：

```sql
shell> sudo cat /backups/metadata 
Started dump at: 2017-08-20 12:44:09
SHOW MASTER STATUS:
    Log: server1.000008
    Pos: 154
    GTID:
```

在从服务器上，它捕获主服务器和从服务器的二进制日志位置：

```sql
shell> cat /backups/metadataStarted dump at: 2017-08-26 06:26:19
SHOW MASTER STATUS:
 Log: server1.000012
 Pos: 154
 GTID:
SHOW SLAVE STATUS:
 Host: 35.186.158.188
 Log: master-bin.000013
```

```sql
 Pos: 4633
 GTID:
Finished dump at: 2017-08-26 06:26:24
```

# 单表备份

以下命令将`employees`数据库的`employees`表备份到`/backups`目录中：

```sql
shell> mydumper -u root --password=<password> -B employees -T employees --triggers --events --routines  --outputdir /backups/employee_table
```

```sql
shell> ls -lhtr /backups/employee_table/
total 17M
-rw-r--r-- 1 root root 71 Aug 13 10:35 employees-schema-create.sql
-rw-r--r-- 1 root root 397 Aug 13 10:35 employees.employees-schema.sql
-rw-r--r-- 1 root root 3.4K Aug 13 10:35 employees-schema-post.sql
-rw-r--r-- 1 root root 75 Aug 13 10:35 metadata
-rw-r--r-- 1 root root 17M Aug 13 10:35 employees.employees.sql
```

文件的约定如下：

+   `employees-schema-create.sql`包含`CREATE DATABASE`语句

+   `employees.employees-schema.sql`包含`CREATE TABLE`语句

+   `employees-schema-post.sql`包含`ROUTINES`，`TRIGGERS`和`EVENTS`

+   `employees.employees.sql`包含`INSERT`语句形式的实际数据

# 使用正则表达式备份特定数据库

您可以使用`regex`选项包括/排除特定数据库。以下命令将从备份中排除`mysql`和`test`数据库：

```sql
shell> mydumper -u root --password=<password> --regex '^(?!(mysql|test))' --outputdir /backups/specific_dbs
```

# 使用 mydumper 备份大表

为了加快大表的转储和恢复速度，您可以将其分成小块。块大小可以通过它包含的行数来指定，每个块将被写入单独的文件中：

```sql
shell> mydumper -u root --password=<password> -B employees -T employees --triggers --events --routines --rows=10000 -t 8 --trx-consistency-only --outputdir /backups/employee_table_chunks
```

+   `-t`：指定线程数

+   `--trx-consistency-only`：如果只使用事务表，例如`InnoDB`，使用此选项将最小化锁定

+   `--rows`：将表拆分为此行数的块

对于每个块，将创建一个文件，格式为`<database_name>.<table_name>.<number>.sql`；数字用五个零填充：

```sql
shell> ls -lhr /backups/employee_table_chunks
total 17M
-rw-r--r-- 1 root root 71 Aug 13 10:45 employees-schema-create.sql
-rw-r--r-- 1 root root 75 Aug 13 10:45 metadata
-rw-r--r-- 1 root root 397 Aug 13 10:45 employees.employees-schema.sql
-rw-r--r-- 1 root root 3.4K Aug 13 10:45 employees-schema-post.sql
-rw-r--r-- 1 root root 633K Aug 13 10:45 employees.employees.00008.sql
-rw-r--r-- 1 root root 634K Aug 13 10:45 employees.employees.00002.sql
-rw-r--r-- 1 root root 1.3M Aug 13 10:45 employees.employees.00006.sql
-rw-r--r-- 1 root root 1.9M Aug 13 10:45 employees.employees.00004.sql
-rw-r--r-- 1 root root 2.5M Aug 13 10:45 employees.employees.00000.sql
-rw-r--r-- 1 root root 2.5M Aug 13 10:45 employees.employees.00001.sql
-rw-r--r-- 1 root root 2.6M Aug 13 10:45 employees.employees.00005.sql
-rw-r--r-- 1 root root 2.6M Aug 13 10:45 employees.employees.00009.sql
-rw-r--r-- 1 root root 2.6M Aug 13 10:45 employees.employees.00010.sql
```

# 非阻塞备份

为了提供一致的备份，`mydumper`通过执行`FLUSH TABLES WITH READ LOCK`获取`GLOBAL LOCK`。

如果有任何长时间运行的事务（在第五章“事务”中解释）使用`FLUSH TABLES WITH READ LOCK`是多么危险。为了避免这种情况，您可以传递`--kill-long-queries`选项来杀死阻塞查询，而不是中止`mydumper`。

+   `--trx-consistency-only`：这相当于`mysqldump`的`--single-transaction`，但带有`binlog`位置。显然，此位置仅适用于事务表。使用此选项的优点之一是全局读锁仅用于线程协调，因此一旦事务开始，它就会被释放。

+   --use-savepoints 减少元数据锁定问题（需要`SUPER`权限）。

# 压缩备份

您可以指定`--compress`选项来压缩备份：

```sql
shell> mydumper -u root --password=<password> -B employees -T employees -t 8 --trx-consistency-only --compress --outputdir /backups/employees_compress
```

```sql
shell> ls -lhtr /backups/employees_compress
total 5.3M
-rw-r--r-- 1 root root 91 Aug 13 11:01 employees-schema-create.sql.gz
-rw-r--r-- 1 root root 263 Aug 13 11:01 employees.employees-schema.sql.gz
-rw-r--r-- 1 root root 75 Aug 13 11:01 metadata
-rw-r--r-- 1 root root 5.3M Aug 13 11:01 employees.employees.sql.gz
```

# 仅备份数据

您可以使用`--no-schemas`选项跳过模式并进行仅数据备份：

```sql
shell> mydumper -u root --password=<password> -B employees -T employees -t 8 --no-schemas --compress --trx-consistency-only --outputdir /backups/employees_data
```

# 使用平面文件进行备份

这是一种物理备份方法，通过直接复制`data directory`中的文件来进行备份。由于在复制文件时会写入新数据，因此备份将是不一致的，无法使用。为了避免这种情况，您必须关闭 MySQL，复制文件，然后启动 MySQL。这种方法不适用于日常备份，但在维护窗口期间进行升级、降级或进行主机交换时非常合适。

# 如何做...

1.  关闭 MySQL 服务器：

```sql
shell> sudo service mysqld stop
```

1.  将文件复制到`data directory`（您的目录可能不同）：

```sql
shell> sudo rsync -av /data/mysql /backups
or do rsync over ssh to remote server
shell> rsync -e ssh -az /data/mysql/ backup_user@remote_server:/backups
```

1.  启动 MySQL 服务器：

```sql
shell> sudo service mysqld start
```

# 使用 XtraBackup 进行备份

XtraBackup 是由 Percona 提供的开源备份软件。它在不关闭服务器的情况下复制平面文件，但为了避免不一致性，它使用重做日志文件。许多公司将其作为标准备份工具广泛使用。其优点是与逻辑备份工具相比非常快，恢复速度也非常快。

这是 Percona XtraBackup 的工作原理（摘自 Percona XtraBackup 文档）：

1.  它会复制您的`InnoDB`数据文件，这将导致数据在内部不一致；然后它会对文件执行崩溃恢复，使它们成为一致的可用数据库。

1.  这是因为`InnoDB`维护着一个重做日志，也称为事务日志。这包含了对`InnoDB`数据的每一次更改的记录。当`InnoDB`启动时，它会检查数据文件和事务日志，并执行两个步骤。它将已提交的事务日志条目应用于数据文件，并对修改数据但未提交的任何事务执行撤消操作。

1.  Percona XtraBackup 在启动时通过记住**日志序列号**（**LSN**）来工作，然后复制数据文件。这需要一些时间，因此如果文件在更改，则它们反映了数据库在不同时间点的状态。同时，Percona XtraBackup 运行一个后台进程，监视事务日志文件，并从中复制更改。Percona XtraBackup 需要不断执行此操作，因为事务日志是以循环方式写入的，并且一段时间后可以被重用。自执行以来，Percona XtraBackup 需要事务日志记录，以获取自开始执行以来对数据文件的每次更改。

# 如何做...

在撰写本文时，Percona XtraBackup 不支持 MySQL 8。最终，Percona 将发布支持 MySQL 8 的新版本 XtraBackup；因此只涵盖了安装部分。

# 安装

安装步骤在以下部分中。 

# 在 CentOS/Red Hat/Fedora 上

1.  安装`mysql-community-libs-compat`：

```sql
shell> sudo yum install -y mysql-community-libs-compat
```

1.  安装 Percona 存储库：

```sql
shell> sudo yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm
```

您应该看到以下输出：

```sql
Retrieving http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm
Preparing...                ########################################### [100%]
   1:percona-release        ########################################### [100%]
```

1.  测试存储库：

```sql
shell> yum list | grep xtrabackup
holland-xtrabackup.noarch 1.0.14-3.el7 epel 
percona-xtrabackup.x86_64 2.3.9-1.el7 percona-release-x86_64
percona-xtrabackup-22.x86_64 2.2.13-1.el7 percona-release-x86_64
percona-xtrabackup-22-debuginfo.x86_64 2.2.13-1.el7 percona-release-x86_64
percona-xtrabackup-24.x86_64 2.4.8-1.el7 percona-release-x86_64
percona-xtrabackup-24-debuginfo.x86_64 2.4.8-1.el7 percona-release-x86_64
percona-xtrabackup-debuginfo.x86_64 2.3.9-1.el7 percona-release-x86_64
percona-xtrabackup-test.x86_64 2.3.9-1.el7 percona-release-x86_64
percona-xtrabackup-test-22.x86_64 2.2.13-1.el7 percona-release-x86_64
percona-xtrabackup-test-24.x86_64 2.4.8-1.el7 percona-release-x86_64
```

1.  安装 XtraBackup：

```sql
shell> sudo yum install percona-xtrabackup-24
```

# 在 Debian/Ubuntu 上

1.  从 Percona 获取存储库软件包：

```sql
shell> wget https://repo.percona.com/apt/percona-release_0.1-4.$(lsb_release -sc)_all.deb
```

1.  使用`dpkg`安装下载的软件包。为此，请以`root`或`sudo`身份运行以下命令：

```sql
shell> sudo dpkg -i percona-release_0.1-4.$(lsb_release -sc)_all.deb
```

安装此软件包后，应该添加 Percona 存储库。您可以在`/etc/apt/sources.list.d/percona-release.list`文件中检查存储库设置。

1.  记得更新本地缓存：

```sql
shell> sudo apt-get update
```

1.  之后，您可以安装软件包：

```sql
shell> sudo apt-get install percona-xtrabackup-24
```

# 锁定备份实例

从 MySQL 8 开始，您可以锁定实例以进行备份，这将允许在线备份期间进行 DML，并阻止可能导致不一致快照的所有操作。

# 如何做...

在开始备份之前，请锁定实例以进行备份：

```sql
mysql> LOCK INSTANCE FOR BACKUP;
```

执行备份，完成后解锁实例：

```sql
mysql> UNLOCK INSTANCE;
```

# 二进制日志备份

您知道二进制日志对于时点恢复是必需的。在本节中，您将了解如何备份二进制日志。该过程将二进制日志从数据库服务器流式传输到远程备份服务器。您可以从从服务器或主服务器中获取二进制日志备份。如果您从主服务器获取二进制日志备份，并且从从服务器获取实际备份，您应该使用`--dump-slave`来获取相应的主日志位置。如果您使用`mydumper`或 XtraBackup，它会提供主服务器和从服务器的二进制日志位置。

# 如何做到这一点...

1.  在服务器上创建一个复制用户。创建一个强密码：

```sql
mysql> GRANT REPLICATION SLAVE ON *.* TO 'binlog_user'@'%' IDENTIFIED BY 'binlog_pass';Query OK, 0 rows affected, 1 warning (0.03 sec)
```

1.  检查服务器上的二进制日志：

```sql
mysql> SHOW BINARY LOGS;+----------------+-----------+
| Log_name       | File_size |
+----------------+-----------+
| server1.000008 |      2451 |
| server1.000009 |       199 |
| server1.000010 |      1120 |
| server1.000011 |       471 |
| server1.000012 |       154 |
+----------------+-----------+
5 rows in set (0.00 sec)
```

您可以在服务器上找到第一个可用的二进制日志；从这里，您可以开始备份。在这种情况下，它是`server1.000008`。

1.  登录到备份服务器并执行以下命令。这将从 MySQL 服务器复制二进制日志到备份服务器。您可以开始使用`nohup`或`disown`：

```sql
shell> mysqlbinlog -u <user> -p<pass> -h <server> --read-from-remote-server --stop-never 
--to-last-log --raw server1.000008 &
shell> disown -a
```

1.  验证二进制日志是否已备份：

```sql
shell> ls -lhtr server1.0000*-rw-r-----. 1 mysql mysql 2.4K Aug 25 12:22 server1.000008
-rw-r-----. 1 mysql mysql  199 Aug 25 12:22 server1.000009
-rw-r-----. 1 mysql mysql 1.1K Aug 25 12:22 server1.000010
-rw-r-----. 1 mysql mysql  471 Aug 25 12:22 server1.000011
-rw-r-----. 1 mysql mysql  154 Aug 25 12:22 server1.000012 
```
