# 第二章：使用 MySQL

在本章中，我们将介绍以下内容：

+   使用命令行客户端连接到 MySQL

+   创建数据库

+   创建表

+   插入、更新和删除行

+   加载示例数据

+   选择数据

+   排序结果

+   分组结果（聚合函数）

+   创建用户

+   授予和撤销用户的访问权限

+   将数据选择到文件和表中

+   将数据加载到表中

+   连接表

+   存储过程

+   函数

+   触发器

+   视图

+   事件

+   获取有关数据库和表的信息

# 介绍

在接下来的教程中，我们将学到很多东西。让我们详细看看每一个。

# 使用命令行客户端连接到 MySQL

到目前为止，您已经学会了如何在各种平台上安装 MySQL 8.0。除了安装之外，您还将获得名为`mysql`的命令行客户端实用程序，我们将用它来连接到任何 MySQL 服务器。

# 准备工作

首先，您需要知道要连接到哪个服务器。如果您在一个主机上安装了 MySQL 服务器，并且正在尝试从不同的主机（通常称为客户端）连接到服务器，则应指定服务器的主机名或 IP 地址，并且客户端上应安装`mysql-client`软件包。在上一章中，您安装了 MySQL 服务器和客户端软件包。如果您已经在服务器上（通过 SSH），可以指定`localhost`、`127.0.0.1`或`::1`。

其次，由于您已连接到服务器，下一步需要指定要连接到服务器的端口。默认情况下，MySQL 在端口`3306`上运行。因此，您应该指定`3306`。

现在您知道要连接到哪里了。下一个明显的事情是用户名和密码以登录服务器。您还没有创建任何用户，因此使用 root 用户进行连接。在安装时，您可能已经提供了密码，请使用该密码进行连接。如果更改了密码，请使用新密码。

# 如何做...

可以使用以下任何命令连接到 MySQL 客户端：

```
shell> mysql -h localhost -P 3306 -u <username> -p<password>
shell> mysql --host=localhost --port=3306 --user=root --password=<password>
shell> mysql --host localhost --port 3306 --user root --password=<password>
```

```
shell> mysql --host=localhost --port=3306 --user=root --password  
Enter Password:
```

```
shell> whoami
```

强烈建议不要在命令行中提供密码，而是可以将字段留空；系统会提示您输入密码：

```
mysql> ^DBye
shell> 
```

1.  传递`-P`参数（大写）以指定端口。

1.  传递`-p`参数（小写）以指定密码。

1.  在`-p`参数后没有空格。

1.  对于密码，在`=`后没有空格。

默认情况下，主机被视为`localhost`，端口被视为`3306`，用户被视为当前 shell 用户。

1.  要知道当前用户：

```
mysql> exit;
Bye
shell>
```

1.  要断开连接，请按*Ctrl *+ *D*或输入`exit`：

```
mysql> SELECT 1;
+---+
| 1 |
+---+
| 1 |
+---+
1 row in set (0.00 sec)
```

或使用：

```
mysql> SELECT ^C
mysql> SELECT \c
```

1.  连接到`mysql`提示后，您可以执行后跟分隔符的语句。默认分隔符是分号（`;`）：

```
Warning: Using a password on the command line interface can be insecure.
```

1.  要取消命令，请按*Ctrl *+ *C*或输入`\c`：

```
customer id=1, first_name=Mike, last_name=Christensen country=USA
customer id=2, first_name=Andy, last_name=Hollands, country=Australia
customer id=3, first_name=Ravi, last_name=Vedantam, country=India
customer id=4, first_name= Rajiv, last_name=Perera, country=Sri Lanka
```

不建议使用 root 用户连接到 MySQL。您可以创建用户并通过授予适当的权限来限制用户，这将在*创建用户*和*授予和撤销用户的访问权限*部分中讨论。在那之前，您可以使用 root 用户连接到 MySQL。

# 另请参阅

连接后，您可能会注意到一个警告：

```
shell> mysql -u root -p
Enter Password:
mysql> CREATE DATABASE company;
mysql> CREATE DATABASE `my.contacts`;
```

要了解安全连接的安全方式，请参阅第十四章，*安全*。

一旦连接到命令行提示符，您可以执行 SQL 语句，这些语句可以由`;`、`\g`或`\G`终止。

`;`或`\g`—输出水平显示，`\G`—输出垂直显示。

# 创建数据库

好了，您已经安装了 MySQL 8.0 并连接到了它。现在是时候在其中存储一些数据了，毕竟这就是数据库的用途。在任何**关系数据库管理系统**（**RDBMS**）中，数据存储在行中，这是数据库的基本构建块。行包含列，我们可以在其中存储多组值。

例如，如果您想在数据库中存储有关客户的信息。

这是数据集：

```
mysql> USE company
mysql> USE `my.contacts`
```

你应该将它们保存为行：`(1, 'Mike', 'Christensen', 'USA')`, `(2, 'Andy', 'Hollands', 'Australia')`, `(3, 'Ravi', 'Vedantam', 'India')`, `(4, 'Rajiv', 'Perera', 'Sri Lanka')`。对于这个数据集，有三列描述的四行（id，first_name，last_name 和 country），它们存储在一个表中。表可以容纳的列数应该在创建表的时候定义，这是关系数据库管理系统的主要限制。但是，我们可以随时更改表的定义，但在这样做时应该重建整个表。在某些情况下，更改表时表将不可用。表的更改将在第九章中详细讨论，*表维护*。

数据库是许多表的集合，数据库服务器可以容纳许多这些数据库。流程如下：

数据库服务器—>数据库—>表（由列定义）—>行

数据库和表被称为数据库对象。任何操作，如创建、修改或删除数据库对象，都称为**数据定义语言**（**DDL**）。

数据的组织作为数据库构建的蓝图（分为数据库和表）称为**模式**。

# 如何做...

连接到 MySQL 服务器：

```
shell> mysql -u root -p company
```

反引号字符（`` ` ``）用于引用标识符，如数据库和表名。当数据库名称包含特殊字符，如点(`.`)时，您需要使用它。

您可以在数据库之间切换：

```
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| company    |
+------------+
1 row in set (0.00 sec)
```

无需切换，您可以直接通过命令行指定所需数据库进行连接：

```
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| company            |
| my.contacts        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
6 rows in set (0.00 sec)
```

要查找您当前连接的数据库，请使用以下命令：

```
mysql> SHOW VARIABLES LIKE 'datadir';
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| datadir       | /usr/local/mysql/data/ |
+---------------+------------------------+
1 row in set (0.00 sec)
```

要查找您有权访问的所有数据库，请使用：

```
shell> sudo ls -lhtr /usr/local/mysql/data/ 
total 185M
-rw-r----- 1 mysql mysql   56 Jun  2 16:57 auto.cnf
-rw-r----- 1 mysql mysql  257 Jun  2 16:57 performance_sche_3.SDI
drwxr-x--- 2 mysql mysql 4.0K Jun  2 16:57 performance_schema
drwxr-x--- 2 mysql mysql 4.0K Jun  2 16:57 mysql
-rw-r----- 1 mysql mysql  242 Jun  2 16:57 sys_4.SDI
drwxr-x--- 2 mysql mysql 4.0K Jun  2 16:57 sys
-rw------- 1 mysql root  1.7K Jun  2 16:58 ca-key.pem
-rw-r--r-- 1 mysql root  1.1K Jun  2 16:58 ca.pem
-rw------- 1 mysql root  1.7K Jun  2 16:58 server-key.pem
-rw-r--r-- 1 mysql root  1.1K Jun  2 16:58 server-cert.pem
-rw------- 1 mysql root  1.7K Jun  2 16:58 client-key.pem
-rw-r--r-- 1 mysql root  1.1K Jun  2 16:58 client-cert.pem
-rw------- 1 mysql root  1.7K Jun  2 16:58 private_key.pem
-rw-r--r-- 1 mysql root   451 Jun  2 16:58 public_key.pem
-rw-r----- 1 mysql mysql 1.4K Jun  2 17:46 ib_buffer_pool
-rw-r----- 1 mysql mysql    5 Jun  2 17:46 server1.pid
-rw-r----- 1 mysql mysql  247 Jun  3 13:55 company_5.SDI
drwxr-x--- 2 mysql mysql 4.0K Jun  4 08:13 company
-rw-r----- 1 mysql mysql  12K Jun  4 18:58 server1.err
-rw-r----- 1 mysql mysql  249 Jun  5 16:17 employees_8.SDI
drwxr-x--- 2 mysql mysql 4.0K Jun  5 16:17 employees
-rw-r----- 1 mysql mysql  76M Jun  5 16:18 ibdata1
-rw-r----- 1 mysql mysql  48M Jun  5 16:18 ib_logfile1
-rw-r----- 1 mysql mysql  48M Jun  5 16:18 ib_logfile0
-rw-r----- 1 mysql mysql  12M Jun 10 10:29 ibtmp1
```

数据库作为`data 目录`内的一个目录创建。基于仓库的安装默认`data 目录`为`/var/lib/mysql`，而通过二进制安装则为`/usr/local/mysql/data/`。要了解您的当前`data 目录`，您可以执行：

```
mysql> CREATE TABLE IF NOT EXISTS `company`.`customers` (
`id` int unsigned AUTO_INCREMENT PRIMARY KEY,
`first_name` varchar(20),
`last_name` varchar(20),
`country` varchar(20)
) ENGINE=InnoDB;
```

检查`data 目录`内的文件：

```
mysql> SHOW ENGINES\G
*************************** 1\. row ***************************
      Engine: MRG_MYISAM
     Support: YES
     Comment: Collection of identical MyISAM tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 2\. row ***************************
      Engine: FEDERATED
     Support: NO
     Comment: Federated MySQL storage engine
Transactions: NULL
          XA: NULL
  Savepoints: NULL
*************************** 3\. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
*************************** 4\. row ***************************
      Engine: BLACKHOLE
     Support: YES
     Comment: /dev/null storage engine (anything you write to it disappears)
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 5\. row ***************************
      Engine: CSV
     Support: YES
     Comment: CSV storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 6\. row ***************************
      Engine: MEMORY
     Support: YES
     Comment: Hash based, stored in memory, useful for temporary tables
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 7\. row ***************************
      Engine: PERFORMANCE_SCHEMA
     Support: YES
     Comment: Performance Schema
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 8\. row ***************************
      Engine: ARCHIVE
     Support: YES
     Comment: Archive storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
*************************** 9\. row ***************************
      Engine: MyISAM
     Support: YES
     Comment: MyISAM storage engine
Transactions: NO
          XA: NO
  Savepoints: NO
9 rows in set (0.00 sec)
```

# 参见

您可能会对其他文件和目录感到好奇，例如`information_schema`和`performance_schema`，这些并非您所创建。`information_schema`将在*获取数据库和表信息*部分讨论，而`performance_schema`将在第十三章*性能调优*中的*使用 performance_schema*部分讨论。

# 创建表

在定义表中的列时，应提及列名、数据类型（整数、浮点数、字符串等）及默认值（如有）。MySQL 支持多种数据类型。更多详情请参阅 MySQL 文档（[`dev.mysql.com/doc/refman/8.0/en/data-types.html`](https://dev.mysql.com/doc/refman/8.0/en/data-types.html)）。以下是所有数据类型的概览。`JSON`数据类型是一个新的扩展，将在第三章*使用 MySQL（高级）*中讨论：

1.  数值类型：`TINYINT`, `SMALLINT`, `MEDIUMINT`, `INT`, `BIGINT`, 和 `BIT`。

1.  浮点数类型：`DECIMAL`, `FLOAT`, 和 `DOUBLE`。

1.  字符串类型：`CHAR`, `VARCHAR`, `BINARY`, `VARBINARY`, `BLOB`, `TEXT`, `ENUM`, 和 `SET`。

1.  还支持空间数据类型。更多详情请参阅[`dev.mysql.com/doc/refman/8.0/en/spatial-extensions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-extensions.html)。

1.  `JSON`数据类型——将在下一章详细讨论。

您可以在一个数据库中创建多个表。

# 操作方法...

表包含列定义：

```
mysql> CREATE TABLE `company`.`payments`(
`customer_name` varchar(20) PRIMARY KEY,
`payment` float
);
```

选项解释如下：

+   **点表示法**：可以使用*数据库名点表名*（`database.table`）引用表。如果已连接到数据库，则可以直接使用`customers`代替`company.customers`。

+   `IF NOT EXISTS`：如果存在同名表且您指定此子句，MySQL 仅会发出警告表明该表已存在。否则，MySQL 将抛出错误。

