# 第六章：二进制日志记录

在本章中，我们将介绍以下配方：

+   使用二进制日志记录

+   二进制日志格式

+   从二进制日志中提取语句

+   忽略数据库以写入二进制日志

+   重新定位二进制日志

# 介绍

二进制日志包含对数据库的所有更改的记录，包括数据和结构。二进制日志不用于不修改数据的语句，如`SELECT`或`SHOW`。运行启用二进制日志的服务器会略微影响性能。二进制日志是崩溃安全的。只有完整的事件或事务才会被记录或读取。

为什么要使用二进制日志？

+   **复制**：您可以使用二进制日志将对服务器所做的更改流式传输到另一个服务器。从服务器充当镜像副本，并可用于分发负载。接受写入的服务器称为主服务器，镜像副本服务器称为从服务器。

+   **时间点恢复**：假设您在星期日的 00:00 进行备份，而您的数据库在星期日的 8:00 崩溃。使用备份，您可以恢复到星期日的 00:00。使用二进制日志，您可以回放它们，以恢复到 08:00。

# 使用二进制日志记录

要启用`binlog`，必须设置`log_bin`和`server_id`并重新启动服务器。您可以在`log_bin`中提及路径和基本名称。例如，`log_bin`设置为`/data/mysql/binlogs/server1`，则二进制日志存储在`/data/mysql/binlogs`文件夹中，名称为`server1.000001`，`server1.000002`等。服务器每次启动或刷新日志或当前日志大小达到`max_binlog_size`时，都会创建一个新文件。它维护`server1.index`文件，其中包含每个二进制日志的位置。

# 如何做...

让我们看看如何处理日志。我相信您会喜欢学习它们。

# 启用二进制日志记录

1.  启用二进制日志记录并设置`server_id`。在您喜欢的编辑器中打开 MySQL `config`文件并追加以下行。选择`server_id`，使其对您基础架构中的每个 MySQL 服务器都是唯一的。

您还可以简单地将`log_bin`变量放在`my.cnf`中，而不设置任何值。在这种情况下，二进制日志将在`data directory`目录中创建，并使用`hostname`作为其名称。

```sql
shell> sudo vi /etc/my.cnf
[mysqld]
log_bin = /data/mysql/binlogs/server1
server_id = 100
```

1.  重新启动 MySQL 服务器：

```sql
shell> sudo systemctl restart mysql
```

1.  验证是否创建了二进制日志：

```sql
mysql> SHOW VARIABLES LIKE 'log_bin%';
+---------------------------------+-----------------------------------+
| Variable_name                   | Value                             |
+---------------------------------+-----------------------------------+
| log_bin                         | ON                                |
| log_bin_basename                | /data/mysql/binlogs/server1       |
| log_bin_index                   | /data/mysql/binlogs/server1.index |
| log_bin_trust_function_creators | OFF                               |
| log_bin_use_v1_row_events       | OFF                               |
+---------------------------------+-----------------------------------+
5 rows in set (0.00 sec)
```

```sql
mysql> SHOW MASTER LOGS;
+----------------+-----------+
| Log_name       | File_size |
+----------------+-----------+
| server1.000001 | 154       |
+----------------+-----------+
1 row in set (0.00 sec)
```

```sql
shell> sudo ls -lhtr /data/mysql/binlogs
total 8.0K
-rw-r----- 1 mysql mysql 34 Aug 15 05:01 server1.index
-rw-r----- 1 mysql mysql 154 Aug 15 05:01 server1.000001
```

1.  执行`SHOW BINARY LOGS;`或`SHOW MASTER LOGS;`以显示服务器的所有二进制日志。

1.  执行`SHOW MASTER STATUS;`命令以获取当前二进制日志位置：

```sql
mysql> SHOW MASTER STATUS;
+----------------+----------+--------------+------------------+------------------+++++++++++++++++++++++++++++++++++++-+
| File           | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------+----------+--------------+------------------+-------------------+
| server1.000002 |     3273 |              |                  |                   |
+----------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

一旦`server1.000001`达到`max_binlog_size`（默认为 1GB），将创建一个新文件`server1.000002`并将其添加到`server1.index`中。您可以配置使用`SET @@global.max_binlog_size=536870912`动态设置`max_binlog_size`。

# 禁用会话的二进制日志记录

可能存在不希望将语句复制到其他服务器的情况。为此，可以使用以下命令禁用该会话的二进制日志记录：

```sql
mysql> SET SQL_LOG_BIN = 0;
```

在执行前一个语句后记录的所有 SQL 语句不会记录到二进制日志中。这仅适用于该会话。

要启用，可以执行以下操作：

```sql
mysql> SET SQL_LOG_BIN = 1;
```

# 移至下一个日志

您可以使用`FLUSH LOGS`命令关闭当前的二进制日志并打开一个新的：

```sql
mysql> SHOW BINARY LOGS;
+----------------+-----------+
| Log_name       | File_size |
+----------------+-----------+
| server1.000001 |       154 |
+----------------+-----------+
1 row in set (0.00 sec)

