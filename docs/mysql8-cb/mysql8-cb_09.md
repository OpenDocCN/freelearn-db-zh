# 第九章：复制

在这一章中，我们将涵盖以下内容：

+   设置复制

+   设置主-主复制

+   设置多源复制

+   设置复制过滤器

+   将从主-从复制切换到链式复制

+   将从链式复制切换到主-从复制

+   设置延迟复制

+   设置 GTID 复制

+   设置半同步复制

# 介绍

如第六章中所解释的，*二进制日志*，复制使得来自一个 MySQL 数据库服务器（主服务器）的数据被复制到一个或多个 MySQL 数据库服务器（从服务器）。复制默认是异步的；从服务器不需要永久连接以接收来自主服务器的更新。您可以配置复制所有数据库、选定的数据库，甚至是数据库中的选定表。

在这一章中，您将学习如何设置传统复制；复制选定的数据库和表；以及设置多源复制、链式复制、延迟复制和半同步复制。

在高层次上，复制的工作原理是这样的：在一个服务器上执行的所有 DDL 和 DML 语句（**主服务器**）都被记录到二进制日志中，这些日志被连接到它的服务器（称为**从服务器**）拉取。二进制日志简单地被复制到从服务器并保存为中继日志。这个过程由一个叫做**IO 线程**的线程来处理。还有一个叫做**SQL 线程**的线程，按顺序执行中继日志中的语句。

复制的工作原理在这篇博客中得到了很清楚的解释：

