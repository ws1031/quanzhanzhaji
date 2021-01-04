MySQL Explain命令详解：type列详解及案例分析
---

**Explain** 命令中的 **type** 列，显示MySQL查询所使用的 **关联类型（Join Types）** 或者 **访问类型**，它表明 **MySQL决定如何查找表中符合条件的行**。
常见访问类型性能由最差到最优依次为：**ALL < index < range < index_subquery < unique_subquery < index_merge < ref_or_null < fulltext < ref < eq_ref < const < system**。

### 0、测试环境简述

> 本文 MySQL 实例版本为 5.7，表存储引擎为 InnoDB

数据库 **t** 中有两张表 **user**、**user_captcha**，每张表中有2W+条数据，下面是两张表的建表语句（表结构只为满足实验要求，没有实际业务逻辑参考价值）：

#### user 表

* **id** 字段是主键
* **email** 字段建立了唯一索引
* **phone** 与 **country_code** 字段组成联合唯一索引
* **birth_year** 与 **gender** 字段组成联合普通索引
* **nickname** 字段前10个字符建立了普通索引

```
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `nickname` varchar(255) DEFAULT NULL,
  `country_code` smallint(6) unsigned NOT NULL DEFAULT '0',
  `phone` varchar(12) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL DEFAULT '',
  `email` varchar(255) CHARACTER SET latin1 COLLATE latin1_bin DEFAULT NULL,
  `gender` tinyint(4) DEFAULT NULL,
  `birth_year` smallint(11) unsigned DEFAULT NULL,
  `created_at` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unq_phone_country_code` (`phone`,`country_code`) USING BTREE,
  UNIQUE KEY `unq_email` (`email`),
  KEY `idx_birth_year_gender` (`birth_year`,`gender`) USING BTREE,
  KEY `idx_nickname` (`nickname`(10))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

#### user_captcha 表

* **id** 字段是主键
* **user_id** 字段建立了唯一索引，可以为空
* **receiver** 字段建立了唯一索引

```
CREATE TABLE `user_captcha` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) unsigned DEFAULT NULL,
  `code` char(6) COLLATE utf8_unicode_ci NOT NULL COMMENT '验证码',
  `retry_times` int(11) NOT NULL COMMENT '重试次数',
  `last_request_at` int(11) unsigned DEFAULT NULL COMMENT '最后请求时间',
  `receiver` varchar(255) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL COMMENT '接收者（手机号或邮箱）',
  `created_at` int(11) NOT NULL,
  `expired_at` int(11) NOT NULL COMMENT '过期时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `unq_receiver` (`receiver`) USING BTREE,
  UNIQUE KEY `unique_user` (`user_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```

### 1. ALL

**全表扫描**，通常意味着MySQL必须从头到尾扫描整张表，去查找匹配的行的行，性能极差。
**但是，如果在查询里使用了 `LIMIT n`，虽然 type 依然是 **ALL**，但是MySQL只需要扫描到符合条件的前 n 行数据，就会停止继续扫描**。

* 查询昵称中带 **雪** 字的用户数据，因为使用了前缀模糊匹配，不能命中索引，会导致全表扫描
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `nickname` LIKE '%雪%' LIMIT 1 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 22748
     filtered: 11.11
        Extra: Using where
```

* 查询根据用户id可以被10整除的用户数据。因为在 **=** 前的索引列上进行了表达式运算，不能命中索引，会全表扫描。

```
mysql> EXPLAIN SELECT * FROM `user` WHERE id%10=0 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 22293
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.01 sec)
```

* 查询手机号是 18888888888 的用户数据，由于数据表中 **phone** 字段是字符串类型，而查询时使用了数字类型，会触发隐式类型转换，不会命中索引，因此会全表扫描。
```
mysql> EXPLAIN SELECT * FROM `user` WHERE phone=18888888888 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: unq_phone_country_code
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 22293
     filtered: 10.00
        Extra: Using where
```


