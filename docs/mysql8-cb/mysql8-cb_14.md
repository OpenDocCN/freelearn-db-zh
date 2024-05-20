# 第十四章：安全

在本章中，我们将介绍以下配方：

+   安装安全

+   限制网络和用户

+   使用 mysql_config_editor 进行无密码身份验证

+   重置 root 密码

+   使用 X509 建立加密连接

+   设置 SSL 复制

# 介绍

本章涵盖了 MySQL 的安全方面，包括限制网络、强密码、使用 SSL、数据库内的访问控制、安装安全和安全插件。

# 安装安全

安装完成后，建议您使用`mysql_secure_installation`实用程序保护您的安装。

# 如何做...

```go
shell> mysql_secure_installation

Securing the MySQL server deployment.

Enter password for user root: 
The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration
of the plugin.
Using existing password for root.

Estimated strength of the password: 100 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : 

 ... skipping.
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.

Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```

默认情况下，`mysqld`进程在`mysql`用户下运行。您还可以通过更改`mysqld`使用的所有目录的所有权（例如`datadir`，如果有的话，`binlog`目录，其他磁盘上的表空间等）并在`my.cnf`中添加`user=<user>`来以另一个用户身份运行`mysqld`。有关更改 MySQL 用户的更多信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/changing-mysql-user.html`](https://dev.mysql.com/doc/refman/8.0/en/changing-mysql-user.html)。

强烈建议不要将`mysqld`作为 Unix 根用户运行。一个原因是任何拥有`FILE`权限的用户都可以使服务器以 root 身份创建文件。

# FILE 权限

在授予任何用户`FILE`权限时要小心，因为用户可以使用`mysqld`守护程序的权限在文件系统的任何位置写入文件，其中包括服务器的`数据目录`。但是，他们不能覆盖现有文件。此外，用户可以将 MySQL（或运行`mysqld`的用户）可以访问的任何文件读取到数据库表中。`FILE`是全局权限，这意味着您无法将其限制为特定数据库：

```go
mysql> SHOW GRANTS;
+--------------------------------------------------------------------+
| Grants for company_admin@%                                         |
+--------------------------------------------------------------------+
| GRANT FILE ON *.* TO `company_admin`@`%`                           |
| GRANT SELECT, INSERT, CREATE ON `company`.* TO `company_admin`@`%` |
+--------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> USE company;
Database changed
mysql> CREATE TABLE hack (ibdfile longblob);
Query OK, 0 rows affected (0.05 sec)

mysql> LOAD DATA INFILE '/var/lib/mysql/employees/salaries.ibd' INTO TABLE hack CHARACTER SET latin1 FIELDS TERMINATED BY '@@@@@';
Query OK, 366830 rows affected (18.98 sec)
Records: 366830  Deleted: 0  Skipped: 0  Warnings: 0

