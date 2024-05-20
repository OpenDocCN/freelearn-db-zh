# 第十三章：性能调优

在本章中，我们将介绍以下内容：

+   解释计划

+   基准测试查询和服务器

+   添加索引

+   不可见索引

+   降序索引

+   使用 pt-query-digest 分析慢查询

+   优化数据类型

+   删除重复和冗余索引

+   检查索引使用

+   控制查询优化器

+   使用索引提示

+   使用生成列为 JSON 建立索引

+   使用资源组

+   使用 performance_schema

+   使用 sys 模式

# 介绍

本章将带您了解查询和模式调优。数据库是用于执行查询的，使其运行更快是调优的最终目标。数据库的性能取决于许多因素，主要是查询、模式、配置设置和硬件。

在本章中，我们将使用 employees 数据库来解释所有示例。您可能已经在前面的章节中以多种方式转换了 employees 数据库。建议在尝试本章中提到的示例之前，再次加载示例 employees 数据。您可以参考*第二章加载示例数据*了解如何加载示例数据。

# 解释计划

MySQL 执行查询的方式是数据库性能的主要因素之一。您可以使用`EXPLAIN`命令验证 MySQL 执行计划。从 MySQL 5.7.2 开始，您可以使用`EXPLAIN`来检查其他会话中当前执行的查询。`EXPLAIN FORMAT=JSON`提供了详细信息。

# 如何做...

让我们深入了解细节。

# 使用 EXPLAIN

解释计划提供了优化器执行查询的信息。您只需要在查询前加上`EXPLAIN`关键字：

```sql
mysql> EXPLAIN SELECT dept_name FROM dept_emp JOIN employees ON dept_emp.emp_no=employees.emp_no JOIN departments ON departments.dept_no=dept_emp.dept_no WHERE employees.first_name='Aamer'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: ref
possible_keys: PRIMARY,name
          key: name
      key_len: 58
          ref: const
         rows: 228
     filtered: 100.00
        Extra: Using index
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: dept_emp
   partitions: NULL
         type: ref
possible_keys: PRIMARY,dept_no
          key: PRIMARY
      key_len: 4
          ref: employees.employees.emp_no
         rows: 1
     filtered: 100.00
        Extra: Using index
*************************** 3\. row ***************************
           id: 1
  select_type: SIMPLE
        table: departments
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 16
          ref: employees.dept_emp.dept_no
         rows: 1
     filtered: 100.00
        Extra: NULL
3 rows in set, 1 warning (0.00 sec)
```

# 使用 EXPLAIN JSON

以 JSON 格式使用解释计划提供了关于查询执行的完整信息：

```sql
mysql> EXPLAIN FORMAT=JSON SELECT dept_name FROM dept_emp JOIN employees ON dept_emp.emp_no=employees.emp_no JOIN departments ON departments.dept_no=dept_emp.dept_no WHERE employees.first_name='Aamer'\G
*************************** 1\. row ***************************
EXPLAIN: {
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "286.13"
    },
    "nested_loop": [
      {
        "table": {
          "table_name": "employees",
          "access_type": "ref",
          "possible_keys": [
            "PRIMARY",
            "name"
          ],
          "key": "name",
          "used_key_parts": [
            "first_name"
          ],
          "key_length": "58",
          "ref": [
            "const"
          ],
          "rows_examined_per_scan": 228,
          "rows_produced_per_join": 228,
          "filtered": "100.00",
          "using_index": true,
          "cost_info": {
            "read_cost": "1.12",
            "eval_cost": "22.80",
            "prefix_cost": "23.92",
            "data_read_per_join": "30K"
          },
          "used_columns": [
            "emp_no",
            "first_name"
          ]
        }
      },
      {
        "table": {
          "table_name": "dept_emp",
          "access_type": "ref",
          "possible_keys": [
            "PRIMARY",
            "dept_no"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "emp_no"
          ],
          "key_length": "4",
          "ref": [
            "employees.employees.emp_no"
          ],
          "rows_examined_per_scan": 1,
          "rows_produced_per_join": 252,
          "filtered": "100.00",
          "using_index": true,
          "cost_info": {
            "read_cost": "148.78",
            "eval_cost": "25.21",
            "prefix_cost": "197.91",
            "data_read_per_join": "7K"
          },
          "used_columns": [
            "emp_no",
            "dept_no"
          ]
        }
      },
      {
        "table": {
          "table_name": "departments",
          "access_type": "eq_ref",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "PRIMARY",
          "used_key_parts": [
            "dept_no"
          ],
          "key_length": "16",
          "ref": [
            "employees.dept_emp.dept_no"
          ],
          "rows_examined_per_scan": 1,
          "rows_produced_per_join": 252,
          "filtered": "100.00",
          "cost_info": {
            "read_cost": "63.02",
            "eval_cost": "25.21",
            "prefix_cost": "286.13",
            "data_read_per_join": "45K"
          },
          "used_columns": [
            "dept_no",
            "dept_name"
          ]
        }
      }
    ]
  }
}
```

# 使用连接的 EXPLAIN

您可以为已运行的会话运行解释计划。您需要指定连接 ID：

要获取连接 ID，请执行：

```sql
mysql> SELECT CONNECTION_ID();
+-----------------+
| CONNECTION_ID() |
+-----------------+
|             778 |
+-----------------+
1 row in set (0.00 sec)
```

```sql
mysql> EXPLAIN FORMAT=JSON FOR CONNECTION 778\G
*************************** 1\. row ***************************
EXPLAIN: {
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "881.04"
    },
    "nested_loop": [
      {
        "table": {
          "table_name": "employees",
          "access_type": "index",
          "possible_keys": [
            "PRIMARY"
          ],
          "key": "name",
          "used_key_parts": [
            "first_name",
            "last_name"
          ],
          "key_length": "124",
          "rows_examined_per_scan": 1,
          "rows_produced_per_join": 1,
          "filtered": "100.00",
          "using_index": true,
          "cost_info": {
            "read_cost": "880.24",
            "eval_cost": "0.10",
            "prefix_cost": "880.34",
            "data_read_per_join": "136"
          },
~
~
1 row in set (0.00 sec)
```

如果连接没有运行任何`SELECT`/`UPDATE`/`INSERT`/`DELETE`/`REPLACE`查询，它将抛出一个错误：

```sql
mysql> EXPLAIN FOR CONNECTION 779;
ERROR 3012 (HY000): EXPLAIN FOR CONNECTION command is supported only for SELECT/UPDATE/INSERT/DELETE/REPLACE
```

