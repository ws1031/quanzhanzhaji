Linux 网络管理 - 网络环境查看命令
---


## 一、ifconfig

> ifconfig命令被用于配置和显示Linux内核中网络接口的网络参数。用ifconfig命令配置的网卡信息，在网卡重启后机器重启后，配置就不存在。要想将上述的配置信息永远的存的电脑里，那就要修改网卡的配置文件了。

### 1. 安装

若系统默认没有ifconfig命令，则使用下面命令进行安装。

    yum install net-tools

### 2. 常用参数

> `up`：启动指定的网络设备；
> `down`：关闭指定的网络设备；
> `mtu <字节>`：设置网络设备的最大传输单元；
> `netmask <子网掩码>`：设置网络设备的子网掩码；
> `broadcast <广播地址>`：设置网络设备的广播地址；

### 3. 应用

#### **显示网络设备信息（激活状态的）**

    [vagrant@10 ~]$ ifconfig
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
            inet6 fe80::e096:3a76:6df1:bd6d  prefixlen 64  scopeid 0x20<link>
            ether 08:00:27:6b:57:88  txqueuelen 1000  (Ethernet)
            RX packets 952  bytes 85854 (83.8 KiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 621  bytes 73814 (72.0 KiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
    
    lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
            inet 127.0.0.1  netmask 255.0.0.0
            inet6 ::1  prefixlen 128  scopeid 0x10<host>
            loop  txqueuelen 0  (Local Loopback)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

* 说明

    eth0 表示第一块网卡。

    lo是表示主机的回环地址，这个一般是用来测试一个网络程序，但又不想让局域网或外网的用户能够查看，只能在此台主机上运行和查看所用的网络接口。比如把 httpd服务器的指定到回环地址，在浏览器输入127.0.0.1就能看到你所架WEB网站了。但只是您能看得到，局域网的其它主机或用户无从知道。

\ | 解释
-|-
UP|网卡处在开启状态
RUNNING|网卡的网线被接上
MULTICAST|支持组播
mtu 1500|最大传输单元：1500字节
inet | 网卡的IP地址
netmask| 掩码地址
broadcast |广播地址
RX packets [xx] bytes [xx]|接收数据包数量、字节数
TX packets [xx] bytes [xx]|发送数据包数量、字节数

#### **启动关闭指定网卡**

    ifconfig eth0 up    # 启动网卡eth0
    ifconfig eth0 down  # 关闭网卡eth0

* ssh登陆linux服务器操作要小心，关闭了就会断开ssh连接，就不能开启了，除非你有多网卡。

#### **启用和关闭 ARP 协议（地址解析协议)**

    ifconfig eth0 arp    #开启网卡eth0 的arp协议
    ifconfig eth0 -arp   #关闭网卡eth0 的arp协议

#### **配置IP地址、子网掩码、广播地址**

    # 如果不加任何其他参数，则系统会依照该 IP 所在的 class 范围，自动的计算出 netmask 以及 network, broadcast 等 IP 参数；
    [root@localhost ~]# ifconfig eth0 192.168.2.10


    [root@localhost ~]# ifconfig eth0 192.168.2.10 netmask 255.255.255.0
    [root@localhost ~]# ifconfig eth0 192.168.2.10 netmask 255.255.255.0 broadcast 192.168.2.255


#### **设置最大传输单元**

    ifconfig eth0 mtu 1500    #设置能通过的最大数据包大小为 1500 bytes

#### **放弃 `ifconfig` 的全部修改，以 `ifcfg-eth*` 的配置文件重置网络设置**

    /etc/init.d/network restart

> ifconfig 所有配置修改功能都只是临时修改，重启网络服务就会失效。

## 二、ifup 和 ifdown

> 根据 /etc/sysconfig/network-scripts/ifcfg-eth* 配置文件启动和关闭网卡

### 1. 语法

* 启动网卡

    ifup [interface]

* 关闭网卡

    ifdown [interface]

### 2. 命令介绍

ifup 与 ifdown 其实都是 shell 脚本，他会直接到 `/etc/sysconfig/network-scripts` 目录下查找对应的配置文件，例如 `ifup eth0` 会读取 `ifcfg-eth0` 这个文件的内容，然后加以设置。 
不过，由于这两个脚本主要是通过读取配置文件 (ifcfg-eth*) 来启动与关闭网络接口，所以在使用前请确定 `ifcfg-eth*` 是否真的存在于正确的目录内，否则会启动失败。另外，**如果以 `ifconfig eth0 ...` 的方式 设定或修改了网路接口后，就无法再以 `ifdown eth0` 的方式来关闭了!** 因为 `ifdown` 会分析比对目前的网路参数与 `ifcfg-eth0` 是否相符，不符的话，就会放弃本次动作。因此，使用 `ifconfig` 修改完毕后，应该要用 `ifconfig eth0 down` 才能够关闭该接口。

### 3. 应用

* 当前网卡配置
```
[root@10 vagrant]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::e096:3a76:6df1:bd6d  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:6b:57:88  txqueuelen 1000  (Ethernet)
        RX packets 1870  bytes 173264 (169.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1110  bytes 143493 (140.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether 08:00:27:db:78:8f  txqueuelen 1000  (Ethernet)
        RX packets 107  bytes 12570 (12.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17  bytes 1326 (1.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
* 使用 `ifconfig eth1 down` 关闭 `eth1` 网卡
```
[root@10 vagrant]# ifconfig eth1 down
[root@10 vagrant]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::e096:3a76:6df1:bd6d  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:6b:57:88  txqueuelen 1000  (Ethernet)
        RX packets 1938  bytes 178334 (174.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1145  bytes 146715 (143.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
* 使用 `ifdown lo` 关闭 `lo` 网卡
```
[root@10 vagrant]# ifdown lo
[root@10 vagrant]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::e096:3a76:6df1:bd6d  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:6b:57:88  txqueuelen 1000  (Ethernet)
        RX packets 2018  bytes 184304 (179.9 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1186  bytes 150461 (146.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
* 使用 `ifup` 开启 `lo` 和 `eth1` 网卡
```
[root@10 vagrant]# ifup lo
[root@10 vagrant]# ifup eth1
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
[root@10 vagrant]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::e096:3a76:6df1:bd6d  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:6b:57:88  txqueuelen 1000  (Ethernet)
        RX packets 2083  bytes 189104 (184.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1221  bytes 153755 (150.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.33.88  netmask 255.255.255.0  broadcast 192.168.33.255
        inet6 fe80::a00:27ff:fedb:788f  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:db:78:8f  txqueuelen 1000  (Ethernet)
        RX packets 107  bytes 12570 (12.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 24  bytes 1884 (1.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## 三、route

> 显示并设置Linux内核中的网络路由表，route命令设置的路由主要是静态路由。
    
> 要实现两个不同的子网之间的通信，需要一台连接两个网络的路由器，或者同时位于两个网络的网关来实现。
    
> 在Linux系统中设置路由通常是为了解决以下问题：该Linux系统在一个局域网中，局域网中有一个网关，能够让机器访问Internet，那么就需要将这台机器的ip地址设置为Linux机器的默认路由。要注意的是，直接在命令行下执行route命令来添加路由，不会永久保存，当网卡重启或者机器重启之后，该路由就失效了；可以在/etc/rc.local中添加route命令来保证该路由设置永久有效。

> 在一台服务器里，连接内网的网卡是不能进行设置。

### 1. 语法
    
    route [选项] [参数]

### 2. 常用选项

> `-n`：不使用通信协议或主机名，直接显示数字形式的IP地址和端口号；
> `-net`：到一个网络的路由表；
> `-host`：到一个主机的路由表。

### 3. 常用参数

> `add`：增加指定的路由记录；
> `del`：删除指定的路由记录；
> `gw`：设置默认网关；

### 4. 应用

#### **显示当前路由列表**

    route -n

> 功能与 `netstat -rn` 命令一致

```
[root@10 vagrant]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG    102    0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     102    0        0 eth0
192.168.33.0    0.0.0.0         255.255.255.0   U     103    0        0 eth1
```

* Destination, Genmask：分别是 IP 与 子网掩码。它们组合成为一个完整的网域。

* Gateway：该网域是通过哪个 网关 连接出去的。如果显示 0.0.0.0 表示该路由是直接由本机传送，也就是可以通过局域网的 MAC 直接发送；如果有显示 IP 的话，表示该路由需要经过路由器 (网关) 的帮忙才能够传送出去。

* Flags为路由标志，标记当前网络节点的状态，Flags标志说明

Flags|说明
-|-
U| Up表示此路由当前为启动状态。
H |Host，表示此网关为一主机。
G |Gateway，表示此网关为一路由器。
R |Reinstate Route，使用动态路由重新初始化的路由。
D |Dynamically,此路由是动态性地写入。
M |Modified，此路由是由路由守护程序或导向器动态修改。
! 表示此路由当前为关闭状态。

* Iface：这个路由传送数据包的接口

#### **添加设置和删除默认网关**

    route del default gw 192.168..1
    route add default gw 192.168.0.2

#### **添加网关/设置网关**

    route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0    #增加一条到达244.0.0.0的路由。

#### **屏蔽一条路由**

    route add -net 224.0.0.0 netmask 240.0.0.0 reject     #增加一条屏蔽的路由，目的地址为224.x.x.x将被拒绝。
    
#### **删除路由记录**

    route del -net 224.0.0.0 netmask 240.0.0.0
    route del -net 224.0.0.0 netmask 240.0.0.0 reject


## 四、netstat
    
> 查询系统的状态信息。

### 1. 语法
    
    netstat [选项]
    
### 2. 常用选项

> `-t`：列出TCP协议的端口
> `-u`：列出UDP协议的端口
> `-n`：不使用域名与服务名，而使用IP地址和端口号
> `-l`：仅列出在监听状态的网络服务
> `-a`：列出所有的网络连接
> `-p`：显示正在使用Socket的程序识别码和程序名称
> `-r`：显示路由表

### 3. 应用

#### **列出所有端口 (包括监听和未监听的)**

    netstat -a     #列出所有端口
    netstat -at    #列出所有tcp端口
    netstat -au    #列出所有udp端口   

* 实例
```
[root@10 vagrant]# netstat -a
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp        0      0 10.0.2.15:ssh           10.0.2.2:surveyinst     ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
udp        0      0 0.0.0.0:bootpc          0.0.0.0:*
raw6       0      0 [::]:ipv6-icmp          [::]:*                  7
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  3      [ ]         DGRAM                    6409     /run/systemd/notify
... 省略n行 ...
unix  3      [ ]         STREAM     CONNECTED     12780
[root@10 vagrant]# netstat -at
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp        0      0 10.0.2.15:ssh           10.0.2.2:surveyinst     ESTABLISHED
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
[root@10 vagrant]# netstat -au
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp        0      0 0.0.0.0:bootpc          0.0.0.0:*
```

#### **列出所有处于监听状态的 Sockets**

    netstat -l        #只显示监听端口
    netstat -lt       #只列出所有监听 tcp 端口
    netstat -lu       #只列出所有监听 udp 端口

* 实例
```
[root@10 vagrant]# netstat -l
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
udp        0      0 0.0.0.0:bootpc          0.0.0.0:*
raw6       0      0 [::]:ipv6-icmp          [::]:*                  7
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     6422     /run/systemd/journal/stdout
unix  2      [ ACC ]     STREAM     LISTENING     22047    /var/run/NetworkManager/private-dhcp
unix  2      [ ACC ]     STREAM     LISTENING     10811    /run/lvm/lvmetad.socket
unix  2      [ ACC ]     STREAM     LISTENING     10583    /run/lvm/lvmpolld.socket
unix  2      [ ACC ]     STREAM     LISTENING     13211    /var/lib/gssproxy/default.sock
unix  2      [ ACC ]     STREAM     LISTENING     10345    /run/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     12662    /run/dbus/system_bus_socket
unix  2      [ ACC ]     STREAM     LISTENING     12665    /var/run/rpcbind.sock
unix  2      [ ACC ]     STREAM     LISTENING     13212    /run/gssproxy.sock
unix  2      [ ACC ]     SEQPACKET  LISTENING     10439    /run/udev/control
[root@10 vagrant]# netstat -lt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
[root@10 vagrant]# netstat -lu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
udp        0      0 0.0.0.0:bootpc          0.0.0.0:*
```

#### **在netstat输出中显示 PID 和进程名称**

    netstat -tulnp

* 实例
```
[root@10 vagrant]# netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1053/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      4406/sendmail: acce
tcp6       0      0 :::22                   :::*                    LISTEN      1053/sshd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           3824/dhclient
```

#### **找出程序运行的端口**

* 并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。

    netstat -anp | grep ssh
    
* 找出运行在指定端口的进程：

    netstat -anp | grep ':22'

* 实例
```
[root@10 vagrant]# netstat -anp | grep ssh
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1053/sshd
tcp        0      0 10.0.2.15:22            10.0.2.2:3212           ESTABLISHED 3888/sshd: vagrant
tcp6       0      0 :::22                   :::*                    LISTEN      1053/sshd
unix  3      [ ]         STREAM     CONNECTED     29697    3890/sshd: vagrant@
unix  3      [ ]         STREAM     CONNECTED     16196    1053/sshd
unix  3      [ ]         STREAM     CONNECTED     29698    3888/sshd: vagrant
unix  2      [ ]         DGRAM                    29694    3888/sshd: vagrant
[root@10 vagrant]# netstat -anp | grep ':22'
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1053/sshd
tcp        0      0 10.0.2.15:22            10.0.2.2:3212           ESTABLISHED 3888/sshd: vagrant
tcp6       0      0 :::22                   :::*                    LISTEN      1053/sshd
```

#### **查看处在连接状态的进程数**

    netstat -an | grep "ESTABLISHED" | wc -l

* 实例
```
[root@10 vagrant]# netstat -an | grep "ESTABLISHED"
tcp        0      0 10.0.2.15:22            10.0.2.2:3212           ESTABLISHED
[root@10 vagrant]# netstat -an | grep "ESTABLISHED" | wc -l
1
```

#### **查看某个程序的进程数**

    netstat -anop | grep "ssh" | wc -l

* 实例
```
[root@10 vagrant]# netstat -anop | grep "ssh"
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1053/sshd            off (0.00/0/0)
tcp        0      0 10.0.2.15:22            10.0.2.2:3212           ESTABLISHED 3888/sshd: vagrant   keepalive (3494.79/0/0)
tcp6       0      0 :::22                   :::*                    LISTEN      1053/sshd            off (0.00/0/0)
unix  3      [ ]         STREAM     CONNECTED     29697    3890/sshd: vagrant@
unix  3      [ ]         STREAM     CONNECTED     16196    1053/sshd
unix  3      [ ]         STREAM     CONNECTED     29698    3888/sshd: vagrant
unix  2      [ ]         DGRAM                    29694    3888/sshd: vagrant
[root@10 vagrant]# netstat -anop | grep "ssh" | wc -l
7
```

#### **显示当前路由列表**

    netstat -rn

> 功能与 `route -n` 命令一致

* 实例
```
[root@10 vagrant]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG        0 0          0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U         0 0          0 eth0
192.168.33.0    0.0.0.0         255.255.255.0   U         0 0          0 eth1
[root@10 vagrant]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG    102    0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     102    0        0 eth0
192.168.33.0    0.0.0.0         255.255.255.0   U     103    0        0 eth1
```

## 五、ss

> 显示处于活动状态的Socket信息。

> ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。
    
### 1. 语法

    ss [选项]

### 2. 常用选项

> `-t`：列出TCP协议的Socket
> `-u`：列出UDP协议的Socket
> `-n`：不使用域名与服务名，而使用IP地址和端口号
> `-l`：仅列出在监听状态的Socket
> `-a`：列出所有的Socket
> `-p`：显示正在使用Socket的进程信息

### 3. 应用

#### **列出TCP连接和UDP连接**

    ss -at    #列出tcp连接
    ss -au    #列出udp连接

* 实例
```
[root@10 vagrant]# ss -at
State      Recv-Q Send-Q    Local Address:Port                     Peer Address:Port
LISTEN     0      128                   *:ssh                                 *:*
LISTEN     0      10            127.0.0.1:smtp                                *:*
ESTAB      0      0             10.0.2.15:ssh                          10.0.2.2:surveyinst
LISTEN     0      128                  :::ssh                                :::*
[root@10 vagrant]# ss -au
State      Recv-Q Send-Q    Local Address:Port                     Peer Address:Port
UNCONN     0      0                     *:bootpc                              *:*
```

#### **列出所有处于监听状态的 Sockets**

    ss -lt       #列出监听 tcp 端口
    ss -lu       #列出监听 udp 端口

* 实例
```
[root@10 vagrant]# ss -lt
State      Recv-Q Send-Q    Local Address:Port                     Peer Address:Port
LISTEN     0      128                   *:ssh                                 *:*
LISTEN     0      10            127.0.0.1:smtp                                *:*
LISTEN     0      128                  :::ssh                                :::*
[root@10 vagrant]# ss -lu
State      Recv-Q Send-Q    Local Address:Port                     Peer Address:Port
UNCONN     0      0                     *:bootpc                              *:*
```

#### **在netstat输出中显示 PID 和进程名称**

    ss -tulnp

* 实例
```
[root@10 vagrant]# ss -tulnp
Netid  State      Recv-Q Send-Q   Local Address:Port                  Peer Address:Port
udp    UNCONN     0      0                    *:68                               *:*                   users:(("dhclient",pid=3824,fd=6))
tcp    LISTEN     0      128                  *:22                               *:*                   users:(("sshd",pid=1053,fd=3))
tcp    LISTEN     0      10           127.0.0.1:25                               *:*                   users:(("sendmail",pid=4406,fd=4))
tcp    LISTEN     0      128                 :::22                              :::*                   users:(("sshd",pid=1053,fd=4))
```

#### **找出程序运行的端口**

* 并不是所有的进程都能找到，没有权限的会不显示，使用 root 权限查看所有的信息。

    ss -anp | grep ssh
    
* 找出运行在指定端口的进程：

    ss -anp | grep ':22'

* 实例
```
[root@10 vagrant]# ss -anp | grep ssh
u_str  ESTAB      0      0         * 29697                 * 29698               users:(("sshd",pid=3890,fd=5))
u_str  ESTAB      0      0         * 16196                 * 16249               users:(("sshd",pid=1053,fd=2),("sshd",pid=1053,fd=1))
u_str  ESTAB      0      0         * 29698                 * 29697               users:(("sshd",pid=3888,fd=7))
u_dgr  UNCONN     0      0         * 29694                 * 6427                users:(("sshd",pid=3890,fd=4),("sshd",pid=3888,fd=4))
tcp    LISTEN     0      128       *:22                    *:*                   users:(("sshd",pid=1053,fd=3))
tcp    ESTAB      0      0      10.0.2.15:22                 10.0.2.2:3212                users:(("sshd",pid=3890,fd=3),("sshd",pid=3888,fd=3))
tcp    LISTEN     0      128      :::22                   :::*                   users:(("sshd",pid=1053,fd=4))
[root@10 vagrant]# ss -anp | grep ':22'
tcp    LISTEN     0      128       *:22                    *:*                   users:(("sshd",pid=1053,fd=3))
tcp    ESTAB      0      0      10.0.2.15:22                 10.0.2.2:3212                users:(("sshd",pid=3890,fd=3),("sshd",pid=3888,fd=3))
tcp    LISTEN     0      128      :::22                   :::*                   users:(("sshd",pid=1053,fd=4))
```
