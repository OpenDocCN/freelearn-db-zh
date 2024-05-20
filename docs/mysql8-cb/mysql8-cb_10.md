# 第十章：表维护

在本章中，我们将涵盖以下配方：

+   安装 Percona Toolkit

+   修改表

+   在数据库之间移动表

+   使用在线模式更改工具修改表

+   归档表

+   克隆表

+   分区表

+   分区修剪和选择

+   分区管理

+   分区信息

+   高效管理生存时间和软删除行

# 介绍

在维护数据库中，一个关键方面是管理表。通常，您需要更改一个大表或克隆一个表。在本章中，您将学习如何管理大表。由于 MySQL 不支持某些操作，因此使用了一些开源第三方工具。本章还涵盖了第三方工具的安装和使用。 

# 安装 Percona Toolkit

Percona Toolkit 是一套高级开源命令行工具，由 Percona 开发和使用，用于执行各种手动执行的任务。安装在本节中介绍。在后面的部分，您将学习如何使用它。

# 如何做...

让我们看看如何在各种操作系统上安装 Percona Toolkit。

# 在 Debian/Ubuntu 上

1.  下载存储库软件包：

```sql
shell> wget https://repo.percona.com/apt/percona-release_0.1-4.$(lsb_release -sc)_all.deb
```

1.  安装存储库软件包：

```sql
shell> sudo dpkg -i percona-release_0.1-4.$(lsb_release -sc)_all.deb
```

1.  更新本地软件包列表：

```sql
shell> sudo apt-get update
```

1.  确保 Percona 软件包可用：

```sql
shell> apt-cache search percona
```

您应该看到类似以下的输出：

```sql
percona-xtrabackup-dbg - Debug symbols for Percona XtraBackup
percona-xtrabackup-test - Test suite for Percona XtraBackup
percona-xtradb-cluster-client - Percona XtraDB Cluster database client
percona-xtradb-cluster-server - Percona XtraDB Cluster database server
percona-xtradb-cluster-testsuite - Percona XtraDB Cluster database regression test suite
percona-xtradb-cluster-testsuite-5.5 - Percona Server database test suite
...
```

1.  安装`percona-toolkit`软件包：

```sql
shell> sudo apt-get install percona-toolkit
```

如果您不想安装存储库，也可以直接安装：

```sql
shell> wget https://www.percona.com/downloads/percona-toolkit/3.0.4/binary/debian/xenial/x86_64/percona-toolkit_3.0.4-1.xenial_amd64.deb
```

```sql
shell> sudo dpkg -i percona-toolkit_3.0.4-1.yakkety_amd64.deb;
shell> sudo apt-get install -f
```

# 在 CentOS/Red Hat/Fedora 上

1.  安装存储库软件包：

```sql
shell> sudo yum install http://www.percona.com/downloads/percona-release/redhat/0.1-4/percona-release-0.1-4.noarch.rpm
```

如果成功，您应该看到以下内容：

```sql
Installed:
  percona-release.noarch 0:0.1-4

Complete!
```

1.  确保 Percona 软件包可用：

```sql
shell> sudo yum list | grep percona
```

您应该看到类似以下的输出：

```sql
percona-release.noarch                     0.1-4                       @/percona-release-0.1-4.noarch
Percona-Server-55-debuginfo.x86_64         5.5.54-rel38.7.el7          percona-release-x86_64
Percona-Server-56-debuginfo.x86_64         5.6.35-rel81.0.el7          percona-release-x86_64
Percona-Server-57-debuginfo.x86_64         5.7.17-13.1.el7             percona-release-x86_64
...
```

1.  安装 Percona Toolkit：

```sql
shell> sudo yum install percona-toolkit
```

如果您不想安装存储库，可以直接使用 YUM 安装：

```sql
shell> sudo yum install https://www.percona.com/downloads/percona-toolkit/3.0.4/binary/redhat/7/x86_64/percona-toolkit-3.0.4-1.el7.x86_64.rpm
```

# 修改表

`ALTER TABLE`更改表的结构。例如，您可以添加或删除列，创建或销毁索引，更改现有列的类型，或重命名列或表本身。

在执行某些 alter 操作（例如更改列数据类型，添加`SPATIAL INDEX`，删除主键，转换字符集，添加/删除加密等）时，表上的 DML 操作将被阻止。如果表很大，则更改需要更长的时间，并且应用程序在此期间无法访问表，这是不希望发生的。在这种情况下，`pt-online-schema`更改是有帮助的，其中允许 DML 语句。

有两种 alter 操作算法：

+   **原地**（默认）：不需要复制整个表数据

+   **复制**：将数据复制到临时磁盘文件并重命名