+   `id`：由于仅包含整数，故声明为整型。此外，还有两个关键字：`AUTO_INCREMENT`和`PRIMARY KEY`。

+   `AUTO_INCREMENT`：自动生成线性递增序列，因此您无需担心为每行分配`id`。

+   `PRIMARY KEY`：每行通过一个`UNIQUE`且`NOT NULL`的列来标识。表中只应定义一个这样的列。如果表包含`AUTO_INCREMENT`列，则将其视为`PRIMARY KEY`。

+   `first_name`、`last_name`和`country`：它们包含字符串，因此被定义为`varchar`。

+   **引擎**：除了列定义外，你还应该提及存储引擎。一些存储引擎类型包括`InnoDB`、`MyISAM`、`FEDERATED`、`BLACKHOLE`、`CSV`和`MEMORY`。在所有引擎中，`InnoDB`是唯一的事务引擎，也是默认引擎。要了解更多关于事务的信息，请参阅第五章，*事务*。

要列出所有存储引擎，请执行以下操作：

```
mysql> SHOW TABLES;
+-------------------+
| Tables_in_company |
+-------------------+
| customers         |
| payments          |
+-------------------+
2 rows in set (0.00 sec)
```

你可以在数据库中创建多个表。

再创建一个表来跟踪付款：

```
mysql> SHOW CREATE TABLE customers\G
*************************** 1\. row ***************************
 Table: customers
Create Table: CREATE TABLE `customers` (
 `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
 `first_name` varchar(20) DEFAULT NULL,
 `last_name` varchar(20) DEFAULT NULL,
 `country` varchar(20) DEFAULT NULL,
 PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

要列出所有表，请使用：

```
mysql> DESC customers;
+------------+------------------+------+-----+---------+----------------+
| Field      | Type             | Null | Key | Default | Extra          |
+------------+------------------+------+-----+---------+----------------+
| id         | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| first_name | varchar(20)      | YES  |     | NULL    |                |
| last_name  | varchar(20)      | YES  |     | NULL    |                |
| country    | varchar(20)      | YES  |     | NULL    |                |
+------------+------------------+------+-----+---------+----------------+
4 rows in set (0.01 sec)

```

要查看表的结构，请执行以下操作：

```
shell> sudo ls -lhtr /usr/local/mysql/data/company
total 256K
-rw-r----- 1 mysql mysql 128K Jun 4 07:36 customers.ibd
-rw-r----- 1 mysql mysql 128K Jun 4 08:24 payments.ibd
```

或使用此方法：

```
mysql> CREATE TABLE new_customers LIKE customers;
Query OK, 0 rows affected (0.05 sec)
```

MySQL 在`data 目录`内创建`.ibd`文件：

```
mysql> SHOW CREATE TABLE new_customers\G
*************************** 1\. row ***************************
       Table: new_customers
Create Table: CREATE TABLE `new_customers` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `first_name` varchar(20) DEFAULT NULL,
  `last_name` varchar(20) DEFAULT NULL,
  `country` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

# 克隆表结构

你可以将一个表的结构克隆到新表中：

```
mysql> INSERT IGNORE INTO `company`.`customers`(first_name, last_name,country)
VALUES 
('Mike', 'Christensen', 'USA'),
('Andy', 'Hollands', 'Australia'),
('Ravi', 'Vedantam', 'India'),
('Rajiv', 'Perera', 'Sri Lanka');
```

你可以验证新表的结构：

```
mysql> INSERT IGNORE INTO `company`.`customers`(id, first_name, last_name,country)
VALUES 
(1, 'Mike', 'Christensen', 'USA'),
(2, 'Andy', 'Hollands', 'Australia'),
(3, 'Ravi', 'Vedantam', 'India'),
(4, 'Rajiv', 'Perera', 'Sri Lanka');

Query OK, 0 rows affected, 4 warnings (0.00 sec)
Records: 4 Duplicates: 4 Warnings: 4
```

# 另请参阅

有关`Create Table`中的许多其他选项，请参考[`dev.mysql.com/doc/refman/8.0/en/create-table.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)。第十章，*表维护*将讨论表分区，第十一章，*管理表空间*将讨论表压缩。

# 插入、更新和删除行

`INSERT`、`UPDATE`、`DELETE`和`SELECT`操作被称为**数据操纵语言**（**DML**）语句。`INSERT`、`UPDATE`和`DELETE`也称为写操作，或简称为**写**。`SELECT`是一种读操作，简称为**读**。

# 如何操作...

让我们详细了解每一个操作。我相信你会喜欢学习这些内容。我建议你之后也尝试自己操作一些实例。在本节结束时，我们还将掌握截断表的方法。

# 插入

`INSERT`语句用于在表中创建新记录：

```
mysql> SHOW WARNINGS;
+---------+------+---------------------------------------+
| Level   | Code | Message                               |
+---------+------+---------------------------------------+
| Warning | 1062 | Duplicate entry '1' for key 'PRIMARY' |
| Warning | 1062 | Duplicate entry '2' for key 'PRIMARY' |
| Warning | 1062 | Duplicate entry '3' for key 'PRIMARY' |
| Warning | 1062 | Duplicate entry '4' for key 'PRIMARY' |
+---------+------+---------------------------------------+
4 rows in set (0.00 sec)
```

或者，如果你想插入特定的`id`，可以明确提及`id`列：

```
mysql> UPDATE customers SET first_name='Rajiv', country='UK' WHERE id=4;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

`IGNORE`：如果行已存在且给出了`IGNORE`子句，则新数据被忽略，`INSERT`语句仍然成功，产生警告和重复项的数量。否则，如果没有给出`IGNORE`子句，`INSERT`语句会产生错误。行的唯一性由主键确定：

```
mysql> DELETE FROM customers WHERE id=4 AND first_name='Rajiv';
Query OK, 1 row affected (0.03 sec)
```

# 更新

`UPDATE`语句用于修改表中的现有记录：

```
mysql> REPLACE INTO customers VALUES (1,'Mike','Christensen','America');
Query OK, 2 rows affected (0.03 sec)
```

`WHERE`：这是用于过滤的子句。在`WHERE`子句后发出的任何条件都会被评估，过滤后的行将被更新。

`WHERE`子句是**必需的**。如果没有给出，它将`UPDATE`整个表。

建议在事务中进行数据修改，这样一旦发现问题，你可以轻松回滚更改。关于事务的更多信息，请参阅第五章，*事务*。

# 删除

删除记录可以按以下方式进行：

```
mysql> INSERT INTO payments VALUES('Mike Christensen', 200) ON DUPLICATE KEY UPDATE payment=payment+VALUES(payment);
Query OK, 1 row affected (0.00 sec)
mysql> INSERT INTO payments VALUES('Ravi Vedantam',500) ON DUPLICATE KEY UPDATE payment=payment+VALUES(payment);
Query OK, 1 row affected (0.01 sec)
```

`WHERE`子句是**必需的**。如果没有给出，它将`DELETE`表中的所有行。

建议在事务中进行数据修改，这样如果发现任何错误，你可以轻松回滚更改。

# REPLACE、INSERT、ON DUPLICATE KEY UPDATE

处理重复项的情况很多。行的唯一性由主键标识。如果行已存在，`REPLACE` 会简单地删除该行并插入新行。如果行不存在，`REPLACE` 的行为类似于 `INSERT`。

`ON DUPLICATE KEY UPDATE` 用于当你希望行已存在时采取行动。如果你指定 `ON DUPLICATE KEY UPDATE` 选项，并且 `INSERT` 语句导致 `PRIMARY KEY` 中出现重复值，MySQL 会根据新值更新旧行。

假设你希望每当从同一客户收到付款时更新之前的金额，并且如果客户首次付款，则同时插入一条新记录。为此，你需要定义一个金额列，并在每次收到新付款时更新它：

```
mysql> INSERT INTO payments VALUES('Mike Christensen', 300) ON DUPLICATE KEY UPDATE payment=payment+VALUES(payment);
Query OK, 2 rows affected (0.00 sec)
```

你可以看到有两行受影响，一行重复的被删除，一行新的被插入：

```
mysql> TRUNCATE TABLE customers;
Query OK, 0 rows affected (0.03 sec)
```

```
shell> wget 'https://codeload.github.com/datacharmer/test_db/zip/master' -O master.zip
```

当 `Mike Christensen` 下次支付 $300 时，这将更新该行并将此付款添加到之前的付款中：

```
shell> unzip master.zip
```

`VALUES` (payment)：指的是 `INSERT` 语句中给出的值。Payment 指的是表中的列。

# 截断表

删除整个表需要很长时间，因为 MySQL 逐行执行操作。删除表中所有行（保留表结构）的最快方法是使用 `TRUNCATE TABLE` 语句。

截断是 MySQL 中的 DDL 操作，意味着一旦数据被截断，就无法回滚：

```
shell> cd test_db-master

shell> mysql -u root -p < employees.sql
mysql: [Warning] Using a password on the command line interface can be insecure.
INFO
CREATING DATABASE STRUCTURE
INFO
storage engine: InnoDB
INFO
LOADING departments
INFO
LOADING employees
INFO
LOADING dept_emp
INFO
LOADING dept_manager
INFO
LOADING titles
INFO
LOADING salaries
data_load_time_diff
NULL
```

# 加载示例数据

你已经创建了架构（数据库和表）并通过 `INSERT`、`UPDATE` 和 `DELETE` 添加了一些数据。为了解释后续章节，需要更多数据。MySQL 提供了一个示例 `employee` 数据库和大量数据供你使用。在本章中，我们将讨论如何获取这些数据并将其存储在我们的数据库中。

# 如何操作...

1.  下载压缩文件：

```
shell> mysql -u root -p  employees -A
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 35
Server version: 8.0.3-rc-log MySQL Community Server (GPL)
Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

1.  解压缩文件：

```
mysql> SHOW TABLES;
+-------------------------+
| Tables_in_employees     |
+-------------------------+
| current_dept_emp        |
| departments             |
| dept_emp                |
| dept_emp_latest_date    |
| dept_manager            |
| employees               |
| salaries                |
| titles                  |
+-------------------------+
8 rows in set (0.00 sec)
```

1.  加载数据：

```
mysql> DESC employees\G
*************************** 1\. row ***************************
  Field: emp_no
   Type: int(11)
   Null: NO
    Key: PRI
Default: NULL
  Extra: 
*************************** 2\. row ***************************
  Field: birth_date
   Type: date
   Null: NO
    Key: 
Default: NULL
  Extra: 
*************************** 3\. row ***************************
  Field: first_name
   Type: varchar(14)
   Null: NO
    Key: 
Default: NULL
  Extra: 
*************************** 4\. row ***************************
  Field: last_name
   Type: varchar(16)
   Null: NO
    Key: 
Default: NULL
  Extra: 
*************************** 5\. row ***************************
  Field: gender
   Type: enum('M','F')
   Null: NO
    Key: 
Default: NULL
  Extra: 
*************************** 6\. row ***************************
  Field: hire_date
   Type: date
   Null: NO
    Key: 
Default: NULL
  Extra: 
6 rows in set (0.00 sec)
```

1.  验证数据：

```
mysql> SELECT * FROM departments;
+---------+--------------------+
| dept_no | dept_name          |
+---------+--------------------+
| d009    | Customer Service   |
| d005    | Development        |
| d002    | Finance            |
| d003    | Human Resources    |
| d001    | Marketing          |
| d004    | Production         |
| d006    | Quality Management |
| d008    | Research           |
| d007    | Sales              |
+---------+--------------------+
9 rows in set (0.00 sec)
```

```
mysql> SELECT emp_no, dept_no FROM dept_manager;
+--------+---------+
| emp_no | dept_no |
+--------+---------+
| 110022 | d001    |
| 110039 | d001    |
| 110085 | d002    |
| 110114 | d002    |
| 110183 | d003    |
| 110228 | d003    |
| 110303 | d004    |
| 110344 | d004    |
| 110386 | d004    |
| 110420 | d004    |
| 110511 | d005    |
| 110567 | d005    |
| 110725 | d006    |
| 110765 | d006    |
| 110800 | d006    |
| 110854 | d006    |
| 111035 | d007    |
| 111133 | d007    |
| 111400 | d008    |
| 111534 | d008    |
| 111692 | d009    |
| 111784 | d009    |
| 111877 | d009    |
| 111939 | d009    |
+--------+---------+
24 rows in set (0.00 sec)

```

```
mysql> SELECT COUNT(*) FROM employees;
+----------+
| COUNT(*) |
+----------+
|   300024 |
+----------+
1 row in set (0.03 sec)

```

# 选择数据

