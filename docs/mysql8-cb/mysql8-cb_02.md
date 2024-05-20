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

```go
shell> mysql -h localhost -P 3306 -u <username> -p<password>
```

```go
shell> mysql --host=localhost --port=3306 --user=root --password=<password>
```

```go
shell> mysql --host localhost --port 3306 --user root --password=<password>
```

强烈建议不要在命令行中提供密码，而是可以将字段留空；系统会提示您输入密码：

```go
shell> mysql --host=localhost --port=3306 --user=root --password  
Enter Password:
```

1.  传递`-P`参数（大写）以指定端口。

1.  传递`-p`参数（小写）以指定密码。

1.  在`-p`参数后没有空格。

1.  对于密码，在`=`后没有空格。

默认情况下，主机被视为`localhost`，端口被视为`3306`，用户被视为当前 shell 用户。

1.  要知道当前用户：

```go
shell> whoami
```

1.  要断开连接，请按*Ctrl *+ *D*或输入`exit`：

```go
mysql> ^DBye
shell> 
```

或使用：

```go
mysql> exit;
Bye
shell>
```

1.  连接到`mysql`提示后，您可以执行后跟分隔符的语句。默认分隔符是分号（`;`）：

```go
mysql> SELECT 1;
+---+
| 1 |
+---+
| 1 |
+---+
1 row in set (0.00 sec)
```

1.  要取消命令，请按*Ctrl *+ *C*或输入`\c`：

```go
mysql> SELECT ^C
mysql> SELECT \c
```

不建议使用 root 用户连接到 MySQL。您可以创建用户并通过授予适当的权限来限制用户，这将在*创建用户*和*授予和撤销用户的访问权限*部分中讨论。在那之前，您可以使用 root 用户连接到 MySQL。

# 另请参阅

连接后，您可能会注意到一个警告：

```go
Warning: Using a password on the command line interface can be insecure.
```

要了解安全连接的安全方式，请参阅第十四章，*安全*。

一旦连接到命令行提示符，您可以执行 SQL 语句，这些语句可以由`;`、`\g`或`\G`终止。

`;`或`\g`—输出水平显示，`\G`—输出垂直显示。

# 创建数据库

好了，您已经安装了 MySQL 8.0 并连接到了它。现在是时候在其中存储一些数据了，毕竟这就是数据库的用途。在任何**关系数据库管理系统**（**RDBMS**）中，数据存储在行中，这是数据库的基本构建块。行包含列，我们可以在其中存储多组值。

例如，如果您想在数据库中存储有关客户的信息。

这是数据集：

```go
customer id=1, first_name=Mike, last_name=Christensen country=USA
customer id=2, first_name=Andy, last_name=Hollands, country=Australia
customer id=3, first_name=Ravi, last_name=Vedantam, country=India
customer id=4, first_name= Rajiv, last_name=Perera, country=Sri Lanka
```

你应该将它们保存为行：`(1, 'Mike', 'Christensen', 'USA')`, `(2, 'Andy', 'Hollands', 'Australia')`, `(3, 'Ravi', 'Vedantam', 'India')`, `(4, 'Rajiv', 'Perera', 'Sri Lanka')`。对于这个数据集，有三列描述的四行（id，first_name，last_name 和 country），它们存储在一个表中。表可以容纳的列数应该在创建表的时候定义，这是关系数据库管理系统的主要限制。但是，我们可以随时更改表的定义，但在这样做时应该重建整个表。在某些情况下，更改表时表将不可用。表的更改将在第九章中详细讨论，*表维护*。

数据库是许多表的集合，数据库服务器可以容纳许多这些数据库。流程如下：

数据库服务器—>数据库—>表（由列定义）—>行

数据库和表被称为数据库对象。任何操作，如创建、修改或删除数据库对象，都称为**数据定义语言**（**DDL**）。

数据的组织作为数据库构建的蓝图（分为数据库和表）称为**模式**。

# 如何做...

连接到 MySQL 服务器：

```go
shell> mysql -u root -p
Enter Password:
mysql> CREATE DATABASE company;
mysql> CREATE DATABASE `my.contacts`;
```

反引号字符（```go) is used to quote identifiers, such as database and table names. You need to use it when the database name contains special characters, such as dot (`.`).

You can switch between databases:

```

mysql> USE company

mysql> USE `my.contacts`

```go

Instead of switching, you can directly connect to the database you want by specifying it in the command line:

```

shell> mysql -u root -p company

```go

To find which database you are connected to, use the following:

```

mysql> SELECT DATABASE();

+------------+

| DATABASE() |
| --- |

+------------+

| company    |
| --- |

+------------+

1 行（0.00 秒）

```go

To find all the databases you have access to, use:

```

mysql> SHOW DATABASES;

+--------------------+

| Database           |
| --- |

+--------------------+

| company            |
| --- |
| my.contacts        |
| information_schema |
| mysql              |
| performance_schema |
| sys                |

+--------------------+

6 行（0.00 秒）

```go

The database is created as a directory inside `data directory`. The default `data directory` is `/var/lib/mysql` for repository-based installation and `/usr/local/mysql/data/` for installation through binaries. To know your current `data directory`, you can execute:

```

mysql> SHOW VARIABLES LIKE 'datadir';

+---------------+------------------------+

| Variable_name | Value                  |
| --- | --- |

+---------------+------------------------+

| datadir       | /usr/local/mysql/data/ |
| --- | --- |

+---------------+------------------------+

1 行（0.00 秒）

```go

Check the files inside `data directory` :

```

shell> sudo ls -lhtr /usr/local/mysql/data/

总共 185M

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

```go

# See also

You might be wondering about other files and directories, such as `information_schema` and `performance_schema`, which you have not created. `information_schema` will be discussed in the *Getting information about databases and tables* section and `performance_schema` will be discussed in Chapter 13, *Performance Tuning*, in the *Using performance_schema* section.

# Creating tables

While defining columns in a table, you should mention the name of the column, datatype (integer, floating point, string, and so on), and default value (if any). MySQL supports various datatypes. Refer to the MySQL documentation for more details ([`dev.mysql.com/doc/refman/8.0/en/data-types.html`](https://dev.mysql.com/doc/refman/8.0/en/data-types.html))). Here is an overview of all datatypes. The  `JSON` datatype is a new extension, which will be discussed in Chapter 3, *Using MySQL (Advanced)*:

1.  Numeric: `TINYINT`, `SMALLINT`, `MEDIUMINT`, `INT`, `BIGINT`, and `BIT`.
2.  Floating numbers: `DECIMAL`, `FLOAT`, and `DOUBLE`.
3.  Strings: `CHAR`, `VARCHAR`, `BINARY`, `VARBINARY`, `BLOB`, `TEXT`, `ENUM`, and `SET`.
4.  Spatial datatypes are also supported. Refer to [`dev.mysql.com/doc/refman/8.0/en/spatial-extensions.html`](https://dev.mysql.com/doc/refman/8.0/en/spatial-extensions.html) for more details.
5.  The `JSON` datatype - discussed in detail in the next chapter.

You can create many tables inside a database.

# How to do it...

The table contains the column definition:

```

