MySQL修改表结构到底会不会锁表？
---


### 〇、关于DDL、DML和DCL

**DDL**（Data Definition Languages）语句：数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。
常用的语句关键字主要包括 create、drop、alter 等。

**DML**（Data Manipulation Language）语句：数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性。
常用的语句关键字主要包括 insert、delete、udpate 和 select 等。(增删改查）

**DCL**（Data Control Language）语句：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。
主要的语句关键字包括 grant、revoke 等。

---

### 一、DDL 实现方式

在 **MySQL5.6** 版本以前，执行 DDL 主要有两种方式：**Copy 方式** 和 **In-place 方式**

#### Copy 方式执行DDL操作

1. 创建与原表结构定义完全相同的临时表
2. 为原表加MDL（meta data lock，元数据锁）锁，禁止对表中数据进行增删改，允许查询
3. 在临时表上执行DDL语句
4. 按照主键 ID 递增的顺序，把数据一行一行地从原表里读出来再插入到临时表中
5. 升级原表上的锁，禁止对原表中数据进行读写操作
6. 将原表删除，将临时表重命名为原表名，DDL操作完成

#### In-place 方式执行DDL操作

**In-place 方式** 又称为 `Fast Index Creation` 。与 Copy 方式相比，In-place方式不复制数据，因此大大加快了执行速度。但是这种方式仅支持**对二级索引进行添加、删除操作**，而且与Copy方式一样需要全程锁表。下面以添加索引为例，简单介绍In-place方式的实现流程：

1. 创建新索引的数据字典
2. 为原表加MDL（meta data lock，元数据锁）锁，禁止对表中数据进行增删改，允许查询
3. 按照聚簇索引的顺序，查询数据，找到需要的索引列数据，排序后插入到新的索引页中
4. 等待打开当前表的所有只读事务提交
5. 创建索引结束

#### In-place 方式执行DDL操作

**MySQL5.6** 版本之后加入了 **Online DDL** 新特性，用于支持DDL执行期间DML语句的并行操作，提高数据库的吞吐量。
与 Copy 方式和 In-place 方式相比，Online 方式在执行DDL的时候可以对表中数据进行读写操作。
Online DDL可以有效改善DDL期间对数据库的影响：

* Online DDL期间，查询和DML操作在多数情况下可以正常执行，对表格的锁时间也会大大减少，尽可能的保证数据库的可扩展性；
* 允许 In-place 操作的 DDL，避免重建表格占用过多磁盘IO及CPU资源，减少对数据库的整体负荷，使得在DDL期间，能够维持数据库的高性能及高吞吐量；
* 允许 In-place 操作的 DDL，比需要COPY到临时文件的操作要更少占用buffer pool，避免以往DDL过程中性能的临时下降，因为以前需要拷贝数据到临时表，这个过程会占用到buffer pool ，导致内存中的部分频繁访问的数据会被清理出去。


Online DDL 实现实质上也可以分为2种方式：**Copy** 方式和 **In-place** 方式：

* 对于不支持Online DDL的 SQL，则采用 **Copy** 方式，比如**删除主键**，**修改列数据类型**，**变更表字符集**等。这些操作都会导致记录格式发生变化，无法通过简单的全量+增量的方式实现Online DDL。
* 对于支持Online DDL的 SQL，则采用 **In-place** 方式，MySQL 内部以“**是否修改行记录格式**”为标准，又将 **In-place** 方式分为两类：
	* 如果修改了行记录格式，则需要重建表，比如 **添加主键**，**添加、删除字段**，**修行格式ROW_FORMAT**，**OPTIMIZE优化表** 等操作，这种方式被称为 **rebuild方式**。
	* 如果没有修改行记录格式，仅修改表的元数据，则不需要重建表，比如 **添加、删除、重命名二级索引**，**设置、删除字段的默认值**，**重命名字段**，**重命名表** 等操作，这种方式被称为 **no-rebuild方式**。

![Onlive_DDL_实现方式](https://md.s1031.cn/xsj/2021_11_13_Onlive_DDL_实现方式.png)

> 更多详情可查阅官方文档：https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-operations.html

---

### 三、Online DDL 实现流程

**Online DDL** 主要包括3个阶段，Prepare阶段，Execute阶段，Commit阶段。
下面将主要介绍Online DDL执行过程中三个阶段的流程。

Prepare 阶段

* 持有 EXCLUSIVE-MDL 锁，禁止DML语句读写
* 根据DDL类型，确定执行方式(Copy,Online-rebuild,Online-no-rebuild)
* 创建新的 frm 和 ibd 临时文件（ibd临时文件仅rebuild类型需要）
* 分配 row_log 空间，用来记录 DDL Execute 阶段产生的DML操作（仅rebuild类型需要）

Execute 阶段

* 降级 EXCLUSIVE-MDL 锁，允许DML语句读写
* 扫描原表主键以及二级索引的所有数据页，生成B+树，存储到临时文件中
* 将 DDL Execute 阶段产生的DML操作记录到 row_log（仅rebuild类型需要）

Commit 阶段

* 升级到 EXCLUSIVE-MDL 锁，禁止DML语句读写
* 将 row_log 中记录的DML操作应用到临时文件，得到一个逻辑数据上与原表相同的数据文件（仅rebuild类型需要）
* 重命名 frm 和 idb 临时文件，替换原表，将原表文件删除
* 提交事务（刷事务的redo日志），变更完成

由上面的流程可知，Prepare阶段和Commit阶段都禁止读写，只有Execute允许读写，那为什么说**Online DDL 方式在执行过程中可以对表中数据进行读写操作**？
其实是因为Prepare和Commit阶段相对于Execute阶段时间特别短，因此基本可以认为是全程允许读写的。
Prepare阶段和Commit阶段都禁止读写，主要是为了保证数据一致性。

---

### 四、Online DDL 的语法与可选参数

```
ALTER TABLE …. , ALGORITHM[=]{DEFAULT|INPLACE|COPY}, LOCK[=]{DEFAULT|NONE|SHARED|EXCLUSIVE}
```

例句：
```
ALTER TABLE tablename DROP COLUMN age,ALGORITHM=INPLACE,LOCK=DEFAULT;
```

#### ALGORITHM 选项

**COPY**：使用 **Copy 方式** 执行DDL操作，DDL 执行过程中，不允许DML操作。
**INPLACE**：使用 **In-place 方式** 执行DDL操作，DDL 执行过程中，允许DML操作。
**DEFAULT**：默认选项，根据DDL的操作类型，自动选择DDL执行方式，优先选择 **In-place 方式**，不满足条件时选择 **Copy 方式**。

#### LOCK 选项

**EXCLUSIVE**：对整个表添加排他锁（X锁），不允许DML操作
**SHARED**：对整个表添加共享锁（S锁），允许查询操作，但是不允许数据变更操作
**NONE**：不对表加锁，既允许查询操作，也支持数据变更操作，即允许所有的 DML 操作，该模式下并发最好
**DEFAULT**：默认选项，根据DDL的操作类型，最小程度的加锁，尽可能支持DML操作










---

https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-operations.html
https://www.wenjiangs.com/article/mysql-5-6-DDL-lock-the-table.html
online ddl 思维导图 http://blog.itpub.net/22664653/viewspace-2056953/
MySQL Online DDL 原理和踩坑 https://juejin.cn/post/6854573213167386637
数据库系列之InnoDB中在线DDL实现机制 https://www.modb.pro/db/112016
Online DDL过程介绍 https://cloud.tencent.com/developer/article/1143432
MySQL Online DDL的改进与应用 https://blog.51cto.com/33997k7k/1925876
MySQL InnoDB Online DDL学习 https://www.debugger.wiki/article/html/1550203202837500
online-ddl https://www.codenong.com/cs106077583/
mysql 5.6 原生Online DDL解析 https://segmentfault.com/a/1190000005642792
Mysql Online DDL特性（一） https://blog.csdn.net/finalkof1983/article/details/88355314
https://time.geekbang.org/column/article/72388


技术分享 | 深入理解 MySQL MDL Lock https://segmentfault.com/a/1190000022366564


四、常见的 DDL 操作说明

类型	|操作	|是否In-place	|是否重建表	|是否允许并发DML	|是否只修改元数据	|备注
---|---|---|---|---|---|---
index	|创建或添加二级索引	|是	|否	|是	|否	|仅在完成访问表的所有事务完成后才结束<br>索引的初始状态反映了表的最新内容
-|删除索引	|是	|否	|是	|是	|
-|重命名索引	|是	|否	|是	|是	| 
-|添加FULLTEXT索引	|是*	|否*	|否	|否	|FULLTEXT如果没有用户定义的FTS_DOC_ID列<br>则添加第一个索引会重建表<br>FULLTEXT可以添加其他索引而无需重建表
-|添加SPATIAL索引	|是*	|否*	|否	|否	|SPATIAL如果没有用户定义的FTS_DOC_ID列<br>则添加第一个索引会重建表<br>SPATIAL可以添加其他索引而无需重建表
-|更改索引类型	|是	|否	|是	|是	| 
primary key	|添加主键	|是	|是	|是	|否	|对表进行重建，耗费IO
-|删除主键	|否	|是	|否	|否	|仅ALGORITHM=Copy支持删除主键而不在同一ALTER TABLE语句中添加新主键<br>对表进行重建，耗费IO<br>产生临时表时同样需要在原表路径生成表空间临时文件
-|同时删除主键并添加	|是	|是	|是	|否	|对表进行重建，耗费IO
COLUMN	|添加列	|是	|是	|是*	|否	|添加自增列时不允许并发DML，且mysql会自动设置LOCK=SHARED，用来替换默认的LOCK=DEFAULT<br>对表进行重建，耗费IO
-|删除列	|是	|是	|是	|否	|对表进行重建，耗费IO
-|重命名列	|是	|否	|是	|是	| 
-|重新排序列	|是	|是	|是	|否	|对表进行重建，耗费IO
-|设置/删除列默认值	|是	|否	|是	|是	|仅修改表元数据。默认列值存储在 表的.frm文件中，而不是InnoDB 数据字典中。
-|更改列数据类型	|否	|是	|否	|否	|仅ALGORITHM=Copy支持更改列数据类型<br>对表进行重建，耗费IO
-|扩展VARCHAR列大小	|是	|否	|是	|是	|将VARCHAR列大小从0增加到255个字节，可以使用In-place方式。<br>如果改为大于255的值，则只能使用ALGORITHM=Copy
-|更改自动增量值	|是	|否	|是	|否*	|修改存储在内存中的值，而不是数据文件。
-|添加列NULL/NOT NULL	|是	|是	|是	|否	|对表进行重建，耗费IO
-|修改ENUM或SET列的定义	|是	|否	|是	|是	| 
GENERATED COLUMN|添加STORED列	|否	|是	|否	|否	|ALTER TABLE t1 ADD COLUMN (c2 INT GENERATED ALWAYS AS (c1 + 1) STORED), ALGORITHM=Copy;
-|修改STORED列顺序	|否	|是	|否	|否	|ALTER TABLE t1 MODIFY COLUMN c2 INT GENERATED ALWAYS AS (c1 + 1) STORED FIRST, ALGORITHM=Copy;
-|删除STORED列	|是	|是	|是	|否	|ALTER TABLE t1 DROP COLUMN c2, ALGORITHM=In-place, LOCK=NONE;
-|添加VIRTUAL列	|是	|否	|是	|是	|ALTER TABLE t1 ADD COLUMN (c2 INT GENERATED ALWAYS AS (c1 + 1) VIRTUAL), ALGORITHM=In-place, LOCK=NONE;
-|修改VIRTUAL列顺序	|否	|是	|否	|否	|ALTER TABLE t1 MODIFY COLUMN c2 INT GENERATED ALWAYS AS (c1 + 1) VIRTUAL FIRST, ALGORITHM=Copy;
-|删除VIRTUAL列	|是	|否	|是	|是	|ALTER TABLE t1 DROP COLUMN c2, ALGORITHM=In-place, LOCK=NONE;
foreign key	|添加外键约束	|是	|否	|是	|是	| 
-|删除外键约束	|是	|否	|是	|是	| 
table	|修改ROW_FORMAT	|是	|是	|是	|否	|对表进行重建，耗费IO
-|修改KEY_BLOCK_SIZE	|是	|是	|是	|否	|对表进行重建，耗费IO
-|设置持久表统计信息	|是	|否	|是	|是	|仅修改表元数据。
-|指定字符集	|是	|是	|否	|否	|如果新字符编码不同，则重建表。
-|转换字符集	|否	|是	|否	|否	|如果新字符编码不同，则重建表。
-|OPTIMIZE优化表	|是*	|是	|是	|否	|OPTIMIZE TABLE tbl_name;<br>具有FULLTEXT索引的表不支持In-place 
-|执行空重建表	|是*	|是	|是	|否	|ALTER TABLE tbl_name ENGINE=InnoDB;<br>具有FULLTEXT索引的表不支持In-place 
-|重命名表	|是	|否	|是	|是	|重命名与表对应的文件而不进行复制
tablespace	|启用或禁用单表文件表空间加密	|否	|是	|否	|否	|ALTER TABLE tbl_name ENCRYPTION='Y', ALGORITHM=Copy;
PARTITION	|PARTITION BY	|否	|不涉及	|否	|不涉及	|ALGORITHM=Copy， LOCK={DEFAULT|SHARED|EXCLUSIVE}
-|ADD PARTITION	|否	|不涉及	|否	|不涉及	|只允许ALGORITHM=DEFAULT,LOCK=DEFAULT<br>在保持共享锁的同时复制数据
-|DROP PARTITION	|否	|不涉及	|否	|不涉及	|只允许ALGORITHM=DEFAULT,LOCK=DEFAULT
-|DISCARD PARTITION	|否	|不涉及	|否	|不涉及	|只允许ALGORITHM=DEFAULT,LOCK=DEFAULT
-|IMPORT PARTITION	|否	|不涉及	|否	|不涉及	|只允许ALGORITHM=DEFAULT,LOCK=DEFAULT
-|TRUNCATE PARTITION 	|是	|不涉及	|是	|不涉及	|不复制现有数据。只删除行; 不会改变表本身或其任何分区的定义
-|COALESCE PARTITION	|否	|不涉及	|否	|不涉及	|只允许ALGORITHM=DEFAULT， LOCK=DEFAULT<br>在保持共享锁的同时复制数据
-|REORGANIZE PARTITION	|否	|不涉及	|否	|不涉及	|只允许ALGORITHM=DEFAULT， LOCK=DEFAULT<br>保持共享元数据锁的同时从受影响的分区复制数据
-|EXCHANGE PARTITION	|是	|不涉及	|是	|不涉及	| 
-|ANALYZE PARTITION	|是	|不涉及	|是	|不涉及	| 
-|CHECK PARTITION	|是	|不涉及	|是	|不涉及	| 
-|OPTIMIZE PARTITION	|否	|不涉及	|否	|不涉及	|ALGORITHM和LOCK参数被忽略。重建整个表
-|REBUILD PARTITION	|否	|不涉及	|否	|不涉及	|只允许ALGORITHM=DEFAULT， LOCK=DEFAULT<br>保持共享元数据锁的同时从受影响的分区复制数据。
-|REPAIR PARTITION	|是	|不涉及	|是	|不涉及	| 
-|REMOVE PARTITION	|否	|不涉及	|否	|不涉及	|ALGORITHM=Copy， LOCK={DEFAULT|SHARED|EXCLUSIVE}


五、若干问题

2.Online 与数据一致性如何兼得
实际上，Online DDL并非整个过程都是Online，在Prepare阶段和Commit阶段都会持有MDL-Exclusive锁，禁止读写；而在整个DDL执行阶段，允许读写。由于Prepare和Commit阶段相对于DDL执行阶段时间特别短，因此基本可以认为是全程Online的。Prepare阶段和Commit阶段的禁止读写，主要是为了保证数据一致性。Prepare阶段需要生成row_log对象和修改内存的字典；Commit阶段，禁止读写后，重做最后一部分增量，然后提交，保证数据一致。

3.如何实现 Server 层和 InnoDB 层一致性
在Prepare阶段，Server层会生成一个临时的frm文件，里面包含了新表的格式；InnoDB层生成了临时的ibd文件(rebuild方式)；在DDL执行阶段，将数据从原表复制到临时ibd文件，并且将row_log增量应用到临时ibd文件；在Commit阶段，InnoDB层修改表的数据字典，然后提交；最后InnoDB层和mysql层面分别重命名frm和idb文件。

4.对 InnoDB 表做 DDL 过程中异常了，为啥再次做 DDL 报 #sql-xxx already exists
原理
MySQL 5.7 以上的版本中，在执行创建或者删除的操作同时，将DML操作日志写入一个缓存中。待修改完成之后再重做到原表上，以保住数据的一致性。这个缓存大小由InnoDB_Online_alter_log_max_size 控制，默认为 128MB，若用户更改表比较频繁，在线 DML 业务压力较大，则 InnoDB_Online_alter_log_max_size 空间不能存放日志，会抛出错误，此时可以调大 InnoDB_Online_alter_log_max_size 获得更多日志缓存空间解决问题 。


```
SELECT * FROM information_schema.InnoDB_TRX; #查看当前运行的所有事务

eg: select trx_state, trx_started, trx_mysql_thread_id, trx_query from information_schema.InnoDB_trx

SELECT * FROM information_schema.InnoDB_LOCKS; #查看当前出现的锁

SELECT * FROM information_schema.InnoDB_LOCK_WAITS; #查看锁等待的对应关系；
```

在执行结果中可以看到是否有表锁等待或者死锁，如果有死锁发生，可以通过下面的命令来杀掉当前运行的事务：
```
KILL id;   # KILL 后面的数字指的是 trx_mysql_thread_id 值。
```