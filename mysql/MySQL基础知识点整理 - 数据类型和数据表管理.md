MySQL基础知识点整理 - 数据表管理
---

## 〇、数据类型

### 1. 数值数据类型

数值数据类型存储数值。
MySQL支持多种数值数据类型，每种存储的数值具有不同的取值范围。

#### **整数**

类型	|大小	|范围（有符号）|	范围（无符号）
-|-|-|-
TINYINT	|1 字节|	(-128，127)|	(0，255)	
SMALLINT|	2 字节|	(-32768，32767)|	(0，65535)	
MEDIUMINT|	3 字节|	(-8388608，8388607)|	(0，16777215)	
INT或INTEGER|4 字节|	(-2147483648，2147483647)|		(0，4294967295)
BIGINT	|8 字节|(-2^63^, 2^63^ - 1)	|	(0，2^64^)

* 长度 `int(n)` 与 `zerofill`

`int(n)` 只影响显示字符的宽度，不限制数值的合法范围。
`int(3)` 依然可以存储 `123456789` 这么大的数值。
若设置了 `zerofill` 属性，当 `int(3)` 存储 `12` 时，会在前面补0，补足3位。即 `012`；当 `int(5)` 存储 `12` 时，会在前面补三个0，补足5位。即 `00012`

* 有符号或无符号

所有数值数据类型（除 `BIT` 和 `BOOLEAN` 外）都可以有符号或无符号。有符号数值列可以存储正或负的数值，无符号数值列只能存储正数。默认情况为有符号，但如果你知道自己不需要存储负值，可以使用 `UNSIGNED` 关键字，这样做将允许你存储两倍大小的值。


#### **小数**
类型	|大小	|范围（有符号）|	范围（无符号）
-|-|-|-
FLOAT	|4 字节|(-3.402823466 E+38，-1.175494351 E-38)，0，(1.175494351 E-38，3.402823466351 E+38)	|	0，(1.175 494351 E-38，3.402823466 E+38)
DOUBLE|	8 字节|		(-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)|		0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)	
DECIMAL|		对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2	|	依赖于M和D的值	|依赖于M和D的值

> DECIMAL最常用的用法就是用来存储货币，例如 ` DECIMAL(8, 2)`。
> DECIMAL还可以用于存储比BIGINT还大的整数以及精确的小数。


### 2. 串数据类型

类型|	大小|	用途
-|-|-
CHAR|	0 - 255 个字符	|定长字符串
VARCHAR|	0 - 65535 字节	|变长字符串
TINYTEXT|	0 - 255 字节	|短文本字符串
TEXT	|0 - 65535 字节	|长文本数据（<64KB）
MEDIUMTEXT	|0 - 16777215 字节|	中等长度文本数据（<16MB）
LONGTEXT	|0 - 4294967295 字节	|极大文本数据（<4GB）

> 从 MySQL4.1 版本开始，`char(n)` 和 `varchar(n)` 中的 `n` 指字符长度，不再表示之前版本的字节长度。也就是说在不同字符集下，char类型列的内部存储可能不是定长数据。

#### **CHAR***

CHAR 是定长字符串，会直接根据定义字符串时指定的长度分配足够的空间。
CHAR 适合存储所有值长度相同的字符串或很短的字符串。

#### **VARCHAR**

VARCHAR 的最大长度是65535个字节，而 `varchar(n)` 中的 `n` 指字符长度，因此，`n` 的最大值是由当前字段的字符集决定的。当字符集是 `utf8` 时，`n` 的最大值为 21845。当字符集是 `utf8mb4` 时，`n` 的最大值为 16383。（但是实际上MySQL要求一个行的定义长度不能超过65535个字节，因此，除非表中只有这一个字段，否则 `n` 的值达不到上述的最大值）。

VARCHAR 使用1-2个额外字节记录字符串长度，列长度小于等于255个字符时，使用1个字节记录，否则使用2个字节。

#### **最佳实践**

* 对于经常变更的数据， CHAR 比 VARCHAR 更好，CHAR 的磁盘空间利用率更高，不容易产生碎片。
* 当列中数据的长度相同时，选择 CHAR；当列中数据长度参差不齐时，选择 VARCHAR。
* 对于非常短的列，CHAR 比 VARCHAR 在存储上更有效率。
* 只分配真正需要的空间，更长的列会消耗更多的内存。
* 尽量避免使用 BLOB/TEXT 类型，查询时会使用临时表，导致严重的性能开销。如果一定要用，建议单独建表存储该字段。

### 3. 二进制数据类型

