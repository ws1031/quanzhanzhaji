## 什么是 HyperLogLog
Redis 在 2.8.9 版本添加了 HyperLogLog 结构。

Redis HyperLogLog 的本质还是字符串。

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

### 什么是基数?
比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5。 基数估计就是在误差可接受的范围内，快速计算基数。

## 常用命令

### pfadd key element [element ...]
* 向hyperloglog添加元素

### pfcount key [key ...]
* 计算hyperloglog的独立元素总数

### pfmerge destkey sourcekey [sourcekey ...]
* 合并多个hyperloglog

```
127.0.0.1:6379> pfadd key uid-1 uid-2 uid-3
(integer) 1
127.0.0.1:6379> pfcount key
(integer) 3
127.0.0.1:6379> pfadd key uid-1 uid-3 uid-5 uid-7
(integer) 1
127.0.0.1:6379> pfcount key
(integer) 5
127.0.0.1:6379> pfadd key2 uid-1 uid-2 uid-3 uid-4 uid-5 uid-6 uid-7
(integer) 1
127.0.0.1:6379> pfmerge destkey key key2
OK
127.0.0.1:6379> pfcount destkey
(integer) 8
```

## 使用经验
1. 统计百万独立用户内存消耗仅为15K左右。
2. HyperLogLog独立用户统计会有一定的错误几率（错误率约为0.81%）。
3. HyperLogLog无法获取单个元素。