# 第九章：MySQL 8 中的分区

在前一章中，解释了 MySQL 8 中的复制。这包括复制、配置和实现的详细解释。该章还解释了组复制与集群，并涵盖了复制方法作为解决方案。

在本章中，我们将在 MySQL 8 中进行分区。分区是管理和维护具有特定操作的数据的概念，具有多个运算符，并定义了控制分区的规则。基本上，它提供了一个配置钩子，以指定的方式管理底层数据文件。

我们将在分区上涵盖以下主题：

+   分区概述

+   分区类型

+   分区管理

+   分区选择

+   分区修剪

+   分区中的限制和限制

# 分区概述

分区的概念与数据库中数据存储的物理方面相关。如果查看`SQL`标准，它们并没有提供关于这个概念的太多信息，`SQL`语言本身意图独立于用于存储信息或特定于不同模式、表、行或列的数据的媒体或数据结构。先进的数据库管理系统已经添加了指定用于数据存储的物理位置的方法，如硬件、文件系统或两者兼而有之。在 MySQL 中，`InnoDB`存储引擎通过`表空间`的概念支持这些目的。

分区使我们能够将个别表的部分分布为在文件系统中的不同位置存储为单独的表。此外，分布是通过用户指定的规则提供的，例如模数、哈希函数或与简单值或范围匹配，并且用户提供的表达式充当通常称为分区函数的参数。

在 MySQL 8 中，目前`InnoDB`是唯一支持分区的存储引擎。在`InnoDB`存储引擎中，不需要额外的规范来启用分区。分区不能与存储引擎`MyISAM`、`CSV`、`FEDERATED`、`MERGE`一起使用。在本章中给出的所有示例中，我们假设默认存储引擎是`InnoDB`。

创建分区表时，使用默认存储引擎，与创建表时相同，并且可以通过指定`STORAGE ENGINE`选项来覆盖，就像我们对任何表所做的那样。以下示例演示了创建一个分为四个分区的哈希表，所有分区都使用`InnoDB`存储引擎：

```sql
CREATE TABLE tp (tp_id INT, amt DECIMAL(5,2), trx_date DATE)
 ENGINE=INNODB
 PARTITION BY HASH ( MONTH (trx_date) )
 PARTITIONS 4;
```

分区适用于表的所有索引和所有数据。它不适用于索引或数据的任何一方，反之亦然。它可以同时适用于索引和数据，也不能应用于表的一部分。

上述表`tp`没有定义唯一键或主键，但在一般实践中，我们通常有主键、唯一键或两者作为表的一部分，分区列的选择取决于这些键是否存在。分区列的选择在*分区键、主键和唯一键*部分中有详细说明。为了简化分区的概念，所给出的示例可能不包括这些键。

# 分区类型

MySQL 8 支持多种分区类型，列举如下：

+   `RANGE 分区`：根据列值的范围将行分配到分区

+   `LIST 分区`：根据与给定一组值匹配的列值将行分配到分区

+   `COLUMNS 分区`：使用`RANGE`或`LIST`分区将行分配到具有多个列值的分区

+   `HASH 分区`：根据用户指定的表达式在列值上进行评估来分配分区

+   `KEY 分区`：除了`HASH`分区外，还允许使用多个列值

+   子分区：除了分区外，还允许在分区表中进行进一步的划分，也称为复合分区

表的不同行可以分配到不同的物理分区；这被称为水平分区。表的不同列可以分配到不同的物理分区；这被称为垂直分区。MySQL 8 目前支持水平分区。

对于列表、范围和线性哈希类型的分区，分区列的值被传递给分区函数。分区函数返回一个整数值，即记录应存储在其中的分区号。分区函数必须是非随机和非常数的。分区函数不能包含查询，并且可以使用返回整数或 NULL 的 SQL 表达式，其中整数 intval 必须遵循表达式-MAXVALUE <= intval <= MAXVALUE。这里，-MAXVALUE 表示整数类型值的下限，MAXVALUE 是整数类型值的上限。

存储引擎必须对同一表的所有分区相同，但是在同一数据库或 MySQL 服务器中的不同分区表中使用不同的存储引擎没有限制。

# 分区管理

有不同的方法可以使用 SQL 语句修改分区表并执行操作，例如添加、重新定义、合并、删除或拆分现有的分区表。还可以使用 SQL 语句获取有关分区表和分区的信息。

MIN_ROWS 和 MAX_ROWS 可用于配置分区表中存储的最大和最小行数。

# 分区选择和修剪

还提供了分区和子分区的显式选择。它使得行匹配到 where 子句中给定的条件。在分区中，所描述的修剪概念不会扫描可能不存在匹配值的分区，并且使用查询应用，而分区选择适用于查询和许多 DML 语句。

# 分区的限制和限制

存储过程或函数、用户定义函数或插件以及用户变量或声明的变量在分区表达式中受到限制。在详细部分中还有几个适用于分区的限制和限制。

请参阅以下列表，了解分区的一些优点：

+   分区有助于在一个表中存储比文件系统分区或单个磁盘能容纳的更多数据。

+   通过删除仅包含无用数据的分区或分区，可以轻松删除已经变得无用的数据。在某些情况下，需要单独添加特定数据，可以根据指定的规则轻松地在单个或多个分区中进行分区。

+   基于分区数据自动进行的查询优化，不会在不适用于 where 条件的分区中搜索数据。

+   除了分区修剪外，还支持显式的分区选择，其中 where 子句应用于指定的分区或多个分区。

+   通过将数据搜索分离到多个磁盘，可以实现更大的查询吞吐量。

# 分区类型

在本节中，您将了解不同类型的分区以及使用特定分区的目的。以下是 MySQL 8 中可用的分区类型列表：

+   范围分区

+   列表分区

+   列分区

+   哈希分区

+   键分区

+   子分区

除了上述列表，我们还将在 MySQL 8 分区的详细部分中看到对 NULL 的处理。

数据库分区的一个非常常见的用例是按日期对数据进行分隔。MySQL 8 不支持日期分区，一些数据库系统明确提供了日期分区，但可以使用日期、时间或日期时间列创建分区方案，或者基于日期/时间相关表达式创建分区方案，这些表达式评估这些列类型的值。

