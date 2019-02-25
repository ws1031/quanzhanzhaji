Redis学习笔记 - 数据类型与API（1）Key
---

## Key相关命令

### 1. 常用命令
命令|含义|时间复杂度
-|-|-
keys|查找所有符合给定模式 pattern 的 key|O(N)， N 为数据库中 key 的数量
dbsize|计算key的总数|O(1)
exists|检查key是否存在|O(1)
del|删除指定的key-value|O(1)
expire、ttl、persist|设置、查看、去掉key的过期时间|O(1)
type|查看key的类型|O(1)

### 2. keys (遍历key)
> 当key较多时，命令执行时间较长，会造成阻塞，慎用该命令。

* keys * （遍历所有key）
* keys [pattern] (遍历所有正则表达式匹配的key)
### 3. dbsize (计算key的总数)
```
127.0.0.1:6379> mset hello world hehe haha php good phe his
OK
127.0.0.1:6379> keys *
1) "hello"
2) "phe"
3) "php"
4) "hehe"
127.0.0.1:6379> keys he*
1) "hello"
2) "hehe"
127.0.0.1:6379> keys he[h-l]*
1) "hello"
2) "hehe"
127.0.0.1:6379> keys ph?
1) "phe"
2) "php"
127.0.0.1:6379> dbsize
(integer) 4
```

### 4. exists key (检查key是否存在)
### 5. del key [key2 key3 ...] （删除指定的key-value,可一次删除多个）
```
127.0.0.1:6379> exists hello
(integer) 1
127.0.0.1:6379> del hello php
(integer) 2
127.0.0.1:6379> exists hello
(integer) 0
127.0.0.1:6379> get hello
(nil)
```

### 6.  expire、ttl、persist (设置、查看、去掉key的过期时间)
* expire key seconds （key在seconds秒后过期）
* ttl key （查看key剩余过期时间）
> 大于等于0时，表示剩余过期秒数
> -1 表示key存在，并且没有过期时间
> -2 表示key已经不存在了

* persist key （去掉key的过期时间）
```
127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> expire hello 20
(integer) 1
127.0.0.1:6379> ttl hello
(integer) 12
127.0.0.1:6379> get hello
"world"
127.0.0.1:6379> ttl hello
(integer) -2
127.0.0.1:6379> get hello
(nil)

127.0.0.1:6379> set hello world
OK
127.0.0.1:6379> expire hello 20
(integer) 1
127.0.0.1:6379> ttl hello
(integer) 14
127.0.0.1:6379> persist hello
(integer) 1
127.0.0.1:6379> ttl hello
(integer) -1
127.0.0.1:6379> get hello
"world"
```
### 7. type key （查看key的类型）
>  string 
>  hash 
>  list 
>  set 
>  zset 
>  none
```
127.0.0.1:6379> set a 1
OK
127.0.0.1:6379> type a
string
127.0.0.1:6379> sadd myset 1 2 3
(integer) 3
127.0.0.1:6379> type myset
set
```

###  更多 Key 相关命令：http://www.redis.cn/commands.html#generic