### 2. index

**index** 跟 **ALL** 一样，也会进行全表扫描，只是MySQL会**按索引次序进行全表扫描**，而不是直接扫描行数据。它的主要优点是避免了排序；最大的缺点是要承担按索引次序读取整个表的开销。若是按随机次序访问行，开销将会非常大。

* 根据出生年分组去重，查询用户数据。
```
mysql> EXPLAIN SELECT * FROM `user` GROUP BY `birth_year` \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index
possible_keys: idx_birth_year_gender
          key: idx_birth_year_gender
      key_len: 5
          ref: NULL
         rows: 22748
     filtered: 100.00
        Extra: NULL
```

**如果在 Extra 列中看到 `Using index`，说明MySQL正在使用覆盖索引，索引的数据中包含了查询所需的所有字段，因此只需要扫描索引树就能够完成查询任务。它比按索引次序全表扫描的开销要少很多，因为索引树的大小通常要远小于全表数据。**

* 根据出生年分组，查询不同年份出生的用户个数，这里用到了覆盖索引。
```
mysql> EXPLAIN SELECT `birth_year`,COUNT(*) FROM `user` GROUP BY `birth_year`\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index
possible_keys: idx_birth_year_gender
          key: idx_birth_year_gender
      key_len: 5
          ref: NULL
         rows: 22748
     filtered: 100.00
        Extra: Using index
```

* 查询用户的id、性别、出生年数据，由于 `idx_birth_year_gender` 索引中包含 `birth_year` 和 `gender`字段，而 InnoDB的所有索引都包含id字段，不需要回表查询其他数据，因此也能用到覆盖索引。
```
mysql> EXPLAIN SELECT `id`,`birth_year`,`gender` FROM `user` LIMIT 10 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index
possible_keys: NULL
          key: idx_birth_year_gender
      key_len: 5
          ref: NULL
         rows: 22748
     filtered: 100.00
        Extra: Using index
```

* 查询表数据总条数，查询数据条数时，InnoDB存储引擎会自动选择最短的索引，通过遍历该索引，就可以计算出数据总条数，不需要回表查询其他数据，因此也能用到覆盖索引。

```
mysql> EXPLAIN SELECT COUNT(*) FROM user \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index
possible_keys: NULL
          key: idx_birth_year_gender
      key_len: 5
          ref: NULL
         rows: 22748
     filtered: 100.00
        Extra: Using index
```


### 3. range

**范围扫描**，就是一个有范围限制的索引扫描，它开始于索引里的某一点，返回匹配这个范围值的行。**range** 比全索引扫描更高效，因为它用不着遍历全部索引。

**范围扫描**分为以下两种情况：
1. 范围条件查询：在 `WHERE` 子句里带有 `BETWEEN`、`>`、`<`、`>=`、`<=` 的查询。
2. 多个等值条件查询：使用 `IN()` 和 `OR` ，以及使用 `like` 进行前缀匹配模糊查询。


* 查询 `id >= 1000` 且 `id < 2000` 的用户数据。

```
mysql> EXPLAIN SELECT * FROM `user` WHERE `id`>=1000 AND `id`<2000 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 8
     filtered: 100.00
        Extra: Using where
```


* 查询 **90后** 的用户数据。

```
mysql> EXPLAIN SELECT * FROM `user` WHERE `birth_year` BETWEEN 1990 AND 1999 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: idx_birth_year_gender
          key: idx_birth_year_gender
      key_len: 3
          ref: NULL
         rows: 150
     filtered: 100.00
        Extra: Using index condition
```

* 查询昵称以 **雪** 字开头的用户数据。
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `nickname` LIKE '雪%' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: idx_nickname
          key: idx_nickname
      key_len: 43
          ref: NULL
         rows: 30
     filtered: 100.00
        Extra: Using where