mysql> FLUSH LOGS;
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW BINARY LOGS;
+----------------+-----------+
| Log_name       | File_size |
+----------------+-----------+
| server1.000001 |  198      |
| server1.000002 |  154      |
+----------------+-----------+
2 rows in set (0.00 sec)

```

# 过期二进制日志

基于写入次数，二进制日志会占用大量空间。将它们保持不变可能会在短时间内填满磁盘。清理它们是至关重要的：

1.  使用`binlog_expire_logs_seconds`和`expire_logs_days`设置日志的到期时间。

如果要设置以天为单位的到期时间，请设置`expire_logs_days`。例如，如果要删除两天前的所有二进制日志，`SET @@global.expire_logs_days=2`。将值设置为`0`会禁用自动到期。

如果您想要更细粒度，可以使用`binlog_expire_logs_seconds`变量，该变量设置二进制日志的过期时间（以秒为单位）。

此变量和`expire_logs_days`的效果是累积的。例如，如果`expire_logs_days`是`1`，`binlog_expire_logs_seconds`是`43200`，那么二进制日志将每 1.5 天清除一次。这与将`binlog_expire_logs_seconds`设置为`129600`并将`expire_logs_days`设置为 0 的结果相同。在 MySQL 8.0 中，`binlog_expire_logs_seconds`和`expire_logs_days`必须都设置为 0 才能禁用二进制日志的自动清除。

1.  要手动清除日志，请执行`PURGE BINARY LOGS TO '<file_name>'`。例如，如果有文件如`server1.000001`，`server1.000002`，`server1.000003`和`server1.000004`，如果您执行`PURGE BINARY LOGS TO 'server1.000004'`，则所有文件直到`server1.000003`将被删除，`server1.000004`不会被触及：

```sql
mysql> SHOW BINARY LOGS;
+----------------+-----------+
| Log_name      | File_size |
+----------------+-----------+
| server1.000001 |       198 |
| server1.000002 |       198 |
| server1.000003 |       198 |
| server1.000004 |       154 |
+----------------+-----------+
4 rows in set (0.00 sec)
```

```sql
mysql> PURGE BINARY LOGS TO 'server1.000004';
Query OK, 0 rows affected (0.00 sec)

```

```sql
mysql> SHOW BINARY LOGS;
+----------------+-----------+
| Log_name       | File_size |
+----------------+-----------+
| server1.000004 |       154 |
+----------------+-----------+
1 row in set (0.00 sec)

```

您还可以执行命令`PURGE BINARY LOGS BEFORE '2017-08-03 15:45:00'`，而不是指定日志文件。您还可以使用单词`MASTER`而不是`BINARY`。

`mysql> PURGE MASTER LOGS TO 'server1.000004'`也可以。

1.  要删除所有二进制日志并重新开始，请执行`RESET MASTER`：

```sql
mysql> SHOW BINARY LOGS;
+----------------+-----------+
| Log_name       | File_size |
+----------------+-----------|
| server1.000004 |       154 |
+----------------+-----------+
1 row in set (0.00 sec)
```

```sql
mysql> RESET MASTER;
Query OK, 0 rows affected (0.01 sec)
```

```sql
mysql> SHOW BINARY LOGS;
+----------------+-----------+
| Log_name       | File_size |
+----------------+-----------+
| server1.000001 |       154 |
+----------------+-----------+
1 row in set (0.00 sec)
```

如果您正在使用复制，清除或过期日志是一种非常不安全的方法。清除二进制日志的安全方法是使用`mysqlbinlogpurge`脚本，这将在第十二章 *管理日志*中介绍。

# 二进制日志格式

二进制日志可以以三种格式写入：

1.  `STATEMENT`：实际的 SQL 语句被记录。

1.  `ROW`：对每一行所做的更改都会被记录。

例如，更新语句更新了 10 行，所有 10 行的更新信息都被写入日志。而在基于语句的复制中，只有更新语句被写入。默认格式是`ROW`。

1.  `MIXED`：MySQL 根据需要从`STATEMENT`切换到`ROW`。

有些语句在不同服务器上执行时可能会导致不同的结果。例如，`UUID()`函数的输出因服务器而异。这些语句称为非确定性语句，对于基于语句的复制来说是不安全的。在这种情况下，当您设置`MIXED`格式时，MySQL 服务器会切换到基于行的格式。

请参阅[`dev.mysql.com/doc/refman/8.0/en/binary-log-mixed.html`](https://dev.mysql.com/doc/refman/8.0/en/binary-log-mixed.html)了解有关不安全语句和切换发生的更多信息。

# 如何做...

您可以使用动态变量`binlog_format`来设置格式，该变量具有全局和会话范围。在全局级别设置它会使所有客户端使用指定的格式：

```sql
mysql> SET GLOBAL binlog_format = 'STATEMENT';
```

或者：

```sql
mysql> SET GLOBAL binlog_format = 'ROW'; 
```

请参阅[`dev.mysql.com/doc/refman/8.0/en/replication-sbr-rbr.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-sbr-rbr.html)了解各种格式的优缺点。

