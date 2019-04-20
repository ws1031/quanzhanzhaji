## Sorted Set (有序集合)

### 特点
* 有序
* 无重复
* 集合间操作

### 集合 VS 有序集合
集合|有序集合
-|-
无重复元素|无重复元素
无序|有序
element|element + score

### 列表 VS 有序集合
列表| 有序集合
-|-
可以有重复元素|无重复元素
有序|有序
element|element + score

### 常用命令

操作类型|命令
-|-
基本操作|zadd、zrem、zcard、zincrby、zscore
范围操作|zrange、zrangebyscore、zcount、zremrangebyrank
集合操作|zunionstore、zinterstore

命令|含义|时间复杂度
-|-|-
zadd|将一个或多个 member 元素及其 score 值加入到有序集 key 当中|O( M * log(N) )， N 是有序集的基数， M 为成功添加的新成员的数量
zrem|移除有序集 key 中的一个或多个成员，不存在的成员将被忽略|O(M*log(N))， N 为有序集的基数， M 为被成功移除的成员的数量。
zscore|元素的分数|O(1)
zincrby|增加或减少元素的分数|O(log(N))
zcard|元素的总个数|O(1)
zrange|返回指定索引范围内的升序元素【和分值】|O(log(N) + M)，N 为有序集的基数，而 M 为结果集的基数
zrangebyscore|返回指定分数范围内的升序元素【和分值】|O(log(N) + M)，N 为有序集的基数，而 M 为结果集的基数
zcount|返回有序结合内，在指定分数范围内的元素个数|O(log(N) + M)，N 为有序集的基数， M 为值在 min 和 max 之间的元素的数量
zremrangebyrank|删除指定排名内的升序元素|O(log(N) + M)，N 为有序集的基数，而 M 为被移除成员的数量
zremrangebyscore|删除指定分数内的升序元素|O(log(N) + M)，N 为有序集的基数， M 为结果集的基数

#### zadd
* zadd key score element(可以是多对)（向有序集合key添加score和element）

#### zrem
* zrem key element(可以是多个) （删除指定元素）

#### zscore
* zscore key element (返回元素的分数)

#### zincrby
* zincrby key increScore element (增加或减少元素的分数)

#### zcard
* zcard key (返回元素的总个数)

#### zrange 
* zrange key start end [withscores] (返回指定索引范围内的升序元素【和分值】)

#### zrangebyscore
* zrangebyscore key minScore maxScore [withscores] (返回指定分数范围内的升序元素【和分值】)

#### zcount
* zcount key minScore maxScore (返回有序结合内，在指定分数范围内的元素个数)

#### zremrangebyrank
* zremrangebyrank key start end (删除指定排名内的升序元素)

#### zremrangebyscore
* zremrangebyscore key minScore maxScore (删除指定分数内的升序元素)

```
127.0.0.1:6379> zadd report 100 xiaoming 98 xiaohong 85 laowang 60 zhangsan 55 lisi
(integer) 5
127.0.0.1:6379> zscore report laowang
"85"
127.0.0.1:6379> zcard report
(integer) 5
127.0.0.1:6379> zrank report xiaohong
(integer) 3
127.0.0.1:6379> zrank report xiaoming
(integer) 4
127.0.0.1:6379> zrem report lisi
(integer) 1
127.0.0.1:6379> zrange report 0 -1 withscores
1) "zhangsan"
2) "60"
3) "laowang"
4) "85"
5) "xiaohong"
6) "98"
7) "xiaoming"
8) "100"
127.0.0.1:6379> zrangebyscore report 85 100 withscores
1) "laowang"
2) "85"
3) "xiaohong"
4) "98"
5) "xiaoming"
6) "100"
127.0.0.1:6379> zcount report 85 100
(integer) 3
127.0.0.1:6379> zremrangebyrank report 1 1
(integer) 1
127.0.0.1:6379> zrange report 0 -1 withscores
1) "zhangsan"
2) "60"
3) "xiaohong"
4) "98"
5) "xiaoming"
6) "100"
127.0.0.1:6379> zremrangebyscore report 85 98
(integer) 1
127.0.0.1:6379> zrange report 0 -1 withscores
1) "zhangsan"
2) "60"
3) "xiaoming"
4) "100"
```
#### 其他命令
* zrevrank
* zrevrange
* zrevrangebyscore
* zinterstore
* zunionstore

> #### 更多 Sorted Set 相关命令：http://www.redis.cn/commands.html#sorted_set

### 实战
* 排行榜：新书榜、畅销榜、关注榜等。
