Linux权限管理（2）ACL权限
---

## 一、ACL权限的简介

Linux 下用户对文件的操作权限有 r-读, w-写, x-可执行三种。  
而对linux 下的文件而言，用户身份分为：所有者, 所属组, 其它人三种。且文件的所有者，所属组都只能是一个。  
所以在对文件分配用户的使用权限时，只能对这三种身份进行分配读、写、执行权限。

Linux 主要作为服务器系统使用，用户众多。所以在实际使用场景中，这三种身份并不能很好地实现资源权限分配问题，所以就有了ACL权限。  
ACL 权限就是为了解决linux 下三种身份不能满足资源权限分配需求的问题的。

## 二、分区ACL权限开启

### 1. 查看分区ACL权限是否开启

    dumpe2fs -h [分区]

> `dumpe2fs` 是查询指定分区详细文件系统信息的命令
> `-h` 表示仅显示超级块中的信息，而不显示磁盘块组的详细信息

如果输出中 `Default mount options` 项有 `acl`，说明分区ACL权限开启。

如果没有开启分区ACL权限，则需要手动开启。

#### **实例**

```
[root~]# dumpe2fs -h /dev/sda1 | grep 'Default mount options'
dumpe2fs 1.42.9 (4-Feb-2014)
Default mount options:    user_xattr acl
```

### 2. 临时开启分区ACL权限

#### **重新挂载根分区，并在挂载时加入acl权限**
```
mount -o remount,acl / 
```
### 3. 永久开启分区ACL权限

#### **编辑配置文件 `/etc/fstab`**

* 找到根分区磁盘的列，在第四列后追加 `acl`
```
UUID=64ecdf77-db7b-48a8-9066-abfb837f2e24   /   ext4    defaults,acl    1   1
```

* 重新挂载文件系统或重启系统，是修改生效
```
mount -o remount / 
```

## 三、getfacl 和 setfacl 命令

### 1. getfacl

    查看ACL权限

#### 语法

    getfacl [文件名]

### 2. setfacl

    添加、删除ACL权限

#### **语法**

    setfacl [选项] [文件名]

#### **选项**

> `-m <ACL规则>`：设定ACL权限，多条ACL规则以逗号(`,`)隔开
> `-x <ACL规则>`：删除指定ACL权限，多条ACL规则以逗号(`,`)隔开
> `-b`：删除所有ACL权限
> `-d`：设定默认ACL权限
> `-k`：删除默认ACL权限
> `-R`：递归设定ACL权限

#### **ACL规则**

setfacl命令可以识别以下的规则格式：

* `[d:]u:[用户名]:[权限(rwx)]`：指定用户的权限
* `[d:]g:[组名]:[权限(rwx)]`：指定用户组的权限
* `[d:]m:[权限(rwx)]`：指定最大有效权限

> `[d:]` 表示设定为默认权限

## 四、设定ACL权限

### 1. 给用户设定ACL权限

#### **语法**

    setfacl -m u:[用户名]:[权限(rwx)] [文件名]

#### **应用**

* 新建一个目录 `res/` ，并设置权限为 `750`
```
[root/tmp/acl]# mkdir res
[root/tmp/acl]# chmod 750 res/
[root/tmp/acl]# ll
总用量 0
drwxr-x--- 2 root root 6 6月   8 01:11 res/
```

* 为用户 `wang` 设置对目录 `res/` 的读写权限
```
[root/tmp/acl]# setfacl -m u:wang:rx res/
```

* 查看目录信息，可见在权限列末尾多了一个 `+`
```
[root/tmp/acl]# ll
总用量 0
drwxr-x---+ 2 root root 6 6月   8 01:11 res/
```

* 查看目录 `res/` 的acl权限，可见多了一行 `user:wang:r-x`
```
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
user:wang:r-x
group::r-x
mask::r-x
other::---
```

* 此时切换到用户 `wang`，可以进入目录 `res/`，可以查看目录内的文件，不能创建文件，说明用户 `wang` 对目录 `res/` 的权限为 `r-x`
```
[root~]# su wang

[wang@10 root]$ cd /tmp/acl/

[wang@10 acl]$ ll
总用量 0
drwxr-x---+ 2 root root 6 6月   8 01:11 res

[wang@10 acl]$ cd res/

[wang@10 res]$ ll
总用量 0

[wang@10 res]$ touch file
touch: 无法创建"file": 权限不够
```


### 2. 给用户组设定ACL权限

#### **语法**

    setfacl -m g:[组名]:[权限(rwx)] [文件名]

#### **应用**

* 还是之前的 `res/` 目录
```
[root/tmp/acl]# ll
总用量 0
drwxr-x--- 2 root root 6 6月   8 01:11 res/
```

