Linux系统资源查看：vmstat,dmesg,free,uptime,uname,lsb_release,lsof
---

## 一、`vmstat` 命令（监控系统资源）**【常用】**

### 1. 命令格式

    vmstat [刷新延时 刷新次数]

### 2. 输出说明

#### **procs：进程信息字段**

> `r`：等待运行的进程数。
> `b`：不可被唤醒的进程数。

> 这两个数量越大，表示系统越繁忙

#### **memory：内存信息字段**

> `swpd`：虚拟内存的使用情况，单位KB。
> `free`：空闲的内存容量，单位KB。
> `buff`：缓冲的内存容量，单位KB。
> `cache`：缓存的内存容量，单位KB。

> 缓冲（buffer）和缓冲（cache）的区别：
简单来说，缓存（cache）是用来加速数据从硬盘中“读取”的，而缓冲（buffer）是用来加速数据“写入”硬盘的。

#### **swap：交换分区的信息字段**

> `si`：从磁盘中交换到内存中数据的数量，单位KB。
> `so`：从内存中交换到磁盘中数据的数量，单位KB。

> 这两个数据越大，说明数据需要经常在磁盘和内存之间交换，系统性能越差。

#### **io：磁盘读写信息字段**

> `bi`：由磁盘写入的块数量，单位是块。
> `bo`：写入到磁盘中去的块数量，单位是块（现在的Linux版本块的大小为1kb）。

> 这两个数据越大，说明系统的I/O越繁忙。


#### **system：系统信息字段**

> `in`：每秒被中断的进程次数。
> `cs`：每秒进行的时间切换次数。

> 这两个数据越大，说明系统与接口设备的通信非常频繁。这些接口设备包含磁盘、网卡、时钟等。

#### **cpu：CPU信息字段**

> `us`：非内核进程消耗CPU运算时间的百分比。
> `sy`：内核进程消耗CPU运算时间的百分比。
> `id`：空闲CPU的百分比。
> `wa`：等待I/O所消耗的CPU百分比。
> `st`：被虚拟机所盗用的CPU百分比数。

### 3. 实例

```
[vagrant~] ]$vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 259312  28320 108272    0    0    30     4   41  202  0  0 99  0  0
 0  0      0 259304  28320 108272    0    0     0     0   50   93  0  1 99  0  0
 0  0      0 259300  28320 108272    0    0     0     0   33   73  0  0 100  0  0
```

## 二、`dmesg` 命令（开机时内核检查信息）**【常用】**

### 1. 输出所有内核开机时的信息
#### **由于输出数据较多，可试用 less 命令查看：`dmesg | less`**

```
[vagrant~] ]$dmesg | less
[    0.000000] Initializing cgroup subsys cpuset
[    0.000000] Initializing cgroup subsys cpu
[    0.000000] Initializing cgroup subsys cpuacct
[    0.000000] Linux version 3.13.0-24-generic (buildd@panlong) (gcc version 4.8.2 (Ubuntu 4.8.2-19ubuntu1) ) #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014 (Ubuntu 3.13.0-24.46-generic 3.13.9)
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-3.13.0-24-generic root=UUID=64ecdf77-db7b-48a8-9066-abfb837f2e24 ro
...省略n行...
[    0.000000] Zone ranges:
[    0.000000]   DMA      [mem 0x00001000-0x00ffffff]
[    0.000000]   DMA32    [mem 0x01000000-0xffffffff]
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
:
```

### 2. 查找包含指定内容的信息

#### **查找开机时CPU的相关信息：`dmesg | grep CPU`**
```
[vagrant~] ]$dmesg | grep CPU
[    0.000000] CPU MTRRs all blank - virtualized system.
[    0.000000] ACPI: SSDT 000000001fff02a0 0001CC (v01 VBOX   VBOXCPUT 00000002 INTL 20100528)
[    0.000000] smpboot: Allowing 1 CPUs, 0 hotplug CPUs
[    0.000000] setup_percpu: NR_CPUS:256 nr_cpumask_bits:256 nr_cpu_ids:1 nr_node_ids:1
[    0.000000] PERCPU: Embedded 29 pages/cpu @ffff88001fc00000 s86336 r8192 d24256 u2097152
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000]  RCU restricting CPUs from NR_CPUS=256 to nr_cpu_ids=1.
[    0.000000]  Offload RCU callbacks from all CPUs
[    0.000000]  Offload RCU callbacks from CPUs: 0.
[    0.004582] mce: CPU supports 0 MCE banks
[    0.078327] smpboot: CPU0: AMD A10-7870K Radeon R7, 12 Compute Cores 4C+8G (fam: 15, model: 38, stepping: 01)
[    0.083763] x86: Booted up 1 node, 1 CPUs
[    0.844045] microcode: CPU0: patch_level=0x06000626
[    0.908285] ledtrig-cpu: registered to indicate activity on CPUs
[    3.624181] CPU: 0 PID: 293 Comm: systemd-udevd Tainted: GF          O 3.13.0-24-generic #46-Ubuntu
```