[`www.percona.com/blog/2013/01/09/how-does-mysql-replication-really-work/`](https://www.percona.com/blog/2013/01/09/how-does-mysql-replication-really-work/)

复制的优点（摘自手册，网址为[`dev.mysql.com/doc/refman/8.0/en/replication.html`](https://dev.mysql.com/doc/refman/8.0/en/replication.html)）：

+   **扩展解决方案**：将负载分散在多个从服务器上以提高性能。在这种环境中，所有的写入和更新必须在主服务器上进行。然而，读取可以在一个或多个从服务器上进行。这种模式可以提高写入的性能（因为主服务器专门用于更新），同时大大提高了在越来越多的从服务器上的读取速度。

+   **数据安全**：因为数据被复制到从服务器并且从服务器可以暂停复制过程，所以可以在从服务器上运行备份服务而不会破坏相应的主服务器数据。

+   **分析**：可以在主服务器上创建实时数据，而信息的分析可以在从服务器上进行，而不会影响主服务器的性能。

+   **远程数据分发**：您可以使用复制在远程站点创建数据的本地副本，而无需永久访问主服务器。

# 设置复制

有许多复制拓扑结构。其中一些是传统的主-从复制、链式复制、主-主复制、多源复制等。

**传统复制** 包括一个主服务器和多个从服务器。

![](img/00005.jpeg)

**链式复制** 意味着一个服务器从另一个服务器复制，而另一个服务器又从另一个服务器复制。中间服务器被称为中继主服务器（主服务器 ---> 中继主服务器 ---> 从服务器）。

![](img/00006.jpeg)

这主要用于当您想在两个数据中心之间设置复制时。主服务器和其从服务器将位于一个数据中心。次要主服务器（中继）从另一个数据中心的主服务器进行复制。另一个数据中心的所有从服务器都从次要主服务器进行复制。

**主-主复制**：在这种拓扑结构中，两个主服务器都接受写入并在彼此之间进行复制。

![](img/00007.jpeg)

**多源复制**：在这种拓扑结构中，一个从服务器将从多个主服务器而不是一个主服务器进行复制。

![](img/00008.jpeg)

如果要设置链式复制，可以按照此处提到的相同步骤进行，将主服务器替换为中继主服务器。

# 如何做...

在本节中，解释了单个从服务器的设置。相同的原则可以应用于设置链式复制。通常在设置另一个从服务器时，备份是从从服务器中获取的。

大纲：

1.  在主服务器上启用二进制日志记录

1.  在主服务器上创建一个复制用户

1.  在从服务器上设置唯一的`server_id`

1.  从主服务器备份

1.  在从服务器上恢复备份

1.  执行`CHANGE MASTER TO`命令

1.  开始复制

步骤：

1.  **在主服务器上**：在主服务器上启用二进制日志记录并设置`SERVER_ID`。参考第六章，*二进制日志记录*，了解如何启用二进制日志记录。

1.  **在主服务器上**：创建一个复制用户。从服务器使用这个帐户连接到主服务器：

```go
mysql> GRANT REPLICATION SLAVE ON *.* TO 'binlog_user'@'%' IDENTIFIED BY 'binlog_P@ss12';Query OK, 0 rows affected, 1 warning (0.00 sec)
```

1.  **在从服务器上**：设置唯一的`SERVER_ID`选项（它应该与主服务器上设置的不同）：

```go
mysql> SET @@GLOBAL.SERVER_ID = 32;
```

1.  **在从服务器上**：通过远程连接从主服务器备份。您可以使用`mysqldump`或`mydumper`。不能使用`mysqlpump`，因为二进制日志位置不一致。

`mysqldump`：

```go
shell> mysqldump -h <master_host> -u backup_user --password=<pass> --all-databases --routines --events --single-transaction --master-data  > dump.sql

```

当从另一个从服务器备份时，您必须传递`--slave-dump`选项。`mydumper`：

```go
shell> mydumper -h <master_host> -u backup_user --password=<pass> --use-savepoints  --trx-consistency-only --kill-long-queries --outputdir /backups
```

1.  **在从服务器上**：备份完成后，恢复备份。参考第八章，*恢复数据*，了解恢复方法。

`mysqldump`：

```go
shell> mysql -u <user> -p -f < dump.sql
```

`mydumper`：

```go
shell> myloader --directory=/backups --user=<user> --password=<password> --queries-per-transaction=5000 --threads=8 --overwrite-tables
```

1.  **在从服务器上**：在恢复备份后，您必须执行以下命令：

```go
mysql> CHANGE MASTER TO MASTER_HOST='<master_host>', MASTER_USER='binlog_user', MASTER_PASSWORD='binlog_P@ss12', MASTER_LOG_FILE='<log_file_name>', MASTER_LOG_POS=<position>
```

`mysqldump`：备份转储文件中包含`<log_file_name>`和`<position>`。例如：

```go
shell> less dump.sql
--
-- Position to start replication or point-in-time recovery from (the master of this slave)
--
CHANGE MASTER TO MASTER_LOG_FILE='centos7-bin.000001', MASTER_LOG_POS=463;
```

`mydumper`：`<log_file_name>`和`<position>`存储在元数据文件中：

```go
shell> cat metadata
Started dump at: 2017-08-26 06:26:19
SHOW MASTER STATUS:
    Log: server1.000012
    Pos: 154122
    GTID:
SHOW SLAVE STATUS:
    Host: xx.xxx.xxx.xxx
    Log: centos7-bin.000001
    Pos: 463223
    GTID:
Finished dump at: 2017-08-26 06:26:24
```

如果您从一个从服务器或主服务器备份以设置另一个从服务器，您必须使用`SHOW SLAVE STATUS`中的位置。如果要设置链式复制，可以使用`SHOW MASTER STATUS`中的位置。

1.  在从服务器上，执行`START SLAVE`命令：

```go
mysql> START SLAVE;
```

1.  您可以通过执行以下命令来检查复制的状态：

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
 Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server1-bin.000001
          Read_Master_Log_Pos: 463
               Relay_Log_File: server2-relay-bin.000004
                Relay_Log_Pos: 322
        Relay_Master_Log_File: server1-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 463
              Relay_Log_Space: 1957
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
 Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 32
                  Master_UUID: b52ef45a-7ff4-11e7-9091-42010a940003
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

您应该查找`Seconds_Behind_Master`，它显示了复制的延迟。如果是`0`，表示从服务器与主服务器同步；任何非零值表示延迟的秒数，如果是`NULL`，表示复制没有发生。

# 设置主-主复制

这个教程会吸引很多人，因为我们中的许多人都尝试过这样做。让我们深入了解一下。

# 如何做...

假设主服务器是`master1`和`master2`。

步骤：

1.  按照第九章*复制*中描述的方法在`master1`和`master2`之间设置复制。

1.  使`master2`成为只读：

```go
mysql> SET @@GLOBAL.READ_ONLY=ON;
```

1.  在`master2`上，检查当前的二进制日志坐标。

```go
mysql> SHOW MASTER STATUS;
+----------------+----------+--------------+------------------+-------------------+
| File           | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+----------------+----------+--------------+------------------+-------------------+
| server1.000017 |      473 |              |                  |                   |
+----------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

从前面的输出中，您可以从`server1.000017`和位置`473`开始在`master1`上启动复制。

1.  根据前面步骤中的位置，在`master1`上执行`CHANGE MASTER TO`命令：

```go
mysql> CHANGE MASTER TO MASTER_HOST='<master2_host>', MASTER_USER='binlog_user', MASTER_PASSWORD='binlog_P@ss12', MASTER_LOG_FILE='<log_file_name>', MASTER_LOG_POS=<position>
```

1.  在`master1`上启动从服务器：

```go
mysql> START SLAVE;
```

1.  最后，您可以使`master2`成为读写，应用程序可以开始向其写入。

```go
 mysql> SET @@GLOBAL.READ_ONLY=OFF;
```

# 设置多源复制

MySQL 多源复制使得复制从服务器能够同时接收来自多个源的事务。多源复制可用于将多个服务器备份到单个服务器，合并表分片，并将来自多个服务器的数据合并到单个服务器。多源复制在应用事务时不实现任何冲突检测或解决，如果需要，这些任务将留给应用程序。在多源复制拓扑中，从服务器为应该接收事务的每个主服务器创建一个复制通道。

在本节中，您将学习如何设置具有多个主服务器的从服务器。这种方法与在通道上设置传统复制相同。

# 如何做...

假设您要将`server3`设置为`server1`和`server2`的从服务器。您需要从`server1`到`server3`创建传统复制通道，并从`server2`到`server3`创建另一个通道。为了确保从服务器上的数据一致，请确保复制不同的数据库集或应用程序处理冲突。

开始之前，请从 server1 备份并在`server3`上恢复；类似地，从`server2`备份并在`server3`上恢复，如第九章“复制”中所述。

1.  在`server3`上，将复制存储库从`FILE`修改为`TABLE`。您可以通过运行以下命令动态更改它：

```go
mysql> STOP SLAVE; //If slave is already running
mysql> SET GLOBAL master_info_repository = 'TABLE';
mysql> SET GLOBAL relay_log_info_repository = 'TABLE';
```

还要更改配置文件：

```go
shell> sudo vi /etc/my.cnf
[mysqld]
master-info-repository=TABLE 
relay-log-info-repository=TABLE
```

1.  在`server3`上，执行`CHANGE MASTER TO`命令，使其成为`server1`的从服务器，通道名为`master-1`。您可以随意命名：

```go
mysql> CHANGE MASTER TO MASTER_HOST='server1', MASTER_USER='binlog_user', MASTER_PORT=3306, MASTER_PASSWORD='binlog_P@ss12', MASTER_LOG_FILE='server1.000017', MASTER_LOG_POS=788 FOR CHANNEL 'master-1';
```

1.  在`server3`上，执行`CHANGE MASTER TO`命令，使其成为`server2`的从服务器，通道为`master-2`：

```go
mysql> CHANGE MASTER TO MASTER_HOST='server2', MASTER_USER='binlog_user', MASTER_PORT=3306, MASTER_PASSWORD='binlog_P@ss12', MASTER_LOG_FILE='server2.000014', MASTER_LOG_POS=75438 FOR CHANNEL 'master-2';
```

1.  对于每个通道，执行`START SLAVE FOR CHANNEL`语句如下：

```go
mysql> START SLAVE FOR CHANNEL 'master-1';
Query OK, 0 rows affected (0.01 sec)

mysql> START SLAVE FOR CHANNEL 'master-2';
Query OK, 0 rows affected (0.00 sec)
```

1.  通过执行`SHOW SLAVE STATUS`语句验证从服务器的状态：

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: server1
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server1.000017
          Read_Master_Log_Pos: 788
               Relay_Log_File: server3-relay-bin-master@002d1.000002
                Relay_Log_Pos: 318
        Relay_Master_Log_File: server1.000017
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 788
              Relay_Log_Space: 540
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 32
                  Master_UUID: 7cc7fca7-4deb-11e7-a53e-42010a940002
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: master-1
           Master_TLS_Version: 
*************************** 2\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: server2
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server2.000014
          Read_Master_Log_Pos: 75438
               Relay_Log_File: server3-relay-bin-master@002d2.000002
                Relay_Log_Pos: 322
        Relay_Master_Log_File: server2.000014
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 75438
              Relay_Log_Space: 544
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 32
                  Master_UUID: b52ef45a-7ff4-11e7-9091-42010a940003
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: master-2
           Master_TLS_Version: 
2 rows in set (0.00 sec)
```

1.  要获取特定通道的从服务器状态，请执行：

```go
mysql> SHOW SLAVE STATUS FOR CHANNEL 'master-1' \G
```

1.  这是您可以使用性能模式监视指标的另一种方法：

```go
mysql> SELECT * FROM performance_schema.replication_connection_status\G
*************************** 1\. row ***************************
                                      CHANNEL_NAME: master-1
                                        GROUP_NAME: 
                                       SOURCE_UUID: 7cc7fca7-4deb-11e7-a53e-42010a940002
                                         THREAD_ID: 36
                                     SERVICE_STATE: ON
                         COUNT_RECEIVED_HEARTBEATS: 73
                          LAST_HEARTBEAT_TIMESTAMP: 2017-09-15 12:42:10.910051
                          RECEIVED_TRANSACTION_SET: 
                                 LAST_ERROR_NUMBER: 0
                                LAST_ERROR_MESSAGE: 
                              LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                           LAST_QUEUED_TRANSACTION: 
 LAST_QUEUED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
LAST_QUEUED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
     LAST_QUEUED_TRANSACTION_START_QUEUE_TIMESTAMP: 0000-00-00 00:00:00.000000
       LAST_QUEUED_TRANSACTION_END_QUEUE_TIMESTAMP: 0000-00-00 00:00:00.000000
                              QUEUEING_TRANSACTION: 
    QUEUEING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
   QUEUEING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
        QUEUEING_TRANSACTION_START_QUEUE_TIMESTAMP: 0000-00-00 00:00:00.000000
*************************** 2\. row ***************************
                                      CHANNEL_NAME: master-2
                                        GROUP_NAME: 
                                       SOURCE_UUID: b52ef45a-7ff4-11e7-9091-42010a940003
                                         THREAD_ID: 38
                                     SERVICE_STATE: ON
                         COUNT_RECEIVED_HEARTBEATS: 73
                          LAST_HEARTBEAT_TIMESTAMP: 2017-09-15 12:42:13.986271
                          RECEIVED_TRANSACTION_SET: 
                                 LAST_ERROR_NUMBER: 0
                                LAST_ERROR_MESSAGE: 
                              LAST_ERROR_TIMESTAMP: 0000-00-00 00:00:00.000000
                           LAST_QUEUED_TRANSACTION: 
 LAST_QUEUED_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
LAST_QUEUED_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
     LAST_QUEUED_TRANSACTION_START_QUEUE_TIMESTAMP: 0000-00-00 00:00:00.000000
       LAST_QUEUED_TRANSACTION_END_QUEUE_TIMESTAMP: 0000-00-00 00:00:00.000000
                              QUEUEING_TRANSACTION: 
    QUEUEING_TRANSACTION_ORIGINAL_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
   QUEUEING_TRANSACTION_IMMEDIATE_COMMIT_TIMESTAMP: 0000-00-00 00:00:00.000000
        QUEUEING_TRANSACTION_START_QUEUE_TIMESTAMP: 0000-00-00 00:00:00.000000
2 rows in set (0.00 sec)
```

您可以通过附加`FOR CHANNEL 'channel_name'`指定通道的所有与从服务器相关的命令：

```go
mysql> STOP SLAVE FOR CHANNEL 'master-1';
mysql> RESET SLAVE FOR CHANNEL 'master-2';
```

# 设置复制过滤器

您可以控制要复制的表或数据库。在主服务器上，您可以使用`--binlog-do-db`和`--binlog-ignore-db`选项控制要为其记录更改的数据库，如第六章“二进制日志”中所述。更好的方法是在从服务器端进行控制。您可以使用`--replicate-*`选项或通过创建复制过滤器动态地执行或忽略从主服务器接收的语句。

# 如何做...

要创建过滤器，您需要执行`CHANGE REPLICATION FILTER`语句。

# 仅复制数据库

假设您只想复制`db1`和`db2`。使用以下语句创建复制过滤器。

```go
mysql> CHANGE REPLICATION FILTER REPLICATE_DO_DB = (db1, db2);
```

请注意，您应该在括号内指定所有数据库。

# 复制特定表

您可以使用`REPLICATE_DO_TABLE`指定要复制的表：

```go
mysql> CHANGE REPLICATION FILTER REPLICATE_DO_TABLE = ('db1.table1'); 
```

假设您想要对表使用正则表达式；您可以使用`REPLICATE_WILD_DO_TABLE`选项：

```go
mysql> CHANGE REPLICATION FILTER REPLICATE_WILD_DO_TABLE = ('db1.imp%'); 
```

您可以使用各种`IGNORE`选项使用正则表达式提及一些数据库或表。

# 忽略数据库

就像您可以选择复制数据库一样，您可以使用`REPLICATE_IGNORE_DB`忽略复制中的数据库：

```go
mysql> CHANGE REPLICATION FILTER REPLICATE_IGNORE_DB = (db1, db2);
```

# 忽略特定表

您可以使用`REPLICATE_IGNORE_TABLE`和`REPLICATE_WILD_IGNORE_TABLE`选项忽略某些表。`REPLICATE_WILD_IGNORE_TABLE`选项允许使用通配符字符，而`REPLICATE_IGNORE_TABLE`仅接受完整的表名：

```go
mysql> CHANGE REPLICATION FILTER REPLICATE_IGNORE_TABLE = ('db1.table1'); 
mysql> CHANGE REPLICATION FILTER REPLICATE_WILD_IGNORE_TABLE = ('db1.new%', 'db2.new%'); 
```

您还可以通过指定通道名称为通道设置过滤器：

```go
mysql> CHANGE REPLICATION FILTER REPLICATE_DO_DB = (d1) FOR CHANNEL 'master-1';
```

# 另请参阅

有关复制过滤器的更多详细信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/change-replication-filter.html`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-filter.html)。如果您使用多个过滤器，请参阅[`dev.mysql.com/doc/refman/8.0/en/replication-rules.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-rules.html)以了解有关 MySQL 如何评估过滤器的更多信息。

# 将从主从复制切换到链式复制

如果您设置了主从复制，服务器 B 和 C 从服务器 A 复制：服务器 A -->（服务器 B，服务器 C），并且您希望将服务器 C 设置为服务器 B 的从服务器，则必须在服务器 B 和服务器 C 上停止复制。然后使用`START SLAVE UNTIL`命令将它们带到相同的主日志位置。之后，您可以从服务器 B 获取主日志坐标，并在服务器 C 上执行`CHANGE MASTER TO`命令。

# 如何做...

1.  **在服务器 C 上**：停止从服务器并注意`SHOW SLAVE STATUS\G`命令中的`Relay_Master_Log_File`和`Exec_Master_Log_Pos`位置：

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: 
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_A-bin.000023
          Read_Master_Log_Pos: 2604
               Relay_Log_File: server_C-relay-bin.000002
                Relay_Log_Pos: 1228
 Relay_Master_Log_File: server_A-bin.000023
~
 Exec_Master_Log_Pos: 2604
              Relay_Log_Space: 1437
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
~
1 row in set (0.00 sec)
```

1.  **在服务器 B 上**：停止从服务器并注意`SHOW SLAVE STATUS\G`命令中的`Relay_Master_Log_File`和`Exec_Master_Log_Pos`位置：

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: 
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_A-bin.000023
          Read_Master_Log_Pos: 8250241
               Relay_Log_File: server_B-relay-bin.000002
                Relay_Log_Pos: 1228
 Relay_Master_Log_File: server_A-bin.000023
~
 Exec_Master_Log_Pos: 8250241
              Relay_Log_Space: 8248167
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
~
1 row in set (0.00 sec)
```

1.  比较服务器 B 的日志位置和服务器 C，找出与服务器 A 最新的同步。通常，由于您首先在服务器 C 上停止了从服务器，服务器 B 将领先。在我们的情况下，日志位置是：

服务器 C：（`server_A-bin.000023`，`2604`）

服务器 B：（`server_A-bin.000023`，`8250241`）

服务器 B 领先，所以我们必须将服务器 C 带到服务器 B 的位置。

1.  **在服务器 C 上**：使用`START SLAVE UNTIL`语句同步到服务器 B 的位置：

```go
mysql> START SLAVE UNTIL MASTER_LOG_FILE='centos7-bin.000023', MASTER_LOG_POS=8250241;
Query OK, 0 rows affected, 1 warning (0.03 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1278
Message: It is recommended to use --skip-slave-start when doing step-by-step replication with START SLAVE UNTIL; otherwise, you will get problems if you get an unexpected slave's mysqld restart
1 row in set (0.00 sec)
```

1.  **在服务器 C 上**：等待服务器 C 追上，通过检查`SHOW SLAVE STATUS`输出中的`Exec_Master_Log_Pos`和`Until_Log_Pos`（两者应该相同）：

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_A-bin.000023
          Read_Master_Log_Pos: 8250241
               Relay_Log_File: server_C-relay-bin.000003
                Relay_Log_Pos: 8247959
        Relay_Master_Log_File: server_A-bin.000023
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
~
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
 Exec_Master_Log_Pos: 8250241
              Relay_Log_Space: 8249242
              Until_Condition: Master
               Until_Log_File: server_A-bin.000023
 Until_Log_Pos: 8250241
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
 Seconds_Behind_Master: NULL
~
1 row in set (0.00 sec)
```

1.  **在服务器 B 上**：查找主状态，启动从服务器，并确保它正在复制：

```go
mysql> SHOW MASTER STATUS;
+---------------------+----------+--------------+------------------+-------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------------+----------+--------------+------------------+-------------------+
| server_B-bin.000003 | 36379324 |              |                  |                   |
+---------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

mysql> START SLAVE;
Query OK, 0 rows affected (0.02 sec)

mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: 
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_A-bin.000023
          Read_Master_Log_Pos: 8250241
               Relay_Log_File: server_B-relay-bin.000002
                Relay_Log_Pos: 1228
 Relay_Master_Log_File: server_A-bin.000023
~
 Exec_Master_Log_Pos: 8250241
              Relay_Log_Space: 8248167
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
~
1 row in set (0.00 sec)
```

1.  **在服务器 C 上**：停止从服务器，执行`CHANGE MASTER TO`命令，并指向服务器 B。您必须使用从上一步中获得的位置：

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.04 sec)

mysql> CHANGE MASTER TO MASTER_HOST = 'Server B', MASTER_USER = 'binlog_user', MASTER_PASSWORD = 'binlog_P@ss12', MASTER_LOG_FILE='server_B-bin.000003', MASTER_LOG_POS=36379324;
Query OK, 0 rows affected, 1 warning (0.04 sec)
```

1.  **在服务器 C 上**：启动复制并验证从服务器状态：

```go
mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW SLAVE STATUS\G
Query OK, 0 rows affected, 1 warning (0.00 sec)

*************************** 1\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xxx.xxx.xx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_B-bin.000003
          Read_Master_Log_Pos: 36380416
               Relay_Log_File: server_C-relay-bin.000002
                Relay_Log_Pos: 1413
        Relay_Master_Log_File: server_B-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
 ~
          Exec_Master_Log_Pos: 36380416
              Relay_Log_Space: 1622
 ~
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
~
1 row in set (0.00 sec)
```

# 将从链式复制切换为主从复制

如果您设置了链式复制（例如服务器 A --> 服务器 B --> 服务器 C）并且希望使服务器 C 成为服务器 A 的直接从服务器，则必须在服务器 B 上停止复制，让服务器 C 追上服务器 B，然后找到服务器 A 对应于服务器 B 停止位置的坐标。使用这些坐标，您可以在服务器 C 上执行`CHANGE MASTER TO`命令，并使其成为服务器 A 的从服务器。

# 如何做...

1.  **在服务器 B 上**：停止从服务器并记录主状态：

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.04 sec)