* 新建用户组 `phper`，再新建一个用户 `zhang`，将其加入到 `phper` 组
```
[root/tmp/acl]# groupadd phper

[root/tmp/acl]# useradd -g phper zhang

[root/tmp/acl]# grep phper /etc/group
phper:x:1002:

[root/tmp/acl]# grep zhang /etc/passwd
zhang:x:1002:1002::/home/zhang:/bin/bash
```

* 为用户组 `phper` 设置对目录 `res/` 的读、写、执行权限
```
[root/tmp/acl]# setfacl -m g:phper:rwx res/
```

* 查看目录 `res/` 的acl权限，可见多了一行 `group:phper:rwx`
```
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
user:wang:r-x
group::r-x
group:phper:rwx
mask::rwx
other::---
```

* 此时切换到用户 `zhang`，可以进入目录 `res/`，可以创建文件，可以查看目录内的文件，说明用户 `zhang` 对目录 `res/` 的权限为 `rwx`
```
[root/tmp/acl]# su zhang

[zhang@10 acl]$ ll
总用量 0
drwxrwx---+ 2 root root 6 6月   8 01:11 res

[zhang@10 acl]$ cd res/

[zhang@10 res]$ touch phper.log

[zhang@10 res]$ ll
总用量 0
-rw-r--r-- 1 zhang phper 0 6月   8 02:27 phper.log
```

## 五、删除ACL权限

### 1. 删除指定用户的ACL权限

#### **语法**

    setfacl -x u:[用户名] [文件名]

#### **应用**

* 删除用户 `wang` 对目录 `res/` 的ACL权限
```
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
user:wang:r-x
group::r-x
group:phper:rwx
mask::rwx
other::---

[root/tmp/acl]# setfacl -x u:wang res/

[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx
mask::rwx
other::---
```

### 2. 删除指定用户组的ACL权限

#### **语法**

    setfacl -x g:[组名] [文件名]

#### **应用**

* 删除用户组 `phper` 对目录 `res/` 的ACL权限
```
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx
mask::rwx
other::---

[root/tmp/acl]# setfacl -x g:phper res/
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
mask::r-x
other::---
```

### 3. 删除文件的所有ACL权限

#### **语法**

    setfacl -b [文件名]

#### **应用**

* 再把用户 `wang` 和用户组 `phper` 对目录 `res/` 的ACL权限加回来
```
[root/tmp/acl]# setfacl -m u:wang:r-x res/
[root/tmp/acl]# setfacl -m g:phper:rwx res/
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
user:wang:r-x
group::r-x
group:phper:rwx
mask::rwx
other::---
```

* 删除文件的所有ACL权限
```
[root/tmp/acl]# setfacl -b res/
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
other::---
```

## 六、ACL最大有效权限：mask

> mask 是用来指定最大有效权限的。如果给用户赋予了ACL权限，就需要与 mask 的权限 “相与”，结果才是用户真正获得的权限。

### 1. 查看最大有效权限 mask

    getfacl [文件]

> 文件必须设置了ACL权限才会有mask项

#### **实例**

```
[root/tmp/acl]#  setfacl -m g:phper:rwx res/
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx
mask::rwx
other::---
```

### 2. 修改最大有效权限 mask

    setfacl -m m:[权限(rwx)] [文件名]

#### **实例**

* 修改最大有效权限 mask 为 `r-x`
```
[root/tmp/acl]# setfacl -m m:rx res/

[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx                 #effective:r-x
mask::r-x
other::---
```

* 此时切换到用户 `zhang`，可以进入目录 `res/`，可以查看目录内的文件，不能创建文件，说明用户 `zhang` 对目录 `res/` 的权限为 `r-x`
```
[zhang@10 acl]$ ll
总用量 0
drwxr-x---+ 2 root root 6 6月   8 05:19 res

[zhang@10 acl]$ getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx                 #effective:r-x
mask::r-x
other::---

[zhang@10 acl]$ cd res/

[zhang@10 res]$ ll
总用量 0

[zhang@10 res]$ touch file
touch: 无法创建"file": 权限不够
```

> #### **值得注意的是，mask 值必须在设定完所有用户和组的ACL权限之后再做修改。如果先修改了 mark 的值，再设定用户和组的ACL权限，mark 值会被系统自动重置。**

```
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx                 #effective:r-x
mask::r-x
other::---

[root/tmp/acl]# setfacl -m u:wang:rwx res/

[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
user:wang:rwx
group::r-x
group:phper:rwx
mask::rwx
other::---
```

## 七、递归ACL权限

> 设定父目录ACL权限的同时，为所有子文件和目录设定相同的ACL权限。

> 递归权限只能赋予目录，不能赋予文件。

    setfacl -m u:[用户名]:[权限(rwx)] -R [文件名]

#### 实例

