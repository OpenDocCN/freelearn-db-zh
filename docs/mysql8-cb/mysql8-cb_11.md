# 第十一章：管理表空间

在本章中，我们将涵盖以下内容：

+   更改 InnoDB REDO 日志文件的数量或大小

+   调整 InnoDB 系统表空间的大小

+   在数据目录之外创建文件表空间

+   将文件表空间复制到另一个实例"

+   管理 UNDO 表空间

+   管理通用表空间

+   压缩 InnoDB 表

# 介绍

在开始本章之前，您应该了解 InnoDB 的基础知识。

根据 MySQL 文档，

**系统表空间（共享表空间）** <q class="calibre48">"InnoDB 系统表空间包含 InnoDB 数据字典（与 InnoDB 相关对象的元数据）并且是双写缓冲区、更改缓冲区和撤消日志的存储区域。系统表空间还包含在系统表空间中创建的任何用户创建的表的表和索引数据。系统表空间被认为是共享表空间，因为它被多个表共享。</q>

<q class="calibre48">系统表空间由一个或多个数据文件表示。默认情况下，在 MySQL 数据目录中创建一个名为 ibdata1 的系统数据文件。系统数据文件的大小和数量由 innodb_data_file_path 启动选项控制。</q>

**文件表空间**

文件表空间是在其自己的数据文件中创建的单表表空间，而不是在系统表空间中创建。当启用 innodb_file_per_table 选项时，表将在文件表空间中创建。否则，InnoDB 表将在系统表空间中创建。每个文件表空间由一个.ibd 数据文件表示，默认情况下在数据库目录中创建。

文件表空间支持 DYNAMIC 和 COMPRESSED 行格式，支持变长数据的离页存储和表压缩等功能。

