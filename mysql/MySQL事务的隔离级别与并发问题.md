
MySQL事务的隔离级别与并发问题
---

> MySQL版本：8.0.27

---

### 一、事务并发执行面临的问题

```
CREATE TABLE `user`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` varchar(10) NOT NULL COMMENT '姓名',
  `age` tinyint(3) UNSIGNED NOT NULL COMMENT '年龄',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```
```
INSERT INTO `user` VALUES (10, '小明', 16);
INSERT INTO `user` VALUES (20, '小红', 15);
INSERT INTO `user` VALUES (30, '小丽', 18);
INSERT INTO `user` VALUES (40, '小梅', 21);
INSERT INTO `user` VALUES (50, '小亮', 20);
```
```
SELECT * FROM user;
+----+--------+-----+
| id | name   | age |
+----+--------+-----+
| 10 | 小明   |  16 |
| 20 | 小红   |  15 |
| 30 | 小丽   |  18 |
| 40 | 小梅   |  21 |
| 50 | 小亮   |  20 |
+----+--------+-----+
```
将事务隔离级别设为读未提交，方便演示
```
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```

#### 1. 脏读（Dirty Read）

如果事务A读到了未提交的事务B修改过的数据，就意味着发生了脏读现象。

事务A|事务B
-|-
begin;| begin;
SELECT * FROM user WHERE id=10;| -
-|UPDATE user SET age=11 WHERE id=10;
SELECT * FROM user WHERE id=10; <br> 读取到了事务B未提交的修改（age=11），出现脏读|-
-| rollback;
SELECT * FROM user WHERE id=10; <br> 读取到的数据又变回了（age=10）|-
commit;| -

#### 2. 不可重复读（Non-Repeatable Read）

如果事务B修改了未提交的事务A读取到的数据，就意味着发生了不可重复读现象。

事务A|事务B
-|-
begin;| begin;
SELECT * FROM user WHERE id=10;| -
-|UPDATE user SET age=12 WHERE id=10;
-| commit;
SELECT * FROM user WHERE id=10; <br> 读取到了事务B已提交的修改（age=12），出现不可重复读|-
commit;|

#### 3. 幻读（Phantom Read）

事务A先根据某个范围条件查询出了一些记录，而事务B写入了一些符合该条件的新记录，当事务A再次以相同的条件查询时，查询到了新的记录，就意味着发生了幻读现象。

事务A|事务B
-|-
begin;| begin;
SELECT * FROM user WHERE id>30;| -
-|INSERT INTO user VALUES(60, '小静', 10);
-| commit;
SELECT * FROM user WHERE id>30; <br> 读取到事务B插入的记录，出现幻读|-
commit;|

---

### 二、SQL标准中的四种隔离级别

#### 1. READ UNCOMMITTED（未提交读）

在 `READ UNCOMMITTED` 级别，事务中的修改，即使没有提交，对其他事务也都是可见的。
也就是说该隔离级别会出现**脏读**、**不可重复读**和**幻读**问题。

#### 2. READ COMMITTED（已提交读）

`READ COMMITTED` 解决了**脏读**问题，它满足事务隔离性的简单定义：**一个事务从开始直到提交之前，所做的任何修改对其他事务都是不可见的。**

#### 3. REPEATABLE READ（可重复读）

`REPEATABLE READ` 解决了**脏读**和**不可重复读**的问题。该级别保证了在同一个事务中多次读取同样记录的结果是一致的。但是理论上，可重复读隔离级别还是无法解决幻读（`Phantom Read`）问题。
`InnoDB` 存储引擎通过 **MVCC**（多版本并发控制）和 **Next-Key** (临键锁) 很大程度上避免了幻读问题。
可重复读是MySQL的默认事务隔离级别。


#### 4. SERIALIZABLE（串行化）

`SERIALIZABLE` 通过强制事务串行执行，避免了**脏读**、**不可重复读**和**幻读**的问题。`SERIALIZABLE` 会在读取的每一行数据上都加锁，所以可能导致大量的超时和锁争用问题。

---

### 三、四种隔离级别对比

|隔离级别|  脏读可能性 | 不可重复读可能性  | 幻读可能性  |加锁读 |
|-|-|-|-|-|
|READ UNCOMMITTED|√|√|√|×|
|READ COMMITTED|×|√|√|×|
|REPEATABLE READ|×|×|√|×|
|SERIALIZABLE|×|×|×|√|

---

### 四、四种隔离级别事务并行示例

##### 查看事务的隔离级别

```
SHOW VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
```

##### 设置事务的隔离级别

```
SET [GLOBAL|SESSION] TRANSACTION ISOLATION LEVEL <level>;
```
level 有4个可选值：

* READ UNCOMMITTED
* READ COMMITTED
* REPEATABLE READ 
* SERIALIZABLE

修改全局隔离级别需要退出会话重新连接MySQL生效。

##### 初始数据

```
CREATE TABLE `user`  (
  `id` int(10) UNSIGNED NOT NULL AUTO_INCREMENT,
  `name` varchar(10) NOT NULL COMMENT '姓名',
  `age` tinyint(3) UNSIGNED NOT NULL COMMENT '年龄',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB CHARACTER SET=utf8mb4;
```
```
INSERT INTO `user` VALUES (10, '小明', 16);
INSERT INTO `user` VALUES (20, '小红', 15);
INSERT INTO `user` VALUES (30, '小丽', 18);
INSERT INTO `user` VALUES (40, '小梅', 21);
INSERT INTO `user` VALUES (50, '小亮', 20);
```

#### 1. READ UNCOMMITTED（未提交读）

```
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SHOW VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | READ-UNCOMMITTED |
+-----------------------+-----------------+
```

* 脏读

事务A|事务B
-|-
begin;| begin;
SELECT * FROM user WHERE id=10;| -
-|UPDATE user SET age=11 WHERE id=10;
SELECT * FROM user WHERE id=10; <br> 读取到了事务B未提交的修改（age=11），出现脏读|-
-| rollback;
SELECT * FROM user WHERE id=10; <br> 读取到的数据又变回了（age=16）|-
commit;| -

#### 2. READ COMMITTED（已提交读）

```
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SHOW VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | READ-COMMITTED |
+-----------------------+-----------------+
```

* 不可重复读

事务A|事务B
-|-
begin;| begin;
SELECT * FROM user WHERE id=10;| -
-|UPDATE user SET age=12 WHERE id=10;
SELECT * FROM user WHERE id=10; <br> 读取不到事务B未提交的修改（age=16），没有出现脏读|-
-| commit;
SELECT * FROM user WHERE id=10; <br> 读取到了事务B已提交的修改（age=12），出现不可重复读|-
commit;|

#### 3. REPEATABLE READ（可重复读）

```
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SHOW VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | REPEATABLE-READ |
+-----------------------+-----------------+
```

* 可重复读

事务A|事务B
-|-
begin;| begin;
SELECT * FROM user WHERE id=10;| -
-|UPDATE user SET age=13 WHERE id=10;
SELECT * FROM user WHERE id=10; <br> 读取不到事务B未提交的修改（age=12），没有出现脏读|-
-| commit;
SELECT * FROM user WHERE id=10; <br> 读取不到事务B已提交的修改（age=12），没有出现不可重复读|-
commit;|

* 幻读

由于 MySQL 的 `InnoDB` 存储引擎通过 **MVCC**（多版本并发控制）和 **Next-Key** (临键锁) 很大程度上避免了幻读问题，所以无法演示大部分幻读现象，但是`InnoDB` 存储引擎并不能完全禁止幻读。

事务A|事务B
-|-
begin;| begin;
SELECT * FROM user WHERE id>30;| -
-|INSERT INTO user VALUES(60, '小静', 10);
-| commit;
SELECT * FROM user WHERE id>30; <br> 读取不到事务B插入的记录，没有出现幻读|-
UPDATE user SET age=11 WHERE id=60;|-
SELECT * FROM user WHERE id>30; <br> 读取到了事务B插入的记录，出现了幻读|-
commit;|


#### 4. SERIALIZABLE（串行化）

```
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SHOW VARIABLES LIKE 'transaction_isolation';
+-----------------------+-----------------+
| Variable_name         | Value           |
+-----------------------+-----------------+
| transaction_isolation | SERIALIZABLE |
+-----------------------+-----------------+
```

`SERIALIZABLE` 通过强制事务串行执行，避免了**脏读**、**不可重复读**和**幻读**的问题。

* 当事务A读取一条记录时，这条记录会被加上读锁（共享锁），其他事务可以查询这一条记录，但是无法修改

事务A|事务B
-|-
begin;| 
SELECT * FROM user WHERE id=10;| -
-|SELECT * FROM user WHERE id=10;<br>执行成功
-|UPDATE user SET age=14 WHERE id=10;<br>阻塞

* 当事务A修改一条记录时，这条记录会被加上写锁（排它锁），其他事务都无法查询和修改这一条记录

|事务A|事务B
|---|---
|begin; | -
|UPDATE user SET age=14 WHERE id=10; | - 
| - | SELECT * FROM user WHERE id=10;<br>阻塞
|commit;|-
|-|执行完成

* 当事务A读取范围记录时，该范围都会被加上读锁（共享锁），其他事务无法在该范围内添加、修改记录，也无法将范围外的记录修改为符合范围条件的记录。

|事务A|事务B|
|-|-|
|begin;|-|
|SELECT * FROM user WHERE id>30;| -|
|-|INSERT INTO user VALUES(70, '小花', 10);<br>阻塞|
|commit; | -|
|-|执行完成|

事务A|事务B
|---|---
|begin;| 
|SELECT * FROM user WHERE id>30;| -
|-|UPDATE user SET id=31 WHERE id=10;<br>阻塞
commit; | -
|-|执行完成