请参阅[`dev.mysql.com/doc/refman/8.0/en/explain-output.html`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)了解更多关于解释计划格式的信息。JSON 格式在[`www.percona.com/blog/category/explain-2/explain-formatjson-is-cool/`](https://www.percona.com/blog/category/explain-2/explain-formatjson-is-cool/)中有非常清晰的解释。

# 基准测试查询和服务器

假设您想要找出哪个查询更快。解释计划给了您一个想法，但有时您不能根据它来决定。如果查询时间在几十秒的数量级上，您可以在服务器上执行它们，并找出哪个更快。但是，如果查询时间在几毫秒的数量级上，您不能仅根据单次执行来决定。

您可以使用`mysqlslap`实用程序（它随 MySQL 客户端安装一起提供），它模拟了 MySQL 服务器的客户端负载，并报告每个阶段的时间。它的工作方式就好像多个客户端正在访问服务器。在本节中，您将了解`mysqlslap`的用法；在后续章节中，您将了解`mysqlslap`的强大之处。

# 如何做...

假设您想要测量查询的时间；如果您在 MySQL 客户端中执行它，您可以知道大约的执行时间，精度为 100 毫秒：

```sql
mysql> pager grep rows
PAGER set to 'grep rows'
mysql> SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE (first_name='Adam');
2384 rows in set (0.00 sec)
```

您可以使用`mysqlslap`模拟客户端负载，并在多次迭代中同时运行前述 SQL：

```sql
shell> mysqlslap -u <user> -p<pass> --create-schema=employees --query="SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE (first_name='Adam');" -c 1000 i 100
mysqlslap: [Warning] Using a password on the command line interface can be insecure.
Benchmark
    Average number of seconds to run all queries: 3.216 seconds
    Minimum number of seconds to run all queries: 3.216 seconds
    Maximum number of seconds to run all queries: 3.216 seconds
    Number of clients running queries: 1000
    Average number of queries per client: 1
```

前述查询以 1,000 并发和 100 次迭代执行，平均耗时 3.216 秒。

您可以在文件中指定多个 SQL 并指定分隔符。`mysqlslap`运行文件中的所有查询：

```sql
shell> cat queries.sql
SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE (first_name='Adam');
SELECT * FROM employees WHERE first_name='Adam' OR last_name='Adam';
SELECT * FROM employees WHERE first_name='Adam';shell> mysqlslap -u <user> -p<pass> --create-schema=employees --concurrency=10 --iterations=10 --query=query.sql --query=queries.sql --delimiter=";"
mysqlslap: [Warning] Using a password on the command line interface can be insecure.
Benchmark
    Average number of seconds to run all queries: 5.173 seconds
    Minimum number of seconds to run all queries: 5.010 seconds
    Maximum number of seconds to run all queries: 5.257 seconds
    Number of clients running queries: 10
    Average number of queries per client: 3
```

您甚至可以自动生成表和 SQL 语句。这样，您可以将结果与先前的服务器设置进行比较：

```sql
shell> mysqlslap -u <user> -p<pass> --concurrency=100 --iterations=10 --number-int-cols=4 --number-char-cols=10  --auto-generate-sql
mysqlslap: [Warning] Using a password on the command line interface can be insecure.
Benchmark
    Average number of seconds to run all queries: 1.640 seconds
    Minimum number of seconds to run all queries: 1.511 seconds
    Maximum number of seconds to run all queries: 1.791 seconds
    Number of clients running queries: 100
    Average number of queries per client: 0
```

您还可以使用`performance_schema`来获取所有与查询相关的指标，这在*使用 performance_schema*部分有解释。

# 添加索引

没有索引，MySQL 必须逐行扫描整个表来找到相关的行。如果表上有你正在过滤的列的索引，MySQL 可以快速在大数据文件中找到行，而不必扫描整个文件。

MySQL 可以在`WHERE`、`ORDER BY`和`GROUP BY`子句中使用索引来过滤行，也可以用于连接表。如果一个列上有多个索引，MySQL 会选择能够最大程度过滤行的索引。

你可以执行`ALTER TABLE`命令来添加或删除索引。索引的添加和删除都是在线操作，不会影响表上的 DML 操作，但在较大的表上需要很长时间。

# 主键（聚集索引）和次要索引

在继续之前，重要的是要理解主键（或聚集索引）是什么，以及次要索引是什么。

`InnoDB`按照主键的顺序存储行，以加快涉及主键列的查询和排序。这也被称为**索引组织表**，在 Oracle 术语中。所有其他索引都被称为次要键，它们存储主键的值（它们不直接引用行）。

假设表是：

```sql
mysql> CREATE TABLE index_example ( 
col1 int PRIMARY KEY,
col2 char(10),
KEY `col2`(`col2`)
);
```

表行根据`col1`的值进行排序和存储。如果你搜索`col1`的任何值，它可以直接指向物理行；这就是为什么聚集索引非常快。`col2`上的索引也包含`col1`的值，如果你搜索`col2`，`col1`的值会返回，然后在聚集索引中搜索实际行。

选择主键的提示：

+   它应该是`UNIQUE`和`NOT NULL`。

+   选择尽可能小的键，因为所有的次要索引都存储主键。所以如果它很大，整体索引大小会占用更多的空间。

+   选择一个单调递增的值。物理行是根据主键排序的。所以如果你选择一个随机键，需要更多的行重新排列，这会导致性能下降。`AUTO_INCREMENT`是主键的完美选择。

+   始终选择一个主键；如果找不到任何主键，添加一个`AUTO_INCREMENT`列。如果你不选择任何主键，`InnoDB`内部会生成一个隐藏的聚集索引，带有 6 字节的行 ID。

# 如何做…

你可以通过查看表的定义来查看表的索引。你会注意到`first_name`和`last_name`上有一个索引。如果你通过指定`first_name`或者两者（`first_name`和`last_name`）来过滤行，MySQL 可以使用索引来加快查询。然而，如果你只指定`last_name`，索引就不能被使用；这是因为优化器只能使用索引的最左边的前缀。更详细的例子请参考[`dev.mysql.com/doc/refman/8.0/en/multiple-column-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/multiple-column-indexes.html)：

```sql
mysql> ALTER TABLE employees ADD INDEX name(first_name, last_name);
Query OK, 0 rows affected (2.23 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`),
 KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

# 添加索引

你可以通过执行`ALTER TABLE ADD INDEX`命令添加索引。例如，如果你想在`last_name`上添加一个索引，请参考以下代码：

```sql
mysql> ALTER TABLE employees ADD INDEX (last_name);
Query OK, 0 rows affected (1.28 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`),
  KEY `name` (`first_name`,`last_name`),
 KEY `last_name` (`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.01 sec)
```

你可以指定索引的名称；如果不指定，最左边的前缀将被用作名称。如果有任何重复，名称将被附加`_2`、`_3`等。

例如：

```sql
mysql> ALTER TABLE employees ADD INDEX index_last_name (last_name);
```

# 唯一索引

如果你希望索引是唯一的，可以指定关键字`UNIQUE`。例如：

```sql
mysql> ALTER TABLE employees ADD UNIQUE INDEX unique_name (last_name, first_name);
# There are few duplicate entries in employees database, the above statement is shown for illustration purpose only.
```

# 前缀索引

对于字符串列，可以创建只使用列值前导部分而不是整个列的索引。你需要指定前导部分的长度：

```sql
## `last_name` varchar(16) NOT NULL
mysql> ALTER TABLE employees ADD INDEX (last_name(10));
Query OK, 0 rows affected (1.78 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

`last_name`的最大长度是`16`个字符，但索引只在前 10 个字符上创建。

# 删除索引

你可以使用`ALTER TABLE`命令删除索引：

```sql
mysql> ALTER TABLE employees DROP INDEX last_name;
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

# 在生成的列上创建索引

不能在函数中使用列上的索引。假设你在`hire_date`上添加了一个索引：

```sql
mysql> ALTER TABLE employees ADD INDEX(hire_date);
Query OK, 0 rows affected (0.93 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

在`WHERE`子句中有`hire_date`的查询可以使用`hire_date`上的索引：

```sql
mysql> EXPLAIN SELECT COUNT(*) FROM employees WHERE hire_date>'2000-01-01'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: range
possible_keys: hire_date
          key: hire_date
      key_len: 3
          ref: NULL
         rows: 14
     filtered: 100.00
 Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)
```

相反，如果将`hire_date`放在函数中，MySQL 必须扫描整个表：

```sql
mysql> EXPLAIN SELECT COUNT(*) FROM employees WHERE YEAR(hire_date)>=2000\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index
possible_keys: NULL
          key: hire_date
      key_len: 3
          ref: NULL
 rows: 291892
     filtered: 100.00
        Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)
```

因此，尽量避免在函数内部放置一个带索引的列。如果无法避免使用函数，则创建一个虚拟列并在虚拟列上添加索引：

```sql
mysql> ALTER TABLE employees ADD hire_date_year YEAR AS (YEAR(hire_date)) VIRTUAL, ADD INDEX (hire_date_year);
Query OK, 0 rows affected (1.16 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
 `hire_date_year` year(4) GENERATED ALWAYS AS (year(`hire_date`)) VIRTUAL,
  PRIMARY KEY (`emp_no`),
  KEY `name` (`first_name`,`last_name`),
  KEY `hire_date` (`hire_date`),
 KEY `hire_date_year` (`hire_date_year`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

现在，不再在查询中使用`YEAR()`函数，而是可以直接在`WHERE`子句中使用`hire_date_year`：

```sql
mysql> EXPLAIN SELECT COUNT(*) FROM employees WHERE hire_date_year>=2000\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: range
possible_keys: hire_date_year
 key: hire_date_year
      key_len: 2
          ref: NULL
 rows: 15
     filtered: 100.00
 Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)
```

请注意，即使使用`YEAR(hire_date)`，优化器也会认识到`YEAR()`表达式与`hire_date_year`的定义相匹配，并且`hire_date_year`被索引；因此在执行计划构建过程中会考虑该索引：

```sql
mysql> EXPLAIN SELECT COUNT(*) FROM employees WHERE YEAR(hire_date)>=2000\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: range
possible_keys: hire_date_year
 key: hire_date_year
      key_len: 2
          ref: NULL
         rows: 15
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

# 不可见索引

如果要删除未使用的索引，而不是立即删除，可以将其标记为不可见，监视应用程序行为，然后再删除。以后，如果需要该索引，可以将其标记为可见，这与删除和重新添加索引相比非常快。

解释不可见索引，如果尚未添加普通索引，则需要添加普通索引。例如：

```sql
mysql> ALTER TABLE employees ADD INDEX (last_name);
Query OK, 0 rows affected (1.81 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

# 如何做...

如果您希望删除`last_name`上的索引，而不是直接删除，可以使用`ALTER TABLE`命令将其标记为不可见：

```sql
mysql> EXPLAIN SELECT * FROM employees WHERE last_name='Aamodt'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: ref
possible_keys: last_name
          key: last_name
      key_len: 66
          ref: const
         rows: 205
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)

mysql> ALTER TABLE employees ALTER INDEX last_name INVISIBLE;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> EXPLAIN SELECT * FROM employees WHERE last_name='Aamodt'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 299733
     filtered: 10.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)

mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`),
  KEY `name` (`first_name`,`last_name`),
 KEY `last_name` (`last_name`) /*!80000 INVISIBLE */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

您会注意到通过`last_name`进行查询过滤时使用了`last_name`索引；标记为不可见后，它无法使用。您可以再次将其标记为可见：

```sql
mysql> ALTER TABLE employees ALTER INDEX last_name VISIBLE;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

# 降序索引

在 MySQL 8 之前，索引定义可以包含顺序（升序或降序），但它只是被解析而不是被实现。索引值总是以升序存储的。MySQL 8.0 引入了对降序索引的支持。因此，索引定义中指定的顺序不会被忽略。降序索引实际上以降序存储键值。请记住，对降序查询来说，以相反的方式扫描升序索引是低效的。

考虑这样一种情况，在多列索引中，可以指定某些列为降序。这对于查询中有升序和降序`ORDER BY`子句的查询很有帮助。

假设您想对`employees`表进行排序，按`first_name`升序和`last_name`降序；MySQL 无法使用`first_name`和`last_name`上的索引。没有降序索引：

```sql
mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`),
 KEY `name` (`first_name`,`last_name`),
  KEY `last_name` (`last_name`) /*!80000 INVISIBLE */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

在解释计划中，您会注意到索引名称（`first_name`和`last_name`）未被使用：

```sql
mysql> EXPLAIN SELECT * FROM employees ORDER BY first_name ASC, last_name DESC LIMIT 10\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 299733
     filtered: 100.00
        Extra: Using filesort
```

# 如何做...

1.  添加一个降序索引：

```sql
mysql> ALTER TABLE employees ADD INDEX name_desc(first_name ASC, last_name DESC);
Query OK, 0 rows affected (1.61 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

1.  添加降序索引后，查询可以使用索引：

```sql
mysql> EXPLAIN SELECT * FROM employees ORDER BY first_name ASC, last_name DESC LIMIT 10\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index
possible_keys: NULL
 key: name_desc
      key_len: 124
          ref: NULL
         rows: 10
     filtered: 100.00
        Extra: NULL
```

1.  相同的索引可以用于另一种排序方式，即通过反向索引扫描按`first_name`降序和`last_name`升序排序：

```sql
mysql> EXPLAIN SELECT * FROM employees ORDER BY first_name DESC, last_name ASC LIMIT 10\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index
possible_keys: NULL
          key: name_desc
      key_len: 124
          ref: NULL
         rows: 10
     filtered: 100.00
 Extra: Backward index scan
```

# 使用`pt-query-digest`分析慢查询

`pt-query-digest`是 Percona Toolkit 的一部分，用于分析查询。可以通过以下任何一种方式收集查询：

+   慢查询日志

+   一般查询日志

+   进程列表

+   二进制日志

+   TCP 转储

Percona Toolkit 的安装在第十章中有介绍，*表维护*，*安装 Percona Toolkit*部分。在本节中，您将学习如何使用`pt-query-digest`。每种方法都有缺点。慢查询日志不包括所有查询，除非指定`long_query_time`为`0`，这会严重减慢系统。一般查询日志不包括查询时间。您无法从进程列表中获取完整的查询。只能使用二进制日志分析写入，并且使用 TCP 转储会导致服务器降级。通常，此工具用于具有 1 秒或更高`long_query_time`的慢查询日志。

# 如何做...

让我们详细分析使用`pt-query-digest`分析慢查询的细节。

# 慢查询日志

启用和配置慢查询日志在第十二章中有解释，*管理日志*，*管理一般查询日志和慢查询日志*。一旦启用慢查询日志并收集查询，就可以通过传递慢查询日志来运行`pt-query-digest`。

假设慢查询文件位于`/var/lib/mysql/mysql-slow.log`：

```sql
shell> sudo pt-query-digest /var/lib/mysql/ubuntu-slow.log > query_digest
```

摘要报告包含按查询执行次数乘以查询时间排名的查询。摘要中显示了查询的详细信息，如查询校验和（每种查询的唯一值）、平均时间、百分比时间和执行次数。您可以通过搜索查询校验和来深入了解特定查询。

摘要报告如下所示：

```sql
# 286.8s user time, 850ms system time, 232.75M rss, 315.73M vsz
# Current date: Sat Nov 18 05:16:55 2017
# Hostname: db1
# Files: /var/lib/mysql/db1-slow.log
# Rate limits apply
# Overall: 638.54k total, 2.06k unique, 0.49 QPS, 0.14x concurrency ______
# Time range: 2017-11-03 01:02:40 to 2017-11-18 05:16:47
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time        179486s     3us   2713s   281ms    21ms     15s   176us
# Lock time          1157s       0     36s     2ms   194us   124ms    49us
# Rows sent         18.25M       0 753.66k   29.96  212.52   1.63k    0.99
# Rows examine     157.39G       0   3.30G 258.45k   3.35k  24.78M    0.99
# Rows affecte       3.66M       0 294.77k    6.01    0.99   1.16k       0
# Bytes sent         3.08G       0  95.15M   5.05k  13.78k 206.42k  174.84
# Merge passes       2.84k       0      97    0.00       0    0.16       0
# Tmp tables       129.02k       0    1009    0.21    0.99    1.43       0
# Tmp disk tbl      25.20k       0     850    0.04       0    1.09       0
# Tmp tbl size      26.21G       0 218.27M  43.04k       0   2.06M       0
# Query size       178.92M       6 452.25k  293.81  592.07   5.26k   72.65
# InnoDB:
# IO r bytes        79.06G       0   2.09G 200.37k       0  12.94M       0
# IO r ops           7.26M       0 233.16k   18.39       0   1.36k       0
# IO r wait         96525s       0   3452s   233ms       0     18s       0
# pages distin     526.99M       0 608.33k   1.30k  964.41   9.15k    1.96
# queue wait             0       0       0       0       0       0       0
# rec lock wai         46s       0      9s   111us       0    28ms       0
# Boolean:
# Filesort       5% yes,  94% no
# Filesort on    0% yes,  99% no
# Full join      3% yes,  96% no
# Full scan     40% yes,  59% no
# Tmp table     13% yes,  86% no
# Tmp table on   2% yes,  97% no
```

查询概要将如下所示：

```sql
# Rank Query ID           Response time    Calls  R/Call    V/M   Item
# ==== ================== ================ ====== ========= ===== ========
#    1 0x55F499860A034BCB 76560.4220 42.7%     47 1628.9451 18.06 SELECT orders 
#    2 0x3A2F0B98DA39BCB9 10490.4155  5.8%   2680    3.9143 33... SELECT orders order_status 
#    3 0x25119C7C31A24011  7378.8763  4.1%   1534    4.8102 30.11 SELECT orders users 
#    4 0x41106CE92AD9DFED  5412.7326  3.0%  15589    0.3472  2.98 SELECT sessions
#    5 0x860DCDE7AE0AD554  5187.5257  2.9%    500   10.3751 54.99 SELECT orders sessions 
#    6 0x5DF64920B008AD63  4517.5041  2.5%     58   77.8880 22.23 UPDATE SELECT 
#    7 0xC9F9A31DE77B93A1  4473.0208  2.5%     58   77.1210 96... INSERT SELECT tmpMove 
#    8 0x8BF88451DA989BFF  4036.4413  2.2%     13  310.4955 16... UPDATE SELECT orders tmpDel
```

从前面的输出中，您可以推断出对于查询`#1`（`0x55F499860A034BCB`），所有执行的累积响应时间为`76560`秒。这占所有查询的累积响应时间的 42.7%。执行次数为 47，平均查询时间为`1628`秒。

您可以通过搜索校验和来查找任何查询。完整的查询、解释计划的命令和表状态都会显示出来。例如：

```sql
# Query 1: 0.00 QPS, 0.06x concurrency, ID 0x55F499860A034BCB at byte 249542900
# This item is included in the report because it matches --limit.
# Scores: V/M = 18.06
# Time range: 2017-11-03 01:39:19 to 2017-11-18 01:46:50
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          0      47
# Exec time     42  76560s   1182s   1854s   1629s   1819s    172s   1649s
# Lock time      0      3s   102us   994ms    70ms   293ms   174ms   467us
# Rows sent      0  78.78k     212   5.66k   1.68k   4.95k   1.71k  652.75
# Rows examine  85 135.34G   2.11G   3.30G   2.88G   3.17G 303.82M   2.87G
# Rows affecte   0       0       0       0       0       0       0       0
# Bytes sent     0   3.22M  10.20k 226.13k  70.14k 201.74k  66.71k  31.59k
# Merge passes   0       0       0       0       0       0       0       0
# Tmp tables     0       0       0       0       0       0       0       0
# Tmp disk tbl   0       0       0       0       0       0       0       0
# Tmp tbl size   0       0       0       0       0       0       0       0
# Query size     0  11.66k     254     254     254     254       0     254
# InnoDB:
# IO r bytes     1   1.11G       0  53.79M  24.20M  51.29M  21.04M  20.30M
# IO r ops       1 142.14k       0   6.72k   3.02k   6.63k   2.67k   2.50k
# IO r wait      0     92s       0     14s      2s      5s      3s      1s
# pages distin   0 325.46k   6.10k   7.30k   6.92k   6.96k  350.84   6.96k
# queue wait     0       0       0       0       0       0       0       0
# rec lock wai   0       0       0       0       0       0       0       0
# Boolean:
# Full scan    100% yes,   0% no
# String:
# Databases    lashrenew_... (32/68%), betsy_db (15/31%)
# Hosts        10.37.69.197
# InnoDB trxID CF22C985 (1/2%), CF23455A (1/2%)... 45 more
# Last errno   0
# rate limit   query:100
# Users        db1_... (32/68%), dba (15/31%)
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms
#  10ms
# 100ms
#    1s
#  10s+  ################################################################
# Tables
#    SHOW TABLE STATUS FROM `db1` LIKE 'orders'\G
#    SHOW CREATE TABLE `db1`.`orders`\G
#    SHOW TABLE STATUS FROM `db1` LIKE 'shipping_tracking_history'\G
#    SHOW CREATE TABLE `db1`.`shipping_tracking_history`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT  tracking_num, carrier, order_id, userID FROM orders o WHERE tracking_num!="" 
and NOT EXISTS (SELECT 1 FROM shipping_tracking_history sth WHERE sth.order_id=o.order_id AND sth.is_final=1)
AND o.date_finalized>date_add(curdate(),interval -1 month)\G
```

# 常规查询日志

您可以通过传递参数`--type genlog`来使用`pt-query-digest`分析常规查询日志。由于常规日志不报告查询时间，因此只显示计数聚合：

```sql
 shell> sudo pt-query-digest --type genlog /var/lib/mysql/db1.log   > general_query_digest
```

输出将类似于这样：

```sql
# 400ms user time, 0 system time, 28.84M rss, 99.35M vsz
# Current date: Sat Nov 18 09:02:08 2017
# Hostname: db1
# Files: /var/lib/mysql/db1.log
# Overall: 511 total, 39 unique, 30.06 QPS, 0x concurrency _______________
# Time range: 2017-11-18 09:01:09 to 09:01:26
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time              0       0       0       0       0       0       0
# Query size        92.18k      10   3.22k  184.71  363.48  348.86  102.22
```

查询概要将类似于这样：

```sql
# Profile
# Rank Query ID           Response time Calls R/Call V/M   Item
# ==== ================== ============= ===== ====== ===== ===============
#    1 0x625BF8F82D174492  0.0000  0.0%   130 0.0000  0.00 SELECT facebook_like_details
#    2 0xAA353644DE4C4CB4  0.0000  0.0%    44 0.0000  0.00 ADMIN QUIT
#    3 0x5D51E5F01B88B79E  0.0000  0.0%    44 0.0000  0.00 ADMIN CONNECT
```

# 进程列表

您可以使用`pt-query-digest`从进程列表中读取查询，而不是使用日志文件：

```sql
shell> pt-query-digest --processlist h=localhost  --iterations 10 --run-time 1m -u <user> -p<pass>
```

`run-time`指定每次迭代应运行多长时间。在前面的示例中，该工具将在 10 分钟内每分钟生成一次报告。

# 二进制日志

要使用`pt-query-digest`分析二进制日志，您应该使用`mysqlbinlog`实用程序将其转换为文本格式：

```sql
shell> sudo mysqlbinlog /var/lib/mysql/binlog.000639 > binlog.00063

shell> pt-query-digest --type binlog binlog.000639  > binlog_digest
```

# TCP 转储

您可以使用`tcpdump`命令捕获 TCP 流量，并将其发送到`pt-query-digest`进行分析：

```sql
shell> sudo tcpdump -s 65535 -x -nn -q -tttt -i any -c 1000 port 3306 > mysql.tcp.txt

shell> pt-query-digest --type tcpdump mysql.tcp.txt > tcpdump_digest
```

`pt-query-digest`中有很多选项可用，例如过滤特定时间窗口的查询、过滤特定查询和生成报告。有关更多详细信息，请参阅 Percona 文档[`www.percona.com/doc/percona-toolkit/LATEST/pt-query-digest.html`](https://www.percona.com/doc/percona-toolkit/LATEST/pt-query-digest.html)。

# 另请参阅

请参阅[`engineering.linkedin.com/blog/2017/09/query-analyzer--a-tool-for-analyzing-mysql-queries-without-overh`](https://engineering.linkedin.com/blog/2017/09/query-analyzer--a-tool-for-analyzing-mysql-queries-without-overh)以了解有关分析所有查询的新方法而无需任何开销的更多信息。

# 优化数据类型

您应该定义表，使其在磁盘上占用最少的空间，同时容纳所有可能的值。

如果大小较小：

+   写入或从磁盘读取的数据较少，这使得查询更快。

+   在处理查询时，磁盘上的内容会加载到主内存中。因此，较小的表在主内存中占用的空间较小。

+   索引占用的空间较少。

# 如何做…

1.  如果您想存储最大可能值为 500,000 的员工编号，则最佳数据类型是`MEDIUMINT UNSIGNED`（占用 3 个字节）。如果您将其存储为占用 4 个字节的`INT`，则每行都会浪费一个字节。

1.  如果您想存储长度可变且最大可能值为 20 的名字，最好将其声明为`varchar(20)`。如果您将其存储为`char(20)`，并且只有少数名字长度为 20 个字符，而其余名字长度都不到 10 个字符，那么您就浪费了 10 个字符的空间。

1.  在声明`varchar`列时，您应该考虑长度。虽然`varchar`在磁盘上进行了优化，但在加载到内存时，它会占用全部长度。例如，如果您将`first_name`存储在`varchar(255)`中，实际长度为 10，在磁盘上占用 10+1（用于存储长度的额外字节）；但在内存中，它将占用 255 个字节的全部长度。

1.  如果`varchar`列的长度超过 255 个字符，则需要 2 个字节来存储长度。

1.  如果不存储空值，请将列声明为`NOT NULL`。这样可以避免测试每个值是否为空的开销，还可以节省一些存储空间：每列 1 位。

1.  如果长度是固定的，请使用`char`而不是`varchar`，因为`varchar`需要一个或两个字节来存储字符串的长度。

1.  如果值是固定的，请使用`ENUM`而不是`varchar`。例如，如果要存储可以是 pending、approved、rejected、deployed、undeployed、failed 或 deleted 的值，可以使用`ENUM`。它只占用 1 或 2 个字节，而`char(10)`占用 10 个字节。

1.  优先使用整数而不是字符串。

1.  尝试利用前缀索引。

1.  尝试利用`InnoDB`压缩。

参考[`dev.mysql.com/doc/refman/8.0/en/storage-requirements.html`](https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html)了解每种数据类型的存储要求，以及[`dev.mysql.com/doc/refman/8.0/en/integer-types.html`](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)了解每种整数类型的范围。

如果您想知道优化的数据类型，可以使用`PROCEDURE ANALYZE`函数。虽然不太准确，但可以对字段有一个大致的了解。不幸的是，它在 MySQL 8 中已被弃用：

```sql
mysql> SELECT user_id, first_name FROM user PROCEDURE ANALYSE(1,100)\G
*************************** 1\. row ***************************
             Field_name: db1.user.user_id
              Min_value: 100000@nat.test123.net
              Max_value: test1234@nat.test123.net
             Min_length: 22
             Max_length: 33
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 25.8003
                    Std: NULL
      Optimal_fieldtype: VARCHAR(33) NOT NULL
*************************** 2\. row ***************************
             Field_name: db1.user.first_name
              Min_value: *Alan
              Max_value: Zuniga 102031
             Min_length: 3
             Max_length: 33
       Empties_or_zeros: 0
                  Nulls: 0
Avg_value_or_avg_length: 10.1588
                    Std: NULL
      Optimal_fieldtype: VARCHAR(33) NOT NULL
2 rows in set (0.02 sec)
```

# 删除重复和冗余索引

您可以在一列上定义多个索引。可能会错误地再次定义相同的索引（相同的列，相同的列顺序或相同的键顺序），这被称为**重复索引**。如果只有部分索引（最左边的列）是重复的，则称为**冗余索引**。重复索引没有优势。冗余索引在某些情况下可能有用（在本节末尾的注释中提到了一个用例），但两者都会减慢插入速度。因此，识别并删除它们非常重要。

有三个工具可以帮助找出重复的索引：

+   `pt-duplicate-key-checker`是 Percona Toolkit 的一部分。安装 Percona Toolkit 在第十章，*表维护*，*安装 Percona Toolkit*部分有介绍。

+   `mysqlindexcheck`是 MySQL 实用程序的一部分。安装 MySQL 实用程序在第一章中有介绍，*MySQL 8.0 – 安装和升级*。

+   使用`sys`模式，这将在下一节中介绍。

考虑以下`employees`表：

```sql
mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`),
  KEY `last_name` (`last_name`) /*!80000 INVISIBLE */,
  KEY `full_name` (`first_name`,`last_name`),
  KEY `full_name_desc` (`first_name` DESC,`last_name`),
  KEY `first_name` (`first_name`),
  KEY `full_name_1` (`first_name`,`last_name`),
  KEY `first_name_emp_no` (`first_name`,`emp_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

索引`full_name_1`是`full_name`的重复，因为两个索引都在相同的列上，列的顺序相同，键的顺序也相同（升序或降序）。

索引`first_name`是冗余索引，因为列`first_name`已经包含在`first_name`索引的最左边的后缀中。

索引`first_name_emp_no`是冗余索引，因为它包含了最右边的后缀中的主键。`InnoDB`的次要索引已经包含了主键，因此在次要索引中声明主键是多余的。但是，在过滤`first_name`并按`emp_no`排序的查询中可能会有用：

```sql
SELECT * FROM employees WHERE first_name='Adam' ORDER BY emp_no;
```

`full_name_desc`选项不是`full_name`的重复，因为键的排序不同。

# 如何做...

让我们详细了解一下删除重复和冗余索引。

# pt-duplicate-key-checker

`pt-duplicate-key-checker`给出了删除重复键的确切`ALTER`语句：

```sql
shell> pt-duplicate-key-checker -u <user> -p<pass>

# A software update is available:
# ########################################################################
# employees.employees                                                     
# ########################################################################

# full_name_1 is a duplicate of full_name
# Key definitions:
#   KEY `full_name_1` (`first_name`,`last_name`),
#   KEY `full_name` (`first_name`,`last_name`),
# Column types:
#      `first_name` varchar(14) not null
#      `last_name` varchar(16) not null
# To remove this duplicate index, execute:
ALTER TABLE `employees`.`employees` DROP INDEX `full_name_1`;

# first_name is a left-prefix of full_name
# Key definitions:
#   KEY `first_name` (`first_name`),
#   KEY `full_name` (`first_name`,`last_name`),
# Column types:
#      `first_name` varchar(14) not null
#      `last_name` varchar(16) not null
# To remove this duplicate index, execute:
ALTER TABLE `employees`.`employees` DROP INDEX `first_name`;

# Key first_name_emp_no ends with a prefix of the clustered index
# Key definitions:
#   KEY `first_name_emp_no` (`first_name`,`emp_no`)
#   PRIMARY KEY (`emp_no`),
# Column types:
#      `first_name` varchar(14) not null
#      `emp_no` int(11) not null
# To shorten this duplicate clustered index, execute:
ALTER TABLE `employees`.`employees` DROP INDEX `first_name_emp_no`, ADD INDEX `first_name_emp_no` (`first_name`);
```

该工具建议您通过从最右边的后缀中删除`PRIMARY KEY`来缩短重复的聚集索引。请注意，这可能会导致另一个重复的索引。如果您希望忽略重复的聚集索引，可以传递`--noclustered`选项。

要检查特定数据库的重复索引，可以传递`--databases <database name>`选项：

```sql
shell> pt-duplicate-key-checker -u <user> -p<pass> --database employees
```

要删除键，甚至可以将`pt-duplicate-key-checker`的输出导入到`mysql`中：

```sql
shell> pt-duplicate-key-checker -u <user> -p<pass> | mysql -u <user> -p<pass>
```

# mysqlindexcheck

请注意，`mysqlindexcheck`忽略降序索引。例如，`full_name_desc`（`first_name`降序和`last_name`）被视为`full_name`（`first_name`和`last_name`）的重复索引：

```sql
shell> mysqlindexcheck --server=<user>:<pass>@localhost:3306 employees --show-drops 
WARNING: Using a password on the command line interface can be insecure.
# Source on localhost: ... connected.
# The following indexes are duplicates or redundant for table employees.employees:
#
CREATE INDEX `full_name_desc` ON `employees`.`employees` (`first_name`, `last_name`) USING BTREE
#     may be redundant or duplicate of:
CREATE INDEX `full_name` ON `employees`.`employees` (`first_name`, `last_name`) USING BTREE
#
CREATE INDEX `first_name` ON `employees`.`employees` (`first_name`) USING BTREE
#     may be redundant or duplicate of:
CREATE INDEX `full_name` ON `employees`.`employees` (`first_name`, `last_name`) USING BTREE
#
CREATE INDEX `full_name_1` ON `employees`.`employees` (`first_name`, `last_name`) USING BTREE
#     may be redundant or duplicate of:
CREATE INDEX `full_name` ON `employees`.`employees` (`first_name`, `last_name`) USING BTREE
#
# DROP statements:
#
ALTER TABLE `employees`.`employees` DROP INDEX `full_name_desc`;
ALTER TABLE `employees`.`employees` DROP INDEX `first_name`;
ALTER TABLE `employees`.`employees` DROP INDEX `full_name_1`;
#
# The following index for table employees.employees contains the clustered index and might be redundant:
#
CREATE INDEX `first_name_emp_no` ON `employees`.`employees` (`first_name`, `emp_no`) USING BTREE
#
# DROP/ADD statement:
#
ALTER TABLE `employees`.`employees` DROP INDEX `first_name_emp_no`, ADD INDEX `first_name_emp_no` (first_name);
#
```

如前所述，多余的索引在某些情况下可能有用。您必须考虑您的应用程序是否需要这些情况。

创建索引以了解以下示例

```sql
mysql> ALTER TABLE employees DROP PRIMARY KEY, ADD PRIMARY KEY(emp_no, hire_date), ADD INDEX `name` (`first_name`,`last_name`);

mysql> ALTER TABLE salaries ADD INDEX from_date(from_date), ADD INDEX from_date_2(from_date,emp_no);
```

考虑以下`employees`和`salaries`表：

```sql
mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`,`hire_date`),
  KEY `name` (`first_name`,`last_name`)
) /*!50100 TABLESPACE `innodb_system` */ ENGINE=InnoDB DEFAULT CHARSET=utf8mb4

mysql> SHOW CREATE TABLE salaries\G
*************************** 1\. row ***************************
Create Table: CREATE TABLE `salaries` (
 `emp_no` int(11) NOT NULL,
 `salary` int(11) NOT NULL,
 `from_date` date NOT NULL,
 `to_date` date NOT NULL,
 PRIMARY KEY (`emp_no`,`from_date`),
 KEY `from_date` (`from_date`),
 KEY `from_date_2` (`from_date`,`emp_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

似乎`from_date`是`from_date_2`的多余索引，但是检查以下查询的解释计划！它使用了两个索引的交集。`from_date`索引用于过滤，`from_date_2`用于与`employees`表进行连接。优化器在每个表中只扫描一行：

```sql
mysql> EXPLAIN SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE from_date='2001-05-23'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: s
   partitions: NULL
         type: index_merge
possible_keys: PRIMARY,from_date_2,from_date
 key: from_date_2,from_date
      key_len: 3,3
          ref: NULL
 rows: 1
     filtered: 100.00
 Extra: Using intersect(from_date_2,from_date); Using where
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: employees.s.emp_no
         rows: 1
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)
```

现在删除多余的`from_date`索引并检查解释计划。您可以看到优化器在`salaries`表中扫描 90 行和`employees`表中的一行。但是看一下`ref`列；它显示常量与`key`列中命名的索引（`from_date_2`）进行比较，以从表中选择行。您可以通过传递优化器提示或索引提示来测试这种行为，这将在下一节中介绍：

```sql
mysql> EXPLAIN SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE from_date='2001-05-23'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: s
   partitions: NULL
         type: ref
possible_keys: PRIMARY,from_date_2
          key: from_date_2
      key_len: 3
 ref: const
         rows: 90
     filtered: 100.00
        Extra: NULL
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: employees.s.emp_no
         rows: 1
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)
```

现在您需要确定哪个查询更快：

+   **计划 1**：使用`intersect(from_date, from_date_2)`；扫描一行，ref 为空

+   **计划 2**：使用`from_date_2`；扫描 90 行，ref 为常量

您可以使用`mysqlslap`实用程序来查找（不要直接在生产主机上运行），并确保并发小于`max_connections`。

计划 1 的基准测试如下：

```sql
shell> mysqlslap -u <user> -p<pass> --create-schema='employees' -c 500 -i 100 --query="SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE from_date='2001-05-23'"
mysqlslap: [Warning] Using a password on the command line interface can be insecure.
Benchmark
    Average number of seconds to run all queries: 0.466 seconds
    Minimum number of seconds to run all queries: 0.424 seconds
    Maximum number of seconds to run all queries: 0.568 seconds
    Number of clients running queries: 500
    Average number of queries per client: 1
```

计划 2 的基准测试如下：

```sql
shell> mysqlslap -u <user> -p<pass> --create-schema='employees' -c 500 -i 100 --query="SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE from_date='2001-05-23'"
mysqlslap: [Warning] Using a password on the command line interface can be insecure.
Benchmark
    Average number of seconds to run all queries: 0.435 seconds
    Minimum number of seconds to run all queries: 0.376 seconds
    Maximum number of seconds to run all queries: 0.504 seconds
    Number of clients running queries: 500
    Average number of queries per client: 1
```

结果表明，计划 1 和计划 2 的平均查询时间分别为 0.466 秒和 0.435 秒。由于结果非常接近，您可以选择删除多余的索引。使用计划 2。

这只是一个示例，可以让您学习并应用在您的应用程序场景中。

# 检查索引使用情况

在前面的部分中，您了解了如何删除多余和重复的索引。在设计应用程序时，您可能已经考虑过根据列过滤查询并添加索引。但是随着应用程序的变化，您可能不再需要该索引。在本节中，您将了解如何识别这些未使用的索引。

您可以通过两种方式找到未使用的索引：

+   使用`pt-index-usage`（在本节中介绍）

+   使用`sys`模式（在下一节中介绍）

# 如何做到...

我们可以使用 Percona Toolkit 中的`pt-index-usage`工具进行索引分析。它从慢查询日志中获取查询，为每个查询运行解释计划，并识别未使用的索引。如果您有一系列查询，可以将它们保存为慢查询格式并将其传递给该工具。请注意，这只是一个近似值，因为慢查询日志不包括所有查询：

```sql
shell> sudo pt-index-usage slow -u <user> -p<password> /var/lib/mysql/db1-slow.log > unused_indexes
```

# 控制查询优化器

查询优化器的任务是为执行 SQL 查询找到最佳计划。在连接表时，可能有多个执行查询的计划，特别是要检查的计划数量呈指数增长。在本节中，您将了解如何调整优化器以满足您的需求。

以`employees`表为例，添加必要的索引；

```sql
mysql> CREATE TABLE `employees_index_example` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`),
  KEY `last_name` (`last_name`) /*!80000 INVISIBLE */,
  KEY `full_name` (`first_name`,`last_name`),
  KEY `full_name_desc` (`first_name` DESC,`last_name`),
  KEY `first_name` (`first_name`),
  KEY `full_name_1` (`first_name`,`last_name`),
  KEY `first_name_emp_no` (`first_name`,`emp_no`),
  KEY `last_name_2` (`last_name`(10))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
Query OK, 0 rows affected, 1 warning (0.08 sec)

mysql> SHOW WARNINGS;
+---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                                                |
+---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1831 | Duplicate index 'full_name_1' defined on the table 'employees.employees_index_example'. This is deprecated and will be disallowed in a future release. |
+---------+------+--------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> INSERT INTO employees_index_example SELECT emp_no,birth_date,first_name,last_name,gender,hire_date FROM employees;

mysql> RENAME TABLE employees TO employees_old;
mysql> RENAME TABLE employees_index_example TO employees;
```

假设您想检查`first_name`或`last_name`是否为`Adam`：

解释计划如下：

```sql
mysql> EXPLAIN SELECT emp_no FROM employees WHERE first_name='Adam' OR last_name='Adam'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index_merge
possible_keys: full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2
          key: first_name,last_name_2
      key_len: 58,42
          ref: NULL
         rows: 252
     filtered: 100.00
        Extra: Using sort_union(first_name,last_name_2); Using where
1 row in set, 1 warning (0.00 sec)
```

您会注意到优化器有许多选项可用于满足查询。它可以使用`possible_keys`中列出的任何索引：（full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2）。优化器验证所有计划，并确定哪个计划涉及的成本最低。

查询中涉及的一些成本示例包括从磁盘访问数据，从内存访问数据，创建临时表，在内存中对结果进行排序等。MySQL 为每个操作分配一个相对值，并对每个计划的总成本进行求和。它执行涉及最小成本的计划。

# 如何做...

您可以通过向查询传递提示或调整全局或会话级别的变量来控制优化器。甚至可以调整操作的成本。建议将这些值保留为默认值，除非您知道自己在做什么。

# optimizer_search_depth

来自[`jorgenloland.blogspot.in/2012/04/improvements-for-many-table-joins-in.html`](http://jorgenloland.blogspot.in/2012/04/improvements-for-many-table-joins-in.html)的 Jørgen 观点如下：

"MySQL 使用贪婪搜索算法来找到最佳的表连接顺序。当你只连接几个表时，计算所有连接顺序组合的成本然后选择最佳计划是没有问题的。然而，由于可能的组合数为(#tables)!，计算它们的成本很快变得太高：例如，对于五个表，有 120 种组合是可以计算的。对于 10 个表，有 360 万种组合，对于 15 个表，有 1307 亿种组合。因此，MySQL 做出了一个权衡：使用启发式方法只探索有前途的计划。这应该显著减少 MySQL 需要计算的计划数量，但同时也存在风险找不到最佳计划。"

MySQL 文档中说：

"optimizer_search_depth 变量告诉优化器应该查看每个不完整计划的“未来”多远，以评估是否应进一步扩展。较小的 optimizer_search_depth 值可能导致查询编译时间减少数个数量级。例如，如果使用 optimizer_search_depth 接近查询中的表数，查询可能需要几个小时甚至几天才能编译。同时，如果使用 optimizer_search_depth 等于 3 或 4 进行编译，对于相同的查询，优化器可能在不到一分钟内编译。如果您不确定 optimizer_search_depth 的合理值是多少，可以将该变量设置为 0，告诉优化器自动确定值。"

`optimizer_search_depth`的默认值为`62`，非常贪婪，但由于启发式方法，MySQL 很快选择了计划。从文档中不清楚为什么默认值设置为`62`而不是`0`。

如果您要连接超过七个表，可以将`optimizer_search_depth`设置为`0`或传递优化器提示（您将在下一节中了解到）。自动选择最小值（表的数量，七），将搜索深度限制为合理值：

```sql
mysql> SHOW VARIABLES LIKE 'optimizer_search_depth';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| optimizer_search_depth | 62    |
+------------------------+-------+
1 row in set (0.00 sec)

mysql> SET @@SESSION.optimizer_search_depth=0;
Query OK, 0 rows affected (0.00 sec)
```

# 如何知道查询在评估计划时花费了多少时间？

如果您要连接 10 个表（通常是 ORM 自动生成的），运行一个解释计划。如果花费更多时间，这意味着查询在评估计划时花费了太多时间。调整`optimizer_search_depth`的值（可能设置为`0`），并检查解释计划花费了多少时间。还要注意调整`optimizer_search_depth`的值时计划的变化。

# 优化器开关

`optimizer_switch`系统变量是一组标志。您可以将这些标志中的每一个设置为`ON`或`OFF`以启用或禁用相应的优化器行为。您可以在会话级别或全局级别动态设置它。如果您在会话级别调整了优化器开关，则该会话中的所有查询都会受到影响，如果在全局级别，则所有查询都会受到影响。

例如，您已经注意到前面的查询，`SELECT emp_no FROM employees WHERE first_name='Adam' OR last_name='Adam'`，正在使用`sort_union(first_name,last_name_2)`。如果您认为该优化对该查询不正确，您可以调整`optimizer_switch`以切换到另一种优化：

```sql
mysql> SHOW VARIABLES LIKE 'optimizer_switch'\G
*************************** 1\. row ***************************
Variable_name: optimizer_switch
        Value: index_merge=on,index_merge_union=on,index_merge_sort_union=on,index_merge_intersection=on,engine_condition_pushdown=on,index_condition_pushdown=on,mrr=on,mrr_cost_based=on,block_nested_loop=on,batched_key_access=off,materialization=on,semijoin=on,loosescan=on,firstmatch=on,duplicateweedout=on,subquery_materialization_cost_based=on,use_index_extensions=on,condition_fanout_filter=on,derived_merge=on
1 row in set (0.00 sec)
```

最初，`index_merge_union`是打开的：

```sql
mysql> EXPLAIN SELECT emp_no FROM employees WHERE first_name='Adam' OR last_name='Adam'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index_merge
possible_keys: full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2
          key: first_name,last_name_2
      key_len: 58,42
          ref: NULL
         rows: 252
     filtered: 100.00
 Extra: Using sort_union(first_name,last_name_2); Using where
1 row in set, 1 warning (0.00 sec)
```

优化器能够使用`sort_union`：

```sql
mysql> SET @@SESSION.optimizer_switch="index_merge_sort_union=off";
Query OK, 0 rows affected (0.00 sec)
```

您可以在会话级别关闭`index_merge_sort_union`优化，以便只有该会话中的查询受到影响：

```sql
mysql>  SHOW VARIABLES LIKE 'optimizer_switch'\G
*************************** 1\. row ***************************
Variable_name: optimizer_switch
        Value: index_merge=on,index_merge_union=on,index_merge_sort_union=off,index_merge_intersection=on,engine_condition_pushdown=on,index_condition_pushdown=on,mrr=on,mrr_cost_based=on,block_nested_loop=on,batched_key_access=off,materialization=on,semijoin=on,loosescan=on,firstmatch=on,duplicateweedout=on,subquery_materialization_cost_based=on,use_index_extensions=on,condition_fanout_filter=on,derived_merge=on
1 row in set (0.00 sec)
```

您会注意到在关闭`index_merge_sort_union`后计划发生变化；它不再使用`sort_union`优化：

```sql
mysql> EXPLAIN SELECT emp_no FROM employees WHERE first_name='Adam' OR last_name='Adam'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index
possible_keys: full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2
          key: full_name
      key_len: 124
          ref: NULL
         rows: 299379
     filtered: 19.00
 Extra: Using where; Using index
1 row in set, 1 warning (0.00 sec)
```

您还可以进一步发现，在这种情况下，使用`sort_union`是最佳选择。请参阅[`dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html`](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html)获取有关所有类型优化器开关的更多详细信息。

# 优化器提示

不要在会话级别调整优化器开关或`optimizer_search_depth`变量，您可以提示优化器使用或不使用某些优化。优化器提示的范围仅限于给您更好地控制查询的语句，而优化器开关可以在会话或全局级别设置。

再次以前面的查询为例；如果您觉得使用`sort_union`不是最佳选择，您可以通过在查询本身中传递提示来关闭它：

```sql
mysql> EXPLAIN SELECT /*+ NO_INDEX_MERGE(employees first_name,last_name_2) */ * FROM employees WHERE first_name='Adam' OR last_name='Adam'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: ALL
possible_keys: full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 299379
     filtered: 19.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

请记住，在冗余索引部分，我们删除了冗余索引以找出哪个计划更好。而不是这样做，您可以使用优化器提示来忽略`from_date`和`from_date_2`的交集：

```sql
mysql> EXPLAIN SELECT /*+ NO_INDEX_MERGE(s from_date,from_date_2) */ e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE from_date='2001-05-23'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: s
   partitions: NULL
         type: ref
possible_keys: PRIMARY,from_date,from_date_2
 key: from_date
      key_len: 3
 ref: const
         rows: 90
     filtered: 100.00
        Extra: NULL
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: employees.s.emp_no
         rows: 1
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)

```

另一个使用优化器提示的很好的例子是设置`JOIN`顺序：

```sql
mysql> EXPLAIN SELECT e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE (first_name='Adam' OR last_name='Adam') ORDER BY from_date DESC\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: index_merge
possible_keys: PRIMARY,full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2
          key: first_name,last_name_2
      key_len: 58,42
          ref: NULL
         rows: 252
     filtered: 100.00
        Extra: Using sort_union(first_name,last_name_2); Using where; Using temporary; Using filesort
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: s
   partitions: NULL
         type: ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: employees.e.emp_no
         rows: 9
     filtered: 100.00
        Extra: NULL
2 rows in set, 1 warning (0.00 sec)
```

在前面的查询中，优化器首先考虑`employees`表，并与`salaries`表进行连接。您可以通过传递提示`/*+ JOIN_ORDER(s,e ) */`来更改这一点：

```sql
mysql> EXPLAIN SELECT /*+ JOIN_ORDER(s, e) */ e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE (first_name='Adam' OR last_name='Adam') ORDER BY from_date DESC\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: s
   partitions: NULL
         type: ALL
possible_keys: PRIMARY
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 2838426
     filtered: 100.00
        Extra: Using filesort
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY,full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2
          key: PRIMARY
      key_len: 4
          ref: employees.s.emp_no
         rows: 1
     filtered: 19.00
        Extra: Using where
2 rows in set, 1 warning (0.00 sec)
```

您现在会注意到首先考虑`salaries`表，这样可以避免创建临时表，但它会对`salaries`表进行全表扫描。

优化器提示的另一个用例如下：而不是为每个语句或会话设置会话变量，您可以仅为该语句设置它们。假设您使用了一个对查询结果进行排序的`ORDER BY`子句，但在`ORDER BY`子句上没有索引。优化器使用`sort_buffer_size`来加速排序。默认情况下，`sort_buffer_size`的值为`256K`。如果`sort_buffer_size`不足，排序算法必须执行的合并次数会增加。您可以通过会话变量`sort_merge_passes`来衡量这一点：

```sql
mysql> SHOW SESSION status LIKE 'sort_merge_passes';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Sort_merge_passes | 0     |
+-------------------+-------+
1 row in set (0.00 sec)

mysql> pager grep "rows in set"; SELECT * FROM employees ORDER BY hire_date DESC;nopager;
PAGER set to 'grep "rows in set"'
300025 rows in set (0.45 sec)

PAGER set to stdout
mysql> SHOW SESSION status LIKE 'sort_merge_passes';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Sort_merge_passes | 8     |
+-------------------+-------+
1 row in set (0.00 sec)
```

您会注意到 MySQL 没有足够的`sort_buffer_size`，必须执行八次`sort_merge_passes`。您可以通过优化器提示将`sort_buffer_size`设置为诸如`16M`之类的大值，并检查`sort_merge_passes`：

```sql
mysql> SHOW SESSION status LIKE 'sort_merge_passes';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Sort_merge_passes | 0     |
+-------------------+-------+
1 row in set (0.00 sec)

mysql> pager grep "rows in set"; SELECT /*+ SET_VAR(sort_buffer_size = 16M) */ * FROM employees ORDER BY hire_date DESC;nopager;
PAGER set to 'grep "rows in set"'
300025 rows in set (0.45 sec)

PAGER set to stdout
mysql> SHOW SESSION status LIKE 'sort_merge_passes';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| Sort_merge_passes | 0     |
+-------------------+-------+
1 row in set (0.00 sec)
```

当`sort_buffer_size`设置为`16M`时，您会注意到`sort_merge_passes`为`0`。

强烈建议通过使用索引来优化您的查询，而不是依赖于`sort_buffer_size`。您可以考虑增加`sort_buffer_size`的值，以加快无法通过查询优化或改进索引的`ORDER BY`或`GROUP BY`操作的速度。

使用`SET_VAR`，您可以在语句级别设置`optimizer_switch`：

```sql
mysql> EXPLAIN SELECT /*+ SET_VAR(optimizer_switch = 'index_merge_sort_union=off') */ e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE from_date='2001-05-23'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: index
possible_keys: PRIMARY
          key: name
      key_len: 124
          ref: NULL
         rows: 299379
     filtered: 100.00
        Extra: Using index
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: s
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 7
          ref: employees.e.emp_no,const
         rows: 1
     filtered: 100.00
        Extra: NULL
2 rows in set, 1 warning (0.00 sec)
```

您还可以为查询设置最大执行时间，这意味着查询在指定时间后会自动终止使用`/*+ MAX_EXECUTION_TIME(毫秒) */`：

```sql
mysql> SELECT /*+ MAX_EXECUTION_TIME(100) */ * FROM employees ORDER BY hire_date DESC;
ERROR 1028 (HY000): Sort aborted: Query execution was interrupted, maximum statement execution time exceeded
```

您可以提示优化器做很多其他事情，请参阅[`dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html)获取完整列表和更多示例。

# 调整优化器成本模型

为了生成执行计划，优化器使用基于查询执行过程中各种操作成本的估算的成本模型。优化器具有一组编译的默认成本常量可供其使用，以便决定执行计划。您可以通过更新或插入`mysql.engine_cost`表并执行`FLUSH OPTIMIZER_COSTS`命令来调整它们：

```sql
mysql> SELECT * FROM mysql.engine_cost\G
*************************** 1\. row ***************************
  engine_name: InnoDB
  device_type: 0
    cost_name: io_block_read_cost
 cost_value: 1
  last_update: 2017-11-20 16:24:56
      comment: NULL
default_value: 1
*************************** 2\. row ***************************
  engine_name: InnoDB
  device_type: 0
    cost_name: memory_block_read_cost
   cost_value: 0.25
  last_update: 2017-11-19 13:58:32
      comment: NULL
default_value: 0.25
2 rows in set (0.00 sec)
```

假设您有一个超快的磁盘；您可以减少`io_block_read_cost`的`cost_value`：

```sql
mysql> UPDATE mysql.engine_cost SET cost_value=0.5 WHERE cost_name='io_block_read_cost';
Query OK, 1 row affected (0.08 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> FLUSH OPTIMIZER_COSTS;
Query OK, 0 rows affected (0.01 sec)

mysql> SELECT * FROM mysql.engine_cost\G
*************************** 1\. row ***************************
  engine_name: InnoDB
  device_type: 0
    cost_name: io_block_read_cost
 cost_value: 0.5
  last_update: 2017-11-20 17:02:43
      comment: NULL
default_value: 1
*************************** 2\. row ***************************
  engine_name: InnoDB
  device_type: 0
    cost_name: memory_block_read_cost
   cost_value: 0.25
  last_update: 2017-11-19 13:58:32
      comment: NULL
default_value: 0.25
2 rows in set (0.00 sec)
```

要了解更多有关优化器成本模型的信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/cost-model.html`](https://dev.mysql.com/doc/refman/8.0/en/cost-model.html)。

# 使用索引提示

使用索引提示，您可以提示优化器使用或忽略索引。这与优化器提示不同。在优化器提示中，您提示优化器使用或忽略某些优化方法。索引和优化器提示可以分别或一起使用以实现所需的计划。索引提示是在表名后指定的。

当您执行涉及多个表连接的复杂查询时，如果优化器在评估计划时花费太多时间，您可以确定最佳计划并给出查询的提示。但请确保您建议的计划是最佳的，并且在所有情况下都应该有效。

# 如何做...

以评估冗余索引的使用为例，使用相同的查询；它使用`intersect(from_date,from_date_2)`。通过传递优化器提示`(/*+ NO_INDEX_MERGE(s from_date,from_date_2) */)`，您避免了使用 intersect。您可以通过提示优化器忽略`from_date_2`索引来实现相同的行为：

```sql
mysql> EXPLAIN SELECT e.emp_no, salary FROM salaries s IGNORE INDEX(from_date_2) JOIN employees e ON s.emp_no=e.emp_no WHERE from_date='2001-05-23'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: s
   partitions: NULL
         type: ref
possible_keys: PRIMARY,from_date
          key: from_date
      key_len: 3
          ref: const
         rows: 90
     filtered: 100.00
        Extra: NULL
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: e
   partitions: NULL
         type: ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: employees.s.emp_no
         rows: 1
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)
```

另一个用例是提示优化器并节省评估多个计划的成本。考虑以下`employees`表和查询（与*控制查询优化器*部分开头讨论的相同）：

```sql
mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`),
  KEY `last_name` (`last_name`) /*!80000 INVISIBLE */,
  KEY `full_name` (`first_name`,`last_name`),
  KEY `full_name_desc` (`first_name` DESC,`last_name`),
  KEY `first_name` (`first_name`),
  KEY `full_name_1` (`first_name`,`last_name`),
  KEY `first_name_emp_no` (`first_name`,`emp_no`),
  KEY `last_name_2` (`last_name`(10))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)

mysql> EXPLAIN SELECT emp_no FROM employees WHERE first_name='Adam' OR last_name='Adam'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index_merge
possible_keys: full_name,full_name_desc,first_name,full_name_1,first_name_emp_no,last_name_2
          key: first_name,last_name_2
      key_len: 58,42
          ref: NULL
         rows: 252
     filtered: 100.00
        Extra: Using sort_union(first_name,last_name_2); Using where
1 row in set, 1 warning (0.00 sec)
```

您可以看到优化器必须评估`full_name`、`full_name_desc`、`first_name`、`full_name_1`、`first_name_emp_no`、`last_name_2`索引以得出最佳计划。您可以通过传递`USE INDEX(first_name,last_name_2)`来提示优化器，这将消除对其他索引的扫描：

```sql
mysql> EXPLAIN SELECT emp_no FROM employees USE INDEX(first_name,last_name_2) WHERE first_name='Adam' OR last_name='Adam'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: employees
   partitions: NULL
         type: index_merge
possible_keys: first_name,last_name_2
 key: first_name,last_name_2
      key_len: 58,42
          ref: NULL
         rows: 252
     filtered: 100.00
        Extra: Using sort_union(first_name,last_name_2); Using where
1 row in set, 1 warning (0.00 sec)
```

由于这是一个简单的查询，表非常小，性能提升可以忽略不计。当查询复杂且每小时执行数百万次时，性能提升可能会显著。

# 使用生成的列进行 JSON 索引

JSON 列不能直接建立索引。因此，如果您想在 JSON 列上使用索引，可以使用虚拟列和虚拟列上的创建索引来提取信息。

# 如何做...

1.  考虑您在第三章中创建的`emp_details`表，*使用 MySQL（高级）*，*使用 JSON*部分：

```sql
mysql> SHOW CREATE TABLE emp_details\G
*************************** 1\. row ***************************
       Table: emp_details
Create Table: CREATE TABLE `emp_details` (
  `emp_no` int(11) NOT NULL,
  `details` json DEFAULT NULL,
  PRIMARY KEY (`emp_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.00 sec)
```

1.  插入一些虚拟记录：

```sql
mysql> INSERT IGNORE INTO emp_details(emp_no, details) VALUES 
     ('1', '{ "location": "IN", "phone": "+11800000000", "email": "abc@example.com", "address": { "line1": "abc", "line2": "xyz street", "city": "Bangalore", "pin": "560103"}}'),
     ('2', '{ "location": "IN", "phone": "+11800000000", "email": "def@example.com", "address": { "line1": "abc", "line2": "xyz street", "city": "Delhi", "pin": "560103"}}'),
     ('3', '{ "location": "IN", "phone": "+11800000000", "email": "ghi@example.com", "address": { "line1": "abc", "line2": "xyz street", "city": "Mumbai", "pin": "560103"}}'),
     ('4', '{ "location": "IN", "phone": "+11800000000", "email": "jkl@example.com", "address": { "line1": "abc", "line2": "xyz street", "city": "Delhi", "pin": "560103"}}'),
     ('5', '{ "location": "US", "phone": "+11800000000", "email": "mno@example.com", "address": { "line1": "abc", "line2": "xyz street", "city": "Sunnyvale", "pin": "560103"}}');
Query OK, 5 rows affected (0.00 sec)
Records: 5  Duplicates: 0  Warnings: 0
```

1.  假设您想检索城市为`Bangalore`的`emp_no`：

```sql
mysql> EXPLAIN SELECT emp_no FROM emp_details WHERE details->>'$.address.city'="Bangalore"\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: emp_details
   partitions: NULL
         type: ALL
possible_keys: NULL
 key: NULL
      key_len: NULL
          ref: NULL
         rows: 5
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

您会注意到查询无法使用索引并扫描所有行。

1.  您可以将城市作为虚拟列检索并在其上添加索引：

```sql
mysql> ALTER TABLE emp_details ADD COLUMN city varchar(20) AS (details->>'$.address.city'), ADD INDEX (city);
Query OK, 0 rows affected (0.22 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE emp_details\G
*************************** 1\. row ***************************
       Table: emp_details
Create Table: CREATE TABLE `emp_details` (
  `emp_no` int(11) NOT NULL,
  `details` json DEFAULT NULL,
 `city` varchar(20) GENERATED ALWAYS AS (json_unquote(json_extract(`details`,_utf8'$.address.city'))) VIRTUAL,
  PRIMARY KEY (`emp_no`),
 KEY `city` (`city`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
1 row in set (0.01 sec)
```

1.  如果您现在检查解释计划，您会注意到查询能够使用`city`上的索引并且只扫描一行：

```sql
mysql> EXPLAIN SELECT emp_no FROM emp_details WHERE details->>'$.address.city'="Bangalore"\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: emp_details
   partitions: NULL
         type: ref
possible_keys: city
 key: city
      key_len: 83
 ref: const
 rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.00 sec)
```

要了解有关生成列上的辅助索引的更多信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/create-table-secondary-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-secondary-indexes.html)。

# 使用资源组

您可以限制查询仅使用一定数量的系统资源，使用资源组。目前，只有 CPU 时间是可管理的资源，由**虚拟 CPU**（**VCPU**）表示，其中包括 CPU 核心、超线程、硬件线程等。您可以创建一个资源组并将 VCPUs 分配给它。除了 CPU 外，资源组的属性是线程优先级。

您可以为线程分配资源组，在会话级别设置默认资源组，或将资源组作为优化器提示传递。例如，您想要以最低优先级运行一些查询（比如报告查询）；您可以将它们分配给具有最少资源的资源组。

# 如何做...

1.  将`CAP_SYS_NICE`功能设置为`mysqld`：

```sql
shell> ps aux | grep mysqld | grep -v grep
mysql     5238  0.0 28.1 1253368 488472 ?      Sl   Nov19   4:04 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

shell> sudo setcap cap_sys_nice+ep /usr/sbin/mysqld

shell> getcap /usr/sbin/mysqld
/usr/sbin/mysqld = cap_sys_nice+ep
```

1.  使用`CREATE RESOURCE GROUP`语句创建资源组。您必须提到资源组名称、VCPUS 数量、线程优先级和类型，可以是`USER`或`SYSTEM`。如果不指定 VCPUs，将使用所有 CPU：

```sql
mysql> CREATE RESOURCE GROUP report_group
TYPE = USER
VCPU = 2-3
THREAD_PRIORITY = 15
ENABLE;
# You should have at least 4 CPUs for the above resource group to create. If you have less CPUs, you can use VCPU = 0-1 for testing the example.
```

VCPU 表示 CPU 编号为 0-5，包括 CPU 0、1、2、3、4 和 5；0-3、8-9 和 11 包括 CPU 0、1、2、3、8、9 和 11。

`THREAD_PRIORITY`类似于 CPU 的优先级；系统资源组的范围为-20 到 0，用户组的范围为 0 到 19。-20 是最高优先级，19 是最低优先级。

您还可以启用或禁用资源组。默认情况下，在创建时启用资源组。禁用的组不能分配线程。

1.  创建后，您可以验证已创建的资源组：

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.RESOURCE_GROUPS\G
*************************** 1\. row ***************************
   RESOURCE_GROUP_NAME: USR_default
   RESOURCE_GROUP_TYPE: USER
RESOURCE_GROUP_ENABLED: 1
              VCPU_IDS: 0-0
       THREAD_PRIORITY: 0
*************************** 2\. row ***************************
   RESOURCE_GROUP_NAME: SYS_default
   RESOURCE_GROUP_TYPE: SYSTEM
RESOURCE_GROUP_ENABLED: 1
              VCPU_IDS: 0-0
       THREAD_PRIORITY: 0
*************************** 3\. row ***************************
 RESOURCE_GROUP_NAME: report_group
 RESOURCE_GROUP_TYPE: USER
RESOURCE_GROUP_ENABLED: 1
 VCPU_IDS: 2-3
 THREAD_PRIORITY: 15 
```

`USR_default`和`SYS_default`是默认资源组，不能被删除或修改。

1.  为线程分配一个组：

```sql
mysql> SET RESOURCE GROUP report_group FOR <thread_id>;
```

1.  设置会话资源组；该会话中的所有查询将在`report_group`下执行：

```sql
mysql> SET RESOURCE GROUP report_group;
```

1.  使用`RESOURCE_GROUP`优化器提示使用`report_group`执行单个语句：

```sql
mysql> SELECT /*+ RESOURCE_GROUP(report_group) */ * FROM employees;
```

# 更改和删除资源组

您可以动态调整资源组的 CPU 数量或`thread_priority`。如果系统负载过重，可以降低线程优先级：

```sql
mysql> ALTER RESOURCE GROUP report_group VCPU = 3 THREAD_PRIORITY = 19;
Query OK, 0 rows affected (0.12 sec)
```

类似地，当系统负载较轻时，您可以增加优先级：

```sql
mysql> ALTER RESOURCE GROUP report_group VCPU = 0-12 THREAD_PRIORITY = 0;
Query OK, 0 rows affected (0.12 sec)
```

您可以禁用资源组：

```sql
mysql> ALTER RESOURCE GROUP report_group DISABLE FORCE;
Query OK, 0 rows affected (0.00 sec)
```

您还可以使用`DROP RESOURCE GROUP`语句删除资源组：

```sql
mysql> DROP RESOURCE GROUP report_group FORCE;
```

如果给定`FORCE`，则正在运行的线程将移动到默认资源组（系统线程移动到`SYS_default`，用户线程移动到`USR_default`）。

如果未给出`FORCE`，则组中的现有线程将继续运行直到终止，但新线程不能分配给该组。

资源组仅限于本地服务器，并且与资源组相关的语句都不会被复制。要了解有关资源组的更多信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/resource-groups.html`](https://dev.mysql.com/doc/refman/8.0/en/resource-groups.html)。

# 使用 performance_schema

您可以使用`performance_schema`在运行时检查服务器的内部执行。这不应与信息模式混淆，信息模式用于检查元数据。

`performance_schema`中有许多事件消费者，会影响服务器的时间，例如函数调用、等待操作系统、SQL 语句执行阶段（如解析或排序）、单个语句或一组语句。所有收集的信息都存储在`performance_schema`中，不会被复制。

`performance_schema`默认启用；如果要禁用它，可以在`my.cnf`文件中设置`performance_schema=OFF`。默认情况下，并非所有消费者和仪器都启用；您可以通过更新`performance_schema.setup_instruments`和`performance_schema.setup_consumers`表来关闭/打开它们。

# 如何做...

我们将看看如何使用`performance_schema`。

# 启用/禁用 performance_schema

要禁用它，请将`performance_schema`设置为`0`：

```sql
shell> sudo vi /etc/my.cnf
[mysqld]
performance_schema = 0
```

# 启用/禁用消费者和仪器

您可以在`setup_consumers`表中看到可用的消费者列表，如下所示：

```sql
mysql> SELECT * FROM performance_schema.setup_consumers;
+----------------------------------+---------+
| NAME                             | ENABLED |
+----------------------------------+---------+
| events_stages_current            | NO      |
| events_stages_history            | NO      |
| events_stages_history_long       | NO      |
| events_statements_current        | YES     |
| events_statements_history        | YES     |
| events_statements_history_long   | NO      |
| events_transactions_current      | YES     |
| events_transactions_history      | YES     |
| events_transactions_history_long | NO      |
| events_waits_current             | NO      |
| events_waits_history             | NO      |
| events_waits_history_long        | NO      |
| global_instrumentation           | YES     |
| thread_instrumentation           | YES     |
| statements_digest                | YES     |
+----------------------------------+---------+
15 rows in set (0.00 sec)
```

假设您想要启用`events_waits_current`：

```sql
mysql> UPDATE performance_schema.setup_consumers SET ENABLED='YES' WHERE NAME='events_waits_current';
```

类似地，您可以从`setup_instruments`表中禁用或启用仪器。有大约 1182 个仪器（取决于版本）：

```sql
mysql> SELECT NAME, ENABLED, TIMED FROM setup_instruments LIMIT 10;
+---------------------------------------------------------+---------+-------+
| NAME                                                    | ENABLED | TIMED |
+---------------------------------------------------------+---------+-------+
| wait/synch/mutex/pfs/LOCK_pfs_share_list                | NO      | NO    |
| wait/synch/mutex/sql/TC_LOG_MMAP::LOCK_tc               | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit         | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_commit_queue   | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_done           | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_flush_queue    | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_index          | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_log            | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_binlog_end_pos | NO      | NO    |
| wait/synch/mutex/sql/MYSQL_BIN_LOG::LOCK_sync           | NO      | NO    |
+---------------------------------------------------------+---------+-------+
10 rows in set (0.00 sec)
```

# performance_schema 表

`performance_schema`中有五种主要类型的表。它们是当前事件表、事件历史表、事件摘要表、对象实例表和设置（配置）表：

```sql
mysql> SHOW TABLES LIKE '%current%';
+------------------------------------------+
| Tables_in_performance_schema (%current%) |
+------------------------------------------+
| events_stages_current                    |
| events_statements_current                |
| events_transactions_current              |
| events_waits_current                     |
+------------------------------------------+
4 rows in set (0.00 sec)

mysql> SHOW TABLES LIKE '%history%';
+------------------------------------------+
| Tables_in_performance_schema (%history%) |
+------------------------------------------+
| events_stages_history                    |
| events_stages_history_long               |
| events_statements_history                |
| events_statements_history_long           |
| events_transactions_history              |
| events_transactions_history_long         |
| events_waits_history                     |
| events_waits_history_long                |
+------------------------------------------+
8 rows in set (0.00 sec)

mysql> SHOW TABLES LIKE '%summary%';
+------------------------------------------------------+
| Tables_in_performance_schema (%summary%)             |
+------------------------------------------------------+
| events_errors_summary_by_account_by_error            |
| events_errors_summary_by_host_by_error               |
~
~
| table_io_waits_summary_by_table                      |
| table_lock_waits_summary_by_table                    |
+------------------------------------------------------+
41 rows in set (0.00 sec)

mysql> SHOW TABLES LIKE '%setup%';
+----------------------------------------+
| Tables_in_performance_schema (%setup%) |
+----------------------------------------+
| setup_actors                           |
| setup_consumers                        |
| setup_instruments                      |
| setup_objects                          |
| setup_threads                          |
| setup_timers                           |
+----------------------------------------+
6 rows in set (0.00 sec)
```

假设您想要找出哪个文件被访问最多：

```sql
mysql> SELECT EVENT_NAME, COUNT_STAR from file_summary_by_event_name ORDER BY count_star DESC LIMIT 10;
+-------------------------------------------------+------------+
| EVENT_NAME                                      | COUNT_STAR |
+-------------------------------------------------+------------+
| wait/io/file/innodb/innodb_data_file            |      35014 |
| wait/io/file/sql/io_cache                       |      13454 |
| wait/io/file/sql/binlog                         |       8785 |
| wait/io/file/innodb/innodb_log_file             |       2070 |
| wait/io/file/sql/query_log                      |       1257 |
| wait/io/file/innodb/innodb_temp_file            |         96 |
```

```sql
| wait/io/file/innodb/innodb_tablespace_open_file |         88 |
| wait/io/file/sql/casetest                       |         15 |
| wait/io/file/sql/binlog_index                   |         14 |
| wait/io/file/mysys/cnf                          |          5 |
+-------------------------------------------------+------------+
10 rows in set (0.00 sec)
```

或者您想要找出哪个文件在写入时花费了最多的时间：

```sql
mysql> SELECT EVENT_NAME, SUM_TIMER_WRITE FROM file_summary_by_event_name ORDER BY SUM_TIMER_WRITE DESC LIMIT 10;
+-------------------------------------------------+-----------------+
| EVENT_NAME                                      | SUM_TIMER_WRITE |
+-------------------------------------------------+-----------------+
| wait/io/file/innodb/innodb_data_file            |    410909759715 |
| wait/io/file/innodb/innodb_log_file             |    366157166830 |
| wait/io/file/sql/io_cache                       |    341899621700 |
| wait/io/file/sql/query_log                      |    203975010330 |
| wait/io/file/sql/binlog                         |     85261691515 |
| wait/io/file/innodb/innodb_temp_file            |     25291378385 |
| wait/io/file/innodb/innodb_tablespace_open_file |       674778195 |
| wait/io/file/sql/SDI                            |        18981690 |
| wait/io/file/sql/pid                            |        10233405 |
| wait/io/file/archive/FRM                        |               0 |
+-------------------------------------------------+-----------------+
```

您可以使用`events_statements_summary_by_digest`表来获取查询报告，就像您为`pt-query-digest`所做的那样。花费时间最多的顶级查询：

```sql
mysql> SELECT SCHEMA_NAME, digest, digest_text, round(sum_timer_wait/ 1000000000000, 6) as avg_time, count_star FROM performance_schema.events_statements_summary_by_digest ORDER BY sum_timer_wait DESC LIMIT 1\G
*************************** 1\. row ***************************
SCHEMA_NAME: NULL
     digest: 719f469393f90c27d84681a1d0ab3c19
digest_text: SELECT `sleep` (?) 
   avg_time: 60.000442
 count_star: 1
1 row in set (0.00 sec)
```

按执行次数最多的顶级查询：

```sql
mysql> SELECT SCHEMA_NAME, digest, digest_text, round(sum_timer_wait/ 1000000000000, 6) as avg_time, count_star FROM performance_schema.events_statements_summary_by_digest ORDER BY count_star DESC LIMIT 1\G
*************************** 1\. row ***************************
SCHEMA_NAME: employees
     digest: f5296ec6642c0fb977b448b350a2ba9b
digest_text: INSERT INTO `salaries` VALUES (...) /* , ... */ 
   avg_time: 32.736742
 count_star: 114
1 row in set (0.01 sec)
```

假设您想要查找特定查询的统计信息；而不是依赖于`mysqlslap`基准测试，您可以使用`performance_schema`来检查所有统计信息：

```sql
mysql> SELECT * FROM events_statements_summary_by_digest WHERE DIGEST_TEXT LIKE '%SELECT%employee%ORDER%' LIMIT 1\G
*************************** 1\. row ***************************
                SCHEMA_NAME: employees
                     DIGEST: d3b56f71f362f1bf6b067bfa358c04ab
                DIGEST_TEXT: EXPLAIN SELECT /*+ SET_VAR ( `sort_buffer_size` = ? ) */ `e` . `emp_no` , `salary` FROM `salaries` `s` JOIN `employees` `e` ON `s` . `emp_no` = `e` . `emp_no` WHERE ( `first_name` = ? OR `last_name` = ? ) ORDER BY `from_date` DESC 
                 COUNT_STAR: 1
             SUM_TIMER_WAIT: 643710000
             MIN_TIMER_WAIT: 643710000
             AVG_TIMER_WAIT: 643710000
             MAX_TIMER_WAIT: 643710000
              SUM_LOCK_TIME: 288000000
                 SUM_ERRORS: 0
               SUM_WARNINGS: 1
          SUM_ROWS_AFFECTED: 0
              SUM_ROWS_SENT: 2
          SUM_ROWS_EXAMINED: 0
SUM_CREATED_TMP_DISK_TABLES: 0
     SUM_CREATED_TMP_TABLES: 0
       SUM_SELECT_FULL_JOIN: 0
~
                 FIRST_SEEN: 2017-11-23 08:40:28.565406
                  LAST_SEEN: 2017-11-23 08:40:28.565406
                QUANTILE_95: 301995172
                QUANTILE_99: 301995172
               QUANTILE_999: 301995172
 QUERY_SAMPLE_TEXT: EXPLAIN SELECT /*+ SET_VAR(sort_buffer_size = 16M) */ e.emp_no, salary FROM salaries s JOIN employees e ON s.emp_no=e.emp_no WHERE (first_name='Adam' OR last_name='Adam') ORDER BY from_date DESC
          QUERY_SAMPLE_SEEN: 2017-11-23 08:40:28.565406
    QUERY_SAMPLE_TIMER_WAIT: 643710000
```

# 使用 sys 模式

`sys`模式帮助您以更简单和更易理解的形式解释从`performance_schema`收集的数据。`performance_schema`应该启用`sys`模式才能工作。要充分利用`sys`模式，您需要在`performance_schema`上启用所有的消费者和计时器，但这会影响服务器的性能。因此，只为您寻找的那些启用消费者。

带有`x$`前缀的视图以皮秒显示数据，其他表是人类可读的，这些数据被其他工具用于进一步处理。

# 如何做...

从`sys`模式启用一个工具：

```sql
mysql> CALL sys.ps_setup_enable_instrument('statement');
+------------------------+
| summary                |
+------------------------+
| Enabled 22 instruments |
+------------------------+
1 row in set (0.08 sec)

Query OK, 0 rows affected (0.08 sec)
```

如果要重置为默认值，请执行以下操作：

```sql
mysql> CALL sys.ps_setup_reset_to_default(TRUE)\G
*************************** 1\. row ***************************
status: Resetting: setup_actors
DELETE FROM performance_schema.setup_actors WHERE NOT (HOST = '%' AND USER = '%' AND `ROLE` = '%')
1 row in set (0.01 sec)
~
*************************** 1\. row ***************************
status: Resetting: threads
UPDATE performance_schema.threads SET INSTRUMENTED = 'YES'
1 row in set (0.03 sec)

Query OK, 0 rows affected (0.03 sec)
```

`sys`模式中有许多表；本节显示了一些最常用的表。

# 每个主机的类型声明（INSERT 和 SELECT）

```sql
mysql> SELECT statement, total, total_latency, rows_sent, rows_examined, rows_affected, full_scans FROM sys.host_summary_by_statement_type WHERE host='localhost' ORDER BY total DESC LIMIT 5;
+------------+--------+---------------+-----------+---------------+---------------+------------+
| statement  | total  | total_latency | rows_sent | rows_examined | rows_affected | full_scans |
+------------+--------+---------------+-----------+---------------+---------------+------------+
| select     | 208526 | 1.14 d        |  27484761 |     799220003 |             0 |       9265 |
| Quit       | 199551 | 4.76 s        |         0 |             0 |             0 |          0 |
| insert     |   9848 | 12.75 m       |         0 |             0 |       5075058 |          0 |
| Ping       |   4674 | 278.76 ms     |         0 |             0 |             0 |          0 |
| set_option |   2552 | 634.76 ms     |         0 |             0 |             0 |          0 |
+------------+--------+---------------+-----------+---------------+---------------+------------+
6 rows in set (0.00 sec)
```

# 每个用户的类型声明

```sql
mysql> SELECT statement, total, total_latency, rows_sent, rows_examined, rows_affected, full_scans FROM sys.user_summary_by_statement_type ORDER BY total DESC LIMIT 5;
+------------+--------+---------------+-----------+---------------+---------------+------------+
| statement  | total  | total_latency | rows_sent | rows_examined | rows_affected | full_scans |
+------------+--------+---------------+-----------+---------------+---------------+------------+
| select     | 208535 | 1.14 d        |  27485256 |     799246972 |             0 |       9273 |
| Quit       | 199551 | 4.76 s        |         0 |             0 |             0 |          0 |
| insert     |   9848 | 12.75 m       |         0 |             0 |       5075058 |          0 |
| Ping       |   4674 | 278.76 ms     |         0 |             0 |             0 |          0 |
| set_option |   2552 | 634.76 ms     |         0 |             0 |             0 |          0 |
+------------+--------+---------------+-----------+---------------+---------------+------------+
5 rows in set (0.01 sec)
```

# 冗余索引

```sql
mysql> SELECT * FROM sys.schema_redundant_indexes WHERE table_name='employees'\G
*************************** 1\. row ***************************
              table_schema: employees
                table_name: employees
      redundant_index_name: first_name
   redundant_index_columns: first_name
redundant_index_non_unique: 1
       dominant_index_name: first_name_emp_no
    dominant_index_columns: first_name,emp_no
 dominant_index_non_unique: 1
            subpart_exists: 0
            sql_drop_index: ALTER TABLE `employees`.`employees` DROP INDEX `first_name`
~
*************************** 8\. row ***************************
              table_schema: employees
                table_name: employees
      redundant_index_name: last_name_2
   redundant_index_columns: last_name
redundant_index_non_unique: 1
       dominant_index_name: last_name
    dominant_index_columns: last_name
 dominant_index_non_unique: 1
            subpart_exists: 1
            sql_drop_index: ALTER TABLE `employees`.`employees` DROP INDEX `last_name_2`
8 rows in set (0.00 sec)
```

# 未使用的索引

```sql
mysql> SELECT * FROM sys.schema_unused_indexes WHERE object_schema='employees';
+---------------+----------------+-------------------+
| object_schema | object_name    | index_name        |
+---------------+----------------+-------------------+
| employees     | departments    | dept_name         |
| employees     | dept_emp       | dept_no           |
| employees     | dept_manager   | dept_no           |
| employees     | employees      | name              |
| employees     | employees1     | last_name         |
| employees     | employees1     | full_name         |
| employees     | employees1     | full_name_desc    |
| employees     | employees1     | first_name        |
| employees     | employees1     | full_name_1       |
| employees     | employees1     | first_name_emp_no |
| employees     | employees1     | last_name_2       |
| employees     | employees_mgr  | manager_id        |
| employees     | employees_test | name              |
| employees     | emp_details    | city              |
+---------------+----------------+-------------------+
14 rows in set (0.00 sec)
```

# 每个主机执行的语句

```sql
mysql> SELECT * FROM sys.host_summary ORDER BY statements DESC LIMIT 1\G
*************************** 1\. row ***************************
                  host: localhost
            statements: 431214
     statement_latency: 1.15 d
 statement_avg_latency: 231.14 ms
           table_scans: 9424
              file_ios: 671972
       file_io_latency: 4.13 m
   current_connections: 3
     total_connections: 200193
          unique_users: 1
        current_memory: 0 bytes
total_memory_allocated: 0 bytes
1 row in set (0.02 sec)
```

# 表统计

```sql
mysql> SELECT * FROM sys.schema_table_statistics LIMIT 1\G
*************************** 1\. row ***************************
     table_schema: employees
       table_name: employees
    total_latency: 14.03 h
     rows_fetched: 731760045
    fetch_latency: 14.03 h
    rows_inserted: 300025
   insert_latency: 2.81 s
     rows_updated: 0
   update_latency: 0 ps
     rows_deleted: 0
   delete_latency: 0 ps
 io_read_requests: NULL
          io_read: NULL
  io_read_latency: NULL
io_write_requests: NULL
         io_write: NULL
 io_write_latency: NULL
 io_misc_requests: NULL
  io_misc_latency: NULL
1 row in set (0.01 sec)
```

# 带有缓冲区的表统计

```sql
mysql> SELECT * FROM sys.schema_table_statistics_with_buffer LIMIT 1\G
*************************** 1\. row ***************************
              table_schema: employees
                table_name: employees
              rows_fetched: 731760045
             fetch_latency: 14.03 h
             rows_inserted: 300025
            insert_latency: 2.81 s
              rows_updated: 0
            update_latency: 0 ps
              rows_deleted: 0
            delete_latency: 0 ps
~
   innodb_buffer_allocated: 6.80 MiB
        innodb_buffer_data: 6.23 MiB
        innodb_buffer_free: 582.77 KiB
       innodb_buffer_pages: 435
innodb_buffer_pages_hashed: 0
   innodb_buffer_pages_old: 435
 innodb_buffer_rows_cached: 147734
1 row in set (0.13 sec)
```

# 语句分析

此输出类似于`performance_schema.events_statements_summary_by_digest`和`pt-query-digest`的输出。

按执行次数最多的查询如下：

```sql
mysql> SELECT * FROM sys.statement_analysis ORDER BY exec_count DESC LIMIT 1\G
*************************** 1\. row ***************************
            query: SELECT `e` . `emp_no` , `salar ... emp_no` WHERE `from_date` = ? 
               db: employees
        full_scan: 
       exec_count: 159997
        err_count: 0
       warn_count: 0
    total_latency: 1.98 h
      max_latency: 661.58 ms
      avg_latency: 44.54 ms
     lock_latency: 1.28 m
        rows_sent: 14400270
    rows_sent_avg: 90
    rows_examined: 28800540
rows_examined_avg: 180
    rows_affected: 0
rows_affected_avg: 0
       tmp_tables: 0
  tmp_disk_tables: 0
      rows_sorted: 0
sort_merge_passes: 0
           digest: 94c925c0f00e06566d0447822066b1fe
       first_seen: 2017-11-23 05:39:09
        last_seen: 2017-11-23 05:45:45
1 row in set (0.01 sec)
```

消耗最大`tmp_disk_tables`的语句：

```sql
mysql> SELECT * FROM sys.statement_analysis ORDER BY tmp_disk_tables DESC LIMIT 1\G
*************************** 1\. row ***************************
            query: SELECT `cat` . `name` AS `TABL ... SE `col` . `type` WHEN ? THEN 
               db: employees
        full_scan: 
       exec_count: 195
        err_count: 0
       warn_count: 0
    total_latency: 249.55 ms
      max_latency: 2.84 ms
      avg_latency: 1.28 ms
     lock_latency: 97.95 ms
        rows_sent: 732
    rows_sent_avg: 4
    rows_examined: 4245
rows_examined_avg: 22
    rows_affected: 0
rows_affected_avg: 0
       tmp_tables: 195
  tmp_disk_tables: 195
      rows_sorted: 732
sort_merge_passes: 0
           digest: 8e8c46a210908a2efc2f1e96dd998130
       first_seen: 2017-11-19 05:27:24
        last_seen: 2017-11-20 17:24:34
1 row in set (0.01 sec)
```

要了解更多关于`sys`模式对象的信息，请参阅[`dev.mysql.com/doc/refman/8.0/en/sys-schema-object-index.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-schema-object-index.html)。