如果使用`KEY`或`LINEAR KEY`分区，可以使用日期、时间或日期时间类型作为分区列的列值，而不需要进行任何修改，而在其他分区类型中，需要使用返回整数或`NULL`值的表达式。

无论您使用哪种类型的分区，分区始终会自动以整数顺序编号，按照创建顺序对每个分区进行编号。例如，如果表使用四个分区，它们将按照创建顺序分别编号为 0、1、2 和 3。

当您指定分区的数量时，它必须评估为正的、非零的整数，没有前导零。不允许使用小数分数作为分区号。

分区的名称不区分大小写，应遵循约定或规则，就像其他 MySQL 标识符（如表）一样。分区定义中使用的选项已经由`CREATE TABLE`语法提供。

现在，让我们详细查看分区，并检查每种类型，以了解它们之间的不同之处。

# RANGE 分区

在这种类型的分区中，正如名称所示，`RANGE`是在一个表达式中给出的，该表达式评估值是否在给定范围内。范围是使用`VALUES LESS THAN`运算符定义的，它们不应该重叠且应该是连续的。

在接下来的几个示例中，假设我们正在创建一个表，用于保存 25 家食品店的员工个人记录。这些商店的编号从 1 到 25，是一家拥有 25 家食品店的连锁店，如下所示：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
 first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
    termination_date DATE NOT NULL DEFAULT '9999-12-31',
 job_code INT NOT NULL,
 store_id INT NOT NULL
);
```

现在让我们对表进行分区，这样您就可以根据需要按范围对表进行分区。假设您考虑使用除法将数据分为五部分，以`store_id`范围进行分区。为此，表创建定义将如下所示：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
 termination_date DATE NOT NULL DEFAULT '9999-12-31',
 job_code INT NOT NULL,
 store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN (21),
 PARTITION p4 VALUES LESS THAN (26)
);
```

因此，根据上述分区方案，所有插入的行，其中包含在 1 到 5 号店工作的员工，都存储在`p0`分区中，1 到 10 号店工作的员工存储在`p1`分区中，依此类推。如果您查看分区定义，分区按照最低到最高的`store_id`列值排序，`PARTITION BY RANGE`语法看起来类似于编程语句`if… elseif…`语句，不是吗？

好吧，您可能会想知道如果一条记录带有`store_id` `26`会发生什么；这将导致错误，因为服务器不知道在哪里放置记录。有两种方法可以防止发生此错误：

1.  通过在`INSERT`语句中使用`IGNORE`关键字。

1.  使用`MAXVALUE`而不是指定范围（`26`）。

当然，您也可以使用`ALTER TABLE`语句扩展限制，为 26-30 号店、30-35 号店等添加新的分区。

与`store_id`类似，您还可以根据作业代码对表进行分区-基于列值的范围。假设管理职位使用 5 位代码，办公室和支持人员使用 4 位代码，普通工人使用 3 位代码，那么分区表创建定义将如下所示：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
    termination_date DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (job_code) (
 PARTITION p0 VALUES LESS THAN (1000),
 PARTITION p1 VALUES LESS THAN (10000),
    PARTITION p2 VALUES LESS THAN (100000)
);
```

您还可以根据员工加入的年份进行分区，例如根据`YEAR(hired_date)`的值进行分区。现在表定义将如下所示：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
    first_name VARCHAR(30),
    last_name VARCHAR(30),
    hired_date DATE NOT NULL DEFAULT '1990-01-01',
    termination_date DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (YEAR(hired_date)) (
    PARTITION p0 VALUES LESS THAN (1996),
    PARTITION p1 VALUES LESS THAN (2001),
 PARTITION p2 VALUES LESS THAN (2006),
 PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

根据这个方案，所有在`1996`年之前录用的员工记录将存储在分区`p0`中，然后在`2001`年之前录用的记录将存储在分区`p1`中，`2001`年到`2006`年之间的记录将存储在`p2`中，其余的记录将存储在分区`p3`中。

**基于时间间隔的分区方案**可以使用以下两个选项来实现：

1.  通过`RANGE`对表进行分区，并使用在日期、时间或日期时间列值上操作的函数返回整数值作为分区表达式

1.  通过`RANGE COLUMN`对表进行分区，并使用日期、时间或日期时间列作为分区列

`RANGE COLUMN` 在 MySQL 8 中得到支持，并在`COLUMN PARTITIONING`部分有详细描述。

# LIST partitioning

正如其名称所示，`LIST`分区使用列表进行表分区。列表是在使用`VALUES IN (value_list)`进行分区时定义的逗号分隔的整数值；这里，`value_list`指的是逗号分隔的整数文字。

`LIST`分区在许多方面类似于`RANGE`分区，但也有不同之处。每个分区中使用的运算符是不同的。该运算符使用逗号分隔的值列表与列值或评估为整数值的分区表达式进行匹配。

考虑员工表作为一个例子，使用创建表语法的基本定义如下：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
 first_name VARCHAR(30),
 last_name VARCHAR(30),
 hired_date DATE NOT NULL DEFAULT '1990-01-01',
 termination_date DATE NOT NULL DEFAULT '9999-12-31',
 job_code INT NOT NULL,
 store_id INT NOT NULL
);
```

假设您希望将这 25 家食品店分配到五个区域-北、南、东、西和中央，分别使用店铺 ID 号(1,2,11,12,21,22)、(3,4,13,14,23,24)、(5,6,15,16,25)、(7,8,17,18)和(9,10,19,20)。

