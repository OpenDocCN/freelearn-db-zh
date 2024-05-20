# 第八章：恢复数据

在本章中，我们将介绍以下配方：

+   从 mysqldump 和 mysqlpump 中恢复

+   使用 myloader 从 mydumper 恢复

+   从平面文件备份中恢复

+   执行时间点恢复

# 介绍

在本章中，您将了解各种备份恢复方法。假设备份和二进制日志在服务器上可用。

# 从 mysqldump 和 mysqlpump 中恢复

逻辑备份工具`mysqldump`和`mysqlpump`将数据写入单个文件。

# 如何做...

登录到备份可用的服务器：

```go
shell> cat /backups/full_backup.sql | mysql -u <user> -p
or
shell> mysql -u <user> -p < /backups/full_backup.sql
```

要在远程服务器上恢复，可以提到`-h <hostname>`选项：

```go
shell> cat /backups/full_backup.sql | mysql -u <user> -p -h <remote_hostname>
```

在恢复备份时，备份语句将记录到二进制日志中，这可能会减慢恢复过程。如果不希望恢复过程写入二进制日志，可以在会话级别使用`SET SQL_LOG_BIN=0;`选项禁用它：

```go
shell> (echo "SET SQL_LOG_BIN=0;";cat /backups/full_backup.sql) | mysql -u <user> -p -h <remote_hostname>
```

或使用：

```go
mysql> SET SQL_LOG_BIN=0; SOURCE full_backup.sql
```

# 还有更多...

1.  由于备份恢复需要很长时间，建议在屏幕会话内启动恢复过程，以便即使失去与服务器的连接，恢复也将继续。

1.  有时，在恢复过程中可能会出现故障。如果将`--force`选项传递给 MySQL，恢复将继续：

```go
shell> (echo "SET SQL_LOG_BIN=0;";cat /backups/full_backup.sql) | mysql -u <user> -p -h <remote_hostname> -f
```

# 使用 myloader 从 mydumper 恢复

`myloader`是用于使用`mydumper`获取的备份的多线程恢复的工具。 `myloader`与`mydumper`一起提供，您无需单独安装它。在本节中，您将学习恢复备份的各种方法。

# 如何做...

`myloader`的常见选项是要连接的 MySQL 服务器的主机名（默认值为`localhost`），用户名，密码和端口。

# 恢复完整数据库

```go
shell> myloader --directory=/backups --user=<user> --password=<password> --queries-per-transaction=5000 --threads=8 --compress-protocol --overwrite-tables
```

选项解释如下：

+   `--overwrite-tables`：此选项如果表已经存在，则删除表

+   `--compress-protocol`：此选项在 MySQL 连接上使用压缩

+   `--threads`：此选项指定要使用的线程数；默认值为`4`

+   `--queries-per-transaction`：此选项指定每个事务的查询数；默认值为`1000`

+   `--directory`：指定要导入的转储目录

# 恢复单个数据库

您可以指定`--source-db <db_name>`仅恢复单个数据库。

假设您要恢复`company`数据库：

```go
shell> myloader --directory=/backups --queries-per-transaction=5000 --threads=6 --compress-protocol --user=<user> --password=<password> --source-db company --overwrite-tables
```

# 恢复单个表

`mydumper`将每个表的备份写入单独的`.sql`文件。您可以拾取`.sql`文件并恢复：

```go
shell> mysql -u <user> -p<password> -h <hostname> company -A -f < company.payments.sql
```

如果表被分成多个块，可以将所有块和与表相关的信息复制到一个目录中并指定位置。

复制所需的文件：

```go
shell> sudo cp /backups/employee_table_chunks/employees.employees.* \
/backups/employee_table_chunks/employees.employees-schema.sql \
/backups/employee_table_chunks/employees-schema-create.sql \
/backups/employee_table_chunks/metadata \
/backups/single_table/
```

使用`myloader`进行加载；它将自动检测块并加载它们：

```go
shell> myloader --directory=/backups/single_table/ --queries-per-transaction=50000 --threads=6 --compress-protocol --overwrite-tables
```

# 从平面文件备份中恢复

从平面文件恢复需要停止 MySQL 服务器，替换所有文件，更改权限，然后启动 MySQL。

# 如何做...