mysql> SHOW MASTER STATUS;
+---------------------+----------+--------------+------------------+-------------------+
| File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------------+----------+--------------+------------------+-------------------+
| server_B-bin.000003 | 44627878 |              |                  |                   |
+---------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
```

1.  **在服务器 C 上**：确保从服务器延迟已经追上。`Relay_Master_Log_File`和`Exec_Master_Log_Pos`应该等于服务器 B 上主状态的输出。一旦延迟追上，停止从服务器：

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 35.186.157.16
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_B-bin.000003
          Read_Master_Log_Pos: 44627878
               Relay_Log_File: ubuntu2-relay-bin.000002
                Relay_Log_Pos: 8248875
 Relay_Master_Log_File: server_B-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
 Exec_Master_Log_Pos: 44627878
              Relay_Log_Space: 8249084
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
 Seconds_Behind_Master: 0
~
1 row in set (0.00 sec)
```

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.01 sec)
```

1.  **在服务器 B 上**：从`SHOW SLAVE STATUS`输出中获取服务器 A 的坐标（注意`Relay_Master_Log_File`和`Exec_Master_Log_Pos`）并启动从服务器：

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: 
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_A-bin.000023
          Read_Master_Log_Pos: 16497695
               Relay_Log_File: server_B-relay-bin.000004
                Relay_Log_Pos: 8247776
 Relay_Master_Log_File: server_A-bin.000023
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
 Exec_Master_Log_Pos: 16497695
              Relay_Log_Space: 8248152
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
```

