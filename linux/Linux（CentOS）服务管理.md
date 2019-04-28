## 一、简介与分类

### 1. 系统的运行级别

#### **运行级别**

运行级别|含义
-|-
0|关机
1|单用户模式，可以近似理解为 Windows 的安全模式，主要用于系统修复
2|不完全的命令行模式，不包含NFS服务（Network File System，即网络文件系统）
3|完全命令行模式，即标准的字符界面
4|系统保留
5|图形模式
6|重启

#### **查看系统当前运行级别**

    runlevel

* 实例

```
[root~]$runlevel
N 3
```

**输出说明：**

    第一个数字是有哪个运行级别进入当前运行级别，N 表示开机直接进入当前运行级别。
    第二个数字是当前运行级别。

#### **修改系统运行级别**

    init [运行级别]

```
[root~]$init 5

[root~]$runlevel
3 5
```

#### **系统默认运行级别**

> CentOS6 以前可以通过修改 `/etc/inittab` 配置文件来修改系统默认运行级别

> CentOS7 中 `/etc/inittab` 配置文件已被弃用

```
[root~]$cat /etc/inittab
# inittab is no longer used when using systemd.
#
# ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# Ctrl-Alt-Delete is handled by /usr/lib/systemd/system/ctrl-alt-del.target
#
# systemd uses 'targets' instead of runlevels. By default, there are two main targets:
#
# multi-user.target: analogous to runlevel 3
# graphical.target: analogous to runlevel 5
#
# To view current default target, run:
# systemctl get-default
#
# To set a default target, run:
# systemctl set-default TARGET.target
```
翻译如下：
```
在使用 systemd 后 inittab 已经不再使用。

在这里添加配置不会生效。

Ctrl-Alt-Delete 由 /usr/lib/systemd/system/ctrl-alt-del.target 处理。

systemd 使用 'targets' 替代运行级别（runlevels），默认情况下，有两个主要的 targets：

multi-user.target：类似于运行级别 3 （完全命令行模式）
graphical.target: 类似于运行级别 5 （图形模式）

要查看当前的默认 target，运行：
systemctl get-default

要设置一个默认 target，运行：
systemctl set-default TARGET.target
```

* 实例

```
[root~]$systemctl get-default
multi-user.target

[root~]$systemctl set-default graphical.target
Removed symlink /etc/systemd/system/default.target.
Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/graphical.target.

[root~]$systemctl get-default
graphical.target
```


### 2. 服务的分类

#### **RPM包服务**

* 显示所有已启动的服务

    `systemctl list-units --type=service`

```
[root~]$systemctl list-units --type=service
  UNIT                               LOAD   ACTIVE SUB     DESCRIPTION
  auditd.service                     loaded active running Security Auditing Service
  crond.service                      loaded active running Command Scheduler
  dbus.service                       loaded active running D-Bus System Message Bus
  firewalld.service                  loaded active running firewalld - dynamic firewall daemon
  getty@tty1.service                 loaded active running Getty on tty1
  gssproxy.service                   loaded active running GSSAPI Proxy Daemon
  irqbalance.service                 loaded active running irqbalance daemon
● network.service                    loaded failed failed  LSB: Bring up/down networking
  NetworkManager-wait-online.service loaded active exited  Network Manager Wait Online
  NetworkManager.service             loaded active running Network Manager
...省略...
  vboxadd-service.service            loaded active running vboxadd-service.service
  vboxadd-x11.service                loaded active exited  vboxadd-x11.service
  vboxadd.service                    loaded active exited  vboxadd.service

LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.

37 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```

#### **源码包服务**

* 查询已安装的服务

    查看服务安装位置，一般在 `/usr/local/` 下

### 3. 服务与端口

#### **端口是什么**

    如果把IP地址化作一间房子，端口就是这间房子里的门。真正的房子里只有几个门，但是IP地址的端口可以有65536个。
    
    端口是传输层想应用层传递数据的门。

#### **常用端口与服务对应关系**

    /etc/services

> `/etc/services` 文件只是记录常用端口与服务对应关系，但是这个关系并不是绝对的，仅供参考。

* 查看服务对应的端口号

    `grep [服务] /etc/services`

```
[root~]$grep memcache /etc/services
memcache        11211/tcp               # Memory cache service
memcache        11211/udp               # Memory cache service
```

