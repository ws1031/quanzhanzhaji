> SUID权限、SGID权限、SBIT权限 都属于极其不安全的权限，这里只是作为了解学习，在生产环境尽量不去设置这些权限！

## 一、Set UID权限（SUID权限）

### 1. Set UID权限的限制与功能

* 只有可执行的二进制程序才能设定SUID权限（**对普通shell脚本无效**）
* 命令执行者对该程序拥有执行（`x`）权限
* 命令执行者在执行该程序时获得该程序文件属主的身份（在执行程序的过程中灵魂附体为文件属主）
* SUID权限只在该程序执行的过程中有效

### 2. 设定Set UID权限的方法

    chmod 4755 文件名
    
    chmod +x,u+s 文件名

#### **TIPS**

> 1. 当初的umask值是0022，第一位是指特殊权限位，而在设定权限的时候常输入`755` `644`，此时是将特殊权限省略了。
> 2. 在文件普通权限前加 `4` ，代表SUID权限。
> 3. 当文件拥有Set UID权限后，文件所有者对应的权限会变为 `rws`。
> 4. 要设定Set UID权限，所有者必须拥有该文件的执行（`x`）权限，否则文件所有者对应的权限会变为 `rwS`，这是无效的权限。

#### **实例**
```
[root/tmp/suid]# touch 4755.sh
[root/tmp/suid]# touch u+s.sh
[root/tmp/suid]# ll
总用量 4.0K
-rw-r--r-- 1 root root  0 6月   8 09:22 4755.sh
-rw-r--r-- 1 root root  0 6月   8 09:22 u+s.sh

[root/tmp/suid]# chmod 4755 4755.sh
[root/tmp/suid]# chmod +x,u+s u+s.sh
[root/tmp/suid]# ll
总用量 4.0K
-rwsr-xr-x 1 root root  0 6月   8 09:22 4755.sh*
-rwsr-xr-x 1 root root  0 6月   8 09:22 u+s.sh*
```

### 3. 取消Set UID权限的方法

    chmod 0755 文件名
    
    chmod u-s 文件名

#### **实例**
```
[root/tmp/suid]# ll
总用量 4.0K
-rwsr-xr-x 1 root root  0 6月   8 09:22 4755.sh*
-rwsr-xr-x 1 root root  0 6月   8 09:22 u+s.sh*

[root/tmp/suid]# chmod 0755 4755.sh

[root/tmp/suid]# chmod u-s u+s.sh

[root/tmp/suid]# ll
总用量 4.0K
-rwxr-xr-x 1 root root  0 6月   8 09:22 4755.sh*
-rwxr-xr-x 1 root root  0 6月   8 09:22 u+s.sh*
```

### 4. 应用SUID权限的系统命令

#### **passwd 命令**

* passwd 命令实际上修改的是 `/etc/shadow` 文件，而该文件的权限是 `000`。理论上来说，应该只有root用户才有权限修改该文件。

```
[root/tmp/suid]# ll /etc/shadow
---------- 1 root root 829 6月   8 02:21 /etc/shadow
```

* passwd 拥有SUID权限，普通用户在执行 `passwd` 命令时，身份自动切换成root，所以普通用户可以修改自己的密码。
```
[root/tmp/suid]# ll /usr/bin/passwd
-rwsr-xr-x. 1 root root 28K 6月  10 2014 /usr/bin/passwd*
```

#### **其他常见的拥有SUID权限的命令（文件）**

    find / -perm -4000

```
/usr/bin/mount
/usr/bin/su
/usr/bin/chsh
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/umount
/usr/bin/pkexec
/usr/bin/crontab
/usr/bin/sudo
/usr/bin/passwd
/usr/sbin/pam_timestamp_check
/usr/sbin/unix_chkpwd
/usr/sbin/usernetctl
/usr/sbin/mount.nfs
```

### 5. 危险的Set UID