类型|	大小|	用途
-|-|-
TINYBLOB	|0 - 255 字节	|不超过 255 个字符的二进制字符串
BLOB	|0 - 65535 字节	|二进制形式的长文本数据（<64KB）
MEDIUMBLOB|	0 - 16777215 字节	|二进制形式的中等长度文本数据（<16MB）
LONGBLOB|	0 - 4294967295 字节	|二进制形式的极大文本数据（<4GB）

### 4. 日期和时间类型

类型	|大小(字节)|	范围	|格式|	用途
-|-|-|-|-
YEAR|	1	|1901 / 2155|	YYYY	|年份值
DATE|	3|	1000-01-01 / 9999-12-31|	YYYY-MM-DD	|日期值
DATETIME	|8|	1000-01-01 00:00:00/9999-12-31 23:59:59|	YYYY-MM-DD HH:MM:SS|	混合日期和时间值
TIMESTAMP|	4	|1970-01-01 00:00:00/2038<br>结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07|YYYYMMDD HHMMSS|	混合日期和时间值，时间戳

#### **最佳实践**

* 尽量使用 TIMESTAMP，比 DATETIME 的空间利用率高。

## 一、创建数据表

#### **CREATE TABLE**

使用CREATE TABLE 创建表，必须给出下列信息：

* 表的名字，在关键字 CREATE TABLE 之后给出；
* 表中字段的名字和定义，用逗号分隔。

以下为创建MySQL数据表的SQL通用语法：
```sql
CREATE TABLE table_name (
	column1 datatype [NULL|NOT NULL] [DEFAULT ],
	column2 datatype,
);
```

#### **实例**

* 创建用户表
```sql
CREATE TABLE IF NOT EXISTS `user` (
	`id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  	`username` varchar(190) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  	`email` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  	`password` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '密码',
  	`status` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '状态',
  	`created_at` int(11) unsigned NOT NULL,
	PRIMARY KEY (`id`) USING BTREE,
  	UNIQUE KEY `unq_email` (`email`) USING BTREE,
 	KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci COMMENT '用户表';
