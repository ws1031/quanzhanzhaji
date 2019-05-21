Linux用户登录查看命令总结 - w,who,last,lastlog
---

## 1. 查看登录用户信息

    who -H

#### 命令输出

    NAME: 用户名
    LINE: 登录终端
    TIME: 登录时间 (登录来源IP地址)

#### 实例

    [vagrant~] ]$who -H
    NAME     LINE         TIME             COMMENT
    vagrant  pts/2        2018-04-23 00:40 (10.0.2.2)

## 2. 查看登录用户的信息及他们的行为

    w [用户名]

#### 命令输出

    User：    登录的用户名 
    TTY：     登录后系统分配的终端号 
    From：    远程主机IP，即从哪个IP登录的
    login@：  登录时间 
    IDLE：    用户空闲时间。这是个计时器，一旦用户执行任何操作，改计时器就会被重置。 
    JCPU：    和终端连接的所有进程占用时间。包括当前正在运行的后台作业占用时间 
    PCPU：    当前进程所占用时间 
    WHAT：    当前正在运行进程的命令行

#### 实例

    [vagrant~] ]$w
     00:46:28 up 6 min,  1 user,  load average: 0.25, 0.27, 0.15
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    vagrant  pts/2    10.0.2.2         00:40    4.00s  0.13s  0.00s w
    [vagrant~] ]$w root
     00:46:33 up 6 min,  1 user,  load average: 0.23, 0.26, 0.15
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT

## 3. 查询当前登录和过去登录的用户信息

    last

> last 命令默认读取 `/var/log/wtmp` 文件数据

#### 命令输出

    - 用户名
    - 登录终端
    - 登录IP
    - 登录时间
    - 退出时间 (在线时间)
    
#### 实例

    [vagrant~] ]$last
    vagrant  pts/2        10.0.2.2         Mon Apr 23 00:40   still logged in
    reboot   system boot  3.13.0-24-generi Mon Apr 23 00:40 - 02:37  (01:57)
    reboot   system boot  3.13.0-24-generi Mon Apr 23 00:19 - 00:38  (00:19)
    vagrant  pts/0        10.0.2.2         Fri Apr 20 12:00 - 12:06  (00:05)
    vagrant  pts/0        10.0.2.2         Fri Apr 20 10:51 - 11:50  (00:58)
    vagrant  pts/0        10.0.2.2         Fri Apr 20 10:35 - 10:51  (00:16)
    vagrant  pts/0        10.0.2.2         Fri Apr 20 01:42 - 10:35  (08:52)
    reboot   system boot  3.13.0-24-generi Fri Apr 20 00:16 - 00:38 (3+00:21)
    vagrant  pts/0        10.0.2.2         Thu Apr 19 11:16 - 12:19  (01:02)
    reboot   system boot  3.13.0-24-generi Thu Apr 19 11:15 - 00:38 (3+13:22)
    vagrant  pts/0        10.0.2.2         Thu Apr 19 00:51 - down   (10:21)
    reboot   system boot  3.13.0-24-generi Thu Apr 19 00:42 - 11:13  (10:30)
    vagrant  pts/0        10.0.2.2         Wed Apr 18 02:45 - 12:09  (09:23)
    reboot   system boot  3.13.0-24-generi Wed Apr 18 00:25 - 11:13 (1+10:47)
    vagrant  pts/1        10.0.2.2         Tue Apr 17 02:54 - 12:29  (09:34)
    reboot   system boot  3.13.0-24-generi Tue Apr 17 02:54 - 11:13 (2+08:18)
    ...... 中间省略n行......
    vagrant  pts/0        10.0.2.2         Sun Apr  8 10:07 - 10:51  (00:43)
    
    wtmp begins Sun Apr  8 10:07:57 2018
    
## 4. 查看所有用户的最后一次登录时间

    lastlog
    
> lastlog 命令默认读取 `/var/log/lastlog` 文件数据

#### 命令输出

    - 用户名
    - 登录终端
    - 登录IP
    - 最后一次登录时间
    
#### 实例

    [vagrant~] ]$lastlog
    Username         Port     From             Latest
    root                                       **Never logged in**
    daemon                                     **Never logged in**
    bin                                        **Never logged in**
    sys                                        **Never logged in**
    sync                                       **Never logged in**
    games                                      **Never logged in**
    man                                        **Never logged in**
    lp                                         **Never logged in**
    mail                                       **Never logged in**
    news                                       **Never logged in**
    uucp                                       **Never logged in**
    proxy                                      **Never logged in**
    www-data                                   **Never logged in**
    backup                                     **Never logged in**
    list                                       **Never logged in**
    irc                                        **Never logged in**
    gnats                                      **Never logged in**
    nobody                                     **Never logged in**
    libuuid                                    **Never logged in**
    syslog                                     **Never logged in**
    messagebus                                 **Never logged in**
    sshd                                       **Never logged in**
    statd                                      **Never logged in**
    vagrant          pts/2    10.0.2.2         Mon Apr 23 00:40:38 +0000 2018
    colord                                     **Never logged in**
    mysql                                      **Never logged in**