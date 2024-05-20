# 第十一章：MySQL 8 - 安全性

在之前的章节中，我们学习了 MySQL 8 的可伸缩性以及在扩展 MySQL 8 时如何解决挑战。除此之外，我们还学习了如何使 MySQL 8 具有高可用性。现在，安全对于任何应用程序都很重要，对吧？当我们谈论安全时，它包括帐户管理、角色、权限等。考虑到这些方面，我们将在本章中涵盖所有这些主题。本章主要关注 MySQL 8 数据库安全及其相关功能。本章涵盖以下主题：

+   MySQL 8 的安全概述

+   常见安全问题

+   MySQL 8 中的访问控制

+   MySQL 8 中的帐户管理

+   MySQL 8 中的加密

+   安全插件

# MySQL 8 的安全概述

安全这个术语不局限于特定主题；它涵盖了与 MySQL 8 相关的各种主题。在开始对其进行详细讨论之前，让我们提到与安全相关的一些重要要点：

+   考虑数据库中的安全性，需要管理用户及其与各种数据库对象相关的权限。

+   用户的密码安全。

+   在安装过程中进行安全配置，包括各种类型的文件，如日志文件、数据文件等。这些文件必须受到保护，以防止读/写操作。

+   为处理系统级故障场景，您必须拥有备份和恢复计划。这包括所有必需的文件，如数据库文件、配置文件等。

+   管理安装了 MySQL 8 的系统的网络安全，允许有限数量的主机进行连接。

现在，您的旅程将开始另一个重要且非常有趣的主题。我们开始吧。

# 常见安全问题

在深入讨论复杂问题之前，您必须首先了解一些基本要点，这将有助于防止滥用或攻击。

# 一般指南

在 MySQL 8 中，用户执行的所有连接、查询和操作都基于**访问控制列表**（**ACLs**）安全。以下是与安全相关的一些一般指南：

+   不要允许任何用户访问`user`表，除了 root 帐户。使用`GRANT`和`REVOKE`语句管理用户权限。

+   在进行互联网数据传输时，使用加密协议，如 SSH 或 SSL。MySQL 8 支持 SSL 连接。

+   在客户端使用应用程序将数据输入 MySQL 时，使用适当的防御性编程技术。

+   使用哈希函数将密码存储到 MySQL 8 数据库中；不要将明文存储为密码。对于密码恢复，考虑一些字符串作为盐，并使用`hash(hash(password)+salt)`值。

+   使用适当的密码策略来防止密码被破解。这意味着您的系统应该只接受符合您规则/约定的密码。

+   使用防火墙可以减少 50%的故障几率，并为您的系统提供更多保护。将 MySQL 定义在一个非军事区或防火墙后面，以防止来自不信任主机的攻击。

+   基于 Linux 的系统提供了`tcpdump`命令，以更安全的方式执行传输任务。该命令在网络层上提供安全性。例如，使用以下命令，您可以检查 MySQL 数据流是否加密：

```sql
        shell> tcpdump -l -i eth0 -w - src or dst port 3306 | strings
```

# 安全密码的指南

在本节中，我们描述了关于不同用户的密码安全的指南，并介绍了如何在登录过程中进行管理。MySQL 8 提供了`validate_password`插件来定义可接受密码的策略。

# 最终用户的指南

本节描述了定义密码的各种方法，作为最终用户，以最安全的方式。它解释了如何使您的密码更安全。最安全的方法是在受保护的选项文件中定义密码，或在客户端程序中提示输入密码。请参阅以下不同的定义密码的方式：

+   使用以下选项在命令行中提供密码：

```sql
 cmd>mysql -u root --password=your_pwd
 --OR
 cmd> 
```

+   在前两个命令中，您必须在命令行中指定密码，这是不可取的。MySQL 8 提供了另一种安全的连接方式。执行以下命令，它将提示您输入密码。一旦输入密码，MySQL 会为每个密码字符显示星号（`*`）：

```sql
 cmd>mysql -u root -p
 Enter password: *********
```

这比前两种方法更安全，前两种方法中，您在命令行参数中定义密码：

+   使用`MYSQL_PWD`环境变量来定义您的密码。与其他方法相比，这种方法是不安全的，因为环境变量可能被其他用户访问。

+   使用`mysql_config_editor`实用程序定义密码，这是一种提供的选项，用于将密码存储到加密的登录路径文件中，命名为`named.mylogin.cnf`。 MySQL 8 稍后将使用此文件与 MySQL 服务器连接。

+   使用选项文件存储密码。在将凭据定义到文件中时，请确保其他用户无法访问该文件。例如，在基于 UNIX 的系统中，您可以在客户端部分的选项文件中定义密码，如下所示：

```sql
 [client]
 password=your_pass
```

要使文件安全或设置其访问模式，请执行以下命令：

```sql
shell> chmod 600 .my.cnf
```

# 管理员指南

对于数据库管理员，应遵循以下准则来保护密码：

+   使用`validate_password`来对接受的密码应用策略

+   MySQL 8 使用`mysql.user`表来存储用户密码，因此配置系统以使只有管理员用户可以访问此表

+   用户应该被允许在密码过期的情况下重置帐户密码

+   如果日志文件包含密码，请对其进行保护

+   管理对插件目录和`my.cnf`文件的访问，因为它可以修改插件提供的功能

# 密码和日志记录