* 关键目录应严格控制写权限。比如 `/`，`/usr` 等。
* 对系统中默认应该具有的Set UID权限的文件做一个列表，定时检查有没有其他文件被赋予Set UID权限。

#### **查看所有拥有SUID或者SGID权限的文件**
```
find / -perm -4000 -o -perm -2000
```
#### **检查SUID、SGID权限的脚本**

```
#!/bin/bash

find / -perm -4000 -o -perm -2000 > /tmp/setuid.check

for i in $(cat /tmp/setuid.check)
	do
		grep $i /home/suid.log> /dev/null

		if [ "$?" != "0" ]
			then
			echo "$i isn't in listfile!">>/home/suid_log_$(date +%F)
		fi
	done

rm -rf /tmp/setuid.check
```

## 二、Set GID权限（SGID权限）

### 1. Set GID 针对文件的作用

* 只有可执行二进制程序才能设定SGID权限
* 命令执行者要对该程序拥有执行（`x`）权限
* 命令执行者在执行程序时，组身份升级为该程序文件的属组
* Set GID权限只在程序执行过程中有效

### 2. Set GID 针对目录的作用

* 普通用户必须对此目录必须拥有 `r` 和 `x` 权限，才能进入此目录。
* 普通用户在此目录中的有效组会变成此目录的属组。
* 若普通用户对此目录拥有 `w` 权限，新建的文件的默认属组是这个目录的属组。

### 3. 设定 Set GID

    chmod 2755 文件名
    
    chmod +x,g+s 文件名

#### **TIPS**

> 2. 在文件普通权限前加 `2` ，代表SGID权限。
> 3. 当文件拥有SGID权限后，文件所属组对应的权限会变为 `rws`。
> 4. 要设定SGID权限，所属组必须拥有该文件的执行（`x`）权限，否则文件所属组对应的权限会变为 `rwS`，这是无效的权限。

#### **实例**

* 新建一个文件file，若没有执行（`x`）权限，直接设定SGID权限，则组权限会变为 `r-S`，此权限不可用
```
[root/tmp/sgid]# touch file
[root/tmp/sgid]# chmod g+s file

[root/tmp/sgid]# ll
总用量 0
-rw-r-Sr-- 1 root root 0 6月   8 10:44 file
```

* 给 file 文件添加执行权限，变为正常的SUID权限
```
[root/tmp/sgid]# chmod +x file

[root/tmp/sgid]# ll
总用量 0
-rwxr-sr-x 1 root root 0 6月   8 10:44 file*
```

* 创建一个目录 dir，将权限改设置为 `2777` （为了让普通用户可以在该目录下创建文件）
```
[root/tmp/sgid]# chmod 2777 dir/
[root/tmp/sgid]# ll
总用量 0
drwxrwsrwx 2 root root 6 6月   8 10:48 dir/
```

* 切换为普通用户 vagrant，在 dir 目录下创建文件，可见，文件所属组为 root（dir 所属组）
```
[vagrant/tmp/sgid/dir]$ touch file
[vagrant/tmp/sgid/dir]$ ll
总用量 0
-rw-rw-r-- 1 vagrant root 0 6月   8 10:52 file
```

### 4. 取消Set GID

    chmod 0755 文件名（似乎对文件夹不好用）
    
    chmod g-s 文件名

#### **实例**

* `chmod 0755 文件名` 对文件有效，但似乎对文件夹无效
```
[root/tmp/sgid]# ll
总用量 0
drwxrwsrwx 2 root root 6 6月   8 10:48 dir/
-rwxr-sr-x 1 root root 0 6月   8 10:44 file*

[root/tmp/sgid]# chmod 0755 dir/
[root/tmp/sgid]# chmod 0755 file

[root/tmp/sgid]# ll
总用量 0
drwxr-sr-x 2 root root 17 6月   8 10:52 dir/
-rwxr-xr-x 1 root root  0 6月   8 10:44 file*
```