```go
mysql> START SLAVE;
Query OK, 0 rows affected (0.01 sec)
```

1.  **在服务器 C 上**：停止从服务器并执行`CHANGE MASTER TO COMMAND`指向服务器 A。使用从上一步中记录的位置（`server_A-bin.000023`和`16497695`）。最后启动从服务器并验证从服务器状态：

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.07 sec)
```

```go
mysql> CHANGE MASTER TO MASTER_HOST = 'Server A', MASTER_USER = 'binlog_user', MASTER_PASSWORD = 'binlog_P@ss12', MASTER_LOG_FILE='server_A-bin.000023', MASTER_LOG_POS=16497695;
Query OK, 0 rows affected, 1 warning (0.02 sec)
```

```go
mysql> START SLAVE;
Query OK, 0 rows affected (0.07 sec)

mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: 
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_A-bin.000023
          Read_Master_Log_Pos: 16497695
               Relay_Log_File: server_C-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: server_A-bin.000023
             Slave_IO_Running: No
            Slave_SQL_Running: No
  ~
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 16497695
              Relay_Log_Space: 154
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
~
1 row in set (0.00 sec)
```

# 设置延迟复制

有时，您需要一个延迟的从服务器用于灾难恢复目的。假设主服务器上执行了灾难性语句（如`DROP DATABASE`命令）。您必须使用备份的*时间点恢复*来恢复数据库。这将导致巨大的停机时间，具体取决于数据库的大小。为了避免这种情况，您可以使用延迟的从服务器，它将始终比主服务器延迟一定的时间。如果发生灾难并且该语句未被延迟的从服务器应用，您可以停止从服务器并启动直到灾难性语句，以便灾难性语句不会被执行。然后将其提升为主服务器。

该过程与设置正常复制完全相同，只是在`CHANGE MASTER TO`命令中指定`MASTER_DELAY`。

**延迟是如何衡量的？**

在 MySQL 8.0 之前的版本中，延迟是基于`Seconds_Behind_Master`值来衡量的。在 MySQL 8.0 中，它是基于`original_commit_timestamp`和`immediate_commit_timestamp`来衡量的，这些值写入了二进制日志。

`original_commit_timestamp`是事务写入（提交）到原始主服务器的二进制日志时距离时代开始的微秒数。

`immediate_commit_timestamp`是事务写入（提交）到直接主服务器的二进制日志时距离时代开始的微秒数。

# 如何做...

1.  停止从服务器：

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.06 sec)
```