1.  MySQL 8.0 使用版本 2 的二进制日志行事件，这些事件不能被 MySQL 5.6.6 之前的版本读取。将`log-bin-use-v1-row-events`设置为`1`以使用版本 1，以便可以被 MySQL 5.6.6 之前的版本读取。默认值为`0`。

```sql
mysql> SET @@GLOBAL.log_bin_use_v1_row_events=0;
```

1.  当您创建存储函数时，必须声明它是确定性的或者不修改数据。否则，它可能对二进制日志记录不安全。默认情况下，要接受`CREATE FUNCTION`语句，必须显式指定`DETERMINISTIC`，`NO SQL`或`READS SQL DATA`中的至少一个。否则会发生错误：

```sql
ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)
```

您可以在例程中写入非确定性语句，并且仍然声明为`DETERMINISTIC`（这不是一个好的做法），如果您想要复制未声明为`DETERMINISTIC`的例程，可以设置`log_bin_trust_function_creators`变量：

`mysql> SET GLOBAL log_bin_trust_function_creators = 1;`

# 另请参阅

请参阅[`dev.mysql.com/doc/refman/8.0/en/stored-programs-logging.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-programs-logging.html)了解有关存储程序如何复制的更多信息。

# 从二进制日志中提取语句

您可以使用（随 MySQL 一起提供的）`mysqlbinlog`实用程序从二进制日志中提取内容并将其应用到其他服务器上。

# 准备工作

使用各种二进制格式执行几个语句。当您将`binlog_format`设置为`GLOBAL`级别时，您必须断开连接并重新连接以获取更改。如果您想保持连接，请设置为`SESSION`级别。

切换到**基于语句的复制**（**SBR**）：

```sql
mysql> SET @@GLOBAL.BINLOG_FORMAT='STATEMENT';
Query OK, 0 rows affected (0.00 sec)
```

更新几行：

```sql
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE salaries SET salary=salary*2 WHERE emp_no<10002;
Query OK, 18 rows affected (0.00 sec)
Rows matched: 18  Changed: 18  Warnings: 0

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)
```

切换到**基于行的复制**（**RBR**）：

```sql
mysql> SET @@GLOBAL.BINLOG_FORMAT='ROW';
Query OK, 0 rows affected (0.00 sec)
```

更新几行：

```sql
mysql> BEGIN;Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE salaries SET salary=salary/2 WHERE emp_no<10002;Query OK, 18 rows affected (0.00 sec)
Rows matched: 18  Changed: 18  Warnings: 0

mysql> COMMIT;Query OK, 0 rows affected (0.00 sec)
```

切换到`MIXED`格式：

```sql
mysql> SET @@GLOBAL.BINLOG_FORMAT='MIXED';
Query OK, 0 rows affected (0.00 sec)
```

更新几行：

```sql
mysql> BEGIN;Query OK, 0 rows affected (0.00 sec)

mysql> UPDATE salaries SET salary=salary*2 WHERE emp_no<10002;Query OK, 18 rows affected (0.00 sec)
Rows matched: 18  Changed: 18  Warnings: 0

mysql> INSERT INTO departments VALUES('d010',UUID());Query OK, 1 row affected (0.00 sec)

mysql> COMMIT;Query OK, 0 rows affected (0.00 sec)
```

# 如何操作...

要显示`server1.000001`的内容，请执行以下操作：

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001
```