```

* 分别使用 `IN()` 和 `OR` 两种方式查询出生年份在 **1990,2000,2010** 的用户数据。

```
mysql> EXPLAIN SELECT * FROM `user` WHERE `birth_year` IN (1990,2000,2010) \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: idx_birth_year_gender
          key: idx_birth_year_gender
      key_len: 3
          ref: NULL
         rows: 41
     filtered: 100.00
        Extra: Using index condition

mysql> EXPLAIN SELECT * FROM `user` WHERE `birth_year`=1990 OR `birth_year`=2000 OR `birth_year`=2010 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: idx_birth_year_gender
          key: idx_birth_year_gender
      key_len: 3
          ref: NULL
         rows: 41
     filtered: 100.00
        Extra: Using index condition
```

### 4. index_subquery

**index_subquery** 替换了以下形式的子查询中的 **eq_ref** 访问类型，其中 **key_column** 是非唯一索引。

```
value IN (SELECT key_column FROM single_table WHERE some_expr)
```
**index_subquery** 只是一个索引查找函数，它可以完全替换子查询，提高查询效率。

大多数情况下，使用**SELECT**子查询时，MySQL查询优化器会自动将**子查询**优化为**联表查询**，因此 **type** 不会显示为 **index_subquery**。

* 在MySQL查询优化器判定可以对 **SELECT** 子查询进行优化的情况下，使用**子查询**与**联表查询**的执行计划是相同的。

```
mysql> EXPLAIN SELECT code FROM user_captcha LEFT JOIN user ON user.phone=user_captcha.receiver WHERE  phone like '1888%' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: unq_phone_country_code
          key: unq_phone_country_code
      key_len: 14
          ref: NULL
         rows: 44
     filtered: 100.00
        Extra: Using where; Using index
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: user_captcha
   partitions: NULL
         type: eq_ref
possible_keys: unq_receiver
          key: unq_receiver
      key_len: 257
          ref: t.user.phone
         rows: 1
     filtered: 100.00
        Extra: Using index condition

mysql> EXPLAIN SELECT code FROM user_captcha WHERE receiver IN (SELECT phone FROM `user` WHERE phone like '1888%') \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: range
possible_keys: unq_phone_country_code
          key: unq_phone_country_code
      key_len: 14
          ref: NULL
         rows: 44
     filtered: 100.00
        Extra: Using where; Using index; LooseScan
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: user_captcha
   partitions: NULL
         type: eq_ref
possible_keys: unq_receiver
          key: unq_receiver
      key_len: 257
          ref: t.user.phone
         rows: 1
     filtered: 100.00
        Extra: Using index condition
```

* 我们可以通过在 **UPDATE** 语句的执行计划中看到 **index_subquery**。

```
mysql> EXPLAIN UPDATE user_captcha SET retry_times=1 WHERE receiver IN (SELECT phone FROM `user` WHERE phone like '1888%') \G
*************************** 1. row ***************************
           id: 1
  select_type: UPDATE
        table: user_captcha
   partitions: NULL
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 22433
     filtered: 100.00
        Extra: Using where
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: user
   partitions: NULL
         type: index_subquery
possible_keys: unq_phone_country_code
          key: unq_phone_country_code
      key_len: 14
          ref: func
         rows: 1
     filtered: 100.00
        Extra: Using where; Using index
```


### 5. unique_subquery

**unique_subquery** 跟 **index_subquery** 类似，它替换了以下形式的子查询中的 **eq_ref** 访问类型，其中 **primary_key** 可以是主键索引或唯一索引。

```
value IN (SELECT primary_key FROM single_table WHERE some_expr)
```
**unique_subquery** 只是一个索引查找函数，它可以完全替换子查询，提高查询效率。

* 由于MySQL查询优化器会对 **SELECT** 子查询进行优化，我们可以在 **UPDATE** 语句的执行计划中看到 **unique_subquery**。

```
mysql> EXPLAIN UPDATE user_captcha SET retry_times=1 WHERE user_id IN (SELECT id FROM `user` WHERE phone like '%1888%') \G
*************************** 1. row ***************************
           id: 1
  select_type: UPDATE
        table: user_captcha
   partitions: NULL
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 22433
     filtered: 100.00
        Extra: Using where