* 创建一个目录 `res/`，再在 `res/` 目录下创建 `dir/` 和 `file` 两个文件，都没有设定ACL权限
```
[root/tmp/acl]# ll
总用量 0
drwxr-xr-x 3 root root 28 6月   8 06:04 res/

[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

[root/tmp/acl]# cd res

[root/tmp/acl/res]# ll
总用量 0
drwxr-xr-x 2 root root 6 6月   8 06:02 dir/
-rw-r--r-- 1 root root 0 6月   8 06:02 file

[root/tmp/acl/res]# getfacl dir/
# file: dir/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

[root/tmp/acl/res]# getfacl file
# file: file
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

* 递归设置ACL权限，给 `res/` 及其下的所有文件和目录都设置了ACL权限
```
[root/tmp/acl]# setfacl -m g:phper:rwx -R res/

[root/tmp/acl]# ll
总用量 0
drwxrwxr-x+ 3 root root 27 6月   8 06:06 res/

[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx
mask::rwx
other::r-x

[root/tmp/acl]# cd res/
[root/tmp/acl/res]# ll
总用量 0
drwxrwxr-x+ 2 root root 6 6月   8 06:02 dir/
-rw-rwxr--+ 1 root root 0 6月   8 06:02 file*

[root/tmp/acl/res]# getfacl dir/
# file: dir/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx
mask::rwx
other::r-x

[root/tmp/acl/res]# getfacl file
# file: file
# owner: root
# group: root
user::rw-
group::r--
group:phper:rwx
mask::rwx
other::r--
```

## 八、默认ACL权限


> 如果给目录设定了默认ACL权限，那么在这个目录新建的所有文件和目录都会继承父目录的ACL权限。

> 默认权限只能赋予目录，不能赋予文件。

> 即使默认ACL权限设置了执行权限，目录下新创建的文件也不会拥有执行权限！

    setfacl -m d:u:[用户名]:[权限(rwx)] [文件名]

#### 实例

* 创建一个目录 `res/`，并在 `res/` 目录下创建 `file` 文件，都没有设定ACL权限
```
[root/tmp/acl]# mkdir res
[root/tmp/acl]# touch res/file

[root/tmp/acl]# ll
总用量 0
drwxr-xr-x 2 root root 17 6月   8 06:18 res/

[root/tmp/acl]# ll res/
总用量 0
-rw-r--r-- 1 root root 0 6月   8 06:18 file

[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x

[root/tmp/acl]# getfacl res/file
# file: res/file
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

* 给 `res/` 目录设定默认ACL权限
```
[root/tmp/acl]# setfacl -m d:g:phper:rwx res/
[root/tmp/acl]# ll
总用量 0
drwxr-xr-x+ 2 root root 17 6月   8 06:18 res/
[root/tmp/acl]# getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:group::r-x
default:group:phper:rwx
default:mask::rwx
default:other::r-x
```

---
> #### **值得注意的是，`res/` 目录本身和其下已经存在的文件 `file` 并没有ACL权限**

* 切换到属于 `phper` 组的用户 `zhang`，可以看到，`zhang` 只获得了 `other` 用户该有的权限，并不能在 `res/` 目录下创建文件，也不能修改 `res/file` 文件
```
[zhang@10 acl]$ ll
总用量 0
drwxr-xr-x+ 3 root root 27 6月   8 06:26 res

[zhang@10 acl]$ getfacl res/
# file: res/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:group::r-x
default:group:phper:rwx
default:mask::rwx
default:other::r-x

[zhang@10 acl]$ cd res/
[zhang@10 res]$ ll
总用量 4
-rw-r--r--  1 root root 0 6月   8 06:18 file

[zhang@10 res]$ getfacl file
# file: file
# owner: root
# group: root
user::rw-
group::r--
other::r--

[zhang@10 res]$ touch file2
touch: 无法创建"file2": 权限不够

[zhang@10 res]$ date > file
bash: file: 权限不够
```

---

* 在 `res/` 目录下创建目录 `dir/`，自动拥有了父目录设置的默认ACL权限

```
[root/tmp/acl/res]# mkdir dir

[root/tmp/acl/res]# getfacl dir/
# file: dir/
# owner: root
# group: root
user::rwx
group::r-x
group:phper:rwx
mask::rwx
other::r-x
default:user::rwx
default:group::r-x
default:group:phper:rwx
default:mask::rwx
default:other::r-x
```

* 在 `res/` 目录下创建文件 `file_new`，自动拥有了父目录设置的默认ACL权限，执行权限除外
```
[root/tmp/acl/res]# touch file_new

[root/tmp/acl/res]# getfacl file_new
# file: file_new
# owner: root
# group: root
user::rw-
group::r-x                      #effective:r--
group:phper:rwx                 #effective:rw-
mask::rw-
other::r--
```

> 在任何情况下，新建的文件都不可能自动拥有执行权限！即使默认ACL权限设置了执行权限，目录下新创建的文件也不会拥有执行权限！