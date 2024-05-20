# 第五章：事务

在本章中，我们将涵盖以下示例：

+   执行事务

+   使用保存点

+   隔离级别

+   锁定

# 介绍

在接下来的示例中，我们将讨论 MySQL 中的事务和各种隔离级别。事务意味着一组应该一起成功或失败的 SQL 语句。事务还应该满足**原子性、一致性、隔离性和** **持久性**（**ACID**）属性。以一个非常基本的例子，从账户`A`转账到账户`B`。假设`A`有 600 美元，`B`有 400 美元，`B`希望从`A`转账 100 美元给自己。

银行将从`A`扣除 100 美元，并使用以下 SQL 代码将其添加到`B`（仅供说明）：

```go
mysql> SELECT balance INTO @a.bal FROM account WHERE account_number='A';
```

以编程方式检查`@a.bal`是否大于或等于 100：

```go
mysql> UPDATE account SET balance=@a.bal-100 WHERE account_number='A';
mysql> SELECT balance INTO @b.bal FROM account WHERE account_number='B';
```

以编程方式检查`@b.bal`是否`NOT NULL`：

```go
mysql> UPDATE account SET balance=@b.bal+100 WHERE account_number='B';
```

这四行 SQL 应该是一个单独的事务，并满足以下 ACID 属性：

+   **原子性**：要么所有的 SQL 都应该成功，要么都应该失败。不应该有任何部分更新。如果不遵守这个属性，如果数据库在运行两个 SQL 后崩溃，那么`A`将会损失 100 美元。

+   一致性：事务必须以允许的方式仅改变受影响的数据。在这个例子中，如果带有`B`的`account_number`不存在，整个事务应该被回滚。

+   **隔离性**：同时发生的事务（并发事务）不应该导致数据库处于不一致的状态。每个事务应该被执行，就好像它是系统中唯一的事务一样。没有任何事务应该影响任何其他事务的存在。假设`A`在转账给`B`的同时完全转移了这 600 美元；两个事务应该独立运行，确保在转移金额之前的余额。

+   **持久性**：数据应该持久存在于磁盘上，不应该在任何数据库或系统故障时丢失。

`InnoDB`是 MySQL 中的默认存储引擎，支持事务，而 MyISAM 不支持事务。

# 执行事务

创建虚拟表和示例数据以理解这个示例：

```go
mysql> CREATE DATABASE bank;
mysql> USE bank;
mysql> CREATE TABLE account(account_number varchar(10) PRIMARY KEY, balance int);
mysql> INSERT INTO account VALUES('A',600),('B',400);
```

# 如何做...

要开始一个事务（一组 SQL），执行`START TRANSACTION`或`BEGIN`语句：

```go
mysql> START TRANSACTION;
or 
mysql> BEGIN;
```

然后执行所有希望在事务内部的语句，比如从`A`转账 100 到`B`：

```go
mysql> SELECT balance INTO @a.bal FROM account WHERE account_number='A';

Programmatically check if @a.bal is greater than or equal to 100 
mysql> UPDATE account SET balance=@a.bal-100 WHERE account_number='A';
mysql> SELECT balance INTO @b.bal FROM account WHERE account_number='B';

Programmatically check if @b.bal IS NOT NULL 
mysql> UPDATE account SET balance=@b.bal+100 WHERE account_number='B';
```

确保所有 SQL 都成功执行后，执行`COMMIT`语句，完成事务并提交数据：

```go
mysql> COMMIT;
```

如果在中间遇到任何错误并希望中止事务，可以发出`ROLLBACK`语句而不是`COMMIT`。

例如，如果`A`想要转账到一个不存在的账户而不是发送给`B`，你应该中止事务并将金额退还给`A`：

