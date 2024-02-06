MySQL基础知识点整理 - 账号和权限管理
---

## 一、账号管理

### 1. 查看账号列表

MySQL用户账号和信息存储在名为 `mysql` 的数据库中。一般不需要直接访问 `mysql` 数据库和表，但有时需要直接访问。例如，查看数据库所有用户账号列表时。

#### **语法**
```sql
USE mysql;
SELECT DISTINCT(`user`) FROM user;
```

* 数据库 `mysql` 有一个名为 `user` 的表，它包含所有用户账号。 `user` 表有一个名为 `user` 的字段，它存储账号名。
* 进入数据库 `mysql`，查看 `user` 表中的 `user` 列，由于有些账号会分多行记录，`DISTINCT` 用于去重。

#### **实例**

```sql
mysql> USE mysql;
Database changed

mysql> SELECT DISTINCT(`user`) FROM user;
+-----------+
| user      |
+-----------+
| root      |
| mysql.sys |
+-----------+
2 rows in set (0.07 sec)
```

* **user字段** 表示账号名，表示当前数据库有 `root` 和 `mysql.sys` 两个账号。

### 2. 创建账号

可以使用 `CREATE USER` 语句创建一个新用户账号。

#### **语法**
```sql
CREATE USER account_name IDENTIFIED BY 'password';
```

* `IDENTIFIED BY` 用于设定密码，MySQL 会先将密码进行加密，在将其保存到 user 表。

> 使用 `GRANT` 或 `INSERT GRANT` 语句也可以创建用户账号，但一般来说 `CREATE USER` 是最清楚和最简单的句子。
> 使用 `CREATE USER` 创建用户账号，必须接着分配访问权限。新创建的用户账号没有访问权限。它们能登录MySQL，但不能看到数据，不能执行任何数据库操作。
> 可以使用 `GRANT` 语句创建用户账号并授权，该语句会在文章授权部分讲解。

> 用于账号都存储在数据库 `mysql` 的 `user` 表中，理论上也可以通过直接插入行到 user 表来增加用户，不过为安全起见，一般不建议这样做。MySQL用来存储用户账号信息的表（以及表模式等）极为重要，对它们的任何毁坏都可能严重地伤害到MySQL服务器。因此，最好不要直接修改数据库 `mysql` 中表的数据。

#### **实例**

* 为数据库添加账号 `zhangsan` 密码为 `123456`
```sql
mysql> CREATE USER zhangsan IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.06 sec)

mysql> SELECT DISTINCT(`user`) FROM user;
+-----------+
| user      |
+-----------+
| root      |
| zhangsan  |
| mysql.sys |
+-----------+
3 rows in set (0.07 sec)

```

* 使用 zhangsan 的账号和密码登录，成功登录 MySQL。
```sql
[vagrant~] ]$mysql -uzhangsan -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
......
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>

```

> 注意，不要直接在命令行中输入密码，因为命令行的输入历史都会被记录下来，很同意导致密码泄露。


### 3. 账号重命名

可以使用  `RENAME USER` 语句为账号重命名。

#### **语法**
```sql
RENAME USER old_name TO new_name;
```

> 仅 MySQL 5及之后的版本支持 `RENAME USER`。
> MySQL 5以前的版本，要重命名一个用户，可使用 UPDATE 直接更新 user 表（**谨慎操作**）。

#### **实例**

* 将数据库账号 `zhangsan` 重命名为 `lisi`

```sql
mysql> RENAME USER zhangsan TO lisi;
Query OK, 0 rows affected (0.34 sec)

mysql> SELECT DISTINCT(`user`) FROM user;
+-----------+
| user      |
+-----------+
| lisi      |
| root      |
| mysql.sys |
+-----------+
3 rows in set (0.08 sec)
```

### 4. 重置账号密码

可以使用  `SET PASSWORD` 语句重置账号密码。

#### **语法**
```sql
SET PASSWORD FOR account_name = Password('password');
```

> 使用 `SET PASSWORD` 重置账号密码。新密码必须通过 Password() 函数进行加密。

当不指定用户名时， `SET PASSWORD` 会重置当前登录用户的密码。

#### **语法**
```sql
SET PASSWORD = Password('password');
```

#### **实例**

* 将账号 `lisi` 的密码改为 `abcdef`
```sql
mysql> SET PASSWORD FOR lisi = Password('abcdef');
Query OK, 0 rows affected (0.03 sec)
```

* 使用 lisi 的账号和新密码登录，成功登录 MySQL。