MySQL 8 允许您在 SQL 语句中以纯文本形式编写密码，例如`CREATE USER`，`SET PASSWORD`和`GRANT`。如果我们执行这些语句，MySQL 8 将密码以文本形式写入日志文件，并且所有可以访问日志文件的用户都可以看到。为了解决这个问题，避免使用上述 SQL 语句直接更新授权表。

# 保护 MYSQL 8 免受攻击

为了保护 MySQL 8 免受攻击，请强烈考虑以下几点：

+   为所有 MySQL 帐户设置密码。永远不要定义没有密码的帐户，因为这允许任何用户访问您的帐户。

+   与 MySQL 8 建立连接时，使用安全协议/通道，例如压缩协议，MySQL 8 内部 SSL 连接或用于加密 TCP/IP 连接的 SSH。

+   对于基于 Unix 的系统，为运行`mysqld`的 Unix 帐户设置数据目录的读/写权限。不要使用 root 用户启动 MySQL 8 服务器。

+   使用`secure_file_priv`变量指定读写权限的目录。使用此变量，您可以限制非管理员用户访问重要目录。使用此变量设置`plugin_dir`的权限非常重要。同样，不要向所有用户提供`FILE`权限，因为这允许用户在系统中的任何位置写文件。

+   使用`max_user_connections`变量限制每个帐户的连接数。

+   在创建授权表条目时，正确使用通配符。最好使用 IP 而不是 DNS。

+   在存储过程和视图创建期间遵循安全准则。

# MySQL 8 提供的安全选项和变量

MySQL 8 提供了以下选项和变量以确保安全：

| **名称** | **命令行** | **选项文件** | **系统变量** | **状态变量** | **变量范围** | **动态** |
| --- | --- | --- | --- | --- | --- | --- |
| `allow-suspicious-udfs` | 是 | 是 |   |   |   |   |
| `automatic_sp_privileges` |   |   | 是 |   | 全局 | 是 |
| - `chroot` | 是 | 是 |   |   |   |   |
| - `des-key-file` | 是 | 是 |   |   |   |   |
| - `local_infile` |   |   | 是 |   | 全局 | 是 |
| - `old_passwords` |   |   | 是 |   | 两者 | 是 |
| - `safe-user-create` | 是 | 是 |   |   |   |   |
| - `secure-auth` | 是 | 是 |   |   | 全局 | 是 |
| - `- 变量：secure_auth` |   |   | 是 |   | 全局 | 是 |
| - `secure-file-priv` | 是 | 是 |   |   | 全局 | 否 |
| - `- 变量：secure_file_priv` |   |   | 是 |   | 全局 | 否 |
| - `skip-grant-tables` | 是 | 是 |   |   |   |   |
| - `skip-name-resolve` | 是 | 是 |   |   | 全局 | 否 |
| - `- 变量：skip_name_resolve` |   |   | 是 |   | 全局 | 否 |
| - `skip-networking` | 是 | 是 |   |   | 全局 | 否 |
| - `- 变量：skip_networking` |   |   | 是 |   | 全局 | 否 |
| - `skip-show-database` | 是 | 是 |   |   | 全局 | 否 |
| - `- 变量：skip_show_database` |   |   | 是 |   | 全局 | 否 |