1.  执行`CHANGE MASTER TO MASTER_DELAY =`并启动从服务器。假设您想要 1 小时的延迟，您可以将`MASTER_DELAY`设置为`3600`秒：

```go
mysql> CHANGE MASTER TO MASTER_DELAY = 3600;
Query OK, 0 rows affected (0.04 sec)

mysql> START SLAVE;
Query OK, 0 rows affected (0.00 sec)
```

1.  在从服务器状态中检查以下内容：

`SQL_Delay`: 从服务器必须滞后主服务器的秒数。

`SQL_Remaining_Delay`: 延迟剩余的秒数。当存在延迟时，此值为 NULL。

`Slave_SQL_Running_State`: SQL 线程的状态。

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 35.186.158.188
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server_A-bin.000023
          Read_Master_Log_Pos: 24745149
               Relay_Log_File: server_B-relay-bin.000002
                Relay_Log_Pos: 322
        Relay_Master_Log_File: server_A-bin.000023
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
~
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 16497695
              Relay_Log_Space: 8247985
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
~
 Seconds_Behind_Master: 52
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
~
 SQL_Delay: 3600
 SQL_Remaining_Delay: 3549
 Slave_SQL_Running_State: Waiting until MASTER_DELAY seconds after master executed event
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
~
1 row in set (0.00 sec)
```

请注意，一旦延迟被维持，`Seconds_Behind_Master`将显示为`0`。

# 设置 GTID 复制

**全局事务标识符**（**GTID**）是在原始服务器（主服务器）上提交的每个事务创建并关联的唯一标识符。此标识符不仅对于其起源服务器是唯一的，而且对于给定复制设置中的所有服务器也是唯一的。所有事务和所有 GTID 之间存在一对一的映射关系。

GTID 表示为一对坐标，用冒号（`:`）分隔。

```go
GTID = source_id:transaction_id
```

`source_id`选项标识了原始服务器。通常，服务器的`server_uuid`选项用于此目的。`transaction_id`选项是由事务在此服务器上提交的顺序确定的序列号。例如，第一个提交的事务其`transaction_id`为`1`，在同一原始服务器上提交的第十个事务被分配了`transaction_id`为`10`。

正如您在之前的方法中所看到的，您必须在复制的起点上提到二进制日志文件和位置。如果您要将一个从服务器从一个主服务器切换到另一个主服务器，特别是在故障转移期间，您必须从新主服务器获取位置以同步从服务器，这可能很痛苦。为了避免这些问题，您可以使用基于 GTID 的复制，MySQL 会自动使用 GTID 检测二进制日志位置。

# 如何操作...

如果服务器之间已经设置了复制，请按照以下步骤操作：

1.  在`my.cnf`中启用 GTID：

```go
shell> sudo vi /etc/my.cnf [mysqld]gtid_mode=ON
enforce-gtid-consistency=true
skip_slave_start
```

1.  将主服务器设置为只读，并确保所有从服务器与主服务器保持同步。这非常重要，因为主服务器和从服务器之间不应该存在任何数据不一致。

```go
On master mysql> SET @@global.read_only = ON; On Slaves (if replication is already setup) mysql> SHOW SLAVE STATUS\G
```

1.  重新启动所有从服务器以使 GTID 生效。由于在配置文件中给出了`skip_slave_start`，从服务器在指定`START SLAVE`命令之前不会启动。如果启动从服务器，它将失败，并显示此错误——`The replication receiver thread cannot start because the master has GTID_MODE = OFF and this server has GTID_MODE = ON`。

```go
shell> sudo systemctl restart mysql
```

1.  重新启动主服务器。重新启动主服务器后，它将以读写模式开始，并在 GTID 模式下开始接受写操作：

```go
shell> sudo systemctl restart mysql
```

1.  执行`CHANGE MASTER TO`命令以设置 GTID 复制：

```go
mysql> CHANGE MASTER TO MASTER_HOST = <master_host>, MASTER_PORT = <port>, MASTER_USER = 'binlog_user', MASTER_PASSWORD = 'binlog_P@ss12', MASTER_AUTO_POSITION = 1;
```

您可以观察到二进制日志文件和位置未给出；相反，给出了`MASTER_AUTO_POSITION`，它会自动找到已执行的 GTID。

1.  在所有从服务器上执行`START SLAVE`：

```go
mysql> START SLAVE;
```

1.  验证从服务器是否正在复制：

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: xx.xxx.xxx.xxx
                  Master_User: binlog_user
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: server1-bin.000002
          Read_Master_Log_Pos: 345
               Relay_Log_File: server2-relay-bin.000002
                Relay_Log_Pos: 562
        Relay_Master_Log_File: server1-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 345
              Relay_Log_Space: 770
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
 Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 32
                  Master_UUID: b52ef45a-7ff4-11e7-9091-42010a940003
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: b52ef45a-7ff4-11e7-9091-42010a940003:1
            Executed_Gtid_Set: b52ef45a-7ff4-11e7-9091-42010a940003:1
 Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

要了解有关 GTID 的更多信息，请参阅[`dev.mysql.com/doc/refman/5.6/en/replication-gtids-concepts.html`](https://dev.mysql.com/doc/refman/5.6/en/replication-gtids-concepts.html)。

# 设置半同步复制

默认情况下，复制是异步的。主服务器不知道写操作是否已到达从服务器。如果主服务器和从服务器之间存在延迟，并且主服务器崩溃，那么尚未到达从服务器的数据将会丢失。为了克服这种情况，您可以使用半同步复制。

在半同步复制中，主服务器会等待至少一个从服务器接收写操作。默认情况下，`rpl_semi_sync_master_wait_point`的值为`AFTER_SYNC`；这意味着主服务器将事务同步到从服务器消耗的二进制日志中。

之后，从服务器向主服务器发送确认，然后主服务器提交事务并将结果返回给客户端。因此，如果写入已到达中继日志，则从服务器无需提交事务。您可以通过将变量`rpl_semi_sync_master_wait_point`更改为`AFTER_COMMIT`来更改此行为。在这种情况下，主服务器将事务提交给存储引擎，但不将结果返回给客户端。一旦从服务器上提交了事务，主服务器将收到事务的确认，然后将结果返回给客户端。

如果要在更多从服务器上确认事务，可以增加动态变量`rpl_semi_sync_master_wait_for_slave_count`的值。您还可以通过动态变量`rpl_semi_sync_master_timeout`设置主服务器必须等待从服务器确认的毫秒数；默认值为`10`秒。

在完全同步复制中，主服务器会等待直到所有从服务器都提交了事务。要实现这一点，您必须使用 Galera Cluster。

# 如何做…

在高层次上，您需要在主服务器和所有希望进行半同步复制的从服务器上安装和启用半同步插件。您必须重新启动从服务器 IO 线程以使更改生效。您可以根据您的网络和应用程序调整`rpl_semi_sync_master_timeout`的值。`1`秒的值是一个很好的起点：

1.  在主服务器上，安装`rpl_semi_sync_master`插件：

```go
mysql> INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
Query OK, 0 rows affected (0.86 sec)
```

验证插件是否已激活：

```go
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME LIKE '%semi%';
+----------------------+---------------+
| PLUGIN_NAME          | PLUGIN_STATUS |
+----------------------+---------------+
| rpl_semi_sync_master | ACTIVE        |
+----------------------+---------------+
1 row in set (0.01 sec)
```

1.  在主服务器上，启用半同步复制并调整超时时间（比如 1 秒）：

```go
mysql> SET @@GLOBAL.rpl_semi_sync_master_enabled=1;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'rpl_semi_sync_master_enabled';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| rpl_semi_sync_master_enabled | ON    |
+------------------------------+-------+
1 row in set (0.00 sec)