您将获得类似以下内容的输出：

```sql
# at 226
#170815 12:49:24 server id 200  end_log_pos 312 CRC32 0x9197bf88  Query thread_id=5 exec_time=0 error_code=0
BINLOG '
~
~
```

在第一行中，`# at`后面的数字表示二进制日志文件中事件的起始位置（文件偏移量）。第二行包含语句在服务器上开始的时间戳。时间戳后面是`server id`、`end_log_pos`、`thread_id`、`exec_time`和`error_code`。

+   `server id`：是事件发生的服务器的`server_id`值（在本例中为`200`）。

+   `end_log_pos`：是下一个事件的起始位置。

+   `thread_id`：指示执行事件的线程。

+   `exec_time`：是在主服务器上执行事件所花费的时间。在从服务器上，它是从从服务器上的结束执行时间减去主服务器上的开始执行时间的差异。这个差异作为指示从服务器落后主服务器的程度的指标。

+   `error_code`：指示执行事件的结果。零表示没有发生错误。

# 观察

1.  您在基于语句的复制中执行了`UPDATE`语句，并且相同的语句记录在二进制日志中。除了服务器外，会话变量也保存在二进制日志中，以在从服务器上复制相同的行为：

```sql
# at 226
#170815 13:28:38 server id 200  end_log_pos 324 CRC32 0x9d27fc78  Query thread_id=8 exec_time=0 error_code=0
SET TIMESTAMP=1502803718/*!*/;
SET @@session.pseudo_thread_id=8/*!*/;
SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0,
@@session.unique_checks=1, @@session.autocommit=1/*!*/;
SET @@session.sql_mode=1436549152/*!*/;
SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
/*!\C utf8 *//*!*/;
SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=255/*!*/;
SET @@session.lc_time_names=0/*!*/;
SET @@session.collation_database=DEFAULT/*!*/;

```

```sql
BEGIN
/*!*/;
# at 324
#170815 13:28:38 server id 200  end_log_pos 471 CRC32 0x35c2ba45  Query thread_id=8 exec_time=0 error_code=0
use `employees`/*!*/;
SET TIMESTAMP=1502803718/*!*/;
UPDATE salaries SET salary=salary*2 WHERE emp_no<10002 /*!*/;
# at 471
#170815 13:28:40 server id 200  end_log_pos 502 CRC32 0xb84cfeda  Xid = 53
COMMIT/*!*/;

```

1.  当使用基于行的复制时，保存的不是语句，而是以二进制格式保存的`ROW`，您无法阅读。此外，您可以观察到，单个更新语句生成了如此多的数据。查看*提取行事件显示*部分，该部分解释了如何查看二进制格式。

```sql
BEGIN
/*!*/;
# at 660
#170815 13:29:02 server id 200  end_log_pos 722 CRC32 0xe0a2ec74  Table_map:`employees`.`salaries` mapped to number 165
# at 722
#170815 13:29:02 server id 200  end_log_pos 1298 CRC32 0xf0ef8b05  Update_rows: table id 165 flags: STMT_END_F

BINLOG '
HveSWRPIAAAAPgAAANICAAAAAKUAAAAAAAEACWVtcGxveWVlcwAIc2FsYXJpZXMABAMDCgoAAAEBAHTsouA=HveSWR/IAAAAQAIAABIFAAAAAKUAAAAAAAEAAgAE///wEScAAFSrAwDahA/ahg/wEScAAKrVAQDahA/ahg/wEScAAFjKAwDahg/ZiA/wEScAACzlAQDahg/ZiA/wEScAAGgIBADZiA/Zig/wEScAADQEAgDZiA/Zig/wEScAAJAQBADZig/ZjA/wEScAAEgIAgDZig/ZjA/wEScAAEQWBADZjA/Zjg/wEScAACILAgDZjA/Zjg/wEScAABhWBADZjg/YkA/wEScAAAwrAgDZjg/YkA/wEScAAHSJBADYkA/Ykg/wEScAALpEAgDYkA/Ykg/wEScAAFiYBADYkg/YlA/wEScAACxMAgDYkg/YlA/wEScAAGijBADYlA/Ylg/wEScAALRRAgDYlA/Ylg/wEScAAFCxBADYlg/XmA/wEScAAKhYAgDYlg/XmA/wEScAADTiBADXmA/Xmg/wEScAABpxAgDXmA/Xmg/wEScAAATyBADXmg/XnA/wEScAAAJ5AgDXmg/XnA/wEScAACTzBADXnA/Xng/wEScAAJJ5AgDXnA/Xng/wEScAANQuBQDXng/WoA/wEScAAGqXAgDXng/WoA/wEScAAOAxBQDWoA/Wog/wEScAAPCYAgDWoA/Wog/wEScAAKQxBQDWog/WpA/wEScAANKYAgDWog/WpA/wEScAAIAaBgDWpA8hHk7wEScAAEANAwDWpA8hHk7wEScAAIAaBgDSwg8hHk7wEScAAEANAwDSwg8hHk4Fi+/w '/*!*/;
# at 1298
#170815 13:29:02 server id 200  end_log_pos 1329 CRC32 0xa6dac5dc  Xid = 56
COMMIT/*!*/;
```