* 查看指定端口对应的服务

    `grep [端口号] /etc/services`

```
[root~]$grep 11211 /etc/services
memcache        11211/tcp               # Memory cache service
memcache        11211/udp               # Memory cache service

[root~]$grep ' 80/' /etc/services
http            80/tcp          www www-http    # WorldWideWeb HTTP
http            80/udp          www www-http    # HyperText Transfer Protocol
http            80/sctp                         # HyperText Transfer Protocol
```

#### **列出系统中开启的服务与对应的端口**

    netstat -tulnp

    -t：列出TCP协议的端口
    -u：列出UDP协议的端口
    -l：仅列出在监听状态的网络服务
    -n：不使用域名与服务名，而使用IP地址和端口号
    -p：显示正在使用Socket的程序识别码和程序名称

* 实例

```
[root~]$netstat -tulnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1069/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      3121/sendmail: acce
tcp6       0      0 :::22                   :::*                    LISTEN      1069/sshd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           3048/dhclient
```

## 二、RPM包服务管理

### 1. RPM 包服务的安装目录

> 仅供参考

目录|说明
-|-
`/etc/init.d/`|启动脚本所在目录
`/etc/sysconfig/`|初始化环境配置文件目录
`/etc/`|配置文件安装目录
`/var/log/`|日志文件目录
`/usr/bin/`|可执行的命令安装目录
`/usr/lib/`|程序所使用的函数库保存目录
`/usr/share/doc/`|基本的软件使用手册保存目录
`/usr/share/man/`|帮助文件保存目录

### 2. RPM 包服务启动|停止|重启|状态

#### **`systemctl [start|stop|restart|status] 服务名`**

```
[root~]$systemctl stop crond

[root~]$systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Thu 2018-05-24 04:23:44 UTC; 5s ago
  Process: 588 ExecStart=/usr/sbin/crond -n $CRONDARGS (code=exited, status=0/SUCCESS)
 Main PID: 588 (code=exited, status=0/SUCCESS)

May 24 01:03:51 localhost.localdomain systemd[1]: Started Command Scheduler.
May 24 01:03:51 localhost.localdomain systemd[1]: Starting Command Scheduler...
May 24 01:03:51 localhost.localdomain crond[588]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 12% if used.)
May 24 01:03:51 localhost.localdomain crond[588]: (CRON) INFO (running with inotify support)
May 24 04:23:44 10.0.2.15 systemd[1]: Stopping Command Scheduler...
May 24 04:23:44 10.0.2.15 systemd[1]: Stopped Command Scheduler.

[root~]$systemctl start crond

[root~]$systemctl restart crond

[root~]$systemctl status crond
● crond.service - Command Scheduler
   Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-05-24 04:24:15 UTC; 1s ago
 Main PID: 7480 (crond)
   CGroup: /system.slice/crond.service
           └─7480 /usr/sbin/crond -n

May 24 04:24:15 10.0.2.15 systemd[1]: Started Command Scheduler.
May 24 04:24:15 10.0.2.15 systemd[1]: Starting Command Scheduler...
May 24 04:24:15 10.0.2.15 crond[7480]: (CRON) INFO (RANDOM_DELAY will be scaled with factor 82% if used.)
May 24 04:24:15 10.0.2.15 crond[7480]: (CRON) INFO (running with inotify support)
May 24 04:24:15 10.0.2.15 crond[7480]: (CRON) INFO (@reboot jobs will be run at computer's startup.)
```

### 3. RPM 包服务的自启动设置

#### **`systemctl [enable|disable] 服务名`**

> 使用 `systemctl is-enabled 服务名` 可以查看服务自启动状态

```
[root~]$systemctl is-enabled crond
enabled
[root~]$systemctl disable crond
Removed symlink /etc/systemd/system/multi-user.target.wants/crond.service.
[root~]$systemctl is-enabled crond
disabled
[root~]$systemctl enable crond
Created symlink from /etc/systemd/system/multi-user.target.wants/crond.service to /usr/lib/systemd/system/crond.service.
[root~]$systemctl is-enabled crond
enabled
```

#### **修改 `/etc/rc.d/rc.local` 文件**
> 在文件中加入开机需要执行的命令即可，别忘了给文件添加执行权限。
```
[root~]$cat /etc/rc.d/rc.local
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local
```