mysql> SET @@GLOBAL.rpl_semi_sync_master_timeout=1000;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW VARIABLES LIKE 'rpl_semi_sync_master_timeout';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| rpl_semi_sync_master_timeout | 1000  |
+------------------------------+-------+
1 row in set (0.00 sec)
```

1.  在从服务器上，安装`rpl_semi_sync_slave`插件：

```go
mysql> INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
Query OK, 0 rows affected (0.22 sec)

mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS FROM INFORMATION_SCHEMA.PLUGINS WHERE PLUGIN_NAME LIKE '%semi%';
+---------------------+---------------+
| PLUGIN_NAME         | PLUGIN_STATUS |
+---------------------+---------------+
| rpl_semi_sync_slave | ACTIVE        |
+---------------------+---------------+
1 row in set (0.08 sec)
```

1.  在从服务器上，启用半同步复制并重新启动从服务器 IO 线程：

```go
mysql> SET GLOBAL rpl_semi_sync_slave_enabled = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> STOP SLAVE IO_THREAD;
Query OK, 0 rows affected (0.02 sec)

mysql> START SLAVE IO_THREAD;
Query OK, 0 rows affected (0.00 sec)
```

1.  您可以通过以下方式监视半同步复制的状态：

要查找连接为半同步的客户端数量，请在主服务器上执行：

```go
mysql> SHOW STATUS LIKE 'Rpl_semi_sync_master_clients';
+------------------------------+-------+
| Variable_name                | Value |
+------------------------------+-------+
| Rpl_semi_sync_master_clients | 1     |
+------------------------------+-------+
1 row in set (0.01 sec)
```

当超时发生并且从服务器赶上时，主服务器在异步和半同步复制之间切换。要检查主服务器正在使用的复制类型，请检查`Rpl_semi_sync_master_status`的状态（打开表示半同步，关闭表示异步）：

```go
mysql> SHOW STATUS LIKE 'Rpl_semi_sync_master_status';
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| Rpl_semi_sync_master_status | ON    |
+-----------------------------+-------+
1 row in set (0.00 sec)
```

您可以使用此方法验证半同步复制：

1.  停止从服务器：

```go
mysql> STOP SLAVE;
Query OK, 0 rows affected (0.01 sec)
```

1.  在主服务器上，执行任何语句：

```go
mysql> USE employees;
Database changed