使用区域列表对表进行分区将为表分区提供以下定义：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
 first_name VARCHAR(30),
 last_name VARCHAR(30),
 hired_date DATE NOT NULL DEFAULT '1990-01-01',
 termination_date DATE NOT NULL DEFAULT '9999-12-31',
 job_code INT NOT NULL,
 store_id INT NOT NULL
)
PARTITION BY LIST (store_id) (
 PARTITION pNorth VALUES IN (1,2,11,12,21,22),
 PARTITION pSouth VALUES IN (3,4,13,14,23,24),
 PARTITION pEast VALUES IN (5,6,15,16,25),
 PARTITION pWest VALUES IN (7,8,17,18),
 PARTITION pCentral VALUES IN (9,10,19,20)
);
```

如前面的陈述所示，按区域进行分区意味着可以很容易地根据特定分区内的区域更新商店的记录。假设组织将西区卖给另一家公司；那么您可能需要使用查询中的`pWest`分区来删除西区的所有员工记录。执行`ALTER TABLE` employee `TRUNCATE PARTITION pWest`比`DELETE`语句`DELETE from employee where store_id IN (7,8,17,18)`更容易和高效；此外，您还可以使用`DROP`语句来删除员工记录- `ALTER TABLE employee DROP PARTITION pWest`。除了前面的语句执行，您还将从表分区定义中删除`pWest PARTITION`，然后需要再次使用`ALTER`语句来添加`pWest PARTITION`并恢复先前的分区表方案。

与`RANGE`分区类似，您还可以使用哈希或键来使用`LIST`分区生成复合分区，这也被称为`子分区`。随后的`子分区`专门部分将更详细地介绍`子分区`。

在`LIST`分区中，没有像`MAXVALUE`这样可以包含所有可能值的捕获机制。相反，您必须在`values_list`中管理预期的值列表，否则`INSERT`语句将导致错误，例如在以下示例中，表中没有值为 9 的分区：

```sql
CREATE TABLE tpl (
 cl1 INT,
 cl2 INT
)
PARTITION BY LIST (cl1) (
 PARTITION p0 VALUES IN (1,3,4,5),
 PARTITION p1 VALUES IN (2,6,7,8)
);

INSERT INTO tpl VALUES (9,5) ;
```

如前面的`INSERT`语句所示，值 9 不是在分区模式中给定的列表的一部分，因此会出现错误。如果使用多个值插入语句，同样的错误可能导致所有插入失败，不会插入任何记录；而是使用`IGNORE`关键字来避免这样的错误，如以下`INSERT`语句示例：

```sql
INSERT IGNORE INTO tpl VALUES (1,2), (3,4), (5,6), (7,8), (9,11);
```

# COLUMNS partitioning

顾名思义，这种类型的分区使用列本身。我们可以使用两种版本的列分区。一种是`RANGE COLUMN`，另一种是`LIST COLUMN`。除了`RANGE COLUMN`和`LIST COLUMN`分区之外，MySQL 8 还支持使用非整数类型的列来定义值范围或列表值。允许的数据类型列表如下：

+   所有`INT`、`BIGINT`、`MEDIUMINT`、`SMALLINT`和`TINYINT`列类型都支持`RANGE`和`LIST`分区列，但不支持其他数值列类型，如`FLOAT`或`DECIMAL`

+   支持`DATE`和`DATETIME`，但不支持与日期和时间相关的其他列类型作为分区列

+   支持字符串列类型`BINARY`、`VARBINARY`、`CHAR`和`VARCHAR`，但不支持`TEXT`和`BLOB`列类型作为分区列

现在，让我们逐一详细查看`RANGE COLUMN`分区和`LIST COLUMN`分区。

# RANGE COLUMN 分区

顾名思义，您可以使用`RANGE`分区和`RANGE COLUMN`分区使用列定义范围，但不同之处在于您可以定义多个列提供范围，并且还可以选择除整数之外的列类型。

因此，`RANGE COLUMN`分区与`RANGE`分区在以下列出的方式上有所不同：

+   `RANGE COLUMNS`可以使用一个或多个列，比较发生在列值列表之间，而不是标量值之间

+   `RANGE COLUMNS`只能使用列名，而不能使用任何表达式

+   `RANGE COLUMNS`分区列类型不仅限于`INTEGER`列类型，还可以使用字符串、日期和日期时间列类型作为分区列。

通过`RANGE COLUMNS`对表进行分区具有以下基本语法：

```sql
CREATE TABLE table_name
PARTITION BY RANGE COLUMNS (column_list) (
 PARTITION partition_name VALUES LESS THAN (value_list) [,
 PARTITION partition_name VALUES LESS THAN (value_list) ] [,
...]
)
column_list:
 column_name[, column_name] [, ...]
value_list :
 value[, value][, ...]