```go
mysql> BEGIN;

mysql> SELECT balance INTO @a.bal FROM account WHERE account_number='A';

mysql> UPDATE account SET balance=@a.bal-100 WHERE account_number='A';

mysql> SELECT balance INTO @b.bal FROM account WHERE account_number='C';
Query OK, 0 rows affected, 1 warning (0.07 sec)

mysql> SHOW WARNINGS;
+---------+------+-----------------------------------------------------+
| Level   | Code | Message                                             |
+---------+------+-----------------------------------------------------+
| Warning | 1329 | No data - zero rows fetched, selected, or processed |
+---------+------+-----------------------------------------------------+
1 row in set (0.02 sec)

mysql> SELECT @b.bal;
+--------+
| @b.bal |
+--------+
| NULL   |
+--------+
1 row in set (0.00 sec)

mysql> ROLLBACK;
Query OK, 0 rows affected (0.01 sec)
```

# 自动提交

默认情况下，自动提交是`ON`，这意味着所有单独的语句在执行时都会被提交，除非它们在`BEGIN...COMMIT`块中。如果自动提交是`OFF`，你需要显式地发出`COMMIT`语句来提交一个事务。要禁用它，执行：

```go
mysql> SET autocommit=0;
```

DDL 语句，比如数据库的`CREATE`或`DROP`，以及表或存储过程的`CREATE`、`DROP`或`ALTER`，不能被回滚。

有一些语句，比如 DDLs、`LOAD DATA INFILE`、`ANALYZE TABLE`、与复制相关的语句等会导致隐式的`COMMIT`。有关这些语句的更多细节，请参阅[`dev.mysql.com/doc/refman/8.0/en/implicit-commit.html`](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)。

# 使用保存点

使用保存点，你可以在事务中回滚到某些点，而不终止事务。你可以使用`SAVEPOINT identifier`来为事务设置一个名称，并使用`ROLLBACK TO identifier`语句来将事务回滚到指定的保存点，而不终止事务。

# 如何做...

假设`A`想要转账给多个账户；即使向一个账户的转账失败，其他账户也不应该被回滚：

```go
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT balance INTO @a.bal FROM account WHERE account_number='A';
Query OK, 1 row affected (0.01 sec)

mysql> UPDATE account SET balance=@a.bal-100 WHERE account_number='A';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1 Changed: 1 Warnings: 0

mysql> UPDATE account SET balance=balance+100 WHERE account_number='B';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1 Changed: 1 Warnings: 0

mysql> SAVEPOINT transfer_to_b;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT balance INTO @a.bal FROM account WHERE account_number='A';
Query OK, 1 row affected (0.00 sec)

mysql> UPDATE account SET balance=balance+100 WHERE account_number='C';
Query OK, 0 rows affected (0.00 sec)
Rows matched: 0 Changed: 0 Warnings: 0

### Since there are no rows updated, meaning there is no account with 'C', you can rollback the transaction to SAVEPOINT where transfer to B is successful. Then 'A' will get back 100 which was deducted to transfer to C. If you wish not to use the save point, you should do these in two transactions.

mysql> ROLLBACK TO transfer_to_b;
Query OK, 0 rows affected (0.00 sec)

mysql> COMMIT;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT balance FROM account WHERE account_number='A';
+---------+
| balance |
+---------+
| 400     |
+---------+
1 row in set (0.00 sec)

mysql> SELECT balance FROM account WHERE account_number='B';
+---------+
| balance |
+---------+
| 600     |
+---------+
1 row in set (0.00 sec)
```

# 隔离级别

当两个或更多事务同时发生时，隔离级别定义了事务与其他事务所做的资源或数据修改隔离的程度。有四种隔离级别；要更改隔离级别，您需要设置动态的具有会话级范围的`tx_isolation`变量。

# 如何做...

要更改此级别，请执行`SET @@transaction_isolation = 'READ-COMMITTED';`。

# 读取未提交

当前事务可以读取另一个未提交事务写入的数据，这也称为**脏读**。

例如，`A`想要向他的账户添加一些金额并转账给`B`。假设两个交易同时发生；流程将如下。

`A`最初有 400 美元，想要在向`B`转账 500 美元后向他的账户添加 500 美元。

| **# 事务 1（添加金额）** | **# 事务 2（转账金额）** |
| --- | --- |

|

```go
BEGIN;
```

