# 第三章：使用 MySQL（高级）

在本章中，我们将涵盖以下配方：

+   使用 JSON

+   公共表达式（CTE）

+   生成的列

+   窗口函数

# 介绍

在本章中，您将了解 MySQL 的新引入功能。

# 使用 JSON

正如您在上一章中所看到的，要在 MySQL 中存储数据，您必须定义数据库和表结构（模式），这是一个重大的限制。为了应对这一点，从 MySQL 5.7 开始，MySQL 支持**JavaScript 对象表示**（**JSON**）数据类型。以前没有单独的数据类型，它被存储为字符串。新的 JSON 数据类型提供了 JSON 文档的自动验证和优化的存储格式。

JSON 文档以二进制格式存储，这使得以下操作成为可能：

+   快速读取文档元素

+   当服务器再次读取 JSON 时，不需要从文本表示中解析值

+   直接通过键或数组索引查找子对象或嵌套值，而无需读取文档中它们之前或之后的所有值

# 如何做...

假设您想要存储有关员工的更多详细信息；您可以使用 JSON 保存它们：

```go
CREATE TABLE emp_details( 
  emp_no int primary key, 
  details json 
);
```

# 插入 JSON

```go
INSERT INTO emp_details(emp_no, details)
VALUES ('1',
'{ "location": "IN", "phone": "+11800000000", "email": "abc@example.com", "address": { "line1": "abc", "line2": "xyz street", "city": "Bangalore", "pin": "560103"} }'
);
```

# 检索 JSON

您可以使用`->`和`->>`运算符检索 JSON 列的字段：

```go
mysql> SELECT emp_no, details->'$.address.pin' pin FROM emp_details;
+--------+----------+
| emp_no | pin      |
+--------+----------+
| 1      | "560103" |
+--------+----------+
1 row in set (0.00 sec)
```

要检索没有引号的数据，请使用`->>`运算符：

```go
mysql> SELECT emp_no, details->>'$.address.pin' pin FROM emp_details;
+--------+--------+
| emp_no | pin    |
+--------+--------+
| 1      | 560103 |
+--------+--------+
1 row in set (0.00 sec)
```

# JSON 函数

MySQL 提供了许多处理 JSON 数据的函数。让我们看看最常用的函数。

# 漂亮的视图

要以漂亮的格式显示 JSON 值，请使用`JSON_PRETTY()`函数：

```go
mysql> SELECT emp_no, JSON_PRETTY(details) FROM emp_details \G
*************************** 1\. row ***************************
 emp_no: 1
JSON_PRETTY(details): {
 "email": "abc@example.com",
 "phone": "+11800000000",
 "address": {
 "pin": "560103",
 "city": "Bangalore",
 "line1": "abc",
 "line2": "xyz street"
 },
 "location": "IN"
}
1 row in set (0.00 sec)
```

不漂亮的：

```go
mysql> SELECT emp_no, details FROM emp_details \G
*************************** 1\. row ***************************
 emp_no: 1
details: {"email": "abc@example.com", "phone": "+11800000000", "address": {"pin": "560100", "city": "Bangalore", "line1": "abc", "line2": "xyz street"}, "location": "IN"}
1 row in set (0.00 sec)
```

# 搜索

您可以在`WHERE`子句中使用`col->>path`运算符引用 JSON 列：

```go
mysql> SELECT emp_no FROM emp_details WHERE details->>'$.address.pin'="560103";
+--------+
| emp_no |
+--------+
| 1      |
+--------+
1 row in set (0.00 sec)
```

您还可以使用`JSON_CONTAINS`函数搜索数据。如果找到数据，则返回`1`，否则返回`0`：

```go
mysql> SELECT JSON_CONTAINS(details->>'$.address.pin', "560103") FROM emp_details;
+----------------------------------------------------+
| JSON_CONTAINS(details->>'$.address.pin', "560103") |
+----------------------------------------------------+
| 1                                                  |
+----------------------------------------------------+
1 row in set (0.00 sec)
```