```sql
[vagrant~] ]$mysql -ulisi -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
......
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

* 将当前登录账号的密码重置为 10086
```sql
mysql> SET PASSWORD = Password('10086');
Query OK, 0 rows affected (0.00 sec)
```

### 5. 删除账号

可以使用  `DROP USER` 语句删除账号（以及相关的权限）。

#### **语法**
```sql
DROP USER account_name;
```

> MySQL 5及之后的版本，`DROP USER` 删除用户账号时会自动删除所有相关的账号权限。
> 在MySQL 5以前， `DROP USER` 只能用来删除用户账号，不能删除相关的权限。因此，如果使用旧版本的MySQL，需要先用 `REVOKE` 删除与账号相关的权限，然后再用 `DROP USER` 删除账号。

#### **实例**

* 删除账号 `lisi`
```sql
mysql> DROP USER lisi;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT DISTINCT(`user`) FROM user;
+-----------+
| user      |
+-----------+
| root      |
| mysql.sys |
+-----------+
2 rows in set (0.07 sec)
```


## 二、权限管理

### 0. 访问控制

MySQL服务器的安全基础是：用户应该对他们需要的数据具有适当的访问权，既不能多也不能少。

考虑以下情况：
* 多数用户只需要对表进行读和写，但少数用户甚至需要能创建和删除表；
* 某些用户需要读表，但可能不需要更新表；
* 你可能想允许用户添加数据，但不允许他们删除数据；
* 某些用户（管理员）可能需要处理用户账号的权限，但多数用户不需要；
* 你可能想让用户通过存储过程访问数据，但不允许他们直接访问数据；
* 你可能想根据用户登录的地点限制对某些功能的访问。

这些都只是例子，但有助于说明一个重要的事实，即你需要给用户提供他们所需的访问权，且仅提供他们所需的访问权。这就是所谓的访问控制，管理访问控制需要创建和管理用户账号。

#### **严肃对待 root 账号的使用**

MySQL 会默认创建一个名为 `root` 的用户账号，它对整个 MySQL 服务器具有完全的控制。不过在日常的 MySQL 操作中（特别是生产环境），决不能使用 root 账号登录。应该创建一系列的账号，有的用于管理，有的供用户使用，有的供开发人员使用，等等。应该严肃对待 root 账号的使用，仅在绝对需要时使用它。


#### **访问控制的目的**

* 防止用户的恶意企图。
* 防止用户无意识错误造成数据错乱。这是更为常见的情况。如错打 SQL 语句，在不合适的数据库中操作或其他一些用户错误。

通过保证用户不能执行他们不应该执行的语句，访问控制有助于避免这些情况的发生。


### 1. 查看账号权限

#### **语法**
```sql
SHOW GRANTS[ FOR account_name][@host];
```

* 当不使用 `FOR` 指定用户时，默认是查看自己的账号权限。
* 当使用 `SHOW GRANTS FOR account_name@host` 时，可查看指定账号在指定主机下的权限。可以在数据库 `mysql` 中使用 `SELECT user,host FROM user;` 查看账号的主机列表。

```sql
mysql> USE mysql;
Database changed

mysql> SELECT user,host FROM user;
+-----------+-----------+
| user      | host      |
+-----------+-----------+
| root      | %         |
| mysql.sys | localhost |
| root      | localhost |
+-----------+-----------+
3 rows in set (0.06 sec)
```
**host字段：** 表示账号可以在哪些主机或IP地址登录。`%` 代表任何IP地址都可以登录（生产环境中这样做是非常危险的），`localhost` 表示只允许本机登录。

> 将 `host` 设为 `%`，就代表任何IP地址都可以访问该数据库，在生产环境中这样做是非常危险的。
> 也可以使用其他手段来提高数据库安全性：比如设置防火墙、iptable；如果是云平台的数据库还可以通过安全组等方式提高安全性。 

#### **实例**

* 查看自己的账号（root）权限。

```sql
mysql> SHOW GRANTS;
+-------------------------------------------------------------+
| Grants for root@%                                           |
+-------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION |
+-------------------------------------------------------------+
1 row in set (0.00 sec)     
```

* 查看账号 root 在 localhost 下的权限。

```sql
mysql> SHOW GRANTS FOR root@localhost;
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION |
+---------------------------------------------------------------------+
1 row in set (0.00 sec)
```


> 输出结果显示账号 `root` 有一个权限 `ALL PRIVILEGES ON *.*`，表示 `root` 账号可以操作所有数据库和所有表。

* 使用 `CREATE USER` 创建一个账号 `zhangsan`，并查看  `zhangsan` 的账号权限。
```sql
mysql> CREATE USER zhangsan IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GRANTS FOR zhangsan;
+---------------------------------------------------------------------------------------------------------+
| Grants for zhangsan@%                                                                                   |
+---------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'zhangsan'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
+---------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

