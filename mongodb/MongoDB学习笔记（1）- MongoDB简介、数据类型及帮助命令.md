MongoDB学习笔记（1）- MongoDB简介及数据类型
---

> 本文所使用的MongoDB版本为 4.0.10
```
> db.version();
4.0.10
```

## 一、MongoDB 介绍

### 1. MongoDB 的特点

MongoDB 是一个可扩展、高性能的 NoSQL 数据库，由 C++ 语言编写，旨在为 web 应用提供高性能可扩展的数据存储解决方案。
它的特点是高性能、易部署、易使用，存储数据非常方便，主要特性有：

* 模式自由，支持动态查询、完全索引，可轻易查询文档中内嵌的对象及数组。
* 面向集合存储，易存储对象类型的数据 , 包括文档内嵌对象及数组。
* 高效的数据存储 , 支持二进制数据及大型对象 ( 如照片和视频 )。
* 支持复制和故障恢复；提供了 主-从、主-主模式的数据复制及服务器之间的数据复制。
* 自动分片以支持云级别的伸缩性，支持水平的数据库集群，可动态添加额外的服务器。

### 2. MongoDB的优点与适用场景

**MongoDB的优点**

* 高性能，速度非常快（如果你的内存足够的话）。
* 没有固定的表结构，不用为了修改表结构而进行数据迁移。
* 查询语言简单，容易上手。
* 使用Sharding实现水平扩展。
* 部署方便。

**MongoDB的适用场景**

* 适合作为信息基础设施的持久化缓存层 。
* 适合实时的插入，更新与查询，并具备应用程序实时数据存储所需的复制及高度伸缩性。
* Mongo 的 BSON 数据格式非常适合文档化格式的存储及查询。
* 适合由数十或数百台服务器组成的数据库。因为Mongo 已经包含了对 MapReduce 引擎的内置支持。

## 二、SQL 与 NoSQL对比 

-|SQL数据库|NoSQL数据库
-|-|-
类型|所有类型都支持SQL标准|存在多种类型，比如文档存储、键值存储、列数据库等
示例|MySQL、SQL Server、Oracle|MongoDB、HBase、Cassandra
数据存储模型|数据被存储在表的行和列中，其中每一列都有一个特定类型。<br>表通常都是按照标准化原则创建的。<br>使用联接来从多个表中检索数据。|数据模型取决于数据库类型。<br>比如数据被存储为键值对以用于键值存储。在基于文档的数据库中，数据会被存储为文档。<br>NoSQL的数据模型是灵活的，与SQL数据库死板的表模型相反。
模式|固定的结构和模式，因此对模式的任何变更都涉及修改数据库|动态模式，通过扩展或修改当前模式就能适应新的数据类型或结构。<br>可以动态添加新的字段
可扩展性|使用了纵向扩展方式。这意味着随着负荷的增加，需要购买更大、更贵的服务器来容纳数据。|使用了横向扩展方式。这意味着可以将数据负荷分散到多台廉价服务器上。
支持事务|支持ACID和事务|支持分区和可用性,会损害事务。<br>事务存在于某个级别，比如数据库级别或文档级别。
一致性|强一致性|取决于产品。有些产品选择提供强一致性，而有些提供最终一致性。
查询功能|可通过易用的GUI界面来使用|查询可能需要编程专业技术和知识。与UI不同,其重心在于功能和编程接口

### MongoDB 与 Mysql 概念对应关系

|mongodb|mysql|
|-|-|
|数据库（datebase）|数据库（datebase）|
|集合（collection）|表（table）|
|文档（document）|记录（row）|
|字段|列 / 字段|
|索引|索引|
|嵌入和引用| 表内联结|

## 三、MongoDB 支持的数据类型

### 1. null

**null用于表示空值或不存在的字段**
```json
{ "x" : null }
```

### 2. 布尔

**布尔类型有两个值 true 和 false**
```json
{ "x": true }
```

### 3. 32位整数
在 Mongo Shell 中不支持这个类型。JavaScript仅支持64位浮点数，所以32位整数会被自动转换为64位浮点数。

### 4. 64位整数
在 Mongo Shell 中也不支持这个类型。Mongo Shell 会使用一个特殊的内嵌文档来显示64位整数。

### 5. 64位浮点数
Mongo Shell 中的数字都是这种类型。
```json
{  "pi" : 3.14 }
```

> JavaScript 中只有一种 “数字” 类型。因为 MongoDB 中有3种数字类型(32位整数、64位整数和64位浮点数), shell 必须绕过 JavaScript 的限制。默认情况下，shell 中的数字都被 MongoDB 当做是双精度数。这意味着如果你从数据库中获得的是一个32位整数，修改文档后，将文档存回数据库的时候，这个整数也被转换成了浮点数，即便保持这个整数原封不动也会这样的。所以明智的做法是尽量不要在 shell 下覆盖整个文档。

> 数字只能表示为双精度数(64位浮点数)的另外一个问题是，有些64位的整数并不能精确地表示为64位浮点数。所以，如果存入了一个64位整数，在shell中查看，它会显示为一个内嵌文档。但是在数据库中实际存储的值是准确的。