如何搜索键？假设您想要检查`address.line1`是否存在：

```go
mysql> SELECT JSON_CONTAINS_PATH(details, 'one', "$.address.line1") FROM emp_details;
+--------------------------------------------------------------------------+
| JSON_CONTAINS_PATH(details, 'one', "$.address.line1")                    |
+--------------------------------------------------------------------------+
| 1                                                                        |
+--------------------------------------------------------------------------+
1 row in set (0.01 sec)
```

在这里，`one`表示至少应该存在一个键。假设您想要检查`address.line1`或`address.line2`是否存在：

```go
mysql> SELECT JSON_CONTAINS_PATH(details, 'one', "$.address.line1", "$.address.line5") FROM emp_details;
+--------------------------------------------------------------------------+
| JSON_CONTAINS_PATH(details, 'one', "$.address.line1", "$.address.line2") |
+--------------------------------------------------------------------------+
| 1                                                                        |
+--------------------------------------------------------------------------+
```

如果要检查`address.line1`和`address.line5`是否都存在，可以使用`and`而不是`one`：

```go
mysql> SELECT JSON_CONTAINS_PATH(details, 'all', "$.address.line1", "$.address.line5") FROM emp_details;
+--------------------------------------------------------------------------+
| JSON_CONTAINS_PATH(details, 'all', "$.address.line1", "$.address.line5") |
+--------------------------------------------------------------------------+
| 0                                                                        |
+--------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

# 修改

您可以使用三种不同的函数修改数据：`JSON_SET()`，`JSON_INSERT()`，`JSON_REPLACE()`。在 MySQL 8 之前，我们需要对整个列进行完全更新，这不是最佳方式：

+   `JSON_SET`: 替换现有值并添加不存在的值。

假设您想要替换员工的邮政编码并添加昵称的详细信息：

```go
mysql> UPDATE 
    emp_details 
SET 
    details = JSON_SET(details, "$.address.pin", "560100", "$.nickname", "kai")
WHERE 
    emp_no = 1;
Query OK, 1 row affected (0.03 sec)
Rows matched: 1 Changed: 1 Warnings: 0
```

+   `JSON_INSERT()`: 插入值而不替换现有值

假设您想要添加一个新列而不更新现有值；您可以使用`JSON_INSERT()`：

```go
mysql> UPDATE emp_details SET details=JSON_INSERT(details, "$.address.pin", "560132", "$.address.line4", "A Wing") WHERE emp_no = 1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1 Changed: 1 Warnings: 0
```

在这种情况下，`pin`不会被更新；只会添加一个新的`address.line4`字段。

+   `JSON_REPLACE()`: 仅替换现有值

假设您只想替换字段而不添加新字段：

```go
mysql> UPDATE emp_details SET details=JSON_REPLACE(details, "$.address.pin", "560132", "$.address.line5", "Landmark") WHERE 
emp_no = 1;
Query OK, 1 row affected (0.04 sec)
Rows matched: 1 Changed: 1 Warnings: 0
```

在这种情况下，`line5`不会被添加。只有`pin`会被更新。

# 删除

`JSON_REMOVE`从 JSON 文档中删除数据。

假设您不再需要地址中的`line5`：

```go
mysql> UPDATE emp_details SET details=JSON_REMOVE(details, "$.address.line5") WHERE emp_no = 1;
Query OK, 1 row affected (0.04 sec)
Rows matched: 1 Changed: 1 Warnings: 0
```

# 其他函数

其他一些函数如下：

+   `JSON_KEYS()`: 获取 JSON 文档中的所有键：

```go
mysql> SELECT JSON_KEYS(details) FROM emp_details WHERE emp_no = 1;
*************************** 1\. row ***************************
JSON_KEYS(details): ["email", "phone", "address", "nickname", "locatation"]
```

+   `JSON_LENGTH()`: 给出 JSON 文档中元素的数量：

```go
mysql> SELECT JSON_LENGTH(details) FROM emp_details WHERE emp_no = 1;
*************************** 1\. row ***************************
JSON_LENGTH(details): 5
```

# 参见

您可以在[`dev.mysql.com/doc/refman/8.0/en/json-function-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html)查看完整的函数列表。