```

> 输出结果显示账号 `zhangsan` 有一个权限 `USAGE ON *.*` 。 **USAGE 表示根本没有权限**，所以，此结果表示 `zhangsan` 对任意数据库和任意表上对任何东西都没有操作权限。

* 登录账号 `zhangsan`，并尝试进入 `test` 数据库，被拒绝。
```sql
[vagrant~] ]$mysql -uzhangsan -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
......
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use test;
ERROR 1044 (42000): Access denied for user 'zhangsan'@'%' to database 'test'
```

> 使用 `CREATE USER` 创建的用户账号默认没有访问权限。它们能登录MySQL，但不能看到数据，不能执行任何数据库操作。

### 2. 为已存在的账号设置权限

可以使用 `GRANT` 语句为账号设置权限。至少给出以下信息：

* 账号名。
* 被授予访问权限的数据库或表。
* 要授予的权限。

#### **语法**
```sql
GRANT <权限> ON <数据库名>.<表名> TO <账户名>;
```

#### **实例**

* 给账号 `zhangsan` 赋予在 `test` 数据库内的任意表查找和添加数据的权限。
```sql
mysql> GRANT SELECT, INSERT ON test.* TO 'zhangsan';
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GRANTS FOR zhangsan;
+---------------------------------------------------------------------------------------------------------+
| Grants for zhangsan@%                                                                                   |
+---------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'zhangsan'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
| GRANT SELECT, INSERT ON `test`.* TO 'zhangsan'@'%'                                                      |
+---------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)
```

> 授权后必须FLUSH PRIVILEGES，否则无法立即生效。

* 登录账号 `zhangsan`，可以成功进入到数据库 `test`，在 `user` 表中插入一条数据，并从 `user` 表查找数据。

```sql
mysql> USE test;
Database changed

mysql> SHOW TABLES;
+----------------+
| Tables_in_test |
+----------------+
| user           |
+----------------+
1 row in set (0.00 sec)

mysql> INSERT INTO user (username, email) VALUES ('zhangsan', 'zhangsan@gmail.com');
Query OK, 1 row affected, 1 warning (0.00 sec)

mysql> SELECT * FROM user;
+----+----------+--------------------+----------+--------+------------+
| id | username | email              | password | status | created_at |
+----+----------+--------------------+----------+--------+------------+
|  1 | zhangsan | zhangsan@gmail.com | NULL     |      0 |          0 |
+----+----------+--------------------+----------+--------+------------+
1 row in set (0.00 sec)

```

* 当 `zhangsan` 想要使用 `UPDATE` 和 `DELETE` 命令修改和删除这条数据时，被提示没有权限。
```sql
mysql> UPDATE user SET email='zhangsanfeng@gmail.com' WHERE username='zhangsan';
ERROR 1142 (42000): UPDATE command denied to user 'zhangsan'@'localhost' for table 'user'

mysql> DELETE FROM user WHERE username='zhangsan';
ERROR 1142 (42000): DELETE command denied to user 'zhangsan'@'localhost' for table 'user'
```

### 3. 创建账号并设置权限

#### **语法**
```sql
GRANT <权限> ON <数据库名>.<表名> TO <账户名>@<主机名/IP> IDENTIFIED BY '<密码>'[ WITH GRANT OPTION];
```

> `WITH GRANT OPTION` 用于赋予账号使用 `GRANT` 和 `REVOKE` 命令的权限，用于给账号授权和取消授权。此权限级别极高，一般只会将此权限授予数据库管理员账号。

#### **实例**

* 创建账号 `lisi` 密码为 `abcdef`，在任何IP地址都可以登录，同时赋予 `lisi` 在 `test` 数据库内的任意表数据的增删改查权限。

```sql
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON test.* TO 'lisi'@'%' IDENTIFIED BY 'abcdef';
Query OK, 0 rows affected (0.00 sec)
```

> 将 `host` 设为 `%`，就代表任何IP地址都可以访问该数据库，在生产环境中这样做是非常危险的。
> 也可以使用其他手段来提高数据库安全性：比如设置防火墙、iptable；如果是云平台的数据库还可以通过安全组等方式提高安全性。 

* 登录账号 `lisi`，可以成功进入到数据库 `test`，并对 `user` 表数据进行增删改查。
```sql
[vagrant~] ]$mysql -ulisi -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
......
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use test;
Database changed

mysql> INSERT INTO user (username, email) VALUES ('lisi', 'lisi@gmail.com');
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> SELECT * FROM user;
+----+----------+--------------------+----------+--------+------------+
| id | username | email              | password | status | created_at |
+----+----------+--------------------+----------+--------+------------+
|  1 | zhangsan | zhangsan@gmail.com | NULL     |      0 |          0 |
|  2 | lisi     | lisi@gmail.com     | NULL     |      0 |          0 |
+----+----------+--------------------+----------+--------+------------+
2 rows in set (0.00 sec)

