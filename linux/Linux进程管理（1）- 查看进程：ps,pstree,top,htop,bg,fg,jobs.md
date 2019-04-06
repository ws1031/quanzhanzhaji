## 一、进程管理的作用

* 判断服务器健康状态
* 查看系统中所有进程
* 杀死进程

## 二、`ps` 命令（查看当前系统中进程的快照）

* `ps` 命令的输出说明

> USER：该命令是由哪个用户产生的。
> PID：进程的ID号。
> %CPU：该进程占用CPU资源的百分比。
> %MEM：该进城占用物理内存的百分比。
> VSZ：该进程占用虚拟内存的大小，单位KB。
> RSS：该进程占用实际物理内存的大小，单位KB。
> TTY：该进程在哪个终端中运行。其中tty1-tty7代表本地控制台终端，tty1-tty6是本地字符界面终端，tty7是图形终端。pst/0-255代表虚拟终端。
> STAT：进程状态。常见的状态有：
>   > R：运行
>   > S：睡眠
>   > T：停止状态
>   > s：包含子进程
>   > +：位于后台
>
> START：该进程的启动时间
> TIME：该进程占用CPU的运算时间，注意不是系统时间
> COMMAND：产生此进程的命令

### 1. 不加参数执行ps命令

```
[vagrant/tmp] ]$ps
  PID TTY          TIME CMD
 1558 pts/0    00:00:00 bash
 2990 pts/0    00:00:00 ps
```
结果默认会显示4列信息。

* PID: 运行着的命令(CMD)的进程编号
* TTY: 命令所运行的终端
* TIME: 运行着的该命令所占用的CPU处理时间
* CMD: 该进程所运行的命令

### 2. 显示所有当前进程
```
ps -aux
```
这个命令的结果或许会很长。为了便于查看，可以结合less命令和管道来使用。
```
[vagrant/tmp] ]$ps -aux | less
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.5  33788  2984 ?        Ss   00:41   0:04 /sbin/init
root         2  0.0  0.0      0     0 ?        S    00:41   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    00:41   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   00:41   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S    00:41   0:01 [rcu_sched]
root         8  0.0  0.0      0     0 ?        S    00:41   0:01 [rcuos/0]
root         9  0.0  0.0      0     0 ?        S    00:41   0:00 [rcu_bh]
... 中间省略n行 ...
root       832  0.0  3.3 423224 16632 ?        Ss   00:41   0:02 php-fpm: master process (/etc/php/7.2/fpm/php-fpm.conf)
root       836  0.0  0.2  26016  1088 ?        Ss   00:41   0:00 cron
mysql      865  0.0 10.8 685816 54500 ?        Ssl  00:41   0:16 /usr/sbin/mysqld
www-data   877  0.0  3.5 423808 17984 ?        S    00:41   0:03 php-fpm: pool www
www-data   878  0.0  3.4 425852 17508 ?        S    00:41   0:02 php-fpm: pool www
root       983  0.0  0.2  84680  1444 ?        Ss   00:41   0:00 nginx: master process /usr/sbin/nginx
:
```

### 3. 根据用户过滤进程

在需要查看特定用户进程的情况下，我们可以使用 -u 参数。比如我们要查看用户'www-data'的进程，可以通过下面的命令：

```
[vagrant/tmp] ]$ps -u www-data
  PID TTY          TIME CMD
  877 ?        00:00:03 php-fpm7.2
  878 ?        00:00:02 php-fpm7.2
  984 ?        00:00:01 nginx
  985 ?        00:00:01 nginx
  986 ?        00:00:00 nginx
  987 ?        00:00:01 nginx
```