mysql> SELECT * FROM hack;
```

注意，拥有`FILE`权限的公司用户可以从`employees`表中读取数据。

您不需要担心前面的黑客攻击，因为默认情况下可以读取和写入文件的位置仅限于`/var/lib/mysql-files`，使用`secure_file_priv`变量。问题出在当您将`secure_file_priv`变量设置为`NULL`，空字符串，MySQL`数据目录`或 MySQL 可以访问的任何敏感目录（例如 MySQL`数据目录`之外的表空间）时。如果将`secure_file_priv`设置为不存在的目录，将导致错误。

建议将`secure_file_priv`保留为默认值：

```go
mysql> SHOW VARIABLES LIKE 'secure_file_priv';
+------------------+-----------------------+
| Variable_name    | Value                 |
+------------------+-----------------------+
| secure_file_priv | /var/lib/mysql-files/ |
+------------------+-----------------------+
1 row in set (0.00 sec)
```

永远不要让任何人访问`mysql.user`表。有关安全准则的更多信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/security-guidelines.html`](https://dev.mysql.com/doc/refman/8.0/en/security-guidelines.html)和[`dev.mysql.com/doc/refman/8.0/en/security-against-attack.html`](https://dev.mysql.com/doc/refman/8.0/en/security-against-attack.html)。

# 限制网络和用户

不要将数据库开放给整个网络，这意味着 MySQL 运行的端口（`3306`）不应该从其他网络访问。它应该只对应用服务器开放。您可以使用 iptables 或`host.access`文件设置防火墙以限制对端口`3306`的访问。如果您在云上使用 MySQL，提供商还将提供防火墙。

# 如何做...

要测试这一点，您可以使用`telnet`：

```go
shell> telnet <mysql ip> 3306
# if telnet is not installed you can install it or use nc (netcat)
```

如果 telnet 挂起或连接被拒绝，这意味着端口已关闭。请注意，如果看到这样的输出，这意味着端口没有被阻止：

```go
shell> telnet 35.186.158.188 3306
Trying 35.186.158.188...
Connected to 188.158.186.35.bc.googleusercontent.com.
Escape character is '^]'.
FHost '183.82.17.137' is not allowed to connect to this MySQL serverConnection closed by foreign host.
```

这意味着端口是打开的，但 MySQL 正在限制访问。

在创建用户时，避免从任何地方（`%`选项）授予访问权限。限制访问 IP 范围或子域。还限制用户只能访问所需的数据库。例如，`employees`数据库的`read_only`用户不应能够访问其他数据库：

```go
mysql> CREATE user 'employee_read_only'@'10.10.%.%' IDENTIFIED BY '<Str0ng_P@$$word>';
Query OK, 0 rows affected (0.00 sec)

mysql> GRANT SELECT ON employee.* TO 'employee_read_only'@'10.10.%.%';
Query OK, 0 rows affected (0.01 sec)
```

`employee_read_only`用户将只能从`10.10.%.%`子网访问，并且只能访问`employee`数据库。

# 使用 mysql_config_editor 进行无密码身份验证

每当您使用命令行客户端输入密码时，您可能会注意到以下警告：

```go
shell> mysql -u dbadmin -p'$troNgP@$$w0rd'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1345
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
~
mysql>
```

如果不在命令行中传递密码并在提示时输入，您将不会收到警告：

```go
shell> mysql -u dbadmin -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1334
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
~
mysql>
```

但是，当您在客户端实用程序上开发一些脚本时，使用密码提示很困难。避免这种情况的一种方法是将密码存储在`home`目录中的`.my.cnf`文件中。默认情况下，`mysql`命令行实用程序会读取`.my.cnf`文件，而不会要求输入密码：

```go
shell> cat $HOME/.my.cnf
[client]
user=dbadmin
password=$troNgP@$$w0rd

shell> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1396
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
~
mysql>
```

请注意，您可以在不提供任何密码的情况下连接，但这会引起安全问题；密码是明文的。为了克服这一点，MySQL 引入了`mysql_config_editor`，它以加密格式存储密码。客户端程序可以解密文件（仅在内存中使用）以连接到服务器。

# 如何做...

使用`mysql_config_editor`创建`.mylogin.cnf`文件：

```go
shell> mysql_config_editor set --login-path=dbadmin_local --host=localhost --user=dbadmin --password
Enter password:
```

通过更改登录路径，可以添加多个主机名和密码。如果更改了密码，可以再次运行此实用程序，以更新文件中的密码：

```go
shell> mysql_config_editor set --login-path=dbadmin_remote --host=35.186.157.16 --user=dbadmin --password
Enter password: 
```

如果要使用`dbadmin`用户登录`35.186.157.16`，只需执行`mysql --login-path=dbadmin_remote`：

```go
shell> mysql --login-path=dbadmin_remote 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 215074
~
mysql> SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|         200 |
+-------------+
1 row in set (0.00 sec)
```

要连接到`localhost`，只需执行`mysql`或`mysql --login-path=dbadmin_local`：

```go
shell> mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1523
~
mysql> SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|           1 |
+-------------+
1 row in set (0.00 sec)

shell> mysql --login-path=dbadmin_local
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1524
~
mysql> SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|           1 |
+-------------+
1 row in set (0.00 sec)
```

如果`dbadmin`的密码在所有服务器上都相同，可以通过指定主机名连接到任何服务器。您无需指定密码：

```go
shell> mysql -h 35.198.210.229
Welcome to the MySQL monitor.  Commands end with ; or \g.
~
mysql> SELECT @@server_id;
+-------------+
| @@server_id |
+-------------+
|         364 |
+-------------+
1 row in set (0.00 sec)
```

如果要打印所有登录路径，请执行以下操作：

```go
shell> mysql_config_editor print --all
[dbadmin_local]
user = dbadmin
password = *****
host = localhost
[dbadmin_remote]
user = dbadmin
password = *****
host = 35.186.157.16
```

您可以注意到该实用程序掩盖了密码。如果尝试读取文件，您只会看到无意义的字符：

```go
shell> cat .mylogin.cnf 
 ?-z???|???-B????dU?bz4-?W???g?q?BmV?????K?I?? h%?+b???_??@V???vli?J???X`?qP
```

此实用程序仅帮助您避免存储明文密码并简化连接到 MySQL 的过程。有许多方法可以解密存储在`.mylogin.cnf`文件中的密码。因此，如果使用`mysql_config_editor`，请不要认为密码是安全的。您可以将此文件复制到其他服务器（仅当用户名和密码相同时才有效），而不是每次创建`.mylogin.cnf`文件。

# 重置 root 密码

如果忘记了 root 密码，可以通过以下两种方法重置密码。

# 如何做...

让我们深入了解细节。

# 使用 init-file

在类 Unix 系统上，您可以通过指定 init-file 停止服务器并启动服务器。您可以将`ALTER USER 'root'@'localhost' IDENTIFIED BY 'New$trongPass1'` SQL 代码保存在该文件中。MySQL 在启动时执行文件的内容，从而更改 root 用户的密码：

1.  停止服务器：

```go
shell> sudo systemctl stop mysqld
shell> pgrep mysqld
```

1.  将 SQL 代码保存在`/var/lib/mysql/mysql-init-password`中；使其仅对 MySQL 可读：

```go
shell> vi /var/lib/mysql/mysql-init-password
ALTER USER 'root'@'localhost' IDENTIFIED BY 'New$trongPass1';

shell> sudo chmod 400 /var/lib/mysql/mysql-init-password

shell> sudo chown mysql:mysql /var/lib/mysql/mysql-init-password
```

1.  使用`--init-file`选项和其他所需选项启动 MySQL 服务器：

```go
shell> sudo -u mysql /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid --user=mysql --init-file=/var/lib/mysql/mysql-init-password
mysqld will log errors to /var/log/mysqld.log
mysqld is running as pid 28244
```

1.  验证错误日志文件：

```go
shell> sudo tail /var/log/mysqld.log 
~
2017-11-27T07:32:25.219483Z 0 [Note] Execution of init_file '/var/lib/mysql/mysql-init-password' started.
2017-11-27T07:32:25.219639Z 4 [Note] Event Scheduler: scheduler thread started with id 4
2017-11-27T07:32:25.223528Z 0 [Note] Execution of init_file '/var/lib/mysql/mysql-init-password' ended.
2017-11-27T07:32:25.223610Z 0 [Note] /usr/sbin/mysqld: ready for connections. Version: '8.0.3-rc-log'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server (GPL)
```

1.  验证是否能够使用新密码登录：

```go
shell> mysql -u root -p'New$trongPass1'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
~
mysql>
```

1.  现在，最重要的事情！删除`/var/lib/mysql/mysql-init-password`文件：

```go
shell> sudo rm -rf /var/lib/mysql/mysql-init-password
```

1.  可选地，您可以停止服务器，然后正常启动，而无需`--init-file`选项。

# 使用--skip-grant-tables

在此方法中，您可以通过指定`--skip-grant-tables`停止服务器，然后启动服务器，这将不会加载授予表。您可以作为 root 连接到服务器而无需密码，并设置密码。由于服务器在没有授予的情况下运行，因此可能会有其他网络的用户连接到服务器。因此，从 MySQL 8.0.3 开始，`--skip-grant-tables`会自动启用`--skip-networking`，这将不允许远程连接：

1.  停止服务器：

```go
shell> sudo systemctl stop mysqld
shell> ps aux | grep mysqld | grep -v grep
```

1.  使用`--skip-grant-tables`选项启动服务器：

```go
shell> sudo -u mysql /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid --user=mysql --skip-grant-tables
mysqld will log errors to /var/log/mysqld.log
mysqld is running as pid 28757
```

1.  连接到 MySQL 而不需要密码，执行`FLUSH PRIVILEGES`重新加载授权，并更改用户以更改密码：

```go
shell> mysql -u root
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
~
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.04 sec)

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'New$trongPass1';
Query OK, 0 rows affected (0.01 sec)
```

1.  使用新密码测试与 MySQL 的连接：

```go
shell> mysql -u root -p'New$trongPass1'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
~
mysql>
```

1.  重新启动 MySQL 服务器：

```go
shell> ps aux | grep mysqld | grep -v grep
mysql    28757  0.0 13.3 1151796 231724 ?      Sl   08:16   0:00 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid --user=mysql --skip-grant-tables
shell> sudo kill -9 28757
shell> ps aux | grep mysqld | grep -v grep
shell> sudo systemctl start mysqld
shell> ps aux | grep mysqld | grep -v grep
mysql    29033  5.3 16.8 1240224 292744 ?      Sl   08:27   0:00 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
```

# 使用 X509 设置加密连接

如果客户端和 MySQL 服务器之间的连接未加密，任何可以访问网络的人都可以检查数据。如果客户端和服务器位于不同的数据中心，建议使用加密连接。默认情况下，MySQL 8 使用加密连接，但如果加密连接失败，它将退回到未加密连接。您可以通过检查`Ssl_cipher`变量的状态来测试。如果连接是由`localhost`建立的，则不会使用密码：

```go
mysql> SHOW STATUS LIKE 'Ssl_cipher';
+---------------+--------------------+
| Variable_name | Value              |
+---------------+--------------------+
| Ssl_cipher    | DHE-RSA-AES256-SHA |
+---------------+--------------------+
1 row in set (0.00 sec)
```

如果不使用 SSL，`Ssl_cipher`将为空白。

您可以要求某些用户只能通过加密连接连接（通过指定`REQUIRE SSL`子句），并对其他用户将其作为可选项。

根据 MySQL 文档：

MySQL 支持使用 TLS（传输层安全）协议在客户端和服务器之间建立加密连接。TLS 有时被称为 SSL（安全套接字层），但 MySQL 实际上并不使用 SSL 协议进行加密连接，因为其加密较弱。TLS 使用加密算法来确保可以信任通过公共网络接收的数据。它具有检测数据更改、丢失或重放的机制。TLS 还包含使用 X509 标准进行身份验证的算法。

在本节中，您将学习如何使用 X509 建立 SSL 连接。

所有与 SSL（X509）相关的文件（`ca.pem`、`server-cert.pem`、`server-key.pem`、`client-cert.pem`和`client-key.pem`）都是 MySQL 在安装过程中创建并保存在`数据目录`下的。服务器需要`ca.pem`、`server-cert.pem`和`server-key.pem`文件，客户端使用`client-cert.pem`和`client-key.pem`文件连接到服务器。

# 如何操作...

1.  验证`数据目录`中的文件，更新`my.cnf`，重新启动服务器，并检查与 SSL 相关的变量。在 MySQL 8 中，默认情况下设置了以下值：

```go
shell> sudo ls -lhtr /var/lib/mysql | grep pem
-rw-------. 1 mysql mysql 1.7K Nov 19 13:53 ca-key.pem
-rw-r--r--. 1 mysql mysql 1.1K Nov 19 13:53 ca.pem
-rw-------. 1 mysql mysql 1.7K Nov 19 13:53 server-key.pem
-rw-r--r--. 1 mysql mysql 1.1K Nov 19 13:53 server-cert.pem
-rw-------. 1 mysql mysql 1.7K Nov 19 13:53 client-key.pem
-rw-r--r--. 1 mysql mysql 1.1K Nov 19 13:53 client-cert.pem
-rw-------. 1 mysql mysql 1.7K Nov 19 13:53 private_key.pem
-rw-r--r--. 1 mysql mysql  451 Nov 19 13:53 public_key.pem
```

```go
shell> sudo vi /etc/my.cnf
[mysqld]
ssl-ca=/var/lib/mysql/ca.pem
ssl-cert=/var/lib/mysql/server-cert.pem
ssl-key=/var/lib/mysql/server-key.pem
```

```go
shell> sudo systemctl restart mysqld
```

```go
mysql> SHOW VARIABLES LIKE '%ssl%';
+---------------+--------------------------------+
| Variable_name | Value                          |
+---------------+--------------------------------+
| have_openssl  | YES                            |
| have_ssl      | YES                            |
| ssl_ca        | /var/lib/mysql/ca.pem          |
| ssl_capath    |                                |
| ssl_cert      | /var/lib/mysql/server-cert.pem |
| ssl_cipher    |                                |
| ssl_crl       |                                |
| ssl_crlpath   |                                |
| ssl_key       | /var/lib/mysql/server-key.pem  |
+---------------+--------------------------------+
9 rows in set (0.01 sec)
```

1.  将`client-cert.pem`和`client-key.pem`文件从服务器的`数据目录`复制到客户端位置：

```go
shell> sudo scp -i $HOME/.ssh/id_rsa /var/lib/mysql/client-key.pem /var/lib/mysql/client-cert.pem <user>@<client_ip>:
# change the ssh private key path as needed.
```

1.  通过传递`--ssl-cert`和`--ssl-key`选项连接到服务器：

```go
shell> mysql --ssl-cert=client-cert.pem --ssl-key=client-key.pem -h 35.186.158.188
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 666
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
~
mysql>
```

1.  强制用户只能通过 X509 连接：

```go
mysql> ALTER USER `dbadmin`@`%` REQUIRE X509;
Query OK, 0 rows affected (0.08 sec)
```

1.  测试连接：

```go
shell> mysql --login-path=dbadmin_remote -h 35.186.158.188 --ssl-cert=client-cert.pem --ssl-key=client-key.pem
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 795
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
~
mysql> ^DBye
```

1.  如果不指定`--ssl-cert`或`--ssl-key`，则将无法登录：

```go
shell> mysql --login-path=dbadmin_remote -h 35.186.158.188 
ERROR 1045 (28000): Access denied for user 'dbadmin'@'35.186.157.16' (using password: YES)

