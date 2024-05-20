# 第四章：配置 MySQL

在本章中，我们将涵盖以下配方：

+   使用配置文件

+   使用全局和会话变量

+   使用启动脚本的参数

+   配置参数

+   更改数据目录

# 介绍

MySQL 有两种类型的参数：

+   **静态**，在重新启动 MySQL 服务器后生效

+   **动态**，可以在不重新启动 MySQL 服务器的情况下进行更改

变量可以通过以下方式设置：

+   **配置文件**：MySQL 有一个配置文件，我们可以在其中指定数据的位置，MySQL 可以使用的内存，以及各种其他参数。

+   **启动脚本**：您可以直接将参数传递给`mysqld`进程。它仅在服务器的该调用中生效。

+   **使用 SET 命令**（仅动态变量）：这将持续到服务器重新启动。您还需要在配置文件中设置变量，以使更改在重新启动时持久。使更改持久的另一种方法是在变量名称之前加上`PERSIST`关键字或`@@persist`。

# 使用配置文件

默认配置文件为`/etc/my.cnf`（在 Red Hat 和 CentOS 系统上）和`/etc/mysql/my.cnf`（Debian 系统）。在您喜欢的编辑器中打开文件并根据需要修改参数。本章讨论了主要参数。

# 如何做...

配置文件由`section_name`指定的部分。所有与部分相关的参数都可以放在它们下面，例如：

```go
[mysqld] <---section name <parameter_name> = <value> <---parameter values
[client] <parameter_name> = <value>
[mysqldump] <parameter_name> = <value>
[mysqld_safe] <parameter_name> = <value>
[server]
<parameter_name> = <value>
```

+   `[mysql]`：该部分由`mysql`命令行客户端读取

+   `[client]`：该部分由所有连接的客户端（包括`mysql cli`）读取

+   `[mysqld]`：该部分由`mysql`服务器读取

+   `[mysqldump]`：该部分由名为`mysqldump`的备份实用程序读取

+   `[mysqld_safe]`：由`mysqld_safe`进程（MySQL 服务器启动脚本）读取

除此之外，`mysqld_safe`进程从选项文件中的`[mysqld]`和`[server]`部分读取所有选项。

例如，`mysqld_safe`进程从`mysqld`部分读取`pid-file`选项。

```go
shell> sudo vi /etc/my.cnf
[mysqld]
pid-file = /var/lib/mysql/mysqld.pid
```

在使用`systemd`的系统中，`mysqld_safe`将不会安装。要配置启动脚本，您需要在`/etc/systemd/system/mysqld.service.d/override.conf`中设置值。

例如：

```go
[Service]
LimitNOFILE=max_open_files
PIDFile=/path/to/pid/file
LimitCore=core_file_limit
Environment="LD_PRELOAD=/path/to/malloc/library"
Environment="TZ=time_zone_setting"
```

# 使用全局和会话变量

正如您在前几章中所看到的，您可以通过连接到 MySQL 并执行`SET`命令来设置参数。

根据变量的范围，有两种类型的变量：

+   **全局**：适用于所有新连接

+   **会话**：仅适用于当前连接（会话）

# 如何做...

例如，如果您想记录所有慢于一秒的查询，可以执行：

```go
mysql> SET GLOBAL long_query_time = 1;
```

要使更改在重新启动时持久，请使用：

```go
mysql> SET PERSIST long_query_time = 1;
Query OK, 0 rows affected (0.01 sec)
```

或：

```go
mysql> SET @@persist.long_query_time = 1;
Query OK, 0 rows affected (0.00 sec)
```

持久的全局系统变量设置存储在位于数据目录中的 mysqld-auto.cnf 中。

假设您只想记录此会话的查询，而不是所有连接的查询。您可以使用以下命令：

```go
mysql> SET SESSION long_query_time = 1;
```

# 使用启动脚本的参数

假设您希望使用启动脚本启动 MySQL，而不是通过`systemd`，特别是用于测试或进行一些临时更改。您可以将变量传递给脚本，而不是在配置文件中更改它。

# 如何做...

```go
shell> /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/usr/local/mysql/data/centos7.err --pid-file=/usr/local/mysql/data/centos7.pid --init-file=/tmp/mysql-init &
```

您可以看到`--init-file`参数被传递给服务器。服务器在启动之前执行该文件中的 SQL 语句。

# 配置参数

安装后，您需要配置的基本事项在本节中都有所涵盖。其余的都可以保持默认或根据负载稍后进行调整。

# 如何做...

让我们深入了解。

# 数据目录

由 MySQL 服务器管理的数据存储在一个名为`数据目录`的目录下。`数据目录`的每个子目录都是一个数据库目录，对应于服务器管理的数据库。默认情况下，

`数据目录`有三个子目录：

+   `mysql`：MySQL 系统数据库

+   `performance_schema`：提供了用于在运行时检查服务器内部执行的信息

+   `sys`：提供了一组对象，以帮助更轻松地解释性能模式信息

除此之外，`data directory`还包含日志文件、`InnoDB`表空间和`InnoDB`日志文件、SSL 和 RSA 密钥文件、`mysqld`的`pid`以及`mysqld-auto.cnf`，其中存储了持久化的全局系统变量设置。

