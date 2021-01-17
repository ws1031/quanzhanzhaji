Linux用户和用户组管理
---

### 一、用户信息存储文件

#### 1. `/etc/passwd` 存储当前系统中所有用户的信息

##### **文件内容格式**


用户名**:**密码占位符**:**用户ID**:**用户组ID**:**用户注释信息**:**用户家目录**:**shell类型

> 信息之间以 `:` 分隔

#### **实例**

```
[root/etc]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
vagrant:x:1001:1001::/home/vagrant:/bin/bash
vboxadd:x:997:1::/var/run/vboxadd:/bin/false
memcached:x:996:995:Memcached daemon:/run/memcached:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
... 省略n行 ...
```

#### 2. `/etc/shadow` 存储当前系统中所有用户的密码信息

##### **文件内容格式**

用户名**:**密码(加密后)**:**最后一次修改时间**:**最小时间间隔**:**密码有效期**:**警告时间**:**不重要，省略...

> 1. 信息之间以 `:` 分隔
> 2. "密码" 字段存放的是加密后的用户密码：
> * 如果为空，则对应用户没有密码，登录时不需要密码；
> * 星号（`*`）代表帐号被锁定，不能登录；
> * 双叹号 (`!!`) 表示这个密码已经过期了；
> * `$6$` 开头，表明是用SHA-512加密；
> * `$1$` 开头，表明是用MD5加密；
> * `$2$` 开头，表明是用Blowfish加密；
> * `$5$` 开头，表明是用 SHA-256加密；
> 3. "最后一次修改时间" 表示的是从1970年1月1日起，到用户最后一次修改密码的天数。
> 4. "最小时间间隔" 指的是两次修改密码之间所需的最小天数。
> 5. "密码有效期” 指的是密码保持有效的最大天数。
> 6. "警告时间” 指的是从系统开始警告用户到用户密码正式失效之间的天数。

##### **实例**

```
[root/etc]# cat /etc/shadow
root:$1$damlkd,f$UC/u5pUts5QiU3ow.CSso/:16630:0:99999:7:::
bin:*:16372:0:99999:7:::
sync:*:16372:0:99999:7:::
mail:*:16372:0:99999:7:::
nobody:*:16372:0:99999:7:::
sshd:!!:16630::::::
vagrant:$1$5eWJGPyo$/T4TT4hm2e.uKUhx9V0mi.:16630:0:99999:7:::
vboxadd:!!:16630::::::
memcached:!!:17667::::::
apache:!!:17675::::::
... 省略n行 ...
```

### 二、用户组信息存储文件

#### 1. `/etc/group` 存储当前系统中所有用户组的信息

##### **文件内容格式**

用户组名**:**组密码占位符**:**用户组ID**:**组内用户列表

> 1. 信息之间以 `:` 分隔
> 2. "组内用户列表" 是属于这个组的所有用户的列表，不同用户之间用逗号(`,`)分隔。这个用户组可能是用户的主组，也可能是附加组。

##### **实例**

```
[root/etc]# cat /etc/group
root:x:0:
bin:x:1:
wheel:x:10:veewee,vagrant
mail:x:12:
nobody:x:99:
users:x:100:
sshd:x:74:
veewee:x:1000:
vagrant:x:1001:
memcached:x:995:
slocate:x:21:
apache:x:48:
... 省略n行 ...
```

#### 2. `/etc/gshadow` 存储当前系统中所有用户组的密码信息

##### **文件内容格式**

用户组名**:**组密码**:**组管理者**:**组内用户列表

> 1. 信息之间以 `:` 分隔
> 2. "组密码" 为空或 `!` 时，表示用户组没有密码
> 3. "组管理者" 为空时，表示组内每个用户都是管理者。多个管理者之间可以使用逗号(`,`)分隔
> 4. "组内用户列表" 是属于这个组的所有用户的列表，不同用户之间用逗号(`,`)分隔。这个用户组可能是用户的主组，也可能是附加组。

##### **实例**

```
[root/etc]# cat /etc/gshadow
root:::
bin:::
wheel:::veewee,vagrant
mail:::
nobody:::
users:::
sshd:!::
veewee:!::
vagrant:!::
memcached:!::
slocate:!::
apache:!::
... 省略n行 ...
```

### 三、常用用户管理命令

#### 1. `useradd` 创建用户并设置基本信息

##### **语法**

    useradd [选项] [用户名]
    
##### **选项**

> `-m`：自动建立用户的家目录
> `-d <dir>`：指定用户的家目录
> `-g <用户组>`：指定用户所属的用户组
> `-s <shell>`：指定用户登入后所使用的shell
> `-u <uid>`：指定用户id
> `-c <备注>`：加上备注文字。备注文字会保存在passwd的备注栏位中