|

```go
BEGIN;
```

|

|

```go
UPDATE account
 SET balance=balance+500
 WHERE account_number='A';
```

| -- |
| --- |
|  -- |

```go
SELECT balance INTO @a.bal
 FROM account
 WHERE account_number='A';
 # A sees 900 here
```

|

|

```go
ROLLBACK;
 # Assume due to some reason the
 transaction got rolled back
```

| -- |
| --- |
| -- |

```go
# A transfers 900 to B since
 A has 900 in previous SELECT
 UPDATE account
 SET balance=balance-900
 WHERE account_number='A';
```

|

|  -- |
| --- |

```go
# B receives the amount UPDATE account
 SET balance=balance+900
 WHERE account_number='B';
```

|

|  -- |
| --- |

```go
# Transaction 2 completes successfully
COMMIT;
```

|

您可以注意到*事务 2*已经读取了未提交或回滚的*事务 1*的数据，导致此事务后账户`A`的余额变为负数，这显然是不希望的。

# 读取提交

当前事务只能读取另一个事务提交的数据，这也称为**不可重复读**。

再举一个例子，`A`有 400 美元，`B`有 600 美元。

| **# 事务 1（添加金额）** | **# 事务 2（转账金额）** |
| --- | --- |

|

```go
BEGIN;
```

|

```go
BEGIN;
```

|

|

```go
UPDATE account SET balance=balance+500
WHERE account_number='A';
```

|  -- |
| --- |
| -- |

```go
SELECT balance INTO @a.bal
FROM account
WHERE account_number='A';
# A sees 400 here because transaction 1 has not committed the data yet 
```

|

|

```go
COMMIT;
```

| -- |
| --- |
| -- |

```go
SELECT balance INTO @a.bal
FROM account
WHERE account_number='A';
# A sees 900 here because transaction 1 has committed the data. 
```

|

您可以注意到，在同一事务中，相同的`SELECT`语句获取了不同的结果。

# 可重复读

即使另一个事务已经提交了数据，事务仍将看到由第一个语句读取的相同数据。同一事务中的所有一致读取都读取第一次读取建立的快照。例外是可以读取同一事务中更改的数据的事务。

当事务开始并执行其第一次读取时，将创建一个读取视图，并保持打开直到事务结束。为了在事务结束之前提供相同的结果集，`InnoDB`使用行版本和`UNDO`信息。假设*事务 1*选择了一些行，另一个事务删除了这些行并提交了数据。如果*事务 1*仍然打开，它应该能够看到它一开始选择的行。已删除的行被保留在`UNDO`日志空间中以满足*事务 1*。一旦*事务 1*完成，这些行将被标记为从`UNDO`日志中删除。这称为**多版本并发控制**（**MVCC**）。

再举一个例子，`A`有 400 美元，`B`有 600 美元。

| **# 事务 1（添加金额）** | **# 事务 2（转账金额）** |
| --- | --- |

|

```go
BEGIN;
```

|

```go
BEGIN;
```

|

|  -- |
| --- |

```go
SELECT balance INTO @a.bal
FROM account
WHERE account_number='A';
# A sees 400 here
```

|

|

```go
UPDATE account
SET balance=balance+500
WHERE account_number='A';
```

| -- |
| --- |
| -- |

```go
SELECT balance INTO @a.bal
FROM account
WHERE account_number='A';
# A sees still 400 even though transaction 1 is committed
```

|

|

```go
COMMIT;
```

| -- |
| --- |
|  -- |

```go
COMMIT;
```

|

|  -- |
| --- |

```go
SELECT balance INTO @a.bal
FROM account
WHERE account_number='A';
# A sees 900 here because this is a fresh transaction
```

|

这仅适用于`SELECT`语句，不一定适用于 DML 语句。如果您插入或修改了一些行，然后提交该事务，来自另一个并发的`REPEATABLE READ`事务的`DELETE`或`UPDATE`语句可能会影响那些刚刚提交的行，即使会话无法查询它们。如果一个事务更新或删除了另一个事务提交的行，这些更改将对当前事务可见。