*************************** 2. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: user
   partitions: NULL
         type: unique_subquery
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: func
         rows: 1
     filtered: 11.11
        Extra: Using where
```

### 6. index_merge

表示出现了**索引合并**优化，通常是将多个索引字段的范围扫描合并为一个。包括单表中多个索引的交集，并集以及交集之间的并集，但不包括跨多张表和全文索引。
这种优化并非必然发生的，当查询优化器判断优化后查询效率更优时才会进行优化。详情可查看[官方文档](https://dev.mysql.com/doc/refman/5.7/en/index-merge-optimization.html)

* 查询出生年在 **1990,2000,2010** 年，或 **id<1000** 的用户数据
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `birth_year` IN (1990,2000,2010) OR `id`<1000 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index_merge
possible_keys: PRIMARY,idx_birth_year_gender
          key: idx_birth_year_gender,PRIMARY
      key_len: 3,4
          ref: NULL
         rows: 46
     filtered: 100.00
        Extra: Using sort_union(idx_birth_year_gender,PRIMARY); Using where
```

* 查询手机号以 **183** 开头或 出生年 **大于1990** 年的用户数据
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `phone` like '183%' OR `birth_year`>1990 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index_merge
possible_keys: unq_phone_country_code,idx_birth_year_gender
          key: unq_phone_country_code,idx_birth_year_gender
      key_len: 14,3
          ref: NULL
         rows: 1105
     filtered: 100.00
        Extra: Using sort_union(unq_phone_country_code,idx_birth_year_gender); Using where
```

* 查询出生年在 **1990** 年或 **id=1000** 的用户数据

```
mysql> EXPLAIN SELECT * FROM `user` WHERE `birth_year`=1990 OR `id`=1000 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: index_merge
possible_keys: PRIMARY,idx_birth_year_gender
          key: idx_birth_year_gender,PRIMARY
      key_len: 3,4
          ref: NULL
         rows: 11
     filtered: 100.00
        Extra: Using sort_union(idx_birth_year_gender,PRIMARY); Using where
1 row in set, 1 warning (0.01 sec)
```

### 7. ref_or_null

**ref_or_null** 于 **ref** 类似，但是MySQL必须对包含 **NULL** 值的行就行额外搜索。

* 查找昵称是 空字符串 **''** 或 **NULL** 的用户数据

```
mysql> EXPLAIN SELECT * FROM `user` WHERE `nickname`='' OR `nickname` IS NULL \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref_or_null
possible_keys: idx_nickname
          key: idx_nickname
      key_len: 43
          ref: const
         rows: 2
     filtered: 100.00
        Extra: Using where
```

### 8. fulltext

命中全文索引时 **type** 为 **fulltext**。

### 9. ref

**索引访问**（有时也叫做**索引查找**），它返回所有匹配某个单个值的行。然而，它可能会找到多个符合条件的行，因此，它是查找和扫描的混合体。**此类索引访问只有当使用非唯一性索引或者唯一性索引的非唯一性前缀时才会发生**。把它叫做 **ref** 是因为索引要跟某个参考值相比较。这个参考值或者是一个常数，或者是来自多表查询前一个表里的结果值。

* 查找出生年在**2000**年的用户数据。
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `birth_year`=2000 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: idx_birth_year_gender
          key: idx_birth_year_gender
      key_len: 3
          ref: const
         rows: 30
     filtered: 100.00
        Extra: NULL
```

* 查找电话号码是 **18888888888** 的用户数据，`phone` 与 `country_count` 联合组成唯一索引 `unq_phone_country_code`，`phone` 是唯一索引 `unq_phone_country_code` 的非唯一性前缀。