参考：[`dev.mysql.com/doc/refman/8.0/en/security-options.html`](https://dev.mysql.com/doc/refman/8.0/en/security-options.html)

# 客户端编程的安全指南

不要相信应用程序用户输入的任何数据，因为用户有可能输入了针对 MySQL 数据库的`drop`或`delete`语句。因此，存在安全漏洞和数据丢失的风险。作为 MySQL 数据库的管理员，应遵循以下检查表：

+   在将数据传递给 MySQL 8 之前，必须检查数据的大小。

+   为使 MySQL 8 更加严格，启用严格的 MySQL 模式。

+   对于数字字段，应输入字符、特殊字符和空格，而不是数字本身。在将字段值发送到 MySQL 8 服务器之前，通过应用程序将其更改为原始形式。

+   使用两个不同的用户进行应用程序连接到数据库和数据库管理。

+   通过在动态 URL 和 Web 表单的情况下将数据类型从数字更改为字符类型并添加引号来修改数据类型。还在动态 URL 中添加%22（"）、%23（#）和%27（'）。

先前定义的功能内置于所有编程接口中。例如，Java JDBC 提供带占位符的预编译语句，Ruby DBI 提供`quote()`方法。

# MySQL 8 中的访问控制

特权主要用于验证用户并验证用户凭据，检查用户是否被允许进行请求的操作。当我们连接到 MySQL 8 服务器时，它将首先通过提供的主机和用户名检查用户的身份。连接后，当请求到来时，系统将根据用户的身份授予特权。基于这一理解，我们可以说在使用客户端程序连接到 MySQL 8 服务器时，访问控制包含两个阶段：

+   **阶段 1**：MySQL 服务器将根据提供的身份接受或拒绝连接

+   **阶段 2**：从 MySQL 服务器获取连接后，当用户发送执行任何操作的请求时，服务器将检查用户是否具有足够的权限

MySQL 8 特权系统存在一些限制：

+   不允许用户在特定对象（如表或例程）上设置密码。MySQL 8 允许在账户级别全局设置密码。

+   作为管理员用户，我们不能以允许创建/删除表但不允许创建/删除该表的数据库的方式指定权限。

不允许显式限制用户访问，这意味着无法显式匹配用户并拒绝其连接。MySQL 8 在内存中管理授予表的内容，因此在`INSERT`、`UPDATE`和`DELETE`语句的情况下，对授予表的执行需要服务器重新启动才能生效。为了避免服务器重新启动，MySQL 提供了一个刷新权限的命令。我们可以以三种不同的方式执行此命令：

1.  通过发出`FLUSH PRIVILEGES`。

1.  使用`mysqladmin reload`。

1.  使用`mysqladmin flush-privileges`。

当我们重新加载授予表时，它将按照以下提到的要点工作：

+   **表和列特权**：这些特权的更改将在下一个客户端请求中生效

+   **数据库特权**：这些特权的更改将在客户端执行`USE dbname`语句的下一次生效

+   **全局特权和密码**：这些特权的更改对连接的客户端不受影响；它将适用于随后的连接

# MySQL 8 提供的特权

特权定义了用户帐户可以执行哪些操作。根据操作的级别和应用的上下文，它将起作用。它主要分为以下几类：

+   **数据库特权**：应用于数据库及其内的所有对象。它可以授予单个数据库，也可以全局定义以应用于所有数据库。

+   **管理特权**：它在全局级别定义，因此不限于单个数据库。它使用户能够管理 MySQL 8 服务器的操作。

+   **数据库对象的特权**：用于定义对数据库对象（如表、视图、索引和存储例程）的特权。它可以应用于数据库的特定对象，可以应用于数据库中给定类型的所有对象，也可以全局应用于所有数据库中给定类型的所有对象。

MySQL 8 将帐户特权相关信息存储到授予表中，并在服务器启动时将这些表的内容存储到内存中，以提高性能。特权进一步分为静态和动态特权：

+   **静态特权**：这些特权内置于服务器中，无法注销。这些特权始终可供用户授予。

+   **动态特权**：这些特权可以在运行时注册或注销。如果特权未注册，则不可供用户帐户授予。

# 授予表

授予表包含与用户帐户和授予的特权相关的信息。当我们在数据库中执行任何帐户管理语句时，如`CREATE USER`，`GRANT`和`REVOKE`，MySQL 8 会自动将数据插入这些表中。MySQL 允许管理员用户在授予表上进行插入、更新或删除操作，但这并不是一个理想的方法。MySQL 8 数据库的以下表包含授予信息：

+   `user`：它包含与用户帐户、全局特权和其他非特权列相关的详细信息

+   `password_history`：它包含密码更改的历史记录

+   `columns_priv`：它包含列级特权

+   `procs_priv`：它包含与存储过程和函数相关的特权

+   `proxies_priv`：它包含代理用户的特权

+   `tables_priv`：它包含表级特权

+   `global_grants`：它包含与动态全局特权分配相关的详细信息

+   `role_edges`：它包含角色子图的边缘

+   `db`：它包含数据库级别的特权

+   `default_roles`：它包含与默认用户角色相关的详细信息

授予表包含范围和特权列：

+   **范围列**：此列定义表中行的范围，即行适用的上下文。

+   **特权列**：此列指示用户被允许执行哪些操作。MySQL 服务器从各种授予表中合并信息，以构建用户特权的完整详细信息。

从 MySQL 8.0 开始，授予表使用`InnoDB`存储引擎管理事务状态，但在此之前，MySQL 使用`MyISAM`引擎管理非事务状态。这种改变使用户能够以事务模式管理所有帐户管理语句，因此在多个语句的情况下，要么全部成功执行，要么全部不执行。

# 访问控制阶段的验证

MySQL 8 在两个不同的阶段执行访问控制检查。

# 第 1 阶段 - 连接验证

这是连接验证阶段，因此在验证后，MySQL 8 将接受或拒绝您的连接请求。将根据以下条件执行验证：

1.  基于用户的身份，以及其密码。

1.  用户账户是否被锁定。

如果这两种情况中的任何一种失败，服务器将拒绝访问。在这里，身份包含请求来源的用户名和主机名。MySQL 对用户表的`account_locked`列进行锁定检查，并对用户表范围的三列`Host`、`User`和`authentication_string`进行凭据检查。

# 第 2 阶段 - 请求验证

一旦与 MySQL 服务器建立连接，第 2 阶段就会出现，MySQL 服务器会检查您要执行的操作以及您是否有权限执行。为了进行此验证，MySQL 使用授权表的特权列；它可能来自`user`、`db`、`tables_priv`、`columns_priv`或`procs_priv`表。

# MySQL 8 中的账户管理

顾名思义，本主题描述了如何在 MySQL 8 中管理用户账户。我们将描述如何添加新账户，如何删除账户，如何为账户定义用户名和密码，以及更多。

# 添加和删除用户账户

MySQL 8 提供了创建账户的两种不同方式：

+   **使用账户管理语句**：这些语句用于创建用户并设置其特权；例如，使用`CREATE USER`和`GRANT`语句，通知服务器对授权表进行修改

+   **使用授权表的操作**：使用`INSERT`、`UPDATE`和`DELETE`语句，我们可以操作授权表

在这两种方法中，账户管理语句更可取，因为它们更简洁，更不容易出错。现在，让我们看一个使用命令的例子：

```sql
#1 mysql> CREATE USER 'user1'@'localhost' IDENTIFIED BY 'user1_password';
#2 mysql> GRANT ALL PRIVILEGES ON *.* TO 'user1'@'localhost' WITH GRANT OPTION;

#3 mysql> CREATE USER 'user2'@'%' IDENTIFIED BY 'user2_password';
#4 mysql> GRANT ALL PRIVILEGES ON *.* TO 'user2'@'%' WITH GRANT OPTION;

#5 mysql> CREATE USER 'adminuser'@'localhost' IDENTIFIED BY 'password';
#6 mysql> GRANT RELOAD,PROCESS ON *.* TO 'adminuser'@'localhost';

#7 mysql> CREATE USER 'tempuser'@'localhost';

#8 mysql> CREATE USER 'user4'@'host4.mycompany.com' IDENTIFIED BY 'password';
#9 mysql> GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP ON db1.* TO 'user4'@'host4\. mycompany.com';
```

前面的命令执行以下操作：

+   `#1`命令创建`'user1'`，命令`#2`为`'user1'`分配了完整的特权。但是`'user1'@'localhost'`表示`'user1'`只允许与`localhost`连接。

+   `#3`命令创建`'user2'`，命令`#4`为`'user2'`分配了完整的特权，与`'user1'`相同。但在#4 中，提到了`'user2'@'%'`，这表示`'user2'`可以与任何主机连接。

+   `#5`创建`'adminuser'`并允许它仅与`localhost`连接。在`#6`中，我们可以看到仅为`'adminuser'`提供了`RELOAD`和`PROCESS`特权。它允许`'adminuser'`执行`mysqladmin reload`、`mysqladmin refresh`、`mysqladmin flush-xxx`命令和`mysqladmin processlist`命令，但它无法访问任何数据库。

+   `#7`创建了没有密码的`'tempuser'`账户，并允许用户仅与`localhost`连接。但是没有为`'tempuser'`指定授权，因此此用户无法访问数据库，也无法执行任何管理命令。

+   `#8`创建`'user4'`并允许用户仅使用`'host4'`访问数据库。`#10`表示`'user4'`在`'db1'`上对所有提及的操作具有授权。

要删除用户账户，请执行`DROP USER`命令如下：

```sql
mysql> DROP USER 'user1'@'localhost';
```

此命令将从系统中删除`'user1'`账户。

# 使用角色的安全性

与用户账户角色具有特权一样，我们也可以说角色是特权的集合。作为管理员用户，我们可以向角色授予和撤销特权。MySQL 8 提供了以下与角色配置相关的命令、函数和变量。

# 设置角色

`SET ROLE`在当前会话中更改活动角色。参考以下与`SET ROLE`相关的命令：

```sql
mysql> SET ROLE NONE; SELECT CURRENT_ROLE();
+----------------+
| CURRENT_ROLE() |
+----------------+
| NONE |
+----------------+
mysql> SET ROLE 'developer_read'; SELECT CURRENT_ROLE();
+----------------+
| CURRENT_ROLE() |
+----------------+
| `developer_read`@`%` |
+----------------+
```

第一个命令将在当前会话中取消用户的所有角色。您可以使用`CURRENT_ROLE();`函数查看效果。在第二个命令中，我们将`'developer_read'`角色设置为默认角色，然后再次使用预定义函数检查当前角色。

# 创建角色

`CREATE ROLE`用于创建角色；参考以下命令，它将创建一个名为`'developer_role'`的角色：

```sql
CREATE ROLE 'developer_role';
```

# 删除角色

`DROP ROLE`用于删除角色。参考以下命令，它将删除`'developer_role'`角色：

```sql
DROP ROLE 'developer_role';
```

# 授予权限

`GRANT`分配权限给角色，并将角色分配给帐户。例如，以下命令将所有权限分配给开发人员角色：

```sql
GRANT ALL ON my_db.* TO 'developer_role';
```

同样，要将角色分配给用户帐户，请执行以下命令：

```sql
GRANT 'developer_role' TO 'developer1'@'localhost';
```

此命令将`'developer_role'`角色分配给`developer1`帐户。MySQL 8 还提供了从用户到用户和从角色到角色的`GRANT`分配功能。考虑以下示例：

```sql
CREATE USER 'user1';
CREATE ROLE 'role1';
GRANT SELECT ON mydb.* TO 'user1';
GRANT SELECT ON mydb.* TO 'role1';
CREATE USER 'user2';
CREATE ROLE 'role2';
GRANT 'user1', 'role1'TO 'user2';
GRANT 'user1', 'role1'TO 'role2';
```

在此示例中，通过使用`GRANT`命令以简单的方式在`user1`和`role1`上应用了`GRANT`。现在，对于`user2`和`role2`，我们已经分别从`user1`和`role1`应用了`GRANT`。

# 撤销

`REVOKE`用于从角色中删除权限，并从用户帐户中删除角色分配。参考以下命令：

```sql
REVOKE developer_role FROM user1;
REVOKE INSERT, UPDATE ON app_db.* FROM 'role1';
```

第一个命令用于删除`user1`的`'developer_role'`，第二个命令用于从`'app_db'`上的`'role1'`中删除插入和更新权限。

# 设置默认角色

`SET DEFAULT ROLE`指示默认情况下活动的角色，每当用户登录时，默认角色对用户可用。要设置默认根角色，请执行以下命令：

```sql
mysql>SET DEFAULT ROLE app_developer TO root@localhost;

mysql> SELECT CURRENT_ROLE();
+---------------------+
| CURRENT_ROLE() |
+---------------------+
| `app_developer`@`%` |
+---------------------+
1 row in set (0.04 sec)
```

设置默认角色后，重新启动服务器并执行`current_role()`函数，以检查是否分配了角色。

# 显示授予权限

`SHOW GRANTS`列出与帐户和角色相关的权限和角色分配。对于一个角色，执行以下命令：

```sql
mysql> show grants for app_developer;
+-------------------------------------------+
| Grants for app_developer@% |
+-------------------------------------------+
| GRANT USAGE ON *.* TO `app_developer`@`%` |
+-------------------------------------------+
1 row in set (0.05 sec)
```

此命令显示了`'app_developer'`角色上可用的授予权限。同样，要检查用户的授予权限，请执行以下命令：

```sql
mysql> show grants for root@localhost;
```

前面的命令列出了用户 root 拥有的所有访问权限：

+   `CURRENT_ROLE()`：此函数用于列出当前会话中的当前角色。如默认角色命令中所述，它显示用户当前分配的角色。

+   `activate_all_roles_on_login`：这是一个系统变量，用于在用户登录时自动激活所有授予的角色。默认情况下，角色的自动激活是禁用的。

+   `mandatory_roles`：这是一个系统变量，用于定义强制角色。请记住，定义为强制角色的角色不能使用`drop`命令删除。在服务器文件`my.cnf`中定义您的强制角色如下：

```sql
 [mysqld]
 mandatory_roles='app_developer'
```

要在运行时持久化和设置这些角色，请使用以下语句：

```sql
SET PERSIST mandatory_roles = 'app_developer';
```

此语句应用于运行中的 MySQL 8 实例的更改，并保存以供后续重新启动。如果要应用运行实例的更改而不是其他重新启动的更改，则使用关键字`GLOBAL`而不是`PERSIST`。

# 密码管理

MySQL 8 提供了以下与密码管理相关的功能：

+   **密码过期**：用于定义密码过期的时间段，以便用户可以定期更改密码。MySQL 8 允许为帐户手动设置密码过期，以及设置过期策略。对于过期策略，可以使用`mysql_native_password`、`sha256_password`或`caching_sha2_password`插件。要手动设置密码，请执行以下命令：

```sql
 ALTER USER 'testuser'@'localhost' PASSWORD EXPIRE;
```

这将标记指定用户的密码已过期。对于密码策略，您必须以天数为单位定义持续时间。MySQL 使用系统变量`default_password_lifetime`，其中包含一个正整数来定义天数。我们可以在`my.cnf`文件中定义它，也可以使用`PERSIST`选项在运行时定义它：

+   **密码重用限制**：用于防止再次使用旧密码。MySQL 8 基于两个参数定义此限制-更改次数和经过的时间；它们可以单独或结合使用。MySQL 8 分别定义了`password_history`和`password_reuse_interval`系统变量来应用限制。我们可以在`my.cnf`文件中定义这些变量，也可以使其持久化。

+   `password_history`：此变量表示新密码不能从旧密码设置/复制。在这里，根据指定的次数考虑最近的旧密码。

+   `password_reuse_interval`: 此变量表示密码不能从旧密码设置/复制。在这里，间隔定义了特定的时间段，MySQL 8 将检查用户在该时间段内所有密码与新密码是否匹配。例如，如果间隔设置为 20 天，则新密码在过去 20 天内的更改数据中不应存在。

+   **密码强度评估**：用于定义强密码。它使用`validate_password`插件实现。

# MySQL 8 中的加密

当需要在网络上传输数据时，必须使用加密进行连接。如果使用未加密数据，则可以轻松观察网络访问权限的人员查看客户端和服务器之间传输的所有流量，并查看传输的数据。为了保护您在网络上传输的数据，请使用加密。确保所使用的加密算法包含安全元素，以保护连接免受已知攻击，如更改消息顺序或在数据上重复两次。根据应用程序要求，可以选择加密或未加密类型的连接。MySQL 8 使用**传输层安全性**（**TLS**）协议对每个连接执行加密。

# 配置 MySQL 8 以使用加密连接

本节描述了如何配置服务器和客户端以进行加密连接。

# 服务器端配置加密连接

在服务器端，MySQL 8 使用`-ssl`选项来指定与加密相关的属性。以下选项用于在服务器端配置加密：

+   `--ssl-ca`：此选项指定**证书颁发机构**（**CA**）证书文件的路径名

+   `--ssl-cert`：此选项指定服务器公钥证书文件的路径名

+   `--ssl-key`：此选项指定服务器私钥文件的路径名

您可以通过在`my.cnf`文件中指定上述选项来使用这些选项：

```sql
[mysqld]
ssl-ca=ca.pem
ssl-cert=server-cert.pem
ssl-key=server-key.pem
```

`--ssl`选项默认启用，因此在服务器启动时，MySQL 8 将尝试在数据目录下查找证书和密钥文件，即使您没有在`my.cnf`文件中定义它。如果找到这些文件，MySQL 8 将提供加密连接，否则将继续不加密连接。

# 客户端端配置加密连接

在客户端，MySQL 使用与服务器端相同的`-ssl`选项来指定证书和密钥文件，但除此之外，还有`-ssl-mode`选项。默认情况下，如果服务器允许，客户端可以与服务器建立加密连接。为了进一步控制，客户端程序使用以下`-ssl-mode`选项：

+   `--ssl-mode=REQUIRED`：此选项表示必须建立加密连接，如果未建立则失败

+   `--ssl-mode=PREFFERED`：此选项表示客户端程序可以建立加密连接，如果服务器允许，否则建立未加密连接而不会失败

+   `--ssl-mode=DISABLED`：此选项表示客户端程序无法使用加密连接，只允许未加密连接

+   `--ssl-mode=VERIFY_CA`：此选项与`REQUIRED`相同，但除此之外，它还会验证 CA 证书与配置的 CA 证书匹配，并在找不到匹配项时返回失败

+   `--ssl-mode=VERIFY_IDENTITY`：与`VERIFY_CA`选项相同，但除此之外，它还将执行主机名身份验证

# 加密连接的命令选项

MySQL 8 提供了用于加密连接的几个选项。您可以在命令行上使用这些选项，也可以在选项文件中定义它们：

| **格式** | **描述** |
| --- | --- |
| `--skip-ssl` | 不使用加密连接 |
| `--ssl` | 启用加密连接 |
| `--ssl-ca` | 包含受信任的 SSL 证书颁发机构列表的文件 |
| `--ssl-capath` | 包含受信任的 SSL 证书颁发机构证书文件的目录 |
| `--ssl-cert` | 包含 X509 证书的文件 |
| `--ssl-cipher` | 连接加密的允许密码列表 |
| `--ssl-crl` | 包含证书吊销列表的文件 |
| `--ssl-crlpath` | 包含证书吊销列表文件的目录 |
| `--ssl-key` | 包含 X509 密钥的文件 |
| `--ssl-mode` | 与服务器连接的安全状态 |
| `--tls-version` | 允许加密连接的协议 |

参考：[`dev.mysql.com/doc/refman/8.0/en/encrypted-connection-options.html`](https://dev.mysql.com/doc/refman/8.0/en/encrypted-connection-options.html)

# 从 Windows 远程连接到 MySQL 8 并使用 SSH

要从 Microsoft Windows 系统远程连接到 MYSQL 8 并使用 SSH，执行以下步骤：

1.  在本地系统上安装 SSH 客户端。

1.  启动 SSH 客户端后，通过要连接到服务器的主机名和用户 ID 进行设置。

1.  配置端口转发如下并保存信息：

+   **对于远程转发配置**：`local_port:3306`，`remote_host:mysqlservername_or_ip`，`remote_port:3306`

+   **对于本地转发配置**：`local_port:3306`，`remote_host:localhost`，`remote_port:3306`

1.  使用创建的 SSH 会话登录服务器。

1.  在本地的 Microsoft Windows 机器上，启动任何 ODBC 应用程序，如 Microsoft Access。

1.  在本地系统中，创建新文件并尝试使用 ODBC 驱动程序链接到 MySQL 服务器。确保在连接中定义了`localhost`而不是`mysqlservername`。

# 安全插件

MySQL 8 提供了几个插件来实现安全性。这些插件提供了与身份验证协议、密码验证、安全存储等相关的各种功能。让我们详细讨论各种类型的插件。

# 认证插件

以下是认证插件的列表及其详细信息：

+   **本机可插拔身份验证**：为了实现本机身份验证，MySQL 8 使用`mysql_native_password`插件。此插件在服务器和客户端两侧都使用一个通用名称，并由 MySQL 8 为服务器和客户端程序提供内置支持。

+   SHA-256 可插拔身份验证

为了实现 SHA-256 哈希，MySQL 8 提供了两种不同的插件：

1.  `sha256_password`：此插件用于实现基本的 SHA-256 身份验证。

1.  `caching_sha2_password`：此插件实现了 SHA-256 身份验证，并具有缓存功能以提高性能，与基本插件相比具有一些附加功能。

此插件与 MySQL 8 服务器和客户端程序内置提供，名称相同为`sha256_password`。在客户端中，它位于`libmysqlclient`库下。要为帐户使用此插件，请执行以下命令：

```sql
CREATE USER 'testsha256user'@'localhost'
IDENTIFIED WITH sha256_password BY 'userpassword';
```

# SHA-2 可插拔身份验证

SHA-2 可插拔身份验证与 SHA-256 可插拔插件相同，只是其插件名称为`caching_sha2_password`**。**与`sha256_password`相比，此插件具有以下优点：

1.  如果使用 Unix 套接字文件和共享内存协议，则为客户端连接提供支持。

1.  SHA-2 插件中提供了内存缓存，为以前连接过的用户提供更快的重新认证。

1.  该插件提供了基于 RSA 的密码交换，可以在 MySQL 8 提供的 SSL 库无关的情况下工作。

# 客户端明文可插拔认证

该插件用于将密码发送到服务器而不进行哈希或加密。在客户端库中以`mysql_clear_password`的名称提供。MySQL 8 在客户端库中内置了它。

# 无登录可插拔认证

这是一个服务器端插件，用于阻止使用它的任何帐户的所有客户端连接。插件名称是`'mysql_no_login'`，它不是 MySQL 的内置插件，因此我们必须使用`mysql_no_login.so`库。要使其可用，首先将库文件放在插件目录下，然后执行以下步骤之一：

1.  通过在`my.cnf`文件中添加`--plugin-load-add`参数在服务器启动时加载插件：

```sql
 [mysqld]
 plugin-load-add=mysql_no_login.so
```

1.  要在运行时注册插件，请执行以下命令：

```sql
 INSTALL PLUGIN mysql_no_login SONAME 'mysql_no_login.so';
```

要卸载此插件，请执行以下命令：

1.  如果使用`--plugin-load-adoption`在服务器启动时安装了插件，则通过删除该选项重启服务器来卸载插件。

1.  如果使用`INSTALL PLUGIN`命令安装了插件，则使用卸载命令将其移除：

```sql
UNINSTALL PLUGIN mysql_no_login;
```

# 套接字对等凭证可插拔认证

名为`auth_socket`的服务器端插件用于对从本地主机使用 Unix 套接字文件连接的客户端进行身份验证。它仅用于支持`SO_PEERCRED`选项的系统。`SO_PEERCRED`用于获取有关运行客户端程序的用户的信息。这不是一个内置插件；我们必须使用`auth_socket.so`库来使用这个插件。要使其可用，首先将库文件放在插件目录下，然后执行以下步骤之一：

1.  通过在`my.cnf`文件中添加`--plugin-load-add`参数在服务器启动时加载插件：

```sql
 [mysqld]
 plugin-load-add=auth_socket.so
```

1.  通过执行以下命令在运行时注册插件：

```sql
 INSTALL PLUGIN auth_socket SONAME 'auth_socket.so';
```

要卸载此插件，请执行以下命令：

1.  如果使用`--plugin-load-addoption`在服务器启动时安装了插件，则通过删除该选项重启服务器来卸载插件。

1.  如果使用`INSTALL PLUGIN`命令安装了插件，则使用`UNINSTALL`命令将其移除：

```sql
 UNINSTALL PLUGIN auth_socket;
```

# 测试可插拔认证

MySQL 8 提供了一个测试插件，用于检查帐户凭据并在服务器日志中记录成功或失败。这不是一个内置插件，需要在使用之前安装。它适用于服务器端和客户端，分别命名为`test_plugin_server`和`auth_test_plugin`。MySQL 8 使用`auth_test_plugin.so`库来提供此插件。要安装和卸载此插件，请执行与前面插件中提到的相同的步骤。

# 连接控制插件

MySQL 8 使用这些插件在特定数量的连接尝试失败后向客户端的服务器响应中引入逐渐增加的延迟。MySQL 为连接控制提供了两个插件。

# CONNECTION_CONTROL

这个插件将检查所有传入连接的请求，并根据需要在服务器响应中添加延迟。该插件使用一些系统变量进行配置，并使用状态变量进行监视。它还使用其他一些插件、事件类和进程，比如审计插件、`MYSQL_AUDIT_CONNECTION_CLASSMASK`事件类、`MYSQL_AUDIT_CONNECTION_CONNECT`和`MYSQL_AUDIT_CONNECTION_CHANGE_USER`进程，以检查服务器是否应该在处理任何客户端连接之前添加延迟：

```sql
CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS
```

该插件实现了对`INFORMATION_SCHEMA`表的使用，以提供有关失败连接监视的详细信息。

# 插件安装

我们必须使用`connection_control.so`库来使用这个插件。要使其可用，首先将库文件放在插件目录下，然后执行以下步骤之一：

1.  在`my.cnf`文件中添加`--plugin-load-add`参数，以在服务器启动时加载插件：

```sql
 [mysqld]
 plugin-load-add= connection_control.so
```

1.  在`my.cnf`文件中添加`--plugin-load-add`参数，以在服务器启动时加载插件：

```sql
 INSTALL PLUGIN CONNECTION_CONTROL SONAME 
          'connection_control.so';
 INSTALL PLUGIN CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS SONAME 
          'connection_control.so';
```

# 与连接控制相关的变量

以下变量由`CONNECTION-CONTROL`插件提供：

+   `Connection_control_delay_generated`: 这是一个状态变量，主要用于管理计数器。它指示服务器在连接失败尝试时添加延迟的次数。它还取决于`connection_control_failed_connections_threshold`系统变量，因为除非尝试次数达到阈值变量定义的限制，否则此状态变量不会增加计数。

+   `connection_control_failed_connections_threshold`: 这是一个系统变量，指示在服务器对每次尝试添加延迟之前，客户端允许连续失败的尝试次数。

+   `connection_control_max_connection_delay`: 这是一个系统变量，定义了服务器在连接失败尝试时的最大延迟时间（以毫秒为单位）。一旦阈值变量包含一个大于零的值，MySQL 8 将考虑这个变量。

+   `connection_control_min_connection_delay`: 该系统变量定义了服务器对连接失败尝试的最小延迟时间（以毫秒为单位）。一旦阈值变量包含一个大于零的值，MySQL 8 将考虑这个变量。

# 密码验证插件

对于密码验证，MySQL 提供了一个名为`validate_password`的插件。它主要用于测试密码并提高安全性。以下是该插件的两个主要功能：

+   `VALIDATE_PASSWORD_STRENGTH()`: 一个 SQL 函数，用于查找密码的强度。它以密码作为参数，并返回一个介于 0 和 100 之间的整数值。这里，0 表示弱密码，100 表示强密码。

+   **按照 SQL 语句中的策略检查密码**：对于所有使用明文密码的 SQL 语句，插件将检查提供的密码是否符合密码策略，并根据此返回响应。对于弱密码，插件将返回一个`ER_NOT_VALID_PASSWORD`错误。如果密码在参数中以明文形式定义，`ALTER USER`、`CREATE USER`、`GRANT`、`SET PASSWORD`语句和`PASSWORD()`函数始终由该插件检查。

# 安装密码验证插件

我们必须使用`validate_password.so`库与该插件。要使其可用，首先将库文件放在插件目录下，然后执行以下步骤之一：

1.  在服务器启动时加载插件，通过在`my.cnf`文件中添加`--plugin-load-add`参数：

```sql
 [mysqld]
 plugin-load-add=validate_password.so
```

1.  在运行时注册插件，执行以下命令：

```sql
 INSTALL PLUGIN validate_password SONAME 'validate_password.so';
```

# 与密码验证插件相关的变量和选项

MySQL 8 提供了以下与密码验证插件相关的系统变量、状态变量和选项。

+   `validate_password_check_user_name`: 这是一个系统变量，在 MySQL 8 中默认启用。顾名思义，它用于将密码与当前有效用户的用户名进行比较。如果密码与用户名或其反转匹配，MySQL 8 将拒绝密码，而不管`VALIDATE_PASSWORD_STRENGTH()`函数的值如何。

+   `validate_password_dictionary_file`: 该系统变量包含了`validate_password`插件使用的目录的路径名。您可以在运行时设置它，无需重新启动服务器，并且一旦安装了插件，它就可用。如果您定义了用于密码检查的目录，将密码策略值设置为 2（强）。密码策略的可能值在`validate_password_policy`系统变量下描述。

+   `validate_password_length`: 一旦安装了插件，该系统变量可用于定义密码与`validate_password`插件进行检查所需的最小字符数。

+   `validate_password_mixed_case_count`：一旦安装了插件，此系统变量可用于定义密码检查中小写和大写字符的最小数量。

+   `validate_password_number_count`：一旦安装了插件，此系统变量可用于定义密码检查中数字的最小数量。

+   `validate_password_special_char_count`：一旦安装了插件，此系统变量可用于定义密码检查中非字母数字字符的最小数量。

+   `validate_password_policy`：一旦安装了插件，此系统变量可用，并指示插件在其他系统变量情况下应如何行为。此变量的以下值描述了`validate_password`插件的行为：

| **策略** | **执行的测试** |
| --- | --- |
| 0 或 LOW | 长度 |
| 1 或 MEDIUM | 长度；数字，小写/大写和特殊字符 |
| 2 或 STRONG | 长度；数字，小写/大写和特殊字符；字典文件 |

参考：[`dev.mysql.com/doc/refman/8.0/en/validate-password-options-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/validate-password-options-variables.html)

+   `validate_password_dictionary_file_last_parsed`：这是一个状态变量，用于指示上次解析目录文件的时间。

+   `validate_password_dictionary_file_words_count`：这是一个状态变量，用于指示从目录文件中读取的单词数量。

+   `--validate-password[=value]`：此选项用于定义服务器在启动时如何加载`validate_password`插件。此选项仅在插件使用`INSTALL PLUGIN`注册或使用`--plugin-load-add`功能加载时可用。

# MySQL 8 keyring

MySQL 8 提供了一个 keyring 服务，允许 MySQL 服务器的内部组件和插件存储它们的敏感信息以供以后使用。对于此功能，MySQL 8 使用`keyring_file`插件，该插件将数据存储在服务器主机上的文件中。此插件在所有 MySQL 的发行版中都可用，如社区版和企业版。

# 安装 keyring 插件

我们必须使用`keyring_file.so`库与此插件。为了使其可用，首先将库文件放在插件目录下，然后执行以下步骤之一：

+   通过在`my.cnf`文件中添加`--plugin-load-add`参数，在服务器启动时加载插件：

```sql
 mysqld]
 plugin-load-add=keyring_file.so
```

+   通过执行以下命令在运行时注册插件：

```sql
 INSTALL PLUGIN keyring_file SONAME 'keyring_file.so';
```

# 与 keyring 插件相关的系统变量

MySQL 8 提供了以下与 keyring 插件相关的系统变量：

+   `keyring_file_data`：一旦安装了插件，此系统变量可用于定义`keyring_file`插件用于存储安全数据的数据文件的路径。Keyring 操作是事务性的，因此此插件在写操作期间使用备份文件来处理回滚情况。在这种情况下，备份文件的命名约定与`keyring_file_data`系统变量中定义的相同，并带有`.backup`后缀。

# 总结

在本章中，我们首先概述了安全性，然后介绍了 MySQL 8 安全相关的功能。首先我们讨论了一些常见的安全问题，然后展示了如何分配权限以及如何在 MySQL 8 中管理访问控制。本章还涵盖了加密，以保护您的敏感数据。最后，我们介绍了一些重要的安全插件，这些插件对于在 MySQL 8 中实现安全性非常有用。

现在是时候转到我们的下一章了，在那里我们将为优化配置 MySQL 8。对于优化，我们将涵盖数据库的不同领域，如优化查询，优化表，优化缓冲和缓存等等。除了服务器配置，它还涵盖了如何为优化配置客户端。