你已经在表中插入了和更新了数据。现在是时候学习如何从数据库检索信息了。在本节中，我们将讨论如何从我们创建的示例 `employee` 数据库中检索数据。

使用 `SELECT` 可以做很多事情。本节将讨论最常见的用例。有关语法和其他用例的更多详细信息，请参阅 [`dev.mysql.com/doc/refman/8.0/en/select.html`](https://dev.mysql.com/doc/refman/8.0/en/select.html)。

# 如何操作...

从 `employee` 数据库的 `departments` 表中选择所有数据。你可以使用星号 (`*`) 来选择表中的所有列。不建议这样做，你应该始终只选择所需的数据：

```
mysql> SELECT emp_no FROM employees WHERE first_name='Georgi' AND last_name='Facello';
+--------+
| emp_no |
+--------+
|  10001 |
|  55649 |
+--------+
2 rows in set (0.08 sec)
```

# 选择列

假设你需要从 `dept_manager` 表中获取 `emp_no` 和 `dept_no`：

```
mysql> SELECT COUNT(*) FROM employees WHERE last_name IN ('Christ', 'Lamba', 'Baba');
+----------+
| COUNT(*) |
+----------+
|      626 |
+----------+
1 row in set (0.08 sec)
```

# 计数

从 `employees` 表中查找员工数量：

```
mysql> SELECT COUNT(*) FROM employees WHERE hire_date BETWEEN '1986-12-01' AND '1986-12-31';
+----------+
| COUNT(*) |
+----------+
|     3081 |
+----------+
1 row in set (0.06 sec)
```

# 基于条件过滤

查找名为`Georgi Facello`的员工的`emp_no`：

```
mysql> SELECT COUNT(*) FROM employees WHERE hire_date NOT BETWEEN '1986-12-01' AND '1986-12-31';
+----------+
| COUNT(*) |
+----------+
|   296943 |
+----------+
1 row in set (0.08 sec)
```

所有过滤条件均通过`WHERE`子句给出。除整数和浮点数外，其他所有内容都应放在引号内。

# 操作符

MySQL 支持多种过滤结果的操作符。详细列表请参考[`dev.mysql.com/doc/refman/8.0/en/comparison-operators.html`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html)。这里我们将讨论几个操作符。`LIKE`和`RLIKE`将在后续示例中详细解释：

+   **等式**: 参考前述示例，其中使用了`=`进行筛选。

+   `IN`: 检查一个值是否在一组值中。

    例如，查找姓氏为`Christ`、`Lamba`或`Baba`的员工数量：

```
mysql> SELECT COUNT(*) FROM employees WHERE first_name LIKE 'christ%';
+----------+
| COUNT(*) |
+----------+
|     1157 |
+----------+
1 row in set (0.06 sec)
```

+   `BETWEEN...AND`: 检查一个值是否在一个值范围内。

    例如，查找 1986 年 12 月雇佣的员工人数：

```
mysql> SELECT COUNT(*) FROM employees WHERE first_name LIKE 'christ%ed';
+----------+
| COUNT(*) |
+----------+
|      228 |
+----------+
1 row in set (0.06 sec)
```

+   `NOT`: 只需在前面加上`NOT`操作符即可简单否定结果。

    例如，查找非 1986 年 12 月雇佣的员工人数：

```
mysql> SELECT COUNT(*) FROM employees WHERE first_name LIKE '%sri%';
+----------+
| COUNT(*) |
+----------+
|      253 |
+----------+
1 row in set (0.08 sec)
```

# 简单模式匹配

你可以使用`LIKE`操作符。使用下划线（`_`）匹配恰好一个字符，使用`%`匹配任意数量的字符。

+   查找名字以`Christ`开头的员工数量：

```
mysql> SELECT COUNT(*) FROM employees WHERE first_name LIKE '%er';
+----------+
| COUNT(*) |
+----------+
|     5388 |
+----------+
1 row in set (0.08 sec)
```

+   查找名字以`Christ`开头且以`ed`结尾的员工数量：

```
mysql> SELECT COUNT(*) FROM employees WHERE first_name LIKE '__ka%';
+----------+
| COUNT(*) |
+----------+
|     1918 |
+----------+
1 row in set (0.06 sec)
```

+   查找名字中包含`sri`的员工数量：

```
mysql> SELECT COUNT(*) FROM employees WHERE first_name RLIKE '^christ';
+----------+
| COUNT(*) |
+----------+
|     1157 |
+----------+
1 row in set (0.18 sec)
```

+   查找名字以`er`结尾的员工数量：

```
mysql> SELECT COUNT(*) FROM employees WHERE last_name REGEXP 'ba$';
+----------+
| COUNT(*) |
+----------+
|     1008 |
+----------+
1 row in set (0.15 sec)
```

+   查找名字以任意两个字符开头，后接`ka`，再接任意数量字符的员工数量：

```
mysql> SELECT COUNT(*) FROM employees WHERE last_name NOT REGEXP '[aeiou]';
+----------+
| COUNT(*) |
+----------+
|      148 |
+----------+
1 row in set (0.11 sec)
```

# 正则表达式

你可以在`WHERE`子句中使用`RLIKE`或`REGEXP`操作符使用正则表达式。`REGEXP`有多种用法，更多示例请参考[`dev.mysql.com/doc/refman/8.0/en/regexp.html`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html)：

| **表达式** | **描述** |
| --- | --- |
| `*` | 零或多个重复 |
| `+` | 一个或多个重复 |
| `?` | 可选字符 |
| `.` | 任意字符 |
| `\.` | 句点 |
| `^` | 以...开始 |
| `$` | 以...结束 |
| `[abc]` | 仅*a*、*b*或*c* |
| `[^abc]` | 非*a*、*b*、*c* |
| `[a-z]` | 字符 a 至 z |
| `[0-9]` | 数字 0 至 9 |
| `^...$` | 起始与结束 |
| `\d` | 任意数字 |
| `\D` | 任意非数字字符 |
| `\s` | 任意空白字符 |
| `\S` | 任意非空白字符 |
| `\w` | 任意字母数字字符 |
| `\W` | 任意非字母数字字符 |
| `{m}` | *m*次重复 |
| `{m,n}` | *m*至*n*次重复 |

+   查找名字以`Christ`开头的员工数量：

```
mysql> SELECT first_name, last_name FROM employees WHERE hire_date < '1986-01-01' LIMIT 10;
+------------+------------+
| first_name | last_name  |
+------------+------------+
| Bezalel    | Simmel     |
| Sumant     | Peac       |
| Eberhardt  | Terkki     |
| Otmar      | Herbst     |
| Florian    | Syrotiuk   |
| Tse        | Herber     |
| Udi        | Jansch     |
| Reuven     | Garigliano |
| Erez       | Ritzmann   |
| Premal     | Baek       |
+------------+------------+
10 rows in set (0.00 sec)
```

+   查找所有姓氏以`ba`结尾的员工数量：

```
mysql> SELECT COUNT(*) AS count FROM employees WHERE hire_date BETWEEN '1986-12-01' AND '1986-12-31';
+-------+
| count |
+-------+
|  3081 |
+-------+
1 row in set (0.06 sec)
```

+   查找姓氏不包含元音（a, e, i, o, u）的员工数量：

```
mysql> SELECT emp_no,salary FROM salaries ORDER BY salary DESC LIMIT 5;
+--------+--------+
| emp_no | salary |
+--------+--------+
|  43624 | 158220 |
|  43624 | 157821 |
| 254466 | 156286 |
|  47978 | 155709 |
| 253939 | 155513 |
+--------+--------+
5 rows in set (0.74 sec)
```

# 限制结果

选择 1986 年之前入职的任何 10 名员工的姓名。您可以通过在语句末尾使用`LIMIT`子句来实现这一点：

```
mysql> SELECT emp_no,salary FROM salaries ORDER BY 2 DESC LIMIT 5;
+--------+--------+
| emp_no | salary |
+--------+--------+
|  43624 | 158220 |
|  43624 | 157821 |
| 254466 | 156286 |
|  47978 | 155709 |
| 253939 | 155513 |
+--------+--------+
5 rows in set (0.78 sec)
```

# 使用表别名

默认情况下，您在`SELECT`子句中给出的任何列都将出现在结果中。在前面的示例中，您找到了计数，但它显示为`COUNT(*)`。您可以通过使用`AS`别名来更改它：

```
mysql> SELECT gender, COUNT(*) AS count FROM employees GROUP BY gender;
+--------+--------+
| gender | count  |
+--------+--------+
| M      | 179973 |
| F      | 120051 |
+--------+--------+
2 rows in set (0.14 sec)
```

# 排序结果

您可以根据列或别名列对结果进行排序。您可以指定`DESC`表示降序，或`ASC`表示升序。默认情况下，排序将是升序。您可以将`LIMIT`子句与`ORDER BY`结合使用来限制结果。

# 如何操作...

找出收入最高的前五名员工的员工 ID。

```
mysql> SELECT first_name, COUNT(first_name) AS count FROM employees GROUP BY first_name ORDER BY count DESC LIMIT 10;
+-------------+-------+
| first_name  | count |
+-------------+-------+
| Shahab      |   295 |
| Tetsushi    |   291 |
| Elgin       |   279 |
| Anyuan      |   278 |
| Huican      |   276 |
| Make        |   275 |
| Panayotis   |   272 |
| Sreekrishna |   272 |
| Hatem       |   271 |
| Giri        |   270 |
+-------------+-------+
10 rows in set (0.21 sec)
```

您不必指定列名，也可以在`SELECT`语句中提及列的位置。例如，您正在选择`SELECT`语句中第二位置的薪资。因此，您可以指定`ORDER BY 2`：

```
mysql> SELECT '2017-06-12', YEAR('2017-06-12');
+------------+--------------------+
| 2017-06-12 | YEAR('2017-06-12') |
+------------+--------------------+
| 2017-06-12 |               2017 |
+------------+--------------------+
1 row in set (0.00 sec)
mysql>  SELECT YEAR(from_date), SUM(salary) AS sum FROM salaries GROUP BY YEAR(from_date) ORDER BY sum DESC;
+-----------------+-------------+
| YEAR(from_date) | sum         |
+-----------------+-------------+
|            2000 | 17535667603 |
|            2001 | 17507737308 |
|            1999 | 17360258862 |
|            1998 | 16220495471 |
|            1997 | 15056011781 |
|            1996 | 13888587737 |
|            1995 | 12638817464 |
|            1994 | 11429450113 |
|            2002 | 10243347616 |
|            1993 | 10215059054 |
|            1992 |  9027872610 |
|            1991 |  7798804412 |
|            1990 |  6626146391 |
|            1989 |  5454260439 |
|            1988 |  4295598688 |
|            1987 |  3156881054 |
|            1986 |  2052895941 |
|            1985 |   972864875 |
+-----------------+-------------+
18 rows in set (1.47 sec)
```

# 分组结果（聚合函数）

您可以使用`GROUP BY`子句对列进行分组，然后使用`聚合`函数，如`COUNT`、`MAX`、`MIN`和`AVERAGE`。您也可以在分组子句中对列使用函数。请参阅`SUM`示例，其中您将使用`YEAR()`函数。

# 如何操作...

前面提到的每个聚合函数都将在这里详细介绍给您。

# COUNT

1.  找出男性和女性员工的数量：

```
mysql>  SELECT emp_no, AVG(salary) AS avg FROM salaries GROUP BY emp_no ORDER BY avg DESC LIMIT 10;
+--------+-------------+
| emp_no | avg         |
+--------+-------------+
| 109334 | 141835.3333 |
| 205000 | 141064.6364 |
|  43624 | 138492.9444 |
| 493158 | 138312.8750 |
|  37558 | 138215.8571 |
| 276633 | 136711.7333 |
| 238117 | 136026.2000 |
|  46439 | 135747.7333 |
| 254466 | 135541.0625 |
| 253939 | 135042.2500 |
+--------+-------------+
10 rows in set (0.91 sec
```

1.  您想要找出员工中最常见的 10 个名字。您可以使用`GROUP BY first_name`来分组所有名字，然后使用`COUNT(first_name)`来计算组内的数量，最后使用`ORDER BY`计数来排序结果。将这些结果限制为前 10 名：

```
mysql> SELECT DISTINCT title FROM titles;
+--------------------+
| title              |
+--------------------+
| Senior Engineer    |
| Staff              |
| Engineer           |
| Senior Staff       |
| Assistant Engineer |
| Technique Leader   |
| Manager            |
+--------------------+
7 rows in set (0.30 sec)
```

# SUM

找出每年向员工支付的工资总额，并按工资排序结果。`YEAR()`函数返回给定日期的`YEAR`：

```
mysql>  SELECT emp_no, AVG(salary) AS avg FROM salaries GROUP BY emp_no HAVING avg > 140000 ORDER BY avg DESC;
+--------+-------------+
| emp_no | avg         |
+--------+-------------+
| 109334 | 141835.3333 |
| 205000 | 141064.6364 |
+--------+-------------+
2 rows in set (0.80 sec)
```

```
mysql> CREATE USER IF NOT EXISTS 'company_read_only'@'localhost' 
IDENTIFIED WITH mysql_native_password 
BY 'company_pass' 
WITH MAX_QUERIES_PER_HOUR 500 
MAX_UPDATES_PER_HOUR 100;
```

# AVERAGE

找出平均薪资最高的 10 名员工：

```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

# DISTINCT

您可以使用`DISTINCT`子句来过滤表中的不同条目：

```
mysql> SELECT PASSWORD('company_pass');
+-------------------------------------------+
|PASSWORD('company_pass')                   |
+-------------------------------------------+
| *EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18 |
+-------------------------------------------+
1 row in set, 1 warning (0.00 sec)
mysql> CREATE USER IF NOT EXISTS 'company_read_only'@'localhost' 
IDENTIFIED WITH mysql_native_password 
AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18' 
WITH MAX_QUERIES_PER_HOUR 500 
MAX_UPDATES_PER_HOUR 100;
```

# 使用 HAVING 进行筛选

您可以通过添加`HAVING`子句来筛选`GROUP BY`子句的结果。

例如，找出平均薪资超过 140,000 的员工：

```
mysql> GRANT SELECT ON company.* TO 'company_read_only'@'localhost';
Query OK, 0 rows affected (0.06 sec)
```

# 另请参阅

还有许多其他聚合函数，请参阅[`dev.mysql.com/doc/refman/8.0/en/group-by-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html)了解更多信息。