### 4. 通过cpu和内存使用来过滤进程
* 根据 `CPU 使用` 来降序排序
```
[vagrant/tmp] ]$ps -aux --sort -pcpu | less
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root      3617  5.5  0.1  20364   956 pts/0    D    07:26   0:00 cp -r /home/ .
root         1  0.0  0.5  33788  2984 ?        Ds   00:41   0:04 /sbin/init
root         2  0.0  0.0      0     0 ?        S    00:41   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    00:41   0:00 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   00:41   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S    00:41   0:01 [rcu_sched]
root         8  0.0  0.0      0     0 ?        S    00:41   0:01 [rcuos/0]
... 中间省略n行 ...
root       836  0.0  0.2  26016  1088 ?        Ss   00:41   0:00 cron
mysql      865  0.0 10.8 685816 54500 ?        Ssl  00:41   0:17 /usr/sbin/mysqld
www-data   877  0.0  3.5 423808 17984 ?        S    00:41   0:03 php-fpm: pool www
www-data   878  0.0  3.4 425852 17508 ?        S    00:41   0:02 php-fpm: pool www
:
```
* 根据 `内存使用` 来降序排序
```
[vagrant/tmp] ]$ps -aux --sort -pmem | less
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mysql      865  0.0 10.8 685816 54500 ?        Ssl  00:41   0:17 /usr/sbin/mysqld
www-data   877  0.0  3.5 423808 17984 ?        S    00:41   0:03 php-fpm: pool www
www-data   878  0.0  3.4 425852 17508 ?        S    00:41   0:02 php-fpm: pool www
root       832  0.0  3.3 423224 16632 ?        Ss   00:41   0:02 php-fpm: master process (/etc/php/7.2/fpm/php-fpm.conf)
root      1538  0.0  0.8 103744  4128 ?        Ss   00:45   0:00 sshd: vagrant [priv]
vagrant   1558  0.0  0.7  22668  3992 pts/0    Ss   00:45   0:00 -bash
root      1468  0.0  0.6  61556  3056 ?        Ss   00:41   0:00 /usr/sbin/sshd -D
root         1  0.0  0.5  33788  2984 ?        Ss   00:41   0:04 /sbin/init
... 中间省略n行 ...
root        30  0.0  0.0      0     0 ?        S    00:41   0:00 [fsnotify_mark]
root        31  0.0  0.0      0     0 ?        S    00:41   0:00 [ecryptfs-kthrea]
root        32  0.0  0.0      0     0 ?        S<   00:41   0:00 [crypto]
root        44  0.0  0.0      0     0 ?        S<   00:41   0:00 [kthrotld]
:
```
* 我们也可以将它们合并到一个命令，并通过管道显示前10个结果：
```
[vagrant/tmp] ]$ps -aux --sort -pmem,+pcpu | head -n 10
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mysql      865  0.0 10.8 685816 54500 ?        Ssl  00:41   0:17 /usr/sbin/mysqld
www-data   877  0.0  3.5 423808 17984 ?        S    00:41   0:03 php-fpm: pool www
www-data   878  0.0  3.4 425852 17508 ?        S    00:41   0:02 php-fpm: pool www
root       832  0.0  3.3 423224 16632 ?        Ss   00:41   0:02 php-fpm: master process (/etc/php/7.2/fpm/php-fpm.conf)
root      1538  0.0  0.8 103744  4128 ?        Ss   00:45   0:00 sshd: vagrant [priv]
vagrant   1558  0.0  0.7  22668  3992 pts/0    Ss   00:45   0:00 -bash
root      1468  0.0  0.6  61556  3056 ?        Ss   00:41   0:00 /usr/sbin/sshd -D
root         1  0.0  0.5  33788  2984 ?        Ss   00:41   0:04 /sbin/init
root       509  0.0  0.4  10220  2416 ?        Ss   00:41   0:00 dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
```

### 5. 通过进程名过滤
* 使用 -C 参数，后面跟你要找的进程的名字。比如想显示一个名为getty的进程的信息，就可以使用下面的命令：
```
[vagrant/tmp] ]$ps -fC nginx
UID        PID  PPID  C STIME TTY          TIME CMD
root       983     1  0 00:41 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data   984   983  0 00:41 ?        00:00:01 nginx: worker process
www-data   985   983  0 00:41 ?        00:00:02 nginx: worker process
www-data   986   983  0 00:41 ?        00:00:00 nginx: worker process
www-data   987   983  0 00:41 ?        00:00:02 nginx: worker process
```

### 6. 使用PS实时监控进程状态
ps 命令会显示你系统当前的进程状态，但是这个结果是静态的。

当有一种情况，我们需要像上面第四点中提到的通过CPU和内存的使用率来筛选进程，并且我们希望结果能够每秒刷新一次。为此，我们可以将ps命令和watch命令结合起来。