* `chmod g-s 文件名` 对文件和文件夹都有效
```
[root/tmp/sgid]# ll
总用量 0
drwxrwsrwx 2 root root 6 6月   8 10:57 dir/
-rwxr-sr-x 1 root root 0 6月   8 10:44 file*

[root/tmp/sgid]# chmod g-s dir/
[root/tmp/sgid]# chmod g-s file

[root/tmp/sgid]# ll
总用量 0
drwxrwxrwx 2 root root 6 6月   8 10:57 dir/
-rwxr-xr-x 1 root root 0 6月   8 10:44 file*
```

### 5. 应用SUID权限的系统命令

#### **locate 命令**

* locate 命令实际上查询的是 `/var/lib/mlocate/mlocate.db` 文件，而该文件的权限是 `640`。理论上来说，普通用户没有权限查阅该文件。

```
[root/tmp/sgid]# ll /var/lib/mlocate/mlocate.db
-rw-r----- 1 root slocate 2.4M 6月   8 03:20 /var/lib/mlocate/mlocate.db
```

* locate 拥有SGID权限，普通用户在执行 `locate` 命令时，组身份自动切换成slocate，而slocate组对 `/var/lib/mlocate/mlocate.db` 文件有读权限，所以普通用户可以使用locate命令搜索文件。
```
[vagrant/tmp/sgid/dir]$ ll /usr/bin/locate
-rwx--s--x 1 root slocate 40K 4月  11 03:46 /usr/bin/locate*
```

#### **其他常见的拥有SGID权限的命令（文件）**
    
    find / -perm -2000

```
/usr/bin/wall
/usr/bin/lockfile
/usr/bin/write
/usr/bin/ssh-agent
/usr/bin/locate
/usr/sbin/netreport
/usr/sbin/sendmail.sendmail
/usr/libexec/utempter/utempter
/usr/libexec/openssh/ssh-keysign
```

## 三、Sticky BIT（SBIT权限）

### 1. SBIT粘着位作用

* 粘着位目前只对目录有效
* 普通用户对该目录拥有w和x权限，即普通用户可以在此目录有写入权限
* 如果没有粘着位，因为普通用户拥有w权限，所以可以删除此目录下所有文件，包括其他用户建立的文件。一单赋予了粘着位，除了root可以删除所有文件，普通用户就算有w权限，也只能删除自己建立的文件，
但是不能删除其他用户建立的文件

不建议手工建立拥有SBIT的目录

### 2. 设置SBIT

    chmod 1755 目录名
    
    chmod o+t 目录名

#### **TIPS**

> 1. 在文件普通权限前加 `1` ，代表SBIT权限。
> 2. 当文件拥有SGID权限后，文件所属组对应的权限会变为 `rwt`。

#### **实例**

* 创建 `dir_1755` 和 `dir_o+t` 两个目录，分别赋予SBIT权限
```
[root/tmp/sbit]# mkdir dir_1755
[root/tmp/sbit]# mkdir dir_o+t
[root/tmp/sbit]#
[root/tmp/sbit]#
[root/tmp/sbit]# chmod 1755 dir_1755/
[root/tmp/sbit]# chmod o+t dir_o+t/
[root/tmp/sbit]# ll
总用量 0
drwxr-xr-t 2 root root 6 6月   8 11:34 dir_1755/
drwxr-xr-t 2 root root 6 6月   8 11:34 dir_o+t/
```

### 3. 取消SBIT

    chmod 0755 目录名
    
    chmod o-t 目录名

#### **实例**

```
[root/tmp/sbit]# ll
总用量 0
drwxr-xr-t 2 root root 6 6月   8 11:34 dir_1755/
drwxr-xr-t 2 root root 6 6月   8 11:34 dir_o+t/

[root/tmp/sbit]# chmod 0755 dir_1755/
[root/tmp/sbit]# chmod o-t dir_o+t/

[root/tmp/sbit]# ll
总用量 0
drwxr-xr-x 2 root root 6 6月   8 11:34 dir_1755/
drwxr-xr-x 2 root root 6 6月   8 11:34 dir_o+t/
```
### 4. 应用SBIT权限的系统目录