# 公共表达式（CTE）

MySQL 8 支持公共表达式，包括非递归和递归。

公共表达式使得可以使用命名的临时结果集，通过允许在`SELECT`语句和某些其他语句之前使用`WITH`子句。

**为什么需要 CTE？**在同一查询中不可能引用派生表两次。因此，派生表会被评估两次或多次，这表明存在严重的性能问题。使用 CTE，子查询只评估一次。

# 如何做...

递归和非递归 CTE 将在以下部分中进行讨论。

# 非递归 CTE

**公共表达式**（**CTE**）就像派生表一样，但其声明放在查询块之前，而不是在`FROM`子句中。

**派生表**

```go
SELECT... FROM (subquery) AS derived, t1 ...
```

**CTE**

```go
SELECT... WITH derived AS (subquery) SELECT ... FROM derived, t1 ...
```

CTE 可以在`SELECT`/`UPDATE`/`DELETE`之前，包括子查询`WITH`派生`AS`（子查询），例如：

```go
DELETE FROM t1 WHERE t1.a IN (SELECT b FROM derived);
```

假设您想要找出每年薪水相对于上一年的百分比增长。没有 CTE，您需要编写两个子查询，它们本质上是相同的。MySQL 不足够聪明，无法检测到这一点，并且子查询会执行两次：

```go
mysql> SELECT 
    q1.year, 
    q2.year AS next_year, 
    q1.sum, 
    q2.sum AS next_sum, 
    100*(q2.sum-q1.sum)/q1.sum AS pct 
FROM 
    (SELECT year(from_date) as year, sum(salary) as sum FROM salaries GROUP BY year) AS q1,             (SELECT year(from_date) as year, sum(salary) as sum FROM salaries GROUP BY year) AS q2 
WHERE q1.year = q2.year-1;
+------+-----------+-------------+-------------+----------+
| year | next_year | sum         | next_sum    | pct      |
+------+-----------+-------------+-------------+----------+
| 1985 |      1986 | 972864875   | 2052895941  | 111.0155 |
| 1986 |      1987 | 2052895941  | 3156881054  | 53.7770  |
| 1987 |      1988 | 3156881054  | 4295598688  | 36.0710  |
| 1988 |      1989 | 4295598688  | 5454260439  | 26.9732  |
| 1989 |      1990 | 5454260439  | 6626146391  | 21.4857  |
| 1990 |      1991 | 6626146391  | 7798804412  | 17.6974  |
| 1991 |      1992 | 7798804412  | 9027872610  | 15.7597  |
| 1992 |      1993 | 9027872610  | 10215059054 | 13.1502  |
| 1993 |      1994 | 10215059054 | 11429450113 | 11.8882  |
| 1994 |      1995 | 11429450113 | 12638817464 | 10.5812  |
| 1995 |      1996 | 12638817464 | 13888587737 | 9.8883   |
| 1996 |      1997 | 13888587737 | 15056011781 | 8.4056   |
| 1997 |      1998 | 15056011781 | 16220495471 | 7.7343   |
| 1998 |      1999 | 16220495471 | 17360258862 | 7.0267   |
| 1999 |      2000 | 17360258862 | 17535667603 | 1.0104   |
| 2000 |      2001 | 17535667603 | 17507737308 | -0.1593  |
| 2001 |      2002 | 17507737308 | 10243358658 | -41.4924 |
+------+-----------+-------------+-------------+----------+
17 rows in set (3.22 sec)
```

使用非递归 CTE，派生查询仅执行一次并被重用：