```

#### **实例解析**

**如果数据库中不存在 `user` 表时，创建该表。存储引擎为 `InnoDB`，默认字符集为`utf8`**
ENGINE 设置存储引擎，CHARSET 设置编码。
```sql
CREATE TABLE IF NOT EXISTS `user` (
...
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci COMMENT '用户表'
```

**创建名为 `id` 的字段，整型，非负数，不能为空，自增。**

如果你不想字段为 NULL 可以设置字段的属性为 NOT NULL， 在操作数据库时如果输入该字段的数据为NULL ，就会报错。
AUTO_INCREMENT定义列为自增的属性，一般用于主键，数值会自动加1。
```sql
`id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT
```

**创建名为 `username` 的字段，字符串类型，最大长度为190个字符，字符集为 `utf8mb4`，不能为空**
```sql
`username` varchar(190) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
```

**将字段 `id` 设为主键，使用 `BTREE` 索引 **

PRIMARY KEY关键字用于定义列为主键。 可以使用多列来定义主键，列间以逗号分隔。
```sql
PRIMARY KEY (`id`) USING BTREE,
```

**为字段 `email` 添加唯一索引，索引名称为 `unq_email`**
设置了唯一索引的字段不能出现重复的值，但是如果字段可以为 `null`，则允许出现多个 `null` 值。
```sql
UNIQUE KEY `unq_email` (`email`) USING BTREE
```

**为字段 `username` 添加普通索引，索引名称为 `idx_username`**
```
KEY `idx_username` (`username`)
```


## 二、查看数据表

### 1. 查看数据库中的所有数据表

`SHOW TABLES` 用于查看数据库中的所有数据表。

```sql
mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| user           |
+----------------+
1 row in set (0.07 sec)
```

### 2. 查看数据表的建表SQL语句

`SHOW CREATE TABLE` 用于查看指定数据表的建表SQL语句

语法：
```sql
SHOW CREATE TABLE table_name
```

* 查看 `user` 表的建表语句
```sql
mysql> SHOW CREATE TABLE `user`\G
*************************** 1. row ***************************
       Table: user
Create Table: CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(190) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  `email` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  `password` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL COMMENT '密码',
  `status` tinyint(3) unsigned NOT NULL DEFAULT '0' COMMENT '状态',
  `created_at` int(11) unsigned NOT NULL,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `unq_email` (`email`) USING BTREE,
  KEY `idx_username` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci COMMENT='用户表'
1 row in set (0.00 sec)

```

### 3. 查看数据表结构

#### **DESCRIBE 和 DESC**

`DESCRIBE` 可用于查看表结构，`DESC` 是 `DESCRIBE` 的缩写。

语法：
```sql
DESCRIBE table_name
```

* 查看 `user` 表的表结构
```sql
mysql> DESCRIBE `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
6 rows in set (0.00 sec)
```
```sql
mysql> DESC `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
6 rows in set (0.00 sec)
```

#### **EXPLAIN**

`EXPLAIN` 也可以用于查看表结构。

语法：
```sql
EXPLAIN table_name
```

`DESCRIBE` 和 `EXPLAIN` 语句是同义词，实际上在平时使用过程中 `DESCRIBE` 多用于获取表结构的信息，而 `EXPLAIN` 多用于获取SQL语句的执行计划。

* 查看 `user` 表的表结构
```sql
mysql> EXPLAIN `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
6 rows in set (0.03 sec)
```

## 三、修改数据表

### 1. 重命名数据表

* 语法
```
RENAME TABLE old_name TO new_name;
```

* 将 `user` 表重命名为 `consumer` ，再改回 `user`
```sql
mysql> RENAME TABLE `user` TO `consumer`;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| consumer       |
+----------------+
1 row in set (0.06 sec)

mysql> RENAME TABLE `consumer` TO `user`;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| user           |
+----------------+
1 row in set (0.05 sec)
```

### 2. 增加字段

* 语法
```sql
ALTER TABLE table_name ADD column_name column_type
```

* 给 `user` 表添加一个字段 `intro`
```sql
mysql> ALTER TABLE `user` ADD `intro` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci COMMENT '简介' AFTER `email`;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| intro      | varchar(255)        | YES  |     | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
7 rows in set (0.06 sec)
```

### 3. 修改字段

#### **修改字段名和属性**

* 语法
```sql
ALTER TABLE table_name CHANGE old_name new_name column_type;
```

* 将 `user` 表的 `intro` 字段名改为 `about`
```sql
mysql> ALTER TABLE `user` CHANGE `intro` `about` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci COMMENT '简介';
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| about      | varchar(255)        | YES  |     | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
7 rows in set (0.07 sec)
```

#### **修改字段属性**

* 语法
```sql
ALTER TABLE table_name MODIFY column_name column_type;
```

* 将 `user` 表的 `about` 字段字符串最大长度改为200个字符
```sql
mysql> ALTER TABLE `user` MODIFY `about` varchar(200) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci COMMENT '简介';
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| about      | varchar(200)        | YES  |     | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
7 rows in set (0.09 sec)
```

### 4. 删除字段

* 语法
```
ALTER TABLE table_name DROP COLUMN column_name;
```

* 从 `user` 表中删除 `about` 字段
```sql
mysql> ALTER TABLE `user` DROP COLUMN `about`;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
6 rows in set (0.08 sec)
```

### 5. 添加索引

* 语法
```sql
ALTER TABLE `user` ADD [ KEY | UNIQUE KEY | PRIMARY KEY] idx_name (column_name);
```

* 为 `user` 表中的 `created_at` 字段添加普通索引，索引名为 `idx_created_at`
```sql
mysql> ALTER TABLE `user` ADD KEY `idx_created_at` (`created_at`);
Query OK, 0 rows affected (0.02 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  | UNI | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   | MUL | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
6 rows in set (0.07 sec)
```

### 6. 删除索引

* 语法
```sql
ALTER TABLE `user` DROP KEY idx_name;
```

* 删除 `user` 表中的 `created_at` 和 `email` 两个字段的索引，索引名为 `idx_created_at` 和 `unq_email`
```sql
mysql> ALTER TABLE `user` DROP KEY `idx_created_at`;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> ALTER TABLE `user` DROP KEY `unq_email`;
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> DESC `user`;
+------------+---------------------+------+-----+---------+----------------+
| Field      | Type                | Null | Key | Default | Extra          |
+------------+---------------------+------+-----+---------+----------------+
| id         | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| username   | varchar(190)        | NO   | MUL | NULL    |                |
| email      | varchar(255)        | YES  |     | NULL    |                |
| password   | varchar(255)        | YES  |     | NULL    |                |
| status     | tinyint(3) unsigned | NO   |     | 0       |                |
| created_at | int(11) unsigned    | NO   |     | NULL    |                |
+------------+---------------------+------+-----+---------+----------------+
6 rows in set (0.06 sec)

```


## 四、删除数据表

可以使用 `DROP TABLE` 命令删除一个或者多个数据表。

> 在使用  `DROP TABLE` 删除数据表时，要删除的数据表必须存在，否则会报错。

### 1. 删除一个数据表

```sql
DROP TABLE tablename
```

### 2. 批量删除数据表

```sql
DROP TABLE tablename1,tablename2,tablename3
```

* 删除 `user` 表
```sql
mysql> DROP TABLE `user`;
Query OK, 0 rows affected (0.01 sec)

mysql> DESC `user`;
1146 - Table 'test.user' doesn't exist

mysql> SHOW TABLES;
Empty set
```