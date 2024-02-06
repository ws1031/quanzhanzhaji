Redis学习笔记-数据类型与API（3）List
---

## List (列表)
### 特点
* 有序
* 可以重复
* 左右两边插入弹出

![List](http://md.ws1031.cn/xsj/2018_7_9_1180112327-5aa749aaa1f16_articlex.jpg)

![List](http://md.ws1031.cn/xsj/2018_7_9_2984455238-5aa749b5b8ad2_articlex.jpg)

### 常用命令

命令|含义|时间复杂度
-|-|-
lrange|获取列表指定索引范围的所有item|O(S+N)，S 为偏移量 start，N 为指定区间内元素的数量。
lpush、rpush|从列表左/右侧插入1-N个值|O(1)
lpop、rpop|从列表左/右侧弹出1个值|O(1)
linsert|在list指定的值前/后插入newValue|O(N)， N 为寻找 pivot 过程中经过的元素数量。
lrem|从列表中删除value相等的项 |O(N)， N 为列表的长度。
ltrim| 按照索引范围修剪列表 | O(N)，N 为被移除的元素的数量。
lindex |获取列表指定索引的item | O(N)， N 为到达下标 index 过程中经过的元素数量。因此，对列表的头元素和尾元素执行 LINDEX 命令，复杂度为O(1)。
llen| 获取列表长度 | O(1)
lset|设置列表指定索引值为newValue |对头元素或尾元素进行 LSET 操作，复杂度为 O(1)。其他情况下，为 O(N)， N 为列表的长度。

#### lrange
* lrange key start end (获取列表指定索引范围的所有item(包含start和end))
> ### \a - b - c - d - e - f
> 索引从左到右 0 ~ 5
> 索引从右到左 -1 ~ -6
```
127.0.0.1:6379> rpush listkey a b c d e f
(integer) 6
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
127.0.0.1:6379> lrange listkey 0 2
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> lrange listkey 1 -2
1) "b"
2) "c"
3) "d"
4) "e"
```

#### lpush、rpush
* lpush key value1 value2 value3 ... (从列表左侧插入1-N个值)
* rpush key value1 value2 value3 ... (从列表右侧插入1-N个值)
```
127.0.0.1:6379> lpush listkey c b a
(integer) 3
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> rpush listkey c b a
(integer) 6
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "c"
4) "c"
5) "b"
6) "a"
```

#### lpop、rpop
* lpop key (从列表左侧弹出一个item)
* rpop key (从列表右侧弹出一个item)
```
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "c"
4) "c"
5) "b"
6) "a"
127.0.0.1:6379> lpop listkey
"a"
127.0.0.1:6379> lrange listkey 0 -1
1) "b"
2) "c"
3) "c"
4) "b"
5) "a"
127.0.0.1:6379> rpop listkey
"a"
127.0.0.1:6379> lrange listkey 0 -1
1) "b"
2) "c"
3) "c"
4) "b"
```

#### linsert
* linsert key before|after value newValue (在list指定的值前|后插入newValue)
```
127.0.0.1:6379> lrange listkey 0 -1
1) "b"
2) "c"
3) "c"
4) "b"
127.0.0.1:6379> linsert listkey before b 0
(integer) 5
127.0.0.1:6379> lrange listkey 0 -1
1) "0"
2) "b"
3) "c"
4) "c"
5) "b"
127.0.0.1:6379> linsert listkey after c 1
(integer) 6
127.0.0.1:6379> lrange listkey 0 -1
1) "0"
2) "b"
3) "c"
4) "1"
5) "c"
6) "b"
```

#### lrem
* lrem key count value (根据count的值，从列表中删除value相等的项)
> (1) count>0，从左到右，删除最多count个value相等的项
> (2) count<0，从右到左，删除最多abs(count)个value相等的项
> (3) count=0，从左到右，删除所有value相等的项
```
127.0.0.1:6379> lrange listkey 0 -1
1) "0"
2) "b"
3) "c"
4) "1"
5) "c"
6) "b"
127.0.0.1:6379> lrem listkey 1 b
(integer) 1
127.0.0.1:6379> lrange listkey 0 -1
1) "0"
2) "c"
3) "1"
4) "c"
5) "b"
127.0.0.1:6379> lrem listkey -1 c
(integer) 1
127.0.0.1:6379> lrange listkey 0 -1
1) "0"
2) "c"
3) "1"
4) "b"
```

#### ltrim
* ltrim key start end (按照索引范围修剪列表)
```
127.0.0.1:6379> lrange listkey 0 -1
1) "c"
2) "b"
3) "a"
4) "a"
5) "b"
6) "c"
127.0.0.1:6379> ltrim listkey 1 3
OK
127.0.0.1:6379> lrange listkey 0 -1
1) "b"
2) "a"
3) "a"
```
#### lindex
* lindex key index (获取列表指定索引的item)

#### llen
* llen key (获取列表长度)
```
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
6) "f"
127.0.0.1:6379> lindex listkey 0
"a"
127.0.0.1:6379> lindex listkey -1
"f"
127.0.0.1:6379> llen listkey
(integer) 6
```

#### lset
* lset key index newValue (设置列表指定索引值为newValue)
```
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "c"
127.0.0.1:6379> lset listkey 2 0
OK
127.0.0.1:6379> lrange listkey 0 -1
1) "a"
2) "b"
3) "0"
```
> #### 更多 List 相关命令：http://www.redis.cn/commands.html#list

### Tips
> lpush + lpop = Stack (栈)
> lpush + rpop = Queue (队列)
> lpush + ltrim = Capped Collection (定容集合)
> lpush + brpop = Message Queue (消息队列) 