只有某些 alter 操作可以就地完成。在线 DDL 操作的性能在很大程度上取决于操作是在原地执行还是需要复制和重建整个表。请参阅[`dev.mysql.com/doc/refman/8.0/en/innodb-create-index-overview.html#innodb-online-ddl-summary-grid`](https://dev.mysql.com/doc/refman/8.0/en/innodb-create-index-overview.html#innodb-online-ddl-summary-grid)查看可以就地执行的操作类型，以及避免表复制操作的任何要求。

*复制算法的工作原理*（摘自参考手册-[`dev.mysql.com/doc/refman/8.0/en/alter-table.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html)）

不是*就地*执行的`ALTER TABLE`操作会创建原始表的临时副本。MySQL 等待正在修改表的其他操作，然后继续。它将更改合并到副本中，删除原始表，并重命名新表。在执行`ALTER TABLE`时，原始表可被其他会话读取。在`ALTER TABLE`操作开始后开始的对表的更新和写入将被暂停，直到新表准备就绪，然后会自动重定向到新表，而不会有任何更新失败。原始表的临时副本创建在新表的数据库目录中。这可能与重命名表到不同数据库的`ALTER TABLE`操作的原始表的数据库目录不同。

要了解 DDL 操作是就地执行还是表复制，请查看命令完成后显示的`受影响行数`值：

+   更改列的默认值（超快，根本不影响表数据），输出将类似于这样：

```sql
Query OK, 0 rows affected (0.07 sec)
```

+   添加索引（需要时间，但`0 行受影响`表明表没有被复制），输出将类似于这样：

`查询 OK，0 行受影响（21.42 秒）`

+   更改列的数据类型（需要大量时间，并且确实需要重建表的所有行），输出将类似于这样：

```sql
Query OK, 1671168 rows affected (1 min 35.54 sec)
```

更改列的数据类型需要重建表的所有行，除了更改`VARCHAR`大小之外，可以使用在线`ALTER TABLE`来执行。请参阅*使用在线模式更改工具修改表*部分中提到的示例，该示例显示了如何使用`pt-online-schema`修改列属性。

# 如何做...

如果要向`employees`表添加新列，可以执行`ADD COLUMN`语句：

```sql
mysql> ALTER TABLE employees ADD COLUMN address varchar(100);
Query OK, 0 rows affected (5.10 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

您会看到受影响的行数为`0`，这意味着表没有被复制，操作是就地完成的。

如果您想增加`varchar`列的长度，可以执行`MODIFY COLUMN`语句：

```sql
mysql> ALTER TABLE employees MODIFY COLUMN address VARCHAR(255);
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

如果您认为`varchar(255)`不足以存储地址，并且想要将其更改为`tinytext`，您可以使用`MODIFY COLUMN`语句。但是，在这种情况下，由于您正在修改列的数据类型，所有现有表的行都应该被修改，这需要表复制，并且会阻塞 DMLs：

```sql
mysql> ALTER TABLE employees MODIFY COLUMN address tinytext;
Query OK, 300025 rows affected (4.36 sec)
Records: 300025  Duplicates: 0  Warnings: 0
```

您会注意到受影响的行数为`300025`，这是表的大小。

还有其他各种操作，例如重命名列，更改默认值，重新排序列位置等；有关更多详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/innodb-create-index-overview.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-create-index-overview.html)的手册。

添加一个虚拟生成的列只是一个元数据更改，几乎是瞬时的：

```sql
mysql> ALTER TABLE employees ADD COLUMN full_name VARCHAR(40) AS (CONCAT('first_name', ' ', 'last_name'));
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

但是，添加`STORED GENERATED`列和修改`VIRTUAL GENERATED`列不是在线的：

```sql
mysql> ALTER TABLE employees MODIFY COLUMN full_name VARCHAR(40) AS (CONCAT(first_name, '-', last_name)) VIRTUAL;
Query OK, 300026 rows affected (4.37 sec)
Records: 300026  Duplicates: 0  Warnings: 0
```

# 移动表格跨数据库

您可以通过执行`RENAME TABLE`语句来重命名表。

为了使以下示例起作用，请创建示例表和数据库

```sql
mysql> CREATE DATABASE prod;
mysql> CREATE TABLE prod.audit_log (id int NOT NULL, msg varchar(64));
mysql> CREATE DATABASE archive;
```

# 如何做...

例如，如果要将`audit_log`表重命名为`audit_log_archive_2018`，可以执行以下操作：

```sql
mysql> USE prod;
Database changed

mysql> RENAME TABLE audit_log TO audit_log_archive_2018;
Query OK, 0 rows affected (0.07 sec)
```

如果要将表从一个数据库移动到另一个数据库，可以使用点表示法指定数据库名称。例如，如果要将名为`audit_log`的表从名为`prod`的数据库移动到名为`archive`的数据库，执行以下操作：

```sql
mysql> USE prod
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

mysql> SHOW TABLES;
+------------------------+
| Tables_in_prod         |
+------------------------+
| audit_log_archive_2018 |
+------------------------+
1 row in set (0.00 sec)

mysql> RENAME TABLE audit_log_archive_2018 TO archive.audit_log;
Query OK, 0 rows affected (0.03 sec)

mysql> SHOW TABLES;
Empty set (0.00 sec)

mysql> USE archive
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SHOW TABLES;
+-------------------+
| Tables_in_archive |
+-------------------+
| audit_log         |
+-------------------+
1 row in set (0.00 sec)
```

# 使用在线模式更改工具修改表

在本节中，您将了解 Percona 的`pt-online-schema-change`（`pt-osc`）工具，该工具用于执行`ALTER TABLE`操作而不会阻塞 DMLs。

`pt-osc`与 Percona Toolkit 一起提供。Percona Toolkit 的安装已在本章前面进行了描述。

# 它是如何工作的...

（摘自[`www.percona.com/doc/percona-toolkit/LATEST/pt-online-schema-change.html`](https://www.percona.com/doc/percona-toolkit/LATEST/pt-online-schema-change.html)。）

`pt-online-schema-change`的工作原理是创建要更改的表的空副本，根据需要对其进行修改，然后将原始表中的行复制到新表中。复制完成后，它会将原始表移开，并用新表替换。默认情况下，它还会删除原始表。

数据复制过程是以小数据块的形式执行的，这些数据块的大小是不同的，以尝试使它们在特定的时间内执行。在复制过程中对原始表中的数据进行的任何修改都将反映在新表中，因为该工具会在原始表上创建触发器，以更新新表中的相应行。使用触发器意味着如果表上已经定义了任何触发器，该工具将无法工作。

当工具完成将数据复制到新表中后，它会使用原子`RENAME TABLE`操作同时重命名原始表和新表。完成此操作后，工具会删除原始表。

外键会使工具的操作复杂化并引入额外的风险。在外键引用表时，原子重命名原始表和新表的技术无法正常工作。工具必须在模式更改完成后更新外键以引用新表。该工具支持两种方法来实现这一点。您可以在`--alter-foreign-keys-method`的文档中了解更多信息。

# 如何做...

修改列数据类型的方法如下：

```sql
shell> pt-online-schema-change D=employees,t=employees,h=localhost -u root --ask-pass --alter="MODIFY COLUMN address VARCHAR(100)" --alter-foreign-keys-method=auto --execute
Enter MySQL password: 
No slaves found.  See --recursion-method if host server1 has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.
Operation, tries, wait:
  analyze_table, 10, 1
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Child tables:
  `employees`.`dept_emp` (approx. 331143 rows)
  `employees`.`titles` (approx. 442605 rows)
  `employees`.`salaries` (approx. 2838426 rows)
  `employees`.`dept_manager` (approx. 24 rows)
Will automatically choose the method to update foreign keys.
Altering `employees`.`employees`...
Creating new table...
Created new table employees._employees_new OK.
Altering new table...
Altered `employees`.`_employees_new` OK.
2017-09-24T09:56:49 Creating triggers...
2017-09-24T09:56:49 Created triggers OK.
2017-09-24T09:56:49 Copying approximately 299478 rows...
2017-09-24T09:56:56 Copied rows OK.
2017-09-24T09:56:56 Max rows for the rebuild_constraints method: 88074
Determining the method to update foreign keys...
2017-09-24T09:56:56   `employees`.`dept_emp`: too many rows: 331143; must use drop_swap
2017-09-24T09:56:56 Drop-swapping tables...
2017-09-24T09:56:56 Analyzing new table...
2017-09-24T09:56:56 Dropped and swapped tables OK.
Not dropping old table because --no-drop-old-table was specified.
2017-09-24T09:56:56 Dropping triggers...
2017-09-24T09:56:56 Dropped triggers OK.
Successfully altered `employees`.`employees`.
```

您会注意到该工具已经创建了一个具有修改结构的新表，为该表创建了触发器，将行复制到新表中，最后重命名了新表。

如果要更改已经具有触发器的`salaries`表，您需要指定`--preserver-triggers`选项，否则将出现错误：`The table `employees`.`salaries` has triggers but --preserve-triggers was not specified.`：

```sql
shell> pt-online-schema-change D=employees,t=salaries,h=localhost -u user --ask-pass --alter="MODIFY COLUMN salary int" --alter-foreign-keys-method=auto --execute --no-drop-old-table --preserve-triggers 
No slaves found.  See --recursion-method if host server1 has slaves.
Not checking slave lag because no slaves were found and --check-slave-lag was not specified.

Operation, tries, wait:
  analyze_table, 10, 1
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
No foreign keys reference `employees`.`salaries`; ignoring --alter-foreign-keys-method.
Altering `employees`.`salaries`...
Creating new table...
Created new table employees._salaries_new OK.
Altering new table...
Altered `employees`.`_salaries_new` OK.
2017-09-24T11:11:58 Creating triggers...
2017-09-24T11:11:58 Created triggers OK.
2017-09-24T11:11:58 Copying approximately 2838045 rows...
2017-09-24T11:12:20 Copied rows OK.
2017-09-24T11:12:20 Adding original triggers to new table.
2017-09-24T11:12:21 Analyzing new table...
2017-09-24T11:12:21 Swapping tables...
2017-09-24T11:12:21 Swapped original and new tables OK.
Not dropping old table because --no-drop-old-table was specified.
2017-09-24T11:12:21 Dropping triggers...
2017-09-24T11:12:21 Dropped triggers OK.
Successfully altered `employees`.`salaries`
```

如果服务器有从属服务器，该工具在从现有表复制到新表时可能会创建从属延迟。为了避免这种情况，您可以指定`--check-slave-lag`（默认启用）；它会暂停数据复制，直到此副本的延迟小于`--max-lag`，默认为 1 秒。您可以通过传递`--max-lag`选项来指定`--max-lag`。

如果要确保从属服务器的延迟不会超过 10 秒，请传递`--max-lag=10`：

```sql
shell> pt-online-schema-change D=employees,t=employees,h=localhost -u user --ask-pass --alter="MODIFY COLUMN address VARCHAR(100)" --alter-foreign-keys-method=auto --execute --preserve-triggers --max-lag=10
Enter MySQL password: 
Found 1 slaves:
server2 -> xx.xxx.xxx.xx:socket
Will check slave lag on:
server2 -> xx.xxx.xxx.xx:socket
Operation, tries, wait:
  analyze_table, 10, 1
  copy_rows, 10, 0.25
  create_triggers, 10, 1
  drop_triggers, 10, 1
  swap_tables, 10, 1
  update_foreign_keys, 10, 1
Child tables:
  `employees`.`dept_emp` (approx. 331143 rows)
  `employees`.`titles` (approx. 442605 rows)
  `employees`.`salaries` (approx. 2838426 rows)
  `employees`.`dept_manager` (approx. 24 rows)
Will automatically choose the method to update foreign keys.
Altering `employees`.`employees`...
Creating new table...
Created new table employees._employees_new OK.
Waiting forever for new table `employees`.`_employees_new` to replicate to ubuntu...
Altering new table...
Altered `employees`.`_employees_new` OK.
2017-09-24T12:00:58 Creating triggers...
2017-09-24T12:00:58 Created triggers OK.
2017-09-24T12:00:58 Copying approximately 299342 rows...
2017-09-24T12:01:05 Copied rows OK.
2017-09-24T12:01:05 Max rows for the rebuild_constraints method: 86446
Determining the method to update foreign keys...
2017-09-24T12:01:05   `employees`.`dept_emp`: too many rows: 331143; must use drop_swap
2017-09-24T12:01:05 Skipping triggers creation since --no-swap-tables was specified along with --drop-new-table
2017-09-24T12:01:05 Drop-swapping tables...
2017-09-24T12:01:05 Analyzing new table...
2017-09-24T12:01:05 Dropped and swapped tables OK.
Not dropping old table because --no-drop-old-table was specified.
2017-09-24T12:01:05 Dropping triggers...
2017-09-24T12:01:05 Dropped triggers OK.
Successfully altered `employees`.`employees`.
```

有关更多详细信息和选项，请参阅 Percona 文档，网址为[`www.percona.com/doc/percona-toolkit/LATEST/pt-online-schema-change.html`](https://www.percona.com/doc/percona-toolkit/LATEST/pt-online-schema-change.html)。

`pt-online-schema-change`仅在有主键或唯一键时才有效，否则将出现以下错误：

```sql
The new table `employees`.`_employees_new` does not have a PRIMARY KEY or a unique index which is required for the DELETE trigger.
```

因此，如果表没有任何唯一键，您无法使用`pt-online-schema-change`。

# 归档表

有时，您不希望保留旧数据并希望删除它。如果要删除所有上个月访问的行，如果表很小（<10k 行），您可以直接使用以下命令：

```sql
DELETE FROM <TABLE> WHERE last_accessed<DATE_ADD(NOW(), INTERVAL -1 MONTH)
```

如果表很大会发生什么？您知道`InnoDB`会创建一个`UNDO`日志来恢复失败的事务。因此，所有删除的行都保存在`UNDO`日志空间中，以便在`DELETE`语句中止时用于恢复。不幸的是，如果`DELETE`语句在中途中止，`InnoDB`会从`UNDO`日志空间复制行到表中，这可能会导致表无法访问。

为了克服这种行为，您可以`LIMIT`删除的行数并`COMMIT`事务，重复运行相同的操作，直到删除所有不需要的行。

这是一个伪代码示例：

```sql
WHILE count<=0:
    DELETE FROM <TABLE> WHERE last_accessed<DATE_ADD(NOW(), INTERVAL -1 MONTH) LIMIT 10000;
    count=SELECT COUNT(*) FROM <TABLE> WHERE last_accessed<DATE_ADD(NOW(), INTERVAL -1 MONTH);
```

如果`last_accessed`上没有`INDEX`，它可以锁定表。在这种情况下，您需要找出已删除行的主键，并基于`PRIMARY KEY`进行删除。

这是伪代码，假设`id`是`PRIMARY KEY`：

```sql
WHILE count<=0:
    SELECT id FROM <TABLE> WHERE last_accessed < DATE_ADD(NOW(), INTERVAL -1 MONTH) LIMIT 10000;
    DELETE FROM <TABLE> WHERE id IN ('ids from above statement');
    count=SELECT COUNT(*) FROM <TABLE> WHERE     last_accessed<DATE_ADD(NOW(), INTERVAL -1 MONTH);
```

您可以使用 Percona 的`pt-archiver`工具而不是编写删除行的代码，它本质上做的是相同的，并提供许多其他选项，例如将行保存到另一个表或文件中、对负载和复制延迟进行精细控制等。

# 如何做...

`pt-archiver`中有许多选项，我们将从简单的清除开始。

# 清除数据

如果您想要删除`employees`表中`hire_date`早于 30 年的所有行，您可以执行以下操作：

```sql
shell> pt-archiver --source h=localhost,D=employees,t=employees -u <user> -p<pass> --where="hire_date<DATE_ADD(NOW(), INTERVAL -30 YEAR)" --no-check-charset --limit 10000 --commit-each
```

您可以通过`--source`选项传递主机名、数据库名称和表名。您可以使用`--limit`选项批量限制要删除的行数。

如果您指定`--progress`，输出是一个标题行，加上间隔的状态输出。状态输出中的每一行都列出了当前日期和时间、`pt-archiver`运行了多少秒以及它归档了多少行。

如果您指定`--statistics`，`pt-archiver`将输出时间和其他信息，以帮助您确定归档过程中哪一部分花费了最多时间。

如果您指定`--check-slave-lag`，工具将暂停归档，直到从属滞后小于`--max-lag`。

# 归档数据

如果您想要在删除后将行保存到单独的表或文件中，可以指定`--dest`选项。

假设您想要将`employees`数据库的`employees`表的所有行移动到`employees_archive`表中，您可以执行以下操作：

```sql
shell> pt-archiver --source h=localhost,D=employees,t=employees --dest h=localhost,D=employees_archive -u <user> -p<pass> --where="1=1" --no-check-charset --limit 10000 --commit-each
```

如果您指定`--where="1=1"`，它将复制所有行。

# 复制数据

如果您想要从一张表复制数据到另一张表，您可以使用`mysqldump`或`mysqlpump`备份特定行，然后将它们加载到目标表中。作为替代，您也可以使用`pt-archive`。如果您指定`--no-delete`选项，`pt-archiver`将不会从源中删除行。

```sql
shell> pt-archiver --source h=localhost,D=employees,t=employees --dest h=localhost,D=employees_archive -u <user> -p<pass> --where="1=1" --no-check-charset --limit 10000 --commit-each --no-delete
```

# 另请参阅

有关`pt-archiver`的更多详细信息和选项，请参阅[`www.percona.com/doc/percona-toolkit/LATEST/pt-archiver.html`](https://www.percona.com/doc/percona-toolkit/LATEST/pt-archiver.html)。

# 克隆表

如果您想要克隆一个表，有很多选项。

# 如何做...

1.  使用`INSERT INTO SELECT`语句：

```sql
mysql> CREATE TABLE employees_clone LIKE employees;
mysql> INSERT INTO employees_clone SELECT * FROM employees;
```

请注意，如果有任何生成的列，上述语句将不起作用。在这种情况下，您应该提供完整的插入语句，不包括生成的列。

```sql
mysql> INSERT INTO employees_clone SELECT * FROM employees;
ERROR 3105 (HY000): The value specified for generated column 'hire_date_year' in table 'employees_clone' is not allowed.

mysql> INSERT INTO employees_clone(emp_no, birth_date, first_name, last_name, gender, hire_date) SELECT emp_no, birth_date, first_name, last_name, gender, hire_date FROM employees;
Query OK, 300024 rows affected (3.21 sec)
Records: 300024  Duplicates: 0  Warnings: 0
```

但是在大表上，上述语句非常慢且危险。请记住，如果语句失败，为了恢复表状态，`InnoDB`会将所有行保存在`UNDO`日志中。

1.  使用`mysqldump`或`mysqlpump`备份单个表并在目标上恢复。如果表很大，这可能需要很长时间。

1.  使用`Innobackupex`备份特定表并将数据文件恢复到目标上。

1.  使用`pt-archiver`和`--no-delete`选项，它将把所需的行或所有行复制到目标表中。

您还可以使用可传输表空间来克隆表，这在《第十一章》的*管理表空间*部分的*将文件表空间复制到另一个实例*部分中有解释。

# 分区表

您可以使用分区将单个表的部分分布到文件系统中。用户选择的数据划分规则称为分区函数，可以是模数、简单匹配一组范围或值列表、内部哈希函数或线性哈希函数。

表的不同行可以分配到不同的物理分区，这称为水平分区。MySQL 不支持垂直分区，即将表的不同列分配到不同的物理分区。

有许多分区表的方法：

+   `RANGE`：这种类型的分区根据列值是否在给定范围内将行分配到分区中。

+   `LIST`：类似于基于`RANGE`的分区，只是根据与一组离散值匹配的列选择分区。

+   `HASH`：使用这种类型的分区，根据用户定义的表达式返回的值选择分区，该表达式在要插入表中的行的列值上操作。该函数可以由 MySQL 中的任何有效表达式组成，产生非负整数值。

+   `KEY`：这种类型的分区与`HASH`分区类似，只是提供了要评估的一个或多个列，并且 MySQL 服务器提供了自己的哈希函数。这些列可以包含除整数值以外的其他值，因为 MySQL 提供的哈希函数保证了整数结果，无论列数据类型如何。

前面的每种分区类型都有一个扩展。`RANGE`有`RANGE COLUMNS`，`LIST`有`LIST COLUMNS`，`HASH`有`LINEAR HASH`，`KEY`有`LINEAR KEY`。

对于`[LINEAR] KEY`，`RANGE COLUMNS`和`LIST COLUMNS`分区，分区表达式由一个或多个列的列表组成。

在`RANGE`，`LIST`和`[LINEAR] HASH`分区的情况下，分区列的值将传递给分区函数，该函数返回一个整数值，表示应将该特定记录存储在其中的分区的编号。此函数必须是非常数和非随机的。

数据库分区的一个非常常见的用途是按日期对数据进行分隔。

有关分区的优势和其他详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/partitioning-overview.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-overview.html)。

请注意，分区仅适用于`InnoDB`表，并且外键尚不支持与分区一起使用。

# 如何做到...

您可以在创建表时指定分区，也可以通过执行`ALTER TABLE`命令来指定。分区列应该是表中所有唯一键的一部分。

如果您根据`created_at`列定义了分区，并且`id`是主键，那么您应该将`create_at`列包括在`PRIMARY KEY`中，即(`id`，`created_at`)。

以下示例假设没有外键引用到表。

如果您希望在 MySQL 8.0 中基于时间范围或间隔实现分区方案，有两种选择：

+   通过`RANGE`对表进行分区，并且对分区表达式使用在`DATE`，`TIME`或`DATETIME`列上操作并返回整数值的函数。

+   通过`RANGE COLUMNS`对表进行分区，使用`DATE`或`DATETIME`列作为分区列

# 范围分区

如果您想根据`emp_no`对`employees`表进行分区，并且想要在一个分区中保留 100,000 名员工，可以这样创建：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (emp_no)
(PARTITION p0 VALUES LESS THAN (100000) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (200000) ENGINE = InnoDB,
 PARTITION p2 VALUES LESS THAN (300000) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (400000) ENGINE = InnoDB,
 PARTITION p4 VALUES LESS THAN (500000) ENGINE = InnoDB);
```

因此，所有`emp_no`小于 100,000 的员工将进入分区`p0`，所有`emp_no`小于`200000`且大于`100000`的员工将进入分区`p1`，依此类推。

如果员工号大于`500000`，由于没有为它们定义分区，插入将失败并显示错误。为了避免这种情况，您必须定期检查并添加分区，或者创建一个`MAXVALUE`分区来捕获所有这些异常：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (emp_no)
(PARTITION p0 VALUES LESS THAN (100000) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (200000) ENGINE = InnoDB,
 PARTITION p2 VALUES LESS THAN (300000) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (400000) ENGINE = InnoDB,
 PARTITION p4 VALUES LESS THAN (500000) ENGINE = InnoDB,
 PARTITION pmax VALUES LESS THAN MAXVALUE ENGINE = InnoDB
);
```

如果您想基于`hire_date`进行分区，可以使用`YEAR(hire_date)`函数作为分区表达式：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (YEAR(hire_date))
(PARTITION p1980 VALUES LESS THAN (1980) ENGINE = InnoDB,
 PARTITION p1990 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p2000 VALUES LESS THAN (2000) ENGINE = InnoDB,
 PARTITION p2010 VALUES LESS THAN (2010) ENGINE = InnoDB,
 PARTITION p2020 VALUES LESS THAN (2020) ENGINE = InnoDB,
 PARTITION pmax VALUES LESS THAN MAXVALUE ENGINE = InnoDB
);
```

MySQL 中的分区广泛用于`date`、`datetime`或`timestamp`列。如果您想要在数据库中存储一些事件，并且所有查询都基于时间范围，您可以使用这样的分区。

分区函数`to_days()`返回自`0000-01-01`以来的天数，这是一个整数：

```sql
mysql> CREATE TABLE `event_history` (
  `event_id` int(11) NOT NULL,
  `event_name` varchar(10) NOT NULL,
  `created_at` datetime NOT NULL,
  `last_updated` timestamp DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `event_type` varchar(10) NOT NULL,
  `msg` tinytext NOT NULL,
  PRIMARY KEY (`event_id`,`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (to_days(created_at))
(PARTITION p20170930 VALUES LESS THAN (736967) ENGINE = InnoDB,
PARTITION p20171001 VALUES LESS THAN (736968) ENGINE = InnoDB,
PARTITION p20171002 VALUES LESS THAN (736969) ENGINE = InnoDB,
PARTITION p20171003 VALUES LESS THAN (736970) ENGINE = InnoDB,
PARTITION p20171004 VALUES LESS THAN (736971) ENGINE = InnoDB,
PARTITION p20171005 VALUES LESS THAN (736972) ENGINE = InnoDB,
PARTITION p20171006 VALUES LESS THAN (736973) ENGINE = InnoDB,
PARTITION p20171007 VALUES LESS THAN (736974) ENGINE = InnoDB,
PARTITION p20171008 VALUES LESS THAN (736975) ENGINE = InnoDB,
PARTITION p20171009 VALUES LESS THAN (736976) ENGINE = InnoDB,
PARTITION p20171010 VALUES LESS THAN (736977) ENGINE = InnoDB,
PARTITION p20171011 VALUES LESS THAN (736978) ENGINE = InnoDB,
PARTITION p20171012 VALUES LESS THAN (736979) ENGINE = InnoDB,
PARTITION p20171013 VALUES LESS THAN (736980) ENGINE = InnoDB,
PARTITION p20171014 VALUES LESS THAN (736981) ENGINE = InnoDB,
PARTITION p20171015 VALUES LESS THAN (736982) ENGINE = InnoDB,
PARTITION pmax VALUES LESS THAN MAXVALUE ENGINE = InnoDB
);
```

如果要将现有表转换为分区表，并且分区键不是`PRIMARY KEY`的一部分，则需要删除`PRIMARY KEY`并将分区键作为`PRIMARY KEY`和所有唯一键的一部分添加。否则，您将收到错误`ERROR 1503 (HY000): A PRIMARY KEY must include all columns in the table's partitioning function.`。您可以按以下方式执行：

```sql
mysql> ALTER TABLE employees DROP PRIMARY KEY, ADD PRIMARY KEY(emp_no,hire_date);
Query OK, 0 rows affected (0.11 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

```sql
mysql> ALTER TABLE employees PARTITION BY RANGE (YEAR(hire_date))
        (PARTITION p1980 VALUES LESS THAN (1980) ENGINE = InnoDB,
        PARTITION p1990 VALUES LESS THAN (1990) ENGINE = InnoDB,
        PARTITION p2000 VALUES LESS THAN (2000) ENGINE = InnoDB,
        PARTITION p2010 VALUES LESS THAN (2010) ENGINE = InnoDB,
        PARTITION p2020 VALUES LESS THAN (2020) ENGINE = InnoDB,
        PARTITION pmax VALUES LESS THAN MAXVALUE ENGINE = InnoDB
       );
Query OK, 300025 rows affected (4.71 sec)
Records: 300025  Duplicates: 0  Warnings: 0
```

有关`RANGE`分区的更多详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/partitioning-range.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-range.html)。

# 移除分区

如果您希望移除分区，可以执行`REMOVE PARTITIONING`语句：

```sql
mysql> ALTER TABLE employees REMOVE PARTITIONING;
Query OK, 0 rows affected (0.09 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

# `RANGE COLUMNS`分区

`RANGE COLUMNS`分区类似于`RANGE`分区，但允许您基于多个列值的范围定义分区。此外，您可以使用除整数类型以外的列来定义范围。`RANGE COLUMNS`分区与`RANGE`分区有以下显著不同：

+   `RANGE COLUMNS`不接受表达式，只接受列名

+   `RANGE COLUMNS`接受一个或多个列的列表

+   `RANGE COLUMNS`分区列不限于整数列；字符串、`DATE`和`DATETIME`列也可以用作分区列

您可以直接在`RANGE COLUMNS`中使用`hire_date`列，而不是使用`to_days()`或`year()`函数：

```sql
mysql> ALTER TABLE employees 
    PARTITION BY RANGE COLUMNS (hire_date) 
    (PARTITION p0 VALUES LESS THAN ('1970-01-01'),
     PARTITION p1 VALUES LESS THAN ('1980-01-01'),
     PARTITION p2 VALUES LESS THAN ('1990-01-01'),
     PARTITION p3 VALUES LESS THAN ('2000-01-01'),
     PARTITION p4 VALUES LESS THAN ('2010-01-01'),
     PARTITION p5 VALUES LESS THAN (MAXVALUE)
    );
Query OK, 300025 rows affected (4.71 sec)
Records: 300025  Duplicates: 0  Warnings: 0
```

或者您可以根据他们的`last_name`来划分员工。这将不能保证在分区之间的均匀分布：

```sql
mysql> ALTER TABLE employees 
PARTITION BY RANGE COLUMNS (last_name) 
    (PARTITION p0 VALUES LESS THAN ('b'),
     PARTITION p1 VALUES LESS THAN ('f'),
     PARTITION p2 VALUES LESS THAN ('l'),
     PARTITION p3 VALUES LESS THAN ('q'),
     PARTITION p4 VALUES LESS THAN ('u'),
     PARTITION p5 VALUES LESS THAN ('z')
  );
Query OK, 300025 rows affected (4.71 sec)
Records: 300025  Duplicates: 0  Warnings: 0
```

使用`RANGE COLUMNS`，您可以在分区函数中放置多个列：

```sql
mysql> CREATE TABLE range_columns_example (
    a INT,
    b INT,
    c INT,
    d INT,
    e INT,
    PRIMARY KEY(a, b, c)
)
PARTITION BY RANGE COLUMNS(a,b,c) (
    PARTITION p0 VALUES LESS THAN (0,25,50),
    PARTITION p1 VALUES LESS THAN (10,50,100),
    PARTITION p2 VALUES LESS THAN (10,100,200),
    PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
 );
```

如果插入值`a=10`、`b=20`、`c=100`、`d=100`、`e=100`，它将进入`p1`。在设计按`RANGE COLUMNS`分区的表时，您可以通过使用`mysql`客户端来测试连续的分区定义，如下所示：

```sql
mysql> SELECT (10,20,100) < (0,25,50) p0, (10,20,100) < (10,50,100) p1, (10,20,100) < (10,100,200) p2;
+----+----+----+
| p0 | p1 | p2 |
+----+----+----+
|  0 |  1 |  1 |
+----+----+----+
1 row in set (0.00 sec)
```

在这种情况下，插入将进入`p1`。

# LIST 和 LIST COLUMNS 分区

`LIST`分区类似于`RANGE`分区，每个分区根据列值在一组值列表中的成员资格而不是一组连续值范围中的成员资格来定义和选择。

您需要通过`PARTITION BY LIST(<expr>)`来定义，其中`expr`是一个列值或基于列值并返回整数值的表达式。

分区定义包含`VALUES IN (<value_list>)`，其中`value_list`是一个逗号分隔的整数列表，而不是`VALUES LESS THAN (<value>)`。

如果您希望使用除整数以外的数据类型，可以使用`LIST COLUMNS`。

与`RANGE`分区不同，没有`MAXVALUE`这样的`catch-all`；分区表达式中应包含分区表达式的所有预期值。

假设有一个带有邮政编码和城市的客户表。例如，如果您想要将具有特定邮政编码的客户划分到一个分区中，您可以使用`LIST`分区：

```sql
mysql> CREATE TABLE customer (
customer_id INT,
zipcode INT,
city varchar(100),
PRIMARY KEY (customer_id, zipcode)
)
PARTITION BY LIST(zipcode) (
   PARTITION pnorth VALUES IN (560030, 560007, 560051, 560084),
   PARTITION peast VALUES IN (560040, 560008, 560061, 560085),
   PARTITION pwest VALUES IN (560050, 560009, 560062, 560086),
   PARTITION pcentral VALUES IN (560060, 560010, 560063, 560087)
);
```

如果您希望直接使用列而不是整数，可以使用`LIST COLUMNS`：

```sql
mysql> CREATE TABLE customer (
customer_id INT,
zipcode INT,
city varchar(100),
PRIMARY KEY (customer_id, city)
)
PARTITION BY LIST COLUMNS(city) (
   PARTITION pnorth VALUES IN ('city1','city2','city3'),
   PARTITION peast VALUES IN ('city4','city5','city6'),
   PARTITION pwest VALUES IN ('city7','city8','city9'),
   PARTITION pcentral VALUES IN ('city10','city11','city12')
);
```

# HASH 和 LINEAR HASH 分区

使用`HASH`进行分区主要是为了确保数据在预定数量的分区中均匀分布。对于范围或列表分区，您必须明确指定给定列值或列值集应存储在哪个分区；而对于哈希分区，这个决定已经为您处理，您只需要指定一个要进行哈希处理的列值或基于列值的表达式，以及要将分区表分成的分区数。

如果要均匀分配员工，可以使用`YEAR(hire_date)`的`HASH`并指定分区的数量，而不是对`YEAR(hire_date)`进行`RANGE`分区。当使用`PARTITION BY HASH`时，存储引擎根据表达式的结果的模来确定要使用的分区。

例如，如果`hire_date`是`1987-11-28`，`YEAR(hire_date)`将是`1987`，`MOD(1987,8)`是`3`。因此，行进入第三个分区：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY HASH(YEAR(hire_date))
PARTITIONS 8;
```

最有效的哈希函数是对单个表列进行操作，并且其值随着列值的增加或减少而一致变化。

在`LINEAR HASH`分区中，您可以使用相同的语法，只是添加一个`LINEAR`关键字。MySQL 使用二的幂算法来确定分区，而不是使用`MODULUS`操作。有关更多详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/partitioning-linear-hash.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-linear-hash.html)：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY LINEAR HASH(YEAR(hire_date))
PARTITIONS 8;
```

# KEY 和 LINEAR KEY 分区

按键分区类似于按哈希分区，只是哈希分区使用用户定义的表达式，而键分区的哈希函数由 MySQL 服务器提供。这个内部哈希函数基于与`PASSWORD()`函数相同的算法。

`KEY`只接受零个或多个列名的列表。如果将用作分区键的列，必须包括表的主键的一部分或全部，如果表有主键。如果未指定列名作为分区键，则使用表的主键，如果有的话：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY KEY()
PARTITIONS 8;
```

# 子分区

您可以进一步将每个分区划分为分区表。这称为**子分区**或**复合分区**：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE( YEAR(hire_date) )
  SUBPARTITION BY HASH(emp_no)
    SUBPARTITIONS 4 (
        PARTITION p0 VALUES LESS THAN (1990),
        PARTITION p1 VALUES LESS THAN (2000),
        PARTITION p2 VALUES LESS THAN (2010),
        PARTITION p3 VALUES LESS THAN (2020),
        PARTITION p4 VALUES LESS THAN MAXVALUE
    );
```

# 分区修剪和选择

MySQL 不会扫描没有匹配值的分区；这是自动的，称为分区修剪。MySQL 优化器评估给定值的分区表达式，确定包含该值的分区，并仅扫描该分区。

`SELECT`、`DELETE`和`UPDATE`语句支持分区修剪。`INSERT`语句目前无法修剪。

您还可以明确指定匹配给定`WHERE`条件的分区和子分区。

# 如何做...

分区修剪仅适用于查询，但支持对查询和多个 DML 语句进行分区的显式分区选择。

# 分区修剪

以基于`emp_no`进行分区的`employees`表为例：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (YEAR(hire_date))
(PARTITION p1980 VALUES LESS THAN (1980) ENGINE = InnoDB,
 PARTITION p1990 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p2000 VALUES LESS THAN (2000) ENGINE = InnoDB,
 PARTITION p2010 VALUES LESS THAN (2010) ENGINE = InnoDB,
 PARTITION p2020 VALUES LESS THAN (2020) ENGINE = InnoDB,
 PARTITION pmax VALUES LESS THAN MAXVALUE ENGINE = InnoDB
);
```

假设执行以下`SELECT`查询：

```sql
mysql> SELECT last_name,birth_date FROM employees WHERE hire_date='1999-02-01' AND first_name='Mariangiola';
```

MySQL 优化器检测到查询中使用了分区列，并自动确定要扫描的分区。

在此查询中，首先计算`YEAR('1999-02-01')`，即`1999`，然后扫描`p2000`分区而不是整个表。这大大减少了查询时间。

如果给出的不是`hire_date='1999-02-01'`，而是一个范围，比如`hire_date>='1999-02-01'`，那么将扫描`p2000`、`p2010`、`p2020`和`pmax`分区。

如果在`WHERE`子句中没有给出`hire_date='1999-02-01'`表达式，MySQL 必须扫描整个表。

要了解优化器扫描的分区，可以执行查询的`EXPLAIN`计划，该计划在第十三章的*Explain plan*部分中有解释，*性能调整*：

```sql
mysql> EXPLAIN SELECT last_name,birth_date FROM employees WHERE hire_date='1999-02-01' AND first_name='Mariangiola'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
 partitions: p2000
         type: ref
possible_keys: name
          key: name
      key_len: 58
          ref: const
         rows: 120
     filtered: 10.00
        Extra: Using index condition

```

```sql
mysql> EXPLAIN SELECT last_name,birth_date FROM employees WHERE hire_date>='1999-02-01' AND first_name='Mariangiola'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
 partitions: p2000,p2010,p2020,pmax
         type: ref
possible_keys: name
          key: name
      key_len: 58
          ref: const
         rows: 121
     filtered: 33.33
        Extra: Using index condition
1 row in set, 1 warning (0.00 sec)
```

# 分区选择

分区修剪是基于`WHERE`子句的自动选择。您可以在查询中明确指定要扫描的分区。查询可以是`SELECT`、`DELETE`、`INSERT`、`REPLACE`、`UPDATE`、`LOAD DATA`和`LOAD XML`。`PARTITION`选项用于从给定表中选择分区，您应该在所有其他选项之前，包括任何表别名，指定关键字`PARTITION` <partition name>，例如：

```sql
mysql> SELECT emp_no,hire_date FROM employees PARTITION (p1990) LIMIT 10;
+--------+------------+
| emp_no | hire_date  |
+--------+------------+
| 413688 | 1989-12-10 |
| 242368 | 1989-08-06 |
| 283280 | 1985-11-22 |
| 405098 | 1985-11-16 |
|  30404 | 1985-07-17 |
| 419259 | 1988-03-21 |
| 466254 | 1986-11-28 |
| 428971 | 1986-12-13 |
|  94467 | 1987-01-28 |
| 259555 | 1987-07-30 |
+--------+------------+
10 rows in set (0.00 sec)
```

同样，我们可以删除：

```sql
mysql> DELETE FROM employees PARTITION (p1980, p1990) WHERE first_name LIKE 'j%';
Query OK, 7001 rows affected (0.12 sec)
```

# 分区管理

在管理分区时最重要的是提前添加足够的分区以进行基于时间的`RANGE`分区。如果未能这样做，将在插入时出现错误，或者如果定义了`MAXVALUE`分区，则所有插入都将进入`MAXVALUE`分区。例如，考虑没有`pmax`分区的`event_history`表：

```sql
mysql> CREATE TABLE `event_history` (
  `event_id` int(11) NOT NULL,
  `event_name` date NOT NULL,
  `created_at` datetime NOT NULL,
  `last_updated` timestamp DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `event_type` varchar(10) NOT NULL,
  `msg` tinytext NOT NULL,
  PRIMARY KEY (`event_id`,`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (to_days(created_at))
(PARTITION p20170930 VALUES LESS THAN (736967) ENGINE = InnoDB,
PARTITION p20171001 VALUES LESS THAN (736968) ENGINE = InnoDB,
PARTITION p20171002 VALUES LESS THAN (736969) ENGINE = InnoDB,
PARTITION p20171003 VALUES LESS THAN (736970) ENGINE = InnoDB,
PARTITION p20171004 VALUES LESS THAN (736971) ENGINE = InnoDB,
PARTITION p20171005 VALUES LESS THAN (736972) ENGINE = InnoDB,
PARTITION p20171006 VALUES LESS THAN (736973) ENGINE = InnoDB,
PARTITION p20171007 VALUES LESS THAN (736974) ENGINE = InnoDB,
PARTITION p20171008 VALUES LESS THAN (736975) ENGINE = InnoDB,
PARTITION p20171009 VALUES LESS THAN (736976) ENGINE = InnoDB,
PARTITION p20171010 VALUES LESS THAN (736977) ENGINE = InnoDB,
PARTITION p20171011 VALUES LESS THAN (736978) ENGINE = InnoDB,
PARTITION p20171012 VALUES LESS THAN (736979) ENGINE = InnoDB,
PARTITION p20171013 VALUES LESS THAN (736980) ENGINE = InnoDB,
PARTITION p20171014 VALUES LESS THAN (736981) ENGINE = InnoDB,
PARTITION p20171015 VALUES LESS THAN (736982) ENGINE = InnoDB
);
```

该表接受`INSERTS`直到 2017 年 10 月 15 日；之后，`INSERTS`将失败。

另一个重要的事情是在数据超过保留期限后进行`DELETE`。

# 如何做...

要执行这些操作，您需要执行`ALTER`命令。

# 添加分区

要添加新分区，请执行`ADD PARTITION (<PARTITION DEFINITION>)`语句：

```sql
mysql> ALTER TABLE event_history ADD PARTITION (
PARTITION p20171016 VALUES LESS THAN (736983) ENGINE = InnoDB,
PARTITION p20171017 VALUES LESS THAN (736984) ENGINE = InnoDB
);
```

此语句会锁定整个表的时间非常短。

# 重新组织分区

如果存在`MAXVALUE`分区，则无法在`MAXVALUE`之后添加分区；在这种情况下，您需要将`REORGANIZE MAXVALUE`分区分成两个分区：

```sql
mysql> ALTER TABLE event_history REORGANIZE PARTITION pmax INTO (PARTITION p20171016 VALUES LESS THAN (736983) ENGINE = InnoDB,
PARTITION pmax VALUES LESS THAN MAXVALUE ENGINE = InnoDB);
```

请记住，当重新组织分区时，MySQL 必须大幅移动数据，表在此期间将被锁定。

您还可以将多个分区重新组织为单个分区：

```sql
mysql> ALTER TABLE event_history REORGANIZE PARTITION p20171001,p20171002,p20171003,p20171004,p20171005,p20171006,p20171007 
INTO (PARTITION p2017_oct_week1 VALUES LESS THAN (736974));
```

# 删除分区

如果数据已经超过保留期限，您可以使用`DROP`整个分区，与传统的`DELETE FROM TABLE`语句相比，这是非常快速的。这对于高效地存档数据非常有帮助。

如果`p20170930`已经超过了保留期限，您可以使用`ALTER TABLE ... DROP PARTITION`语句删除该分区：

```sql
mysql> ALTER TABLE event_history DROP PARTITION p20170930;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

删除分区会从表中删除`PARTITION DEFINITION`。

# 截断分区

如果您希望在表中保留`PARTITION DEFINITION`并仅删除数据，可以执行`TRUNCATE PARTITION`命令：

```sql
mysql> ALTER TABLE event_history TRUNCATE PARTITION p20171001;
Query OK, 0 rows affected (0.08 sec)
```

# 管理 HASH 和 KEY 分区

对`HASH`和`KEY`分区执行的操作是完全不同的。您只能减少或增加分区的数量。

假设`employees`表是基于`HASH`进行分区的：

```sql
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY HASH(YEAR(hire_date))
PARTITIONS 8;
```

要将分区从`8`减少到`6`，您可以执行`COALESCE PARTITION`语句，并指定要减少的分区数，即*8-6=2*：

```sql
mysql> ALTER TABLE employees COALESCE PARTITION 2;
Query OK, 0 rows affected (0.31 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

要将分区从`6`增加到`16`，您可以执行`ADD PARTITION`语句，并指定要增加的分区数，即*16-6=10*：

```sql
mysql> ALTER TABLE employees ADD PARTITION PARTITIONS 10;
Query OK, 0 rows affected (5.11 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

# 其他操作

您还可以执行其他操作，例如`REBUILD`，`OPTIMIZE`，`ANALYZE`和`REPAIR`语句，例如：

```sql
mysql> ALTER TABLE event_history REPAIR PARTITION p20171009, p20171010;
```

# 分区信息

本节讨论了获取有关现有分区的信息，可以通过多种方式完成。

# 如何做...

让我们深入了解一下。

# 使用 SHOW CREATE TABLE

要知道表是否已分区，可以执行`SHOW CREATE TABLE\G`语句，该语句显示了表定义以及分区，例如：

```sql
mysql> SHOW CREATE TABLE employees \G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  `address` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
/*!50100 PARTITION BY RANGE (YEAR(hire_date))
(PARTITION p1980 VALUES LESS THAN (1980) ENGINE = InnoDB,
 PARTITION p1990 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p2000 VALUES LESS THAN (2000) ENGINE = InnoDB,
 PARTITION p2010 VALUES LESS THAN (2010) ENGINE = InnoDB,
 PARTITION p2020 VALUES LESS THAN (2020) ENGINE = InnoDB,
 PARTITION pmax VALUES LESS THAN MAXVALUE ENGINE = InnoDB) */
```

# 使用 SHOW TABLE STATUS

您可以执行`SHOW TABLE STATUS`命令，并在输出中检查`Create_options`：

```sql
mysql> SHOW TABLE STATUS LIKE 'employees'\G
*************************** 1\. row ***************************
           Name: employees
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: NULL
 Avg_row_length: NULL
    Data_length: NULL
Max_data_length: NULL
   Index_length: NULL
      Data_free: NULL
 Auto_increment: NULL
    Create_time: 2017-10-01 05:01:53
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options: partitioned
        Comment: 
1 row in set (0.00 sec)
```

# 使用 EXPLAIN

`EXPLAIN`计划显示了查询所扫描的所有分区。如果您对`SELECT * FROM <table>`运行`EXPLAIN`计划，它将列出所有分区，例如：

```sql
mysql> EXPLAIN SELECT * FROM employees\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
 partitions: p1980,p1990,p2000,p2010,p2020,pmax
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 292695
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

# 查询 INFORMATION_SCHEMA.PARTITIONS 表

与所有前面的方法相比，`INFORMATION_SCHEMA.PARTITIONS`提供了有关分区的更多信息：

```sql
mysql> SHOW CREATE TABLE INFORMATION_SCHEMA.PARTITIONS\G
*************************** 1\. row ***************************
       Table: PARTITIONS
Create Table: CREATE TEMPORARY TABLE `PARTITIONS` (
  `TABLE_CATALOG` varchar(512) NOT NULL DEFAULT '',
  `TABLE_SCHEMA` varchar(64) NOT NULL DEFAULT '',
  `TABLE_NAME` varchar(64) NOT NULL DEFAULT '',
  `PARTITION_NAME` varchar(64) DEFAULT NULL,
  `SUBPARTITION_NAME` varchar(64) DEFAULT NULL,
  `PARTITION_ORDINAL_POSITION` bigint(21) unsigned DEFAULT NULL,
  `SUBPARTITION_ORDINAL_POSITION` bigint(21) unsigned DEFAULT NULL,
  `PARTITION_METHOD` varchar(18) DEFAULT NULL,
  `SUBPARTITION_METHOD` varchar(12) DEFAULT NULL,
  `PARTITION_EXPRESSION` longtext,
  `SUBPARTITION_EXPRESSION` longtext,
  `PARTITION_DESCRIPTION` longtext,
  `TABLE_ROWS` bigint(21) unsigned NOT NULL DEFAULT '0',
  `AVG_ROW_LENGTH` bigint(21) unsigned NOT NULL DEFAULT '0',
  `DATA_LENGTH` bigint(21) unsigned NOT NULL DEFAULT '0',
  `MAX_DATA_LENGTH` bigint(21) unsigned DEFAULT NULL,
  `INDEX_LENGTH` bigint(21) unsigned NOT NULL DEFAULT '0',
  `DATA_FREE` bigint(21) unsigned NOT NULL DEFAULT '0',
  `CREATE_TIME` datetime DEFAULT NULL,
  `UPDATE_TIME` datetime DEFAULT NULL,
  `CHECK_TIME` datetime DEFAULT NULL,
  `CHECKSUM` bigint(21) unsigned DEFAULT NULL,
  `PARTITION_COMMENT` varchar(80) NOT NULL DEFAULT '',
  `NODEGROUP` varchar(12) NOT NULL DEFAULT '',
  `TABLESPACE_NAME` varchar(64) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
```

要了解有关表分区的更多详细信息，您可以通过指定数据库名称和表名称来查询`INFORMATION_SCHEMA.PARTITIONS`表，例如：

```sql
mysql> SELECT PARTITION_NAME FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='employees' AND TABLE_NAME='employees';
+----------------+
| PARTITION_NAME |
+----------------+
| p1980          |
| p1990          |
| p2000          |
| p2010          |
| p2020          |
| pmax           |
+----------------+
6 rows in set (0.00 sec)
```

您可以在该分区中获取诸如`PARTITION_METHOD`，`PARTITION_EXPRESSION`，`PARTITION_DESCRIPTION`和`TABLE_ROWS`等详细信息：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_SCHEMA='employees' AND TABLE_NAME='employees' AND PARTITION_NAME='p1990'\G
*************************** 1\. row ***************************
                TABLE_CATALOG: def
                 TABLE_SCHEMA: employees
                   TABLE_NAME: employees
               PARTITION_NAME: p1990
            SUBPARTITION_NAME: NULL
   PARTITION_ORDINAL_POSITION: 2
SUBPARTITION_ORDINAL_POSITION: NULL
             PARTITION_METHOD: RANGE
          SUBPARTITION_METHOD: NULL
         PARTITION_EXPRESSION: YEAR(hire_date)
      SUBPARTITION_EXPRESSION: NULL
        PARTITION_DESCRIPTION: 1990
                   TABLE_ROWS: 157588
               AVG_ROW_LENGTH: 56
                  DATA_LENGTH: 8929280
              MAX_DATA_LENGTH: NULL
                 INDEX_LENGTH: 8929280
                    DATA_FREE: 0
                  CREATE_TIME: NULL
                  UPDATE_TIME: NULL
                   CHECK_TIME: NULL
                     CHECKSUM: NULL
            PARTITION_COMMENT: 
                    NODEGROUP: default
              TABLESPACE_NAME: NULL
1 row in set (0.00 sec)
```

有关更多详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/partitions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/partitions-table.html)。

# 高效管理生存时间和软删除行

`RANGE COLUMNS`在管理生存期和软删除行方面非常有用。假设您有一个应用程序，它指定了行的到期时间（在超过到期时间后将被删除的行），并且到期时间是变化的。

假设应用程序可以执行以下类型的插入：

+   插入持久数据

+   带有到期日的插入

如果到期时间是恒定的，即所有插入的行都将在一定时间后被删除，我们可以使用范围分区。但是，如果到期时间是变化的，即一些行将在一周内被删除，一些将在一个月内被删除，一些将在一年内被删除，一些没有到期时间，那么就不可能创建分区。在这种情况下，您可以使用下面解释的`RANGE COLUMNS`分区。

# 它是如何工作的...

我们引入一个名为`soft_delete`的列，将由触发器设置。`soft_delete`列将成为范围列分区的一部分。

分区将是（`soft_delete`，expires）。`soft_delete`和 expires 共同控制行应该进入哪个分区。soft_delete 列决定了行的保留。如果 expires 为 0，则触发器将`soft_delete`值设置为 0，将行放入`no_retention`分区，如果 expires 的值超出分区范围，触发器将`soft_delete`值设置为 1，并将行放入`long_retention`分区。如果 expires 的值在分区范围内，触发器将`soft_delete`值设置为`2`。根据 expires 的值，行将被放入相应的分区。

总之，`soft_delete`将是：

+   `0`：如果过期值为 0

+   `1`：如果过期时间距离时间戳超过 30 天

+   `2`：如果过期时间距离时间戳不到 30 天

我们创建

+   1 个`no_retention`分区（`soft_delete = 0`）

+   1 个`long_retention`分区（`soft_delete = 1`）

+   8 个每日分区（`soft_delete = 2`）

# 如何做...

您可以创建一个如下的表：

```sql
mysql> CREATE TABLE `customer_data` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `msg` text,
  `timestamp` bigint(20) NOT NULL DEFAULT '0',
  `expires` bigint(20) NOT NULL DEFAULT '0',
  `soft_delete` tinyint(3) unsigned NOT NULL DEFAULT '1',
  PRIMARY KEY (`id`,`expires`,`soft_delete`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
/*!50500 PARTITION BY RANGE COLUMNS(soft_delete,expires)
(PARTITION no_retention VALUES LESS THAN (0,MAXVALUE) ENGINE = InnoDB,
 PARTITION long_retention VALUES LESS THAN (1,MAXVALUE) ENGINE = InnoDB,
 PARTITION pd20171017 VALUES LESS THAN (2,1508198400000) ENGINE = InnoDB,
 PARTITION pd20171018 VALUES LESS THAN (2,1508284800000) ENGINE = InnoDB,
 PARTITION pd20171019 VALUES LESS THAN (2,1508371200000) ENGINE = InnoDB,
 PARTITION pd20171020 VALUES LESS THAN (2,1508457600000) ENGINE = InnoDB,
 PARTITION pd20171021 VALUES LESS THAN (2,1508544000000) ENGINE = InnoDB,
 PARTITION pd20171022 VALUES LESS THAN (2,1508630400000) ENGINE = InnoDB,
 PARTITION pd20171023 VALUES LESS THAN (2,1508716800000) ENGINE = InnoDB,
 PARTITION pd20171024 VALUES LESS THAN (3,1508803200000) ENGINE = InnoDB,
 PARTITION pd20171025 VALUES LESS THAN (3,1508869800000) ENGINE = InnoDB,
 PARTITION pd20171026 VALUES LESS THAN (3,1508956200000) ENGINE = InnoDB) */;
```

将有一个缓冲周分区，将会在 42 天后，并且始终为空，以便我们可以分割和 7+2 个每日分区，带有 2 个缓冲。

```sql
mysql> DROP TRIGGER IF EXISTS customer_data_insert;
DELIMITER $$
CREATE TRIGGER customer_data_insert
BEFORE INSERT
   ON customer_data FOR EACH ROW
BEGIN
    SET NEW.soft_delete = (IF((NEW.expires = 0),0,IF((ROUND((((((NEW.expires - NEW.timestamp) / 1000) / 60) / 60) / 24),0) <= 7),2,1)));
END;
$$
DELIMITER ;
```

```sql
mysql> DROP TRIGGER IF EXISTS customer_data_update;
DELIMITER $$
CREATE TRIGGER customer_data_update
BEFORE UPDATE
   ON customer_data FOR EACH ROW
BEGIN
    SET NEW.soft_delete = (IF((NEW.expires = 0),0,IF((ROUND((((((NEW.expires - NEW.timestamp) / 1000) / 60) / 60) / 24),0) <= 7),2,1)));
END;
$$
DELIMITER ;
```

+   假设客户端插入了一个时间戳为 1508265000（2017-10-17 18:30:00）并且到期值为 1508351400（2017-10-18 18:30:00）的行，soft_delete 将为 2，这将使其进入分区 pd20171019

```sql
mysql> INSERT INTO customer_data(id, msg, timestamp, expires) VALUES(1,'test',1508265000000,1508351400000);
Query OK, 1 row affected (0.05 sec)

```

```sql
mysql> SELECT * FROM customer_data PARTITION (pd20171019);
+----+------+---------------+---------------+-------------+
| id | msg  | timestamp     | expires       | soft_delete |
+----+------+---------------+---------------+-------------+
|  1 | test | 1508265000000 | 1508351400000 |           2 |
+----+------+---------------+---------------+-------------+
1 row in set (0.00 sec)
```

+   假设客户端没有设置到期时间，expires 列将为 0，这将使`soft_delete`为`0`，并且将进入`no_retention`分区。

```sql
mysql> INSERT INTO customer_data(id, msg, timestamp, expires)  VALUES(2,'non_expiry_row',1508265000000,0);
Query OK, 1 row affected (0.07 sec)
```

```sql
mysql> SELECT * FROM customer_data PARTITION (no_retention);
+----+----------------+---------------+---------+-------------+
| id | msg            | timestamp     | expires | soft_delete |
+----+----------------+---------------+---------+-------------+
|  2 | non_expiry_row | 1508265000000 |       0 |           0 |
+----+----------------+---------------+---------+-------------+
1 row in set (0.00 sec)
```

+   假设客户端希望设置到期时间（假设为 2017-10-19 06:30:00），到期列可以更新，这将把行从`no_retention`分区移动到相应的分区（这会有一些性能影响，因为行必须在分区之间移动）

```sql
mysql> UPDATE customer_data SET expires=1508394600000 WHERE id=2;
Query OK, 1 row affected (0.06 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

```sql
mysql> SELECT * FROM customer_data PARTITION (no_retention);
Empty set (0.00 sec)
```

```sql
mysql> SELECT * FROM customer_data PARTITION (pd20171020);
+----+----------------+---------------+---------------+-------------+
| id | msg            | timestamp     | expires       | soft_delete |
+----+----------------+---------------+---------------+-------------+
|  2 | non_expiry_row | 1508265000000 | 1508394600000 |           2 |
+----+----------------+---------------+---------------+-------------+
1 row in set (0.00 sec)
```

+   假设客户端设置了一个超出我们分区范围的到期时间，它将自动进入`long_retention`分区。

```sql
mysql> INSERT INTO customer_data(id, msg, timestamp, expires)  VALUES(3,'long_expiry',1507852800000,1608025600000);

mysql> SELECT * FROM customer_data PARTITION (long_retention);
+----+-------------+---------------+---------------+-------------+
| id | msg         | timestamp     | expires       | soft_delete |
+----+-------------+---------------+---------------+-------------+
|  3 | long_expiry | 1507852800000 | 1608025600000 |           1 |
+----+-------------+---------------+---------------+-------------+
1 row in set (0.00 sec)

```

如果更新`soft_delete`，则跨分区移动行的速度很慢，行将从默认分区移动到其他分区。

**扩展逻辑**

我们可以扩展逻辑并增加`soft_delete`的值，以适应更多类型的分区。

+   `0`：如果过期值为 0

+   `3`：如果过期时间距离时间戳不到 7 天

+   `2`：如果过期时间距离时间戳不到 60 天

+   `1`：如果过期时间距离时间戳超过 60 天

`soft_delete`列将成为分区的一部分。我们创建

+   单个`no_retention`分区，如果`soft_delete`的值为`0`

+   单个`long_retention`分区，如果`soft_delete 1`的值

+   每周分区，如果`soft_delete 2`的值

+   每日分区，如果`soft_delete 3`的值

**示例分区表结构**

将有一个缓冲周分区，将会在 42 天后，并且始终为空，以便我们可以分割和 7+2 个每日分区，带有 2 个缓冲。

```sql
mysql> DROP TRIGGER IF EXISTS customer_data_insert;
DELIMITER $$
CREATE TRIGGER customer_data_insert
BEFORE INSERT
   ON customer_data FOR EACH ROW
BEGIN
    SET NEW.soft_delete = (IF((NEW.expires = 0),0,IF((ROUND((((((NEW.expires - NEW.timestamp) / 1000) / 60) / 60) / 24),0) <= 7),3,IF((ROUND((((((NEW.expires - NEW.timestamp) / 1000) / 60) / 60) / 24),0) <= 42),2,1))));
END;
$$
DELIMITER ; 
```

```sql
mysql> DROP TRIGGER IF EXISTS customer_data_update;
DELIMITER $$
CREATE TRIGGER customer_data_update
BEFORE INSERT
   ON customer_data FOR EACH ROW
BEGIN
    SET NEW.soft_delete = (IF((NEW.expires = 0),0,IF((ROUND((((((NEW.expires - NEW.timestamp) / 1000) / 60) / 60) / 24),0) <= 7),3,IF((ROUND((((((NEW.expires - NEW.timestamp) / 1000) / 60) / 60) / 24),0) <= 42),2,1))));
END;
$$
DELIMITER ;

mysql> CREATE TABLE `customer_data` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `msg` text,
  `timestamp` bigint(20) NOT NULL DEFAULT '0',
  `expires` bigint(20) NOT NULL DEFAULT '0',
  `soft_delete` tinyint(3) unsigned NOT NULL DEFAULT '1',
  PRIMARY KEY (`id`,`expires`,`soft_delete`)
) ENGINE=InnoDB AUTO_INCREMENT=609585360 DEFAULT CHARSET=utf8
/*!50500 PARTITION BY RANGE  COLUMNS(`soft_delete`,`expires`)
(
 PARTITION no_retention VALUES LESS THAN (0,MAXVALUE) ENGINE = InnoDB,
 PARTITION long_retention VALUES LESS THAN (1,MAXVALUE) ENGINE = InnoDB, 
 PARTITION pw20171022 VALUES LESS THAN (2,1508630400000) ENGINE = InnoDB,
 PARTITION pw20171029 VALUES LESS THAN (2,1509235200000) ENGINE = InnoDB,
 PARTITION pw20171105 VALUES LESS THAN (2,1509840000000) ENGINE = InnoDB,
 PARTITION pw20171112 VALUES LESS THAN (2,1510444800000) ENGINE = InnoDB,
 PARTITION pw20171119 VALUES LESS THAN (2,1511049600000) ENGINE = InnoDB,
 PARTITION pw20171126 VALUES LESS THAN (2,1511654400000) ENGINE = InnoDB,
 PARTITION pw20171203 VALUES LESS THAN (2,1512259200000) ENGINE = InnoDB,
 -- buffer partition which will be 67 days away and will be always empty so that we can split
 PARTITION pw20171210 VALUES LESS THAN (2,1512864000000) ENGINE = InnoDB, 
 PARTITION pd20171016 VALUES LESS THAN (3,1508112000000) ENGINE = InnoDB,
 PARTITION pd20171017 VALUES LESS THAN (3,1508198400000) ENGINE = InnoDB,
 PARTITION pd20171018 VALUES LESS THAN (3,1508284800000) ENGINE = InnoDB,
 PARTITION pd20171019 VALUES LESS THAN (3,1508371200000) ENGINE = InnoDB,
 PARTITION pd20171020 VALUES LESS THAN (3,1508457600000) ENGINE = InnoDB,
 PARTITION pd20171021 VALUES LESS THAN (3,1508544000000) ENGINE = InnoDB,
 PARTITION pd20171022 VALUES LESS THAN (3,1508630400000) ENGINE = InnoDB,
 PARTITION pd20171023 VALUES LESS THAN (3,1508716800000) ENGINE = InnoDB,
 PARTITION pd20171024 VALUES LESS THAN (3,1508803200000) ENGINE = InnoDB
 ) */;
```

**管理分区**

您可以在 Linux 中创建一个`CRON`或在 mysql 中创建一个`EVENT`来管理分区。随着保留期的临近，分区管理工具应该将缓冲分区重新组织为一个可用分区和一个缓冲分区，并且删除已经超过保留期的分区。

例如，以前提到的`customer_data`表为例。

**在 20171203，您需要将分区 pw20171210 拆分为 pw20171210 和 pw20171217。**

**在 20171017，您需要将分区 pd20171024 拆分为 pd20171024 和 pd20171025。**

如果没有查询锁定表，分割（重新组织）分区将非常快（~毫秒级），只要没有（或者非常少量的）数据。因此，在数据进入分区之前，我们应该通过重新组织来保持分区为空。