要了解文件表空间的优缺点，请参考[`dev.mysql.com/doc/refman/8.0/en/innodb-multiple-tablespaces.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-multiple-tablespaces.html)和[`dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_file_per_table`](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_file_per_table)。

**通用表空间**

通用表空间是使用 CREATE TABLESPACE 语法创建的共享 InnoDB 表空间。通用表空间可以在 MySQL 数据目录之外创建，能够容纳多个表，并支持所有行格式的表。

**UNDO 表空间**

撤消日志是与单个事务相关联的撤消日志记录的集合。撤消日志记录包含有关如何撤消事务对聚集索引记录的最新更改的信息。如果另一个事务需要查看原始数据（作为一致性读取操作的一部分），则从撤消日志记录中检索未修改的数据。撤消日志存在于撤消日志段中，这些段包含在回滚段中。回滚段驻留在系统表空间、临时表空间和 UNDO 表空间中。

UNDO 表空间包括一个或多个包含撤消日志的文件。InnoDB 使用的 UNDO 表空间数量由 innodb_undo_tablespaces 配置选项定义。

这些日志用于回滚事务，也用于多版本并发控制。

**数据字典**

数据字典是元数据，用于跟踪数据库对象，如表、索引和表列。对于 MySQL 8.0 中引入的 MySQL 数据字典，元数据实际上位于 MySQL 数据库目录中的 InnoDB 文件表空间文件中。对于 InnoDB 数据字典，元数据实际上位于 InnoDB 系统表空间中。

**MySQL 数据字典**

MySQL 服务器包含一个事务性的`数据字典`，用于存储有关数据库对象的信息。在以前的 MySQL 版本中，字典数据存储在元数据文件、非事务表和特定于存储引擎的`数据字典`中。

在以前的 MySQL 版本中，字典数据部分存储在元数据文件中。基于文件的元数据存储的问题包括昂贵的文件扫描、易受文件系统相关错误的影响、用于处理复制和崩溃恢复失败状态的复杂代码，以及缺乏可扩展性，使得难以为新功能和关系对象添加元数据。

MySQL `数据字典`的好处包括：

+   统一存储字典数据的集中`数据字典`模式的简单性

+   删除基于文件的元数据存储

+   字典数据的事务性、崩溃安全存储

+   字典对象的统一和集中缓存

+   一些`INFORMATION_SCHEMA`表的更简单和改进的实现

+   原子 DDL

以下列出的元数据文件已从 MySQL 中删除。除非另有说明，以前存储在元数据文件中的数据现在存储在`数据字典`表中：

+   `.frm`文件：表定义的表元数据文件。

+   `.par`文件：分区定义文件。`InnoDB`在 MySQL 5.7 中停止使用`.definition`分区文件，引入了`InnoDB`表的本机分区支持。

+   `.trn`文件：触发器命名空间文件。

+   `.trg`文件：触发器参数文件。

+   `.isl`文件：包含在 MySQL `data directory`之外创建的基于文件的表空间文件的`InnoDB`符号链接文件。

+   `db.opt`文件：数据库配置文件。这些文件，每个数据库目录一个，包含数据库默认字符集属性。

MySQL `数据字典`的限制如下：

+   在`data directory`下手动创建数据库目录（例如，使用`mkdir`）是不受支持的。手动创建的数据库目录不被 MySQL 服务器识别。

+   通过复制和移动 MyISAM 数据文件来移动存储在 MyISAM 表中的数据是不受支持的。使用此方法移动的表不会被服务器发现。

+   不支持使用复制的数据文件对个别 MyISAM 表进行简单备份和还原。

+   由于写入存储、撤销日志和重做日志，DDL 操作需要更长的时间，而不是`.frm`文件。

**字典数据的事务性存储**

`数据字典`模式将字典数据存储在事务性（`InnoDB`）表中。`数据字典`表位于`mysql`数据库中，与`非数据字典`系统表一起。

`数据字典`表在名为`mysql.ibd`的单个`InnoDB`表空间中创建在 MySQL `data directory`中。`mysql.ibd`表空间文件必须驻留在 MySQL `data directory`中，其名称不能被修改或被其他表空间使用。以前，这些表是在 MySQL 数据库目录中的单独表空间文件中创建的。

# 更改 InnoDB 重做日志文件的数量或大小

`ib_logfile0`文件和`ib_logfile1`是默认的`InnoDB`重做日志文件，每个文件大小为 48 MB，创建在`data directory`内。如果您希望更改重做日志文件的大小，只需在配置文件中更改并重新启动 MySQL。在以前的版本中，您必须对 MySQL 服务器进行缓慢的关闭，删除重做日志文件，更改配置文件，然后启动 MySQL 服务器。

从 MySQL 8 开始，`InnoDB`检测到`innodb_log_file_size`与重做日志文件大小不同。它写入一个日志检查点，关闭并删除旧的日志文件，以请求的大小创建新的日志文件，并打开新的日志文件。

# 如何做...

1.  检查当前文件的大小：

```go
shell> sudo ls -lhtr /var/lib/mysql/ib_logfile*
-rw-r-----. 1 mysql mysql 48M Oct  7 10:16 /var/lib/mysql/ib_logfile1
-rw-r-----. 1 mysql mysql 48M Oct  7 10:18 /var/lib/mysql/ib_logfile0
```

1.  停止 MySQL 服务器，并确保它在没有错误的情况下关闭：

```go
shell> sudo systemctl stop mysqld
```

1.  编辑配置文件：

```go
shell> sudo vi /etc/my.cnf
[mysqld]
innodb_log_file_size=128M
innodb_log_files_in_group=4
```

1.  启动 MySQL 服务器：

```go
shell> sudo systemctl start mysqld
```

1.  您可以验证 MySQL 在日志文件中的操作：

```go
shell> sudo less /var/log/mysqld.log
2017-10-07T11:09:35.111926Z 1 [Warning] InnoDB: Resizing redo log from 2*3072 to 4*8192 pages, LSN=249633608
2017-10-07T11:09:35.213717Z 1 [Warning] InnoDB: Starting to delete and rewrite log files.
2017-10-07T11:09:35.224724Z 1 [Note] InnoDB: Setting log file ./ib_logfile101 size to 128 MB
2017-10-07T11:09:35.225531Z 1 [Note] InnoDB: Progress in MB:
 100
2017-10-07T11:09:38.924955Z 1 [Note] InnoDB: Setting log file ./ib_logfile1 size to 128 MB
2017-10-07T11:09:38.925173Z 1 [Note] InnoDB: Progress in MB:
 100
2017-10-07T11:09:42.516065Z 1 [Note] InnoDB: Setting log file ./ib_logfile2 size to 128 MB
2017-10-07T11:09:42.516309Z 1 [Note] InnoDB: Progress in MB:
 100
2017-10-07T11:09:46.098023Z 1 [Note] InnoDB: Setting log file ./ib_logfile3 size to 128 MB
2017-10-07T11:09:46.098246Z 1 [Note] InnoDB: Progress in MB:
 100
2017-10-07T11:09:49.715400Z 1 [Note] InnoDB: Renaming log file ./ib_logfile101 to ./ib_logfile0
2017-10-07T11:09:49.715497Z 1 [Warning] InnoDB: New log files created, LSN=249633608
```

1.  您还可以查看新创建的日志文件：

```go
shell> sudo ls -lhtr /var/lib/mysql/ib_logfile*
-rw-r-----. 1 mysql mysql 128M Oct  7 11:09 /var/lib/mysql/ib_logfile1
-rw-r-----. 1 mysql mysql 128M Oct  7 11:09 /var/lib/mysql/ib_logfile2
-rw-r-----. 1 mysql mysql 128M Oct  7 11:09 /var/lib/mysql/ib_logfile3
-rw-r-----. 1 mysql mysql 128M Oct  7 11:09 /var/lib/mysql/ib_logfile0
```

# 调整 InnoDB 系统表空间的大小

`数据目录`中的`ibdata1`文件是默认的系统表空间。您可以使用`innodb_data_file_path`和`innodb_data_home_dir`配置选项来配置`ibdata1`。`innodb_data_file_path`配置选项用于配置`InnoDB`系统表空间数据文件。`innodb_data_file_path`的值应该是一个或多个数据文件规范的列表。如果命名了两个或更多数据文件，请使用分号(`;`)字符将它们分开。

如果要在`数据目录`中包含一个固定大小的 50MB 数据文件名为`ibdata1`和一个 50MB 自动扩展文件名为`ibdata2`的表空间，可以进行如下配置：

```go
shell> sudo vi /etc/my.cnf
[mysqld]
innodb_data_file_path=ibdata1:50M;ibdata2:50M:autoextend
```

如果`ibdata`文件变得如此庞大，特别是当未启用`innodb_file_per_table`且磁盘变满时，您可能希望在另一个磁盘上添加另一个数据文件。

# 如何做...

调整`InnoDB`系统表空间的大小是一个您会越来越想了解更多的主题。让我们深入了解其细节。

# 增加 InnoDB 系统表空间

假设`innodb_data_file_path`是`ibdata1:50M:autoextend`，大小已达到 76MB，您的磁盘只有 100MB，您可以添加另一个磁盘并配置在新磁盘上添加另一个表空间：

1.  停止 MySQL 服务器：

```go
shell> sudo systemctl stop mysql
```

1.  检查现有`ibdata1`文件的大小：

```go
shell> sudo ls -lhtr /var/lib/mysql/ibdata1 
-rw-r----- 1 mysql mysql 76M Oct  6 13:33 /var/lib/mysql/ibdata1
```

1.  挂载新磁盘。假设它挂载在`/var/lib/mysql_extend`上，更改所有权为`mysql`；确保文件尚未创建。如果您使用 AppArmour 或 SELinux，请确保正确设置别名或上下文：

```go
shell> sudo chown mysql:mysql /var/lib/mysql_extend
shell> sudo chmod 750 /var/lib/mysql_extend
shell> sudo ls -lhtr /var/lib/mysql_extend
```

1.  打开`my.cnf`并添加以下内容：

```go
shell> sudo vi /etc/my.cnf [mysqld]
innodb_data_home_dir=
innodb_data_file_path = ibdata1:76M;/var/lib/mysql_extend/ibdata2:50M:autoextend
```

由于`ibdata1`的现有大小为 76MB，您必须选择至少 76MB 的 maxvalue。下一个`ibdata`文件将在挂载在`/var/lib/mysql_extend/`上的新磁盘上创建。应该指定`innodb_data_home_dir`选项；否则，`mysqld`会查看不同的路径并因错误而失败：

```go
2017-10-07T06:30:00.658039Z 1 [ERROR] InnoDB: Operating system error number 2 in a file operation.
2017-10-07T06:30:00.658084Z 1 [ERROR] InnoDB: The error means the system cannot find the path specified.
2017-10-07T06:30:00.658088Z 1 [ERROR] InnoDB: If you are installing InnoDB, remember that you must create directories yourself, InnoDB does not create them.
2017-10-07T06:30:00.658092Z 1 [ERROR] InnoDB: File .//var/lib/mysql_extend/ibdata2: 'create' returned OS error 71\. Cannot continue operation
```

1.  启动 MySQL 服务器：

```go
shell> sudo systemctl start mysql
```

1.  验证新文件。由于您已将其指定为 50MB，因此文件的初始大小将为 50MB：

```go
shell> sudo ls -lhtr /var/lib/mysql_extend/
total 50M
-rw-r-----. 1 mysql mysql 50M Oct  7 07:38 ibdata2
```

```go
mysql> SHOW VARIABLES LIKE 'innodb_data_file_path';
+-----------------------+----------------------------------------------------------+
| Variable_name         | Value                                                    |
+-----------------------+----------------------------------------------------------+
| innodb_data_file_path | ibdata1:12M;/var/lib/mysql_extend/ibdata2:50M:autoextend |
+-----------------------+----------------------------------------------------------+
1 row in set (0.00 sec)
```

# 缩小 InnoDB 系统表空间

如果不使用`innodb_file_per_table`，则所有表数据都存储在系统表空间中。如果删除表，则不会回收空间。您可以缩小系统表空间并回收磁盘空间。这需要较长的停机时间，因此建议在从服务器上执行该任务，并将其从轮换中取出，然后将其提升为主服务器。

您可以通过查询`INFORMATION_SCHEMA`表来检查可用空间：

```go
mysql> SELECT SUM(data_free)/1024/1024 FROM INFORMATION_SCHEMA.TABLES;
+--------------------------+
| sum(data_free)/1024/1024 |
+--------------------------+
|               6.00000000 |
+--------------------------+
1 row in set (0.00 sec)
```

1.  停止对数据库的写入。如果是主服务器，则`mysql> SET @@GLOBAL.READ_ONLY=1;`；如果是从服务器，请停止复制并保存二进制日志坐标：

```go
mysql> STOP SLAVE;
mysql> SHOW SLAVE STATUS\G
```

1.  使用`mysqldump`或`mydumper`进行完整备份，不包括`sys`数据库：

```go
shell> mydumper -u root --password=<password> --trx-consistency-only --kill-long-queries --long-query-guard 500 --regex '^(?!sys)' --outputdir /backups
```

1.  停止 MySQL 服务器：

```go
shell> sudo systemctl stop mysql
```

1.  删除所有`*.ibd`、`*.ib_log`和`ibdata`文件。如果只使用`InnoDB`表，可以清除`数据目录`和存储系统表空间的所有位置(`innodb_data_file_path`)：

```go
shell> sudo rm -rf /var/lib/mysql/ib* /var/lib/mysql/<database directories>
shell> sudo rm -rf /var/lib/mysql_extend/*
```

1.  初始化`数据目录`：

```go
shell> sudo mysqld --initialize --datadir=/var/lib/mysql
shell> chown -R  mysql:mysql  /var/lib/mysql/
shell> chown -R  mysql:mysql  /var/lib/mysql_extend/
```

1.  获取临时密码：

```go
shell> sudo grep "temporary password is generated" /var/log/mysql/error.log | tail -1
2017-10-07T09:33:31.966223Z 4 [Note] A temporary password is generated for root@localhost: lI-qerr5agpa
```

1.  启动 MySQL 并更改密码：

```go
shell> sudo systemctl start mysqld
shell> mysql -u root -plI-qerr5agpa

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'xxxx';
Query OK, 0 rows affected (0.01 sec)
```

1.  恢复备份。使用临时密码连接到 MySQL：

```go
shell> /opt/mydumper/myloader --directory=/backups/ --queries-per-transaction=50000 --threads=6 --user=root --password=xxxx  --overwrite-tables
```

1.  如果是主服务器，请启用写入

`mysql> SET @@GLOBAL.READ_ONLY=0;`。如果是从服务器，请通过执行`CHANGE MASTER TO COMMAND`和`START SLAVE;`来恢复复制。

# 在数据目录之外创建基于文件的表空间

在上一节中，您了解了如何在另一个磁盘上创建系统表空间。在本节中，您将学习如何在另一个磁盘上创建单独的表空间。

# 如何做...

您可以挂载具有特定性能或容量特性的新磁盘，例如快速 SSD 或高容量 HDD，到目录并配置`InnoDB`以使用该磁盘。在目标目录中，MySQL 创建一个与数据库名称对应的子目录，并在其中为新表创建一个`.ibd`文件。请记住，您不能在`ALTER TABLE`语句中使用`DATA DIRECTORY`子句：

1.  挂载新磁盘并更改权限。如果您使用 AppArmour 或 SELinux，请确保正确设置别名或上下文：

```go
shell> sudo chown -R mysql:mysql /var/lib/mysql_fast_storage
shell> sudo chmod 750 /var/lib/mysql_fast_storage
```

1.  创建表：

```go
mysql> CREATE TABLE event_tracker (
event_id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
event_name varchar(10),
ts timestamp NOT NULL,
event_type varchar(10)
) 
TABLESPACE = innodb_file_per_table
DATA DIRECTORY = '/var/lib/mysql_fast_storage';
```

1.  检查在新设备上创建的`.ibd`文件：

```go
shell> sudo ls -lhtr  /var/lib/mysql_fast_storage/employees/
total 128K
-rw-r-----. 1 mysql mysql 128K Oct  7 13:48 event_tracker.ibd
```

# 将文件表空间复制到另一个实例

复制表空间文件（`.ibd`文件）是移动数据的最快方式，而不是通过`mysqldump`或`mydumper`导出和导入。数据立即可用，而不必重新插入和重建索引。有许多原因可能会复制`InnoDB`文件表空间到不同的实例：

+   在生产服务器上运行报告而不会给服务器增加额外负载

+   为新的从服务器设置相同的表数据

+   在出现问题或错误后恢复表或分区的备份版本

+   在 SSD 设备上有繁忙的表，或在高容量 HDD 设备上有大表

# 如何做...

概述是：在目的地创建与源上相同表定义的表，并在目的地上执行`DISCARD TABLESPACE`命令。在源上执行`FLUSH TABLES FOR EXPORT`，这确保了对命名表的更改已刷新到磁盘，因此可以在实例运行时进行二进制表复制。在该语句之后，表被锁定，不接受任何写入；但是，可以进行读取。您可以将该表的`.ibd`文件复制到目的地，在源上执行`UNLOCK`表，最后执行`IMPORT TABLESPACE`命令，该命令接受复制的`.ibd`文件。

例如，您希望将测试数据库中的`events_history`表从一个服务器（源）复制到另一个服务器（目的地）。

如果尚未创建，请创建`event_history`并插入一些行以进行演示：

```go
mysql> USE test;
mysql> CREATE TABLE IF NOT EXISTS `event_history`(
  `event_id` int(11) NOT NULL,
  `event_name` varchar(10) DEFAULT NULL,
  `created_at` datetime NOT NULL,
  `last_updated` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `event_type` varchar(10) NOT NULL,
  `msg` tinytext NOT NULL,
  PRIMARY KEY (`event_id`,`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (to_days(`created_at`))
(PARTITION 2017_oct_week1 VALUES LESS THAN (736974) ENGINE = InnoDB,
 PARTITION p20171008 VALUES LESS THAN (736975) ENGINE = InnoDB,
 PARTITION p20171009 VALUES LESS THAN (736976) ENGINE = InnoDB,
 PARTITION p20171010 VALUES LESS THAN (736977) ENGINE = InnoDB,
 PARTITION p20171011 VALUES LESS THAN (736978) ENGINE = InnoDB,
 PARTITION p20171012 VALUES LESS THAN (736979) ENGINE = InnoDB,
 PARTITION p20171013 VALUES LESS THAN (736980) ENGINE = InnoDB,
 PARTITION p20171014 VALUES LESS THAN (736981) ENGINE = InnoDB,
 PARTITION p20171015 VALUES LESS THAN (736982) ENGINE = InnoDB,
 PARTITION p20171016 VALUES LESS THAN (736983) ENGINE = InnoDB,
 PARTITION p20171017 VALUES LESS THAN (736984) ENGINE = InnoDB);
```

```go
mysql> INSERT INTO event_history VALUES
(1,'test','2017-10-07','2017-10-08','click','test_message'),
(2,'test','2017-10-08','2017-10-08','click','test_message'),
(3,'test','2017-10-09','2017-10-09','click','test_message'),
(4,'test','2017-10-10','2017-10-10','click','test_message'),
(5,'test','2017-10-11','2017-10-11','click','test_message'),
(6,'test','2017-10-12','2017-10-12','click','test_message'),
(7,'test','2017-10-13','2017-10-13','click','test_message'),
(8,'test','2017-10-14','2017-10-14','click','test_message');
Query OK, 8 rows affected (0.01 sec)
Records: 8  Duplicates: 0  Warnings: 0
```

# 复制完整表

1.  **在目的地**：创建与源上相同定义的表：

```go
mysql> USE test;
mysql> CREATE TABLE IF NOT EXISTS `event_history`(
  `event_id` int(11) NOT NULL,
  `event_name` varchar(10) DEFAULT NULL,
  `created_at` datetime NOT NULL,
  `last_updated` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `event_type` varchar(10) NOT NULL,
  `msg` tinytext NOT NULL,
  PRIMARY KEY (`event_id`,`created_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
PARTITION BY RANGE (to_days(`created_at`))
(PARTITION 2017_oct_week1 VALUES LESS THAN (736974) ENGINE = InnoDB,
 PARTITION p20171008 VALUES LESS THAN (736975) ENGINE = InnoDB,
 PARTITION p20171009 VALUES LESS THAN (736976) ENGINE = InnoDB,
 PARTITION p20171010 VALUES LESS THAN (736977) ENGINE = InnoDB,
 PARTITION p20171011 VALUES LESS THAN (736978) ENGINE = InnoDB,
 PARTITION p20171012 VALUES LESS THAN (736979) ENGINE = InnoDB,
 PARTITION p20171013 VALUES LESS THAN (736980) ENGINE = InnoDB,
 PARTITION p20171014 VALUES LESS THAN (736981) ENGINE = InnoDB,
 PARTITION p20171015 VALUES LESS THAN (736982) ENGINE = InnoDB,
 PARTITION p20171016 VALUES LESS THAN (736983) ENGINE = InnoDB,
 PARTITION p20171017 VALUES LESS THAN (736984) ENGINE = InnoDB);
```

1.  **在目的地**：丢弃表空间：

```go
mysql> ALTER TABLE event_history DISCARD TABLESPACE;
Query OK, 0 rows affected (0.05 sec)
```

1.  **在源上**：执行`FLUSH TABLES FOR EXPORT`：

```go
mysql> FLUSH TABLES event_history FOR EXPORT;
Query OK, 0 rows affected (0.00 sec)
```

1.  **在源上**：从源的`数据目录`目录中复制所有与表相关的文件（`.ibd`，`.cfg`）到目的地的`数据目录`：

```go
shell> sudo scp -i /home/mysql/.ssh/id_rsa /var/lib/mysql/test/event_history#P#* mysql@xx.xxx.xxx.xxx:/var/lib/mysql/test/
```

1.  **在源上**：解锁表以进行写入：

```go
mysql> UNLOCK TABLES;
Query OK, 0 rows affected (0.00 sec)
```

1.  **在目的地**：确保文件的所有权设置为`mysql`：

```go
shell> sudo ls -lhtr /var/lib/mysql/test
total 1.4M
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171017.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171016.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171015.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171014.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171013.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171012.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171011.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171010.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171009.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#p20171008.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:17 event_history#P#2017_oct_week1.ibd
```

1.  **在目的地**：导入表空间。只要表定义相同，就可以忽略警告。如果您也复制了`.cfg`文件，则不会出现警告：

```go
mysql> ALTER TABLE event_history IMPORT TABLESPACE;
Query OK, 0 rows affected, 12 warnings (0.31 sec)
```

1.  **在目的地**：验证数据：

```go
mysql> SELECT * FROM event_history;
+----------+------------+---------------------+---------------------+------------+--------------+
| event_id | event_name | created_at          | last_updated        | event_type | msg          |
+----------+------------+---------------------+---------------------+------------+--------------+
|        1 | test       | 2017-10-07 00:00:00 | 2017-10-08 00:00:00 | click      | test_message |
|        2 | test       | 2017-10-08 00:00:00 | 2017-10-08 00:00:00 | click      | test_message |
|        3 | test       | 2017-10-09 00:00:00 | 2017-10-09 00:00:00 | click      | test_message |
|        4 | test       | 2017-10-10 00:00:00 | 2017-10-10 00:00:00 | click      | test_message |
|        5 | test       | 2017-10-11 00:00:00 | 2017-10-11 00:00:00 | click      | test_message |
|        6 | test       | 2017-10-12 00:00:00 | 2017-10-12 00:00:00 | click      | test_message |
|        7 | test       | 2017-10-13 00:00:00 | 2017-10-13 00:00:00 | click      | test_message |
|        8 | test       | 2017-10-14 00:00:00 | 2017-10-14 00:00:00 | click      | test_message |
+----------+------------+---------------------+---------------------+------------+--------------+
8 rows in set (0.00 sec)
```

如果您在生产系统上进行操作，为了最小化停机时间，您可以将文件复制到本地，这非常快。立即执行`UNLOCK TABLES`，然后将文件复制到目的地。如果您无法承受停机时间，可以使用 Percona XtraBackup，备份单个表，并应用重做日志，生成`.ibd`文件。您可以将它们复制到目的地并导入。

# 复制表的单个分区

您在源上添加了`events_history`表的新分区，并且希望仅将新分区复制到目的地。为了您的理解，请在`events_history`表上创建新分区并插入一些行：

```go
mysql> ALTER TABLE event_history ADD PARTITION
(PARTITION p20171018 VALUES LESS THAN (736985) ENGINE = InnoDB,
 PARTITION p20171019 VALUES LESS THAN (736986) ENGINE = InnoDB);
Query OK, 0 rows affected (0.06 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> INSERT INTO event_history VALUES
(9,'test','2017-10-17','2017-10-17','click','test_message'),(10,'test','2017-10-18','2017-10-18','click','test_message');
Query OK, 1 row affected (0.01 sec)

mysql> SELECT * FROM event_history PARTITION (p20171018,p20171019);
+----------+------------+---------------------+---------------------+------------+--------------+
| event_id | event_name | created_at          | last_updated        | event_type | msg          |
+----------+------------+---------------------+---------------------+------------+--------------+
|        9 | test       | 2017-10-17 00:00:00 | 2017-10-17 00:00:00 | click      | test_message |
|       10 | test       | 2017-10-18 00:00:00 | 2017-10-18 00:00:00 | click      | test_message |
+----------+------------+---------------------+---------------------+------------+--------------+
2 rows in set (0.00 sec)
```

假设您希望将新创建的分区复制到目的地。

1.  **在目的地**：创建分区：

```go
mysql> ALTER TABLE event_history ADD PARTITION
(PARTITION p20171018 VALUES LESS THAN (736985) ENGINE = InnoDB,
 PARTITION p20171019 VALUES LESS THAN (736986) ENGINE = InnoDB);
Query OK, 0 rows affected (0.05 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

1.  **在目的地**：仅丢弃要导入的分区：

```go
mysql> ALTER TABLE event_history DISCARD PARTITION p20171018, p20171019 TABLESPACE;
 Query OK, 0 rows affected (0.06 sec)
```

1.  **在源上**：执行`FLUSH TABLE FOR EXPORT`：

```go
mysql> FLUSH TABLES event_history FOR EXPORT;
Query OK, 0 rows affected (0.01 sec)
```

1.  **在源上**：将分区的`.ibd`文件复制到目的地：

```go
shell> sudo scp -i /home/mysql/.ssh/id_rsa \
/var/lib/mysql/test/event_history#P#p20171018.ibd \
/var/lib/mysql/test/event_history#P#p20171019.ibd \
mysql@35.198.210.229:/var/lib/mysql/test/
event_history#P#p20171018.ibd                              100%  128KB 128.0KB/s   00:00   event_history#P#p20171019.ibd                              100%  128KB 128.0KB/s   00:00
```

1.  **在目的地**：确保所需分区的`.ibd`文件已复制并且所有者为`mysql`：

```go
shell> sudo ls -lhtr /var/lib/mysql/test/event_history#P#p20171018.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:54 /var/lib/mysql/test/event_history#P#p20171018.ibd

shell> sudo ls -lhtr /var/lib/mysql/test/event_history#P#p20171019.ibd
-rw-r----- 1 mysql mysql 128K Oct  7 17:54 /var/lib/mysql/test/event_history#P#p20171019.ibd
```

1.  **在目的地上：**执行`IMPORT PARTITION TABLESPACE`：

```go
mysql> ALTER TABLE event_history IMPORT PARTITION p20171018, p20171019  TABLESPACE;
Query OK, 0 rows affected, 2 warnings (0.10 sec)
```

只要表定义相同，您可以忽略警告。如果您也复制了`.cfg`文件，则不会出现警告：

```go
mysql> SHOW WARNINGS;
+---------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
|  Level   | Code | Message                                                                                                                                                         |
+---------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
|  Warning | 1810 | InnoDB: IO Read error: (2, No such file or directory) Error opening './test/event_history#P#p20171018.cfg', will attempt to import without schema verification |
| Warning | 1810 | InnoDB: IO Read error: (2, No such file or directory) Error opening './test/event_history#P#p20171019.cfg', will attempt to import without schema verification |
+---------+------+----------------------------------------------------------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

1.  **在目的地上：**验证数据：

```go
mysql> SELECT * FROM event_history PARTITION (p20171018,p20171019);
+----------+------------+---------------------+---------------------+------------+--------------+
| event_id | event_name | created_at          | last_updated        | event_type | msg          |
+----------+------------+---------------------+---------------------+------------+--------------+
|        9 | test       | 2017-10-17 00:00:00 | 2017-10-17 00:00:00 | click      | test_message |
|       10 | test       | 2017-10-18 00:00:00 | 2017-10-18 00:00:00 | click      | test_message |
+----------+------------+---------------------+---------------------+------------+--------------+
2 rows in set (0.00 sec)
```

# 另请参阅

请参阅[`dev.mysql.com/doc/refman/8.0/en/tablespace-copying.html`](https://dev.mysql.com/doc/refman/8.0/en/tablespace-copying.html)以了解有关此过程的限制的更多信息。

# 管理 UNDO 表空间

您可以通过动态变量`innodb_max_undo_log_size`（默认为 1GB）和`innodb_undo_tablespaces`（默认为 2GB，从 MySQL 8.0.2 开始为动态）来管理`UNDO`表空间的大小。

默认情况下，`innodb_undo_log_truncate`已启用。超过`innodb_max_undo_log_size`定义的阈值的表空间将被标记为截断。只有撤消表空间可以被截断。不支持截断驻留在系统表空间中的撤消日志。要进行截断，必须至少有两个撤消表空间。

# 如何做...

验证`UNDO`日志的大小：

```go
shell> sudo ls -lhtr /var/lib/mysql/undo_00*
-rw-r-----. 1 mysql mysql 19M Oct  7 17:43 /var/lib/mysql/undo_002
-rw-r-----. 1 mysql mysql 16M Oct  7 17:43 /var/lib/mysql/undo_001
```

假设您想要减少大于 15MB 的文件。请记住，只能截断一个撤消表空间。选择要截断的撤消表空间是循环进行的，以避免每次都截断相同的撤消表空间。在撤消表空间中的所有回滚段被释放后，截断操作将运行，并且撤消表空间将被截断为其初始大小。撤消表空间文件的初始大小为 10MB：

1.  确保`innodb_undo_log_truncate`已启用：

```go
mysql> SELECT @@GLOBAL.innodb_undo_log_truncate;
+-----------------------------------+
| @@GLOBAL.innodb_undo_log_truncate |
+-----------------------------------+
|                                 1 |
+-----------------------------------+
1 row in set (0.00 sec)
```

1.  将`innodb_max_undo_log_size`设置为 15MB：

```go
mysql> SELECT @@GLOBAL.innodb_max_undo_log_size;
+-----------------------------------+
| @@GLOBAL.innodb_max_undo_log_size |
+-----------------------------------+
|                        1073741824 |
+-----------------------------------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.innodb_max_undo_log_size=15*1024*1024;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@GLOBAL.innodb_max_undo_log_size;
+-----------------------------------+
| @@GLOBAL.innodb_max_undo_log_size |
+-----------------------------------+
|                          15728640 |
+-----------------------------------+
1 row in set (0.00 sec)
```

1.  直到其回滚段被释放，撤消表空间才能被截断。通常，清除系统每 128 次调用一次。为了加快撤消表空间的截断，使用`innodb_purge_rseg_truncate_frequency`选项临时增加清除系统释放回滚段的频率：

```go
mysql> SELECT @@GLOBAL.innodb_purge_rseg_truncate_frequency;
+-----------------------------------------------+
| @@GLOBAL.innodb_purge_rseg_truncate_frequency |
+-----------------------------------------------+
|                                           128 |
+-----------------------------------------------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.innodb_purge_rseg_truncate_frequency=1;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@GLOBAL.innodb_purge_rseg_truncate_frequency;
+-----------------------------------------------+
| @@GLOBAL.innodb_purge_rseg_truncate_frequency |
+-----------------------------------------------+
|                                             1 |
+-----------------------------------------------+
1 row in set (0.00 sec)
```

1.  通常在繁忙的系统上，至少会启动一个清除操作，并且截断将已经开始。如果您在自己的机器上练习，可以通过创建一个大事务来启动清除：

```go
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> DELETE FROM employees;
Query OK, 300025 rows affected (16.23 sec)

mysql> ROLLBACK;
Query OK, 0 rows affected (2.38 sec)
```

1.  在删除正在进行时，您可以观察`UNDO`日志文件的增长：

```go
shell> sudo ls -lhtr /var/lib/mysql/undo_00*
-rw-r-----. 1 mysql mysql 19M Oct  7 17:43 /var/lib/mysql/undo_002
-rw-r-----. 1 mysql mysql 16M Oct  7 17:43 /var/lib/mysql/undo_001

shell> sudo ls -lhtr /var/lib/mysql/undo_00*
-rw-r-----. 1 mysql mysql 10M Oct  8 04:52 /var/lib/mysql/undo_001
-rw-r-----. 1 mysql mysql 27M Oct  8 04:52 /var/lib/mysql/undo_002

shell> sudo ls -lhtr /var/lib/mysql/undo_00*
-rw-r-----. 1 mysql mysql 10M Oct  8 04:52 /var/lib/mysql/undo_001
-rw-r-----. 1 mysql mysql 28M Oct  8 04:52 /var/lib/mysql/undo_002

shell> sudo ls -lhtr /var/lib/mysql/undo_00*
-rw-r-----. 1 mysql mysql 10M Oct  8 04:52 /var/lib/mysql/undo_001
-rw-r-----. 1 mysql mysql 29M Oct  8 04:52 /var/lib/mysql/undo_002

shell> sudo ls -lhtr /var/lib/mysql/undo_00*
-rw-r-----. 1 mysql mysql 10M Oct  8 04:52 /var/lib/mysql/undo_001
-rw-r-----. 1 mysql mysql 29M Oct  8 04:52 /var/lib/mysql/undo_002
```

您可能会注意到`undo_001`被截断为 10MB，而`undo_002`正在增长，以容纳`DELETE`语句的已删除行。

1.  一段时间后，您可能会注意到`unod_002`也被截断为 10MB：

```go
shell> sudo ls -lhtr /var/lib/mysql/undo_00*
-rw-r-----. 1 mysql mysql 10M Oct  8 04:52 /var/lib/mysql/undo_001
-rw-r-----. 1 mysql mysql 10M Oct  8 04:54 /var/lib/mysql/undo_002
```

1.  一旦您已经减少了`UNDO`表空间，将`innodb_purge_rseg_truncate_frequency`设置为默认值`128`：

```go
mysql> SELECT @@GLOBAL.innodb_purge_rseg_truncate_frequency;
+-----------------------------------------------+
| @@GLOBAL.innodb_purge_rseg_truncate_frequency |
+-----------------------------------------------+
|                                             1 |
+-----------------------------------------------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.innodb_purge_rseg_truncate_frequency=128;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@GLOBAL.innodb_purge_rseg_truncate_frequency;
+-----------------------------------------------+
| @@GLOBAL.innodb_purge_rseg_truncate_frequency |
+-----------------------------------------------+
|                                           128 |
+-----------------------------------------------+
1 row in set (0.01 sec)
```

# 管理通用表空间

直到 MySQL 8 之前，有两种类型的表空间：系统表空间和单独的表空间。这两种类型都有优点和缺点。为了克服缺点，MySQL 8 引入了通用表空间。与系统表空间类似，通用表空间是可以存储多个表数据的共享表空间。但是，您可以对通用表空间进行精细控制。较少的通用表空间中的多个表消耗的表空间元数据比在单独的文件表表空间中的相同数量的表少。

限制如下：

+   与系统表空间类似，截断或删除存储在通用表空间中的表会在通用表空间的`.ibd`数据文件中创建内部的可用空间，该空间只能用于新的`InnoDB`数据。与文件表表空间一样，空间不会释放回操作系统。

+   通用表空间不支持属于通用表空间的表的可传输表空间。

在本节中，您将学习如何创建通用表空间以及向其中添加和删除表。

**实际用法：**

最初，`InnoDB`维护一个包含表结构的`.frm`文件。MySQL 需要打开和关闭`.frm`文件，这会降低性能。使用 MySQL 8，`.frm`文件被删除，所有的元数据都使用事务性`数据字典`处理。这使得可以使用通用表空间。

假设您正在为每个客户单独创建模式，并且每个客户都有数百个表的 SaaS 或多租户中使用 MySQL 5.7 或更早版本。如果您的客户增长，您将注意到性能问题。但是，随着 MySQL 8 中`.frm`文件的删除，性能得到了极大改善。此外，您可以为每个模式（客户）创建单独的表空间。

# 如何做...

让我们开始创建它。

# 创建通用表空间

您可以在 MySQL 的`数据目录`内或外创建通用表空间。

要在 MySQL 的`数据目录`中创建一个：

```go
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;
Query OK, 0 rows affected (0.02 sec)
```

要在外部创建表空间，请将新磁盘挂载到`/var/lib/mysql_general_ts`并将所有权更改为`mysql`：

```go
shell> sudo chown mysql:mysql /var/lib/mysql_general_ts

mysql> CREATE TABLESPACE `ts2` ADD DATAFILE '/var/lib/mysql_general_ts/ts2.ibd' Engine=InnoDB;Query OK, 0 rows affected (0.02 sec)
```

# 向通用表空间添加表

在创建表时，您可以将表添加到表空间中，或者可以运行`ALTER`命令将表从一个表空间移动到另一个表空间：

```go
mysql> CREATE TABLE employees.table_gen_ts1 (id INT PRIMARY KEY) TABLESPACE ts1;
Query OK, 0 rows affected (0.01 sec)
```

假设您想将`employees`表移动到`TABLESPACE ts2`：

```go
mysql> USE employees;
Database changed

mysql> ALTER TABLE employees TABLESPACE ts2;
Query OK, 0 rows affected (3.93 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

您可以注意到`ts2.ibd`文件的增加：

```go
shell> sudo ls -lhtr /var/lib/mysql_general_ts/ts2.ibd
-rw-r-----. 1 mysql mysql 32M Oct  8 17:07 /var/lib/mysql_general_ts/ts2.ibd
```

# 在表空间之间移动非分区表

您可以按以下方式移动表：

1.  这是如何将表从一个通用表空间移动到另一个通用表空间。

假设您想将`employees`表从`ts2`移动到`ts1`：

```go
mysql> ALTER TABLE employees TABLESPACE ts1;
Query OK, 0 rows affected (3.83 sec)
Records: 0  Duplicates: 0  Warnings: 0

shell> sudo ls -lhtr /var/lib/mysql/ts1.ibd 
-rw-r-----. 1 mysql mysql 32M Oct  8 17:16 /var/lib/mysql/ts1.ibd
```

1.  这是如何将表移动到每个文件一个表。

假设您想将`employees`表从`ts1`移动到每个文件一个表：

```go
mysql> ALTER TABLE employees TABLESPACE innodb_file_per_table;
Query OK, 0 rows affected (4.05 sec)
Records: 0  Duplicates: 0  Warnings: 0

shell> sudo ls -lhtr /var/lib/mysql/employees/employees.ibd 
-rw-r-----. 1 mysql mysql 32M Oct  8 17:18 /var/lib/mysql/employees/employees.ibd
```

1.  这是如何将表移动到系统表空间。

假设您想将`employees`表从每个文件一个表移动到系统表空间：

```go
mysql> ALTER TABLE employees TABLESPACE innodb_system;
Query OK, 0 rows affected (5.28 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

# 在通用表空间中管理分区表

您可以在多个表空间中创建具有分区的表：

```go
mysql> CREATE TABLE table_gen_part_ts1 (id INT, value varchar(100)) ENGINE = InnoDB
       PARTITION BY RANGE(id) (
        PARTITION p1 VALUES LESS THAN (1000000) TABLESPACE ts1,
        PARTITION p2 VALUES LESS THAN (2000000) TABLESPACE ts2,
        PARTITION p3 VALUES LESS THAN (3000000) TABLESPACE innodb_file_per_table,
        PARTITION pmax VALUES LESS THAN (MAXVALUE) TABLESPACE innodb_system);
Query OK, 0 rows affected (0.19 sec)
```

您可以在另一个表空间中添加新的分区，或者如果您没有提及任何内容，它将在表的默认表空间中创建。对分区表执行的`ALTER TABLE tbl_name TABLESPACE tablespace_name`操作只会修改表的默认表空间。它不会移动表分区。但是，在更改默认表空间之后，重建表的操作（例如使用`ALGORITHM=COPY`的`ALTER TABLE`操作）将分区移动到默认表空间，如果没有使用`TABLESPACE`子句显式定义另一个表空间。

如果您希望在表空间之间移动分区，则需要对分区进行`REORGANIZE`。例如，您想将分区`p3`移动到`ts2`：

```go
mysql> ALTER TABLE table_gen_part_ts1 REORGANIZE PARTITION p3 INTO (PARTITION p3 VALUES LESS THAN (3000000) TABLESPACE ts2);
```

# 删除通用表空间

您可以使用`DROP TABLESPACE`命令删除表空间。但是，该表空间内的所有表应该被删除或移动：

```go
mysql> DROP TABLESPACE ts2;
ERROR 3120 (HY000): Tablespace `ts2` is not empty.
```

在删除之前，您必须将`table_gen_part_ts1`表的`ts2`表空间中的分区`p2`和`p3`移动到其他表空间：

```go
mysql> ALTER TABLE table_gen_part_ts1 REORGANIZE PARTITION p2 INTO (PARTITION p2 VALUES LESS THAN (3000000) TABLESPACE ts1);

mysql> ALTER TABLE table_gen_part_ts1 REORGANIZE PARTITION p3 INTO (PARTITION p3 VALUES LESS THAN (3000000) TABLESPACE ts1);
```

现在您可以删除表空间：

```go
mysql> DROP TABLESPACE ts2;
Query OK, 0 rows affected (0.01 sec)
```

# InnoDB 表的压缩

您可以创建数据以压缩形式存储的表。压缩可以帮助提高原始性能和可伸缩性。压缩意味着在磁盘和内存之间传输的数据更少，并且在磁盘和内存中占用的空间更少。

根据 MySQL 文档：

<q class="calibre48">“因为处理器和缓存内存的速度增加比磁盘存储设备更快，许多工作负载受限于磁盘。数据压缩使数据库大小更小，减少 I/O，提高吞吐量，代价是增加 CPU 利用率。压缩对于读密集型应用特别有价值，在具有足够 RAM 以将经常使用的数据保留在内存中的系统上。对于具有辅助索引的表，好处尤为明显，因为索引数据也被压缩。”</q>

要启用压缩，需要使用`ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE`选项创建或更改表。您可以变化`KEY_BLOCK_SIZE`参数，该参数在磁盘上使用比配置的`innodb_page_size`值更小的页面大小。如果表在系统表空间中，则压缩将无法工作。

要在通用表空间中创建压缩表，必须为创建表空间时指定的通用表空间定义`FILE_BLOCK_SIZE`。`FILE_BLOCK_SIZE`值必须是与`innodb_page_size`值相关的有效压缩页面大小，并且由`CREATE TABLE`或`ALTER TABLE KEY_BLOCK_SIZE`子句定义的压缩表的页面大小必须等于`FILE_BLOCK_SIZE/1024`。

在缓冲池中，压缩数据以小页面的形式保存，页面大小基于`KEY_BLOCK_SIZE`值。对于提取或更新列值，MySQL 还在缓冲池中创建一个包含未压缩数据的未压缩页面。在缓冲池中，对未压缩页面的任何更新也会被重写回等效的压缩页面。您可能需要调整缓冲池的大小，以容纳压缩和未压缩页面的额外数据，尽管在需要空间时未压缩页面会从缓冲池中驱逐，然后在下一次访问时再次解压缩。

**何时使用压缩？**

一般来说，压缩最适用于包含合理数量的字符串列的表，以及数据被读取的频率远远高于写入的情况。因为没有保证的方法来预测压缩是否对特定情况有益，所以始终要使用特定的工作负载和数据集在代表性配置上进行测试。

# 如何做...

您需要选择参数`KEY_BLOCK_SIZE`。`innodb_page_size`为 16,000；理想情况下，一半为 8,000，这是一个很好的起点。要调整压缩，请参阅[`dev.mysql.com/doc/refman/8.0/en/innodb-compression-tuning.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-tuning.html)。

# 为`file_per_table`表启用压缩

1.  确保启用了`file_per_table`：

```go
mysql> SET GLOBAL innodb_file_per_table=1;
```

1.  在创建语句中指定`ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8`：

```go
mysql> CREATE TABLE compressed_table (id INT PRIMARY KEY) ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
Query OK, 0 rows affected (0.07 sec)
```

如果表已经存在，可以执行`ALTER`：

```go
mysql> ALTER TABLE event_history ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
Query OK, 0 rows affected (0.67 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

如果尝试压缩位于系统表空间中的表，将会出现错误：

```go
mysql> ALTER TABLE employees ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
ERROR 1478 (HY000): InnoDB: Tablespace `innodb_system` cannot contain a COMPRESSED table
```

# 为`file_per_table`表禁用压缩

要禁用压缩，执行`ALTER`表并指定`ROW_FORMAT=DYNAMIC`或`ROW_FORMAT=COMPACT`，然后是`KEY_BLOCK_SIZE=0`。

例如，如果不希望在`event_history`表上使用压缩：

```go
mysql> ALTER TABLE event_history ROW_FORMAT=DYNAMIC KEY_BLOCK_SIZE=0;
Query OK, 0 rows affected (0.53 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

# 为通用表空间启用压缩

首先，您需要通过提及`FILE_BLOCK_SIZE`来创建一个压缩表空间；您不能更改表空间的`FILE_BLOCK_SIZE`。

如果要创建压缩表，需要在启用压缩的通用表空间中创建表；此外，`KEY_BLOCK_SIZE`必须等于`FILE_BLOCK_SIZE/1024`。如果不提及`KEY_BLOCK_SIZE`，则该值将自动从`FILE_BLOCK_SIZE`中获取。

您可以创建多个具有不同`FILE_BLOCK_SIZE`值的压缩通用表空间，并将表添加到所需的表空间中：

1.  创建一个通用的压缩表空间。您可以创建一个`FILE_BLOCK_SIZE`为 8k 的表空间，另一个为 4k 的表空间，并将所有`KEY_BLOCK_SIZE`为 8 的表移动到 8k，将 4 移动到 4k：

```go
mysql> CREATE TABLESPACE `ts_8k` ADD DATAFILE 'ts_8k.ibd' FILE_BLOCK_SIZE = 8192 Engine=InnoDB;
Query OK, 0 rows affected (0.01 sec)

mysql> CREATE TABLESPACE `ts_4k` ADD DATAFILE 'ts_4k.ibd' FILE_BLOCK_SIZE = 4096 Engine=InnoDB;
Query OK, 0 rows affected (0.04 sec)
```

1.  通过提及`ROW_FORMAT=COMPRESSED`在这些表空间中创建压缩表：

```go
mysql> CREATE TABLE compress_table_1_8k (id INT PRIMARY KEY) TABLESPACE ts_8k ROW_FORMAT=COMPRESSED;
Query OK, 0 rows affected (0.01 sec)
```

如果不提及`ROW_FORMAT=COMPRESSED`，将会出现错误：

```go
mysql> CREATE TABLE compress_table_2_8k (id INT PRIMARY KEY) TABLESPACE ts_8k;
ERROR 1478 (HY000): InnoDB: Tablespace `ts_8k` uses block size 8192 and cannot contain a table with physical page size 16384
```

可选地，您可以提及`KEY_BLOCK_SIZE=FILE_BLOCK_SIZE/1024`：

```go
mysql> CREATE TABLE compress_table_8k (id INT PRIMARY KEY) TABLESPACE ts_8k ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=8;
Query OK, 0 rows affected (0.01 sec)
```

如果提及的内容不是`FILE_BLOCK_SIZE/1024`，将会出现错误：

```go
mysql> CREATE TABLE compress_table_2_8k (id INT PRIMARY KEY) TABLESPACE ts_8k ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=4;
ERROR 1478 (HY000): InnoDB: Tablespace `ts_8k` uses block size 8192 and cannot contain a table with physical page size 4096
```

1.  只有`KEY_BLOCK_SIZE`匹配时，才能将表从`file_per_table`表空间移动到压缩通用表空间。否则，将会出现错误：

```go
mysql> CREATE TABLE compress_tables_4k (id INT PRIMARY KEY) TABLESPACE innodb_file_per_table ROW_FORMAT=COMPRESSED KEY_BLOCK_SIZE=4;
Query OK, 0 rows affected (0.02 sec)

mysql> ALTER TABLE compress_tables_4k TABLESPACE ts_4k;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE compress_tables_4k TABLESPACE ts_8k;
ERROR 1478 (HY000): InnoDB: Tablespace `ts_8k` uses block size 8192 and cannot contain a table with physical page size 4096
```