mysql> UPDATE user SET email='lisiguang@gmail.com' WHERE username='lisi';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> DELETE FROM user WHERE username='zhangsan';
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM user;
+----+----------+---------------------+----------+--------+------------+
| id | username | email               | password | status | created_at |
+----+----------+---------------------+----------+--------+------------+
|  2 | lisi     | lisiguang@gmail.com | NULL     |      0 |          0 |
+----+----------+---------------------+----------+--------+------------+
1 row in set (0.00 sec)
```

### 4. 撤销账号指定权限

可以使用 `REVOKE` 语句撤销账号指定权限。`REVOKE` 是 `GRANT` 的反操作。

#### **语法**
```sql
REVOKE <权限> ON <数据库名>.<表名> FROM <账户名>;
```

#### **实例**

* 删除账号 `lisi` 的  DELETE 权限。
```sql
mysql> REVOKE DELETE ON test.* FROM lisi;
Query OK, 0 rows affected (0.00 sec)

mysql> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

mysql> SHOW GRANTS FOR lisi;
+-----------------------------------------------------------------------------------------------------+
| Grants for lisi@%                                                                                   |
+-----------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'lisi'@'%' IDENTIFIED BY PASSWORD '*C2D24DCA38E9E862098B85BF0AB35CAA52803797' |
| GRANT SELECT, INSERT, UPDATE ON `test`.* TO 'lisi'@'%'                                              |
+-----------------------------------------------------------------------------------------------------+
2 rows in set (0.04 sec)
```

* 当 `lisi` 想要使用 `DELETE` 命令删除数据时，被提示没有权限。
```sql
mysql> DELETE FROM user WHERE username='lisi';
ERROR 1142 (42000): DELETE command denied to user 'lisi'@'localhost' for table 'user'
```

### 5. 权限说明

`GRANT` 和 `REVOKE` 可在几个层次上控制访问权限：

* 整个服务器，使用 `GRANT ALL` 和 `REVOKE ALL`；
* 整个数据库，使用 `ON database.*`；
* 特定的表，使用 `ON database.table`；
* 特定的列；
* 特定的存储过程。

权 限  |说 明
-|-
ALL  |除 `GRANT OPTION` 外的所有权限
ALTER  |使用 `ALTER TABLE`
ALTER ROUTINE | 使用 `ALTER PROCEDURE` 和 `DROP PROCEDURE`
CREATE | 使用 `CREATE TABLE`
CREATE ROUTINE  | 使用 `CREATE PROCEDURE`
CREATE TEMPORARY TABLES  | 使用 `CREATE TEMPORARY TABLE`
CREATE USER   |使用 `CREATE USER`、`DROP USER`、`RENAME USER` 和 `REVOKE ALL PRIVILEGES`
CREATE VIEW  | 使用 `CREATE VIEW`
DELETE  | 使用 `DELETE`
DROP  | 使用 `DROP TABLE`
EXECUTE  | 使用 `CALL` 和存储过程
FILE   |使用 `SELECT INTO OUTFILE` 和 `LOAD DATA INFILE`
GRANT OPTION   |使用 `GRANT` 和 `REVOKE`
INDEX   |使用 `CREATE INDEX` 和 `DROP INDEX`
INSERT   |使用 `INSERT`
LOCK TABLES   |使用 `LOCK TABLES`
PROCESS   |使用 `SHOW FULL PROCESSLIST`
RELOAD   |使用 `FLUSH`
REPLICATION CLIENT   |服务器位置的访问
REPLICATION SLAVE  | 由复制从属使用
SELECT   |使用 `SELECT`
SHOW DATABASES   |使用 `SHOW DATABASES`
SHOW VIEW   |使用 `SHOW CREATE VIEW`
SHUTDOWN   |使用 `mysqladmin shutdown`（用来关闭MySQL）
SUPER   |使用 `CHANGE MASTER`、`KILL`、`LOGS`、`PURGE`、`MASTER` 和 `SET GLOBAL`。还允许 `mysqladmin` 调试登录
UPDATE   |使用 `UPDATE` 
USAGE  | 无访问权限

### 6. 权限控制流程

最后附一张《MySQL性能调优与架构设计》中的权限控制流程图。

以 `SELECT id,name FROM test.t4 where status = 'deleted';` 为例。

![MySQL性能调优与架构设计-权限控制流程图](http://md.ws1031.cn/xsj/2019_6_29_MySQL性能调优与架构设计-权限控制流程图.jpg)


---


![欢迎关注公众号【全栈札记】](https://md.s1031.cn/xsj/2021_1_4_扫码_搜索联合传播样式-白色版.png)