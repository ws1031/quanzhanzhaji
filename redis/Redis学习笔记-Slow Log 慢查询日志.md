Redis学习笔记-Slow Log 慢查询日志
---

## 什么是 SLOW LOG

#### 1. Slow log 是 Redis 用来记录查询执行时间的日志系统。

#### 2. 查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

#### 3. slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

## SLOW LOG 慢查询配置项

### 1. slowlog-log-slower-than (慢查询的阈值（单位：微秒）)
* 特点
1. 当执行查询命令消耗时间大于设置的阈值时，会将该条命令记录到慢查询日志
2. slowlog-log-slower-than=0，记录所有命令
3. slowlog-log-slower-than<0，不记录任何命令
4. 默认值为 10000 （10毫秒，1秒 = 1,000毫秒 = 1,000,000微秒）

### 2. slowlog-max-len (慢查询日志最大条数)
* 特点
1. 先进先出的队列 (即当慢查询日志达到最大条数后，会销毁最早记录的日志条目)
2. 固定长度
3. 保存在内存内 (重启redis会清空慢查询日志)
4. 默认值为 128

### 3. 配置方法
1. 修改配置文件 `redis.conf` ，重启redis
2. 使用命令动态配置
* CONFIG SET slowlog-log-slower-than 100
* CONFIG SET slowlog-max-len 1024
3. 查看慢查询配置
* CONFIG GET slowlog-*
```
127.0.0.1:6379> CONFIG GET slowlog-*
1) "slowlog-log-slower-than"
2) "100"
3) "slowlog-max-len"
4) "1024"
```

## 查看慢查询日志
### 1. SLOWLOG GET [n]
* 获取最新的n条慢查询记录
* 若不加 n ,则获取全部慢查询记录
```
127.0.0.1:6379> SLOWLOG GET 3
1) 1) (integer) 14                # 唯一性(unique)的日志标识符
   2) (integer) 1522808219        # 被记录命令的执行时间点，以 UNIX 时间戳格式表示
   3) (integer) 16                # 查询执行时间，以微秒为单位
   4) 1) "keys"                   # 执行的命令，以数组的形式排列
      2) "*"                      # 这里完整的命令是 "keys *"
2) 1) (integer) 13
   2) (integer) 1522808215
   3) (integer) 7
   4) 1) "set"
      2) "name"
      3) "baicai"
3) 1) (integer) 12
   2) (integer) 1522808198
   3) (integer) 101
   4) 1) "set"
      2) "age"
      3) "25"
```

### 2. SLOWLOG LEN
* 查看当前日志的数量
```
127.0.0.1:6379> SLOWLOG LEN
(integer) 16
```

### 3. SLOWLOG RESET
* 清空日志
```
127.0.0.1:6379> SLOWLOG RESET
OK
127.0.0.1:6379> SLOWLOG LEN
(integer) 0
```

## 慢查询运维经验
#### 1. `slowlog-log-slower-than` 不要设置过大，默认为 10 ms，通常可设置为 1ms。
#### 2. `slowlog-max-len` 不要设置过小，通常可设置在1000左右。
#### 3. 理解命令的生命周期
* 一次查询的生命周期

![一次查询的生命周期](http://md.s1031.cn/xsj/2018_7_9_1384679627-5ac42fbb40a0e_articlex.jpg)

#### 4. 定期持久化慢查询日志
* 因为慢查询日志是保存在内存内，重启redis就会清空慢查询日志，若要保留历史的慢查询日志，可定期将慢查询日志存储到mysql等数据库中。
