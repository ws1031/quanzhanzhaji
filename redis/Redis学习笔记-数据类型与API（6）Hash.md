Redis学习笔记-数据类型与API（6）Hash
---

## Hash (哈希)

### 常用命令

命令|含义|时间复杂度
-|-|-
hget、hset|设置、获取hash key对应的field的value|O(1)
hdel|删除hash key对应的一个或多个field|O(N)，N 为要删除的域的数量
hexists|判断hash key是否有指定的field|O(1)
hlen|获取hash key 的field的数量|O(1)
hmget、hmset|批量、获取hash key的一批field对应的值|O(N)，N 为给定field的数量
hkeys、hvals、hgetall|返回hash key对应所有的field、value、field和value|O(N)，N 为哈希表的大小
hsetnx|设置hash key 对应field (如果field已存在，则失败)|O(1)
hincrby、hincrbyfloat|hash key 对应的field的value自增initCounter|O(1)

#### hget、hset、hdel
* hget key field (获取hash key对应的field的value)
* hset key field value (设置hash key 对应field的value)
* hdel key field (删除hash key 对应field的value)
```
127.0.0.1:6379> hset user:1:info age 23
(integer) 1
127.0.0.1:6379> hget user:1:info age
"23"
127.0.0.1:6379> hset user:1:info name ronaldo
(integer) 1
127.0.0.1:6379> hgetall user:1:info
1) "age"
2) "23"
3) "name"
4) "ronaldo"
127.0.0.1:6379> hdel user:1:info age
(integer) 1
127.0.0.1:6379> hgetall user:1:info
1) "name"
2) "ronaldo"
```
#### hexists、hlen
* hexists key field (判断hash key是否有指定的field)
* hlen key (获取hash key 的field的数量)
```
127.0.0.1:6379> hgetall user:1:info
1) "name"
2) "ronaldo"
127.0.0.1:6379> hexists user:1:info name
(integer) 1
127.0.0.1:6379> hexists user:1:info age
(integer) 0
127.0.0.1:6379> hlen user:1:info
(integer) 1
```
#### hmget、hmset
* hmget key field1 field2 field3 ... (批量获取hash key的一批field对应的值)
* hmset key field1 value1 field2 value2 ... (批量设置hash key的一批field value)

#### hgetall、hvals、hkeys
* hkeys key (返回hash key 对应所有field)
* hvals key (返回hash key 对应所有field的value)
* hgetall key (返回hash key 对应所有的field和value)
```
127.0.0.1:6379> hmset user:2:info age 30 name kaka page 50
OK
127.0.0.1:6379> hgetall user:2:info
1) "age"
2) "30"
3) "name"
4) "kaka"
5) "page"
6) "50"
127.0.0.1:6379> hkeys user:2:info
1) "age"
2) "name"
3) "page"
127.0.0.1:6379> hvals user:2:info
1) "30"
2) "kaka"
3) "50"
```
#### hsetnx、hincrby、hincrbyfloat
* hsetnx key field value (设置hash key 对应field (如果field已存在，则失败))
* hincrby key field initCounter (hash key 对应的field的value自增initCounter)
* hincrbyfloat key field floatCounter (hincrby浮点数版)

> #### 更多 Hash 相关命令：http://www.redis.cn/commands.html#string


### 实战

* 记录网站每个用户个人主页的访问量
```
hincrby user:1:info pageview count
```
* 缓存视频对象基本信息

### String vs Hash

#### 相似的API
|String|Hash|
|-|-|
|get|hget|
|set、setnx|hset、hsetnx
|del|hdel
|incr、incrby、decr、decrby|hincrby
|mset|hmset
|mget|hmget