```go
mysql> 
WITH CTE AS 
    (SELECT year(from_date) AS year, SUM(salary) AS sum FROM salaries GROUP BY year) 
SELECT 
    q1.year, q2.year as next_year, q1.sum, q2.sum as next_sum, 100*(q2.sum-q1.sum)/q1.sum as pct FROM 
    CTE AS q1, 
    CTE AS q2 
WHERE 
    q1.year = q2.year-1;
+------+-----------+-------------+-------------+----------+
| year | next_year | sum         | next_sum    | pct      |
+------+-----------+-------------+-------------+----------+
| 1985 |      1986 | 972864875   | 2052895941  | 111.0155 |
| 1986 |      1987 | 2052895941  | 3156881054  | 53.7770  |
| 1987 |      1988 | 3156881054  | 4295598688  | 36.0710  |
| 1988 |      1989 | 4295598688  | 5454260439  | 26.9732  |
| 1989 |      1990 | 5454260439  | 6626146391  | 21.4857  |
| 1990 |      1991 | 6626146391  | 7798804412  | 17.6974  |
| 1991 |      1992 | 7798804412  | 9027872610  | 15.7597  |
| 1992 |      1993 | 9027872610  | 10215059054 | 13.1502  |
| 1993 |      1994 | 10215059054 | 11429450113 | 11.8882  |
| 1994 |      1995 | 11429450113 | 12638817464 | 10.5812  |
| 1995 |      1996 | 12638817464 | 13888587737 | 9.8883   |
| 1996 |      1997 | 13888587737 | 15056011781 | 8.4056   |
```

```go
| 1997 |      1998 | 15056011781 | 16220495471 | 7.7343   |
| 1998 |      1999 | 16220495471 | 17360258862 | 7.0267   |
| 1999 |      2000 | 17360258862 | 17535667603 | 1.0104   |
| 2000 |      2001 | 17535667603 | 17507737308 | -0.1593  |
| 2001 |      2002 | 17507737308 | 10243358658 | -41.4924 |
+------+-----------+-------------+-------------+----------+
17 rows in set (1.63 sec)
```

您可能会注意到，使用 CTE，结果相同，查询时间提高了 50％；可读性很好，并且可以多次引用。

派生查询不能引用其他派生查询：

```go
SELECT ...
 FROM (SELECT ... FROM ...) AS d1, (SELECT ... FROM d1 ...) AS d2 ...
ERROR: 1146 (42S02): Table ‘db.d1’ doesn’t exist 
```

CTEs 可以引用其他 CTEs：

```go
WITH d1 AS (SELECT ... FROM ...), d2 AS (SELECT ... FROM d1 ...) 
SELECT
 FROM d1, d2 ... 
```

# 递归 CTE

递归 CTE 是一个带有引用自身名称的子查询的 CTE。`WITH`子句必须以`WITH RECURSIVE`开头。递归 CTE 子查询有两部分，种子查询和递归查询，由`UNION [ALL]`或`UNION DISTINCT`分隔。

种子`SELECT`执行一次以创建初始数据子集；递归`SELECT`被重复执行以返回数据子集，直到获得完整的结果集。当迭代不再生成任何新行时，递归停止。这对于深入研究层次结构（父/子或部分/子部分）非常有用：

```go
WITH RECURSIVE cte AS
(SELECT ... FROM table_name /* seed SELECT */ 
UNION ALL 
SELECT ... FROM cte, table_name) /* "recursive" SELECT */ 
SELECT ... FROM cte;
```

假设您想打印从`1`到`5`的所有数字：

```go
mysql> WITH RECURSIVE cte (n) AS 
( SELECT 1 /* seed query */
  UNION ALL 
  SELECT n + 1 FROM cte WHERE n < 5 /* recursive query */
) 
SELECT * FROM cte;
+---+
| n |
+---+
| 1 |
| 2 |
| 3 |
| 4 |
| 5 |
+---+
5 rows in set (0.00 sec)
```

在每次迭代中，`SELECT`生成一个新值的行，该值比上一行集合中的`n`值多 1。第一次迭代在初始行集（1）上操作，并产生*1+1=2*；第二次迭代在第一次迭代的行集（2）上操作，并产生*2+1=3*；依此类推。这将持续到递归结束，当`n`不再小于`5`时。

