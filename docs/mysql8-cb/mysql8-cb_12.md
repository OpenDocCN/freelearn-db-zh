# 第十二章：管理日志

在本章中，我们将涵盖以下内容：

+   管理错误日志

+   管理常规查询日志和慢查询日志

+   管理二进制日志

# 介绍

现在您将了解如何管理不同类型的日志：错误日志、常规查询日志、慢查询日志、二进制日志、中继日志和 DDL 日志。

# 管理错误日志

根据 MySQL 文档：

错误日志包含了`mysqld`启动和关闭时间的记录。它还包含了在服务器启动和关闭期间以及服务器运行时发生的错误、警告和注释等诊断消息。

错误日志子系统由执行日志事件过滤和写入的组件以及一个名为`log_error_services`的系统变量组成，该变量配置了要启用哪些组件以实现所需的日志记录结果。`global.log_error_services`的默认值是`log_filter_internal; log_sink_internal`：

```go
mysql> SELECT @@global.log_error_services;
+----------------------------------------+
| @@global.log_error_services            |
+----------------------------------------+
| log_filter_internal; log_sink_internal |
+----------------------------------------+
```

该值表示日志事件首先通过内置过滤组件`log_filter_internal`，然后通过内置日志写入组件`log_sink_internal`。组件顺序很重要，因为服务器按照列出的顺序执行组件。`log_error_services`值中命名的任何可加载（非内置）组件必须首先使用`INSTALL COMPONENT`进行安装，这将在本节中描述。

