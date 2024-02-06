Redis学习笔记 - 数据类型与API（2）String
---

## String (字符串)

### 1. 使用场景
* 缓存 （key-value、存储json）
* 分布式锁
* 计数器
* Bits

### 2. 常用命令

命令|含义|时间复杂度
-|-|-
set、get、del|设置、获取、删除key-value|O(1)
setnx、set xx|根据key是否存在设置key-value|O(1)
incr、decr、incrby、decrby、incrbyfloat|计数|O(1)
mget、mset|批量操作key-value|O(N),N 为给定 key 的数量
getset|为key设置新值，并返回旧值|O(1)
append|将value追加到旧的value后|O(1)
strlen|返回字符串的长度|O(1)
setrange、getrange|设置、获取字符串指定下标对应的值|O(1)

#### **get、set、del**
* get key (获取key对应的value)
* set key value (设置key-value)
* del key (删除key-value)

#### **incr、decr、incrby、decrby**
* incr key (key自增1，如果key不存在，自增后get(key)=1)
* decr key (key自减1，如果key不存在，自增后get(key)=-1)
* incrby key n (key自增n，如果key不存在，自增后get(key)=n)
* decrby key n (key自减n，如果key不存在，自增后get(key)=-n)

#### **set、setnx、set xx**
* set key value (不管key是否存在，都设置)
* setnx key value (key不存在，才设置)
* set key value xx (key存在，才设置)

```
127.0.0.1:6379> exists php
(integer) 0
127.0.0.1:6379> set php good
OK
127.0.0.1:6379> setnx php bad
(integer) 0
127.0.0.1:6379> set php best xx
OK
127.0.0.1:6379> get php
"best"
127.0.0.1:6379> exists java
(integer) 0
127.0.0.1:6379> setnx java best
(integer) 1
127.0.0.1:6379> exists lua
(integer) 0
127.0.0.1:6379> set lua hehe xx
(nil)
```

#### **mget、mget**
* mget key1 key2 key3 ... (批量获取key，原子操作)
* mset key1 value1 key2 value2 key3 value3 ... (批量设置key-value)

#### **getset、append、strlen**
* getset key newvalue (为key设置新值，并返回旧值)
* append key value (将value追加到旧的value后)
* strlen key (返回字符串的长度（注意中文）)
```
127.0.0.1:6379> get java
"best"
127.0.0.1:6379> getset java hello
"best"
127.0.0.1:6379> get java
"hello"
127.0.0.1:6379> append java world
(integer) 10
127.0.0.1:6379> get java
"helloworld"
127.0.0.1:6379> strlen java
(integer) 10
```

#### **incrbyfloat、getrange、setrange**
* incrbyfloat key 3.5 (为key对应的值增加3.5)
* getrange key start end (获取字符串指定下标所有的值)
* setrange key index value (设置指定下标所对应的值)
```
127.0.0.1:6379> incr counter
(integer) 1
127.0.0.1:6379> incrbyfloat counter 1.1
"2.1"
127.0.0.1:6379> get counter
"2.1"
127.0.0.1:6379> set hello javabest
OK
127.0.0.1:6379> getrange hello 0 2
"jav"
127.0.0.1:6379> setrange hello 4 p
(integer) 8
127.0.0.1:6379> get hello
"javapest"
```

> #### **更多 String 相关命令**：http://www.redis.cn/commands.html#string

### 3. 实战

#### 分布式ID生成器
![Redis分布式ID生成器](http://md.ws1031.cn/xsj/2018_7_6_Redis_Distributed_ID_generator.jpg)