例如：

| **# 事务 1** | **# 事务 2** |
| --- | --- |

|

```go
BEGIN;
```

|

```go
BEGIN;
```

|

|

```go
SELECT * FROM account;
# 2 rows are returned
```

|  -- |
| --- |
|  -- |

```go
INSERT INTO account VALUES('C',1000);
# New account is created
```

|

|   -- |
| --- |

```go
COMMIT;
```

|

|

```go
SELECT * FROM account WHERE account_number='C';
# no rows are returned because of MVCC
```

|  -- |
| --- |

|

```go
DELETE FROM account WHERE account_number='C';
# Surprisingly account C gets deleted
```

|  -- |
| --- |
|  -- |

```go
SELECT * FROM account;
# 3 rows are returned because transaction 1 is not yet committed
```

|

|

```go
COMMIT;
```

| -- |
| --- |
|  -- |

```go
SELECT * FROM account;
# 2 rows are returned because transaction 1 is committed
```

|

这是另一个例子：

| **# 事务 1** | **# 事务 2** |
| --- | --- |

|

```go
BEGIN;
```

|

```go
BEGIN;
```

|

|

```go
SELECT * FROM account;
# 2 rows are returned
```

|  -- |
| --- |
| -- |

```go
INSERT INTO account VALUES('D',1000);
```

|

| -- |
| --- |

```go
COMMIT;
```

|

|

```go
SELECT * FROM account;
# 3 rows are returned because of MVCC
```

|  -- |
| --- |

|

```go
UPDATE account SET balance=1000 WHERE account_number='D';
# Surprisingly account D gets updated
```

|  -- |
| --- |

|

```go
SELECT * FROM account;
# Surprisingly 4 rows are returned
```

|  -- |
| --- |

# 可串行化

这提供了通过锁定所有被选中的行来提供最高级别的隔离。这个级别类似于`REPEATABLE READ`，但是如果禁用了自动提交，`InnoDB`会隐式地将所有普通的`SELECT`语句转换为`SELECT...LOCK IN SHARE MODE`。如果启用了自动提交，`SELECT`就是它自己的事务。

例如：

| **# 事务 1** | **# 事务 2** |
| --- | --- |

|

```go
BEGIN;
```

|

```go
BEGIN;
```

|

|

```go
SELECT * FROM account WHERE account_number='A';
```

|  -- |
| --- |
|  -- |

```go
UPDATE account SET balance=1000 WHERE account_number='A';
 # This will wait until the lock held by transaction 1
 on row A is released
```

|

|

```go
COMMIT;
```

| -- |
| --- |
|  -- |

```go
# UPDATE will be successful now
```

|

另一个例子：

| **# 事务 1** | **# 事务 2** |
| --- | --- |

|

```go
BEGIN;
```

|

```go
BEGIN;
```

|

|

```go
SELECT * FROM account WHERE account_number='A';
# Selects values of A
```

|  -- |
| --- |
|  -- |

```go
INSERT INTO account VALUES('D',2000);
# Inserts D
```

|

|

```go
SELECT * FROM account WHERE account_number='D';
 # This will wait until the transaction 2 completes
```

| -- |
| --- |
|  -- |

```go
COMMIT;
```

|

|

```go
# Now the preceding select statement returns values of D
```

| -- |
| --- |

因此，可串行化等待锁，并始终读取最新提交的数据。

# 锁定

有两种类型的锁定：

+   **内部锁定**：MySQL 在服务器内部执行内部锁定，以管理多个会话对表内容的争用

+   **外部锁定**：MySQL 为客户会话提供了显式获取表锁的选项，以防止其他会话访问表。

**内部锁定**：主要有两种类型的锁：

+   **行级锁定**：锁定粒度到行级。只有访问的行被锁定。这允许多个会话同时写入访问，使它们适用于多用户、高并发和 OLTP 应用程序。只有`InnoDB`支持行级锁定。