1.  当使用`MIXED`格式时，`UPDATE`语句被记录为 SQL，而`INSERT`语句以基于行的格式记录，因为`INSERT`具有不确定性的`UUID()`函数：

```sql
BEGIN
/*!*/;
# at 1499
#170815 13:29:27 server id 200  end_log_pos 1646 CRC32 0xc73d68fb  Query thread_id=8 exec_time=0 error_code=0
SET TIMESTAMP=1502803767/*!*/;
UPDATE salaries SET salary=salary*2 WHERE emp_no<10002 /*!*/;
# at 1646
#170815 13:29:50 server id 200  end_log_pos 1715 CRC32 0x03ae0f7e  Table_map: `employees`.`departments` mapped to number 166
# at 1715
#170815 13:29:50 server id 200  end_log_pos 1793 CRC32 0xa43c5dac  Write_rows: table id 166 flags: STMT_END_F
BINLOG 'TveSWRPIAAAARQAAALMGAAAAAKYAAAAAAAMACWVtcGxveWVlcwALZGVwYXJ0bWVudHMAAv4PBP4QoAAAAgP8/wB+D64DTveSWR7IAAAATgAAAAEHAAAAAKYAAAAAAAEAAgAC//wEZDAxMSRkMDNhMjQwZS04MWJkLTExZTctODQxMC00MjAxMGE5NDAwMDKsXTyk '/*!*/;
# at 1793
#170815 13:29:50 server id 200  end_log_pos 1824 CRC32 0x4f63aa2e  Xid = 59
COMMIT/*!*/;
```

提取的日志可以通过管道传输到 MySQL 以重放事件。在重放二进制日志时最好使用 force 选项，因为如果它卡在某一点，它不会停止执行。稍后，您可以找出错误并手动修复数据。

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001 | mysql -f -h <remote_host> -u <username> -p
```

或者您可以保存到文件中，以后执行：

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001 > server1.binlog_extract
shell> cat server1.binlog_extract | mysql -h <remote_host> -u <username> -p
```

# 基于时间和位置提取

您可以通过指定位置从二进制日志中提取部分数据。假设您想进行时间点恢复。假设在`2017-08-19 12:18:00`执行了`DROP DATABASE`命令，并且最新可用的备份是`2017-08-19 12:00:00`，您已经恢复了。现在，您需要从`12:00:01`到`2017-08-19 12:17:00`恢复数据。请记住，如果您提取完整的日志，它也将包含`DROP DATABASE`命令，并且会再次擦除您的数据。

您可以通过`--start-datetime`和`--stop-datatime`选项指定时间窗口来提取数据。

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001 --start-datetime="2017-08-19 00:00:01"  --stop-datetime="2017-08-19 12:17:00" > binlog_extract
```

使用时间窗口的缺点是您将错过灾难发生时发生的事务。为了避免这种情况，您必须使用二进制日志文件中事件的文件偏移量。

一致的备份保存了已备份的二进制日志文件偏移量。一旦备份恢复，您必须从备份提供的偏移量提取二进制日志。您将在下一章中了解更多关于备份的内容。

假设备份给出了偏移量`471`，并且`DROP DATABASE`命令在偏移量`1793`处执行。您可以使用`--start-position`和`--stop-position`选项在偏移量之间提取日志：

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001 --start-position=471  --stop-position=1793 > binlog_extract
```

确保提取的 binlog 中不再出现`DROP DATABASE`命令。

# 基于数据库提取

使用`--database`选项，您可以过滤特定数据库的事件。如果您多次使用此选项，则只会考虑最后一个选项。这对于基于行的复制非常有效。但对于基于语句的复制和`MIXED`，只有在选择默认数据库时才会输出。