要了解所有类型的错误日志记录，请参阅[`dev.mysql.com/doc/refman/8.0/en/error-log.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log.html)

# 如何做...

错误日志在某种程度上很容易。让我们先看看如何配置错误日志。

# 配置错误日志

错误日志由`log_error`变量（启动脚本的`--log-error`）控制。

如果未给出`--log-error`，默认目的地是控制台。

如果给出`--log-error`而没有命名文件，则默认目的地是`data directory`中名为`host_name.err`的文件。

如果给出`--log-error`以命名文件，则默认目的地是该文件（如果名称没有后缀，则添加`.err`后缀），位于`data directory`下，除非给出绝对路径名以指定不同的位置。

`log_error_verbosity`系统变量控制服务器写入错误、警告和注释消息到错误日志的详细程度。允许的`log_error_verbosity`值为`1`（仅错误）、`2`（错误和警告）和`3`（错误、警告和注释），默认值为`3`。

要更改错误日志位置，请编辑配置文件并重新启动 MySQL：

```go
shell> sudo mkdir /var/log/mysql
shell> sudo chown -R mysql:mysql /var/log/mysql

shell> sudo vi /etc/my.cnf
[mysqld]
log-error=/var/log/mysql/mysqld.log

shell> sudo systemctl restart mysql
```

验证错误日志：

```go
mysql> SHOW VARIABLES LIKE 'log_error';
+---------------+---------------------------+
| Variable_name | Value                     |
+---------------+---------------------------+
| log_error     | /var/log/mysql/mysqld.log |
+---------------+---------------------------+
1 row in set (0.00 sec)
```

要调整详细程度，您可以动态更改`log_error_verbosity`变量。但是，建议保持默认值`3`，以便记录错误、警告和注释消息：

```go
mysql> SET @@GLOBAL.log_error_verbosity=2;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@GLOBAL.log_error_verbosity;
+------------------------------+
| @@GLOBAL.log_error_verbosity |
+------------------------------+
|                            2 |
+------------------------------+
1 row in set (0.00 sec)
```

# 旋转错误日志

假设错误日志文件变得更大，您想要对其进行旋转；您可以简单地移动文件并执行`FLUSH LOGS`命令：

```go
shell> sudo mv /var/log/mysql/mysqld.log /var/log/mysql/mysqld.log.0; 

shell> mysqladmin -u root -p<password> flush-logs
mysqladmin: [Warning] Using a password on the command line interface can be insecure.

shell> ls -lhtr /var/log/mysql/mysqld.log
-rw-r-----. 1 mysql mysql 0 Oct 10 14:03 /var/log/mysql/mysqld.log

shell> ls -lhtr /var/log/mysql/mysqld.log.0 
-rw-r-----. 1 mysql mysql 3.4K Oct 10 14:03 /var/log/mysql/mysqld.log.0
```

您可以使用一些脚本自动化上述步骤，并将它们放入`cron`中。

如果错误日志文件的位置对服务器不可写，日志刷新操作将无法创建新的日志文件：

```go
shell> sudo mv /var/log/mysqld.log /var/log/mysqld.log.0 && mysqladmin flush-logs -u root -p<password>
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
mysqladmin: refresh failed; error: 'Unknown error'
```

# 使用系统日志进行日志记录

要使用系统日志进行日志记录，您需要加载名为`log_sink_syseventlog`的系统日志写入器。您可以使用内置过滤器`log_filter_internal`进行过滤：

1.  加载系统日志写入器：

```go
mysql> INSTALL COMPONENT 'file://component_log_sink_syseventlog';
Query OK, 0 rows affected (0.43 sec)
```

1.  使其在重新启动时持久化：

```go
mysql> SET PERSIST log_error_services = 'log_filter_internal; log_sink_syseventlog';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'log_error_services';
+--------------------+-------------------------------------------+
| Variable_name      | Value                                     |
+--------------------+-------------------------------------------+
| log_error_services | log_filter_internal; log_sink_syseventlog |
+--------------------+-------------------------------------------+
1 row in set (0.00 sec)
```

1.  您可以验证日志将被定向到系统日志。在 CentOS 和 Red Hat 上，您可以在`/var/log/messages`中检查；在 Ubuntu 上，您可以在`/var/log/syslog`中检查。

为了演示，服务器已重新启动。您可以在系统日志中看到这些日志：

```go
shell> sudo grep mysqld /var/log/messages | tail
Oct 10 14:50:31 centos7 mysqld[20953]: InnoDB: Buffer pool(s) dump completed at 171010 14:50:31
Oct 10 14:50:32 centos7 mysqld[20953]: InnoDB: Shutdown completed; log sequence number 350327631
Oct 10 14:50:32 centos7 mysqld[20953]: InnoDB: Removed temporary tablespace data file: "ibtmp1"
Oct 10 14:50:32 centos7 mysqld[20953]: Shutting down plugin 'MEMORY'
Oct 10 14:50:32 centos7 mysqld[20953]: Shutting down plugin 'CSV'
Oct 10 14:50:32 centos7 mysqld[20953]: Shutting down plugin 'sha256_password'
Oct 10 14:50:32 centos7 mysqld[20953]: Shutting down plugin 'mysql_native_password'
Oct 10 14:50:32 centos7 mysqld[20953]: Shutting down plugin 'binlog'
Oct 10 14:50:32 centos7 mysqld[20953]: /usr/sbin/mysqld: Shutdown complete
Oct 10 14:50:33 centos7 mysqld[21220]: /usr/sbin/mysqld: ready for connections. Version: '8.0.3-rc-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)
```

如果有多个运行的`mysqld`进程，您可以使用`[]`中指定的 PID 进行区分。否则，您可以设置`log_syslog_tag`变量，该变量会在服务器标识符前附加一个连字符，从而得到一个`mysqld-tag_val`的标识符。

例如，您可以为实例添加标签，如`instance1`：

```go
mysql> SELECT @@GLOBAL.log_syslog_tag;
+-------------------------+
| @@GLOBAL.log_syslog_tag |
+-------------------------+
|                         |
+-------------------------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.log_syslog_tag='instance1';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@GLOBAL.log_syslog_tag;
+-------------------------+
| @@GLOBAL.log_syslog_tag |
+-------------------------+
| instance1               |
+-------------------------+
1 row in set (0.01 sec)

shell> sudo systemctl restart mysqld

shell> sudo grep mysqld /var/log/messages | tail
Oct 10 14:59:20 centos7 mysqld-instance1[21220]: InnoDB: Buffer pool(s) dump completed at 171010 14:59:20
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: InnoDB: Shutdown completed; log sequence number 350355306
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: InnoDB: Removed temporary tablespace data file: "ibtmp1"
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: Shutting down plugin 'MEMORY'
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: Shutting down plugin 'CSV'
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: Shutting down plugin 'sha256_password'
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: Shutting down plugin 'mysql_native_password'
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: Shutting down plugin 'binlog'
Oct 10 14:59:21 centos7 mysqld-instance1[21220]: /usr/sbin/mysqld: Shutdown complete
Oct 10 14:59:22 centos7 mysqld[21309]: /usr/sbin/mysqld: ready for connections. Version: '8.0.3-rc-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)
```

您会注意到`instance1`标签附加到日志中，以便您可以轻松识别多个实例之间的区别。

如果要切换回原始日志记录，可以将`log_error_services`设置为`'log_filter_internal; log_sink_internal'`：

```go
mysql> SET @@global.log_error_services='log_filter_internal; log_sink_internal';
Query OK, 0 rows affected (0.00 sec)
```

# 以 JSON 格式记录错误

要使用 JSON 格式进行记录，您需要加载名为`log_sink_json`的 JSON 日志写入器。您可以使用内置过滤器`log_filter_internal`进行过滤：

1.  安装 JSON 日志写入器：

```go
mysql> INSTALL COMPONENT 'file://component_log_sink_json';
Query OK, 0 rows affected (0.05 sec)
```

1.  使其在重新启动后保持不变：

```go
mysql> SET PERSIST log_error_services = 'log_filter_internal; log_sink_json';
Query OK, 0 rows affected (0.00 sec)
```

1.  JSON 日志写入器根据`log_error`系统变量给出的默认错误日志目的地确定其输出目的地：

```go
mysql> SHOW VARIABLES LIKE 'log_error';
+---------------+---------------------------+
| Variable_name | Value                     |
+---------------+---------------------------+
| log_error     | /var/log/mysql/mysqld.log |
+---------------+---------------------------+
1 row in set (0.00 sec)
```

1.  日志将类似于`mysqld.log.00.json`。

重新启动后，JSON 日志文件如下所示：

```go
shell> sudo less /var/log/mysql/mysqld.log.00.json
{ "prio" : 2, "err_code" : 4356, "subsystem" : "", "SQL_state" : "HY000", "source_file" : "sql_plugin.cc", "function" : "reap_plugins", "msg" : "Shutting down plugin 'sha256_password'", "time" : "2017-10-15T12:29:08.862969Z", "err_symbol" : "ER_PLUGIN_SHUTTING_DOWN_PLUGIN", "label" : "Note" }
{ "prio" : 2, "err_code" : 4356, "subsystem" : "", "SQL_state" : "HY000", "source_file" : "sql_plugin.cc", "function" : "reap_plugins", "msg" : "Shutting down plugin 'mysql_native_password'", "time" : "2017-10-15T12:29:08.862975Z", "err_symbol" : "ER_PLUGIN_SHUTTING_DOWN_PLUGIN", "label" : "Note" }
{ "prio" : 2, "err_code" : 4356, "subsystem" : "", "SQL_state" : "HY000", "source_file" : "sql_plugin.cc", "function" : "reap_plugins", "msg" : "Shutting down plugin 'binlog'", "time" : "2017-10-15T12:29:08.863758Z",  "err_symbol" : "ER_PLUGIN_SHUTTING_DOWN_PLUGIN", "label" : "Note" }
{  "prio" : 2, "err_code" : 1079, "subsystem" : "", "SQL_state" : "HY000",  "source_file" : "mysqld.cc", "function" : "clean_up", "msg" : "/usr/sbin/mysqld: Shutdown complete\u000a", "time" : "2017-10-15T12:29:08.867077Z", "err_symbol" : "ER_SHUTDOWN_COMPLETE", "label" : "Note" }
{ "log_type" : 1, "prio" : 0, "err_code" : 1408, "msg" : "/usr/sbin/mysqld: ready for connections. Version: '8.0.3-rc-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)", "time" : "2017-10-15T12:29:10.952502Z", "err_symbol" : "ER_STARTUP", "SQL_state" : "HY000", "label" : "Note" }
```

如果要切换回原始日志记录，可以将`log_error_services`设置为`'log_filter_internal; log_sink_internal'`：

```go
mysql> SET @@global.log_error_services='log_filter_internal; log_sink_internal';
Query OK, 0 rows affected (0.00 sec)
```

要了解有关错误记录配置的更多信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/error-log-component-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-component-configuration.html)。

# 管理一般查询日志和慢查询日志

有两种方式可以记录查询。一种是通过一般查询日志，另一种是通过慢查询日志。在本节中，您将学习如何配置它们。

# 如何做...

我们将在以下子节中详细介绍。

# 一般查询日志

根据 MySQL 文档：

一般查询日志是`mysqld`正在执行的一般记录。当客户端连接或断开连接时，服务器会将信息写入此日志，并记录从客户端接收到的每个 SQL 语句。当您怀疑客户端中存在错误并想要确切知道客户端发送给`mysqld`的内容时，一般查询日志可能非常有用：

1.  指定日志文件。如果不指定，它将在`data directory`中以`hostname.log`的名称创建。

服务器会在`data directory`中创建文件，除非给出绝对路径名以指定不同的目录：

```go
mysql> SET @@GLOBAL.general_log_file='/var/log/mysql/general_query_log';
Query OK, 0 rows affected (0.04 sec)
```

1.  启用一般查询日志：

```go
mysql> SET GLOBAL general_log = 'ON';
Query OK, 0 rows affected (0.00 sec)
```

1.  您可以看到查询已记录：

```go
shell> sudo cat /var/log/mysql/general_query_log
/usr/sbin/mysqld, Version: 8.0.3-rc-log (MySQL Community Server (GPL)). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
2017-10-11T04:21:00.118944Z      220 Connect    root@localhost on  using Socket
2017-10-11T04:21:00.119212Z      220 Query    select @@version_comment limit 1
2017-10-11T04:21:03.217603Z      220 Query    SELECT DATABASE()
2017-10-11T04:21:03.218275Z      220 Init DB    employees
2017-10-11T04:21:03.219339Z      220 Query    show databases
2017-10-11T04:21:03.220189Z      220 Query    show tables
2017-10-11T04:21:03.227635Z      220 Field List    current_dept_emp
2017-10-11T04:21:03.233820Z      220 Field List    departments 
2017-10-11T04:21:03.235937Z      220 Field List    dept_emp 
2017-10-11T04:21:03.236089Z      220 Field List    dept_emp_latest_date 
2017-10-11T04:21:03.236337Z      220 Field List    dept_manager 
2017-10-11T04:21:03.237291Z      220 Field List    employee_names 
2017-10-11T04:21:03.237999Z      220 Field List    employees 
2017-10-11T04:21:03.247921Z      220 Field List    titles 
2017-10-11T04:21:03.248217Z      220 Field List    titles_only 
~
~
2017-10-11T04:21:09.483117Z      220 Query    select count(*) from employees
2017-10-11T04:21:10.523421Z      220 Quit
```

一般查询日志会生成一个非常大的日志文件。在生产服务器上启用时要非常小心。它会严重影响服务器的性能。

# 慢查询日志

根据 MySQL 文档：

“慢查询日志由执行时间超过 long_query_time 秒并且需要至少 min_examined_row_limit 行才能检查的 SQL 语句组成。”

要记录所有查询，可以将`long_query_time`的值设置为`0`。`long_query_time`的默认值为`10`秒，`min_examined_row_limit`为`0`。

默认情况下，不会记录不使用索引进行查找的查询和管理语句（例如`ALTER TABLE`，`ANALYZE TABLE`，`CHECK TABLE`，`CREATE INDEX`，`DROP INDEX`，`OPTIMIZE TABLE`和`REPAIR TABLE`）。可以使用`log_slow_admin_statements`和`log_queries_not_using_indexes`更改此行为。

要启用慢查询日志，可以动态设置`slow_query_log=1`，并可以使用`slow_query_log_file`设置文件名。要指定日志目的地，请使用`--log-output`：

1.  验证`long_query_time`并根据您的要求进行调整：

```go
mysql> SELECT @@GLOBAL.LONG_QUERY_TIME;
+--------------------------+
| @@GLOBAL.LONG_QUERY_TIME |
+--------------------------+
|                10.000000 |
+--------------------------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.LONG_QUERY_TIME=1;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@GLOBAL.LONG_QUERY_TIME;
+--------------------------+
| @@GLOBAL.LONG_QUERY_TIME |
+--------------------------+
|                 1.000000 |
+--------------------------+
1 row in set (0.00 sec)
```

1.  验证慢查询文件。默认情况下，它将在`data directory`中以`hostname-slow`日志的形式存在：

```go
mysql> SELECT @@GLOBAL.slow_query_log_file;
+---------------------------------+
| @@GLOBAL.slow_query_log_file    |
+---------------------------------+
| /var/lib/mysql/server1-slow.log |
+---------------------------------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.slow_query_log_file='/var/log/mysql/mysql_slow.log';                                                                         Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@GLOBAL.slow_query_log_file;
+-------------------------------+
| @@GLOBAL.slow_query_log_file  |
+-------------------------------+
| /var/log/mysql/mysql_slow.log |
+-------------------------------+
1 row in set (0.00 sec)

mysql> FLUSH LOGS;
Query OK, 0 rows affected (0.03 sec)
```

1.  启用慢查询日志：

```go
mysql> SELECT @@GLOBAL.slow_query_log;
+-------------------------+
| @@GLOBAL.slow_query_log |
+-------------------------+
|                       0 |
+-------------------------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.slow_query_log=1;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT @@GLOBAL.slow_query_log;
+-------------------------+
| @@GLOBAL.slow_query_log |
+-------------------------+
|                       1 |
+-------------------------+
1 row in set (0.00 sec)
```

1.  验证查询是否已记录（您必须执行一些长时间运行的查询才能在慢查询日志中看到它们）：

```go
mysql> SELECT SLEEP(2);
+----------+
| SLEEP(2) |
+----------+
|        0 |
+----------+
1 row in set (2.00 sec)

shell> sudo less /var/log/mysql/mysql_slow.log 
/usr/sbin/mysqld, Version: 8.0.3-rc-log (MySQL Community Server (GPL)). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
# Time: 2017-10-15T12:43:55.038601Z
# User@Host: root[root] @ localhost []  Id:     7
# Query_time: 2.000845  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 0
SET timestamp=1508071435;
SELECT SLEEP(2);
```

# 选择查询日志输出目的地

您可以通过指定`log_output`变量在 MySQL 本身中将查询记录到`FILE`或`TABLE`中，该变量可以是`FILE`或`TABLE`，也可以是`FILE`和`TABLE`。

如果将`log_output`指定为`FILE`，则一般查询日志和慢查询日志将分别写入`general_log_file`和`slow_query_log_file`指定的文件中。

如果您将`log_output`指定为`TABLE`，则一般查询日志和慢查询日志将分别写入`mysql.general_log`和`mysql.slow_log`表中。日志内容可以通过 SQL 语句访问。

例如：

```go
mysql> SET @@GLOBAL.log_output='TABLE';
Query OK, 0 rows affected (0.00 sec)

mysql> SET @@GLOBAL.general_log='ON';
Query OK, 0 rows affected (0.02 sec)
```

执行一些查询，然后查询`mysql.general_log`表：

```go
mysql> SELECT * FROM mysql.general_log WHERE command_type='Query' \G
~
~
*************************** 3\. row ***************************
  event_time: 2017-10-25 10:56:56.416746
   user_host: root[root] @ localhost []
   thread_id: 2421
   server_id: 32
command_type: Query
    argument: show databases
*************************** 4\. row ***************************
  event_time: 2017-10-25 10:56:56.418896
   user_host: root[root] @ localhost []
   thread_id: 2421
   server_id: 32
command_type: Query
    argument: show tables
*************************** 5\. row ***************************
  event_time: 2017-10-25 10:57:08.207964
   user_host: root[root] @ localhost []
   thread_id: 2421
   server_id: 32
command_type: Query
    argument: select * from salaries limit 1
*************************** 6\. row ***************************
  event_time: 2017-10-25 10:57:47.041475
   user_host: root[root] @ localhost []
   thread_id: 2421
   server_id: 32
command_type: Query
    argument: SELECT * FROM mysql.general_log WHERE command_type='Query'
```

您可以以类似的方式使用`slow_log`表：

```go
mysql> SET @@GLOBAL.slow_query_log=1;
Query OK, 0 rows affected (0.00 sec)

mysql> SET @@GLOBAL.long_query_time=1;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT SLEEP(2);
+----------+
| SLEEP(2) |
+----------+
|        0 |
+----------+
1 row in set (2.00 sec)

mysql> SELECT * FROM mysql.slow_log \G
*************************** 1\. row ***************************
    start_time: 2017-10-25 11:01:44.817421
     user_host: root[root] @ localhost []
    query_time: 00:00:02.000530
     lock_time: 00:00:00.000000
     rows_sent: 1
 rows_examined: 0
            db: employees
last_insert_id: 0
     insert_id: 0
     server_id: 32
      sql_text: SELECT SLEEP(2)
     thread_id: 2421
1 row in set (0.00 sec)
```

如果慢查询日志表变得非常庞大，您可以通过创建一个新表并交换它来进行轮换：

1.  创建一个新表`mysql.general_log_new`：

```go
mysql> DROP TABLE IF EXISTS mysql.general_log_new;
Query OK, 0 rows affected, 1 warning (0.19 sec)

mysql> CREATE TABLE mysql.general_log_new LIKE mysql.general_log;
Query OK, 0 rows affected (0.10 sec)
```

1.  使用`RENAME TABLE`命令交换表：

```go
mysql> RENAME TABLE mysql.general_log TO mysql.general_log_1, mysql.general_log_new TO mysql.general_log;
Query OK, 0 rows affected (0.00 sec)
```

# 管理二进制日志

在这一部分，将介绍在复制环境中管理二进制日志。基本的二进制日志处理已经在第六章“二进制日志记录”中介绍过，使用`PURGE BINARY LOGS`命令和`expire_logs_days`变量。

在复制环境中使用这些方法是不安全的，因为如果任何一个从服务器没有消耗二进制日志并且您已经删除了它们，从服务器将会失去同步，您将需要重建它。

安全地删除二进制日志的方法是检查每个从服务器上已读取了哪些二进制日志，并删除它们。您可以使用`mysqlbinlogpurge`实用程序来实现这一点。

# 如何做...

在任何一个服务器上执行`mysqlbinlogpurge`脚本，并指定主服务器和从服务器主机。该脚本连接到所有的从服务器，并找出最新应用的二进制日志。然后清除主二进制日志直到那一点。您需要一个超级用户来连接到所有的从服务器：

1.  连接到任何一个服务器并执行`mysqlbinlogpurge`脚本：

```go
shell> mysqlbinlogpurge --master=dbadmin:<pass>@master:3306 --slaves=dbadmin:<pass>@slave1:3306,dbadmin:<pass>@slave2:3306
```

```go
mysql> SHOW BINARY LOGS;
+--------------------+-----------+
| Log_name           | File_size |
+--------------------+-----------+
| master-bin.000001  |       177 |
~
| master-bin.000018  |     47785 |
| master-bin.000019  |       203 |
| master-bin.000020  |       203 |
| master-bin.000021  |       177 |
| master-bin.000022  |       203 |
| master-bin.000023  |  57739432 |
+--------------------+-----------+
23 rows in set (0.00 sec)

shell> mysqlbinlogpurge --master=dbadmin:<pass>@master:3306 --slaves=dbadmin:<pass>@slave1:3306,dbadmin:<pass>@slave2:3306

# Latest binlog file replicated by all slaves: master-bin.000022
# Purging binary logs prior to 'master-bin.000023'
```

1.  如果您希望在命令中不指定的情况下发现所有的从服务器，您应该在所有的从服务器上设置`report_host`和`report_port`，并重新启动 MySQL 服务器。在每个从服务器上：

```go
shell> sudo vi /etc/my.cnf
[mysqld]
report-host     = slave1
report-port     = 3306

shell> sudo systemctl restart mysql

mysql> SHOW VARIABLES LIKE 'report%';
+-----------------+---------------+
| Variable_name   | Value         |
+-----------------+---------------+
| report_host     | slave1        |
| report_password |               |
| report_port     | 3306          |
| report_user     |               |
+-----------------+---------------+
4 rows in set (0.00 sec)
```

1.  使用`discover-slaves-login`选项执行`mysqlbinlogpurge`：

```go
mysql> SHOW BINARY LOGS;
+--------------------+-----------+
| Log_name           | File_size |
+--------------------+-----------+
| centos7-bin.000025 |       203 |
| centos7-bin.000026 |       203 |
| centos7-bin.000027 |       203 |
| centos7-bin.000028 |       154 |
+--------------------+-----------+
4 rows in set (0.00 sec)

shell> mysqlbinlogpurge --master=dbadmin:<pass>@master --discover-slaves-login=dbadmin:<pass>
# Discovering slaves for master at master:3306
# Discovering slave at slave1:3306
# Found slave: slave1:3306
# Discovering slave at slave2:3306
# Found slave: slave2:3306
# Latest binlog file replicated by all slaves: master-bin.000027
# Purging binary logs prior to 'master-bin.000028'
```