##### **实例**

* 添加一个用户，并为用户在 `/home/` 目录自动创建一个同名的家目录
```
[root/etc]# useradd -m zhang

[root/etc]# id zhang
uid=1002(zhang) gid=1002(zhang) 组=1002(zhang)

[root/etc]# cat /etc/passwd | grep zhang
zhang:x:1002:1002::/home/zhang:/bin/bash

[root/etc]# ll /home/ | grep zhang
drwx------  2 zhang   zhang     59 6月   6 01:19 zhang/
```

* 添加一个用户，指定家目录，用户组，shell，用户ID和备注
```
[root/etc]# useradd -d /home/php -g root -s /bin/bash -u 666 -c 'PHP engineer' wang

[root/etc]# id wang
uid=666(wang) gid=0(root) 组=0(root)

[root/etc]# cat /etc/passwd | grep wang
wang:x:666:0:PHP engineer:/home/php:/bin/bash

[root/etc]# ll /home/ | grep php
drwx------  2 wang    root      59 6月   6 01:26 php/
```

#### 2. `usermod` 修改用户基本信息

##### **语法**

    usermod [选项] [用户名]
    
##### **选项**

> `-l <新用户名>`：修改用户名
> `-d <dir>`：指定用户的家目录
> `-g <用户组>`：指定用户所属的用户组
> `-s <shell>`：指定用户登入后所使用的shell
> `-u <uid>`：指定用户id
> `-c <备注>`：加上备注文字。备注文字会保存在passwd的备注栏位中
> `-L`：锁定用户密码，使密码无效，即该用户不能使用密码登录
> `-U`：解除密码锁定

> 注意事项
> 1. 修改用户信息时，用户不能处在登录状态。
> 2. 用户ID最好不要随便修改，容易造成该用户失去文件的所有者权限。
> 3. 修改用户家目录，系统并不会自动创建该目录。需要手动创建该目录，并设置所有者权限。

##### **实例**

* 修改用户的用户名，用户组，shell和备注
```
[root/etc]# usermod -l zhangfei -g root -s /bin/bash -c 'iOS engineer' zhang

[root/etc]# id zhangfei
uid=1994(zhangfei) gid=0(root) 组=0(root)

[root/etc]# id zhang
id: zhang: no such user

[root/etc]# cat /etc/passwd | grep zhangfei
zhangfei:x:1002:0:iOS engineer:/home/zhang:/bin/bash

[root/etc]# ll /home/ | grep zhangfei
drwx------  2 zhangfei zhang     79 6月   6 01:50 zhang/
```

* 锁定和解锁用户密码登录
```
# 锁定用户密码
[root/etc]# usermod -L zhangfei

# 新开一个bash窗口，尝试使用密码登录，被拒接
[Administrator~/Desktop] ]$ssh zhangfei@192.168.33.88
zhangfei@192.168.33.88's password:
Permission denied, please try again.

# 解锁用户密码
[root/etc]# usermod -U zhangfei

# 使用密码登录成功
[Administrator~/Desktop] ]$ssh zhangfei@192.168.33.88
zhangfei@192.168.33.88's password:
Last login: Wed Jun  6 02:04:20 2018
[zhangfei@10 ~]$
```


#### 3. `userdel` 删除用户

##### **语法**

    userdel [选项] [用户名]
    
##### **选项**

> `-r`：删除用户的同时，删除用户的家目录及里面的文件
> `-f`：强制删除用户，即使用户当前已登录

##### **实例**

```
[root/etc]# userdel -r zhangfei

[root/etc]# id zhangfei
uid=1994(zhangfei) gid=0(root) 组=0(root)

[root/zhang]# id zhangfei
id: zhangfei: no such user

[root/etc]# cat /etc/passwd | grep zhangfei

[root/etc]# ll /home/ | grep zhangfei
```

#### 4. `passwd` 设置用户密码

##### **语法**

    passwd [选项] [用户名]
    
##### **选项**

> 不加任何选项：'passwd [用户名]'，设置用户密码
> `-d`：删除用户密码
> `-l`：锁住用户密码
> `-u`：解锁用户密码

> 注意事项
> 1. 没有设置密码和被删除密码的用户无法通过密码登录

##### **实例**

* 删除用户密码
```
[root~]# passwd -d wang
清除用户的密码 wang。
passwd: 操作成功
```

* 用户无法使用密码登录
```
[Administrator~/Desktop] ]$ssh wang@192.168.33.88
wang@192.168.33.88's password:
Permission denied, please try again.
```

* 设置用户密码
```
[root~]# passwd wang
更改用户 wang 的密码 。
新的 密码：
重新输入新的 密码：
passwd：所有的身份验证令牌已经成功更新。
```