# 创建用户

到目前为止，您仅使用 root 用户连接到 MySQL 并执行语句。root 用户在访问 MySQL 时绝不应使用，除非从`localhost`执行管理任务。您应该创建用户，限制访问，限制资源使用等。要创建新用户，您应该具有将在下一节中讨论的`CREATE USER`权限。在初始设置期间，您可以使用 root 用户创建其他用户。

# 如何操作...

使用 root 用户连接到 mysql 并执行`CREATE USER`命令来创建新用户。

```
mysql> GRANT INSERT ON company.* TO 'company_insert_only'@'localhost' IDENTIFIED BY 'xxxx';
Query OK, 0 rows affected, 1 warning (0.05 sec)
```

如果密码不够强，您可能会遇到以下错误。

```
mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 1287
Message: Using GRANT for creating new user is deprecated and will be removed in future release. Create new user with CREATE USER statement.
1 row in set (0.00 sec)
```

上述语句将创建具有以下权限的用户：

+   `* 用户名`：`company_read_only`。

+   `* 仅限从`：`localhost`访问。

+   您可以限制访问 IP 范围。例如：`10.148.%.%`。通过给出`%`，用户可以从任何主机访问。

+   `* 密码`：`company_pass`。

+   `* 使用 mysql_native_password`（默认）认证。

+   您也可以指定任何可插拔的认证，如`sha256_password`、`LDAP`或 Kerberos。

+   用户每小时可执行的`* 最大查询次数`为 500 次。

+   用户每小时可执行的`* 最大更新次数`为 100 次。

当客户端连接到 MySQL 服务器时，它经历两个阶段：

1.  访问控制—连接验证

1.  访问控制—请求验证

在连接验证期间，服务器通过用户名和连接来源的主机名识别连接。服务器调用用户的认证插件并验证密码。同时检查用户是否被锁定。

在请求验证阶段，服务器检查用户是否对每个操作拥有足够的权限。

在上述语句中，您必须以明文形式给出密码，这可能会记录在命令历史文件`$HOME/.mysql_history`中。为了避免这种情况，您可以在本地服务器上计算哈希值，并直接指定哈希字符串。其语法相同，只是`mysql_native_password BY 'company_pass'`变为`mysql_native_password AS 'hashed_string'`。

```
mysql> GRANT INSERT, DELETE, UPDATE ON company.* TO 'company_write'@'%' IDENTIFIED WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';
Query OK, 0 rows affected, 1 warning (0.04 sec)
```

```
mysql> GRANT SELECT ON employees.employees TO 'employees_read_only'@'%' IDENTIFIED WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';
Query OK, 0 rows affected, 1 warning (0.03 sec)
```

```
mysql> GRANT SELECT(first_name,last_name)  ON employees.employees TO 'employees_ro'@'%' IDENTIFIED WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';
Query OK, 0 rows affected, 1 warning (0.06 sec)
```

您可以直接通过授予权限来创建用户。请参考下一节了解如何授予权限。不过，MySQL 将在下一版本中弃用此功能。

# 另请参阅

更多关于创建用户选项，请参考[`dev.mysql.com/doc/refman/8.0/en/create-user.html`](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)。更安全的选项，如 SSL，使用其他认证方法将在第十四章*安全*中讨论。

# 授予和撤销用户访问权限

您可以限制用户访问特定的数据库或表，并且仅限于特定的操作，如`SELECT`、`INSERT`和`UPDATE`。要授予其他用户权限，您需要拥有`GRANT`权限。

# 如何操作...

在初始设置期间，您可以使用 root 用户授予权限。您也可以创建一个管理账户来管理用户。

# 授予权限

+   授予`company_read_only`用户`READ ONLY(SELECT)`权限：

```
mysql> GRANT SELECT(salary) ON employees.salaries TO 'employees_ro'@'%';
Query OK, 0 rows affected (0.00 sec)
```

星号(`*`)代表数据库内的所有表。

+   授予新用户`company_insert_only` `INSERT`权限：

```
mysql> CREATE USER 'dbadmin'@'%' IDENTIFIED WITH mysql_native_password BY 'DB@dm1n';
Query OK, 0 rows affected (0.01 sec)
```

```
mysql> GRANT ALL ON *.* TO 'dbadmin'@'%';
Query OK, 0 rows affected (0.01 sec)
```

+   授予新用户`company_write` `WRITE`权限：

```
mysql> GRANT GRANT OPTION ON *.* TO 'dbadmin'@'%';
Query OK, 0 rows affected (0.03 sec)
```

+   限制于特定表。将`employees_read_only`用户限制为仅能从`employees`表执行`SELECT`操作：

```
mysql> SHOW GRANTS FOR 'employees_ro'@'%'\G
*************************** 1\. row ***************************
Grants for employees_ro@%: GRANT USAGE ON *.* TO `employees_ro`@`%`
*************************** 2\. row ***************************
Grants for employees_ro@%: GRANT SELECT (`first_name`, `last_name`) ON `employees`.`employees` TO `employees_ro`@`%`
*************************** 3\. row ***************************
Grants for employees_ro@%: GRANT SELECT (`salary`) ON `employees`.`salaries` TO `employees_ro`@`%`
```

+   您可以进一步限制到特定列。将`employees_ro`用户限制为`employees`表的`first_name`和`last_name`列：

```
mysql> SHOW GRANTS FOR 'dbadmin'@'%'\G
*************************** 1\. row ***************************
Grants for dbadmin@%: GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE, CREATE ROLE, DROP ROLE ON *.* TO `dbadmin`@`%` WITH GRANT OPTION
*************************** 2\. row ***************************
Grants for dbadmin@%: GRANT BINLOG_ADMIN,CONNECTION_ADMIN,ENCRYPTION_KEY_ADMIN,GROUP_REPLICATION_ADMIN,REPLICATION_SLAVE_ADMIN,ROLE_ADMIN,SET_USER_ID,SYSTEM_VARIABLES_ADMIN ON *.* TO `dbadmin`@`%`
2 rows in set (0.00 sec)
```

+   扩展授权。您可以通过执行新的授权来扩展授权。将权限扩展到`employees_col_ro`用户，使其能够访问`salaries`表的薪资信息：

```
mysql> REVOKE DELETE ON company.* FROM 'company_write'@'%';
Query OK, 0 rows affected (0.04 sec)
```

+   创建`SUPER`用户。您需要一个管理账户来管理服务器。`ALL`表示所有权限，但不包括`GRANT`权限：

```
mysql> REVOKE SELECT(salary) ON employees.salaries FROM 'employees_ro'@'%';
Query OK, 0 rows affected (0.03 sec)
```

```
mysql> SELECT * FROM mysql.user WHERE user='dbadmin'\G
*************************** 1\. row ***************************
                  Host: %
                  User: dbadmin
           Select_priv: Y
           Insert_priv: Y
           Update_priv: Y
           Delete_priv: Y
           Create_priv: Y
             Drop_priv: Y
           Reload_priv: Y
         Shutdown_priv: Y
          Process_priv: Y
             File_priv: Y
            Grant_priv: Y
       References_priv: Y
            Index_priv: Y
            Alter_priv: Y
          Show_db_priv: Y
            Super_priv: Y
 Create_tmp_table_priv: Y
      Lock_tables_priv: Y
          Execute_priv: Y
       Repl_slave_priv: Y
      Repl_client_priv: Y
      Create_view_priv: Y
        Show_view_priv: Y
   Create_routine_priv: Y
    Alter_routine_priv: Y
      Create_user_priv: Y
            Event_priv: Y
          Trigger_priv: Y
Create_tablespace_priv: Y
              ssl_type: 
            ssl_cipher: 
           x509_issuer: 
          x509_subject: 
         max_questions: 0
           max_updates: 0
       max_connections: 0
  max_user_connections: 0
                plugin: mysql_native_password
 authentication_string: *AB7018ADD9CB4EDBEB680BB3F820479E4CE815D2
      password_expired: N
 password_last_changed: 2017-06-10 16:24:03
     password_lifetime: NULL
        account_locked: N
      Create_role_priv: Y
        Drop_role_priv: Y
1 row in set (0.00 sec)
```

+   授予`GRANT`权限。用户应具有`GRANT OPTION`权限才能将权限授予其他用户。您可以将`GRANT`权限扩展到`dbadmin`超级用户：

```
mysql> UPDATE mysql.user SET host='localhost' WHERE user='dbadmin';
Query OK, 1 row affected (0.02 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
```