```

在前面的语法中，`column_list`代表分区列列表，`value_list`代表分区定义值列表，并且对于每个分区定义，`value_list`必须给出，并且与`column_list`中定义的相同数量的值一起给出。简而言之，`COLUMNS`子句中的列数（`column_list`）必须与`VALUES LESS THAN`子句中的值数（`value_list`）相同。

以下示例清楚地说明了它是什么以及如何与表定义一起使用：

```sql
CREATE TABLE trc (
 p INT,
    q INT,
    r CHAR(3),
    s INT
)
PARTITION BY RANGE COLUMNS (p,s,r) (
 PARTITION p0 VALUES LESS THAN (5,10,'ppp'),
 PARTITION p1 VALUES LESS THAN (10,20,'sss'),
 PARTITION p2 VALUES LESS THAN (15,30,'rrr'),
 PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
);
```

现在，您可以使用以下语句将记录插入到表`trc`中：

```sql
INSERT INTO trc VALUES (5,9,'aaa',2) , (5,10,'bbb',4) , (5,12,'ccc',6) ;
```

# LIST COLUMN 分区

在这种类型的分区中，表分区定义中使用列列表，并且与`RANGE COLUMN`相似，必须提供相应列的值列表。与`RANGE COLUMN`类似，除了整数类型之外，还可以使用其他列类型，即字符串、日期和日期时间列类型。

假设您有这样的要求，即业务遍布 12 个城市，并且出于营销目的，您将它们分为三个城市的四个区域，如下所示：

+   **Zone 1 with cities**: Ahmedabad, Surat, Mumbai

+   **Zone 2 with cities**: Delhi, Gurgaon, Punjab

+   **Zone 3 with cities**: Kolkata, Mizoram, Hyderabad

+   **Zone 4 with cities**: Bangalore, Chennai, Kochi

现在，为客户数据创建一个表，该表有四个对应区域的分区，并用客户所居住的城市的名称列出它们。表分区定义如下：

```sql
CREATE TABLE customer_z (
 first_name VARCHAR(30),
    last_name VARCHAR(30),
    street_1 VARCHAR(35),
    street_2 VARCHAR(35),
    city VARCHAR(15),
    renewal DATE
)
PARTITION BY LIST COLUMNS (city) (
 PARTITION pZone_1 VALUES IN ('Ahmedabad', 'Surat', 'Mumbai'),
 PARTITION pZone_2 VALUES IN ('Delhi', 'Gurgaon', 'Punjab'),
 PARTITION pZone_3 VALUES IN ('Kolkata', 'Mizoram', 'Hyderabad'),
 PARTITION pZone_4 VALUES IN ('Bangalore', 'Chennai', 'Kochi')

);
```

与`RANGE COLUMN`分区类似，不需要在`COLUMNS()`子句中提供任何将列值转换为整数字面值的表达式，除了列名列表本身之外，不允许提供任何其他内容。

# HASH 分区

引入`HASH`分区的主要目的是确保在定义的分区数量之间均匀分布数据。因此，使用`HASH`分区需要指定要对其进行分区的表的列值或评估列值的表达式，以及要将分区表划分为的分区数。

要在表中定义`HASH`分区，需要在表定义中指定`PARTITION BY HASH (expr)`子句，其中`expr`是将返回整数的表达式，并且还需要使用`PARTITIONS n`指定分区的数量，其中`n`是一个正整数，表示分区的数量。

以下定义创建了一个在`store_id`列上使用`HASH`分区的表，分成了五个分区：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
 first_name VARCHAR(30),
 last_name VARCHAR(30),
 hired_date DATE NOT NULL DEFAULT '1990-01-01',
 termination_date DATE NOT NULL DEFAULT '9999-12-31',
 job_code INT NOT NULL,
 store_id INT NOT NULL
)
PARTITION BY HASH (store_id)
PARTITIONS 4;
```

在上面的语句中，如果排除`PARTITIONS`子句，则分区的数量将自动默认为 1。

# 线性哈希分区

MySQL 8 支持线性哈希，它基于线性二次幂算法，而不是基于哈希函数值的模数的常规哈希。`LINEAR HASH`分区需要在`PARTITION BY`子句中使用`LINEAR`关键字，如下所示：

```sql
CREATE TABLE employee (
 employee_id INT NOT NULL,
 first_name VARCHAR(30),
 last_name VARCHAR(30),
 hired_date DATE NOT NULL DEFAULT '1990-01-01',
 termination_date DATE NOT NULL DEFAULT '9999-12-31',
 job_code INT NOT NULL,
 store_id INT NOT NULL
)
PARTITION BY LINEAR HASH ( YEAR(hired_date))
PARTITIONS 4;
```

使用线性哈希的优势是更快的分区操作，劣势是与常规哈希分区相比数据分布不均匀。

# KEY 分区

这种类型的分区与`HASH`分区类似，只是使用了用户定义的表达式而不是哈希函数。`KEY PARTITIONING`在分区定义的`CREATE TABLE`语句中使用`PARTITION BY KEY`子句。`KEY`分区的语法规则与`HASH`分区类似，因此让我们列出一些不同之处以便理解：

+   在分区时使用`KEY`而不是`HASH`。

在`KEY()`中取一个或多个列名列表，如果在`KEY`中没有定义列，但表具有定义的主键或带有`NOT NULL`约束的唯一键，该列将自动作为`KEY`的分区列：

```sql
CREATE TABLE tk1 (
 tk1_id INT NOT NULL PRIMARY KEY,
    note VARCHAR(50)
)
PARTITION BY KEY ()
PARTITIONS 2;
```

与其他分区类型不同，列类型不仅限于`NULL`或整数值：

```sql
CREATE TABLE tk2 (
 cl1 INT NOT NULL,
 cl2 CHAR(10),
 cl3 DATE
)
PARTITION BY LINEAR KEY (cl1)
PARTITIONS 3;
```

如前面的示例语句所示，与`HASH`分区类似，`KEY`分区也支持`LINEAR KEY`分区，并且与`LINEAR HASH`分区具有相同的效果。

# 子分区

子分区也被称为复合分区，正如其名称所示，它只是将每个分区分成一个分区表本身。请参阅以下语句：

```sql
CREATE TABLE trs (trs_id INT, sold DATE)
PARTITION BY RANGE ( YEAR(sold) )
    SUBPARTITION BY HASH ( TO_DAYS(sold) )
    SUBPARTITIONS 2 (
        PARTITION p0 VALUES LESS THAN (1991),
        PARTITION p1 VALUES LESS THAN (2001),
        PARTITION p2 VALUES LESS THAN MAXVALUE
);
```

如前面的示例语句所示，表`trs`有三个`RANGE`分区，每个分区`p0、p1、p2`进一步分成两个子分区。有效地，整个表分成了六个分区。

使用`RANGE`或`LIST`分区的表可以进行子分区，并且子分区可以使用`KEY`或`HASH`分区类型。子分区的语法规则与常规分区相同，唯一的例外是在`KEY`分区中指定默认列，因为它不会自动为子分区获取列。

在使用子分区时需要考虑以下几点：

+   每个定义的分区的分区数量必须相同。

+   必须在`SUBPARTITIONING`子句中指定名称，或者指定一个默认选项。

+   指定的子分区名称必须在整个表中是唯一的。

# 分区中的 NULL 处理

MySQL 8 没有特定于禁止`NULL`作为分区的列值、分区表达式或用户定义表达式的内容。即使`NULL`被允许作为一个值，表达式返回的值也必须是整数，因此 MySQL 8 对分区的实现是将`NULL`视为小于`ORDER BY`子句中的任何非`NULL`值。

`NULL`处理的行为在不同类型的分区中有所不同：

+   **在`RANGE`分区中处理`NULL`**：如果插入了包含`NULL`值的列，行将被插入到范围中指定的最低分区中。

+   **使用`LIST`分区处理`NULL`值**：如果表具有使用`LIST`分区定义的分区，并且其分区使用值列表明确指定`NULL`作为`value_list`中的值，则插入将成功；否则，将出现错误，因为表没有为`NULL`指定分区。