shell> mysql --login-path=dbadmin_remote -h 35.186.158.188 --ssl-cert=client-cert.pem 
mysql: [ERROR] SSL error: Unable to get private key from 'client-cert.pem'
ERROR 2026 (HY000): SSL connection error: Unable to get private key

shell> mysql --login-path=dbadmin_remote -h 35.186.158.188 --ssl-key=client-key.pem
mysql: [ERROR] SSL error: Unable to get certificate from 'client-key.pem'
ERROR 2026 (HY000): SSL connection error: Unable to get certificate
```

默认情况下，所有与 SSL 相关的文件都保存在`数据目录`中。如果要将它们保存在其他位置，可以在`my.cnf`文件中设置`ssl_ca`、`ssl_cert`和`ssl_key`，然后重新启动服务器。您可以通过 MySQL 或 OpenSSL 生成一组新的 SSL 文件。要了解更详细的步骤，请参考[`dev.mysql.com/doc/refman/8.0/en/creating-ssl-rsa-files-using-mysql.html`](https://dev.mysql.com/doc/refman/8.0/en/creating-ssl-rsa-files-using-mysql.html)。还有许多其他身份验证插件可用。您可以参考[`dev.mysql.com/doc/refman/8.0/en/authentication-plugins.html`](https://dev.mysql.com/doc/refman/8.0/en/authentication-plugins.html)了解更多详细信息。

# 设置 SSL 复制

如果启用 SSL 复制，主从之间的二进制日志传输将通过加密连接发送。这类似于前一节中解释的服务器/客户端连接。

# 如何操作...

1.  **在主库上**，如前一节所述，您需要启用 SSL。

1.  **在主库上**，将`client*`证书复制到从库上：

```go
mysql> sudo scp -i $HOME/.ssh/id_rsa /var/lib/mysql/client-key.pem /var/lib/mysql/client-cert.pem <user>@<client_ip>:
```

1.  **在从库上**，创建`mysql-ssl`目录以保存与 SSL 相关的文件，并正确设置权限：

```go
shell> sudo mkdir /etc/mysql-ssl
shell> sudo cp client-key.pem client-cert.pem /etc/mysql-ssl/
shell> sudo chown -R mysql:mysql /etc/mysql-ssl
shell> sudo chmod 600 /etc/mysql-ssl/client-key.pem
shell> sudo chmod 644 /etc/mysql-ssl/client-cert.pem
```

1.  **在从库上**，使用与从库相关的 SSL 更改执行`CHANGE_MASTER`命令：

```go
mysql> STOP SLAVE;