如果输出太长，我们也可以限制它，比如前20条，我们可以使用head命令来做到。
```
[vagrant/tmp] ]$watch -n 1 'ps -aux --sort -pmem | head -n 20'
Every 1.0s: ps -aux --sort -pmem | head -n 20                                                                                                                                              Fri May  4 07:45:30 2018

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mysql      865  0.0 10.8 685816 54644 ?        Ssl  00:41   0:17 /usr/sbin/mysqld
www-data   878  0.0  4.1 425984 20980 ?        S    00:41   0:02 php-fpm: pool www
www-data   877  0.0  3.9 425988 19984 ?        S    00:41   0:03 php-fpm: pool www
root       832  0.0  3.3 423224 16632 ?        Ss   00:41   0:02 php-fpm: master process (/etc/php/7.2/fpm/php-fpm.conf)
root      1538  0.0  0.8 103744  4128 ?        Ss   00:45   0:00 sshd: vagrant [priv]
vagrant   1558  0.0  0.7  22668  4000 pts/0    Ss   00:45   0:00 -bash
root      1468  0.0  0.6  61556  3056 ?        Ss   00:41   0:00 /usr/sbin/sshd -D
root         1  0.0  0.5  33788  2984 ?        Ss   00:41   0:04 /sbin/init
vagrant   4558  0.8  0.5  15080  2624 pts/0    S+   07:45   0:00 watch -n 1 ps -aux --sort -pmem | head -n 20
www-data   986  0.0  0.5  85280  2600 ?        S    00:41   0:00 nginx: worker process
root       509  0.0  0.4  10220  2416 ?        Ss   00:41   0:00 dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
www-data   984  0.0  0.4  84980  2360 ?        S    00:41   0:02 nginx: worker process
vagrant   1557  0.0  0.4 103744  2148 ?        S    00:45   0:05 sshd: vagrant@pts/0
www-data   985  0.0  0.3  84980  1868 ?        S    00:41   0:02 nginx: worker process
www-data   987  0.0  0.3  84980  1868 ?        S    00:41   0:02 nginx: worker process
root       380  0.0  0.3  43640  1816 ?        Ss   00:41   0:00 /lib/systemd/systemd-logind
redis      995  0.0  0.3  36992  1760 ?        Ssl  00:41   0:13 /usr/bin/redis-server 127.0.0.1:6379
root       266  0.0  0.3  51672  1748 ?        Ss   00:41   0:00 /lib/systemd/systemd-udevd --daemon
syslog     396  0.0  0.3 255836  1596 ?        Ssl  00:41   0:00 rsyslogd
```

这里的动态查看并不像top或者htop命令一样。使用ps的好处是你能够自定义显示你想查看的字段。

举个例子，如果你只需要看名为'root'用户的信息，你可以使用下面的命令：
```
[vagrant/tmp] ]$watch -n 1 'ps -U root -u --sort -pmem | head -n 10'
Every 1.0s: ps -U root -u --sort -pmem | head -n 10                                                                                                                                        Fri May  4 07:51:56 2018

USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       832  0.0  3.3 423224 16632 ?        Ss   00:41   0:02 php-fpm: master process (/etc/php/7.2/fpm/php-fpm.conf)
root      1538  0.0  0.8 103744  4128 ?        Ss   00:45   0:00 sshd: vagrant [priv]
root      1468  0.0  0.6  61556  3056 ?        Ss   00:41   0:00 /usr/sbin/sshd -D
root         1  0.0  0.5  33788  2984 ?        Ss   00:41   0:04 /sbin/init
root       509  0.0  0.4  10220  2416 ?        Ss   00:41   0:00 dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
root       380  0.0  0.3  43640  1816 ?        Ss   00:41   0:00 /lib/systemd/systemd-logind
root       266  0.0  0.3  51672  1748 ?        Ss   00:41   0:00 /lib/systemd/systemd-udevd --daemon
root       983  0.0  0.2  84680  1440 ?        Ss   00:41   0:00 nginx: master process /usr/sbin/nginx
root       597  0.0  0.2  23416  1112 ?        Ss   00:41   0:00 rpcbind
```

## 三、`pstree` 命令（查看进程树）

* 选项

    -p：显示进程的PID
    -u：显示进程的所属用户