+   **表级锁定**：MySQL 对`MyISAM`、`MEMORY`和`MERGE`表使用表级锁定，每次只允许一个会话更新这些表。这种锁定级别使这些存储引擎更适合只读、读多或单用户应用程序。

参考[`dev.mysql.com/doc/refman/8.0/en/internal-locking.html`](https://dev.mysql.com/doc/refman/8.0/en/internal-locking.html)和[`dev.mysql.com/doc/refman/8.0/en/innodb-locking.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)以了解更多关于`InnoDB`锁的信息。

**外部锁定**：您可以使用`LOCK TABLE`和`UNLOCK TABLES`语句来控制锁。

对`READ`和`WRITE`的表锁定如下所述：

+   `READ`：当表被锁定为`READ`时，多个会话可以从表中读取数据而不需要获取锁。此外，多个会话可以在同一张表上获取锁，这就是为什么`READ`锁也被称为**共享锁**。当持有`READ`锁时，没有会话可以向表中写入数据（包括持有锁的会话）。如果有任何写入尝试，它将处于等待状态，直到`READ`锁被释放。

+   `WRITE`：当表被锁定为`WRITE`时，除了持有锁的会话外，没有其他会话可以从表中读取和写入数据。直到现有锁被释放，其他会话甚至无法获取任何锁。这就是为什么这被称为`独占锁`。如果有任何读/写尝试，它将处于等待状态，直到`WRITE`锁被释放。

当执行`UNLOCK TABLES`语句或会话终止时，所有锁都会被释放。

# 如何做...

语法如下：

```go
mysql> LOCK TABLES table_name [READ | WRITE]
```

要解锁表，请使用：

```go
mysql> UNLOCK TABLES;
```

要锁定所有数据库中的所有表，请执行以下语句。在对数据库进行一致快照时使用。它会冻结对数据库的所有写入：

```go
mysql> FLUSH TABLES WITH READ LOCK;
```

# 锁定队列

除了共享锁（一张表可以有多个共享锁）外，一张表上不能同时持有两个锁。如果一张表已经有一个共享锁，而独占锁来了，它将被保留在队列中，直到共享锁被释放。当独占锁在队列中时，所有后续的共享锁也会被阻塞并保留在队列中。

`InnoDB`在从表中读取/写入时会获取元数据锁。如果第二个事务请求`WRITE LOCK`，它将被保留在队列中，直到第一个事务完成。如果第三个事务想要读取数据，它必须等到第二个事务完成。

**事务 1：**

```go
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM employees LIMIT 10;
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
|  10001 | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
|  10002 | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 |
|  10003 | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 |
|  10004 | 1954-05-01 | Chirstian  | Koblick   | M      | 1986-12-01 |
|  10005 | 1955-01-21 | Kyoichi    | Maliniak  | M      | 1989-09-12 |
|  10006 | 1953-04-20 | Anneke     | Preusig   | F      | 1989-06-02 |
|  10007 | 1957-05-23 | Tzvetan    | Zielinski | F      | 1989-02-10 |
|  10008 | 1958-02-19 | Saniya     | Kalloufi  | M      | 1994-09-15 |
|  10009 | 1952-04-19 | Sumant     | Peac      | F      | 1985-02-18 |
|  10010 | 1963-06-01 | Duangkaew  | Piveteau  | F      | 1989-08-24 |
+--------+------------+------------+-----------+--------+------------+
10 rows in set (0.00 sec)
```

注意`COMMIT`没有被执行。事务保持打开状态。

**事务 2：**

```go
mysql> LOCK TABLE employees WRITE;
```

此语句必须等到事务 1 完成。

**事务 3：**

```go
mysql> SELECT * FROM employees LIMIT 10;
```

即使事务 3 也不会产生任何结果，因为一个排他锁在队列中（它在等待事务 2 完成）。此外，它会阻塞表上的所有操作。

您可以通过从另一个会话中检查`SHOW PROCESSLIST`来检查这一点：

```go
mysql> SHOW PROCESSLIST;
+----+------+-----------+-----------+---------+------+---------------------------------+----------------------------------+
| Id | User | Host      | db        | Command | Time | State                           | Info                             |
+----+------+-----------+-----------+---------+------+---------------------------------+----------------------------------+
| 20 | root | localhost | employees | Sleep   |   48 |                                 | NULL                             |
| 21 | root | localhost | employees | Query   |   34 | Waiting for table metadata lock | LOCK TABLE employees WRITE       |
| 22 | root | localhost | employees | Query   |   14 | Waiting for table metadata lock | SELECT * FROM employees LIMIT 10 |
| 23 | root | localhost | employees | Query   |    0 | starting                        | SHOW PROCESSLIST                 |
+----+------+-----------+-----------+---------+------+---------------------------------+----------------------------------+
4 rows in set (0.00 sec)
```

您可以注意到事务 2 和事务 3 都在等待事务 1。

要了解有关元数据锁的更多信息，请参考[`dev.mysql.com/doc/refman/8.0/en/metadata-locking.html`](https://dev.mysql.com/doc/refman/8.0/en/metadata-locking.html)。在使用`FLUSH TABLES WITH READ LOCK`时也可以观察到相同的行为。

**事务 1：**

```go
mysql> BEGIN;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM employees LIMIT 10;
+--------+------------+------------+-----------+--------+------------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  |
+--------+------------+------------+-----------+--------+------------+
|  10001 | 1953-09-02 | Georgi     | Facello   | M      | 1986-06-26 |
|  10002 | 1964-06-02 | Bezalel    | Simmel    | F      | 1985-11-21 |
|  10003 | 1959-12-03 | Parto      | Bamford   | M      | 1986-08-28 |
|  10004 | 1954-05-01 | Chirstian  | Koblick   | M      | 1986-12-01 |
|  10005 | 1955-01-21 | Kyoichi    | Maliniak  | M      | 1989-09-12 |
|  10006 | 1953-04-20 | Anneke     | Preusig   | F      | 1989-06-02 |
|  10007 | 1957-05-23 | Tzvetan    | Zielinski | F      | 1989-02-10 |
|  10008 | 1958-02-19 | Saniya     | Kalloufi  | M      | 1994-09-15 |
|  10009 | 1952-04-19 | Sumant     | Peac      | F      | 1985-02-18 |
|  10010 | 1963-06-01 | Duangkaew  | Piveteau  | F      | 1989-08-24 |
+--------+------------+------------+-----------+--------+------------+
10 rows in set (0.00 sec)
```

请注意，`COMMIT`没有被执行。事务保持打开状态。

**事务 2：**

```go
mysql> FLUSH TABLES WITH READ LOCK;
```

**事务 3：**

```go
mysql> SELECT * FROM employees LIMIT 10;
```

即使事务 3 也不会产生任何结果，因为`FLUSH TABLES`在获取锁之前需要等待表上的所有操作完成。此外，它会阻塞表上的所有操作。

您可以通过从另一个会话中检查`SHOW PROCESSLIST`来检查这一点。

```go
mysql> SHOW PROCESSLIST;
+----+------+-----------+-----------+---------+------+-------------------------+--------------------------------------------------+
| Id | User | Host      | db        | Command | Time | State                   | Info                                             |
+----+------+-----------+-----------+---------+------+-------------------------+--------------------------------------------------+
| 20 | root | localhost | employees | Query   |    7 | Creating sort index     | SELECT * FROM employees ORDER BY first_name DESC |
| 21 | root | localhost | employees | Query   |    5 | Waiting for table flush | FLUSH TABLES WITH READ LOCK                      |
| 22 | root | localhost | employees | Query   |    3 | Waiting for table flush | SELECT * FROM employees LIMIT 10                 |
| 23 | root | localhost | employees | Query   |    0 | starting                | SHOW PROCESSLIST                                 |
+----+------+-----------+-----------+---------+------+-------------------------+--------------------------------------------------+
4 rows in set (0.00 sec)
```

为了进行一致的备份，所有备份方法都使用`FLUSH TABLES WITH READ LOCK`，如果表上有长时间运行的事务，这可能非常危险。
