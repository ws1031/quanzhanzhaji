#### 学习 sudo 权限前，先了解一下当用户第一次使用sudo权限时CentOS的系统提示

> 我们信任您已经从系统管理员那里了解了日常注意事项。
> 总结起来无外乎这三点：
>    #1) 尊重别人的隐私。
>    #2) 输入前要先考虑(后果和风险)。
>    #3) 权力越大，责任越大。

---

> sudo权限的作用是：使普通用户可以临时以 root 用户的身份和权限执行系统命令

> sudo权限的操作对象是系统命令


## 一、sudo权限的配置

### 1. 编辑 sudo权限 命令

    visudo

> visudo 命令实际修改的是 `/etc/sudoers` 文件

### 2. `/etc/sudoers` 配置文件

```
[root/etc]# cat /etc/sudoers

... 省略部分内容 ...

## Allow root to run any commands anywhere
## 允许root在任何地方运行任何命令
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software, service management apps and more.
## 允许“sys”用户组的成员运行 网络、软件、服务管理应用等命令。
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
## 允许“wheel”用户组的成员运行所有命令
%wheel  ALL=(ALL)       ALL

## Allows people in group wheel to run all commands without a password
## 允许“wheel”用户组的成员运行所有命令，且运行时不需要输入密码
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## Allows members of the users group to mount and unmount the cdrom as root
## 允许“users”组的成员运行 挂载、卸载光盘的命令
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
## 允许“users”组的成员在本机运行 /sbin/shutdown -h now 命令
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
## 读取 /etc/sudoers.d 目录下所有文件的内容作为配嵌入到此配置文件
## 注意，下面的 # 后面并不是注释
#includedir /etc/sudoers.d
```

#### **为用户配置sudo权限**

    [用户名]    [被管理主机的IP]=([可以使用的身份])   [NOPASSWD: ][授权的命令]

> `[被管理主机的IP]`、`[可以使用的身份]`、`[授权的命令]` 都可以使用 `ALL` 来表示不限制。
> 添加 `[NOPASSWD: ]` 选项可以使用户在使用sudo权限时不需要输入密码。
> `[授权的命令]`要使用绝对路径，多条命令之间可用逗号（`,`）分隔。


* 例：
```
## Allow root to run any commands anywhere
## 允许root在任何地方运行任何命令
root    ALL=(ALL)       ALL
```

#### **为用户组配置sudo权限**

    %[组名]    [被管理主机的IP]=([可以使用的身份])   [NOPASSWD: ][授权的命令]

> `[被管理主机的IP]`、`[可以使用的身份]`、`[授权的命令]` 都可以使用 `ALL` 来表示不限制。
> 添加 `[NOPASSWD: ]` 选项可以使用户在使用sudo权限时不需要输入密码。
> 用户组 与 用户 的唯一区别是用户组前有个 `%`

* 例：
```
## 允许“wheel”用户组的成员运行所有命令，且运行时不需要输入密码
%wheel        ALL=(ALL)       NOPASSWD: ALL
```

### 3. 注意事项

#### **1) 赋予用户sudo权限时一定要谨慎，够用即可，不要赋予过高的权限**
#### **2) `[授权的命令]` 设置得越具体，用户获得的权限越小。**
#### **3) 严禁赋予普通用户 `/usr/bin/passwd`、`/usr/bin/vi`、`/usr/bin/su`、`/usr/bin/bash` 命令的权限，拥此权限的用户可以修改root用户密码，然后为所欲为。**
#### **4) 权力越大，责任越大。**

## 二、sudo 命令介绍

### 1. `sudo [命令]`：以 root 身份来执行命令

> 用户必须有相应命令的sudo权限

#### **例子**

* 普通用户使用 less 命令查看 root 用户的历史命令
```
[vagrant~]$ sudo less /root/.bash_history
cat report.md | grep -v ID | awk '$4 >= 99 {print $2}'
cat report.md | grep -v ID | awk '$4 <= 99 {print $2}'
cat report.md | grep -v ID | awk '$4 == 100 {print $2}'
sed -n '2p' report.md
sed -n '2,4p' report.md
sed '2,4d' report.md
cat -n report.md
sed '1a Begin' report.md
sed '1i Begin' report.md
sed '1a Begin' report.md
sed '1a End' report.md
sed '1c Hello World' report.md
sed '5c Hello World' report.md
... 省略 ...
```

### 2. `sudo su`：切换到root用户

> 用户必须有`/usr/bin/su`命令的sudo权限
> 一旦切换成功，用户可以以root身份执行任何命令

#### **例子**