* 实例
```
[vagrant/tmp] ]$pstree -up
init(1)─┬─VBoxService(1046)─┬─{VBoxService}(1049)
        │                   ├─{VBoxService}(1051)
        │                   ├─{VBoxService}(1052)
        │                   ├─{VBoxService}(1053)
        │                   ├─{VBoxService}(1054)
        │                   ├─{VBoxService}(1057)
        │                   └─{VBoxService}(1058)
        ├─cron(836)
        ├─dbus-daemon(343,messagebus)
        ├─dhclient(509)
        ├─getty(791)
        ├─getty(793)
        ├─getty(797)
        ├─getty(798)
        ├─getty(801)
        ├─getty(1154)
        ├─mysqld(865,mysql)─┬─{mysqld}(900)
        │                   ├─{mysqld}(901)
        │                   ├─{mysqld}(902)
        │                   ├─{mysqld}(903)
        │                   ├─{mysqld}(904)
        │                   ├─{mysqld}(905)
        │                   ├─{mysqld}(906)
        │                   ├─{mysqld}(909)
        │                   ├─{mysqld}(910)
        │                   ├─{mysqld}(913)
        │                   ├─{mysqld}(940)
        │                   ├─{mysqld}(941)
        │                   ├─{mysqld}(942)
        │                   ├─{mysqld}(943)
        │                   ├─{mysqld}(1011)
        │                   ├─{mysqld}(1079)
        │                   ├─{mysqld}(1174)
        │                   └─{mysqld}(2360)
        ├─nginx(983)─┬─nginx(984,www-data)
        │            ├─nginx(985,www-data)
        │            ├─nginx(986,www-data)
        │            └─nginx(987,www-data)
        ├─php-fpm7.2(832)─┬─php-fpm7.2(877,www-data)
        │                 └─php-fpm7.2(878,www-data)
        ├─redis-server(995,redis)─┬─{redis-server}(999)
        │                         └─{redis-server}(1000)
        ├─rpc.idmapd(383)
        ├─rpc.statd(673,statd)
        ├─rpcbind(597)
        ├─rsyslogd(396,syslog)─┬─{rsyslogd}(412)
        │                      ├─{rsyslogd}(413)
        │                      └─{rsyslogd}(414)
        ├─sshd(1468)───sshd(1538)───sshd(1557,vagrant)───bash(1558)───pstree(6094)
        ├─systemd-logind(380)
        ├─systemd-udevd(266)
        ├─upstart-file-br(450)
        ├─upstart-socket-(763)
        └─upstart-udev-br(262)
```

## 四、`top` 命令（实时动态地查看系统的整体运行情况）

* 选项：

    -d 秒数：指定top命令每隔几秒更新。默认是3秒。
    -b ：使用批处理模式输出。一般和 `-n` 配合使用。
    -n 次数：指定top命令执行的次数。一般和 `-b` 配合使用。

* 在top命令的交互模式中可以执行的命令：

    ?或h：显示交互模式的帮助
    P：以CPU使用率排序，默认就是此项
    M：以内存使用率排序
    N：以PID排序
    q：退出top

* 实例
```
top - 08:46:43 up  8:05,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  80 total,   1 running,  79 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:    501832 total,   489188 used,    12644 free,    81144 buffers
KiB Swap:   522236 total,       96 used,   522140 free.   261176 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
   25 root      20   0       0      0      0 S  0.3  0.0   0:12.49 kworker/0:1
 1557 vagrant   20   0  103744   2148   1172 S  0.3  0.4   0:06.54 sshd
 1628 root      20   0       0      0      0 S  0.3  0.0   0:02.72 kworker/u2:0
    1 root      20   0   33788   2984   1492 S  0.0  0.6   0:05.24 init
    2 root      20   0       0      0      0 S  0.0  0.0   0:00.00 kthreadd
    3 root      20   0       0      0      0 S  0.0  0.0   0:00.21 ksoftirqd/0
    5 root       0 -20       0      0      0 S  0.0  0.0   0:00.00 kworker/0:0H
    7 root      20   0       0      0      0 S  0.0  0.0   0:01.86 rcu_sched
    8 root      20   0       0      0      0 S  0.0  0.0   0:01.84 rcuos/0
    9 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcu_bh
   10 root      20   0       0      0      0 S  0.0  0.0   0:00.00 rcuob/0
... 省略n行 ...
```

### 1. 第一行信息为任务队列信息

内容|说明
-|-
08:46:43|系统当前时间
up  8:05|系统运行时间，本机已经运行了8小时5分钟
1 user|当前登录了一个用户
**load average: 0.00, 0.01, 0.05**|**系统在1分钟前，5分钟前，15分钟前的平均负载<br>一般认为小于1时，负载较小。如果大于1，一同已经超出负荷（重点）**

### 2. 第二行信息为进程信息（Tasks）

内容|说明
-|-
80 total|系统中的进程总数
1 running|正在运行的进程数
79 sleeping|休眠中的进程数
0 stopped|正在停止的进程数
**0 zombie**|僵尸进程。（**重点，如果不是0，需要手工检查僵尸进程情况，判断是否需要手动杀死进程**）