+   **使用`HASH`和`KEY`分区处理`NULL`值**：当使用`HASH`或`KEY`分区定义表分区时，`NULL`的处理方式不同，如果分区表达式返回`NULL`，它将被包装为零值。因此，根据分区插入操作将成功将记录插入到零分区。

# 分区管理

使用`SQL`语句修改分区表有很多方法——您可以使用`ALTER TABLE`语句删除、添加、合并、拆分或重新定义分区。还有一些方法可以检索分区表和分区信息。我们将在以下部分中看到每个方法：

+   `RANGE`和`LIST`分区管理

+   `HASH`和`KEY`分区管理

+   分区维护

+   获取分区信息

# RANGE 和 LIST 分区管理

对于`RANGE`和`LIST`分区类型，添加和删除分区的处理方式类似。可以使用`ALTER TABLE`语句的`DROP PARTITION`选项删除通过`RANGE`或`LIST`分区进行分区的表。

在执行`ALTER TABLE ... DROP PARTITION`语句之前，请确保您拥有`DROP`权限。`DROP PARTITION`将删除所有数据，并从表分区定义中删除分区。

以下示例说明了`ALTER TABLE`语句的`DROP PARTITION`选项：

```sql
SET @@SQL_MODE = '';
CREATE TABLE employee (
 id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 first_name VARCHAR(25) NOT NULL,
 last_name VARCHAR(25) NOT NULL,
 store_id INT NOT NULL,
 department_id INT NOT NULL
) 
 PARTITION BY RANGE(id) (
 PARTITION p0 VALUES LESS THAN (5),
 PARTITION p1 VALUES LESS THAN (10),
 PARTITION p2 VALUES LESS THAN (15),
 PARTITION p3 VALUES LESS THAN MAXVALUE
);
INSERT INTO employee VALUES
 ('', 'Chintan', 'Mehta', 3, 2), ('', 'Bhumil', 'Raval', 1, 2),
 ('', 'Subhash', 'Shah', 3, 4), ('', 'Siva', 'Stark', 2, 4),
 ('', 'Chintan', 'Gajjar', 1, 1), ('', 'Mansi', 'Panchal', 2, 3),
 ('', 'Hetal', 'Oza', 2, 1), ('', 'Parag', 'Patel', 3, 1),
 ('', 'Pooja', 'Shah', 1, 3), ('', 'Samir', 'Bhatt', 2, 4),
 ('', 'Pritesh', 'Shah', 1, 4), ('', 'Jaymin', 'Patel', 3, 2),
 ('', 'Ruchek', 'Shah', 1, 2), ('', 'Chandni', 'Patel', 3, 3),
 ('', 'Mittal', 'Patel', 2, 3), ('', 'Shailesh', 'Patel', 2, 2),
 ('', 'Krutika', 'Dave', 3, 3), ('', 'Dinesh', 'Patel', 3, 2);

ALTER TABLE employee DROP PARTITION p2;
```

在执行`ALTER TABLE employee DROP PARTITION p2;`语句后，您可以看到所有数据都从分区`p2`中删除。如果您想删除所有数据但又需要保留表定义和分区方案，可以使用`TRUNCATE PARTITION`选项来实现类似的结果。

要向现有分区表添加新的`LIST`或`RANGE`分区，可以使用`ALTER TABLE ... ADD PARTITION`语句。

通过使用`SHOW CREATE TABLE`语句，您可以验证并查看`ALTER TABLE`语句对表定义和分区模式的影响是否符合预期。

# HASH 和 KEY 分区管理

`HASH`或`KEY`类型的表分区与`RANGE`或`LIST`类型的分区相似。如果表是通过`HASH`或`KEY`类型的分区进行分区，那么无法删除分区，但可以使用`ALTER TABLE ... COALESCE PARTITION`选项合并`HASH`或`KEY`分区。

假设您有一个客户表数据，通过`HASH`分区分割，分为十二个分区如下：

```sql
CREATE TABLE client (
 client_id INT,
 first_name VARCHAR(25),
 last_name VARCHAR(25),
 signed DATE
)
PARTITION BY HASH (MONTH (signed))
PARTITIONS 12;
```

在上述表分区模式中，如果您想将分区数量从十二个减少到八个，可以使用以下`ALTER TABLE`语句：

```sql
ALTER TABLE client COALESCE PARTITION 8;
```

在上述语句中，数字 8 表示要从表中删除的分区数量。您不能删除超过表分区模式中已存在的分区数量。同样，您可以使用`ALTER TABLE... ADD PARTITION`语句添加更多分区。

# 分区维护

有许多维护任务可以通过多个语句在多个表和分区上完成。可以使用`ANALYSE TABLE`、`CHECK TABLE`、`REPAIR TABLE`和`OPTIMIZE TABLE`等语句，这些语句专门用于支持分区表。

有许多`ALTER TABLE`的扩展可用于单个或多个分区表的操作，列举如下：

+   **重建分区**：此选项会删除分区中的所有记录并重新插入，因此在碎片整理过程中很有帮助。以下是一个示例：

```sql
 ALTER TABLE trp REBUILD  PARTITION p0, p1, p2;
```

+   **优化分区**：如果从表的一个或多个分区中删除了许多行，或者在可变长度列类型（如`VARCHAR`、`BLOB`、`TEXT`等）的大量数据中有许多行更改，可以执行`OPTIMIZE PARTITION`来回收分区数据文件中未使用的空间。以下是一个例子：

```sql
 ALTER TABLE top OPTIMIZE PARTITION p0, p1, p2;
```

`ALTER TABLE ... OPTIMIZE PARTITION`与`InnoDB`存储引擎不兼容，因此应改用`ALTER TABLE ... REBUILD PARTITION`和`ALTER TABLE ... ANALYZE PARTITION`。

+   **分析分区**：在此选项中，读取并存储分区的关键分布。以下是一个例子：

```sql
 ALTER TABLE tap ANALYZE  PARTITION p1, p2;
```