#### **/tmp 目录**

* `/tmp` 目录实际拥有 777 权限，正常情况下，任何用户都可以删除`/tmp` 目录下的文件

```
[root/tmp/sbit]# ll -d /tmp
drwxrwxrwt. 12 root root 4.0K 6月   8 11:33 /tmp/
```

* 然而普通用户却无法删除其他人创建的文件
```
[vagrant/tmp]$ ll /tmp/test.md
-rw-r--r-- 1 root root 215 5月  31 04:24 /tmp/test.md

[vagrant/tmp]$ rm test.md
rm：是否删除有写保护的普通文件 "test.md"？yes
rm: 无法删除"test.md": 不允许的操作
```
---

## 四、chattr权限

### 1. chattr 命令

    chattr [+-=] [选项] [文件或目录]

#### **`[+-=]`**

> `+`：增加权限
> `-`：删除权限
> `=`：设定权限

#### **选项**

> `i`：（insert）插入

> * 文件：不能对文件进行删除和改名，不能添加和修改文件数据。
> * 目录：不能建立和删除文件，可以修改目录下文件数据

> `a`：（append）追加
> 
> * 文件：不能删除和修改数据，只能以 `echo '字符串' > file` 的形式向文件中追加数据。
> * 目录：只允许在目录中添加和修改文件，不允许删除文件

### 2. lsattr 查看文件系统属性

    lsattr [选项] [文件名]

#### **选项**
> `-a`：显示所有文件和目录
> `-d`：若目标是目录，仅列出目录本身的属性，而不是子文件的属性

### 3. 实例

#### **文件应用`i`选项**

    不能对文件进行删除和改名，不能添加和修改文件数据。

* 创建文件 file_i，并随意写入一些内容
```
[root/tmp/chattr]$ touch file_i

[root/tmp/chattr]$ date > file_i
[root/tmp/chattr]$ cat file_i
2018年 06月 08日 星期五 12:04:57 UTC
```

* `chattr +i file_i`
```
[root/tmp/chattr]# chattr +i file_i

[root/tmp/chattr]# lsattr file_i
----i----------- file_i
```

* 不能删除
```
[root/tmp/chattr]# rm file_i
rm: 无法删除"file_i": 不允许的操作
```
* 不能改名
```
[root/tmp/chattr]# mv file_i file
mv: 无法将"file_i" 移动至"file": 不允许的操作
```
* 不能增加数据
```
[root/tmp/chattr]# date >> file_i
bash: file_i: 权限不够
```
* 不能修改数据
```
[root/tmp/chattr]# date > file_i
bash: file_i: 权限不够
```


#### **目录应用`i`选项**

    不能建立和删除文件，可以修改目录下文件数据

* 创建目录 dir_i，并在目录下创建一个文件 file
```
[root/tmp/chattr]# mkdir dir_i
[root/tmp/chattr]# touch dir_i/file

[root/tmp/chattr]# ll dir_i/
总用量 0
-rw-r--r-- 1 root root 0 6月   8 12:13 file
```

* `chattr +i file_i`
```
[root/tmp/chattr]# chattr +i dir_i/

[root/tmp/chattr]# lsattr -d dir_i/
----i----------- dir_i/
```

* 不能创建文件
```
[root/tmp/chattr]# touch dir_i/file2
touch: 无法创建"dir_i/file2": 权限不够
```

* 不能删除文件
```
[root/tmp/chattr]# rm dir_i/file
rm: 无法删除"dir_i/file": 权限不够
```
* 可以修改目录下文件数据
```
[root/tmp/chattr]# date > dir_i/file
[root/tmp/chattr]# cat dir_i/file
2018年 06月 08日 星期五 12:17:16 UTC
```