mysql> DROP TABLE IF EXISTS employees_test;
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

您会注意到主服务器已经切换到异步复制，因为即使在 1 秒后（`rpl_semi_sync_master_timeout`的值），它仍未收到从从服务器的任何确认：

```go
mysql> SHOW STATUS LIKE 'Rpl_semi_sync_master_status';
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| Rpl_semi_sync_master_status | ON    |
+-----------------------------+-------+
1 row in set (0.00 sec)

mysql> DROP TABLE IF EXISTS employees_test;
Query OK, 0 rows affected (1.02 sec)
```

```go

mysql> SHOW STATUS LIKE 'Rpl_semi_sync_master_status';
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| Rpl_semi_sync_master_status | OFF   |
+-----------------------------+-------+
1 row in set (0.01 sec
```

1.  启动从服务器：

```go
mysql> START SLAVE;
Query OK, 0 rows affected (0.02 sec)
```

1.  在主服务器上，您会注意到主服务器已经切换回半同步复制。

```go
mysql> SHOW STATUS LIKE 'Rpl_semi_sync_master_status';
+-----------------------------+-------+
| Variable_name               | Value |
+-----------------------------+-------+
| Rpl_semi_sync_master_status | ON    |
+-----------------------------+-------+
1 row in set (0.00 sec)
```