#### **查找开机时硬盘的相关信息：`dmesg | grep -i hd`**
```
[vagrant~] ]$dmesg | grep -i hd
[    0.084000] NMI watchdog: disabled (cpu0): hardware events not enabled
```

## 三、`free` 命令（查看内存使用状态）**【常用】**

    free [选项]

### 1. 选项

> `-b`：以字节为单位显示
> `-k`：以KB为单位显示（默认）
> `-m`：以MB为单位显示
> `-g`：以GB为单位显示

### 2. 输出说明

- |total|used|free|shared|buffers|cached
-|-|-|-|-
Mem|总内存数|已使用内存数|空闲内存数|多个进程共享内存数|缓冲内存数|缓存内存数
Swap|swap总数|已经使用的swap数|空闲的swap数


#### **`-/+ buffers/cache`**

> `-buffers/cache` = `used` - `buffers` - `cached`
> `+buffers/cache` = `free` + `buffers` + `cached`

### 3. 实例
```
[vagrant~] ]$free
             total       used       free     shared    buffers     cached
Mem:        501832     248436     253396       2372      33216     109116
-/+ buffers/cache:     106104     395728
Swap:       522236          0     522236
[vagrant~] ]$free -b
             total       used       free     shared    buffers     cached
Mem:     513875968  254517248  259358720    2428928   34029568  111755264
-/+ buffers/cache:  108732416  405143552
Swap:    534769664          0  534769664
[vagrant~] ]$free -k
             total       used       free     shared    buffers     cached
Mem:        501832     248576     253256       2372      33232     109136
-/+ buffers/cache:     106208     395624
Swap:       522236          0     522236
[vagrant~] ]$free -m
             total       used       free     shared    buffers     cached
Mem:           490        242        247          2         32        106
-/+ buffers/cache:        103        386
Swap:          509          0        509
[vagrant~] ]$free -g
             total       used       free     shared    buffers     cached
Mem:             0          0          0          0          0          0
-/+ buffers/cache:          0          0
Swap:            0          0          0
```

## 四、查看CPU信息

    cat /proc/cpuinfo

#### **实例**
```
[vagrant~] ]$cat /proc/cpuinfo
processor       : 0
vendor_id       : AuthenticAMD
cpu family      : 21
model           : 56
model name      : AMD A10-7870K Radeon R7, 12 Compute Cores 4C+8G
stepping        : 1
microcode       : 0x6000626
cpu MHz         : 3892.536
cache size      : 2048 KB
physical id     : 0
siblings        : 1
core id         : 0
cpu cores       : 1
apicid          : 0
initial apicid  : 0
fpu             : yes
fpu_exception   : yes
cpuid level     : 13
wp              : yes
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx mmxext fxsr_opt rdtscp lm rep_good nopl extd_apicid pni pclmulqdq monitor ssse3 cx16
 sse4_1 sse4_2 popcnt aes xsave avx lahf_lm cr8_legacy abm sse4a misalignsse 3dnowprefetch arat fsgsbase
bogomips        : 7785.07
TLB size        : 1536 4K pages
clflush size    : 64
cache_alignment : 64
address sizes   : 48 bits physical, 48 bits virtual
power management:
```

## 五、`uptime` 命令（显示系统的启动时间和平均负载）

> 输出结果与`top`命令和`w`的第一行相同。

#### **实例**
```
[vagrant~] ]$uptime
 02:35:11 up  2:43,  1 user,  load average: 0.00, 0.06, 0.07
[vagrant~] ]$w
 02:35:13 up  2:43,  1 user,  load average: 0.00, 0.06, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
vagrant  pts/0    10.0.2.2         23:52    1.00s  0.49s  0.00s w
[vagrant~] ]$top

top - 02:36:52 up  2:45,  1 user,  load average: 0.00, 0.04, 0.06
Tasks:  80 total,   1 running,  79 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.3 sy,  0.0 ni, 99.5 id,  0.1 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:    501832 total,   251856 used,   249976 free,    35560 buffers
KiB Swap:   522236 total,        0 used,   522236 free.   109596 cached Mem
...省略n行...
```

## 六、查看系统与内核相关信息

### 1. `uname` 命令
    
    uname [选项]

#### **选项**

> `-a`：查看系统所有相关信息
> `-r`：查看内核版本
> `-s`：查看内核名称
> `-m`：查看系统的硬件名称，例如 i686 或 x86_64 等
> `-p`：查看CPU的类型，与 -m 类似，只是显示的是CPU的类型
> `-i`：查看硬件的平台（ix86）

#### **实例**