以下命令从 employees 数据库中提取事件：

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001 --database=employees > binlog_extract
```

如 MySQL 8 参考手册中所述，假设二进制日志是通过使用基于语句的日志记录执行这些语句创建的：

```sql
mysql>
INSERT INTO test.t1 (i) VALUES(100);
INSERT INTO db2.t2 (j)  VALUES(200);

USE test;
INSERT INTO test.t1 (i) VALUES(101);
INSERT INTO t1 (i) VALUES(102);
INSERT INTO db2.t2 (j) VALUES(201);

USE db2;
INSERT INTO test.t1 (i) VALUES(103);
INSERT INTO db2.t2 (j) VALUES(202);
INSERT INTO t2 (j) VALUES(203);
```

`mysqlbinlog --database=test`不会输出前两个`INSERT`语句，因为没有默认数据库。

它会输出`USE test`后面的三个`INSERT`语句，但不会输出`USE db2`后面的三个`INSERT`语句。

`mysqlbinlog --database=db2`不会输出前两个`INSERT`语句，因为没有默认数据库。

它不会输出`USE` test 后面的三个`INSERT`语句，但会输出`USE db2`后面的三个`INSERT`语句。

# 提取行事件显示

在基于行的复制中，默认情况下显示二进制格式。要查看`ROW`信息，您必须将`--verbose`或`-v`选项传递给`mysqlbinlog`。行事件的二进制格式显示为以`###`开头的行形式的伪 SQL 语句的注释。您可以看到单个`UPDATE`语句被重写为每行的`UPDATE`语句：

```sql
shell>  mysqlbinlog /data/mysql/binlogs/server1.000001 --start-position=660 --stop-position=1298 --verbose
~
~
# at 660
#170815 13:29:02 server id 200  end_log_pos 722 CRC32 0xe0a2ec74     Table_map: `employees`.`salaries` mapped to number 165
# at 722
#170815 13:29:02 server id 200  end_log_pos 1298 CRC32 0xf0ef8b05     Update_rows: table id 165 flags: STMT_END_F

BINLOG '
HveSWRPIAAAAPgAAANICAAAAAKUAAAAAAAEACWVtcGxveWVlcwAIc2FsYXJpZXMABAMDCgoAAAEB
AHTsouA=
~
~
'/*!*/;
### UPDATE `employees`.`salaries`
### WHERE
###   @1=10001
###   @2=240468
###   @3='1986:06:26'
###   @4='1987:06:26'
### SET
###   @1=10001
###   @2=120234
###   @3='1986:06:26'
###   @4='1987:06:26'
~
~
### UPDATE `employees`.`salaries`
### WHERE
###   @1=10001
###   @2=400000
###   @3='2017:06:18'
###   @4='9999:01:01'
### SET
###   @1=10001
###   @2=200000
###   @3='2017:06:18'
###   @4='9999:01:01'
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
/*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
```

如果您只想看到伪 SQL 而不包含二进制行信息，请指定`--base64-output="decode-rows"`以及`--verbose`：

```sql
shell>  sudo mysqlbinlog /data/mysql/binlogs/server1.000001 --start-position=660 --stop-position=1298 --verbose --base64-output="decode-rows"
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 660
#170815 13:29:02 server id 200  end_log_pos 722 CRC32 0xe0a2ec74     Table_map: `employees`.`salaries` mapped to number 165
# at 722
#170815 13:29:02 server id 200  end_log_pos 1298 CRC32 0xf0ef8b05     Update_rows: table id 165 flags: STMT_END_F
### UPDATE `employees`.`salaries`
### WHERE
###   @1=10001
###   @2=240468
###   @3='1986:06:26'
###   @4='1987:06:26'
### SET
###   @1=10001
###   @2=120234
###   @3='1986:06:26'
###   @4='1987:06:26'
~
```

# 重写数据库名称

假设您希望将生产服务器上的`employees`数据库的二进制日志还原为开发服务器上的`employees_dev`。您可以使用`--rewrite-db='from_name->to_name'`选项。这将重写所有`from_name`到`to_name`的出现。