假设您想要对分层数据进行遍历，以生成每个员工的管理链的组织结构图（即从 CEO 到员工的路径）。使用递归 CTE！

创建一个带有`manager_id`的测试表：

```go
mysql> CREATE TABLE employees_mgr (
 id INT PRIMARY KEY NOT NULL,
 name VARCHAR(100) NOT NULL,
 manager_id INT NULL,
 INDEX (manager_id),
FOREIGN KEY (manager_id) REFERENCES employees_mgr (id)
);
```

插入示例数据：

```go
mysql> INSERT INTO employees_mgr VALUES
(333, "Yasmina", NULL), # Yasmina is the CEO (manager_id is NULL)
(198, "John", 333), # John has ID 198 and reports to 333 (Yasmina)
(692, "Tarek", 333),
(29, "Pedro", 198),
(4610, "Sarah", 29),
(72, "Pierre", 29),
(123, "Adil", 692);
```

执行递归 CTE：

```go
mysql> WITH RECURSIVE employee_paths (id, name, path) AS
(
 SELECT id, name, CAST(id AS CHAR(200))
 FROM employees_mgr
 WHERE manager_id IS NULL
 UNION ALL
 SELECT e.id, e.name, CONCAT(ep.path, ',', e.id)
 FROM employee_paths AS ep JOIN employees_mgr AS e
 ON ep.id = e.manager_id
)
SELECT * FROM employee_paths ORDER BY path;
```

它产生以下结果：

```go
+------+---------+-----------------+
| id   | name    | path            |
+------+---------+-----------------+
| 333  | Yasmina | 333             |
| 198  | John    | 333,198         |
| 29   | Pedro   | 333,198,29      |
| 4610 | Sarah   | 333,198,29,4610 |
| 72   | Pierre  | 333,198,29,72   |
| 692  | Tarek   | 333,692         |
| 123  | Adil    | 333,692,123     |
+------+---------+-----------------+
7 rows in set (0.00 sec)
```

`WITH RECURSIVE employee_paths（id，name，path）AS`是 CTE 的名称，列为（`id`，`name`，`path`）。

`SELECT id，name，CAST(id AS CHAR（200））FROM employees_mgr WHERE manager_id IS NULL`是选择 CEO 的种子查询（CEO 没有经理）。

`SELECT e.id，e.name，CONCAT(ep.path，'，'，e.id) FROM employee_paths AS ep JOIN employees_mgr AS e ON ep.id = e.manager_id`是递归查询。

递归查询生成的每一行都会找到直接向以前行生成的员工汇报的所有员工。对于这样的员工，该行包括员工 ID，名称和员工管理链。该链是经理的链，员工 ID 添加到末尾。

# 生成列

生成列也称为虚拟列或计算列。生成列的值是从列定义中包含的表达式计算出来的。有两种类型的生成列：

+   **虚拟**：当从表中读取记录时，该列将在读取时动态计算

+   **存储**：当在表中写入新记录时，将计算该列并将其存储在表中作为常规列

虚拟生成列比存储生成列更有用，因为虚拟列不占用任何存储空间。您可以使用触发器模拟存储生成列的行为。

# 如何做...

假设您的应用程序在从`employees`表中检索数据时使用`full_name`作为`concat('first_name'，' '，'last_name')`，而不是使用表达式，您可以使用虚拟列，该列在读取时计算`full_name`。您可以在表达式后面添加另一列：

```go
mysql> CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
 `full_name` VARCHAR(30) AS (CONCAT(first_name,' ',last_name)),
  PRIMARY KEY (`emp_no`),
  KEY `name` (`first_name`,`last_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

请注意，您应该根据虚拟列修改插入语句。您可以选择使用完整的插入，如下所示：

```go
mysql> INSERT INTO employees (emp_no, birth_date, first_name, last_name, gender, hire_date) VALUES (123456, '1987-10-02', 'ABC' , 'XYZ', 'F', '2008-07-28');
```