要设置`data directory`的更改/添加`datadir`的值到配置文件。默认值为`/var/lib/mysql`：

```go
shell> sudo vi /etc/my.cnf
[mysqld]
datadir = /data/mysql
```

你可以将其设置为任何你想要存储数据的地方，但你应该将`data directory`的所有权更改为`mysql`。

确保承载`data directory`的磁盘卷有足够的空间来容纳所有的数据。

# innodb_buffer_pool_size

这是决定`InnoDB`存储引擎可以使用多少内存来缓存数据和索引的最重要的调整参数。设置得太低会降低 MySQL 服务器的性能，设置得太高会增加 MySQL 进程的内存消耗。MySQL 8 最好的地方在于`innodb_buffer_pool_size`是动态的，这意味着你可以在不重新启动服务器的情况下改变`innodb_buffer_pool_size`。

以下是如何调整它的简单指南：

1.  查找数据集的大小。不要将`innodb_buffer_pool_size`的值设置得高于数据集的大小。假设你有 12GB 的 RAM 机器，你的数据集大小为 3GB；那么你可以将`innodb_buffer_pool_size`设置为 3GB。如果你预计数据会增长，你可以根据需要随时增加它，而无需重新启动 MySQL。

1.  通常，数据集的大小要比可用的 RAM 大得多。在总 RAM 中，你可以为操作系统、其他进程、MySQL 内部的每个线程缓冲区和`InnoDB`之外的 MySQL 服务器分配一些内存。剩下的可以分配给`InnoDB`缓冲池大小。

这是一个非常通用的表，为你提供了一个很好的起点，假设它是一个专用的 MySQL 服务器，所有的表都是`InnoDB`，每个线程的缓冲区都保持默认值。如果系统内存不足，你可以动态减少缓冲池。

| **RAM** | **缓冲池大小（范围）** |
| --- | --- |
| 4 GB | 1 GB-2 GB |
| 8 GB | 4 GB-6 GB |
| 12 GB | 6 GB-10 GB |
| 16 GB | 10 GB-12 GB |
| 32 GB | 24 GB-28 GB |
| 64 GB | 45 GB-56 GB |
| 128 GB | 108 GB-116 GB |
| 256 GB | 220 GB-245 GB |

# innodb_buffer_pool_instances

你可以将`InnoDB`缓冲池划分为单独的区域，以提高并发性，通过减少不同线程对缓存页面的读写而产生的争用。例如，如果缓冲池大小为 64GB，`innodb_buffer_pool_instances`为 32，那么缓冲池将被分割成 32 个每个 2GB 的区域。

如果缓冲池大小超过 16GB，你可以设置实例，以便每个区域至少获得 1GB 的空间。

# innodb_log_file_size

这是用于在数据库崩溃时重放已提交事务的重做日志空间的大小。默认值为 48MB，这对于生产工作负载可能不够。你可以先设置为 1GB 或 2GB。这个更改需要重新启动。停止 MySQL 服务器，并确保它在没有错误的情况下关闭。在`my.cnf`中进行更改并启动服务器。在早期版本中，你需要停止服务器，删除日志文件，然后启动服务器。在 MySQL 8 中，这是自动的。修改重做日志文件在第十一章中有解释，*管理表空间*，在*更改 InnoDB 重做日志文件的数量或大小*部分。

# 更改数据目录

你的数据可能会随着时间的推移而增长，当它超出文件系统时，你需要添加一个磁盘或将`data directory`移动到一个更大的卷中。

# 如何做...

1.  检查当前的`data directory`。默认情况下，`data directory`是`/var/lib/mysql`：

```go
mysql> show variables like '%datadir%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+
1 row in set (0.04 sec)
```

1.  停止`mysql`并确保它已成功停止：

```go
shell> sudo systemctl stop mysql
```

1.  检查状态：

```go
shell> sudo systemctl status mysql
```

应该显示`已停止 MySQL Community Server`。

1.  在新位置创建目录并将所有权更改为`mysql`：

```go
shell> sudo mkdir -pv /data
shell> sudo chown -R mysql:mysql /data/
```

1.  将文件移动到新的`data 目录`：

```go
shell> sudo rsync -av /var/lib/mysql /data
```

1.  在 Ubuntu 中，如果已启用 AppArmor，您需要配置访问控制：

```go
shell> vi /etc/apparmor.d/tunables/alias
alias /var/lib/mysql/ -> /data/mysql/,
shell> sudo systemctl restart apparmor
```

1.  启动 MySQL 服务器并验证`data`目录已更改：

```go
shell> sudo systemctl start mysql
mysql> show variables like '%datadir%'; 
+---------------+--------------+
| Variable_name | Value        |
+---------------+--------------+
| datadir       | /data/mysql/ |
+---------------+--------------+
1 row in set (0.00 sec)
```

1.  验证数据是否完好并删除旧的`data 目录`：

```go
shell> sudo rm -rf /var/lib/mysql
```

如果 MySQL 启动失败并显示错误—`MySQL 数据目录在/var/lib/mysql 未找到，请创建一个`：

执行`sudo mkdir /var/lib/mysql/mysql -p`

如果显示`MySQL 系统数据库未找到`，运行`mysql_install_db`工具，该工具将创建所需的目录。
