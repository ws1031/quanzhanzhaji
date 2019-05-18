Linux Shell编程（6）- 字符操作命令
---

## 一、cut 命令

> cut命令用来显示行中的指定部分

### 1. 语法

    cut [选项] 文件名

### 2. 选项

> `-f 列号`：第几列提取
> `-d 分隔符`：按照指定分隔符分隔列，若不设置，默认为制表符（Tab键）

### 3. 应用

#### **处理以制表符分隔的成绩单**

* report.md
```
[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

* 查看 report.md 中的第二列
```
[root/tmp]# cut -f 2 report.md
Name
Zhang
Li
Wang
```
* 查看 report.md 中的第二列和第四列
```
[root/tmp]# cut -f 2,4 report.md
Name    Score
Zhang   99
Li      68
Wang    100
```

#### **处理以 `:` 分隔的 `/etc/passwd` 文件**

* /etc/passwd
```
[root/tmp]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
veewee:x:1000:1000::/home/veewee:/bin/bash
vagrant:x:1001:1001::/home/vagrant:/bin/bash
...省略n行...
memcached:x:996:995:Memcached daemon:/run/memcached:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

* 查看 /etc/passwd 中的第一列和第七列
```
[root/tmp]# cut -f 1,7 -d ":" /etc/passwd
root:/bin/bash
bin:/sbin/nologin
sync:/bin/sync
mail:/sbin/nologin
nobody:/sbin/nologin
sshd:/sbin/nologin
veewee:/bin/bash
vagrant:/bin/bash
...省略n行...
memcached:/sbin/nologin
apache:/sbin/nologin
```

* 查看所有使用 /bin/bash 登录的普通用户
```
[root/tmp]# grep '/bin/bash' /etc/passwd | cut -f 1 -d ":" /etc/passwd | grep -v root
veewee:/bin/bash
vagrant:/bin/bash
```

### 4. cut 命令的局限性

> cut 命令只可以分析分隔符清晰，格式非常工整的文件，类似 `df` 这类的输出内容完全无能为力。

```
[root/tmp]# df -h | cut -f 5
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root  8.4G  1.8G  6.7G   21% /
devtmpfs                 490M     0  490M    0% /dev
tmpfs                    497M     0  497M    0% /dev/shm
tmpfs                    497M  6.7M  491M    2% /run
tmpfs                    497M     0  497M    0% /sys/fs/cgroup
/dev/sda1                497M  133M  365M   27% /boot
vagrant                   62G   58G  4.4G   93% /vagrant
tmpfs                    100M     0  100M    0% /run/user/1001
[root/tmp]# df -h | cut -f 5 -d ' '

1.8G



```

## 二、printf 命令

> printf命令格式化并输出结果到标准输出。
> printf命令并不会自动加入换行符，如果需要换行，需要手工加入换行符。

### 1. 语法

    printf '[输出类型][输出格式]' [内容]
    
### 2. 常用输出类型

> `%ns`：输出字符串。n是数字，指代输出几个字符。
> `%nd`：输出整数。n是数字，指代输出几个数字。

### 3. 常用输出格式

> `\n`：换行
> `\r`：回车
> `\t`：水平制表符（Tab键）

### 4. 应用

#### **printf 和 echo** 
```
[root/tmp]# echo 1 2 3 4 5 6
1 2 3 4 5 6
[root/tmp]# printf 1 2 3 4 5 6
1[root/tmp]#
```

#### **printf 格式化输出**
```
[root/tmp]# printf '%s' 1 2 3 4 5 6
123456[root/tmp]# printf '%s\n' 1 2 3 4 5 6
1
2
3
4
5
6
[root/tmp]# printf '%s%s\n' 1 2 3 4 5 6
12
34
56
[root/tmp]# printf '%s%s %s\n' 1 2 3 4 5 6
12 3
45 6
[root/tmp]# printf '%s%s %s%s\n' 1 2 3 4 5 6
12 34
56
```

#### **格式化输出 report.md**

* report.md
```
[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

* printf 不支持使用管道符
```
[root/tmp]# cat report.md | printf
printf: 用法:printf [-v var] 格式 [参数]
```

* 格式化输出 report.md
```
[root/tmp]# printf '%s\n' $(cat report.md)
ID
Name
Gender
Score
1
Zhang
F
99
2
Li
M
68
3
Wang
M
100
[root/tmp]# printf '%s\t%s\t%s\t%s\n' $(cat report.md)
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

## 三、awk 命令

### 1. 语法

    awk '条件1{动作1} 条件2{动作2} ...' 文件名