+   **修复分区**：仅在发现损坏的分区需要修复时使用。以下是一个例子：

```sql
 ALTER TABLE trp REPAIR PARTITION p3;
```

+   **检查分区**：此选项用于检查分区中的任何错误，例如在非分区表中使用的`CHECK TABLE`选项。以下是一个例子：

```sql
 ALTER TABLE tcp CHECK PARTITION p0;
```

在所有上述选项中，有一个选项可以使用`ALL`而不是特定分区，以便对所有分区执行操作。

# 获取分区信息

可以通过多种方式获取有关分区的信息，如下所示：

+   `SHOW CREATE TABLE`语句可用于查看包含分区表中所有分区子句的分区模式信息

+   `SHOW TABLE STATUS`语句可用于通过查看其状态检查表是否已分区

+   `EXPLAIN SELECT`语句可用于查看给定`SELECT`选项使用的分区

+   使用`INFORMATION_SCHEMA.PARTITIONS`表查询分区表信息。

以下是使用`SHOW CREATE TABLE`语句选项查看分区信息的示例：

```sql
SHOW CREATE TABLE employee;
```

从前述语句的输出中，可以看到分区模式的单独信息，包括表模式的常见信息。

同样，您可以从`INFORMATION_SCHEMA.PARTITIONS`表中检索有关分区的信息。

`EXPLAIN`选项提供了许多有关分区的信息。例如，它提供了从特定于分区的查询中获取的行数。分区将根据查询语句进行搜索。它还提供有关键的信息。

`EXPLAIN`也用于从非分区表中获取信息。如果没有分区，则不会出现任何错误，但在分区列中会给出一个`NULL`值。

# 分区选择和修剪

在本节中，您将看到分区如何通过称为分区修剪的优化器来优化`SQL`语句的执行，并使用`SQL`语句有效地选择分区数据并对分区进行修改操作。

# 分区修剪

分区修剪与分区中的优化概念相关。在分区修剪中，基于查询语句应用了“不要扫描可能不存在匹配值的分区”的概念。

假设有一个分区表`tp1`，使用以下语句创建：

```sql
CREATE TABLE tp1 (
 first_name VARCHAR (30) NOT NULL,
 last_name VARCHAR (30) NOT NULL,
 zone_code TINYINT UNSIGNED NOT NULL,
 doj DATE NOT NULL
)
PARTITION BY RANGE (zone_code) (
 PARTITION p0 VALUES LESS THAN (65),
 PARTITION p1 VALUES LESS THAN (129),
 PARTITION p2 VALUES LESS THAN (193),
 PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

在前面的示例表`tp1`中，假设您想从以下`SELECT`语句中检索结果：

```sql
SELECT first_name, last_name , doj from tp1 where zone_code > 126 AND zone_code < 131;
```

现在，您可以从前述语句中看到，根据该语句，没有数据存在于分区`p0`或`p3`中，因此我们只需要在`p1`或`p2`中搜索匹配的数据。因此，通过限制搜索，可以在表的所有分区中花费更少的时间和精力进行匹配和搜索数据。这种去除不匹配分区的操作称为修剪。

优化器可以利用分区修剪来执行查询执行，与具有相同模式、数据和查询语句的非分区表相比，速度更快。

优化器可以根据`WHERE`条件的减少在以下情况下进行修剪：

+   `partition_column IN (constant1, constant2, ..., contantN)`

+   `partition_column = constant`

在第一种情况下，优化器评估列表中每个值的分区表达式，并创建在评估期间匹配的分区列表，然后只在此分区列表中执行扫描或搜索。

在第二种情况下，优化器仅根据给定的常量或特定值评估分区表达式，并确定哪个分区包含该值，并且只在此分区上执行搜索或扫描。在这种情况下，可以使用另一个算术比较而不是等于。

目前，修剪不支持`INSERT`语句，但支持`SELECT`，`UPDATE`和`DELETE`语句。

修剪也适用于优化器可以将范围转换为等效值列表的短范围。当分区表达式由可以减少为相等集的相等性或范围组成，或者分区表达式表示递增或递减关系时，可以应用优化器。

修剪也适用于使用`TO_DAYS()`或`YEAR()`函数进行分区的`DATE`或`DATETIME`列类型，并且如果这些表在其分区表达式中使用`TO_SECONDS()`函数，也适用。

假设您有一个表`tp2`，如下语句所示：

```sql
CREATE TABLE tp2 (
 first_name VARCHAR (30) NOT NULL,
 last_name VARCHAR (30) NOT NULL,
 zone_code TINYINT UNSIGNED NOT NULL,
 doj DATE NOT NULL
)
PARTITION BY RANGE (YEAR(doj)) (
 PARTITION p0 VALUES LESS THAN (1971),
 PARTITION p1 VALUES LESS THAN (1976),
 PARTITION p2 VALUES LESS THAN (1981),
 PARTITION p3 VALUES LESS THAN (1986),
 PARTITION p4 VALUES LESS THAN (1991),
 PARTITION p5 VALUES LESS THAN (1996),
 PARTITION p6 VALUES LESS THAN (2001),
 PARTITION p7 VALUES LESS THAN (2006),
 PARTITION p8 VALUES LESS THAN MAXVALUE
);
```

现在，在前面的语句中，以下语句可以从分区修剪中受益：

```sql
SELECT * FROM tp2  WHERE doj = '1982-06-24';
UPDATE tp2  SET region_code = 8 WHERE doj BETWEEN '1991-02-16' AND '1997-04-26';
DELETE FROM tp2  WHERE doj >= '1984-06-22' AND doj <= '1999-06-22';
```

对于最后一条语句，优化器可以采取以下行动：

1.  找到具有范围低端的分区为`YEAR('1984-06-22')`，得到值 1984，找到在`p3`分区中。

1.  找到具有范围高端的分区为`YEAR('1999-06-22')`，得到值 1999，找到在`p5`分区中。

1.  仅扫描上述两个确定的分区和它们之间的任何分区。

因此，在上述情况下，要扫描的分区仅为`p3`，`p4`和`p5`，而其余分区在匹配时可以忽略。

前面的示例使用了`RANGE`分区，但分区修剪也适用于其他类型的分区。假设您有表`tp3`的模式如下语句所示：

```sql
CREATE TABLE tp3 (
 first_name VARCHAR (30) NOT NULL,
 last_name VARCHAR (30) NOT NULL,
 zone_code TINYINT UNSIGNED NOT NULL,
 description VARCHAR (250),
 doj DATE NOT NULL
)
PARTITION BY LIST(zone_code) (
 PARTITION p0 VALUES IN (1, 3),
 PARTITION p1 VALUES IN (2, 5, 8),
 PARTITION p2 VALUES IN (4, 9),
 PARTITION p3 VALUES IN (6, 7, 10)
);
```

对于前面的表模式，请考虑是否要执行此语句`SELECT * FROM tp3 WHERE zone_code BETWEEN 1 AND 3`。优化器确定哪些分区可以具有值`1`，`2`和`3`，并找到`p1`和`p0`，因此跳过其余的分区`p3`和`p2`。

具有常量的列值可以被修剪，如以下示例语句：

```sql
UPDATE tp3 set description = 'This is description for Zone 5' WHERE zone_code = 5;
```

只有当范围的大小小于分区数时才执行优化。

# 分区选择

还支持显式选择分区和子分区，这使得行匹配到 where 子句中给定的条件 - 这称为分区选择。它与分区修剪非常相似，因为只有特定的分区用于匹配，但在以下两个关键方面有所不同：

+   要扫描的分区由发出语句的人指定，而不是像分区修剪那样自动进行。

+   分区修剪仅限于查询，而分区选择支持查询和多个`DML`语句

支持显式分区选择的 SQL 语句如下所示：

+   `INSERT`

+   `SELECT`

+   `UPDATE`

+   `REPLACE`

+   `LOAD DATA`

+   `LOAD XML`

+   `DELETE`

用于显式分区选择的`PARTITION`选项的以下语法：

```sql
PARTITION (partition_names)
partition_names :
 partition_name, ...