更多权限类型请参考[`dev.mysql.com/doc/refman/8.0/en/grant.html`](https://dev.mysql.com/doc/refman/8.0/en/grant.html)。

# 检查授权

您可以查看所有用户的授权。检查`employee_col_ro`用户的授权：

```
mysql> CREATE USER 'developer'@'%' IDENTIFIED WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18' PASSWORD EXPIRE;
Query OK, 0 rows affected (0.04 sec
```

检查`dbadmin`用户的授权。您可以看到`dbadmin`用户可用的所有授权：

```
shell> mysql -u developer -pcompany_pass
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 31
Server version: 8.0.3-rc-log

Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW DATABASES;
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

# 撤销授权

撤销授权的语法与创建授权相同。您将权限授予用户，并从用户那里撤销权限。

+   撤销`'company_write'@'%'`用户的`DELETE`权限：

```
mysql> ALTER USER 'developer'@'%' IDENTIFIED WITH mysql_native_password BY 'new_company_pass';
Query OK, 0 rows affected (0.03 sec)
```

+   撤销`employee_ro`用户对薪资列的访问权限：

```
mysql> ALTER USER 'developer'@'%' PASSWORD EXPIRE;
Query OK, 0 rows affected (0.06 sec)
```

# 修改 mysql.user 表

所有用户信息及其权限都存储在`mysql.user`表中。如果您有权访问`mysql.user`表，您可以直接修改`mysql.user`表来创建用户和授予权限。

如果您间接修改授权表，使用如`GRANT`、`REVOKE`、`SET PASSWORD`或`RENAME USER`等账户管理语句，服务器会注意到这些更改并立即将授权表加载到内存中。

如果您直接使用如`INSERT`、`UPDATE`或`DELETE`等语句修改授权表，您的更改在重启服务器或告知服务器重新加载表之前不会影响权限检查。如果您直接更改授权表但忘记重新加载它们，您的更改在重启服务器之前不会生效。

通过发出`FLUSH PRIVILEGES`语句可以重新加载`GRANT`表。

查询`mysql.user`表以查找`dbadmin`用户的所有条目：

```
mysql> ALTER USER 'developer'@'%' PASSWORD EXPIRE INTERVAL 90 DAY;
Query OK, 0 rows affected (0.04 sec)
```

您可以看到`dbadmin`用户可以从任何主机（%）访问数据库。您只需更新`mysql.user`表并重新加载授权表，即可将其限制为`localhost`：

```
mysql> ALTER USER 'developer'@'%' ACCOUNT LOCK;
Query OK, 0 rows affected (0.05 sec)
```

```
shell> mysql -u developer -pnew_company_pass
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 3118 (HY000): Access denied for user 'developer'@'localhost'. Account is locked.
```

# 设置用户密码过期

您可以设置用户密码在特定时间间隔后过期；之后，他们需要更改密码。

当应用程序开发者请求数据库访问时，您可以创建一个带有默认密码的账户，然后设置其过期。您可以将密码分享给开发者，之后他们必须更改密码才能继续使用 MySQL。

所有账户创建时密码过期时间等于`default_password_lifetime`变量，该变量默认禁用：

+   创建一个密码已过期的用户。当开发者首次登录并尝试执行任何语句时，会抛出`ERROR 1820 (HY000):`。在执行此语句前，必须使用`ALTER USER`语句重置密码：

```
mysql> ALTER USER 'developer'@'%' ACCOUNT UNLOCK;
Query OK, 0 rows affected (0.00 sec)
```

```
mysql> CREATE ROLE 'app_read_only', 'app_writes', 'app_developer';
Query OK, 0 rows affected (0.01 sec)
```

开发者需使用以下命令更改其密码：

```
mysql> GRANT SELECT ON employees.* TO 'app_read_only';
Query OK, 0 rows affected (0.00 sec)
```

+   手动使现有用户过期：

```
mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO 'app_writes';
Query OK, 0 rows affected (0.00 sec)
```

+   要求每 180 天更改一次密码：

```
mysql> GRANT ALL ON employees.* TO 'app_developer';
Query OK, 0 rows affected (0.04 sec)
```

# 锁定用户

若发现账户有任何问题，可以锁定它。MySQL 支持在使用`CREATE USER`或`ALTER USER`时进行锁定。

通过在`ALTER USER`语句中添加`ACCOUNT LOCK`子句来锁定账户：

```
mysql> CREATE user emp_read_only IDENTIFIED BY 'emp_pass';
Query OK, 0 rows affected (0.06 sec)
```

开发者将收到账户已锁定的错误信息：

```
mysql> CREATE user emp_writes IDENTIFIED BY 'emp_pass';
Query OK, 0 rows affected (0.04 sec)
```

确认后，你可以解锁账户：

```
mysql> CREATE user emp_developer IDENTIFIED BY 'emp_pass';
Query OK, 0 rows affected (0.01 sec)
```

# 为用户创建角色

MySQL 角色是权限的命名集合。与用户账户一样，角色可以被授予和撤销权限。用户账户可以被授予角色，从而将角色权限授予该账户。之前，你为读取、写入和管理创建了单独的用户。对于写权限，你已向用户授予`INSERT`、`DELETE`和`UPDATE`。相反，你可以将这些权限授予一个角色，然后将用户分配给该角色。通过这种方式，你可以避免向可能的许多用户账户单独授予权限。

+   创建角色：

```
mysql> CREATE user emp_read_write IDENTIFIED BY 'emp_pass';
Query OK, 0 rows affected (0.00 sec)
```

+   使用`GRANT`语句为角色分配权限：

```
mysql> GRANT 'app_read_only' TO 'emp_read_only'@'%';
Query OK, 0 rows affected (0.04 sec)
```

```
mysql> GRANT 'app_writes' TO 'emp_writes'@'%';
Query OK, 0 rows affected (0.00 sec)
```

```
mysql> GRANT 'app_developer' TO 'emp_developer'@'%';
Query OK, 0 rows affected (0.00 sec)
```

+   创建用户。如果不指定任何主机，将采用`%`：

```
mysql> GRANT 'app_read_only', 'app_writes' TO 'emp_read_write'@'%';
Query OK, 0 rows affected (0.05 sec)
```

```
mysql> GRANT SELECT ON employees.* TO 'user_ro_file'@'%' IDENTIFIED  WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> GRANT FILE ON *.* TO 'user_ro_file'@'%' IDENTIFIED  WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

```
shell> sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

```
shell> sudo systemctl restart mysql
```

+   使用`GRANT`语句为用户分配角色。你可以为一个用户分配多个角色。

    例如，你可以为`emp_read_write`用户分配读写权限：

```
mysql> SELECT first_name, last_name INTO OUTFILE 'result.csv'
       FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
       LINES TERMINATED BY '\n'
       FROM employees WHERE hire_date<'1986-01-01' LIMIT 10;
Query OK, 10 rows affected (0.00 sec)
```

```
shell> sudo cat /var/lib/mysql/employees/result.csv
"Bezalel","Simmel"
"Sumant","Peac"
"Eberhardt","Terkki"
"Otmar","Herbst"
"Florian","Syrotiuk"
"Tse","Herber"
"Udi","Jansch"
"Reuven","Garigliano"
"Erez","Ritzmann"
"Premal","Baek"
```

```
mysql> CREATE TABLE titles_only AS SELECT DISTINCT title FROM titles;
Query OK, 7 rows affected (0.50 sec)
Records: 7  Duplicates: 0  Warnings: 0
```

```
mysql> INSERT INTO titles_only SELECT DISTINCT title FROM titles;
Query OK, 7 rows affected (0.46 sec)
Records: 7  Duplicates: 0  Warnings: 0
```

作为一项安全措施，避免使用`%`并限制应用程序部署所在 IP 的访问。

# 将数据选择到文件和表中

你可以使用`SELECT INTO OUTFILE`语句将输出保存到文件中。

你可以指定列和行分隔符，之后可以将数据导入其他数据平台。

# 操作方法...

你可以将输出目的地保存为文件或表。

# 保存为文件

+   要将输出保存到文件，你需要`FILE`权限。`FILE`是一个全局权限，这意味着你不能将其限制在特定数据库上。但是，你可以限制用户的选择：

```
mysql> CREATE TABLE employee_names (
       `first_name` varchar(14) NOT NULL,
       `last_name` varchar(16) NOT NULL
       ) ENGINE=InnoDB;
Query OK, 0 rows affected (0.07 sec)
```

+   在 Ubuntu 上，默认情况下，MySQL 不允许你写入文件。你应该在配置文件中设置`secure_file_priv`并重启 MySQL。你将在第四章《配置 MySQL》中了解更多关于配置的信息。在 CentOS、Red Hat 上，`secure_file_priv`被设置为`/var/lib/mysql-files`，这意味着所有文件都将保存在该目录中。

+   目前，请按如下方式启用。打开配置文件并添加`secure_file_priv = /var/lib/mysql`：

```
shell> sudo ls -lhtr /var/lib/mysql/employees/result.csv
-rw-rw-rw- 1 mysql mysql 180 Jun 10 14:53 /var/lib/mysql/employees/result.csv
```

+   重启 MySQL 服务器：

```
mysql> LOAD DATA INFILE 'result.csv' INTO TABLE employee_names 
       FIELDS TERMINATED BY ',' 
       OPTIONALLY ENCLOSED BY '"' 
       LINES TERMINATED BY '\n';
Query OK, 10 rows affected (0.01 sec)
Records: 10  Deleted: 0  Skipped: 0  Warnings: 0
```

以下语句将把输出保存为 CSV 格式：

```
mysql> LOAD DATA INFILE 'result.csv' INTO TABLE employee_names 
       FIELDS TERMINATED BY ','
       OPTIONALLY ENCLOSED BY '"'
       LINES TERMINATED BY '\n'
       IGNORE 1 LINES;
```

你可以检查文件的输出，该文件将在`{secure_file_priv}/{database_name}`指定的路径中创建，在本例中是`/var/lib/mysql/employees/`。如果文件已存在，语句将失败，因此每次执行时你需要给出一个唯一名称或移动文件到不同位置：

```
mysql> LOAD DATA INFILE 'result.csv' REPLACE INTO TABLE employee_names FIELDS TERMINATED BY ','OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n';
Query OK, 10 rows affected (0.01 sec)
Records: 10  Deleted: 0  Skipped: 0  Warnings: 0

mysql> LOAD DATA INFILE 'result.csv' IGNORE INTO TABLE employee_names FIELDS TERMINATED BY ','OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n';
Query OK, 10 rows affected (0.06 sec)
Records: 10  Deleted: 0  Skipped: 0  Warnings: 0
```

# 保存为表

你可以将`SELECT`语句的结果保存到表中。即使表不存在，你也可以使用`CREATE`和`SELECT`来创建表并加载数据。如果表已存在，你可以使用`INSERT`和`SELECT`来加载数据。

你可以将标题保存到一个新的`titles_only`表中：

```
mysql> LOAD DATA LOCAL INFILE 'result.csv' IGNORE INTO TABLE employee_names FIELDS TERMINATED BY ','OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n';
```

如果表已存在，你可以使用`INSERT INTO SELECT`语句：

```
mysql> SELECT emp.emp_no, emp.first_name, emp.last_name 
FROM employees AS emp  
WHERE  emp.emp_no=110022;
+--------+------------+------------+
| emp_no | first_name | last_name  |
+--------+------------+------------+
| 110022 | Margareta  | Markovitch |
+--------+------------+------------+
1 row in set (0.00 sec)
```

为了避免重复，你可以使用`INSERT IGNORE`。然而，在这种情况下，`titles_only`表没有`PRIMARY KEY`。因此，`IGNORE`子句没有任何影响。

# 向表中加载数据

你可以将表数据转储到文件中，反之亦然，即从文件加载数据到表中。这在加载大量数据时广泛使用，是向表中快速加载数据的超级方法。你可以指定列分隔符以将数据加载到相应的列中。你应该拥有对表的`FILE`权限和`INSERT`权限。

# 如何操作...

之前，你已将`first_name`和`last_name`保存到文件中。你可以使用同一文件将数据加载到另一个表中。在加载之前，你应该创建该表。如果表已存在，你可以直接加载。表的列应与文件的字段匹配。

创建一个表来存储数据：

```
mysql> SELECT dept_no FROM dept_manager AS dept_mgr WHERE dept_mgr.emp_no=110022;
+---------+
| dept_no |
+---------+
| d001    |
+---------+
1 row in set (0.00 sec)
```

确保文件存在：

```
mysql> SELECT dept_name FROM departments dept WHERE dept.dept_no='d001';
+-----------+
| dept_name |
+-----------+
| Marketing |
+-----------+
1 row in set (0.00 sec)
```

使用`LOAD DATA INFILE`语句加载数据：

```
mysql> SELECT 
    emp.emp_no, 
    emp.first_name, 
    emp.last_name, 
    dept.dept_name 
FROM 
    employees AS emp 
JOIN dept_manager AS dept_mgr 
    ON emp.emp_no=dept_mgr.emp_no AND emp.emp_no=110022 
JOIN departments AS dept 
    ON dept_mgr.dept_no=dept.dept_no;
+--------+------------+------------+-----------+
| emp_no | first_name | last_name  | dept_name |
+--------+------------+------------+-----------+
| 110022 | Margareta  | Markovitch | Marketing |
+--------+------------+------------+-----------+
1 row in set (0.00 sec)
```

文件可以通过完整路径名给出以指定其确切位置。如果以相对路径名给出，则该名称相对于客户端程序启动的目录进行解释。

+   如果文件包含你想忽略的任何标题，指定`IGNORE n LINES`：

```
mysql> SELECT 
    dept_name, 
    AVG(salary) AS avg_salary 
FROM 
    salaries 
JOIN dept_emp 
    ON salaries.emp_no=dept_emp.emp_no 
JOIN departments 
    ON dept_emp.dept_no=departments.dept_no 
GROUP BY 
    dept_emp.dept_no 
ORDER BY 
    avg_salary 
DESC;
+--------------------+------------+
| dept_name          | avg_salary |
+--------------------+------------+
| Sales              | 80667.6058 |
| Marketing          | 71913.2000 |
| Finance            | 70489.3649 |
| Research           | 59665.1817 |
| Production         | 59605.4825 |
| Development        | 59478.9012 |
| Customer Service   | 58770.3665 |
| Quality Management | 57251.2719 |
| Human Resources    | 55574.8794 |
+--------------------+------------+
9 rows in set (8.29 sec)
```

+   你可以指定`REPLACE`或`IGNORE`来处理重复项：

```
mysql> ALTER TABLE employees ADD INDEX name(first_name, last_name);
Query OK, 0 rows affected (1.95 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SELECT 
     emp1.* 
 FROM 
     employees emp1 
 JOIN employees emp2 
     ON emp1.first_name=emp2.first_name 
     AND emp1.last_name=emp2.last_name 
     AND emp1.gender=emp2.gender 
     AND emp1.hire_date=emp2.hire_date 
     AND emp1.emp_no!=emp2.emp_no 
 ORDER BY 
     first_name, last_name;
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
| 232772 | 1962-05-14 | Keung      | Heusch    | M      | 1986-06-01 |
| 493600 | 1964-01-26 | Keung      | Heusch    | M      | 1986-06-01 |
|  64089 | 1958-01-19 | Marit      | Kolvik    | F      | 1993-12-08 |
| 424486 | 1952-07-06 | Marit      | Kolvik    | F      | 1993-12-08 |
|  40965 | 1952-05-11 | Marsha     | Farrow    | M      | 1989-02-18 |
|  14641 | 1953-05-08 | Marsha     | Farrow    | M      | 1989-02-18 |
| 422332 | 1954-08-17 | Naftali    | Mawatari  | M      | 1985-09-14 |
| 427429 | 1962-11-06 | Naftali    | Mawatari  | M      | 1985-09-14 |
|  19454 | 1955-05-14 | Taisook    | Hutter    | F      | 1985-02-26 |
| 243627 | 1957-02-14 | Taisook    | Hutter    | F      | 1985-02-26 |
+--------+------------+------------+-----------+--------+------------+
10 rows in set (34.01 sec)
```

+   MySQL 假设你想要加载的文件在服务器上可用。如果你从远程客户端机器连接到服务器，你可以指定`LOCAL`来加载位于客户端的文件。本地文件将从客户端复制到服务器。文件保存在服务器的标准临时位置。在 Linux 机器上，它是`/tmp`：

```
mysql> SELECT emp_no FROM titles WHERE title="Senior Engineer" AND from_date="1986-06-26";
+--------+
| emp_no |
+--------+
|  10001 |
|  84305 |
| 228917 |
| 426700 |
| 458304 |
+--------+
5 rows in set (0.14 sec)
```

# 连接表

到目前为止，你已经查看了从单个表插入和检索数据。在本节中，我们将讨论如何结合两个或更多表来检索结果。

一个完美的例子是，你想查找`emp_no: 110022`的员工姓名和部门编号：

+   部门编号和名称存储在`departments`表中

+   员工编号及其他详细信息，如`first_name`和`last_name`，存储在`employees`表中

+   员工与部门的映射存储在`dept_manager`表中

如果你不想使用`JOIN`，你可以这样做：

1.  从`employee`表中查找`emp_no`为`110022`的员工姓名：

```
mysql> SELECT first_name, last_name FROM employees WHERE emp_no IN (< output from preceding query>)

mysql> SELECT first_name, last_name FROM employees WHERE emp_no IN (10001,84305,228917,426700,458304);
+------------+-----------+
| first_name | last_name |
+------------+-----------+
| Georgi     | Facello   |
| Minghong   | Kalloufi  |
| Nechama    | Bennet    |
| Nagui      | Restivo   |
| Shuzo      | Kirkerud  |
+------------+-----------+
5 rows in set (0.00 sec
```

1.  从`departments`表中查找部门编号：

```
mysql> SELECT 
    first_name, 
    last_name 
FROM 
    employees 
WHERE 
    emp_no 
IN (SELECT emp_no FROM titles WHERE title="Senior Engineer" AND from_date="1986-06-26");
+------------+-----------+
| first_name | last_name |
+------------+-----------+
| Georgi     | Facello   |
| Minghong   | Kalloufi  |
| Nagui      | Restivo   |
| Nechama    | Bennet    |
| Shuzo      | Kirkerud  |
+------------+-----------+
5 rows in set (0.91 sec)
```

1.  从`departments`表中查找部门名称：

```
mysql> SELECT emp_no FROM salaries WHERE salary=(SELECT MAX(salary) FROM salaries);
+--------+
| emp_no |
+--------+
|  43624 |
+--------+
1 row in set (1.54 sec)
```

# 如何操作...

为了避免使用三个不同的语句在三个不同的表中查找，你可以使用`JOIN`将它们组合起来。需要注意的是，要连接两个表，你应该有一个或多个共同的列来连接。你可以基于`emp_no`将员工表和`dept_manager`表连接起来，它们都有`emp_no`列。虽然列名不需要匹配，但你应该弄清楚你可以基于哪个列进行连接。同样，`dept_mgr`和`departments`有一个共同的列`dept_no`。

就像给列起别名一样，你可以给表起一个别名，并使用别名引用该表的列。例如，你可以通过`FROM employees AS emp`给员工表起一个别名，并使用点表示法，如`emp.emp_no`，引用`employees`表的列：

```
mysql> CREATE TABLE employees_list1 AS SELECT * FROM employees WHERE first_name LIKE 'aa%';
Query OK, 444 rows affected (0.22 sec)
Records: 444  Duplicates: 0  Warnings: 0

mysql> CREATE TABLE employees_list2 AS SELECT * FROM employees WHERE emp_no BETWEEN 400000 AND 500000 AND gender='F';
Query OK, 39892 rows affected (0.59 sec)
Records: 39892  Duplicates: 0  Warnings: 0
```

我们来看另一个例子——你想找出每个部门的平均薪资。为此，你可以使用`AVG`函数并按`dept_no`分组。要获取部门名称，你可以将结果与`departments`表基于`dept_no`进行连接：

```
mysql> SELECT * FROM employees_list1 WHERE emp_no IN (SELECT emp_no FROM  employees_list2);
```

# 使用自连接识别重复项

你想找出表中特定列的重复行。例如，你想找出哪些员工具有相同的`first_name`、相同的`last_name`、相同的`gender`和相同的`hire_date`。在这种情况下，你可以在`JOIN`子句中指定要查找重复项的列，将`employees`表与自身连接。你需要为每个表使用不同的别名。

你需要在你想要连接的列上添加索引。索引将在第十三章*性能调优*中讨论。现在，你可以执行此命令来添加索引：

```
mysql> SELECT l1.* FROM employees_list1 l1 JOIN employees_list2 l2 ON l1.emp_no=l2.emp_no;
```

```
mysql> SELECT * FROM employees_list1 WHERE emp_no NOT IN (SELECT emp_no FROM  employees_list2);
```

你必须提及`emp1.emp_no != emp2.emp_no`，因为员工会有不同的`emp_no`。否则，同一员工会重复出现。

# 使用子查询

子查询是一个嵌套在另一个语句中的`SELECT`语句。假设你想找出在`1986-06-26`作为`Senior Engineer`开始工作的员工姓名。

你可以从`titles`表中获取`emp_no`，从`employees`表中获取姓名。你也可以使用`JOIN`来找出结果。

从 titles 表中获取`emp_no`：

```
mysql> SELECT l1.* FROM employees_list1 l1 LEFT OUTER JOIN employees_list2 l2 ON l1.emp_no=l2.emp_no WHERE l2.emp_no IS NULL;
```

查找名称的方法：

```
mysql> SELECT l1.* FROM employees_list1 l1 LEFT OUTER JOIN employees_list2 l2 ON l1.emp_no=l2.emp_no WHERE l2.emp_no IS NOT NULL;
```

其他子句，如`EXISTS`和`EQUAL`，在 MySQL 中也得到支持。更多详情，请参考参考手册，[`dev.mysql.com/doc/refman/8.0/en/subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html)：

```
/* DROP the existing procedure if any with the same name before creating */
DROP PROCEDURE IF EXISTS create_employee;
/* Change the delimiter to $$ */
DELIMITER $$
/* IN specifies the variables taken as arguments, INOUT specifies the output variable*/
CREATE PROCEDURE create_employee (OUT new_emp_no INT, IN first_name varchar(20), IN last_name varchar(20), IN gender enum('M','F'), IN birth_date date, IN emp_dept_name varchar(40), IN title varchar(50))
BEGIN   
    /* Declare variables for emp_dept_no and salary */
        DECLARE emp_dept_no char(4);
        DECLARE salary int DEFAULT 60000;

    /* Select the maximum employee number into the variable new_emp_no */
    SELECT max(emp_no) INTO new_emp_no FROM employees;
    /* Increment the new_emp_no */
    SET new_emp_no = new_emp_no + 1;

    /* INSERT the data into employees table */
        /* The function CURDATE() gives the current date) */
    INSERT INTO employees VALUES(new_emp_no, birth_date, first_name, last_name, gender, CURDATE());

    /* Find out the dept_no for dept_name */
    SELECT emp_dept_name;
    SELECT dept_no INTO emp_dept_no FROM departments WHERE dept_name=emp_dept_name;
    SELECT emp_dept_no;

    /* Insert into dept_emp */
    INSERT INTO dept_emp VALUES(new_emp_no, emp_dept_no, CURDATE(), '9999-01-01');

    /* Insert into titles */
    INSERT INTO titles VALUES(new_emp_no, title, CURDATE(), '9999-01-01');

    /* Find salary based on title */
    IF title = 'Staff' 
        THEN SET salary = 100000;
    ELSEIF title = 'Senior Staff' 
        THEN SET salary = 120000;
    END IF;

    /* Insert into salaries */
    INSERT INTO salaries VALUES(new_emp_no, salary, CURDATE(), '9999-01-01');
END
$$
/* Change the delimiter back to ; */
DELIMITER ;
```

找出薪资最高的员工：

```
mysql> GRANT EXECUTE ON employees.* TO 'emp_read_only'@'%';
Query OK, 0 rows affected (0.05 sec)
```

`SELECT MAX(salary) FROM salaries`是给出最高薪资的子查询，要找出对应于该薪资的员工编号，你可以在`WHERE`子句中使用该子查询。

# 查找表间不匹配的行

假设您想查找一个表中不在其他表中的行。您可以通过两种方式实现这一点。使用`NOT IN`子句或使用`OUTER JOIN`。

要找到匹配的行，您可以使用普通`JOIN`，如果您想找到不匹配的行，您可以使用`OUTER JOIN`。普通`JOIN`意味着*A 交集 B*。`OUTER JOIN`给出*A*和*B*的匹配记录，以及*A*的`NULL`不匹配记录。如果您想要`A-B`的输出，您可以使用`WHERE <JOIN COLUMN IN B> IS NULL`子句。

要理解`OUTER JOIN`的用法，请创建两个`employee`表并插入一些值：

```
shell> mysql -u emp_read_only -pemp_pass employees -A
```

您已经知道如何找到同时存在于两个列表中的员工：

```
mysql> CALL create_employee(@new_emp_no, 'John', 'Smith', 'M', '1984-06-19', 'Research', 'Staff');
Query OK, 1 row affected (0.01 sec)
```

或者您可以使用`JOIN`：

```
mysql> SELECT @new_emp_no;
+-------------+
| @new_emp_no |
+-------------+
|      500000 |
+-------------+
1 row in set (0.00 sec)
```

要找出存在于`employees_list1`但不在`employees_list2`中的员工：

```
mysql> SELECT * FROM employees WHERE emp_no=500000;
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
| 500000 | 1984-06-19 | John       | Smith     | M      | 2017-06-17 |
+--------+------------+------------+-----------+--------+------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM salaries WHERE emp_no=500000;
+--------+--------+------------+------------+
| emp_no | salary | from_date  | to_date    |
+--------+--------+------------+------------+
| 500000 | 100000 | 2017-06-17 | 9999-01-01 |
+--------+--------+------------+------------+
1 row in set (0.00 sec)

mysql> SELECT * FROM titles WHERE emp_no=500000;
+--------+-------+------------+------------+
| emp_no | title | from_date  | to_date    |
+--------+-------+------------+------------+
| 500000 | Staff | 2017-06-17 | 9999-01-01 |
+--------+-------+------------+------------+
1 row in set (0.00 sec)
```

或者您可以使用`OUTER JOIN`：

```
shell> vi function.sql;
DROP FUNCTION IF EXISTS get_sal_level;
DELIMITER $$
CREATE FUNCTION get_sal_level(emp int) RETURNS VARCHAR(10)
 DETERMINISTIC
BEGIN
 DECLARE sal_level varchar(10);
 DECLARE avg_sal FLOAT;

 SELECT AVG(salary) INTO avg_sal FROM salaries WHERE emp_no=emp;

 IF avg_sal < 50000 THEN
 SET sal_level = 'BRONZE';
 ELSEIF (avg_sal >= 50000 AND avg_sal < 70000) THEN
 SET sal_level = 'SILVER';
 ELSEIF (avg_sal >= 70000 AND avg_sal < 90000) THEN
 SET sal_level = 'GOLD';
 ELSEIF (avg_sal >= 90000) THEN
 SET sal_level = 'PLATINUM';
 ELSE
 SET sal_level = 'NOT FOUND';
 END IF;
 RETURN (sal_level);
END
$$
DELIMITER ;
```

外连接为连接列表中第二个表的每个不匹配行创建`NULL`列。如果您使用`RIGHT JOIN`，则第一个表将为不匹配的行获取`NULL`值。

您也可以使用`OUTER JOIN`来查找匹配的行。而不是`WHERE l2.emp_no IS NULL`，给出`WHERE emp_no IS NOT NULL`：

```
mysql> SOURCE function.sql;
Query OK, 0 rows affected (0.00 sec)
Query OK, 0 rows affected (0.01 sec)
You have to pass the employee number and the function returns the income level.
mysql> SELECT get_sal_level(10002);
+----------------------+
| get_sal_level(10002) |
+----------------------+
| SILVER               |
+----------------------+
1 row in set (0.00 sec)

mysql> SELECT get_sal_level(10001);
+----------------------+
| get_sal_level(10001) |
+----------------------+
| GOLD                 |
+----------------------+
1 row in set (0.00 sec)

mysql> SELECT get_sal_level(1);
+------------------+
| get_sal_level(1) |
+------------------+
| NOT FOUND        |
+------------------+
1 row in set (0.00 sec)
```

# 存储过程

假设您需要在 MySQL 中执行一系列语句，而不是每次发送所有 SQL 语句，您可以将所有语句封装在一个程序中，并在需要时调用它。存储过程是一组不需要返回值的 SQL 语句。

除了 SQL 语句，您还可以利用变量来存储结果并在存储过程中进行编程操作。例如，您可以编写`IF`、`CASE`子句、逻辑运算和`WHILE`循环。

+   存储函数和过程也称为存储例程。

+   要创建存储过程，您应该拥有`CREATE ROUTINE`权限。

+   存储函数将有一个返回值。

+   存储过程没有返回值。

+   所有代码都写在`BEGIN 和 END`块内。

+   存储函数可以直接在`SELECT`语句中调用。

+   存储过程可以使用`CALL`语句调用。

+   由于存储过程中的语句应以分隔符（`;`）结尾，因此您需要更改 MySQL 的分隔符，以便 MySQL 不会将存储例程内的 SQL 语句解释为普通语句。创建过程后，您可以将分隔符改回默认值。

# 如何操作...

例如，您想要添加一名新员工。您应该更新三个表，即`employees`、`salaries`和`titles`。而不是执行三个语句，您可以开发一个存储过程并调用它来创建新的`employee`。

您必须传递员工的`first_name`、`last_name`、`gender`和`birth_date`，以及员工加入的部门。您可以通过输入变量传递这些信息，并且您应该获得员工编号作为输出。存储过程不返回值，但它可以更新变量，您可以使用它。

以下是一个简单的存储过程示例，用于创建新员工并更新`salary`和`department`表：

```
mysql> SELECT * FROM employees WHERE hire_date = CURDATE();
```

创建存储过程，您可以：

+   将其粘贴到命令行客户端中

+   将其保存到文件中，并使用`mysql -u <user> -p employees < stored_procedure.sql`将其导入 MySQL。

+   源`mysql> SOURCE stored_procedure.sql`文件

要使用存储过程，需向`emp_read_only`用户授予执行权限：

```
mysql> SELECT DATE_ADD(CURDATE(), INTERVAL -7 DAY) AS '7 Days Ago';
```

使用`CALL stored_procedure(OUT variable, IN values)`语句和例程名称调用存储过程。

使用`emp_read_only`账户连接到 MySQL：

```
mysql> SELECT CONCAT(first_name, ' ', last_name) FROM employees LIMIT 1;
+------------------------------------+
| CONCAT(first_name, ' ', last_name) |
+------------------------------------+
| Aamer Anger                        |
+------------------------------------+
1 row in set (0.00 sec)
```

传递您希望存储`@new_emp_no`输出的变量，以及所需的输入值：

```
shell> vi before_insert_trigger.sql
DROP TRIGGER IF EXISTS salary_round;
DELIMITER $$
CREATE TRIGGER salary_round BEFORE INSERT ON salaries
FOR EACH ROW
BEGIN   
        SET NEW.salary=ROUND(NEW.salary);
END
$$
DELIMITER ;
```

选择存储在变量`@new_emp_no`中的`emp_no`值：

```
mysql> SOURCE before_insert_trigger.sql;
Query OK, 0 rows affected (0.06 sec)
Query OK, 0 rows affected (0.00 sec)
```

检查`employees`、`salaries`和`titles`表中是否创建了行：

```
mysql> INSERT INTO salaries VALUES(10002, 100000.79, CURDATE(), '9999-01-01');
Query OK, 1 row affected (0.04 sec)
```

您可以看到，尽管`emp_read_only`对表没有写入权限，但它仍能通过调用存储过程进行写入。如果存储过程的`SQL SECURITY`设置为`INVOKER`，则`emp_read_only`无法修改数据。请注意，如果您是通过`localhost`连接，需为`localhost`用户创建权限。

要列出数据库中的所有存储过程，执行`SHOW PROCEDURE STATUS\G`。要检查现有存储例程的定义，可执行`SHOW CREATE PROCEDURE <procedure_name>\G`。

# 还有更多...

存储过程也用于增强安全性。用户需对存储过程拥有`EXECUTE`权限才能执行。

根据存储例程的定义：

+   `DEFINER`子句指定存储例程的创建者。如未指定，则采用当前用户。

+   `SQL SECURITY`子句指定存储例程的执行上下文，可以是`DEFINER`或`INVOKER`。

`DEFINER`：即使只有例程的`EXECUTE`权限的用户也能调用并获取存储例程的输出，无论该用户对底层表是否有权限。只要`DEFINER`拥有权限就足够了。

`INVOKER`：安全上下文切换到调用存储例程的用户。在这种情况下，调用者应对底层表有访问权限。

# 参见

请参阅文档以获取更多示例和语法，位于[`dev.mysql.com/doc/refman/8.0/en/create-procedure.html`](https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html)。

# 函数

与存储过程类似，您也可以创建存储函数。主要区别在于函数应有返回值，并可在`SELECT`中调用。通常，存储函数用于简化复杂计算。

# 如何操作...

以下是一个如何编写函数及如何调用的示例。假设银行家想根据收入水平发放信用卡，而不暴露实际工资，您可以公开此函数以查找收入水平：

```
mysql> SELECT * FROM salaries WHERE emp_no=10002 AND from_date=CURDATE();
+--------+--------+------------+------------+
| emp_no | salary | from_date  | to_date    |
+--------+--------+------------+------------+
|  10002 | 100001 | 2017-06-18 | 9999-01-01 |
+--------+--------+------------+------------+
1 row in set (0.00 sec)
```

创建函数：

```
mysql> CREATE TABLE salary_audit (emp_no int, user varchar(50), date_modified date);
```

```
shell> vi before_insert_trigger.sql
DELIMITER $$
CREATE TRIGGER salary_audit
BEFORE INSERT
   ON salaries FOR EACH ROW PRECEDES salary_round
BEGIN
   INSERT INTO salary_audit VALUES(NEW.emp_no, USER(), CURDATE());
END; $$
DELIMITER ;
```

要列出数据库中的所有存储函数，执行`SHOW FUNCTION STATUS\G`。要检查现有存储函数的定义，你可以执行`SHOW CREATE FUNCTION <function_name>\G`。

在函数创建中给出`DETERMINISTIC`关键字非常重要。如果一个例程对于相同的输入参数总是产生相同的结果，则被认为是`DETERMINISTIC`，否则为`NOT DETERMINISTIC`。如果在例程定义中既没有给出`DETERMINISTIC`也没有给出`NOT DETERMINISTIC`，默认值为`NOT DETERMINISTIC`。要声明一个函数是确定性的，你必须明确指定`DETERMINISTIC`。

将一个`NON DETERMINISTIC`例程声明为`DETERMINISTIC`可能导致意外结果，因为这会使优化器做出错误的执行计划选择。将`DETERMINISTIC`例程声明为`NON DETERMINISTIC`可能会降低性能，因为可用的优化不会被使用。

# 内置函数

MySQL 提供了众多内置函数。你已经使用过`CURDATE()`函数来获取当前日期。

你可以在`WHERE`子句中使用函数：

```
mysql> INSERT INTO salaries VALUES(10003, 100000.79, CURDATE(), '9999-01-01');
Query OK, 1 row affected (0.06 sec)
```

+   例如，以下函数给出了一周前的确切日期：

```
mysql> SELECT * FROM salary_audit WHERE emp_no=10003;
+--------+----------------+---------------+
| emp_no | user           | date_modified |
+--------+----------------+---------------+
|  10003 | root@localhost | 2017-06-18    |
+--------+----------------+---------------+
1 row in set (0.00 sec)
```

+   添加两个字符串：

```
mysql> CREATE ALGORITHM=UNDEFINED 
DEFINER=`root`@`localhost` 
SQL SECURITY DEFINER VIEW salary_view 
AS 
SELECT emp_no, salary FROM salaries WHERE from_date > '2002-01-01';
```

# 另请参阅

请参考 MySQL 参考手册，获取完整的函数列表，网址为[`dev.mysql.com/doc/refman/8.0/en/func-op-summary-ref.html`](https://dev.mysql.com/doc/refman/8.0/en/func-op-summary-ref.html)。

# 触发器

触发器用于在触发事件之前或之后激活某些操作。例如，你可以有一个触发器在插入表中的每行之前激活，或者在更新每行之后激活。

在无需停机的情况下修改表时（参见第十章《表维护》中的《使用在线模式变更工具修改表》部分），以及出于审计目的，触发器非常有用。假设你想找出某行更新前的值，你可以编写一个触发器，在更新前将这些行保存到另一个表中。另一个表作为审计表，保存了之前的记录。

触发器的动作时间可以是`BEFORE`或`AFTER`，这表示触发器是在每行被修改之前还是之后激活。

触发事件可以是`INSERT`、`DELETE`或`UPDATE`：

+   `INSERT`：每当通过`INSERT`、`REPLACE`或`LOAD DATA`插入新行时，触发器就会激活

+   `UPDATE`：通过`UPDATE`语句

+   `DELETE`：通过`DELETE`或`REPLACE`语句

从 MySQL 5.7 开始，一个表可以同时拥有多个触发器。例如，一个表可以有两个`BEFORE INSERT`触发器。你需要使用`FOLLOWS`或`PRECEDES`指定哪个触发器应该先执行。

# 如何操作...

例如，你想在插入`salaries`表之前对薪资进行四舍五入。`NEW`指的是正在插入的新值：

```
mysql> SELECT emp_no, AVG(salary) as avg FROM salary_view GROUP BY emp_no ORDER BY avg DESC LIMIT 5;
```

通过源文件创建触发器：

```
mysql> SHOW FULL TABLES WHERE TABLE_TYPE LIKE 'VIEW';
```

通过插入一个浮点数来测试触发器：

```
mysql> SHOW CREATE VIEW salary_view\G
```

你可以看到薪资已被四舍五入：

```
mysql> UPDATE salary_view SET salary=100000 WHERE emp_no=10001;
Query OK, 1 row affected (0.01 sec)
Rows matched: 2 Changed: 1 Warnings: 0
mysql> INSERT INTO salary_view VALUES(10001,100001);
ERROR 1423 (HY000): Field of view 'employees.salary_view' underlying table doesn't have a default value
```

同样，您可以创建一个 `BEFORE UPDATE` 触发器来四舍五入工资。另一个例子：您想要记录哪个用户插入了 `salaries` 表。创建一个 `audit` 表：

```
mysql> SET GLOBAL event_scheduler = ON;
```

请注意，以下触发器先于 `salary_round` 触发器，由 `PRECEDES salary_round` 指定：

```
mysql> DROP EVENT IF EXISTS purge_salary_audit;
DELIMITER $$
CREATE EVENT IF NOT EXISTS purge_salary_audit
ON SCHEDULE
  EVERY 1 WEEK
  STARTS CURRENT_DATE 
    DO BEGIN
        DELETE FROM salary_audit WHERE date_modified < DATE_ADD(CURDATE(), INTERVAL -7 day);
    END $$
DELIMITER ;
```

插入到 `salaries` 中：

```
mysql> SHOW EVENTS\G
*************************** 1\. row ***************************
                  Db: employees
                Name: purge_salary_audit
             Definer: root@localhost
           Time zone: SYSTEM
                Type: RECURRING
          Execute at: NULL
      Interval value: 1
      Interval field: MINUTE
              Starts: 2017-06-18 00:00:00
                Ends: NULL
              Status: ENABLED
          Originator: 0
character_set_client: utf8
collation_connection: utf8_general_ci
  Database Collation: utf8mb4_0900_ai_ci
1 row in set (0.00 sec)
```

通过查询 `salary_audit` 表找出谁插入了工资：

```
mysql> SHOW CREATE EVENT purge_salary_audit\G
```

如果 `salary_audit` 表被删除或不可用，则 `salaries` 表上的所有插入都将被阻止。如果您不想进行审计，应先删除触发器，然后再删除表。

触发器会根据其复杂性对写入速度产生开销。

要检查所有触发器，执行 `SHOW TRIGGERS\G`。

要检查现有触发器的定义，执行 `SHOW CREATE TRIGGER <trigger_name>`。

# 另请参阅

有关更多详细信息，请参阅 MySQL 参考手册，网址为 [`dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html)。

# 视图

视图是基于 SQL 语句结果集的虚拟表。它也将具有行和列，就像真实表一样，但有一些限制，这些将在后面讨论。视图隐藏了 SQL 的复杂性，更重要的是，提供了额外的安全性。

# 如何操作...

假设您只想授予对 `salaries` 表的 `emp_no` 和 `salary` 列的访问权限，并且 `from_date` 在 `2002-01-01` 之后。为此，您可以创建一个提供所需结果的视图的 SQL。

```
mysql> ALTER EVENT purge_salary_audit DISABLE;
mysql> ALTER EVENT purge_salary_audit ENABLE;
```

现在 `salary_view` 视图已创建，您可以像查询任何其他表一样查询它：

```
mysql> USE INFORMATION_SCHEMA;
mysql> SHOW TABLES;
```

您可以看到视图只能访问特定行（即 `from_date > '2002-01-01'`），而不是所有行。您可以使用视图来限制用户对特定行的访问。

要列出所有视图，执行：

```
mysql> DESC INFORMATION_SCHEMA.TABLES;
+-----------------+--------------------------------------------------------------------+------+-----+-------------------+-----------------------------+
| Field           | Type                                                               | Null | Key | Default           | Extra                       |
+-----------------+--------------------------------------------------------------------+------+-----+-------------------+-----------------------------+
| TABLE_CATALOG   | varchar(64)                                                        | NO   |     | NULL              |                             |
| TABLE_SCHEMA    | varchar(64)                                                        | NO   |     | NULL              |                             |
| TABLE_NAME      | varchar(64)                                                        | NO   |     | NULL              |                             |
| TABLE_TYPE      | enum('BASE TABLE','VIEW','SYSTEM VIEW')                            | NO   |     | NULL              |                             |
| ENGINE          | varchar(64)                                                        | YES  |     | NULL              |                             |
| VERSION         | int(2)                                                             | YES  |     | NULL              |                             |
| ROW_FORMAT      | enum('Fixed','Dynamic','Compressed','Redundant','Compact','Paged') | YES  |     | NULL              |                             |
| TABLE_ROWS      | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| AVG_ROW_LENGTH  | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| DATA_LENGTH     | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| MAX_DATA_LENGTH | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| INDEX_LENGTH    | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| DATA_FREE       | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| AUTO_INCREMENT  | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| CREATE_TIME     | timestamp                                                          | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| UPDATE_TIME     | timestamp                                                          | YES  |     | NULL              |                             |
| CHECK_TIME      | timestamp                                                          | YES  |     | NULL              |                             |
| TABLE_COLLATION | varchar(64)                                                        | YES  |     | NULL              |                             |
| CHECKSUM        | bigint(20) unsigned                                                | YES  |     | NULL              |                             |
| CREATE_OPTIONS  | varchar(256)                                                       | YES  |     | NULL              |                             |
| TABLE_COMMENT   | varchar(256)                                                       | YES  |     | NULL              |                             |
+-----------------+--------------------------------------------------------------------+------+-----+-------------------+-----------------------------+
21 rows in set (0.00 sec)
```

要检查视图的定义，执行：

```
mysql> SELECT SUM(DATA_LENGTH)/1024/1024 AS DATA_SIZE_MB, SUM(INDEX_LENGTH)/1024/1024 AS INDEX_SIZE_MB, SUM(DATA_FREE)/1024/1024 AS DATA_FREE_MB FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='employees';
+--------------+---------------+--------------+
| DATA_SIZE_MB | INDEX_SIZE_MB | DATA_FREE_MB |
+--------------+---------------+--------------+
|  17.39062500 |   14.62500000 |  11.00000000 |
+--------------+---------------+--------------+
1 row in set (0.01 sec)
```

您可能已经注意到 `current_dept_emp` 和 `dept_emp_latest_date` 视图，它们是 `employee` 数据库的一部分。您可以探索其定义并找出其用途。

不包含子查询、`JOIN`、`GROUP BY` 子句、联合等的简单视图可以更新。`salary_view` 是一个简单视图，如果底层表有默认值，则可以更新或插入。

```
mysql> SELECT * FROM COLUMNS WHERE TABLE_NAME='employees'\G
```

```
mysql> SELECT * FROM FILES WHERE FILE_NAME LIKE './employees/employees.ibd'\G
~~~
EXTENT_SIZE: 1048576
AUTOEXTEND_SIZE: 4194304
DATA_FREE: 13631488
~~~
```

如果表有默认值，即使它不匹配视图中的筛选条件，也可以插入一行。为了避免这种情况，并插入满足视图条件的行，必须在定义中提供 `WITH CHECK OPTION`。

`VIEW` 算法：

+   `MERGE`：MySQL 将输入查询与视图定义合并为一个查询，然后执行合并后的查询。`MERGE` 算法仅适用于简单视图。

+   `TEMPTABLE`：MySQL 将结果存储在临时表中，然后针对该临时表执行输入查询。

+   `UNDEFINED`（默认）：MySQL 自动选择`MERGE`或`TEMPTABLE`算法。MySQL 更喜欢`MERGE`算法而不是`TEMPTABLE`算法，因为`MERGE`算法效率更高。

# 事件

正如 Linux 服务器上的 cron 一样，MySQL 使用`EVENTS`来处理计划任务。MySQL 使用一个名为事件调度线程的特殊线程来执行所有计划事件。默认情况下，事件调度线程未启用（版本< 8.0.3），要启用它，请执行以下操作：

```
mysql> SELECT * FROM INNODB_TABLESPACES WHERE NAME='employees/employees'\G
*************************** 1\. row ***************************
 SPACE: 118
 NAME: employees/employees
 FLAG: 16417
 ROW_FORMAT: Dynamic
 PAGE_SIZE: 16384
 ZIP_PAGE_SIZE: 0
 SPACE_TYPE: Single
 FS_BLOCK_SIZE: 4096
 FILE_SIZE: 32505856
ALLOCATED_SIZE: 32509952
1 row in set (0.00 sec)
```

# 如何操作...

假设您不再需要保留超过一个月的工资审计记录，您可以安排一个每天运行的事件，并从`salary_audit`表中删除超过一个月的记录。

```
shell> sudo ls -ltr /var/lib/mysql/employees/employees.ibd
-rw-r----- 1 mysql mysql 32505856 Jun 20 16:50 /var/lib/mysql/employees/employees.ibd
```

一旦创建了事件，它将自动执行清除工资审计记录的工作。

+   要检查事件，请执行以下操作：

```
mysql> SELECT * FROM INNODB_TABLESTATS WHERE NAME='employees/employees'\G
*************************** 1\. row ***************************
 TABLE_ID: 128
 NAME: employees/employees
 STATS_INITIALIZED: Initialized
 NUM_ROWS: 299468
 CLUST_INDEX_SIZE: 1057
 OTHER_INDEX_SIZE: 545
 MODIFIED_COUNTER: 0
 AUTOINC: 0
 REF_COUNT: 1
1 row in set (0.00 sec)
```

+   要检查事件的定义，请执行以下操作：

```
mysql> SELECT * FROM PROCESSLIST\G
*************************** 1\. row ***************************
     ID: 85
   USER: event_scheduler
   HOST: localhost
     DB: NULL
COMMAND: Daemon
   TIME: 44
  STATE: Waiting for next activation
   INFO: NULL
*************************** 2\. row ***************************
     ID: 26231
   USER: root
   HOST: localhost
     DB: information_schema
COMMAND: Query
   TIME: 0
  STATE: executing
   INFO: SELECT * FROM PROCESSLIST
2 rows in set (0.00 sec
```

+   要禁用/启用事件，请执行以下操作：

[PRE171]

# 访问控制

所有存储程序（过程、函数、触发器和事件）和视图都有一个`DEFINER`。如果未指定`DEFINER`，则创建对象的用户将被选为`DEFINER`。

存储例程（过程和函数）和视图具有`SQL SECURITY`特性，其值为`DEFINER`或`INVOKER`，以指定对象是在定义者还是调用者上下文中执行。触发器和事件没有`SQL SECURITY`特性，并且始终在定义者上下文中执行。服务器根据需要自动调用这些对象，因此没有调用用户。

# 另请参阅

安排事件的方式有很多，详情请参阅[`dev.mysql.com/doc/refman/8.0/en/event-scheduler.html`](https://dev.mysql.com/doc/refman/8.0/en/event-scheduler.html)。

# 获取数据库和表的信息

您可能已经注意到数据库列表中的`information_schema`数据库。`information_schema`是一个包含有关所有数据库对象的元数据的视图集合。您可以连接到`information_schema`并探索所有表。本章解释了最广泛使用的表。您可以查询`information_schema`表或使用`SHOW`命令，这本质上做的是相同的事情。

`INFORMATION_SCHEMA`查询作为视图实现于`数据字典`表之上。`INFORMATION_SCHEMA`表中有两种类型的元数据：

+   **静态表元数据**：`TABLE_SCHEMA`、`TABLE_NAME`、`TABLE_TYPE`和`ENGINE`。这些统计数据将直接从`数据字典`读取。

+   **动态表元数据**：`AUTO_INCREMENT`、`AVG_ROW_LENGTH`及`DATA_FREE`。动态元数据经常变动（例如，每次`INSERT`后`AUTO_INCREMENT`值会递增）。在许多情况下，按需准确计算动态元数据会产生一定成本，且对于典型查询而言，精确性未必有益。以`DATA_FREE`统计为例，它显示表中空闲字节数——通常缓存值已足够。

MySQL 8.0 中，动态表元数据默认会被缓存。这可通过`information_schema_stats`设置（默认缓存）进行配置，并可改为`SET @@GLOBAL.information_schema_stats='LATEST'`，以便始终直接从存储引擎获取动态信息（代价是查询执行速度略高）。

作为替代方案，用户也可对表执行`ANALYZE TABLE`，以更新缓存的动态统计信息。

大多数表都有`TABLE_SCHEMA`列，指代数据库名称，以及`TABLE_NAME`列，指代表名。

更多详情，请参考[`mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/`](https://mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/)。

# 操作方法...

查看所有表的列表：

[PRE172]

# TABLES

`TABLES`表包含了关于表的所有信息，如所属数据库`TABLE_SCHEMA`、行数(`TABLE_ROWS`)、`ENGINE`、`DATA_LENGTH`、`INDEX_LENGTH`及`DATA_FREE`：

[PRE173]

例如，你想了解`employees`数据库中的`DATA_LENGTH`、`INDEX_LENGTH`及`DATE_FREE`：

[PRE174]

# COLUMNS

此表列出了每张表的所有列及其定义：

[PRE175]

# FILES

你已知晓 MySQL 将`InnoDB`数据存储在`data`目录中（与数据库同名）的`.ibd`文件内。如需获取文件的更多信息，可查询`FILES`表：

[PRE176]

你应该密切关注`DATA_FREE`，它代表未分配的段加上由于碎片化而在段内空闲的数据。当你重建表时，可以释放`DATA_FREE`中显示的字节。

# INNODB_SYS_TABLESPACES

文件大小也可在`INNODB_TABLESPACES`表中查询：

[PRE177]

你可以在文件系统中验证相同信息：

[PRE178]

# INNODB_TABLESTATS

索引大小及大致行数可在`INNODB_TABLESTATS`表中查询：

[PRE179]

# PROCESSLIST

最常用的视图之一是进程列表，它列出了服务器上运行的所有查询：

[PRE180]

或者您可以执行`SHOW PROCESSLIST;`以获得相同的输出。

**其他表**：`ROUTINES`包含函数和存储过程的定义。`TRIGGERS`包含触发器的定义。`VIEWS`包含视图的定义。

# 另请参阅

要了解`INFORMATION_SCHEMA`的改进，请参阅[`mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/`](http://mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/)。