```
[vagrant~] ]$uname
Linux
[vagrant~] ]$uname -a
Linux vagrant-ubuntu-trusty 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
[vagrant~] ]$uname -s
Linux
[vagrant~] ]$uname -r
3.13.0-24-generic
[vagrant~] ]$uname -m
x86_64
[vagrant~] ]$uname -p
x86_64
[vagrant~] ]$uname -i
x86_64
```

### 2. 判断当前系统的位数

    file /bin/ls

#### **实例**
```
[vagrant~] ]$file /bin/ls
/bin/ls: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=64d095bc6589dd4bfbf1c6d62ae985385965461b, stripped
```

### 3. 查看当前Linux系统的发行版本

    lsb_release -a

#### **实例**
```
[vagrant~] ]$lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 14.04 LTS
Release:        14.04
Codename:       trusty
```

## 七、`lsof` 命令（列出进程打开或使用的文件信息）**【常用】**

    lsof [选项]

#### **选项**

> `-c` 字符串：只列出以字符串开头的进程打开的文件
> `-u` 用户名：只列出某个用户的进程打开的文件
> `-p` pid：只列出某个PID进程打开的文件

### 1. 查询系统中所有进程调用的文件

    lsof | less

#### **实例**

```
[vagrant~] ]$sudo lsof | less
COMMAND    PID  TID       USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
init         1            root  cwd       DIR                8,1     4096          2 /
init         1            root  rtd       DIR                8,1     4096          2 /
init         1            root  txt       REG                8,1   265848    1835051 /sbin/init
init         1            root  mem       REG                8,1    47712    1053013 /lib/x86_64-linux-gnu/libnss_files-2.19.so
init         1            root  mem       REG                8,1    47760    1053033 
...省略n行...
khelper     13            root  cwd       DIR                8,1     4096          2 /
khelper     13            root  rtd       DIR                8,1     4096          2 /
khelper     13            root  txt   unknown                                        /proc/13/exe
:
```

### 2. 查询某个文件被哪个进程调用

    lsof 文件名

#### **实例**

```
[vagrant~] ]$sudo lsof /sbin/init
COMMAND PID USER  FD   TYPE DEVICE SIZE/OFF    NODE NAME
init      1 root txt    REG    8,1   265848 1835051 /sbin/init
```

### 3. 查询某个进程调用了哪些文件

    lsof -c 进程名

#### **实例**

```
[vagrant~] ]$sudo lsof -c nginx
COMMAND  PID     USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
nginx   1000     root  cwd    DIR                8,1     4096       2 /
nginx   1000     root  rtd    DIR                8,1     4096       2 /
nginx   1000     root  txt    REG                8,1   873176 1573477 /usr/sbin/nginx
nginx   1000     root  mem    REG                8,1    47712 1053013 
...省略n行...
nginx   1004 www-data   14u  unix 0xffff88001b776000      0t0    9508 socket
nginx   1004 www-data   15u  0000                0,9        0    5249 anon_inode
```

### 4. 查询某个用户的进程调用了哪些文件

    lsof -u 用户名

#### **实例**

```
[vagrant~] ]$sudo lsof -u vagrant
COMMAND  PID    USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
sshd    1516 vagrant  cwd    DIR                8,1     4096       2 /
sshd    1516 vagrant  rtd    DIR                8,1     4096       2 /
sshd    1516 vagrant  txt    REG                8,1   766784 1583210 /usr/sbin/sshd
sshd    1516 vagrant  DEL    REG                0,4            11386 /dev/zero
sshd    1516 vagrant  mem    REG                8,1    14464 1048876 /lib/x86_64-linux-gnu/security/pam_env.so
...省略n行...
bash    1517 vagrant    0u   CHR              136,0      0t0       3 /dev/pts/0
bash    1517 vagrant    1u   CHR              136,0      0t0       3 /dev/pts/0
bash    1517 vagrant    2u   CHR              136,0      0t0       3 /dev/pts/0
bash    1517 vagrant  255u   CHR              136,0      0t0       3 /dev/pts/0
```

### 5. 查询某个PID的进程调用了哪些文件

    lsof -p PID

#### **实例**

```
[vagrant~] ]$sudo lsof -p 1003
COMMAND  PID     USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
nginx   1003 www-data  cwd    DIR                8,1     4096       2 /
nginx   1003 www-data  rtd    DIR                8,1     4096       2 /
nginx   1003 www-data  txt    REG                8,1   873176 1573477 /usr/sbin/nginx
nginx   1003 www-data  mem    REG                8,1    47712 1053013 /lib/x86_64-linux-gnu/libnss_files-2.19.so
...省略n行...
nginx   1003 www-data    6u  IPv4               9497      0t0     TCP *:http (LISTEN)
nginx   1003 www-data    7u  IPv6               9498      0t0     TCP *:http (LISTEN)
nginx   1003 www-data   13u  0000                0,9        0    5249 anon_inode
```