```

上述选项总是跟随它所属的表结构或表模式。`partition_names`代表分区或子分区的逗号分隔名称列表，将用于分区。`partition_names`中的分区和子分区名称可以是任何顺序，甚至可以重叠，但列表中的每个名称必须是特定表的现有分区或子分区名称，否则语句将失败，并显示错误消息`partition_name`不存在。

如果使用`PARTITION`选项，只有列出的分区和子分区才会被检查匹配的行。`PARTITION`选项也可以用于`SELECT`语句，以检索属于任何给定分区的行。

假设你使用以下语句创建了表`employee`：

```sql
SET @@SQL_MODE = '';
CREATE TABLE employee (
 id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 first_name VARCHAR(25) NOT NULL,
 last_name VARCHAR(25) NOT NULL,
 store_id INT NOT NULL,
 department_id INT NOT NULL
) 
 PARTITION BY RANGE(id) (
 PARTITION p0 VALUES LESS THAN (5),
 PARTITION p1 VALUES LESS THAN (10),
 PARTITION p2 VALUES LESS THAN (15),
 PARTITION p3 VALUES LESS THAN MAXVALUE
);
INSERT INTO employee VALUES
 ('', 'Chintan', 'Mehta', 3, 2), ('', 'Bhumil', 'Raval', 1, 2),
 ('', 'Subhash', 'Shah', 3, 4), ('', 'Siva', 'Stark', 2, 4),
 ('', 'Chintan', 'Gajjar', 1, 1), ('', 'Mansi', 'Panchal', 2, 3),
 ('', 'Hetal', 'Oza', 2, 1), ('', 'Parag', 'Patel', 3, 1),
 ('', 'Pooja', 'Shah', 1, 3), ('', 'Samir', 'Bhatt', 2, 4),
 ('', 'Pritesh', 'Shah', 1, 4), ('', 'Jaymin', 'Patel', 3, 2),
 ('', 'Ruchek', 'Shah', 1, 2), ('', 'Chandni', 'Patel', 3, 3),
 ('', 'Mittal', 'Patel', 2, 3), ('', 'Shailesh', 'Patel', 2, 2),
 ('', 'Krutika', 'Dave', 3, 3), ('', 'Dinesh', 'Patel', 3, 2);
```

现在，如果你检查分区`p1`，你会看到以下输出，因为行添加到分区`p1`中：

```sql
mysql> SELECT * FROM employee PARTITION (p1);
+----+-----------+------------+----------+---------------+
| id | last_name | last_name | store_id | department_id |
+----+-----------+------------+----------+---------------+
| 5 | Chintan | Gajjar | 1 | 1 |
| 6 | Mansi | Panchal | 2 | 3 |
| 7 | Hetal | Oza | 2 | 1 |
| 8 | Parag | Patel | 3 | 1 |
| 9 | Pooja | Shah | 1 | 3 |
+----+-----------+------------+----------+---------------+
5 rows in set (0.00 sec) 
```

如果使用这个语句`SELECT * FROM employee WHERE id BETWEEN 5 AND 9;`，将得到相同的输出。

为了从多个分区中检索行，可以使用逗号分隔的分区名称列表。例如，`SELECT * FROM employee PARTITION (p1,p2)`，将得到来自分区`p1`和`p2`的所有行，并排除其余分区。

可以使用任何支持的分区类型来使用分区选择语句。当使用`LINEAR HASH`或`LINEAR KEY`分区类型创建表时，MySQL 8 会自动添加分区名称，而且这也适用于子分区。在对这个表执行`SELECT`语句时，可以指定 MySQL 8 生成的分区名称来检索特定分区的数据。

`PARTITION`选项也适用于`SELECT`语句，用于`INSERT ... SELECT`语句，可以插入从特定分区或子分区检索的数据。

`PARTITION`选项也适用于具有特定分区或子分区数据的表的连接查询的`SELECT`语句。

# 分区中的限制和限制

在本节中，您将看到 MySQL 8 分区中的限制和限制，涵盖了禁止的结构、性能考虑和与存储引擎和函数相关的限制方面的详细信息，以便从表分区中获得最佳效益。

# 分区键、主键和唯一键

分区键与主键和唯一键之间的关系对于分区模式结构设计非常重要。简而言之，规则是分区表中用于分区的所有列必须包括表的每个唯一键。因此，包括表上的主键列在内的每个唯一键都必须是分区表达式的一部分。看一下以下使用不符合规则的唯一键的`CREATE TABLE`语句的例子：

```sql
CREATE TABLE tk1 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 UNIQUE KEY (cl1, cl2)
)
PARTITION BY HASH(cl3)
PARTITIONS 4;