#### **条件**

> 一般使用关系表达式作为条件
> `x>10`
> `x<10`
> `x>=10`
> `x<=10`
> `x==10`

#### **动作** 

> 格式化输出
> 流程控制语句

### 2. awk命令的输出

> awk 命令的输出中支持 print 和 printf 命令

* printf：标准格式输出命令，并不会自动加入换行符，如果需要换行，需要手工加入换行符。
* print：会在 printf 输出的基础上加入换行符。（print 只能在 awk 命令中使用，Linux默认并没有 print 命令）。

#### **格式化输出 report.md**

* report.md
```
[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

* 输出 report.md 的第二列和第四列
```
[root/tmp]# awk '{print $2 "\t" $4}' report.md
Name    Score
Zhang   99
Li      68
Wang    100

[root/tmp]# cut -f 2,4 report.md
Name    Score
Zhang   99
Li      68
Wang    100
```

#### **格式化输出 df 命令的结果**

* `df -h`
```
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root  8.4G  1.8G  6.7G   21% /
devtmpfs                 490M     0  490M    0% /dev
tmpfs                    497M     0  497M    0% /dev/shm
tmpfs                    497M  6.7M  491M    2% /run
tmpfs                    497M     0  497M    0% /sys/fs/cgroup
/dev/sda1                497M  133M  365M   27% /boot
vagrant                   62G   58G  4.4G   93% /vagrant
tmpfs                    100M     0  100M    0% /run/user/1001
```

* 只输出df命令的第五列
```
[root/tmp]# df -h | awk '{print $5}'
已用%
21%
0%
0%
2%
0%
27%
93%
0%
```

* 只输出文件系统为 '/dev/mapper/centos-root' 的第五列
```
[root/tmp]# df -h | grep '/dev/mapper/centos-root' | awk '{print $5}'
21%
```

* 去掉后面的 '%'，只保留数字
```
[root/tmp]# df -h | grep '/dev/mapper/centos-root' | awk '{print $5}' | cut -d '%' -f 1
21
```

### 3. BEGIN 和 END 语句块

> BEGIN语句块在awk开始从输入流中读取行之前被执行，这是一个可选的语句块，比如变量初始化、打印输出表格的表头等语句通常可以写在BEGIN语句块中。

> END语句块在awk从输入流中读取完所有的行之后即被执行，比如打印所有行的分析结果这类信息汇总都是在END语句块中完成，它也是一个可选语句块。

* BEGIN 和 END 语句块用法
```
[root/tmp]# df -h | awk 'BEGIN{print "I am begining"}END{print "I am end"}{print $5}'
I am begining
已用%
21%
0%
0%
2%
0%
27%
93%
0%
I am end
```

### 4. FS 分隔符

#### **设置FS分隔符的方法**
> 1. awk `-F '[分隔符]'` '条件1{动作1} 条件2{动作2} ...' 文件名
> 2. awk 'BEGIN{FS="[分隔符]"} 条件1{动作1} 条件2{动作2} ...' 文件名

#### **格式化输出以 `:` 分隔的 `/etc/passwd` 文件内容**

* /etc/passwd
```
[root/tmp]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
veewee:x:1000:1000::/home/veewee:/bin/bash
vagrant:x:1001:1001::/home/vagrant:/bin/bash
...省略n行...
memcached:x:996:995:Memcached daemon:/run/memcached:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

* 格式化输出 `/etc/passwd` 中的第一列和第七列
```
[root/tmp]# cat /etc/passwd | awk -F ':' '{print $1 "\t\t" $7}'
root            /bin/bash
bin             /sbin/nologin
sync            /bin/sync
mail            /sbin/nologin
nobody          /sbin/nologin
sshd            /sbin/nologin
veewee          /bin/bash
vagrant         /bin/bash
...省略n行...
memcached               /sbin/nologin
apache          /sbin/nologin
```
```
[root/tmp]# cat /etc/passwd | awk 'BEGIN{FS=":"} {print $1 "\t\t" $7}'
root            /bin/bash
bin             /sbin/nologin
sync            /bin/sync
mail            /sbin/nologin
nobody          /sbin/nologin
sshd            /sbin/nologin
veewee          /bin/bash
vagrant         /bin/bash
...省略n行...
memcached               /sbin/nologin
apache          /sbin/nologin
```

### 5. 关系表达式

> 一般使用关系表达式作为条件
> `x>10`
> `x<10`
> `x>=10`
> `x<=10`
> `x==10`

#### **解析 report.md**