* 用户可以通过密码登录
```
[Administrator~/Desktop] ]$ssh vagrant@192.168.33.88
vagrant@192.168.33.88's password:
Last login: Wed Jun  6 00:13:19 2018 from 10.0.2.2
[vagrant~]$ 
```

* 锁定用户密码
```
[root~]# passwd -l wang
锁定用户 wang 的密码 。
passwd: 操作成功
```

* 用户无法使用密码登录
```
[Administrator~/Desktop] ]$ssh wang@192.168.33.88
wang@192.168.33.88's password:
Permission denied, please try again.
```

* 解锁用户密码
```
[root~]# passwd -u wang
解锁用户 wang 的密码。
passwd: 操作成功
```

* 用户可以通过密码登录
```
[Administrator~/Desktop] ]$ssh vagrant@192.168.33.88
vagrant@192.168.33.88's password:
Last login: Wed Jun  6 02:44:50 2018 from 192.168.33.1
[vagrant~]$ 
```

#### 4. 禁止 `root` 用户外的所有用户登录

* 禁止 `root` 用户外的所有用户登录，只需要在etc目录下创建 `/etc/nologin` 文件即可
```
[root~] touch /etc/nologin
```

* 尝试登录，登录失败
```
[Administrator~/Desktop] ]$ssh vagrant@192.168.33.88
vagrant@192.168.33.88's password:
Connection closed by 192.168.33.88
```

* 删除 etc目录下的 `/etc/nologin` 文件
```
[root~]# rm /etc/nologin
```

* 尝试登录，登录成功
```
[Administrator~/Desktop] ]$ssh vagrant@192.168.33.88
vagrant@192.168.33.88's password:
Last login: Wed Jun  6 00:13:19 2018 from 10.0.2.2
[vagrant~]$
```
### 四、常用用户组管理命令

#### 1. `groupadd` 创建用户组

##### **语法**

    groupadd [选项] [用户组名]
    
##### **选项**

> `-g <gid>`：指定用户组ID

##### **实例**

* 添加一个用户组，并制定ID
```
[root~]# groupadd -g 666 php

[root~]# grep php /etc/group
php:x:666:
```

#### 2. `groupmod` 修改用户组

##### **语法**

    groupmod [选项] [用户组名]
    
##### **选项**

> `-n <新用户组名>`：修改用户组名
> `-g <gid>`：指定用户组ID

##### **实例**

* 修改用户组名
```
[root~]# groupmod -n ios php

[root~]# grep ios /etc/group
ios:x:666:
```

* 修改用户组ID
```
[root~]# groupmod -g 888 ios

[root~]# grep ios /etc/group
ios:x:888:
```

#### 3. `groupdel` 删除用户组

##### **语法**

    groupdel [用户组名]

> 若该用户组中仍有用户，则必须先删除这些用户后，才能删除用户组。

##### **实例**

```
[root~]# groupdel ios

[root~]# grep ios /etc/group
```

#### 4. `gpasswd` 管理用户组成员

##### **语法**

    gpasswd [选项] [用户组名]
    
##### **选项**

> `-a <用户名>`：添加用户到组
> `-d <用户名>`：从组中删除用户

##### **实例**

* 添加用户到组
```
[root~]# gpasswd -a wang php
正在将用户“wang”加入到“php”组中

[root~]# grep php /etc/group
php:x:666:wang

[root~]# id wang
uid=666(wang) gid=0(root) 组=0(root),666(php)
```

* 从组中删除用户
```
[root~]# gpasswd -d wang php
正在将用户“wang”从“php”组中删除

[root~]# grep php /etc/group
php:x:666:

[root~]# id wang
uid=666(wang) gid=0(root) 组=0(root)
```

### 五、其他用户命令

#### 1. `whoami`

    显示当前登录用户名

```
[root~]# whoami
root
```

#### 2. `id [用户名]`

    显示指定用户信息，包括用户ID，用户名，主要组ID及名称，附属组ID及名称

```
[root~]# id
uid=0(root) gid=0(root) 组=0(root)

[root~]# id root
uid=0(root) gid=0(root) 组=0(root)

[root~]# id vagrant
uid=1001(vagrant) gid=1001(vagrant) 组=1001(vagrant),10(wheel)

[root~]# id wang
uid=666(wang) gid=0(root) 组=0(root)
```

#### 3. `groups [用户名]`

    显示指定用户所在的所有组

```
[root~]# groups
root

[root~]# groups root
root : root

[root~]# groups vagrant
vagrant : vagrant wheel

[root~]# groups wang
wang : root
```

---


![欢迎关注公众号【全栈札记】](https://md.s1031.cn/xsj/2021_1_4_扫码_搜索联合传播样式-白色版.png)