## Set（集合）

### 特点
* 无序
* 无重复
* 集合间操作

### 常用命令

命令|含义|时间复杂度
-|-|-
sadd|将一个或多个 member 元素加入到集合 key 当中，已经存在于集合的 member 元素将被忽略|O(N)， N 是被添加的元素的数量。
srem|移除集合 key 中的一个或多个 member 元素，不存在的 member 元素会被忽略。|O(N)， N 为给定 member 元素的数量
smove|将 member 元素从 A 集合移动到 B 集合|O(1)
scard|集合中元素的数量|O(1)
sismember|判断 member 元素是否集合 key 的成员|O(1)
smembers|返回集合 key 中的所有成员|O(N)，N 为集合的基数
srandmember|从集合中随机挑选指定数量元素返回|O(N)，N 为返回数组的元素个数
spop|移除并返回集合中的一个随机元素|O(1)
sdiff、sdiffstore|给定集合之间的差集(将结果保存到新的集合)|O(N)，N 是所有给定集合的成员数量之和。
sinter、sinterstore|给定集合之间的交集(将结果保存到新的集合)|O(N * M)，N 为给定集合当中基数最小的集合，M 为给定集合的个数。
sunion、sunionstore|给定集合之间的并集(将结果保存到新的集合)|O(N)， N 是所有给定集合的成员数量之和

#### sadd、srem
* sadd key element1 element2 ... (向集合key添加element(如果element已经存在，则添加失败))
* srem key element1 element2 ... (移除key中的element元素)

#### scard、sismember、srandmember、smembers
* scard key (计算集合大小)
* sismember key element (判断element是否在集合中)
* srandmember key count （从集合中随机挑选count个元素）
* spop key（从集合中随机弹出一个元素）
* smembers key（获取集合中所有元素）

```
127.0.0.1:6379> sadd setkey go c c++ c# php java python
(integer) 7
127.0.0.1:6379> sadd setkey go
(integer) 0
127.0.0.1:6379> sadd setkey matlab c
(integer) 1
127.0.0.1:6379> smembers setkey
1) "php"
2) "python"
3) "go"
4) "c++"
5) "matlab"
6) "java"
7) "c"
8) "c#"
127.0.0.1:6379> srem setkey c c++
(integer) 2
127.0.0.1:6379> smembers setkey
1) "matlab"
2) "java"
3) "c#"
4) "go"
5) "python"
6) "php"
127.0.0.1:6379> scard setkey
(integer) 6
127.0.0.1:6379> sismember setkey c++
(integer) 0
127.0.0.1:6379> sismember setkey php
(integer) 1
127.0.0.1:6379> srandmember setkey 3
1) "go"
2) "python"
3) "java"
127.0.0.1:6379> srandmember setkey 3
1) "go"
2) "java"
3) "c#"
127.0.0.1:6379> spop setkey
"matlab"
127.0.0.1:6379> spop setkey
"java"
127.0.0.1:6379> smembers setkey
1) "go"
2) "python"
3) "c#"
4) "php"
```
> srandmember 和 spop: spop 从集合中弹出；srandmember 不会破坏集合

#### sdiff、sinter、sunion
* sdiff setkey1 setkey2 (setkey1 setkey2的差集)
* sinter setkey1 setkey2 (setkey1 setkey2的交集)
* sunion setkey1 setkey2 (setkey1 setkey2的并集)
* sdiff|sinter|suion + store destkey setkey1 setkey2 （将差集、交集、并集结果保存到destkey中）

> 示例见下面 **实战** 中的 **粉丝**

> #### 更多 Set 相关命令：http://www.redis.cn/commands.html#set

### 实战
#### 抽奖系统
* spop
#### 记录点赞，踩的用户
#### 标签
* 给用户添加标签
```
sadd user:1:tags tag1 tag2 tag5
sadd user:2:tags tag3 tag4 tag5
...
sadd user:n:tags tag1 tag3 tag5
```
* 给标签添加用户
```
sadd tag:1:users user1 user3
sadd tag:2:users user3 user5 user6
...
sadd tag:n:users user1 user4
```
#### 粉丝
* 关注
```
127.0.0.1:6379> sadd user:1:follow 2 3 5 6 8 9
(integer) 6
127.0.0.1:6379> sadd user:2:follow 1 3 5 7 8 9
(integer) 6
```
* 粉丝
```
127.0.0.1:6379> sadd user:1:fans 2 7 9
(integer) 3
```
* 我关注他，他没关注我
```
127.0.0.1:6379> sdiff user:1:follow user:1:fans
1) "3"
2) "5"
3) "6"
4) "8"
```
* 他关注我，我没关注他
```
127.0.0.1:6379> sdiff user:1:fans user:1:follow
1) "7"
```
* 互粉
```
127.0.0.1:6379> sinter user:1:follow user:1:fans
1) "2"
2) "9"
127.0.0.1:6379> sinterstore user:1:mutual_fans user:1:follow user:1:fans
(integer) 2
127.0.0.1:6379> smembers user:1:mutual_fans
1) "2"
3) "9"
```
* 共同关注
```
127.0.0.1:6379> sinter user:1:follow user:2:follow
1) "3"
2) "5"
3) "8"
4) "9"
```
* 可能认识的人（用户1和用户2关注用户的并集）
```
127.0.0.1:6379> sunion user:1:follow user:2:follow
1) "1"
2) "2"
3) "3"
4) "5"
5) "6"
6) "7"
7) "8"
8) "9"
```
### Tips
> sadd = Tagging (标签)
> spop/srandmember = Random item (随机成员)
> sadd + sinter = Social Craph (社交平台关系)