* report.md
```
[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

* 输出大于80分的人的名字
```
[root/tmp]# cat report.md | grep -v ID | awk '$4 > 80 {print $2}'
Zhang
Wang
```
* 输出小于80分的人的名字
```
[root/tmp]# cat report.md | grep -v ID | awk '$4 < 80 {print $2}'
Li
```
* 输出大于等于99分的人的名字
```
[root/tmp]# cat report.md | grep -v ID | awk '$4 >= 99 {print $2}'
Zhang
Wang
```
* 输出小于等于99分的人的名字
```
[root/tmp]# cat report.md | grep -v ID | awk '$4 <= 99 {print $2}'
Zhang
Li
```
* 输出100分的人的名字
```
[root/tmp]# cat report.md | grep -v ID | awk '$4 == 100 {print $2}'
Wang
```

## 四、sed 命令

> sed 名令是一种几乎包括在所有Unix平台（包括Linux）的轻量级流编辑器。
> sed 命令主要用来对数据进行选取、替换、删除、新增。

### 1. 语法

    sed [选项] '[动作]' [文件名]

### 2. 选项

> `-n`：一般sed命令会输出所有数据，如果加入此选项，则只会输出经过sed命令处理的行。
> `-e`：允许对输入数据应用多条sed命令编辑。
> `-i`：一般sed命令只会将编辑结果输出到屏幕，如果加入此选项，则会直接修改读取数据的文件。

### 3. 动作

> `p`：打印，输出指定的行。
> `i`：插入，在当前行前插入一行或多行。
> `a`：追加，在当前行后添加一行或多行。
> `d`：删除，删除指定的行。
> `c`：行替换，用c后面的字符串替换原数据行。
> `s`：字符串替换，用一个字符串替换另外一个字符串。格式为 `[行范围]s/[旧字符串]/[新字符串]/g` （与vim编辑器的替换格式类似）。

### 4. 查看文件指定行的内容

* 查看 report.md 的第二行数据，如果不加 `-n` 的话默认是输出文件所有数据
```
[root/tmp]# sed '2p' report.md
ID      Name    Gender  Score
1       Zhang   F       99
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

* 查看 report.md 的第二行数据
```
[root/tmp]# sed -n '2p' report.md
1       Zhang   F       99
```

* 查看 report.md 的第一行到第三行数据
```
[root/tmp]# sed -n '1,3p' report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
```

### 5. 在指定行**前**插入内容

* 在第一行**前**插入 'Begin'，但是不修改文件本身
```
[root/tmp]# sed '1i Begin' report.md
Begin
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100

[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

### 6. 在指定行**后**追加内容

* 在第一行**后**插入 'End'，但是不修改文件本身
```
[root/tmp]# sed '1a End' report.md
ID      Name    Gender  Score
End
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

### 7. 删除文件指定行的内容

* 删除第二行到第四行的数据，但是不修改文件本身
```
[root/tmp]# sed '2,4d' report.md
ID      Name    Gender  Score

[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

### 8. 替换指定行的整行内容

* 将第一行替换为 'Hello World'，但是不修改文件本身
```
[root/tmp]# sed '1c Hello World' report.md
Hello World
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100