```
mysql> EXPLAIN SELECT * FROM `user` WHERE `phone`='18888888888'\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ref
possible_keys: unq_phone_country_code
          key: unq_phone_country_code
      key_len: 14
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```


### 10. eq_ref

当进行等值联表查询时，联结字段命中主键索引或唯一的非空索引时，将使用 **eq_ref**。
(《高性能MySQL（第3版）》一书中说"使用主键或唯一索引查询时会用 **eq_ref**"，经过反复测试，并查阅MySQL[5.6](https://dev.mysql.com/doc/refman/5.6/en/explain-output.html#jointype_eq_ref)、[5.7](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#jointype_eq_ref)版本的官方文档，实际上使用主键或唯一索引进行等值条件查询时 **type** 会显示 **const**，《高性能MySQL（第3版）》这里应该是只适用于5.5之前的版本。)

* 以 `user_captcha` 表作为主表，**LEFT JOIN**  `user` 表查询用户数据，因为user表中id字段是主键，所以第二行的 `user` 表的 **type** 为 **eq_ref**

```
mysql> EXPLAIN SELECT * FROM `user_captcha` LEFT JOIN `user` ON `user`.`id`=`user_captcha`.`user_id` \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user_captcha
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 22433
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: eq_ref
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: t.user_captcha.user_id
         rows: 1
     filtered: 100.00
        Extra: Using where
2 rows in set, 1 warning (0.01 sec)
```

* 当使用 `user` 表作为主表，**LEFT JOIN**  `user_captcha` 表时，因为 `user_captcha` 表中 `user_id` 字段与 `device_id` 组成联合唯一索引，`user_id` 并非独立的唯一索引，所以第二行的 `user_captcha` 表的 **type** 为 **ref**，而并非 **eq_ref**
```
mysql> EXPLAIN SELECT * FROM `user` LEFT JOIN `user_captcha` ON `user`.`id`=`user_captcha`.`user_id` \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 22999
     filtered: 100.00
        Extra: NULL
*************************** 2. row ***************************
           id: 1
  select_type: SIMPLE
        table: user_captcha
   partitions: NULL
         type: ref
possible_keys: unique_user
          key: unique_user
      key_len: 5
          ref: t.user.id
         rows: 1
     filtered: 100.00
        Extra: Using where
```


### 11. const

MySQL 知道查询最多只能匹配到一条符合条件的记录。因为只有一行，所以优化器可以将这一行中的列中的值视为常量。**const** 表查询非常快，因为它们只读取一次数据行。
**通常使用主键或唯一索引进行等值条件查询时会用 const。**

* 使用主键查询用户数据
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `id`=120 \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

* 使用唯一索引查询用户数据
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `email`='54222806@qq.com' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: const
possible_keys: unq_email
          key: unq_email
      key_len: 258
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

* 使用联合唯一索引查询用户数据
```
mysql> EXPLAIN SELECT * FROM `user` WHERE `country_code`=86 AND `phone`='18888888888' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: user
   partitions: NULL
         type: const
possible_keys: unq_phone_country_code
          key: unq_phone_country_code
      key_len: 16
          ref: const,const
         rows: 1
     filtered: 100.00
        Extra: NULL
```



### 12. system

官方文档原文是：`The table has only one row (= system table). This is a special case of the const join type.`
该表只有一行（=系统表）。这是 **const** 关联类型的特例。

* 从系统库mysql的系统表 `proxies_priv` 里查询数据，这里的数据在Mysql服务启动时候已经加载在内存中，不需要进行磁盘IO。

```
mysql> EXPLAIN SELECT * FROM `mysql`.`proxies_priv` \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: proxies_priv
   partitions: NULL
         type: system
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: NULL
```

> 参考：
> 《高性能MySQL（第3版）》
> [MySQL 5.7官方文档：explain-join-types](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#explain-join-types)