mysql> CREATE TABLE IF NOT EXISTS `company`.`customers` (

`id` int unsigned AUTO_INCREMENT PRIMARY KEY,

`first_name` varchar(20),

`last_name` varchar(20),

`country` varchar(20)

）ENGINE=InnoDB;

```go

The options are explained as follows:

*   **Dot notation**: Tables can be referenced using *database name dot table name* (`database.table`). If you are connected to the database, you can simply use `customers` instead of `company.customers`.
*   `IF NOT EXISTS`: If a table with the same name exists and you specify this clause, MySQL simply throws a warning that the table already exists. Otherwise, MySQL will throw an error.
*   `id`: It is declared as an integer since it contains only integers. Along with that, there are two key words: `AUTO_INCREMENT` and `PRIMARY KEY`.
*   `AUTO_INCREMENT`: A linearly incremental sequence is automatically generated, so you do not need to worry about assigning `id` to each row.
*   `PRIMARY KEY`: Each row is identified by a `UNIQUE` column that is `NOT NULL`. Only one of these columns should be defined in a table. If a table contains an `AUTO_INCREMENT` column, it is taken as `PRIMARY KEY`.
*   `first_name`, `last_name`, and `country`: They contain strings, so they are defined as `varchar`.
*   **Engine**: Along with the column definition, you should mention the storage engine. Some types of storage engines include `InnoDB`, `MyISAM`, `FEDERATED`, `BLACKHOLE`, `CSV`, and `MEMORY`. Out of all the engines, `InnoDB` is the only transactional engine and it is the default engine. To learn more about transactions, refer to Chapter 5, *Transactions*.

To list all the storage engines, execute the following:

```

mysql> SHOW ENGINES\G

*************************** 1\. row ***************************

引擎：MRG_MYISAM

支持：是

评论：相同 MyISAM 表的集合

事务：否

XA：否

保存点：否

*************************** 2\. 行 ***************************

引擎：FEDERATED

支持：否

注释：联合 MySQL 存储引擎

事务：NULL

XA：NULL

保存点：NULL

*************************** 3\. 行 ***************************

引擎：InnoDB

支持：默认

注释：支持事务，行级锁定和外键

事务：是

XA：是

保存点：是

*************************** 4\. 行 ***************************

引擎：BLACKHOLE

支持：是

注释：/dev/null 存储引擎（您写入其中的任何内容都会消失）

事务：否

XA：否

保存点：否

*************************** 5\. 行 ***************************

引擎：CSV

支持：是

注释：CSV 存储引擎

事务：否

XA：否

保存点：否

*************************** 6\. 行 ***************************

引擎：MEMORY

支持：是

注释：基于哈希，存储在内存中，对临时表有用

事务：否

XA：否

保存点：否

*************************** 7\. 行 ***************************

引擎：PERFORMANCE_SCHEMA

支持：是

注释：性能模式

事务：否

XA：否

保存点：否

*************************** 8\. 行 ***************************

引擎：ARCHIVE

支持：是

注释：存档存储引擎

事务：否

XA：否

保存点：否

*************************** 9\. 行 ***************************

引擎：MyISAM

支持：是

注释：MyISAM 存储引擎

事务：否

XA：否

保存点：否

设置中的 9 行（0.00 秒）

```go

You can create many tables in a database.

Create one more table to track the payments:

```

mysql> CREATE TABLE `company`.`payments`(

`customer_name` varchar(20) PRIMARY KEY，

`payment` float

）;

```go

To list all the tables, use:

```

mysql> SHOW TABLES;

+-------------------+

| Tables_in_company |
| --- |

+-------------------+

| customers         |
| --- |
| payments          |

+-------------------+

设置中的 2 行（0.00 秒）

```go

To see the structure of the table, execute the following:

```

mysql> SHOW CREATE TABLE customers\G

*************************** 1\. 行 ***************************

表：customers

创建表：CREATE TABLE `customers` (

`id` int(10) unsigned NOT NULL AUTO_INCREMENT,

`first_name` varchar(20) DEFAULT NULL,

`last_name` varchar(20) DEFAULT NULL,

`country` varchar(20) DEFAULT NULL,

主键（`id`）

）ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8mb4

设置中的 1 行（0.00 秒）

```go

Or use this:

```

mysql> DESC customers;

+------------+------------------+------+-----+---------+----------------+

| 字段 | 类型 | 空 | 键 | 默认 | 额外 |
| --- | --- | --- | --- | --- | --- |

+------------+------------------+------+-----+---------+----------------+

| id         | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| --- | --- | --- | --- | --- | --- |
| first_name | varchar(20)      | YES  |     | NULL    |                |
| last_name  | varchar(20)      | YES  |     | NULL    |                |
| country    | varchar(20)      | YES  |     | NULL    |                |

+------------+------------------+------+-----+---------+----------------+

设置中的 4 行（0.01 秒）

```go

MySQL creates `.ibd` files inside the `data directory` :

```

shell> sudo ls -lhtr /usr/local/mysql/data/company

总共 256K

-rw-r----- 1 mysql mysql 128K Jun 4 07:36 customers.ibd

-rw-r----- 1 mysql mysql 128K Jun 4 08:24 payments.ibd

```go

# Cloning table structure

You can clone the structure of one table into a new table:

```

mysql> CREATE TABLE new_customers LIKE customers;

查询成功，0 行受影响（0.05 秒）

```go

You can verify the structure of the new table:

```

mysql> SHOW CREATE TABLE new_customers\G

*************************** 1\. 行 ***************************

表：new_customers

创建表：CREATE TABLE `new_customers` (

`id` int(10) unsigned NOT NULL AUTO_INCREMENT，

`first_name` varchar(20) DEFAULT NULL,

`last_name` varchar(20) DEFAULT NULL,

`country` varchar(20) DEFAULT NULL,

主键（`id`）

）ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

设置中的 1 行（0.00 秒）

```go

# See also

Refer to [`dev.mysql.com/doc/refman/8.0/en/create-table.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table.html) for many other options in `Create Table`. Partitioning a table and compressing a table will be discussed in Chapter 10, *Table Maintenance* and Chapter 11, *Managing Tablespace*, respectively.

# Inserting, updating, and deleting rows

The `INSERT`, `UPDATE`, `DELETE`, and `SELECT` operations are called **Data Manipulation Language** (**DML**) statements. `INSERT`, `UPDATE`, and `DELETE` are also called write operations, or simply **write(s)**. `SELECT` is a read operation and is simply called **read(s)**.

# How to do it...

Let's look at each of them in detail. I am sure you will enjoy learning this. I would suggest that you try a few things on your own as well, later. By the end of this recipe, we will also have gotten to grips with truncating tables.

# Inserting

The `INSERT` statement is used to create new records in a table:

```

mysql> INSERT IGNORE INTO `company`.`customers`(first_name, last_name,country)

值

（'Mike'，'Christensen'，'USA'），

（'Andy'，'Hollands'，'Australia'），

（'Ravi'，'Vedantam'，'India'），

（'Rajiv'，'Perera'，'Sri Lanka'）;

```go

Or you can explicitly mention the `id` column, if you want to insert the specific `id`:

```

mysql> INSERT IGNORE INTO `company`.`customers`(id, first_name, last_name,country)

值

（1，'Mike'，'Christensen'，'USA'），

（2，'Andy'，'Hollands'，'Australia'），

（3，'Ravi'，'Vedantam'，'India'），

（4，'Rajiv'，'Perera'，'Sri Lanka'）;

查询成功，0 行受影响，4 个警告（0.00 秒）

记录：4 重复项：4 警告：4

```go

`IGNORE`: If the row already exists and the `IGNORE` clause is given, the new data is ignored and the `INSERT` statement still succeeds in producing a warning and a number of duplicates. Otherwise, if the `IGNORE` clause is not given, the `INSERT` statement produces an error. The uniqueness of a row is identified by the primary key:

```

mysql> 显示警告;

+---------+------+---------------------------------------+

| 级别   | 代码 | 消息                               |
| --- | --- | --- |

+---------+------+---------------------------------------+

| 警告 | 1062 | 键'PRIMARY'的重复条目'1' |
| --- | --- | --- |
| 警告 | 1062 | 键'PRIMARY'的重复条目'2' |
| 警告 | 1062 | 键'PRIMARY'的重复条目'3' |
| 警告 | 1062 | 键'PRIMARY'的重复条目'4' |

+---------+------+---------------------------------------+

共 4 行（0.00 秒）

```go

# Updating

The `UPDATE` statement is used to modify the existing records in a table:

```

mysql> 更新 customers SET first_name='Rajiv'，country='UK' WHERE id=4;

查询成功，影响 1 行（0.00 秒）

匹配的行数：1  更改的行数：1  警告：0

```go

`WHERE`: This is the clause used for filtering. Whatever condition(s) are issued after the `WHERE` clause are evaluated and the filtered rows are updated.

The `WHERE` clause is **mandatory**. Failing to give it will `UPDATE` the whole table.
It is recommended to do data modification in a transaction, so that you can easily rollback the changes if you find anything wrong. You can refer to Chapter 5, *Transactions* to learn more about transactions.

# Deleting

Deleting a record can be done as follows:

```

mysql> 从 customers 中删除 WHERE id=4 AND first_name='Rajiv';

查询成功，影响 1 行（0.03 秒）

```go

The `WHERE` clause is **mandatory**. Failing to give it will `DELETE` all the rows of the table.
It is recommended to do data modification in a transaction, so that you can easily rollback the changes if you find anything wrong.

# REPLACE, INSERT, ON DUPLICATE KEY UPDATE

There are many cases where you need to handle the duplicates. The uniqueness of a row is identified by the primary key. If a row already exists, `REPLACE` simply deletes the row and inserts the new row. If a row is not there, `REPLACE` behaves as `INSERT`.

`ON DUPLICATE KEY UPDATE` is used when you want to take action if the row already exists. If you specify the `ON DUPLICATE KEY UPDATE` option and the `INSERT` statement causes a duplicate value in the `PRIMARY KEY`, MySQL performs an update to the old row based on the new values.

Suppose you want to update the previous amount whenever you get payment from the same customer and concurrently insert a new record if the customer is paying for the first time. To do this, you will define an amount column and update it whenever a new payment comes in:

```

mysql> 替换为 customers 中的值（1，'Mike'，'Christensen'，'America'）;

查询成功，影响 2 行（0.03 秒）

```go

You can see that two rows are affected, one duplicate row is deleted and a new row is inserted:

```

mysql> 插入到 payments 中的值（'Mike Christensen'，200）在重复键更新时更新 payment=payment+VALUES（payment）;

查询成功，影响 1 行（0.00 秒）

```go

```

mysql> 插入到 payments 中的值（'Ravi Vedantam'，500）在重复键更新时更新 payment=payment+VALUES（payment）;

查询成功，影响 1 行（0.01 秒）

```go

When `Mike Christensen` pays $300 next time, this will update the row and add this payment to the previous payment:

```

mysql> 插入到 payments 中的值（'Mike Christensen'，300）在重复键更新时更新 payment=payment+VALUES（payment）;

查询成功，影响 2 行（0.00 秒）

```go

`VALUES` (payment): refers to the value given in the `INSERT` statement. Payment refers to the column of the table.

# Truncating tables

Deleting the whole table takes lot of time, as MySQL performs operations row by row. The quickest way to delete all of the rows of a table (preserving the table structure) is to use the `TRUNCATE TABLE` statement.

Truncate is a DDL operation in MySQL, meaning once the data is truncated, it cannot be rolled back:

```

mysql> 清空表 customers;

查询成功，影响 0 行（0.03 秒）

```go

# Loading sample data

You have created the schema (databases and tables) and some data (through `INSERT`, `UPDATE`, and `DELETE`). To explain the further chapters, more data is needed. MySQL has provided a sample `employee` database and a lot of data to play around with. In this chapter, we will discuss how to get that data and store it in our database.

# How to do it...

1.  Download the zipped file:

```

shell> wget 'https://codeload.github.com/datacharmer/test_db/zip/master' -O master.zip

```go

2.  Unzip the file:

```

shell> 解压 master.zip

```go

3.  Load the data:

```

shell> cd test_db-master

shell> mysql -u root -p < employees.sql

mysql：[警告]在命令行界面上使用密码可能不安全。

信息

创建数据库结构

信息

存储引擎：InnoDB

信息

加载 departments

信息

加载 employees

信息

加载 dept_emp

信息

加载 dept_manager

信息

加载标题

信息

加载薪水

data_load_time_diff

NULL

```go

4.  Verify the data:

```

shell> mysql -u root -p  employees -A

mysql：[警告]在命令行界面上使用密码可能不安全。

欢迎来到 MySQL 监视器。命令以;或\g 结束。

您的 MySQL 连接 ID 为 35

服务器版本：8.0.3-rc-log MySQL 社区服务器（GPL）

版权所有（c）2000 年，2017 年，Oracle 及/或其关联公司。保留所有权利。

Oracle 是 Oracle Corporation 和/或其

关联公司。其他名称可能是其各自的商标

所有者。

类型'help;'或'\h'获取帮助。键入'\c'以清除当前输入语句。

```go

```

mysql> 显示表;

+-------------------------+

| employees 表     |
| --- |

+-------------------------+

| current_dept_emp        |
| --- |
| 部门             |
| dept_emp                |
| dept_emp_latest_date    |
| dept_manager            |
| employees               |
| 薪水                |
| 标题                  |

+-------------------------+

共 8 行（0.00 秒）

```go

```

mysql> DESC employees\G

*************************** 1\. 行 ***************************

字段：员工编号

类型：int（11）

空值：否

关键：PRI

默认值：NULL

额外：

*************************** 2\. 行 ***************************

字段：出生日期

类型：日期

空值：否

关键：

默认值：NULL

额外：

*************************** 3\. 行 ***************************

字段：名

类型：varchar（14）

空值：否

关键：

默认值：NULL

额外：

*************************** 4\. 行 ***************************

字段：姓

类型：varchar（16）

空值：否

关键：

默认值：NULL

额外：

*************************** 5\. 行 ***************************

字段：性别

类型：enum（'M'，'F'）

空值：否

关键：

默认值：NULL

额外：

*************************** 6\. 行 ***************************

字段：雇佣日期

类型：日期

空值：否

关键：

默认值：NULL

额外：

共 6 行（0.00 秒）

```go

# Selecting data

You have inserted and updated data in the tables. Now it is time to learn how to retrieve information from the database. In this section, we will discuss how to retrieve data from the sample `employee` database that we have created.

There are many things that you can do with `SELECT`. The most common use cases will be discussed in this section. For more details on syntax and other use cases, refer to [`dev.mysql.com/doc/refman/8.0/en/select.html`](https://dev.mysql.com/doc/refman/8.0/en/select.html).

# How to do it...

Select all data from the `departments` table of the `employee` database. You can use an asterisk (`*`) to select all columns from a table. It is not recommended to use it, you should always select only the data you need:

```

mysql> 从 departments 中选择*;

+---------+--------------------+

| dept_no | 部门名称          |
| --- | --- |

+---------+--------------------+

| d009    | 客户服务   |
| --- | --- |
| d005    | 开发        |
| d002    | 财务            |
| d003    | 人力资源    |
| d001    | 营销          |
| d004    | 生产         |
| d006    | 质量管理 |
| d008    | 研究           |
| d007    | 销售              |

+---------+--------------------+

9 行设置（0.00 秒）

```go

# Selecting columns

Suppose you need `emp_no` and `dept_no` from `dept_manager`:

```

mysql> 选择 emp_no, dept_no FROM dept_manager;

+--------+---------+

| emp_no | dept_no |
| --- | --- |

+--------+---------+

| 110022 | d001    |
| --- | --- |
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

24 行设置（0.00 秒）

```go

# Count

Find the count of employees from the `employees` table:

```

mysql> 从员工中选择 COUNT(*) ;

+----------+

| COUNT(*) |
| --- |

+----------+

|   300024 |
| --- |

+----------+

1 行设置（0.03 秒）

```go

# Filter based on condition

Find `emp_no` of employees with `first_name` as `Georgi` and `last_name` as `Facello`:

```

mysql> 从员工中选择 emp_no WHERE first_name='Georgi' AND last_name='Facello';

+--------+

| emp_no |
| --- |

+--------+

|  10001 |
| --- |
|  55649 |

+--------+

2 行设置（0.08 秒）

```go

All the filtering conditions are given through the `WHERE` clause. Except integers and floating points, everything else should be put inside quotes.

# Operators

MySQL supports many operators for filtering results. Refer to [`dev.mysql.com/doc/refman/8.0/en/comparison-operators.html`](https://dev.mysql.com/doc/refman/8.0/en/comparison-operators.html) for a list of all the operators. We will discuss a few operators here. `LIKE` and `RLIKE` are explained in detail in the next examples:

*   **Equality**: Refer to the preceding example where you have filtered using `=`.
*   `IN`: Check whether a value is within a set of values.
    For example, find the count of all employees whose last name is either `Christ`, `Lamba`, or `Baba`:

```

mysql> 从员工中选择 COUNT(*) WHERE last_name IN ('Christ', 'Lamba', 'Baba');

+----------+

| COUNT(*) |
| --- |

+----------+

|      626 |
| --- |

+----------+

1 行设置（0.08 秒）

```go

*   `BETWEEN...AND`: Check whether a value is within a range of values.
    For example, find the number of employees who were hired in December 1986:

```

mysql> 从员工中选择 COUNT(*) WHERE hire_date BETWEEN '1986-12-01' AND '1986-12-31';

+----------+

| COUNT(*) |
| --- |

+----------+

|     3081 |
| --- |

+----------+

1 行设置（0.06 秒）

```go

*   `NOT`: You can simply negate the results by preceding with the `NOT` operator.
    For example, find the number of employees who were `NOT` hired in December 1986:

```

mysql> 从员工中选择 COUNT(*) WHERE hire_date NOT BETWEEN '1986-12-01' AND '1986-12-31';

+----------+

| COUNT(*) |
| --- |

+----------+

|   296943 |
| --- |

+----------+

1 行设置（0.08 秒）

```go

# Simple pattern matching

You can use the `LIKE` operator. Use underscore (`_`) for matching exactly one character. Use `%` for matching any number of characters.

*   Find the count of all employees whose first name starts with `Christ`:

```

mysql> 从员工中选择 COUNT(*) WHERE first_name LIKE 'christ%';

+----------+

| COUNT(*) |
| --- |

+----------+

|     1157 |
| --- |

+----------+

1 行设置（0.06 秒）

```go

*   Find the count of all employees whose first name starts with `Christ` and ends with `ed`:

```

mysql> 从员工中选择 COUNT(*) WHERE first_name LIKE 'christ%ed';

+----------+

| COUNT(*) |
| --- |

+----------+

|      228 |
| --- |

+----------+

1 行设置（0.06 秒）

```go

*   Find the count of all employees whose first name contains `sri`:

```

mysql> 从员工中选择 COUNT(*) WHERE first_name LIKE '%sri%';

+----------+

| COUNT(*) |
| --- |

+----------+

|      253 |
| --- |

+----------+

1 行设置（0.08 秒）

```go

*   Find the count of all employees whose first name ends with `er`:

```

mysql> 从员工中选择 COUNT(*) WHERE first_name LIKE '%er';

+----------+

| COUNT(*) |
| --- |

+----------+

|     5388 |
| --- |

+----------+

1 行设置（0.08 秒）

```go

*   Find the count of all employees whose first name starts with any two characters followed by `ka` and then followed by any number of characters:

```

mysql> 从员工中选择 COUNT(*) WHERE first_name LIKE '__ka%';

+----------+

| COUNT(*) |
| --- |

+----------+

|     1918 |
| --- |

+----------+

1 行设置（0.06 秒）

```go

# Regular expressions

You can use regular expressions in the `WHERE` clause by using the `RLIKE` or `REGEXP` operators. There are many ways to use `REGEXP`, refer to [`dev.mysql.com/doc/refman/8.0/en/regexp.html`](https://dev.mysql.com/doc/refman/8.0/en/regexp.html) for more examples:

| **Expression** | **Description** |
| `*` | Zero or more repetitions |
| `+` | One or more repetitions |
| `?` | Optional character |
| `.` | Any character |
| `\.` | Period |
| `^` | Starts with |
| `$` | Ends with |
| `[abc]` | Only *a*, *b*, or *c* |
| `[^abc]` | Neither *a*, *b*, nor *c* |
| `[a-z]` | Characters a to *z* |
| `[0-9]` | Numbers 0 to 9 |
| `^...$` | Starts and ends |
| `\d` | Any digit |
| `\D` | Any non-digit character |
| `\s` | Any whitespace |
| `\S` | Any non-whitespace character |
| `\w` | Any alphanumeric character |
| `\W` | Any non-alphanumeric character |
| `{m}` | *m* repetitions |
| `{m,n}` | *m* to *n* repetitions |

*   Find the count of all employees whose first name starts with `Christ`:

```

mysql> 从员工中选择 COUNT(*) WHERE first_name RLIKE '^christ';

+----------+

| COUNT(*) |
| --- |

+----------+

|     1157 |
| --- |

+----------+

1 行设置（0.18 秒）

```go

*   Find the count of all employees whose last name ends with `ba`:

```

mysql> 从员工中选择 COUNT(*) WHERE last_name REGEXP 'ba$';

+----------+

| COUNT(*) |
| --- |

+----------+

|     1008 |
| --- |

+----------+

1 行设置（0.15 秒）

```go

*   Find the count of all employees whose last name does not contain vowels (a, e, i, o, or u):

```

mysql> 从员工中选择 COUNT(*) WHERE last_name NOT REGEXP '[aeiou]';

+----------+

| COUNT(*) |
| --- |

+----------+

|      148 |
| --- |

+----------+

1 行设置（0.11 秒）

```go

# Limiting results

Select the names of any 10 employees whose `hire_date` is before 1986\. You can get this by using the `LIMIT` clause at the end of the statement:

```

mysql> 选择 first_name, last_name FROM employees WHERE hire_date < '1986-01-01' LIMIT 10;

+------------+------------+

| first_name | last_name  |
| --- | --- |

+------------+------------+

| Bezalel    | Simmel     |
| --- | --- |
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

10 行设置（0.00 秒）

```go

# Using the table alias

By default, whatever column you have given in the `SELECT` clause will appear in the results. In the previous examples, you have found out the count, but it is displayed as `COUNT(*)`. You can change it by using the `AS` alias:

```

mysql> 从员工中选择 COUNT(*) AS count WHERE hire_date BETWEEN '1986-12-01' AND '1986-12-31';

+-------+

| count |
| --- |

+-------+

|  3081 |
| --- |

+-------+

1 行设置（0.06 秒）

```go

# Sorting results

You can order the result based on the column or aliased column. You can be specify `DESC` for descending order or `ASC` for ascending. By default, ordering will be ascending. You can combine the  `LIMIT` clause with `ORDER BY` to limit the results.

# How to do it...

Find the employee IDs of the first five top-paid employees.

```

mysql> 选择 emp_no,salary FROM salaries ORDER BY salary DESC LIMIT 5;

+--------+--------+

| emp_no | salary |
| --- | --- |

+--------+--------+

|  43624 | 158220 |
| --- | --- |
|  43624 | 157821 |
| 254466 | 156286 |
|  47978 | 155709 |
| 253939 | 155513 |

+--------+--------+

5 行设置（0.74 秒）

```go

Instead of specifying the column name, you can also mention the position of the column in the `SELECT` statement. For example, you are selecting the salary at the second position in the `SELECT` statement. So, you can specify `ORDER BY 2`:

```

mysql> SELECT emp_no,salary FROM salaries ORDER BY 2 DESC LIMIT 5;

+--------+--------+

| emp_no | salary |
| --- | --- |

+--------+--------+

|  43624 | 158220 |
| --- | --- |
|  43624 | 157821 |
| 254466 | 156286 |
|  47978 | 155709 |
| 253939 | 155513 |

+--------+--------+

5 rows in set (0.78 sec)

```go

# Grouping results (aggregate functions)

You can group the results using the `GROUP BY` clause on a column and then use `AGGREGATE` functions, such as `COUNT`, `MAX`, `MIN`, and `AVERAGE`. You can also use the function on a column in a group by clause. See the `SUM` example where you will use the `YEAR()` function.

# How to do it...

Each of the previously-mentioned aggregate functions will be introduced to you here in detail.

# COUNT

1.  Find the count of male and female employees:

```

mysql> SELECT gender, COUNT(*) AS count FROM employees GROUP BY gender;

+--------+--------+

| gender | count  |
| --- | --- |

+--------+--------+

| M      | 179973 |
| --- | --- |
| F      | 120051 |

+--------+--------+

2 rows in set (0.14 sec)

```go

2.  You want to find the 10 most common first names of the employees. You can use `GROUP BY first_name` to group all the first names, then `COUNT(first_name)` to find the count inside the group, and finally the `ORDER BY` count to sort the results. `LIMIT` these results to the top 10:

```

mysql> SELECT first_name, COUNT(first_name) AS count FROM employees GROUP BY first_name ORDER BY count DESC LIMIT 10;

+-------------+-------+

| first_name  | count |
| --- | --- |

+-------------+-------+

| Shahab      |   295 |
| --- | --- |
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

```go

# SUM

Find the sum of the salaries given to employees in each year and sort the results by salary. The `YEAR()` function returns the `YEAR` of the given date:

```

mysql> SELECT '2017-06-12', YEAR('2017-06-12');

+------------+--------------------+

| 2017-06-12 | YEAR('2017-06-12') |
| --- | --- |

+------------+--------------------+

| 2017-06-12 |               2017 |
| --- | --- |

+------------+--------------------+

1 row in set (0.00 sec)

```go

```

mysql>  SELECT YEAR(from_date), SUM(salary) AS sum FROM salaries GROUP BY YEAR(from_date) ORDER BY sum DESC;

+-----------------+-------------+

| YEAR(from_date) | sum         |
| --- | --- |

+-----------------+-------------+

|            2000 | 17535667603 |
| --- | --- |
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

```go

# AVERAGE

Find the 10 employees with the highest average salaries:

```

mysql>  SELECT emp_no, AVG(salary) AS avg FROM salaries GROUP BY emp_no ORDER BY avg DESC LIMIT 10;

+--------+-------------+

| emp_no | avg         |
| --- | --- |

+--------+-------------+

| 109334 | 141835.3333 |
| --- | --- |
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

```go

# DISTINCT

You can use the `DISTINCT` clause to filter the distinct entries in a table:

```

mysql> SELECT DISTINCT title FROM titles;

+--------------------+

| title              |
| --- |

+--------------------+

| 高级工程师    |
| --- |
| 员工              |
| 工程师           |
| 高级员工       |
| 助理工程师 |
| 技术领袖   |
| 经理            |

+--------------------+

7 rows in set (0.30 sec)

```go

# Filtering using HAVING

You can filter results of the `GROUP BY` clause by adding the `HAVING` clause.

For example, find the employees with an average salary of more than 140,000:

```

mysql>  SELECT emp_no, AVG(salary) AS avg FROM salaries GROUP BY emp_no HAVING avg > 140000 ORDER BY avg DESC;

+--------+-------------+

| emp_no | avg         |
| --- | --- |

+--------+-------------+

| 109334 | 141835.3333 |
| --- | --- |
| 205000 | 141064.6364 |

+--------+-------------+

2 rows in set (0.80 sec)

```go

# See also

There are many other aggregate functions, refer to [`dev.mysql.com/doc/refman/8.0/en/group-by-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/group-by-functions.html) for more information.

# Creating users

So far, you have used only the root user to connect to MySQL and execute statements. The root user should never be used while accessing MySQL, except for administrative tasks from `localhost`. You should create users, restrict the access, restrict the resource usage, and so on. For creating new users, you should have the `CREATE USER` privilege that will be discussed in the next section. During the initial set up, you can use the root user to create other users.

# How to do it...

Connect to mysql using the root user and execute `CREATE USER` command to create new users.

```

mysql> CREATE USER IF NOT EXISTS 'company_read_only'@'localhost'

IDENTIFIED WITH mysql_native_password

BY 'company_pass'

WITH MAX_QUERIES_PER_HOUR 500

MAX_UPDATES_PER_HOUR 100;

```go

You might get the following error if the password is not strong.

```

ERROR 1819 (HY000): Your password does not satisfy the current policy requirements

```go

The preceding statement will create users with:

*   `* Username`: `company_read_only`.
*   `* access only from`:  `localhost`.
*   You can restrict the access to the IP range. For example: `10.148.%.%`. By giving `%`, the user can access from any host.
*   `* password`: `company_pass`.
*   `* using mysql_native_password` (default) authentication.
*   You can also specify any pluggable authentication, such as `sha256_password`, `LDAP`, or Kerberos.
*   The `* maximum number of queries` the user can execute in an hour is 500.
*   The `* maximum number of updates` the user can execute in an hour is 100.

When a client connects to the MySQL server, it undergoes two stages:

1.  Access control—connection verification
2.  Access control—request verification

During the connection verification, the server identifies the connection by the username and the hostname from which it is connected. The server invokes the authentication plugin for the user and verifies the password. It also checks whether the user is locked or not.

During the request verification stage, the server checks whether the user has sufficient privileges for each operation.

In the preceding statement, you have to give the password in clear text, which can be recorded in the command history file, `$HOME/.mysql_history`. To avoid that, you can compute the hash on your local server and directly specify the hashed string. The syntax for it is the same, except `mysql_native_password BY 'company_pass'` changes to `mysql_native_password AS 'hashed_string'`:

```

mysql> SELECT PASSWORD('company_pass');

+-------------------------------------------+

|PASSWORD('company_pass')                   |

+-------------------------------------------+

| *EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18 |
| --- |

+-------------------------------------------+

1 row in set, 1 warning (0.00 sec)

```go

```

mysql> CREATE USER IF NOT EXISTS 'company_read_only'@'localhost'

IDENTIFIED WITH mysql_native_password

```go

```

AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18'

WITH MAX_QUERIES_PER_HOUR 500

MAX_UPDATES_PER_HOUR 100;

```go

You can directly create users by granting privileges. Refer to the next section on how to grant privileges. However, MySQL will deprecate this feature in the next release.

# See also

Refer to [`dev.mysql.com/doc/refman/8.0/en/create-user.html`](https://dev.mysql.com/doc/refman/8.0/en/create-user.html) for more options on creating users. More secure options, such as SSL, using other authentication methods will be discussed in Chapter 14, *Security*.

# Granting and revoking access to users

You can restrict the user to access specific databases or tables and also only specific operations, such as `SELECT`, `INSERT`, and `UPDATE`. For granting privileges to other users, you should have the `GRANT` privilege. 

# How to do it...

During the initial setup, you can use the root user to grant privileges. You can also create an administrative account to manage the users.

# Granting privileges

*   Grant the `READ ONLY(SELECT)` privileges to the `company_read_only` user:

```

mysql> GRANT SELECT ON company.* TO 'company_read_only'@'localhost';

Query OK, 0 rows affected (0.06 sec)

```go

The asterisk (`*`) represents all tables inside the database.

*   Grant the `INSERT` privilege to the new `company_insert_only` user:

```

mysql> GRANT INSERT ON company.* TO 'company_insert_only'@'localhost' IDENTIFIED BY 'xxxx';

Query OK, 0 rows affected, 1 warning (0.05 sec)

```go

```

mysql> SHOW WARNINGS\G

*************************** 1\. 行 ***************************

Level: Warning

Code: 1287

Message: Using GRANT for creating new user is deprecated and will be removed in future release. Create new user with CREATE USER statement.

1 row in set (0.00 sec)

```go

*   Grant the `WRITE` privileges to the new `company_write` user:

```

mysql> GRANT INSERT, DELETE, UPDATE ON company.* TO 'company_write'@'%' IDENTIFIED WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';

Query OK, 0 rows affected, 1 warning (0.04 sec)

```go

*   Restrict to a specific table. Restrict the `employees_read_only` user to `SELECT` only from the `employees` table:

```

mysql> GRANT SELECT ON employees.employees TO 'employees_read_only'@'%' IDENTIFIED WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';

Query OK, 0 rows affected, 1 warning (0.03 sec)

```go

*   You can further restrict to specific columns. Restrict the `employees_ro` user to the `first_name` and `last_name` columns of the `employees` table:

```

mysql> GRANT SELECT(first_name,last_name)  ON employees.employees TO 'employees_ro'@'%' IDENTIFIED WITH mysql_native_password AS '*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18';

Query OK, 0 rows affected, 1 warning (0.06 sec)

```go

*   Extending grants. You can extend the grants by executing the new grant. Extend the privilege to the `employees_col_ro` user to access the salary of the `salaries` table:

```

mysql> GRANT SELECT(salary) ON employees.salaries TO 'employees_ro'@'%';

Query OK, 0 rows affected (0.00 sec)

```go

*   Creating the `SUPER` user. You need an administrative account to manage the server. `ALL` signifies all privileges expect the `GRANT` privilege:

```

mysql> CREATE USER 'dbadmin'@'%' IDENTIFIED WITH mysql_native_password BY 'DB@dm1n';

Query OK, 0 rows affected (0.01 sec)

```go

```

mysql> GRANT ALL ON *.* TO 'dbadmin'@'%';

Query OK, 0 rows affected (0.01 sec)

```go

*   Granting the `GRANT` privilege. The user should have the `GRANT OPTION` privilege to grant privileges to other users. You can extend the `GRANT` privilege to the `dbadmin` super user:

```

mysql> GRANT GRANT OPTION ON *.* TO 'dbadmin'@'%';

Query OK, 0 rows affected (0.03 sec)

```go

Refer to [`dev.mysql.com/doc/refman/8.0/en/grant.html`](https://dev.mysql.com/doc/refman/8.0/en/grant.html) for more privilege types.

# Checking grants

You can check all the user's grants. Check grants for the `employee_col_ro` user:

```

mysql> SHOW GRANTS FOR 'employees_ro'@'%'\G

*************************** 1\. 行 ***************************

Grants for employees_ro@%: GRANT USAGE ON *.* TO `employees_ro`@`%`

*************************** 2\. 行 ***************************

Grants for employees_ro@%: GRANT SELECT (`first_name`, `last_name`) ON `employees`.`employees` TO `employees_ro`@`%`

*************************** 3\. 行 ***************************

Grants for employees_ro@%: GRANT SELECT (`salary`) ON `employees`.`salaries` TO `employees_ro`@`%`

```go

Check grants for the `dbadmin` user. You can see all the grants that are available to the `dbadmin` user:

```

mysql> SHOW GRANTS FOR 'dbadmin'@'%'\G

*************************** 1\. 行 ***************************

Grants for dbadmin@%: GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD, SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES, SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE, ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE, CREATE ROLE, DROP ROLE ON *.* TO `dbadmin`@`%` WITH GRANT OPTION

*************************** 2\. 行 ***************************

Grants for dbadmin@%: GRANT BINLOG_ADMIN,CONNECTION_ADMIN,ENCRYPTION_KEY_ADMIN,GROUP_REPLICATION_ADMIN,REPLICATION_SLAVE_ADMIN,ROLE_ADMIN,SET_USER_ID,SYSTEM_VARIABLES_ADMIN ON *.* TO `dbadmin`@`%`

2 rows in set (0.00 sec)

```go

# Revoking grants

Revoking grants has the same syntax as creating grants. You grant a privilege `TO` the user and revoke a privilege `FROM` the user.

*   Revoke the `DELETE` access from the `'company_write'@'%'` user:

```

mysql> REVOKE DELETE ON company.* FROM 'company_write'@'%';

Query OK, 0 rows affected (0.04 sec)

```go

*   Revoke the access to the salary column from the `employee_ro` user:

```

mysql> REVOKE SELECT(salary) ON employees.salaries FROM 'employees_ro'@'%';

Query OK, 0 rows affected (0.03 sec)

```go

# Modifying the mysql.user table

All the user information, along with privileges, is stored in the `mysql.user` table. If you have the privilege to access the `mysql.user` table, you can directly modify the `mysql.user` table to create users and grant privileges.

If you modify the grant tables indirectly, using account-management statements such as `GRANT`, `REVOKE`, `SET PASSWORD`, or `RENAME USER`, the server notices these changes and loads the grant tables into memory again immediately.

If you modify the grant tables directly, using statements such as `INSERT`, `UPDATE`, or `DELETE`, your changes have no effect on privilege checking until you either restart the server or tell it to reload the tables. If you change the grant tables directly but forget to reload them, your changes have no effect until you restart the server.

The reloading of the `GRANT` tables can be done by issuing a `FLUSH PRIVILEGES` statement.

Query the `mysql.user` table to find out all the entries for the `dbadmin` user:

```

mysql> SELECT * FROM mysql.user WHERE user='dbadmin'\G

*************************** 1\. 行 ***************************

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

密码上次更改：2017-06-10 16:24:03

密码生命周期：NULL

账户锁定：N

Create_role_priv：Y

Drop_role_priv：Y

1 行（0.00 秒）

```go

You can see that the `dbadmin` user can access the database from any host (%). You can restrict them to `localhost` just by updating the `mysql.user` table and reloading the grant tables:

```

mysql> UPDATE mysql.user SET host='localhost' WHERE user='dbadmin';

查询成功，1 行受影响（0.02 秒）

匹配行数：1  更改行数：1  警告：0

```go

```

mysql> 刷新权限；

查询成功，0 行受影响（0.00 秒）

```go

# Setting password expiry for users

You can expire the passwords of users for a specific interval; after this, they need to change their password.

When an application developer asks for database access, you can create the account with a default password and then set it to expire. You can share the password with the developers, then they have to change the password to continue using MySQL.

All the accounts are created with a password expiry equal to the `default_password_lifetime` variable, which is disabled by default:

*   Create a user with an expired password. When the developer logs in for the first time and tries to execute any statement, `ERROR 1820 (HY000):` is thrown. The password must be reset using the `ALTER USER` statement before executing this statement:

```

mysql> 创建用户'developer'@'%'，用 mysql_native_password 标识为'*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18'密码过期；

查询成功，0 行受影响（0.04 秒）

```go

```

shell> mysql -u developer -pcompany_pass

mysql：[警告]在命令行界面上使用密码可能不安全。

欢迎使用 MySQL 监视器。命令以;或\g 结束。

您的 MySQL 连接 ID 为 31

服务器版本：8.0.3-rc-log

版权所有（c）2000, 2017, Oracle 及/或其附属公司。保留所有权利。

Oracle 是 Oracle Corporation 和/或其

联盟。其他名称可能是其各自商标

所有者。

键入'help;'或'\h'获取帮助。键入'\c'清除当前输入语句。

mysql> 显示数据库；

错误 1820（HY000）：在执行此语句之前，必须使用 ALTER USER 语句重置密码。

```go

The developer has to change their password using the following command:

```

mysql> ALTER USER 'developer'@'%' 用 mysql_native_password 标识为'new_company_pass'更改

查询成功，0 行受影响（0.03 秒）

```go

*   Manually expire the existing user:

```

mysql> ALTER USER 'developer'@'%' 密码过期;

查询成功，0 行受影响（0.06 秒）

```go

*   Require the password to be changed every 180 days:

```

mysql> ALTER USER 'developer'@'%' 密码在 90 天后过期；

查询成功，0 行受影响（0.04 秒）

```go

# Locking users

If you find any issues with the account, you can lock it. MySQL supports locking while using `CREATE USER` or `ALTER USER`.

Lock the account by adding the `ACCOUNT LOCK` clause to the `ALTER USER` statement:

```

mysql> ALTER USER 'developer'@'%' 账户锁定;

查询成功，0 行受影响（0.05 秒）

```go

The developer will get an error saying that the account is locked:

```

shell> mysql -u developer -pnew_company_pass

mysql：[警告]在命令行界面上使用密码可能不安全。

错误 3118（HY000）：拒绝用户'developer'@'localhost'访问。账户已锁定。

```go

You can unlock the account after confirming:

```

mysql> ALTER USER 'developer'@'%' 账户解锁；

查询成功，0 行受影响（0.00 秒）

```go

# Creating roles for users

A MySQL role is a named collection of privileges. Like user accounts, roles can have privileges granted to and revoked from them. A user account can be granted roles, which grants to the account the role privileges. Earlier, you created separate users for reads, writes, and administration. For write privilege, you have granted `INSERT`, `DELETE`, and `UPDATE` to the user. Instead, you can grant those privileges to a role and then assign the user to that role. By this way, you can avoid granting privileges individually to possibly many user accounts. 

*   Creating roles:

```

mysql> 创建角色'app_read_only'，'app_writes'，'app_developer'；

查询成功，0 行受影响（0.01 秒）

```go

*   Assigning privileges to the roles using the `GRANT` statement:

```

mysql> GRANT SELECT ON employees.* TO 'app_read_only';

查询成功，0 行受影响（0.00 秒）

```go

```

mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO 'app_writes';

查询成功，0 行受影响（0.00 秒）

```go

```

mysql> GRANT ALL ON employees.* TO 'app_developer';

查询成功，0 行受影响（0.04 秒）

```go

*   Creating users. If you do not specify any host, `%` will be taken:

```

mysql> 创建用户 emp_read_only，密码为'emp_pass'；

查询成功，0 行受影响（0.06 秒）

```go

```

mysql> 创建用户 emp_writes，密码为'emp_pass'；

查询成功，0 行受影响（0.04 秒）

```go

```

mysql> 创建用户 emp_developer，密码为'emp_pass'；

查询成功，0 行受影响（0.01 秒）

```go

```

mysql> 创建用户 emp_read_write，密码为'emp_pass'；

查询成功，0 行受影响（0.00 秒）

```go

*   Assigning the roles to the users using the `GRANT` statement. You can assign multiple roles to a user.
    For example, you can assign both read and write access to the `emp_read_write` user:

```

mysql> GRANT 'app_read_only' TO 'emp_read_only'@'%';

查询成功，0 行受影响（0.04 秒）

```go

```

mysql> GRANT 'app_writes' TO 'emp_writes'@'%';

查询成功，0 行受影响（0.00 秒）

```go

```

mysql> GRANT 'app_developer' TO 'emp_developer'@'%';

查询成功，0 行受影响（0.00 秒）

```go

```

mysql> GRANT 'app_read_only', 'app_writes' TO 'emp_read_write'@'%';

查询成功，0 行受影响（0.05 秒）

```go

As a security measure, avoid using `%` and restrict the access to the IP where the application is deployed.

# Selecting data into a file and table

You can save the output into a file using the `SELECT INTO OUTFILE` statement.

You can specify the column and line delimiters, and later you can import the data into other data platforms.

# How to do it...

You can save the output destination as a file or a table.

# Saving as a file

*   To save the output into a file, you need the `FILE` privilege. `FILE` is a global privilege, which means you cannot restrict it for a particular database. However, you can restrict what the user selects:

```

mysql> GRANT SELECT ON employees.* TO 'user_ro_file'@'%' 用 mysql_native_password 标识为'*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18'；

查询成功，0 行受影响，1 个警告（0.00 秒）

mysql> GRANT FILE ON *.* TO 'user_ro_file'@'%' 用 mysql_native_password 标识为'*EBD9E3BFD1489CA1EB0D2B4F29F6665F321E8C18'；

查询成功，0 行受影响，1 个警告（0.00 秒）

```go

*   On Ubuntu, by default, MySQL will not allow you to write to file. You should set `secure_file_priv` in the config file and restart MySQL. You will learn more on config in Chapter 4, *Configuring MySQL*. On CentOS, Red Hat, `secure_file_priv` is set to `/var/lib/mysql-files`, which means all the files will be saved in that directory.
*   For now, enable like this. Open the config file and add `secure_file_priv = /var/lib/mysql`:

```

shell> sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

```go

*   Restart the MySQL server:

```

shell> sudo systemctl restart mysql

```go

The following statement will save the output into a CSV format:

```

mysql> 选择 first_name, last_name INTO OUTFILE 'result.csv'

字段由','分隔，可选地由'"'括起来

行由'\n'终止

从雇员中选择雇佣日期<'1986-01-01'的 10 行；

查询成功，10 行受影响（0.00 秒）

```go

You can check the output of the file, which will be created in the path specified by `{secure_file_priv}/{database_name}`, it is `/var/lib/mysql/employees/` in this case. The statement will fail if the file already exists, so you need to give a unique name every time you execute or move the file to a different location:

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

```go

# Saving as a table

You can save the results of a `SELECT` statement into a table. Even if the table does not exist, you can use `CREATE` and `SELECT` to create the table and load the data. If the table already exists, you can use `INSERT` and `SELECT` to load the data.

You can save the titles into a new `titles_only` table:

```

mysql> 创建表 titles_only 作为选择不同的 titles;

查询成功，影响了 7 行（0.50 秒）

记录：7 重复：0 警告：0

```go

If the table already exists, you can use the `INSERT INTO SELECT` statement:

```

mysql> 插入 titles_only 选择不同的 titles;

查询成功，影响了 7 行（0.46 秒）

记录：7 重复：0 警告：0

```go

To avoid duplicates, you can use `INSERT IGNORE`. However, in this case, there is no `PRIMARY KEY` on the `titles_only` table. So the `IGNORE` clause does not make any difference.

# Loading data into a table

The way you can dump a table data into a file, you can do vice-versa, that is, load the data from the file into a table. This is widely used for loading bulk data and is a super fast way to load data into tables. You can specify the column delimiters to load the data into respective columns. You should have the `FILE` privilege and the `INSERT` privilege on the table.

# How to do it...

Earlier, you have saved `first_name` and `last_name` to a file. You can use the same file to load the data into another table. Before loading, you should create the table. If the table already exists, you can directly load. The columns of the table should match the fields of the file.

Create a table to hold the data:

```

mysql> 创建表 employee_names (

`first_name` varchar(14) NOT NULL，

`last_name` varchar(16) NOT NULL

）ENGINE=InnoDB;

查询成功，影响了 0 行（0.07 秒）

```go

Make sure that the file is present:

```

shell> sudo ls -lhtr /var/lib/mysql/employees/result.csv

-rw-rw-rw- 1 mysql mysql 180 Jun 10 14:53 /var/lib/mysql/employees/result.csv

```go

Load the data using the `LOAD DATA INFILE` statement:

```

mysql> LOAD DATA INFILE 'result.csv' INTO TABLE employee_names

字段以','分隔

OPTIONALLY ENCLOSED BY '"'

行终止符为'\n';

查询成功，影响了 10 行（0.01 秒）

记录：10 删除：0 跳过：0 警告：0

```go

The file can be given as a full path name to specify its exact location. If given as a relative path name, the name is interpreted relative to the directory in which the client program was started.

*   If the file contains any headers you want to ignore, specify `IGNORE n LINES`:

```

mysql> LOAD DATA INFILE 'result.csv' INTO TABLE employee_names

字段以','分隔

OPTIONALLY ENCLOSED BY '"'

行终止符为'\n'

忽略 1 行;

```go

*   You can specify `REPLACE` or `IGNORE` to deal with duplicates:

```

mysql> LOAD DATA INFILE 'result.csv' REPLACE INTO TABLE employee_names FIELDS TERMINATED BY ','OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n';

查询成功，影响了 10 行（0.01 秒）

记录：10 删除：0 跳过：0 警告：0

mysql> LOAD DATA INFILE 'result.csv' IGNORE INTO TABLE employee_names FIELDS TERMINATED BY ','OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n';

查询成功，影响了 10 行（0.06 秒）

记录：10 删除：0 跳过：0 警告：0

```go

*   MySQL assumes that the file you want to load is available on the server. If you are connected to the server from a remote client machine, you can specify `LOCAL` to load the file located on the client. The local file will be copied from the client to the server. The file is saved in the standard temporary location of the server. In Linux machines, it is `/tmp`:

```

mysql> LOAD DATA LOCAL INFILE 'result.csv' IGNORE INTO TABLE employee_names FIELDS TERMINATED BY ','OPTIONALLY ENCLOSED BY '"' LINES TERMINATED BY '\n';

```go

# Joining tables

So far you have looked at inserting and retrieving data from a single table. In this section, we will discuss how to combine two or more tables to retrieve the results. 

A perfect example is that you want to find the employee name and department number of a employee with `emp_no: 110022`:

*   The department number and name are stored in the `departments` table
*   The employee number and other details, such as `first_name` and `last_name`, are stored in the `employees` table
*   The mapping of employee and department is stored in the `dept_manager` table

If you do not want to use `JOIN`, you can do this:

1.  Find the employee name with `emp_no` as `110022` from the `employee` table:

```

mysql> 选择 emp.emp_no, emp.first_name, emp.last_name

从 employees 作为 emp

其中 emp.emp_no=110022;

+--------+------------+------------+

| 员工编号 | 名字 | 姓  |
| --- | --- | --- |

+--------+------------+------------+

| 110022 | Margareta  | Markovitch |
| --- | --- | --- |

+--------+------------+------------+

设置中的 1 行（0.00 秒）

```go

2.  Find the department number from the `departments` table:

```

mysql> 选择 dept_no 从 dept_manager 作为 dept_mgr，其中 dept_mgr.emp_no=110022;

+---------+

| 部门编号 |
| --- |

+---------+

| d001    |
| --- |

+---------+

设置中的 1 行（0.00 秒）

```go

3.  Find the department name from the `departments` table:

```

mysql> 选择部门名称从部门中的部门，其中 dept.dept_no='d001';

+-----------+

| 部门名称 |
| --- |

+-----------+

| 市场 |
| --- |

+-----------+

设置中的 1 行（0.00 秒）

```go

# How to do it...

To avoid look ups on three different tables using three statements, you can use `JOIN` to club them. The important thing to note here is to join two tables, you should have one, or more, common column to join. You can join employees and `dept_manager` based on `emp_no`, they both have the `emp_no` column. Though the names don't need to match, you should figure out the column on which you can join. Similarly, `dept_mgr` and `departments` have `dept_no` as a common column.

Like a column alias, you can give table an alias and refer columns of that table using an alias. For example, you can give employees an alias using `FROM employees AS emp` and refer columns of the `employees` table using dot notation, such as `emp.emp_no`:

```

mysql> 选择

emp.emp_no，

emp.first_name，

emp.last_name，

dept.dept_name

从

员工作为 emp

加入 dept_manager 作为 dept_mgr

ON emp.emp_no=dept_mgr.emp_no AND emp.emp_no=110022

加入部门作为 dept

ON dept_mgr.dept_no=dept.dept_no;

+--------+------------+------------+-----------+

| 员工编号 | 名字 | 姓  | 部门名称 |
| --- | --- | --- | --- |

+--------+------------+------------+-----------+

| 110022 | Margareta  | Markovitch | 市场 |
| --- | --- | --- | --- |

+--------+------------+------------+-----------+

设置中的 1 行（0.00 秒）

```go

Let's look at another example—you want to find out the average salary for each department. For this you can use the `AVG` function and group by `dept_no`. To find out the department name, you can join the results with the `departments` table on `dept_no`:

```

mysql> 选择

部门名称，

平均工资作为 avg_salary

从

薪水

加入 dept_emp

ON salaries.emp_no=dept_emp.emp_no

加入部门

ON dept_emp.dept_no=departments.dept_no

按组

dept_emp.dept_no

按照

平均工资

DESC;

+--------------------+------------+

| 部门名称 | 平均工资 |
| --- | --- |

+--------------------+------------+

| 销售              | 80667.6058 |
| --- | --- |
| 市场          | 71913.2000 |
| 财务            | 70489.3649 |
| 研究           | 59665.1817 |
| 生产         | 59605.4825 |
| 发展        | 59478.9012 |
| 客户服务   | 58770.3665 |
| 质量管理 | 57251.2719 |
| 人力资源    | 55574.8794 |

+--------------------+------------+

9 行中设置（8.29 秒）

```go

# Identifying Duplicates using SELF JOIN

You want to find the duplicate rows in a table for specific columns. For example, you want to find out which employees have the same `first_name`, same `last_name`, same `gender`, and same `hire_date`. In that case, you can join the `employees` table with itself while specifying the columns where you want to find duplicates in the `JOIN` clause. You need to use different aliases for each table.

You need to add an index on the columns you want to join. The indexes will be discussed in Chapter 13, *Performance Tuning*. For now, you can execute this command to add an index:

```

mysql> ALTER TABLE employees ADD INDEX name(first_name, last_name);

查询成功，影响了 0 行（1.95 秒）

记录：0 重复：0 警告：0

```go

```

mysql> 选择

emp1.*

从

员工 emp1

加入员工作为 emp2

ON emp1.first_name=emp2.first_name

和 emp1.last_name=emp2.last_name

和 emp1.gender=emp2.gender

和 emp1.hire_date=emp2.hire_date

和 emp1.emp_no!=emp2.emp_no

按顺序

名字，姓;

+--------+------------+------------+-----------+--------+------------+

| 员工编号 | 出生日期 | 名字 | 姓 | 性别 | 入职日期 |
| --- | --- | --- | --- | --- | --- |

+--------+------------+------------+-----------+--------+------------+

| 232772 | 1962-05-14 | Keung      | Heusch    | M      | 1986-06-01 |
| --- | --- | --- | --- | --- | --- |
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

```go

You have to mention `emp1.emp_no != emp2.emp_no` because the employees will have different `emp_no`. Otherwise, the same employee will appear.

# Using SUB queries

A subquery is a `SELECT` statement within another statement. Suppose you want to find the name of the employees who started as a `Senior Engineer` on `1986-06-26`.
You can get the `emp_no` from the `titles` table, and name from the `employees` table. You can also use `JOIN` to find out the results.

To get the `emp_no` from titles:

```

mysql> 选择 emp_no 从 titles WHERE title="Senior Engineer" AND from_date="1986-06-26";

+--------+

| emp_no |
| --- |

+--------+

|  10001 |
| --- |
|  84305 |
| 228917 |
| 426700 |
| 458304 |

+--------+

在集合中有 5 行（0.14 秒）

```go

To find the name:

```

mysql> 选择 first_name，last_name 从 employees WHERE emp_no IN（<从前面的查询输出>）

mysql> 选择 first_name，last_name 从 employees WHERE emp_no IN（10001,84305,228917,426700,458304）;

+------------+-----------+

| first_name | last_name |
| --- | --- |

+------------+-----------+

| Georgi     | Facello   |
| --- | --- |
| Minghong   | Kalloufi  |
| Nechama    | Bennet    |
| Nagui      | Restivo   |
| Shuzo      | Kirkerud  |

+------------+-----------+

在集合中有 5 行（0.00 秒

```go

Other clauses such as `EXISTS` and `EQUAL` are also supported in MySQL. Refer to the reference manual, [`dev.mysql.com/doc/refman/8.0/en/subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/subqueries.html), for more details:

```

mysql> 选择

first_name，

last_name

从

员工

WHERE

emp_no

在（从 titles 选择 emp_no WHERE title="Senior Engineer" AND from_date="1986-06-26"）

+------------+-----------+

| first_name | last_name |
| --- | --- |

+------------+-----------+

| Georgi     | Facello   |
| --- | --- |
| Minghong   | Kalloufi  |
| Nagui      | Restivo   |
| Nechama    | Bennet    |
| Shuzo      | Kirkerud  |

+------------+-----------+

在集合中有 5 行（0.91 秒）

```go

Find the employee making the maximum salary:

```

mysql> 选择 emp_no 从 salaries WHERE salary=(SELECT MAX(salary) FROM salaries);

+--------+

| emp_no |
| --- |

+--------+

|  43624 |
| --- |

+--------+

在集合中有 1 行（1.54 秒）

```go

`SELECT MAX(salary) FROM salaries` is the subquery that gives the maximum salary, to find the employee number corresponding to that salary, you can use that subquery in the `WHERE` clause.

# Finding mismatched rows between tables

Suppose you want to find rows in a table that are not in other tables. You can achieve this in two ways. Using the `NOT IN` clause or using `OUTER JOIN`.

To find the matched rows, you can use normal `JOIN`, if you want to find mismatched rows, you can use `OUTER JOIN`. Normal `JOIN` means *A intersection B*. `OUTER JOIN` gives matching records of both *A* and *B* and also gives unmatched records of *A* with `NULL`. If you want the output of `A-B`, you can use the `WHERE <JOIN COLUMN IN B> IS NULL` clause.

To understand the usage of `OUTER JOIN`, create two `employee` tables and insert some values:

```

mysql> 创建表 employees_list1 AS 选择*从 employees WHERE first_name LIKE 'aa%';

查询成功，影响了 444 行（0.22 秒）

记录：444 重复项：0  警告：0

mysql> 创建表 employees_list2 AS 选择*从 employees WHERE emp_no BETWEEN 400000 AND 500000 AND gender='F';

查询成功，影响了 39892 行（0.59 秒）

记录：39892 重复项：0  警告：0

```go

You already know how to find the employees who exist in both lists:

```

mysql> SELECT * FROM employees_list1 WHERE emp_no IN（SELECT emp_no FROM  employees_list2）;

```go

Or you can use `JOIN`:

```

mysql> SELECT l1.* FROM employees_list1 l1 JOIN employees_list2 l2 ON l1.emp_no=l2.emp_no;

```go

To find out the employees who exist in `employees_list1` but not in `employees_list2`:

```

mysql> 选择*从 employees_list1 WHERE emp_no NOT IN（SELECT emp_no FROM  employees_list2）;

```go

Or you can use `OUTER JOIN`:

```

mysql> SELECT l1.* FROM employees_list1 l1 LEFT OUTER JOIN employees_list2 l2 ON l1.emp_no=l2.emp_no WHERE l2.emp_no IS NULL;

```go

The outer join creates `NULL` columns of the second table in the join list for each unmatched row. If you use `RIGHT JOIN`, the first table will get `NULL` values for the unmatched rows.

You can also use `OUTER JOIN` to find matched rows. Instead of `WHERE l2.emp_no IS NULL`, give `WHERE emp_no IS NOT NULL`:

```

mysql> SELECT l1.* FROM employees_list1 l1 LEFT OUTER JOIN employees_list2 l2 ON l1.emp_no=l2.emp_no WHERE l2.emp_no IS NOT NULL;

```go

# Stored procedures

Suppose you need to execute a series of statements in MySQL, instead of sending all SQL statements every time, you can encapsulate all the statements in a single program and call it whenever required. A stored procedure is a set of SQL statements for which no return value is needed.

Apart from SQL statements, you can make use of variables to store results and do programmatical stuff inside a stored procedure. For example, you can write `IF`, `CASE` clauses, logical operations, and `WHILE` loops.

*   Stored functions and procedures are also referred to as stored routines.
*   For creating a stored procedure, you should have the `CREATE ROUTINE` privilege.
*   Stored functions will have a return value.
*   Stored procedures do not have a return value.
*   All the code is written inside the `BEGIN and END` block.
*   Stored functions can be called directly in a `SELECT` statement.
*   Stored procedures can be called using the `CALL` statement.
*   Since the statements inside stored routines should end with a delimiter (`;`), you have to change the delimiter for MySQL so that MySQL won't interpret the SQL statements inside a stored routine with normal statements. After the creation of the procedure, you can change the delimiter back to the default value.

# How to do it...

For example, you want to add a new employee. You should update three tables, namely `employees`, `salaries`, and `titles`. Instead of executing three statements, you can develop a stored procedure and call it to create a new `employee`.

You have to pass the employee's `first_name`, `last_name`, `gender`, and `birth_date`, as well as the department the employee joins. You can pass those using input variables and you should get the employee number as output. The stored procedure does not return a value, but it can update a variable and you can use it.

Here is a simple example of a stored procedure to create a new employee and update the `salary` and `department` tables:

```

/* 如果有同名的现有过程，请在创建之前删除 */

删除过程如果存在 create_employee;

/* 将分隔符更改为$$ */

DELIMITER $$

/* IN 指定作为参数的变量，INOUT 指定输出变量*/

创建过程创建员工（OUT new_emp_no INT，IN first_name varchar(20)，IN last_name varchar(20)，IN gender enum('M'，'F')，IN birth_date date，IN emp_dept_name varchar(40)，IN title varchar(50))

开始

/* 为 emp_dept_no 和 salary 声明变量 */

DECLARE emp_dept_no char(4);

DECLARE salary int DEFAULT 60000;

/* 选择最大的员工编号到变量 new_emp_no */

SELECT max(emp_no) INTO new_emp_no FROM employees;

/* 增加 new_emp_no */

SET new_emp_no = new_emp_no + 1;

/* 将数据插入 employees 表 */

/* 函数 CURDATE()给出当前日期） */

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

```go

To create a stored procedure, you can:

*   Paste it in the command-line client
*   Save it in the file and import it in MySQL using `mysql -u <user> -p employees < stored_procedure.sql`
*   Source the `mysql> SOURCE stored_procedure.sql` file

To use the stored procedure, grant the execute privilege to the `emp_read_only` user:

```

mysql> GRANT EXECUTE ON employees.* TO 'emp_read_only'@'%';

Query OK, 0 rows affected (0.05 sec)

```go

Invoke the stored procedure using the `CALL stored_procedure(OUT variable, IN values)` statement and name of the routine.

Connect to MySQL using the `emp_read_only` account:

```

shell> mysql -u emp_read_only -pemp_pass employees -A

```go

Pass the variable where you want to store the `@new_emp_no` output and also pass the required input values:

```

mysql> CALL create_employee(@new_emp_no, 'John', 'Smith', 'M', '1984-06-19', 'Research', 'Staff');

Query OK, 1 row affected (0.01 sec)

```go

Select the value of `emp_no`, which is stored in the `@new_emp_no` variable:

```

mysql> SELECT @new_emp_no;

+-------------+

| @new_emp_no |
| --- |

+-------------+

|      500000 |
| --- |

+-------------+

1 row in set (0.00 sec)

```go

Check that the rows are created in the `employees`, `salaries`, and `titles` tables:

```

mysql> SELECT * FROM employees WHERE emp_no=500000;

+--------+------------+------------+-----------+--------+------------+

| emp_no | birth_date | first_name | last_name | gender | hire_date  |
| --- | --- | --- | --- | --- | --- |

+--------+------------+------------+-----------+--------+------------+

| 500000 | 1984-06-19 | John       | Smith     | M      | 2017-06-17 |
| --- | --- | --- | --- | --- | --- |

+--------+------------+------------+-----------+--------+------------+

1 row in set (0.00 sec)

mysql> SELECT * FROM salaries WHERE emp_no=500000;

+--------+--------+------------+------------+

| emp_no | salary | from_date  | to_date    |
| --- | --- | --- | --- |

+--------+--------+------------+------------+

| 500000 | 100000 | 2017-06-17 | 9999-01-01 |
| --- | --- | --- | --- |

+--------+--------+------------+------------+

1 row in set (0.00 sec)

mysql> SELECT * FROM titles WHERE emp_no=500000;

+--------+-------+------------+------------+

| emp_no | title | from_date  | to_date    |
| --- | --- | --- | --- |

+--------+-------+------------+------------+

| 500000 | Staff | 2017-06-17 | 9999-01-01 |
| --- | --- | --- | --- |

+--------+-------+------------+------------+

1 row in set (0.00 sec)

```go

You can see that, even though `emp_read_only` has no write access on the tables, it is able to write by calling the stored procedure. If the `SQL SECURITY` of the stored procedure is created as `INVOKER`, `emp_read_only` cannot modify the data. Note that if you are connecting using `localhost`, create the privileges for the `localhost` user.

To list all the stored procedures in a database, execute `SHOW PROCEDURE STATUS\G`. To check the definition of an existing stored routine, you can execute `SHOW CREATE PROCEDURE <procedure_name>\G`.

# There's more...

Stored procedures are also used to enhance the security. The user needs the `EXECUTE` privilege on the stored procedure to execute it.
By the definition of a stored routine:

*   The `DEFINER` clause specifies the creator of the stored routine. If nothing is specified, the current user is taken.
*   The `SQL SECURITY` clause specifies the execution context of the stored routine. It can be either `DEFINER` or `INVOKER`.

`DEFINER`: A user even with only the `EXECUTE` permission for routine can call and get the output of the stored routine, regardless of whether that user has permission on the underlying tables or not. It is enough if `DEFINER` has privileges.

`INVOKER`: The security context is switched to the user who invokes the stored routine. In this case, the invoker should have access to the underlying tables.

# See also

Refer to the documentation for more examples and syntax, at [`dev.mysql.com/doc/refman/8.0/en/create-procedure.html`](https://dev.mysql.com/doc/refman/8.0/en/create-procedure.html).

# Functions

Just like stored procedures, you can create stored functions. The main difference is functions should have a return value and they can be called in `SELECT`. Usually, stored functions are created to simplify complex calculations.

# How to do it...

Here is an example of how to write a function and how to call it. Suppose a banker wants to give a credit card based on income level, instead of exposing the actual salary, you can expose this function to find out the income level:

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

```go

To create the function:

```

mysql> SOURCE function.sql;

Query OK, 0 rows affected (0.00 sec)

Query OK, 0 rows affected (0.01 sec)

```go

```

您必须传递员工编号，函数返回收入水平。

mysql> SELECT get_sal_level(10002);

+----------------------+

| get_sal_level(10002) |
| --- |

+----------------------+

| SILVER               |
| --- |

+----------------------+

1 row in set (0.00 sec)

mysql> SELECT get_sal_level(10001);

+----------------------+

| get_sal_level(10001) |
| --- |

+----------------------+

| GOLD                 |
| --- |

+----------------------+

1 row in set (0.00 sec)

mysql> SELECT get_sal_level(1);

+------------------+

| get_sal_level(1) |
| --- |

+------------------+

| NOT FOUND        |
| --- |

+------------------+

1 row in set (0.00 sec)

```go

To list all the stored functions in a database, execute `SHOW FUNCTION STATUS\G`. To check the definition of the existing stored function, you can execute `SHOW CREATE FUNCTION <function_name>\G`.

It is very important to give the `DETERMINISTIC` keyword in the function creation. A routine is considered `DETERMINISTIC` if it always produces the same result for the same input parameters, and `NOT DETERMINISTIC` otherwise. If neither `DETERMINISTIC` nor `NOT DETERMINISTIC` is given in the routine definition, the default is `NOT DETERMINISTIC`. To declare that a function is deterministic, you must specify `DETERMINISTIC` explicitly.
Declaring a `NON DETERMINISTIC` routine as `DETERMINISTIC` might lead to unexpected results, by causing the optimizer to make incorrect execution plan choices. Declaring a `DETERMINISTIC` routine as `NON DETERMINISTIC` might diminish performance, by causing available optimizations not to be used.

# Inbuilt functions

MySQL provides numerous inbuilt functions. You have already used the `CURDATE()` function to get the current date.

You can use the functions in the `WHERE` clause:

```

mysql> SELECT * FROM employees WHERE hire_date = CURDATE();

```go

*   For example, the following function gives the date from exactly one week ago:

```

mysql> SELECT DATE_ADD(CURDATE(), INTERVAL -7 DAY) AS '7 Days Ago';

```go

*   Add two strings:

```

mysql> SELECT CONCAT(first_name, ' ', last_name) FROM employees LIMIT 1;

+------------------------------------+

| CONCAT(first_name, ' ', last_name) |
| --- |

+------------------------------------+

| Aamer Anger                        |
| --- |

+------------------------------------+

1 row in set (0.00 sec)

```go

# See also

Refer to the MySQL reference manual for a complete list of functions, at [`dev.mysql.com/doc/refman/8.0/en/func-op-summary-ref.html`](https://dev.mysql.com/doc/refman/8.0/en/func-op-summary-ref.html).

# Triggers

A trigger is used to activate something before or after the trigger event. For example, you can have a trigger activate before each row that is inserted into a table or after each row that is updated.

Triggers are highly useful while altering a table without downtime (Refer to Chapter 10, *Table Maintenance*, in the *Alter table using online schema change tool* section) and also for auditing purposes. Suppose you want to find out the previous value of a row, you can write a trigger that saves those rows in another table before updating. The other table serves as an audit table that has the previous records.

The trigger action time can be `BEFORE` or `AFTER`, which indicates whether the trigger activates before or after each row to be modified.

Trigger events can be `INSERT`, `DELETE`, or `UPDATE`:

*   `INSERT`: Whenever a new row gets inserted through `INSERT`, `REPLACE`, or `LOAD DATA`, the trigger gets activated
*   `UPDATE`: Through the `UPDATE` statement
*   `DELETE`: Through the `DELETE` or `REPLACE` statements

From MySQL 5.7, a table can have multiple triggers at the same time. For example, a table can have two `BEFORE INSERT` triggers. You have to specify which trigger should go first using `FOLLOWS` or `PRECEDES`.

# How to do it...

For example, you want to round off the salary before inserting it into the `salaries` table. `NEW` refers to the new value that is being inserted:

```

shell> vi before_insert_trigger.sql

如果存在删除触发器工资轮；

DELIMITER $$

创建触发器工资轮在工资上的每一行之前插入

对于每一行

开始

设置 NEW.salary=ROUND（NEW.salary）；

END

$$

DELIMITER；

```go

Create the trigger by sourcing the file:

```

mysql> SOURCE before_insert_trigger.sql；

查询成功，0 行受影响（0.06 秒）

查询成功，0 行受影响（0.00 秒）

```go

Test the trigger by inserting a floating number into the salary:

```

mysql> 插入工资值（10002，100000.79，CURDATE（），'9999-01-01'）；

查询成功，影响 1 行（0.04 秒）

```go

You can see that the salary is rounded off:

```

mysql> 选择*从工资中，员工编号=10002 和从日期=CURDATE();

+--------+--------+------------+------------+

| emp_no | salary | from_date | to_date |
| --- | --- | --- | --- |

+--------+--------+------------+------------+

| 10002 | 100001 | 2017-06-18 | 9999-01-01 |
| --- | --- | --- | --- |

+--------+--------+------------+------------+

1 行设置（0.00 秒）

```go

Similarly, you can create a `BEFORE UPDATE` trigger to round off the salary. Another example: you want to log which user has inserted into the `salaries` table. Create an `audit` table:

```

mysql> 创建工资审计表（emp_no int，user varchar（50），date_modified date）；

```go

Note that the following trigger precedes the `salary_round` trigger, which is specified by `PRECEDES salary_round`:

```

shell> vi before_insert_trigger.sql

DELIMITER $$

创建触发器工资审计

在插入之前

在工资上对于每一行在 salary_round 之前

开始

插入工资审计值（NEW.emp_no，USER（），CURDATE（））；

END; $$

DELIMITER；

```go

Insert it into `salaries`:

```

mysql> 插入工资值（10003，100000.79，CURDATE（），'9999-01-01'）；

查询成功，影响 1 行（0.06 秒）

```go

Find out who inserted the salary by querying the `salary_audit` table:

```

mysql> 选择*从工资审计中，员工编号=10003；

+--------+----------------+---------------+

| emp_no | user | date_modified | |
| --- | --- | --- | --- |

+--------+----------------+---------------+

| 10003 | root@localhost | 2017-06-18 |
| --- | --- | --- |

+--------+----------------+---------------+

查询成功，1 行受影响（0.00 秒）

```go

In case the `salary_audit` table is dropped or is not available, all the inserts on the `salaries` table will be blocked. If you do not want to do auditing, you should drop the trigger first and then the table.
Triggers can make overhead on the write speed based on the complexity of it.
To check all the triggers, execute `SHOW TRIGGERS\G`.
To check the definition of an existing trigger, execute `SHOW CREATE TRIGGER <trigger_name>`.

# See also

Refer to the MySQL reference manual, at [`dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html), for more details.

# Views

View is a virtual table based on the result-set of an SQL statement. It will also have rows and columns just like a real table, but few restrictions, which will be discussed later. Views hide the SQL complexity and, more importantly, provide additional security.

# How to do it...

Suppose you want to give access only to the `emp_no` and `salary` columns of the `salaries` table, and `from_date` is after `2002-01-01`. For this, you can create a view with the SQL that gives the required result.

```

mysql> 创建算法=未定义

DEFINER=`root`@`localhost`

SQL 安全性定义器视图 salary_view

AS

选择 emp_no，工资从工资中，从日期>'2002-01-01'；

```go

Now the `salary_view` view is created and you can query it just like any other table:

```

mysql> 选择 emp_no，AVG（salary）作为 avg 从 salary_view 中 GROUP BY emp_no ORDER BY avg DESC LIMIT 5；

```go

You can see that the view has access to particular rows (that is, `from_date > '2002-01-01'`) and not all of the rows. You can use the view to restrict user access to particular rows.

To list all views, execute:

```

mysql> 显示完整的表，其中表类型为“视图”；

```go

To check the definition of the view, execute:

```

mysql> 显示创建视图 salary_view\G

```go

You might have noticed the `current_dept_emp` and `dept_emp_latest_date` views, which are there as part of the `employee` database. You can explore the definition and find out their purpose.

Simple views that do not have sub-queries, `JOINS`, `GROUP BY` clauses, union, and so on, can be updated. `salary_view` is a simple view that can be updated or inserted if the underlying tables have a default value:

```

mysql> 更新 salary_view 设置工资=100000，员工编号=10001；

查询成功，影响 1 行（0.01 秒）

匹配行：2 已更改：1 警告：0

```go

```

mysql> 插入 salary_view 值（10001，100001）；

错误 1423（HY000）：视图'employees.salary_view'的基础表字段没有默认值

```go

If the table has a default value, you could have inserted a row even if it does not match the filter criteria in the view. To avoid that, and to insert rows that satisfy the view condition, you have to provide `WITH CHECK OPTION` in the definition.

The `VIEW` algorithms:

*   `MERGE`: MySQL combines the input query and the view definition into a single query and then executes the combined query. The `MERGE` algorithm is allowed only on simple views.
*   `TEMPTABLE`: MySQL stores the results in the temporary table and then it executes the input query against this temporary table.
*   `UNDEFINED` (default): MySQL automatically chooses the `MERGE` or `TEMPTABLE` algorithm. MySQL prefers the `MERGE` algorithm to the `TEMPTABLE` algorithm because the `MERGE` algorithm is much more efficient.

# Events

Just like a cron on a Linux server, MySQL has `EVENTS` to handle the scheduled tasks. MySQL uses a special thread called the event schedule thread to execute all scheduled events. By default, the event scheduler thread is not enabled (version < 8.0.3), to enable it, execute: 

```

mysql> 设置全局事件调度程序=开；

```go

# How to do it...

Suppose you no longer need to keep salary audit records that are more than a month old, you can schedule an event that runs daily and deletes the records from the `salary_audit` table that are a month old.

```

mysql> 如果存在清除工资审计事件；

DELIMITER $$

创建事件如果不存在清除工资审计

按计划

每 1 周

开始日期

DO BEGIN

删除工资审计，其中修改日期<DATE_ADD（CURDATE（），INTERVAL -7 天）；

END $$

DELIMITER；

```go

Once the event is created, it will automatically do the job of purging the salary audit records.

*   To check the events, execute:

```

mysql> 显示事件\G

*************************** 1\. 行 ***************************

数据库：员工

名称：清除工资审计

定义者：root@localhost

时区：系统

类型：循环

在 NULL 处执行

间隔值：1

间隔字段：分钟

开始日期：2017-06-18 00:00:00

结束：NULL

状态：已启用

发起者：0

客户端字符集：utf8

连接排序：utf8_general_ci

数据库排序：utf8mb4_0900_ai_ci

1 行设置（0.00 秒）

```go

*   To check the definition on the event, execute:

```

mysql> 显示创建事件清除工资审计\G

```go

*   To disable/enable the event, execute the following:

```

mysql> ALTER EVENT purge_salary_audit DISABLE；

mysql> ALTER EVENT purge_salary_audit ENABLE；

```go

# Access control

All stored programs (procedures, functions, triggers, and events) and views have a `DEFINER`. If the `DEFINER` is not specified, the user who creates the object will be chosen as `DEFINER`.

Stored routines (procedures and functions) and views  have an `SQL SECURITY` characteristic with a value of `DEFINER` or `INVOKER` to specify whether the object executes in the definer or invoker context. Triggers and events have no `SQL SECURITY` characteristic and always execute in the definer context. The server invokes these objects automatically as necessary, so there is no invoking user.

# See also

There are lots of ways to schedule events, refer to [`dev.mysql.com/doc/refman/8.0/en/event-scheduler.html`](https://dev.mysql.com/doc/refman/8.0/en/event-scheduler.html) for more details.

# Getting information about databases and tables

You might have already noticed an `information_schema` database in the list of databases. `information_schema` is a collection of views that consist of metadata about all the database objects. You can connect to `information_schema` and explore all the tables. The most widely-used tables are explained in this chapter. You either query the `information_schema` tables or use the `SHOW` command, which essentially does the same.

`INFORMATION_SCHEMA` queries are implemented as views over the `data dictionary` tables. There are two types of metadata in the `INFORMATION_SCHEMA` tables:

*   **Static table metadata**: `TABLE_SCHEMA`, `TABLE_NAME`, `TABLE_TYPE`, and `ENGINE`. These statistics will be read directly from the `data dictionary`.
*   **Dynamic table metadata**: `AUTO_INCREMENT`, `AVG_ROW_LENGTH`, and `DATA_FREE`. Dynamic metadata frequently changes (for example, the `AUTO_INCREMENT` value will advance after each `INSERT`). In many cases, the dynamic metadata will also incur some cost to accurately calculate on demand, and accuracy may not be beneficial for the typical query. Consider the case of the `DATA_FREE` statistic that shows the number of free bytes in a table—a cached value is usually sufficient.

In MySQL 8.0, the dynamic table metadata will default to being cached. This is configurable via the `information_schema_stats` setting (default cached), and can be changed to `SET @@GLOBAL.information_schema_stats='LATEST'` in order to always retrieve the dynamic information directly from the storage engine (at the cost of slightly higher query execution).

As an alternative, the user can also execute `ANALYZE TABLE` on the table, to update the cached dynamic statistics.

Most of the tables have the `TABLE_SCHEMA` column, which refers to the database name, and the `TABLE_NAME` column, which refers to the table name.

Refer to [`mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/`](https://mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/) for more details.

# How to do it...

Check the list of all the tables:

```

mysql> 使用信息模式;

mysql> 显示表；

```go

# TABLES

The `TABLES` table contains all the information about the table, such as which database belongs to `TABLE_SCHEMA`, the number of rows (`TABLE_ROWS`), `ENGINE`, `DATA_LENGTH`, `INDEX_LENGTH`, and `DATA_FREE`:

```

mysql> DESC 信息模式。表；

+-----------------+--------------------------------------------------------------------+------+-----+-------------------+-----------------------------+

| 字段 | 类型 | 空 | 键 | 默认 | 额外 |
| --- | --- | --- | --- | --- | --- |

+-----------------+--------------------------------------------------------------------+------+-----+-------------------+-----------------------------+

| 表目录 | varchar(64) | 否 | | 空 | |
| --- | --- | --- | --- | --- | --- |
| 表模式 | varchar(64) | 否 | | 空 | |
| 表名称 | varchar(64) | 否 | | 空 | |
| 表类型           | enum('BASE TABLE','VIEW','SYSTEM VIEW')                            | 否   |     | NULL              |                             |
| 引擎            | varchar(64)                                                        | 是   |     | NULL              |                             |
| 版本            | int(2)                                                             | 是   |     | NULL              |                             |
| 行格式           | enum('Fixed','Dynamic','Compressed','Redundant','Compact','Paged') | 是   |     | NULL              |                             |
| 表行数           | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 平均行长度       | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 数据长度         | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 最大数据长度     | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 索引长度         | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 空闲数据         | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 自动增量         | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 创建时间         | timestamp                                                          | 否   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| 更新时间         | timestamp                                                          | 是   |     | NULL              |                             |
| 校验时间         | timestamp                                                          | 是   |     | NULL              |                             |
| 表排序规则       | varchar(64)                                                        | 是   |     | NULL              |                             |
| 校验和           | bigint(20) unsigned                                                | 是   |     | NULL              |                             |
| 创建选项         | varchar(256)                                                       | 是   |     | NULL              |                             |
| 表注释           | varchar(256)                                                       | 是   |     | NULL              |                             |

+-----------------+--------------------------------------------------------------------+------+-----+-------------------+-----------------------------+

21 行 (0.00 秒)

```go

For example, you want to know `DATA_LENGTH`, `INDEX_LENGTH`, and `DATE_FREE` inside the `employees` database:

```

mysql> SELECT SUM(DATA_LENGTH)/1024/1024 AS DATA_SIZE_MB, SUM(INDEX_LENGTH)/1024/1024 AS INDEX_SIZE_MB, SUM(DATA_FREE)/1024/1024 AS DATA_FREE_MB FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA='employees';

+--------------+---------------+--------------+

| 数据大小(MB) | 索引大小(MB) | 空闲数据(MB) |
| --- | --- | --- |

+--------------+---------------+--------------+

|  17.39062500 |   14.62500000 |  11.00000000 |
| --- | --- | --- |

+--------------+---------------+--------------+

1 行 (0.01 秒)

```go

# COLUMNS

This table lists all the columns and its definition for each table:

```

mysql> SELECT * FROM COLUMNS WHERE TABLE_NAME='employees'\G

```go

# FILES

You have already seen that MySQL stores the `InnoDB` data in the `.ibd` files inside a directory (with the same name as the database name) in the `data directory` . To get more information about the files, you can query the `FILES` table:

```

mysql> SELECT * FROM FILES WHERE FILE_NAME LIKE './employees/employees.ibd'\G

~~~

EXTENT_SIZE: 1048576

AUTOEXTEND_SIZE: 4194304

DATA_FREE: 13631488

~~~

```go

You should be keen at `DATA_FREE`, which represents the unallocated segments plus the data that is free inside the segments due to fragmentation. When you rebuild the table, you can free up bytes shown in `DATA_FREE`.

# INNODB_SYS_TABLESPACES

The size of the file is also available in the `INNODB_TABLESPACES` table:

```

mysql> SELECT * FROM INNODB_TABLESPACES WHERE NAME='employees/employees'\G

*************************** 1\. 行 ***************************

SPACE: 118

名称: employees/employees

标志: 16417

行格式: Dynamic

PAGE_SIZE: 16384

ZIP_PAGE_SIZE: 0

SPACE_TYPE: Single

FS_BLOCK_SIZE: 4096

文件大小: 32505856

ALLOCATED_SIZE: 32509952

1 行 (0.00 秒)

```go

You can verify the same in the filesystem:

```

shell> sudo ls -ltr /var/lib/mysql/employees/employees.ibd

-rw-r----- 1 mysql mysql 32505856 Jun 20 16:50 /var/lib/mysql/employees/employees.ibd

```go

# INNODB_TABLESTATS

The size of the index and approximate number of rows is available in the `INNODB_TABLESTATS` table:

```

mysql> 从 INNODB_TABLESTATS 中选择* WHERE NAME='employees/employees'\G

*************************** 1\. 行 ***************************

TABLE_ID: 128

名称：employees/employees

STATS_INITIALIZED: 已初始化

NUM_ROWS: 299468

CLUST_INDEX_SIZE: 1057

OTHER_INDEX_SIZE: 545

MODIFIED_COUNTER: 0

AUTOINC: 0

REF_COUNT: 1

1 行（0.00 秒）

```go

# PROCESSLIST

One of the most used views is the process list. It lists all the queries running on the server:

```

mysql> 从 PROCESSLIST\G 中选择* 

*************************** 1\. 行 ***************************

ID: 85

用户：event_scheduler

主机：localhost

数据库：NULL

命令：守护进程

时间：44

状态：等待下一次激活

INFO: NULL

*************************** 2\. 行 ***************************

ID: 26231

用户：root

主机：localhost

数据库：information_schema

命令：查询

时间：0

状态：执行中

INFO: 从 PROCESSLIST 中选择* 

2 行（0.00 秒）

```

或者您可以执行`SHOW PROCESSLIST;`以获得相同的输出。

**其他表**：`ROUTINES`包含函数和存储过程的定义。`TRIGGERS`包含触发器的定义。`VIEWS`包含视图的定义。

# 另请参阅

要了解`INFORMATION_SCHEMA`的改进，请参阅[`mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/`](http://mysqlserverteam.com/mysql-8-0-improvements-to-information_schema/)。