## 三、源码包服务管理

### 1. 源码包安装服务的目录

    一般在 /usr/local/ 下，仅供参考

### 2. 源码包安装服务的启动

> 使用绝对路径，调用启动脚本来启动。
> 不同的源码包的启动脚本不同。
> 可以查看源码包的安装说明，查看启动脚本的方法。

* 例：源码包 nginx 的启动、停止、重启 
```
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/nginx -s stop
/usr/local/nginx/sbin/nginx -s reload
```

### 3. 服务的自启动设置

#### **修改 `/etc/rc.d/rc.local` 文件**
> 在文件中加入开机需要执行的命令即可，别忘了给文件添加执行权限。
```
[root~]$cat /etc/rc.d/rc.local
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local
# 开机启动nginx
/usr/local/nginx/sbin/nginx
```

## 四、常见服务功能简介

服务名称|	功能简介|	建议
-|-|-
acpid|电源管理接口。<br>如果是笔记本用户建议开启，可以监听内核层的相关电源事件|	开启
anacron|系统的定时任务程序。<br>cron 的一个子系统，如果定时任务错过了执行时间，可以通过 anacron 继续唤醒执行|关闭
alsasound|Alsa 声卡驱动。<br>如果使用 alsa 声卡，开启|关闭
apmd|电源管理模块。<br>如果支持 acpid，就不需要 apmd，可以关闭	|关闭
atd|指定系统在特定时间执行某个任务，只能执行一次。<br>如果需要则开启，但我们一般使用 crond 来进行循环定时任务|关闭
auditd|审核子系统。<br>如果开启了此服务，SELinux 的审核信息会写入/var/log/audit/audit.log 文件，如果不开启，审核信息会记录在 syslog 中|开启
autofs|	让服务器可以自动挂载网络中的其他服务器的共享数据，一般用来自动挂载 NFS 服务。<br>如果没有 NFS 服务建议关闭|关闭
avahi-daemon|Avahi是 zeroconf 协议的实现。<br>它可以在没有 DNS服务的局域网里发现基于 zeroconf  协议的设备和服务。<br>除非有兼容设备或使用 zeroconf 协议，否则关闭|关闭
bluetooth|	蓝牙设备支持。<br>一般不会在服务器上启用蓝牙设备，关闭它|关闭
capi|仅对使用 ISND 设备的用户有用|关闭
chargen-dgram|使用 UDP 协议的 chargen  server。<br>主要功能是提供类似远程打字的功能|关闭
chargen-stream|同上|关闭
cpuspeed|可以用来调整 CPU 的频率。<br>当闲置时可以自动降低 CPU 频率来节省电量|开启
crond|系统的定时任务。<br>一般的 Linux 服务器都需要定时任务帮助系统维护。建议开启|开启
cvs|一个版本控制系统|关闭
daytime-dgram|	daytime 使用 TCP 协议的 Daytime 守护进程。<br>该协议为客户机实现从远程服务器获取日期和时间的功能|关闭		
daytime-stream|	同上|关闭
dovecot|邮件服务中 POP3/IMAP 服务的守护进程。<br>主要用来接收信件，如果启动了邮件服务则开启，否则关闭|关闭
echo-dgram|服务器回显客户服务的进程|关闭
echo-stream|同上|关闭
firstboot|系统安装完成之后，有个欢迎界面，需要对系统进程初始设定，就是这个进程的作用。<br>既然不是第一次启动了，关闭吧|关闭
gpm|在字符终端（tty1-tty6）中可以使用鼠标复制和粘贴，就是这个服务的功能|开启
haldaemon|检测盒支持 USB 设备。<br>如果是服务器可以关闭，个人机建议开启|关闭
hidd|蓝牙鼠标、键盘等蓝牙设备检测。<br>必须启动 bluetooth 服务|关闭
hplip|HP 打印机支持<br>如果没有 HP 打印机关闭吧|关闭
httpd|apache 服务的守护进程<br>如果需要启动 apache，就开启|开启
ip6tables|IPv6 的防火墙<br>目前 IPv6 协议并没有使用，可以关闭|关闭
iptables|防火墙功能<br>Linux 中防火墙是内核支持功能。这是服务器的主要防护手段，必须开启。|开启
irda|IrDA 提供红外线设备（笔记本，PDA’s，手机，计算器等等）间的通讯支持。关闭吧|关闭
irqbalance|	支持多核处理器，让 CPU 可以自动分配系统中断（IRQ），提高系统性能<br>目前服务器多是多核 CPU，请开启|开启	
isdn|使用 ISDN 设备连接网络<br>目前主流的联网方式是光纤接入和ADSL，ISDN 已经非常少见，请关闭|关闭
kudzu|该服务可以在开机时进行硬件检测，并会调用相关的设置软件。<br>建议关闭，仅在需要时开启|关闭	
lvm2-monitor|该服务可以让系统支持 LVM 逻辑卷组<br>如果分区采用的是 LVM方式，那么应该开启。建议开启|开启
mcstrans|SELinux 的支持服务。建议启动|开启	
mdmonitor|该服务用来监测 Software RAID 或 LVM 的信息。<br>不是必须服务，建议关闭|关闭	
mdmpd|该服务用来监测 Multi-Path 设备。<br>不是必须服务|关闭
messagebus|这是 Linux 的 IPC（Interprocess Communication，进程间通讯）服务，用来在各个软件中交换信息。<br>个人建议关闭|关闭
microcode_ctl|Intel 系列的 CPU 可以通过这个服务支持额外的微指令集|关闭	
mysqld|mysql 数据库服务器。<br>如果需要就开启，否则关闭|开启
named|DNS 服务的守护进程，用来进行域名解析。<br>如果是 DNS 服务器则开启，否则关闭|关闭
netfs|该服务用于在系统启动时自动挂载网络中的共享文件空间<br>比如：NFS，Samba 等等。需要就开启，否则关闭|关闭
network|提供网络设置功能。<br>通过这个服务来管理网络，所以开启|开启
nfs|NFS（Network File System）服务，Linux 与 Linux 之间的文件共享服务。<br>需要就开启，否则关闭|关闭
nfslock|在 Linux 中如果使用了 NFS 服务，为了避免同一个文件被不同的用户同时编辑，所有有这个锁服务。<br>有 NFS 是开启，否则关闭|关闭
ntpd|该服务可以通过互联网自动更新系统时间，使系统时间永远都准确。<br>需要则开启，但不是必须服务|关闭
pcscd|智能卡检测服务，可以关闭|关闭
portmap|用在远程过程调用（RPC）的服务，如果没有任何 RPC 服务时，可以关闭。<br>主要是 NFS 和 NIS 服务需要|关闭
psacct|该守护进程支持几个监控进程活动的工具|关闭
rdisc|客户端 ICMP 路由协议|关闭
readahead_early|在系统开机的时候，先将某些进程加载如内存整理，可以加快一点启动速度|关闭
readahead_later|同上|关闭
restorecond|用于给 SELinux 监测和重新加载正确的文件上下文。<br>如果开启 SELinux 则需要开启|关闭
rpcgssd|与 NFS 有关的客户端功能。<br>如果没有 NFS 就关闭吧|关闭
rpcidmapd|同上|关闭
rsync|远程数据备份守护进程|关闭	
sendmail|sendmail 邮件服务的守护进程。<br>如果有邮件服务就开启，否则关闭|关闭	
setroubleshoot|	该服务用于将 SELinux 相关信息记录在日志/var/log/messages 中。<br>建议开启|开启
smartd|该服务用于自动检测硬盘状态。<br>建议开启|开启
smb|网络服务 samba 的守护进程。<br>可以让 Linux 和 Windows 之间共享数据。如果需要则开启|关闭
squid|代理服务的守护进程。<br>如果需要则开启，否则关闭|关闭
sshd|ssh 加密远程登陆管理的服务。<br>服务器的远程管理必须使用此服务，不要关闭|开启	
syslog|	日志的守护进程|开启
vsftpd|	vsftp 服务的守护进程。<br>如果需要 FTP 服务则开启，否则关闭|关闭	
xfs|这个是 X Window 的字体守护进程。<br>为图形界面提供字体服务，如果不启动图形界面，就不用开启|关闭
xinetd|	超级守护进程。<br>如果有依赖 xinetd 的服务就必须开启|开启	
ypbind|	为 NIS（网络信息系统）客户机激活 ypbind 服务进程|关闭	
yum-updatesd|yum 的在线升级服务|关闭