mysql> CHANGE MASTER TO MASTER_SSL=1, MASTER_SSL_CERT='/etc/mysql-ssl/client-cert.pem', MASTER_SSL_KEY='/etc/mysql-ssl/client-key.pem';

mysql> START SLAVE;
```

1.  验证从库的状态：

```go
mysql> SHOW SLAVE STATUS\G
*************************** 1\. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 35.186.158.188
~
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
~
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 354
              Relay_Log_Space: 949
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
 Master_SSL_Allowed: Yes
 Master_SSL_CA_File: /etc/mysql-ssl/ca.pem
           Master_SSL_CA_Path: 
 Master_SSL_Cert: /etc/mysql-ssl/client-cert.pem
            Master_SSL_Cipher: 
 Master_SSL_Key: /etc/mysql-ssl/client-key.pem
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
~
                  Master_UUID: fe17bb86-cd30-11e7-bc3b-42010a940003
             Master_Info_File: mysql.slave_master_info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
~
1 row in set (0.00 sec)
```

1.  一旦在所有从库上进行了与 SSL 相关的更改，在主库上，强制复制用户使用 X509：

```go
mysql> ALTER USER `repl`@`%` REQUIRE X509;
Query OK, 0 rows affected (0.00 sec)
```

请注意，这可能会影响其他复制用户。作为替代方案，您可以创建一个带 SSL 的复制用户和一个普通的复制用户。

1.  验证所有从库上的从库状态。