如果您想在`INSERT`语句中包含`full_name`，可以将其指定为`DEFAULT`。所有其他值都会抛出`ERROR 3105 (HY000):`错误。在`employees`表中指定生成列`full_name`的值是不允许的：

```go
mysql> INSERT INTO employees (emp_no, birth_date, first_name, last_name, gender, hire_date, full_name) VALUES (123456, '1987-10-02', 'ABC' , 'XYZ', 'F', '2008-07-28', DEFAULT);
```

您可以直接从`employees`表中选择`full_name`：

```go
mysql> SELECT * FROM employees WHERE emp_no=123456;
+--------+------------+------------+-----------+--------+------------+-----------+
| emp_no | birth_date | first_name | last_name | gender | hire_date  | full_name |
+--------+------------+------------+-----------+--------+------------+-----------+
| 123456 | 1987-10-02 | ABC        | XYZ       | F      | 2017-11-23 | ABC XYZ   |
+--------+------------+------------+-----------+--------+------------+-----------+
1 row in set (0.00 sec)
```

如果您已经创建了表并希望添加新的生成列，请执行 ALTER TABLE 语句，这将在*第十章* *表维护*中详细介绍

例子：

```go
mysql> ALTER TABLE employees ADD hire_date_year YEAR AS (YEAR(hire_date)) VIRTUAL;
```