要转换多个数据库，请多次指定该选项：

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001 --start-position=1499 --stop-position=1646 --rewrite-db='employees->employees_dev'
~
# at 1499
#170815 13:29:27 server id 200  end_log_pos 1646 CRC32 0xc73d68fb     Query    thread_id=8    exec_time=0    error_code=0
use `employees_dev`/*!*/;
~
~
UPDATE salaries SET salary=salary*2 WHERE emp_no<10002
/*!*/;
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
DELIMITER ;
# End of log file
~
```

您可以看到语句`use `employees_dev`/*!*/;`被使用。因此，在还原时，所有更改将应用于`employees_dev`数据库。

如 MySQL 参考手册中所述：

当与`--database`选项一起使用时，首先应用`--rewrite-db`选项，然后使用重写后的数据库名称应用`--database`选项。在这方面，提供选项的顺序没有任何区别。这意味着，例如，如果使用`--rewrite-db='mydb->yourdb' --database=yourdb`启动`mysqlbinlog`，则`mydb`和`yourdb`数据库中任何表的所有更新都包含在输出中。

另一方面，如果使用`--rewrite-db='mydb->yourdb' --database=mydb`启动，则`mysqlbinlog`根本不输出语句：因为在应用`--database`选项之前，对`mydb`的所有更新都首先被重写为对`yourdb`的更新，因此没有更新与`--database=mydb`匹配。

# 禁用二进制日志以进行恢复

在还原二进制日志时，如果您不希望`mysqlbinlog`进程创建二进制日志，您可以使用`--disable-log-bin`选项，以便不写入二进制日志：

```sql
shell> sudo mysqlbinlog /data/mysql/binlogs/server1.000001 --start-position=660 --stop-position=1298 --disable-log-bin > binlog_restore
```

您可以看到`SQL_LOG_BIN=0`被写入`binlog`还原文件，这将阻止创建 binlogs。

`/*!32316 SET @OLD_SQL_LOG_BIN=@@SQL_LOG_BIN, SQL_LOG_BIN=0*/;`

# 显示二进制日志文件中的事件

除了使用`mysqlbinlog`，您还可以使用`SHOW BINLOG EVENTS`命令来显示事件。

以下命令将显示`server1.000008`二进制日志中的事件。如果未指定`LIMIT`，则显示所有事件：

```sql
mysql> SHOW BINLOG EVENTS IN 'server1.000008' LIMIT 10; +----------------+-----+----------------+-----------+-------------+------------------------------------------+
| Log_name       | Pos | Event_type     | Server_id | End_log_pos | Info                                     |
+----------------+-----+----------------+-----------+-------------+------------------------------------------+
| server1.000008 |   4 | Format_desc    |       200 |         123 | Server ver: 8.0.3-rc-log, Binlog ver: 4 |
| server1.000008 | 123 | Previous_gtids |       200 |         154 |                                          |
| server1.000008 | 154 | Anonymous_Gtid |       200 |         226 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'     |
| server1.000008 | 226 | Query          |       200 |         336 | drop database company /* xid=4134 */     |
| server1.000008 | 336 | Anonymous_Gtid |       200 |         408 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'     |
| server1.000008 | 408 | Query          |       200 |         485 | BEGIN                                    |
| server1.000008 | 485 | Table_map      |       200 |         549 | table_id: 975 (employees.emp_details)    |
| server1.000008 | 549 | Write_rows     |       200 |         804 | table_id: 975 flags: STMT_END_F          |
| server1.000008 | 804 | Xid            |       200 |         835 | COMMIT /* xid=9751 */                    |
| server1.000008 | 835 | Anonymous_Gtid |       200 |         907 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'     |
+----------------+-----+----------------+-----------+-------------+------------------------------------------+
10 rows in set (0.00 sec)

