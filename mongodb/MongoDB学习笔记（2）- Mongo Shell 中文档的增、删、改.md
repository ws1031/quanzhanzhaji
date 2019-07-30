MongoDB学习笔记（2）- Mongo Shell 中文档的增、删、改
---

> 本文所使用的MongoDB版本为 4.0.10
```
> db.version();
4.0.10
```

## 一、插入文档

### 1. 插入一个文档

    语法： db.<collection>.insert(document)

向 `test` 数据库中的 `user` 集合中插入一个文档:

```
> use test;
switched to db test

> db.user.insert({ "username" : "Tom", "age" : 10 })
WriteResult({ "nInserted" : 1 })

> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 10 }
```

> 注： _id 字段是系统自动生成的，也可以自己指定任何类型的字，但值不能重复。

### 2. 插入多个文档

    语法： db.<collection>.insert([ document1, document2, ..., documentN ])

向 `test` 数据库中的 `user` 集合中插入多个文档:

```
> db.user.insert([
...     { "username" : "Mary", "age" : 30 },
...     { "username" : "Martin", "age" : 40 },
...     { "username" : "kart", "age" : 50 },
...     { "username" : "Jack", "age" : 20 }
... ])
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 4,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})

> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 10 }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
{ "_id" : ObjectId("5d2f11b814077ad0dab139ca"), "username" : "Jack", "age" : 20 }
```

## 二、删除数据 

    语法： db.<collection>.remove(条件)

删除 user 集合中名字等于 "Jack" 的文档

```
> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 10 }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
{ "_id" : ObjectId("5d2f11b814077ad0dab139ca"), "username" : "Jack", "age" : 20 }

> db.user.remove({ "username" : "Jack" })
WriteResult({ "nRemoved" : 1 })

> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 10 }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

## 三、修改数据 

    语法： update(条件，数据, 是否新增, 是否修改多条)

修改user 集合中年龄等于10的修改为20

**方法一：**

	var u = db.user.findOne( { "age" : 10 } );
	u.age = 20;
	db.user.update( { "age" : 10 } , u ) 或 db.user.save(u);

```
> var u = db.user.findOne({ "age" : 10 })

> u.age = 20
20

> db.user.save(u)
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 20 }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

**方法二：**

    db.user.update( query, object[, upsert_bool, multi_bool] )

```
> var u = db.user.findOne({ "age" : 20 })

> u.age = 10
10

> db.user.update({ "age" : 20 }, u)
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 10 }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

> upsert_bool:如果没有满足查询条件的记录的话，是否新创建这条记录

> 注：如果有多条记录满足{"age":10}，只有第一条记录会被修改。可以设置第四个参数为true，修改所有的记录。

**修改数据时容易出错的地方**

如有这样的数据：
```
{"_id" : 1 , "username" : "abc" , "age" : 20 , "sex" : "boy" }
```

想要修改年龄为22岁，错误代码如下：
```
var u = db.user.update({ "_id" : 1 }, { "age" : 22 })
```

记录修改完之后变成：
```
{ "_id" : 1 , "age" : 22 }
```

正确的方法：
```
var  u = db.user.find({ "_id" : 1 });     --> 取出记录
u.age = 22;    			                --> 在原记录基本上修改
db.user.save(u) 或  db.user.update({ "_id" : 1 } , u)
```

## 四、修改器的使用

### 1. $inc : 加一个数字

把Tom的年龄加2
```
> db.user.update({ "username" : "Tom" }, { $inc : { "age" : 2 } })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find({ "username" : "Tom" })
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12 }
```

### 2. $set : 修改某一个字段，如果该字段不存在就增这个字段

修改Tom的电话号码为10086，如果没有这个字段就新增这个字段
```
> db.user.update({ "username" : "Tom" }, { $set : { "tel" : "10086" } })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find({ "username" : "Tom" })
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
```

**修改数组**

### 3. $push : 向数组中添加元素

向 Tom 的好友中添加一个好友 Jack。
```
> db.user.update({ "username" : "Tom" }, { $push: { "friend" : "Jack" } })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find({ "username": "Tom"}, { "username": 1, "friend": 1 })
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "friend" : [ "Jack" ] }
```

### 4. $each : 遍历操作元素

向 Tom 的好友中批量添加好友 "Mary", "Jocker"。
```
> db.user.update({ "username" : "Tom" }, { $push : { "friend" : { $each : ["Mary", "Jocker"] } } })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find({ "username" : "Tom" }, { "username" : 1, "friend" : 1 })
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "friend" : [ "Jack", "Mary", "Jocker" ] }
```

### 5. $pop : 从数组的首或尾取出数据

从abc的好友中删除最后一个好友

```
> db.user.update({ "username" : "Tom" }, { $pop : { "friend" : 1 }})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find({ "username" : "Tom"}, { "username" : 1, "friend" : 1})
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "friend" : [ "Jack", "Mary" ] }
```

从abc的好友中删除第一个好友
```
> db.user.update({ "username" : "Tom" }, { $pop : { "friend": -1 } })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find({ "username" : "Tom" }, { "username" : 1, "friend" : 1})
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "friend" : [ "Mary" ] }
```

### 6. $unset : 删除一个字段

删除 Tom 的 friend 字段
```
> db.user.update( { "username" : "Tom" }, { $unset : { "friend" : "" } } )
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

> db.user.find({ "username" : "Tom" })
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
```

---

> MongoDB中修改、删除、更新都是瞬间完成的，即客户端只把命令发给服务器，但不会检查这条命令是否执行成功。
> 可以通过在执行完每条命令之后执行　getLastError 来检查是否成功！

```
> db.runCommand({"getLastError": 1})
{
        "connectionId" : 1,
        "n" : 0,
        "syncMillis" : 0,
        "writtenTo" : null,
        "err" : null,
        "ok" : 1
}
```