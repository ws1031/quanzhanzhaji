## 什么是 Bitmaps
Bitmaps 并不是实际的数据类型，而是定义在String类型上的一个面向字节操作的集合。因为字符串是二进制安全的块，他们的最大长度是512M，最适合设置成2^32个不同字节。

Bitmaps 的最大优势之一在存储信息时极其节约空间。例如，在一个以增量用户ID来标识不同用户的系统中，记录用户的四十亿的一个单独bit信息（例如，要知道用户是否想要接收最新的来信）仅仅使用512M内存。

## 常用命令

### 1. getbit key offset
* 获取位图指定索引的值
```
127.0.0.1:6379> set hello big
OK
127.0.0.1:6379> getbit hello 0
(integer) 0
127.0.0.1:6379> getbit hello 1
(integer) 1
127.0.0.1:6379> getbit hello 2
(integer) 1
```

![bitmaps](http://md.ws65535.top/xsj/2018_7_9_973456235-5ac9f1839bda7_articlex.jpg)

### 2. setbit key offset value
* 给位图指定索引设置值，返回该索引位置的原始值

```
127.0.0.1:6379> set hello big
OK
127.0.0.1:6379> getbit hello 7
(integer) 0
127.0.0.1:6379> setbit hello 7 1
(integer) 0
127.0.0.1:6379> get hello
"cig"

127.0.0.1:6379> setbit world 50 1
(integer) 0
127.0.0.1:6379> get world
"\x00\x00\x00\x00\x00\x00 "
127.0.0.1:6379> setbit world 50 0
(integer) 1
127.0.0.1:6379> get world
"\x00\x00\x00\x00\x00\x00\x00"
```
![](http://md.ws65535.top/xsj/2018_7_9_1240966541-5ac9f77d35855_articlex.jpg)

### 3. bitcount key [start end]
* 获取位图指定范围（start到end，单位为字节，如果不指定就是获取全部）位值为1的个数。

```
127.0.0.1:6379> set hello big
OK
127.0.0.1:6379> bitcount hello
(integer) 12
127.0.0.1:6379> setbit hello 7 1
(integer) 0
127.0.0.1:6379> bitcount hello
(integer) 13

127.0.0.1:6379> bitcount hello 0 0
(integer) 4
127.0.0.1:6379> bitcount hello 0 1
(integer) 8
127.0.0.1:6379> bitcount hello 0 2
(integer) 13
127.0.0.1:6379> bitcount hello 1 1
(integer) 4
127.0.0.1:6379> bitcount hello 1 2
(integer) 9
127.0.0.1:6379> bitcount hello 2 2
(integer) 5
```
### 4. bitop and|or|not|xor destkey key [key...]
* 做多个bitmap的and（交集）、or（并集）、not（非）、xor（异或）操作并将结果保存到destkey中。
```
127.0.0.1:6379> set hello big
OK
127.0.0.1:6379> set world big
OK
127.0.0.1:6379> bitop and destkey hello world
(integer) 3
127.0.0.1:6379> get destkey
"big"
127.0.0.1:6379> bitop or destkey hello world
(integer) 3
127.0.0.1:6379> get destkey
"big"
127.0.0.1:6379> bitop not destkey hello
(integer) 3
127.0.0.1:6379> get destkey
"\x9d\x96\x98"
127.0.0.1:6379> bitop xor destkey hello world
(integer) 3
127.0.0.1:6379> get destkey
"\x00\x00\x00"
```

### 5. bitpos key targetBit [start] [end] （起始版本：2.8.7）
* 计算位图指定范围（start到end，单位为字节，如果不指定就是获取全部）第一个偏移量对应的值等于targetBit的位置。
```
127.0.0.1:6379> set hello big
OK
127.0.0.1:6379> bitpos hello 1
(integer) 1
127.0.0.1:6379> bitpos hello 0
(integer) 0
127.0.0.1:6379> bitpos hello 1 2 2
(integer) 17
127.0.0.1:6379> bitpos hello 1 2 3
(integer) 17
127.0.0.1:6379> bitpos hello 0 2 3
(integer) 16
127.0.0.1:6379> bitpos hello 0 0 3
(integer) 0
127.0.0.1:6379> bitpos hello 1 0 3
(integer) 1
```

## 实战应用
### 独立用户访问统计
1. 使用 set 和 Bitmap （前提是用户的ID必须是整型）
2. 1亿用户，五千万独立
数据类型|每个userId占用空间|需要存储的用户量|内存使用总量
-|-|-|-
set|32位(假设userId用的是integer)|50,000,000|32位*50,000,000=200MB
Bitmap|1位|100,000,000|1位*100,000,000=12.5MB

3. 若只有10万独立用户
数据类型|每个userId占用空间|需要存储的用户量|内存使用总量
-|-|-|-
set|32位(假设userId用的是整型)|100,000|32位*100,000=4MB
Bitmap|1位|100,000,000|1位*100,000,000=12.5MB

## 使用经验
1. string类型最大长度为512M。
2. 注意setbit时的偏移量，当偏移量很大时，可能会有较大耗时。
3. 位图不是绝对的好，有时可能更浪费空间。