```

您还可以指定位置和偏移量：

```sql
mysql> SHOW BINLOG EVENTS IN 'server1.000008' FROM 123 LIMIT 2,1; +----------------+-----+------------+-----------+-------------+--------------------------------------+
| Log_name       | Pos | Event_type | Server_id | End_log_pos | Info                                 |
+----------------+-----+------------+-----------+-------------+--------------------------------------+
| server1.000008 | 226 | Query      |       200 |         336 | drop database company /* xid=4134 */ |
+----------------+-----+------------+-----------+-------------+--------------------------------------+
1 row in set (0.00 sec)
```

# 忽略写入二进制日志的数据库

您可以通过在`my.cnf`中指定`--binlog-do-db=db_name`选项来选择应写入二进制日志的数据库。要指定多个数据库，*必须*使用此选项的多个实例。因为数据库名称可以包含逗号，如果提供逗号分隔的列表，该列表将被视为单个数据库的名称。您需要重新启动 MySQL 服务器才能生效。

# 如何操作...

打开`my.cnf`并添加以下行：

```sql
shell> sudo vi /etc/my.cnf
[mysqld]
binlog_do_db=db1
binlog_do_db=db2
```

`binlog-do-db`的行为从基于语句的日志记录更改为基于行的日志记录，就像`mysqlbinlog`实用程序中的`--database`选项一样。

在基于语句的日志记录中，只有那些默认数据库（即由`USE`选择的数据库）写入二进制日志的语句才会被写入。在使用基于语句的日志记录时，使用`binlog-do-db`选项时要非常小心，因为它的工作方式与您在使用基于语句的日志记录时所期望的方式不同。请参阅参考手册中提到的以下示例。

# 示例 1

如果服务器是使用`--binlog-do-db=sales`启动的，并且您发出以下语句，则`UPDATE`语句不会被记录：

```sql
mysql> USE prices;
mysql> UPDATE sales.january SET amount=amount+1000;
```

这种*只检查默认数据库*行为的主要原因是，仅从语句本身很难知道是否应该复制它。如果没有必要，仅检查默认数据库而不是所有数据库也更快。

# 示例 2

如果服务器是使用`--binlog-do-db=sales`启动的，则即使在设置`--binlog-do-db`时未包括价格，以下`UPDATE`语句也会被记录：

```sql
mysql> USE sales;
mysql> UPDATE prices.discounts SET percentage = percentage + 10;
```

当发出`UPDATE`语句时，因为销售是默认数据库，所以`UPDATE`被记录在日志中。

在基于行的日志记录中，它受到数据库`db_name`的限制。只有属于`db_name`的表的更改才会被记录；默认数据库对此没有影响。

`--binlog-do-db`在基于语句的日志记录中处理的另一个重要区别与基于行的日志记录有关，这涉及到引用多个数据库的语句。假设服务器是使用`--binlog-do-db=db1`启动的，并且执行了以下语句：

```sql
mysql> USE db1;
mysql> UPDATE db1.table1 SET col1 = 10, db2.table2 SET col2 = 20;
```

如果您使用基于语句的日志记录，则两个表的更新都将写入二进制日志。但是，使用基于行的格式时，只有`table1`的更改被记录；`table2`位于不同的数据库中，因此不会受到`UPDATE`的影响。

同样，您可以使用`--binlog-ignore-db=db_name`选项来忽略写入二进制日志的数据库。

有关更多信息，请参阅手册：[`dev.mysql.com/doc/refman/8.0/en/replication-rules.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-rules.html)。

# 重新定位二进制日志

由于二进制日志占用更多空间，有时您可能希望更改二进制日志的位置，以下过程有所帮助。仅更改`log_bin`是不够的，您必须移动所有二进制日志并使用新位置更新索引文件。`mysqlbinlogmove`实用程序通过自动化这些任务来简化您的工作。

# 如何操作...

安装 MySQL Utilities 以使用`mysqlbinlogmove`脚本。有关安装步骤，请参阅第一章，*MySQL 8.0 – Installing and Upgrading*。

1.  停止 MySQL 服务器：

```sql
shell> sudo systemctl stop mysql
```

1.  启动`mysqlbinlogmove`实用程序。如果要将二进制日志从`/data/mysql/binlogs`更改为`/binlogs`，则应使用以下命令。如果您的基本名称不是默认值，则必须通过`--bin-log-base name`选项提及您的基本名称：

```sql
shell> sudo mysqlbinlogmove --bin-log-base name=server1  --binlog-dir=/data/mysql/binlogs /binlogs
#
# Moving bin-log files...
# - server1.000001
# - server1.000002
# - server1.000003
# - server1.000004
# - server1.000005
#
#...done.
#
```

1.  编辑`my.cnf`文件并更新`log_bin`的新位置：

```sql
shell> sudo vi /etc/my.cnf
[mysqld]
log_bin=/binlogs
```

1.  启动 MySQL 服务器：

```sql
shell> sudo systemctl start mysql
```

新位置在 AppArmor 或 SELinux 中得到更新。

如果有很多二进制日志，服务器的停机时间会很长。为了避免这种情况，您可以使用`--server`选项来重新定位除当前正在使用的日志之外的所有二进制日志（具有更高的序列号）。然后停止服务器，使用前面的方法，重新定位最后一个二进制日志，这将会快得多，因为只有一个文件在那里。然后您可以更改`my.cnf`并启动服务器。

例如：

```sql
shell> sudo mysqlbinlogmove --server=root:pass@host1:3306 /new/location
```