### 3. 第三行信息为CPU信息 （%Cpu(s)）

内容|说明
-|-
0.0 us|用户模式占用的CPU百分比
0.3 sy|系统模式占用的CPU百分比
0.0 ni|改变过优先级的用户进程占用的CPU百分比
**99.7 id**|空闲CPU的百分比（**重点，一般不应低于20%**）
0.0 wa|等待输入/输出的进程占用CPU百分比
0.0 hi|硬中断请求服务占用的CPU百分比
0.0 si|软中断请求服务占用的CPU百分比
0.0 st|st(Steal time) 虚拟时间百分比。就是当有虚拟机时，虚拟CPU等待实际CPU的时间百分比

### 4. 第四行信息为物理内存信息（KiB Mem）

内容|说明
-|-
501832 total|物理内存总量，单位KB
489188 used|已经使用的物理内存量
**12644 free**|**空闲的物理内存量（重点）**
81144 buffers|作为缓冲的内存量


522236 total,       96 used,   522140 free.   261176 cached Mem

### 5. 第五行信息为交换分区（swap）信息（KiB Swap）

内存|说明
-|-
522236 total|交换分区（虚拟内存）的总大小
96 used|已经使用的交换分区的大小
522140 free|空闲交换分区的大小
261176 cached Mem|作为缓存的交换分区大小

### 6. `top` 命令的增强版 - `htop` 命令

#### Linux Shell 默认并不安装 `htop` 命令，因此需要手动安装。

* Ubuntu apt-get 安装


    apt-get install -y htop

* CentOS yum 安装


    yum install -y htop

#### `htop` 命令使用

```
CPU[                                                                                           0.0%]     Tasks: 35, 30 thr; 1 running
Mem[||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||||145/490MB]     Load average: 0.00 0.04 0.05
Swp[|                                                                                       0/509MB]     Uptime: 10:20:26

PID USER      PRI  NI  VIRT   RES   SHR S CPU% MEM%   TIME+  Command
1 root       20   0 33788  2984  1492 S  0.0  0.6  0:06.11 /sbin/init
262 root       20   0 19468   652   460 S  0.0  0.1  0:00.14 upstart-udev-bridge --daemon
266 root       20   0 51672  1748  1036 S  0.0  0.3  0:00.04 /lib/systemd/systemd-udevd --daemon
343 messagebu  20   0 39400  1224   848 S  0.0  0.2  0:01.43 dbus-daemon --system --fork
380 root       20   0 43640  1816  1456 S  0.0  0.4  0:00.00 /lib/systemd/systemd-logind
383 root       20   0 23476   420   208 S  0.0  0.1  0:00.00 rpc.idmapd
396 syslog     20   0  249M  1596   992 S  0.0  0.3  0:00.02 rsyslogd
... 省略n行 ...
1079 mysql      20   0  669M 54672  7852 S  0.0 10.9  0:00.25 /usr/sbin/mysqld
1154 root       20   0 15816   952   800 S  0.0  0.2  0:00.00 /sbin/getty -8 38400 tty1
1174 mysql      20   0  669M 54672  7852 S  0.0 10.9  0:00.05 /usr/sbin/mysqld
1468 root       20   0 61556  3056  2376 S  0.0  0.6  0:00.00 /usr/sbin/sshd -D
F1Help  F2Setup F3Search F4Filter F5Tree  F6SortBy F7Nice -F8Nice +F9Kill  F10Quit
```

## 五、`jobs`, `fg` 和 `bg` 命令

### 1. `jobs` 列出后台已停止或正在运行的工作

#### 参数
> `-l`：列出PID
> `-r`：仅列出正在后台运行的工作
> `-s`：仅列出正在后台当中暂停的工作

#### 实例

```
[vagrant~] ]$top &
[1] 4468

[1]+  Stopped                 top
[vagrant~] ]$htop &
[2] 4476

[2]+  Stopped                 htop
[vagrant~] ]$jobs
[1]-  Stopped                 top
[2]+  Stopped                 htop
[vagrant~] ]$jobs -l
[1]-  4468 Stopped (signal)        top
[2]+  4476 Stopped (tty output)    htop
[vagrant~] ]$jobs -r
[vagrant~] ]$jobs -s
[1]-  Stopped                 top
[2]+  Stopped                 htop
```

### 2. `fg` 将最近放到后台的工作拿到前台来处理

### 3. `bg` 将放在后台的工作状态变为运行中