* 普通用户使用 sudo su 命令切换到 root 用户，然后修改root用户的密码
```
[vagrant/tmp]$ sudo su

[root/tmp]# passwd
更改用户 root 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```

### 3. `sudo -s <shell>`：切换到root用户的shell

> 可以不加 `<shell>`，会使用默认 shell
> 用户必须有相应shell命令的sudo权限，例如 `/usr/bin/bash`
> 一旦切换成功，用户可以以root身份执行任何命令

#### **例子**

* 普通用户使用 `sudo -s /usr/bin/bash` 命令切换到 root 的shell，然后修改root用户的密码
```
[vagrant/tmp]$ sudo -s /usr/bin/bash

[root/tmp]# passwd
更改用户 root 的密码 。
新的 密码：
无效的密码： 密码少于 8 个字符
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```

* 可以不加 `<shell>`，会使用默认 shell
```
[vagrant/tmp]$ sudo -s

[root/tmp]# exit
```

### 4. `sudo -l`：列出目前用户可用的sudo权限的指令

#### **例子**
```
[vagrant~]$ sudo -l
...省略部分内容...
用户 vagrant 可以在本机上运行以下命令：
    (ALL) /usr/bin/bash, /usr/bin/su, /usr/bin/less
```

## 三、sudo权限的应用

### 1. 授权普通用户可以重启服务器

* 执行 `visudo`，然后添加如下内容
```
user1    ALL=(ALL)       /sbin/shutdown -r now
```

* 切换到user1账号，查看user1可用的sudo权限的指令
```
[user1@10 ~]$ sudo -l
...省略部分内容...
用户 user1 可以在 10 上运行以下命令：
    (ALL) /sbin/shutdown -r now
```

### 2. 授权普通用户可以添加其他用户

#### **功能分析**

要想添加其他用户，必须拥有添加用户和设置密码的权限，即 `/usr/sbin/useradd` 和 `/usr/bin/passwd` 两个命令的sudo权限

若用户完全拥有 `/usr/bin/passwd` 的sudo权限，则可以通过 `sudo passwd` 命令或者 `sudo passwd root` 命令修改 root 密码，这样就会变得非常不安全。

因此，需要严格限制用户对 `/usr/bin/passwd` 的权限：
```
user ALL=(ALL)    /usr/bin/passwd [A-Za-z]*, !/usr/bin/passwd "", !/usr/bin/passwd root
```

`/usr/bin/passwd [A-Za-z]*` 表示 passwd 命令后附加的第一个字符只能是大小写字母。  
`!/usr/bin/passwd ""` 表示 passwd 命令后不能什么都不加。  
`!/usr/bin/passwd root` 表示 passwd 命令后不能加 root。

三条语句的缺一不可，且顺序不能颠倒。

#### **实例**

* 执行 `visudo`，然后添加如下内容
```
user1   ALL=(ALL)    /usr/sbin/useradd
user1   ALL=(ALL)    /usr/bin/passwd [A-Za-z]*, !/usr/bin/passwd "", !/usr/bin/passwd root
```

* 切换到user1账号，查看user1可用的sudo权限的指令
```
[user1@10 ~]$ sudo -l
...省略部分内容...
用户 user1 可以在 10 上运行以下命令：
    (ALL) /usr/sbin/useradd
    (ALL) /usr/bin/passwd [A-Za-z]*, !/usr/bin/passwd \"\", !/usr/bin/passwd root
```

* 添加用户user2，并为其设置密码
```
[user1@10 ~]$ sudo useradd user2

[user1@10 ~]$ sudo passwd user2
更改用户 user2 的密码 。
新的 密码：
无效的密码： 密码是一个回文
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。

[user1@10 ~]$ grep user2 /etc/passwd
user2:x:1006:1007::/home/user2:/bin/bash
```

* 尝试以 `passwd` 和 `passwd root` 两种方式修改 root 用户的密码，全部失败
```
[user1@10 ~]$ sudo passwd
对不起，用户 user1 无权以 root 的身份在 10.0.2.15 上执行 /bin/passwd。

[user1@10 ~]$ sudo passwd root
对不起，用户 user1 无权以 root 的身份在 10.0.2.15 上执行 /bin/passwd root。
```

* 添加用户2_user，尝试为其设置密码，失败。因为 `/usr/bin/passwd [A-Za-z]*` 决定 passwd 命令后附加的第一个字符只能是大小写字母。
```
[user1@10 ~]$ sudo useradd 2_user

[user1@10 ~]$ sudo passwd 2_user
对不起，用户 user1 无权以 root 的身份在 10.0.2.15 上执行 /bin/passwd 2_user。
```