CREATE TABLE tk2 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 UNIQUE KEY (cl1),
 UNIQUE KEY (cl3)
)
PARTITION BY HASH(cl1 + cl3)
PARTITIONS 4;
```

在上述每个用于创建表`tk1`和`tk2`的语句中，建议的表可以至少有一个唯一键，该键不包括分区表达式中的所有列。

现在看一下以下修改后的表创建语句，这些语句已经可以工作，并且从无效变为有效：

```sql
CREATE TABLE tk1 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 UNIQUE KEY (cl1, cl2, cl3)
)
PARTITION BY HASH(cl3)
PARTITIONS 4;

CREATE TABLE tk2 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 UNIQUE KEY (cl1, cl3)
)
PARTITION BY HASH(cl1 + cl3)
PARTITIONS 4;
```

如果你看一下以下表结构，它根本无法分区，因为没有办法包含可以成为分区键列的唯一键列：

```sql
CREATE TABLE tk4 (
 cl1 INT NOT NULL,
 cl2 INT NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 UNIQUE KEY (cl1, cl3),
 UNIQUE KEY (cl2, cl4)
);
```

根据定义，每个主键都是唯一键。这个限制也适用于表的主键。以下是表`tk5`和`tk6`的两个无效语句的例子：

```sql
CREATE TABLE tk5 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 PRIMARY KEY(cl1, cl2)
)
PARTITION BY HASH(cl3)
PARTITIONS 4;

CREATE TABLE tk6 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 PRIMARY KEY(cl1, cl3),
 UNIQUE KEY(cl2)
)
PARTITION BY HASH( YEAR(cl2) )
PARTITIONS 4;
```

在上述两个语句中，所有引用的列都不包括相应的主键在分区表达式中。以下语句是有效的：

```sql
CREATE TABLE tk7 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 PRIMARY KEY(cl1, cl2)
)
PARTITION BY HASH(cl1 + YEAR(cl2))
PARTITIONS 4;

CREATE TABLE tk8 (
 cl1 INT NOT NULL,
 cl2 DATE NOT NULL,
 cl3 INT NOT NULL,
 cl4 INT NOT NULL,
 PRIMARY KEY(cl1, cl2, cl4),
 UNIQUE KEY(cl2, cl1)
)
PARTITION BY HASH(cl1 + YEAR(cl2))
PARTITIONS 4;
```

如果表没有唯一键或主键，则该限制不适用，并且可以根据分区类型的兼容列类型在分区表达式中使用任何列。所有上述限制也适用于`ALTER TABLE`语句。

# 与存储引擎相关的分区限制

分区支持不是由 MySQL 服务器提供的，而是来自 MySQL 8 中的存储引擎自己或本机分区处理程序。在 MySQL 8 中，`InnoDB`存储引擎只提供本机分区处理程序，因此分区表的创建不适用于任何其他存储引擎。

`ALTER TABLE ... OPTIMIZE PARTITION`在`InnoDB`存储引擎中无法正确工作，因此请改用`ALTER TABLE ... REBUILD PARTITION`和`ALTER TABLE ... ANALYZE PARTITION`操作。

# 与函数相关的分区限制

在分区表达式中，只有以下列出的 MySQL 8 函数是允许的：

+   `ABS()`: 它为给定参数提供了绝对值

+   `CEILING()`: 它为给定参数提供可能的最小整数

+   `DAY()`: 它为给定日期提供月份中的日期

+   `DAYOFMONTH()`: 它提供给定日期的月份中的日期，与`DAY()`相同

+   `DAYOFWEEK()`: 它为给定日期提供星期几的编号

+   `DAYOFYEAR()`: 它为给定日期提供一年中的日期

+   `DATEDIFF()`: 它提供两个给定日期之间的天数

+   `EXTRACT()`: 它提供给定参数的一部分

+   `FLOOR()`: 它为给定参数提供了可能的最大整数值

+   `HOUR()`: 它从给定参数中提供小时数

+   `MICROSECOND()`: 它从给定参数中提供微秒数

+   `MINUTE()`: 它从给定参数中提供分钟数

+   `MOD()`: 它执行模运算并提供`N`除以`M`的余数，其中`MOD(N,M)`

+   `MONTH()`: 它从给定参数中提供月份

+   `QUARTER()`: 它从给定参数中提供季度

+   `SECOND()`: 它从给定参数中提供秒数

+   `TIME_TO_SEC()`: 它从给定时间值参数中提供秒数

+   `TO_DAYS()`: 它为给定参数提供了从公元 0 年开始的天数

+   `TO_SECONDS()`: 它为给定参数提供从公元 0 年开始的秒数

+   `UNIX_TIMESTAMP() (with TIMESTAMP columns)`: 它为给定参数提供自'1970-01-01 00:00:00' UTC 以来的秒数

+   `WEEKDAY()`: 它为给定参数提供星期几的索引

+   `YEAR()`: 它为给定参数提供年份

+   `YEARWEEK()`: 它为给定参数提供年份和周数

分区修剪支持 MySQL 8 中的`TO_DAYS()`、`TO_SECONDS()`、`TO_YEAR()`和`UNIX_TIMESTAMP()`函数。

# 总结

在本章中，我们学习了不同类型的分区和分区的需求。我们还详细介绍了管理所有类型的分区的信息。我们学习了分区修剪和选择分区，这是优化器使用的。我们还讨论了在使用分区时需要考虑的适用限制和限制。

在下一章中，您将学习如何在 MySQL 8 中进行扩展，并了解在提供 MySQL 8 可扩展性时面临的常见挑战。您还将学习如何使 MySQL 服务器高度可用并实现高可用性。
