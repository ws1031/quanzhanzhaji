MongoDB学习笔记（3）- Mongo Shell 常用查询命令
---

![](https://md.s1031.cn/xsj/2021_1_16_MongoDB封面.jpeg)

> 本文所使用的MongoDB版本为 4.0.10
```
> db.version();
4.0.10
```

### 一、find 命令进行简查询

    find( 查询条件 ，返回的字段)， 

#### 1. 查询时返回所有字段

**db.user.find()  -->  查询user集合中所有的数据**
```
> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

**db.user.find( {"username":"Mary"} )   --> 列出username=Mary的数据**
```
> db.user.find({"username": "Mary"})
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
```

**db.user.find( {"age":50} )   -->  列出 age = 50 的数据**
```
> db.user.find( {"age":50} )
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 2. 查询时只返回指定几个字段

**db.user.find({}, {"username":1})  --> 列出所有人的 username 字段**
```
> db.user.find({}, {"username": 1})
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary" }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin" }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart" }
```

**db.user.find({}, {"tel": 1})  --> 列出所有人的 tel 字段**
```
> db.user.find({}, {"tel": 1})
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7") }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8") }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9") }
```

**db.user.find({} , {"age": 0} )  -->  除了age字段，其他字段都列出来**
```
> db.user.find({}, {"age": 0})
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary" }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin" }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart" }
```

### 二、常用操作符

#### 1. $lt (<)

**查询年龄小于30岁的用户**
```
> db.user.find( {"age": { $lt: 30 } } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
```

#### 2. $lte (<=)

**查询年龄小于等于30岁的用户**
```
> db.user.find( {"age": { $lte: 30 } } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
```

#### 3. $gt (>)

**查询年龄大于30岁的用户**
```
> db.user.find( {"age": { $gt: 30 } } )
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 4. $gte (>= )

**查询年龄大于等于30岁的用户**
```
> db.user.find( {"age": { $gte: 30 } } )
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 5. $ne ( <> )

**查询年龄不等于30岁的用户**
```
> db.user.find( {"age": { $ne: 30 } } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 6. $and (AND)

**查询年龄大于10岁且小于40岁的用户**
```
> db.user.find( { $and: [ { "age": { $lt: 40 } }  , {"age": { $gt: 10 } } ] } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null }
```

#### 7. $or (OR)

**查询年龄小于20岁或者大于40岁的用户**

```
> db.user.find( { $or: [ { "age": { $lt: 20 } }  , {"age": { $gt:40 } } ] } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 8. $in (IN)

**查询年龄为 20, 30, 40 岁的用户**
```
> db.user.find( { "age" : { $in : [20, 30, 40] } } )
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
```

#### 9. $not (NOT)

**查询年龄不是 20, 40 岁的用户（$not）**

```
> db.user.find( { "age" : { $not: { $in : [20, 40] } } } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 10. $nin (NOT IN)

**查询年龄不是 20, 40 岁的用户**
```
> db.user.find( { "age" : { $nin : [20, 40] } } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 11. $mod (取模)

**查询年龄对 4 取模余 2 的用户**
```
> db.user.find( {"age" : { $mod : [4, 2] } } )
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

#### 12. $exists (存在)

> $exists 用于判断键是否

**给 Mary 添加 tel 字段，值设为 null**
```
> db.user.update({ "username": "Mary" }, { $set : { "tel" : null } })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

**列出存在 tel 字段的记录**
```
> db.user.find({ "tel" : { $exists : 1 }})
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null }
```

> 判断值为 null 时要注意：
`db.user.find({ "friends" : null })`
该命令不仅查出值为null的记录，friends 键不存在的记录也会被取出来。
```
> db.user.find({ "tel" : null })
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

> 如果想取出 friends 键存在，并且值为 null 的记录应该这样来取：
```
> db.user.find( { $and: [ { "tel" : null }, { "tel" : {$exists: 1 } } ] } )
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null }
```

#### 13. $where 

> $where : 根据函数返回值来判断是否返回数据

**取出 `年龄 * 3 + 5 小于 100` 用户的**
```
db.user.find( { $where : function(){ 
    return (this.age * 3 + 5 < 100); 
} } );
```
```
> db.user.find( { $where : function(){ return (this.age * 3 + 5 < 100); } } );
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086" }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null }
```

> `this` 代表当前这个记录。

> 通过这个 $where 基本可以实现任意类型的查询。不过不到必要时候不要用这个方法，因为它的速度比一般查询要慢很多。


### 三、使用正则查询

**查找名字中含有 "art" 的记录**

```
> db.user.find( { "username" : /art/ } )
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

**查找以名称以 "mar" 开头且不区分大小写的记录**

```
> db.user.find( {"username": /^mar/i } );
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40 }
```

### 四、查询数组

#### 1. $all：判断某个数组类型字段包含的多个指定值时。

**给 user 集合中的用户添加一些朋友，结果如下：**
```
> db.user.find()
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086", "friend" : [ "Mary", "Jocker", "Martin", "Kart" ] }
{ "_id" : ObjectId("5d2f102414077ad0dab139c7"), "username" : "Mary", "age" : 30, "tel" : null, "friend" : [ "Jocker", "Martin" ] }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40, "friend" : [ "Mary", "Jocker", "Kart" ] }
{ "_id" : ObjectId("5d2f105414077ad0dab139c9"), "username" : "kart", "age" : 50 }
```

**取出 friend 数组中既有 Mary 又有 Jocker 的记录**
```
> db.user.find({ "friend": { $all : ["Mary", "Jocker"] } })
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086", "friend" : [ "Mary", "Jocker", "Martin", "Kart" ] }
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40, "friend" : [ "Mary", "Jocker", "Kart" ] }
```

#### 2. $size：查询拥有指定元素个数的数组

**取出 friend 数组中有3个值的记录**
```
> db.user.find( { "friend" : { $size  : 3 } } )
{ "_id" : ObjectId("5d2f103414077ad0dab139c8"), "username" : "Martin", "age" : 40, "friend" : [ "Mary", "Jocker", "Kart" ] }
```

#### 3. $slice : 返回一个数组的子集

{$slice : 10}  --> 数组中的前10个
{$slice : -10}  -->  数组中的后10个
{$slice : [20, 10] } -->  从数组中下标为20的元素开始，向后去除10个元素

**取出Tom的前3个朋友**

```
> db.user.find( {"username": "Tom"}, { "friend" : {$slice : 3} } )
{ "_id" : ObjectId("5d2f0a4714077ad0dab139c5"), "username" : "Tom", "age" : 12, "tel" : "10086", "friend" : [ "Mary", "Jocker", "Martin" ] }
```

### 五、查询内嵌文档

#### 1. $elemMatch

如有这样的数据结构：
有个comments字段，该字段是一个数组，每一项是一个内嵌的comment对象

```
{
	"_id": ObjectId("5d31b1d24fd0d7ad0a1a1361"),
	"author": "Tom",
	"content": "I am Tom!",
	"comments": [{
		"user": "Mary",
		"score": 3,
		"comment": "Nice!"
	}, {
		"user": "Martin",
		"score": 6,
		"comment": "I'm reading..."
	}, {
		"user": "Jocker",
		"score": 8,
		"comment": "You're kidding me"
	}]
}
{
	"_id": ObjectId("5d31b3114fd0d7ad0a1a1364"),
	"author": "Martin",
	"content": "I am Martin!",
	"comments": [{
		"user": "Tom",
		"score": 5,
		"comment": "Nice!"
	}, {
		"user": "Mary",
		"score": 6,
		"comment": "I'm reading..."
	}, {
		"user": "Jocker",
		"score": 3,
		"comment": "You're kidding me"
	}]
}
{
	"_id": ObjectId("5d31b3314fd0d7ad0a1a1365"),
	"author": "Mary",
	"content": "I am Mary!",
	"comments": [{
		"user": "Tom",
		"score": 3,
		"comment": "Nice!"
	}, {
		"user": "Martin",
		"score": 5,
		"comment": "I'm reading..."
	}, {
		"user": "Jocker",
		"score": 2,
		"comment": "You're kidding me"
	}]
}
```


**查询评论中包含 score 大于5的文章记录**
```
> db.blog.find ({ "comments" : { "$elemMatch" : { "score" : {"$gt" : 5}}} })
{ "_id" : ObjectId("5d31b1d24fd0d7ad0a1a1361"), "author" : "Tom", "content" : "I am Tom!", "comments" : [ { "user" : "Mary", "score" : 3, "comment" : "Nice!" }, { "user" : "Martin", "score" : 6, "comment" : "I'm reading..." }, { "user" : "Jocker", "score" : 8, "comment" : "You're kidding me" } ] }
{ "_id" : ObjectId("5d31b3114fd0d7ad0a1a1364"), "author" : "Martin", "content" : "I am Martin!", "comments" : [ { "user" : "Tom", "score" : 5, "comment" : "Nice!" }, { "user" : "Mary", "score" : 6, "comment" : "I'm reading..." }, { "user" : "Jocker", "score" : 3, "comment" : "You're kidding me" } ] }
```

**查询Tom的文章的评论中 score 大于 5 的评论记录**
```
> db.blog.find ({"author": "Tom"}, { "comments" : { "$elemMatch" : { "score" : {"$gt" : 3}}} })
{ "_id" : ObjectId("5d31b1d24fd0d7ad0a1a1361"), "comments" : [ { "user" : "Martin", "score" : 6, "comment" : "I'm reading..." } ] }
```



---

![欢迎关注公众号【全栈札记】](https://md.s1031.cn/xsj/2021_1_4_扫码_搜索联合传播样式-白色版.png)