1.  停止 MySQL 服务器：

```go
 shell> sudo systemctl stop mysql
```

1.  将文件移动到`数据目录`：

```go
 shell> sudo mv /backup/mysql /var/lib
```

1.  更改所有权为`mysql`：

```go
 shell> sudo chown -R mysql:mysql /var/lib/mysql
```

1.  启动 MySQL：

```go
 shell> sudo systemctl start mysql
```

为了最小化停机时间，如果磁盘上有足够的空间，可以将备份复制到`/var/lib/mysql2`。然后停止 MySQL，重命名目录，然后启动服务器：

```go
shell> sudo mv /backup/mysql /var/lib/mysql2
shell> sudo systemctl stop mysql
shell> sudo mv /var/lib/mysql2 /var/lib/mysql
shell> sudo chown -R mysql:mysql /var/lib/mysql
shell> sudo systemctl start mysql
```

# 执行时间点恢复

一旦完整备份恢复完成，您需要恢复二进制日志以进行时间点恢复。备份提供了直到备份可用的二进制日志坐标。

如第七章中所解释的，*备份*，在*锁定备份实例*部分，您应该根据`--dump-slave`或`--master-data`选项从正确的服务器选择二进制日志备份`mysqldump`。

# 如何做...

让我们深入了解如何做。这里有很多东西要学习。

# mysqldump 或 mysqlpump

二进制日志信息存储在 SQL 文件中，作为基于您传递给`mysqldump`/`mysqlpump`的选项的`CHANGE MASTER TO`命令。

1.  如果您使用了`--master-data`，您应该使用从服务器的二进制日志：

```go
shell> head -30 /backups/dump.sql
-- MySQL dump 10.13  Distrib 8.0.3-rc, for Linux (x86_64)
--
-- Host: localhost    Database: 
-- ------------------------------------------------------
-- Server version 8.0.3-rc-log
/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!50606 SET @OLD_INNODB_STATS_AUTO_RECALC=@@INNODB_STATS_AUTO_RECALC */;
/*!50606 SET GLOBAL INNODB_STATS_AUTO_RECALC=OFF */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
--
-- Position to start replication or point-in-time recovery from --
CHANGE MASTER TO MASTER_LOG_FILE='server1.000008', MASTER_LOG_POS=154;
```

在这种情况下，您应该从从服务器上位置`154`的`server1.000008`文件开始恢复。

```go
shell> mysqlbinlog --start-position=154 --disable-log-bin /backups/binlogs/server1.000008 | mysql -u<user> -p -h <host> -f
```

1.  如果您使用了`--dump-slave`，您应该使用主服务器的二进制日志：

```go
--
-- Position to start replication or point-in-time recovery from (the master of this slave)
--
CHANGE MASTER TO MASTER_LOG_FILE='centos7-bin.000001', MASTER_LOG_POS=463;
```

在这种情况下，您应该从主服务器上位置`463`的`centos7-bin.000001`文件开始恢复。

```go
shell> mysqlbinlog --start-position=463  --disable-log-bin /backups/binlogs/centos7-bin.000001 | mysql -u<user> -p -h <host> -f
```

# mydumper

二进制日志信息可在元数据中找到。

```go
shell> sudo cat /backups/metadata Started dump at: 2017-08-26 06:26:19
SHOW MASTER STATUS:
 Log: server1.000012
 Pos: 154
</span> GTID:
SHOW SLAVE STATUS:
 Host: 35.186.158.188
 Log: centos7-bin.000001
 Pos: 463
 GTID:
Finished dump at: 2017-08-26 06:26:24 
```

如果您已经从从服务器上获取了二进制日志备份，您应该从位置`154`的`server1.000012`文件开始恢复（`SHOW MASTER STATUS`）：

```go
shell> mysqlbinlog --start-position=154  --disable-log-bin /backups/binlogs/server1.000012 | mysql -u<user> -p -h <host> -f
```

如果您从主服务器上有二进制日志备份，您应该从位置`463`的`centos7-bin.000001`文件开始恢复（`SHOW SLAVE STATUS`）：

```go
shell> mysqlbinlog --start-position=463  --disable-log-bin /backups/binlogs/centos7-bin.000001 | mysql -u<user> -p -h <host> -f
```