[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

* 当替换的行没有数据时，无法替换成功
```
[root/tmp]# sed '5c Hello World' report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```

### 9. 替换指定行的指定字符串

#### **命令格式**

    sed [-i] '[行范围]s/[旧字符串]/[新字符串]/g' 文件名

> `-i`：加 `-i` 只会修改源文件；不加的话只会将编辑结果输出到屏幕，不会修改源文件。
> 结尾的 `g` 为全局替换，若不加，则只会替换每一行第一个符合条件的字符串
> `[旧字符串]` 可以应用正则表达式

#### **行范围格式**
    
* 如果不写行范围，则默认替换整个文件的内容
* `n` 表示只替换第 n 行的内容
* `m,n` 表示替换 从第 m 行到第 n 行的内容
* `n,$` 表示替换 从第 n 行一直到文件结尾的内容

#### **应用**

* passwd 文件是从 /etc/passwd 文件截取的部分内容
```
[root/tmp]# cat -n passwd
     1  rootxroot/root/bin/bash
     2  binxbin/bin/sbin/nologin
     3  syncxsync/sbin/bin/sync
     4  mailxmail/var/spool/mail/sbin/nologin
     5  nobodyxNobody//sbin/nologin
     6  sshdxSSH/var/empty/sshd/sbin/nologin
     7  veeweex/home/veewee/bin/bash
     8  vagrantx/home/vagrant/bin/bash
     9  memcachedxMemcached/run/memcached/sbin/nologin
    10  apachexApache/usr/share/httpd/sbin/nologin
```

* 把文章中所有的 `数字:` 替换为 '*'，直接修改源文件
```
[root/tmp]# sed -i 's/[0-9]\{1,\}:/\*/g' passwd
[root/tmp]# cat -n passwd
     1  root:x:**root:/root:/bin/bash
     2  bin:x:**bin:/bin:/sbin/nologin
     3  sync:x:**sync:/sbin:/bin/sync
     4  mail:x:**mail:/var/spool/mail:/sbin/nologin
     5  nobody:x:**Nobody:/:/sbin/nologin
     6  sshd:x:**SSH:/var/empty/sshd:/sbin/nologin
     7  veewee:x:**:/home/veewee:/bin/bash
     8  vagrant:x:**:/home/vagrant:/bin/bash
     9  memcached:x:**Memcached:/run/memcached:/sbin/nologin
    10  apache:x:**Apache:/usr/share/httpd:/sbin/nologin
```

* 把第九行的 'memcached' 替换为 'mem'
```
[root/tmp]# sed -i '9s/emcached/em/g' passwd
[root/tmp]# cat -n passwd
     1  root:x:**root:/root:/bin/bash
     2  bin:x:**bin:/bin:/sbin/nologin
     3  sync:x:**sync:/sbin:/bin/sync
     4  mail:x:**mail:/var/spool/mail:/sbin/nologin
     5  nobody:x:**Nobody:/:/sbin/nologin
     6  sshd:x:**SSH:/var/empty/sshd:/sbin/nologin
     7  veewee:x:**:/home/veewee:/bin/bash
     8  vagrant:x:**:/home/vagrant:/bin/bash
     9  mem:x:**Mem:/run/mem:/sbin/nologin
    10  apache:x:**Apache:/usr/share/httpd:/sbin/nologin
```

* 把第五行的 'Nobody' 替换为 'no'，把第十行的 'Apache' 替换为 'apa'

方法一：多条命令以分号分隔
```
[root/tmp]# sed '5s/Nobody/no/g;10s/Apache/apa/g'  passwd
root:x:**root:/root:/bin/bash
bin:x:**bin:/bin:/sbin/nologin
sync:x:**sync:/sbin:/bin/sync
mail:x:**mail:/var/spool/mail:/sbin/nologin
nobody:x:**no:/:/sbin/nologin
sshd:x:**SSH:/var/empty/sshd:/sbin/nologin
veewee:x:**:/home/veewee:/bin/bash
vagrant:x:**:/home/vagrant:/bin/bash
mem:x:**Mem:/run/mem:/sbin/nologin
apache:x:**apa:/usr/share/httpd:/sbin/nologin
```
方法二：使用 `-e` 允许对输入数据应用多条sed命令编辑
```
[root/tmp]# sed -i -e '5s/Nobody/no/g' -e '10s/Apache/apa/g'  passwd

[root/tmp]# cat -n passwd
     1  root:x:**root:/root:/bin/bash
     2  bin:x:**bin:/bin:/sbin/nologin
     3  sync:x:**sync:/sbin:/bin/sync
     4  mail:x:**mail:/var/spool/mail:/sbin/nologin
     5  nobody:x:**no:/:/sbin/nologin
     6  sshd:x:**SSH:/var/empty/sshd:/sbin/nologin
     7  veewee:x:**:/home/veewee:/bin/bash
     8  vagrant:x:**:/home/vagrant:/bin/bash
     9  mem:x:**Mem:/run/mem:/sbin/nologin
    10  apache:x:**apa:/usr/share/httpd:/sbin/nologin
```

* 把文件中所有的 `:` 替换为制表符（`\t`）
```
[root/tmp]# sed -i 's/:/\t/g' passwd

[root/tmp]# cat -n passwd
     1  root    x       **root  /root   /bin/bash
     2  bin     x       **bin   /bin    /sbin/nologin
     3  sync    x       **sync  /sbin   /bin/sync
     4  mail    x       **mail  /var/spool/mail /sbin/nologin
     5  nobody  x       **no    /       /sbin/nologin
     6  sshd    x       **SSH   /var/empty/sshd /sbin/nologin
     7  veewee  x       **      /home/veewee    /bin/bash
     8  vagrant x       **      /home/vagrant   /bin/bash
     9  mem     x       **Mem   /run/mem        /sbin/nologin
    10  apache  x       **apa   /usr/share/httpd        /sbin/nologin
```

* 把第五行一直到结尾的 'x' 替换为 'w'
```
[root/tmp]# sed -i '5,$s/x/w/g' passwd

[root/tmp]# cat -n passwd
     1  root    x       **root  /root   /bin/bash
     2  bin     x       **bin   /bin    /sbin/nologin
     3  sync    x       **sync  /sbin   /bin/sync
     4  mail    x       **mail  /var/spool/mail /sbin/nologin
     5  nobody  w       **no    /       /sbin/nologin
     6  sshd    w       **SSH   /var/empty/sshd /sbin/nologin
     7  veewee  w       **      /home/veewee    /bin/bash
     8  vagrant w       **      /home/vagrant   /bin/bash
     9  mem     w       **Mem   /run/mem        /sbin/nologin
    10  apache  w       **apa   /usr/share/httpd        /sbin/nologin
```

## 五、sort 命令

> 将数据进行排序，并将排序结果标准输出。

### 1. 语法

    sort [选项] [文件名]

### 2. 选项

> `-n`：以数值型进行排序，默认是使用字符串型排序
> `-r`：反向排序
> `-f`：忽略大小写
> `-t`：执行分隔符，默认的分隔符是制表符
> `-k n[,m]`：按照指定的字段范围排序。从第n字段开始，第m字段结束（默认到行尾）

### 3. 应用

#### **对 'passwd' 文件内容进行排序**
```
[root/tmp]# cat passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
sshd:x:74:74:SSH:/var/empty/sshd:/sbin/nologin
veewee:x:1000:1000::/home/veewee:/bin/bash
vagrant:x:1001:1001::/home/vagrant:/bin/bash
memcached:x:996:995:Memcached:/run/memcached:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

* 正向排序
```
[root/tmp]# sort passwd
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
memcached:x:996:995:Memcached:/run/memcached:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
root:x:0:0:root:/root:/bin/bash
sshd:x:74:74:SSH:/var/empty/sshd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
vagrant:x:1001:1001::/home/vagrant:/bin/bash
veewee:x:1000:1000::/home/veewee:/bin/bash
```

* 反向排序
```
[root/tmp]# sort -r passwd
veewee:x:1000:1000::/home/veewee:/bin/bash
vagrant:x:1001:1001::/home/vagrant:/bin/bash
sync:x:5:0:sync:/sbin:/bin/sync
sshd:x:74:74:SSH:/var/empty/sshd:/sbin/nologin
root:x:0:0:root:/root:/bin/bash
nobody:x:99:99:Nobody:/:/sbin/nologin
memcached:x:996:995:Memcached:/run/memcached:/sbin/nologin
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
bin:x:1:1:bin:/bin:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
```

#### 按照指定行排序

* 指定分隔符是 ":"，用第三个字段排序
```
[root/tmp]# sort -t ":" -k 3,3 passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
veewee:x:1000:1000::/home/veewee:/bin/bash
vagrant:x:1001:1001::/home/vagrant:/bin/bash
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
sshd:x:74:74:SSH:/var/empty/sshd:/sbin/nologin
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
memcached:x:996:995:Memcached:/run/memcached:/sbin/nologin
```

* 有上面结果可知，sort 默认是按照字符串排序，若想按照数字类型排序，则需要添加 `-n` 选项
```
[root/tmp]# sort -n -t ":" -k 3,3 passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
sshd:x:74:74:SSH:/var/empty/sshd:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
memcached:x:996:995:Memcached:/run/memcached:/sbin/nologin
veewee:x:1000:1000::/home/veewee:/bin/bash
vagrant:x:1001:1001::/home/vagrant:/bin/bash
```

#### 展示当前目录下所有文件和目录的大小，并按照从大到小的顺序排列
```
[root/var]# du -s * | sort -nr
130440  lib
57584   cache
49800   tmp
7500    log
12      spool
8       db
0       var
0       run
0       opt
0       mail
0       lock
0       local
```

## 六、wc 命令

    统计文件的行数、单词数和字符数

### 1. 语法

    wc [选项] 文件名
    
### 2. 选项

> `-l`：只统计行数
> `-w`：只统计单词数
> `-m`：只统计字符数

### 3. 应用
* report.md
```
[root/tmp]# cat report.md
ID      Name    Gender  Score
1       Zhang   F       99
2       Li      M       68
3       Wang    M       100
```
* 统计 report.md 的行数、单词数和字符数
```
[root/tmp]# wc report.md
 4 16 57 report.md
```
* 只统计行数
```
[root/tmp]# wc -l report.md
4 report.md
```