请参考[`dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html)了解更多关于生成列的信息。您将在第十三章 *性能调优*中了解虚拟列的其他用途，在*添加索引*和*使用生成列为 JSON 添加索引*部分。

# 窗口函数

通过使用窗口函数，您可以对与该行相关的行执行计算。这是通过使用`OVER`和`WINDOW`子句来实现的。

以下是您可以进行的计算列表：

+   `CUME_DIST()`: 累积分布值

+   `DENSE_RANK()`: 在其分区内当前行的排名，没有间隙

+   `FIRST_VALUE()`: 窗口帧的第一行的参数值

+   `LAG()`: 分区内当前行的前一行的参数值

+   `LAST_VALUE()`: 窗口帧的第一行的参数值

+   `LEAD()`: 分区内当前行的后一行的参数值

+   `NTH_VALUE()`: 窗口帧的第*n*行的参数值

+   `NTILE()`: 在其分区内当前行的桶编号

+   `PERCENT_RANK()`: 百分比排名值

+   `RANK()`: 在其分区内当前行的排名，有间隙

+   `ROW_NUMBER()`: 在其分区内当前行的编号

# 如何做...

窗口函数可以以各种方式使用。让我们在以下部分了解每一个。为了使这些示例起作用，您需要添加 hire_date_year 虚拟列

```go
mysql> ALTER TABLE employees ADD hire_date_year YEAR AS (YEAR(hire_date)) VIRTUAL;
```

# 行号

您可以为每一行获取行号以对结果进行排名：

```go
mysql> SELECT CONCAT(first_name, " ", last_name) AS full_name, salary, ROW_NUMBER() OVER(ORDER BY salary DESC) AS 'Rank'  FROM employees JOIN salaries ON salaries.emp_no=employees.emp_no LIMIT 10;
+-------------------+--------+------+
| full_name         | salary | Rank |
+-------------------+--------+------+
| Tokuyasu Pesch    | 158220 |    1 |
| Tokuyasu Pesch    | 157821 |    2 |
| Honesty Mukaidono | 156286 |    3 |
| Xiahua Whitcomb   | 155709 |    4 |
| Sanjai Luders     | 155513 |    5 |
| Tsutomu Alameldin | 155377 |    6 |
| Tsutomu Alameldin | 155190 |    7 |
| Tsutomu Alameldin | 154888 |    8 |
| Tsutomu Alameldin | 154885 |    9 |
| Willard Baca      | 154459 |   10 |
+-------------------+--------+------+
10 rows in set (6.24 sec)
```

# 分区结果

您可以在`OVER`子句中对结果进行分区。假设您想要找出每年的薪水排名；可以按如下方式完成：

```go
mysql> SELECT hire_date_year, salary, ROW_NUMBER() OVER(PARTITION BY hire_date_year ORDER BY salary DESC) AS 'Rank' FROM employees JOIN salaries ON salaries.emp_no=employees.emp_no ORDER BY salary DESC LIMIT 10;
+----------------+--------+------+
| hire_date_year | salary | Rank |
+----------------+--------+------+
|           1985 | 158220 |    1 |
|           1985 | 157821 |    2 |
|           1986 | 156286 |    1 |
|           1985 | 155709 |    3 |
|           1987 | 155513 |    1 |
|           1985 | 155377 |    4 |
|           1985 | 155190 |    5 |
|           1985 | 154888 |    6 |
|           1985 | 154885 |    7 |
|           1985 | 154459 |    8 |
+----------------+--------+------+
10 rows in set (8.04 sec)
```

您可以注意到，`1986`年和`1987`年的排名发生了变化，但`1985`年的排名保持不变。

# 命名窗口

您可以命名一个窗口，并且可以根据需要多次使用它，而不是每次重新定义它：

```go
mysql> SELECT hire_date_year, salary, RANK() OVER w AS 'Rank' FROM employees join salaries ON salaries.emp_no=employees.emp_no WINDOW w AS (PARTITION BY hire_date_year ORDER BY salary DESC) ORDER BY salary DESC LIMIT 10;
+----------------+--------+------+
| hire_date_year | salary | Rank |
+----------------+--------+------+
|           1985 | 158220 |    1 |
|           1985 | 157821 |    2 |
|           1986 | 156286 |    1 |
|           1985 | 155709 |    3 |
|           1987 | 155513 |    1 |
|           1985 | 155377 |    4 |
|           1985 | 155190 |    5 |
|           1985 | 154888 |    6 |
|           1985 | 154885 |    7 |
|           1985 | 154459 |    8 |
+----------------+--------+------+
10 rows in set (8.52 sec)
```

# 第一个、最后一个和第 n 个值

您可以在窗口结果中选择第一个、最后一个和第 n 个值。如果行不存在，则返回`NULL`值。

假设您想要从窗口中找到第一个、最后一个和第三个值：

```go
mysql> SELECT hire_date_year, salary, RANK() OVER w AS 'Rank', 
FIRST_VALUE(salary) OVER w AS 'first', 
NTH_VALUE(salary, 3) OVER w AS 'third', 
LAST_VALUE(salary) OVER w AS 'last' 
FROM employees join salaries ON salaries.emp_no=employees.emp_no 
WINDOW w AS (PARTITION BY hire_date_year ORDER BY salary DESC) 
ORDER BY salary DESC LIMIT 10;
+----------------+--------+------+--------+--------+--------+
| hire_date_year | salary | Rank | first  | third  | last   |
+----------------+--------+------+--------+--------+--------+
|           1985 | 158220 |    1 | 158220 |   NULL | 158220 |
|           1985 | 157821 |    2 | 158220 |   NULL | 157821 |
|           1986 | 156286 |    1 | 156286 |   NULL | 156286 |
|           1985 | 155709 |    3 | 158220 | 155709 | 155709 |
|           1987 | 155513 |    1 | 155513 |   NULL | 155513 |
|           1985 | 155377 |    4 | 158220 | 155709 | 155377 |
|           1985 | 155190 |    5 | 158220 | 155709 | 155190 |
|           1985 | 154888 |    6 | 158220 | 155709 | 154888 |
|           1985 | 154885 |    7 | 158220 | 155709 | 154885 |
|           1985 | 154459 |    8 | 158220 | 155709 | 154459 |
+----------------+--------+------+--------+--------+--------+
10 rows in set (12.88 sec)
```

要了解窗口函数的其他用例，请参考[`mysqlserverteam.com/mysql-8-0-2-introducing-window-functions`](https://mysqlserverteam.com/mysql-8-0-2-introducing-window-functions)和[`dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_row-number`](https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html#function_row-number)。