> 32位的整数都能用64位的浮点数精确表示，所以显示起来没什么特别的。

### 6. 字符串

**UTF-8字符串都可表示为字符串类型**
```json
{ "x" : "abcde" }
```

### 7. 对象id

**对象id使用12字节的存储空间，每个字节两位十六进制数字，是一个24位的字符串。**
```json
{ "_id" : ObjectId() }
```

### 8. 日期

**日期类型存储的是亳秒级的时间戳，不存储时区。**
```json
{ "d" : new Date() }
```

### 9. 正则表达式

**文档中可以包含正则表达式，采用JavaScript的正则表达式语法。**
```
{ "x" : /^abc/i }
```

### 10. 代码

**文档中可以包含JavaScript代码**
```json
{ "x" : function(){/********/} }
```

### 11. 数组

**值的集合或者列表可以表示成数组**
```json
{ "d" : [1,2,3,4,5] }
```

### 12. 内嵌文档
**文档中可以包含其他文档，也可以作为值嵌入到父文档中。**
```json
{ "x" : { "y" : "z" } }
```

### 13. undefined

**文档中也可以使用 undefined（(未定义）类型（JavaScript中 null 和 undefined 是不同的类型）。**
```json
{ "a" : undefined }
```

### 14. 二进制数据

**二进制数据可以由任意字节的串组成。可用于存储图片等二进制文件。不过在 Mongo Shell 中无法使用。**


## 四、Mongo Shell 帮助命令


### 1. 系统级帮助：**help**

```
> help
        db.help()                    help on db methods
        db.mycoll.help()             help on collection methods
        sh.help()                    sharding helpers
        rs.help()                    replica set helpers
        help admin                   administrative help
        help connect                 connecting to a db help
        help keys                    key shortcuts
        help misc                    misc things to know
        help mr                      mapreduce

        # 显示所有数据库
        show dbs                     show database names
        # 显示所有集合
        show collections             show collections in current database
        # 显示当前数据库所有用户
        show users                   show users in current database
        show profile                 show most recent system.profile entries with time >= 1ms
        show logs                    show the accessible logger names
        show log [name]              prints out the last segment of log in memory, 'global' is default
        use <db_name>                set current database
        db.foo.find()                list objects in collection foo
        db.foo.find( { a : 1 } )     list objects in foo where a == 1
        it                           result of the last line evaluated; use to further iterate
        DBQuery.shellBatchSize = x   set default number of items to display on shell
        exit                         quit the mongo shell
```

### 2. 查看数据库上可用的操作：**db.help()** 

```
> db.help()
DB methods:
        db.adminCommand(nameOrDocument) - switches to 'admin' db, and runs command [just calls db.runCommand(...)]
        db.aggregate([pipeline], {options}) - performs a collectionless aggregation on this database; returns a cursor
        db.auth(username, password)
        db.cloneDatabase(fromhost) - deprecated
        db.commandHelp(name) returns the help for the command
        db.copyDatabase(fromdb, todb, fromhost) - deprecated
        db.createCollection(name, {size: ..., capped: ..., max: ...})
        db.createView(name, viewOn, [{$operator: {...}}, ...], {viewOptions})
        db.createUser(userDocument)
        db.currentOp() displays currently executing operations in the db
        # 删除数据库
        db.dropDatabase()
        db.eval() - deprecated
        db.fsyncLock() flush data to disk and lock server for backups
        db.fsyncUnlock() unlocks server following a db.fsyncLock()
        db.getCollection(cname) same as db['cname'] or db.cname
        db.getCollectionInfos([filter]) - returns a list that contains the names and options of the db's collections
        # 查看当前数据库中的所有集合
        db.getCollectionNames()
        db.getLastError() - just returns the err msg string
        db.getLastErrorObj() - return full status object
        db.getLogComponents()
        db.getMongo() get the server connection object
        db.getMongo().setSlaveOk() allow queries on a replication slave server
        db.getName()
        db.getPrevError()
        db.getProfilingLevel() - deprecated
        db.getProfilingStatus() - returns if profiling is on and slow threshold
        db.getReplicationInfo()
        db.getSiblingDB(name) get the db at the same server as this one
        db.getWriteConcern() - returns the write concern used for any operations on this db, inherited from server object if set
        db.hostInfo() get details about the server's host
        db.isMaster() check replica primary status
        db.killOp(opid) kills the current operation in the db
        db.listCommands() lists all the db commands
        db.loadServerScripts() loads all the scripts in db.system.js
        db.logout()
        db.printCollectionStats()
        db.printReplicationInfo()
        db.printShardingStatus()
        db.printSlaveReplicationInfo()
        db.dropUser(username)
        db.repairDatabase()
        db.resetError()
        db.runCommand(cmdObj) run a database command.  if cmdObj is a string, turns it into {cmdObj: 1}
        db.serverStatus()
        db.setLogLevel(level,<component>)
        db.setProfilingLevel(level,slowms) 0=off 1=slow 2=all
        db.setWriteConcern(<write concern doc>) - sets the write concern for writes to the db
        db.unsetWriteConcern(<write concern doc>) - unsets the write concern for writes to the db
        db.setVerboseShell(flag) display extra information in shell output
        db.shutdownServer()
        db.stats()
        db.version() current version of the server
```

### 3. 查看集合上可用的操作：**db.集合名.help()** 

```
> db.user.help()
DBCollection help
        db.user.find().help() - show DBCursor help
        db.user.bulkWrite( operations, <optional params> ) - bulk execute write operations, optional parameters are: w, wtimeout, j
        # 集合中的记录数
        db.user.count( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
        db.user.countDocuments( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
        db.user.estimatedDocumentCount( <optional params> ) - estimate the document count using collection metadata, optional parameters are: maxTimeMS
        db.user.copyTo(newColl) - duplicates collection by copying all documents to newColl; no indexes are copied.
        db.user.convertToCapped(maxBytes) - calls {convertToCapped:'user', size:maxBytes}} command
        db.user.createIndex(keypattern[,options])
        db.user.createIndexes([keypatterns], <options>)
        # 集合大小
        db.user.dataSize()
        db.user.deleteOne( filter, <optional params> ) - delete first matching document, optional parameters are: w, wtimeout, j
        db.user.deleteMany( filter, <optional params> ) - delete all matching documents, optional parameters are: w, wtimeout, j
        db.user.distinct( key, query, <optional params> ) - e.g. db.user.distinct( 'x' ), optional parameters are: maxTimeMS
        # 删除集合
        db.user.drop() drop the collection
        db.user.dropIndex(index) - e.g. db.user.dropIndex( "indexName" ) or db.user.dropIndex( { "indexKey" : 1 } )
        # 删除集合内的所有索引
        db.user.dropIndexes()
        db.user.ensureIndex(keypattern[,options]) - DEPRECATED, use createIndex() instead
        db.user.explain().help() - show explain help
        db.user.reIndex()
        db.user.find([query],[fields]) - query is an optional query filter. fields is optional set of fields to return.
                                                      e.g. db.user.find( {x:77} , {name:1, x:1} )
        db.user.find(...).count()
        db.user.find(...).limit(n)
        db.user.find(...).skip(n)
        db.user.find(...).sort(...)
        db.user.findOne([query], [fields], [options], [readConcern])
        db.user.findOneAndDelete( filter, <optional params> ) - delete first matching document, optional parameters are: projection, sort, maxTimeMS
        db.user.findOneAndReplace( filter, replacement, <optional params> ) - replace first matching document, optional parameters are: projection, sort, maxTimeMS, upsert, returnNewDocument
        db.user.findOneAndUpdate( filter, update, <optional params> ) - update first matching document, optional parameters are: projection, sort, maxTimeMS, upsert, returnNewDocument
        db.user.getDB() get DB object associated with collection
        db.user.getPlanCache() get query plan cache associated with collection
        db.user.getIndexes()
        db.user.group( { key : ..., initial: ..., reduce : ...[, cond: ...] } )
        db.user.insert(obj)
        db.user.insertOne( obj, <optional params> ) - insert a document, optional parameters are: w, wtimeout, j
        db.user.insertMany( [objects], <optional params> ) - insert multiple documents, optional parameters are: w, wtimeout, j
        db.user.mapReduce( mapFunction , reduceFunction , <optional params> )
        db.user.aggregate( [pipeline], <optional params> ) - performs an aggregation on a collection; returns a cursor
        db.user.remove(query)
        db.user.replaceOne( filter, replacement, <optional params> ) - replace the first matching document, optional parameters are: upsert, w, wtimeout, j
        db.user.renameCollection( newName , <dropTarget> ) renames the collection.
        db.user.runCommand( name , <options> ) runs a db command with the given name where the first param is the collection name
        db.user.save(obj)
        db.user.stats({scale: N, indexDetails: true/false, indexDetailsKey: <index key>, indexDetailsName: <index name>})
        db.user.storageSize() - includes free space allocated to this collection
        db.user.totalIndexSize() - size in bytes of all the indexes
        db.user.totalSize() - storage allocated for all data and indexes
        db.user.update( query, object[, upsert_bool, multi_bool] ) - instead of two flags, you can pass an object with fields: upsert, multi
        db.user.updateOne( filter, update, <optional params> ) - update the first matching document, optional parameters are: upsert, w, wtimeout, j
        db.user.updateMany( filter, update, <optional params> ) - update all matching documents, optional parameters are: upsert, w, wtimeout, j
        db.user.validate( <full> ) - SLOW
        db.user.getShardVersion() - only for use with sharding
        db.user.getShardDistribution() - prints statistics about data distribution in the cluster
        db.user.getSplitKeysForChunks( <maxChunkSize> ) - calculates split points over all chunks and returns splitter function
        db.user.getWriteConcern() - returns the write concern used for any operations on this collection, inherited from server/db if set
        db.user.setWriteConcern( <write concern doc> ) - sets the write concern for writes to the collection
        db.user.unsetWriteConcern( <write concern doc> ) - unsets the write concern for writes to the collection
        db.user.latencyStats() - display operation latency histograms for this collection

```