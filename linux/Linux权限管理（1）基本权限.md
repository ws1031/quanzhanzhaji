## 一、文件类型与权限

> ### `-rwxrw-r--`

* 第1个字符表示文件类型
    * 若是 `-` ，表示是普通文件 
    * 若是 `d` ，表示是目录
    * 若是 `l` ，表示是链接文件 
    * 若是 `b` ，表示是设备文件里的可供存储的接口设备 
    * 若是 `c` ，表示是设备文件里的串行端口设备，例如鼠标、键盘

* 第2至4个字符是一组，表示所有者权限（`u`）
* 第5至7个字符是一组，表示所属组权限（`g`）
* 第8至10个字符是一组，表示其他人权限（`o`）

* 每组中，`r` 表示读权限，`w` 表示写权限，`x` 表示执行权限。

## 二、权限的作用

### 1. 权限对文件的作用

> `r`：读取文件内容，常用命令有 `cat` `more` `head` `tail`

> `w`：编辑、增加、清空文件内容，常用命令有 `vi` `vim`，**但是不包括删除该文件**

> `x`：可以执行文件

### 2. 权限对目录的作用

> `r`：可以查看目录下的文件，常用命令有 `ls`

> `w`：具有修改目录结构的权限。如在此目录下新建文件和目录，删除此目录下的文件和目录，重命名此目录下的文件和目录。常用命令有 `touch` `rm` `mv` `cp`

> `x`：可以进入目录

## 三、umask 文件权限掩码

### 1. 语法

    umask [权限掩码]

> 若没有 `[权限掩码]` 参数，则为查看umask值命令
> 若有 `[权限掩码]` 参数，则为修改umask值命令（临时修改，退出登录后失效）

> root 用户umask值默认为 `0022`，非root用户umask值默认为 `0002`
> umask值 `0022`，第一位`0`表示文件特殊权限掩码，`022`表示文件普通权限掩码

#### **注意**

* 文件最高权限为666
* 目录最高权限为777
* 权限不能使用数字进行换算，必须转换为字母再进行计算
* umask定义的权限，是系统默认权限中准备丢弃的权限

### 2. 修改 umask 的值

#### **临时修改，退出登录后失效**
    
    umask [权限掩码]

#### **永久修改**

* /etc/profile

```
# By default, we want umask to get set. This sets it for login shell
# Current threshold for system reserved uid/gids is 200
# You could check uidgid reservation validity in
# /usr/share/doc/setup-*/uidgid file
if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
    umask 002
else
    umask 022
fi
...省略其他部分...
```

### 3. 文件（目录）默认权限的计算方式

#### **文件默认权限**

> 文件默认不能建立为可执行文件，必须手工赋予执行权限
> 所以，文件默认权限用字符表示最大为 `rw-rw-rw-`

1.若umask 值为 `0022` 文件的普通权限掩码 `022` 转换为字母 `----w--w-`

**`rw-rw-rw-` - `----w--w-` = `rw-r--r--`**

2.若umask 值为 `0033` 文件的普通权限掩码 `033` 转换为字母 `----wx-wx`

**`rw-rw-rw-` - `----wx-wx` = `rw-r--r--`**

#### **目录默认权限**

> 目录默认权限用字符表示最大为 `rwxrwxrwx`

1.若umask 值为 `0022` 文件的普通权限掩码 `022` 转换为字母 `----w--w-`

**`rwxrwxrwx` - `----w--w-` = `rwxr-xr-x`**

2.若umask 值为 `0033` 文件的普通权限掩码 `033` 转换为字母 `----wx-wx`

**`rwxrwxrwx` - `----wx-wx` = `rwxr--r--`**

#### **实例**

* 查看当前umask值为 `0022`
```
[root/tmp/tmp]# umask
0022
```

* 创建文件，默认权限为 `rw-r--r--`
```
[root/tmp/tmp]# touch 0022

[root/tmp/tmp]# ll
总用量 0
-rw-r--r-- 1 root root 0 6月   6 11:29 0022
```

* 创建目录，默认权限为 `rwxr-xr-x`
```
[root/tmp/tmp]# mkdir 0022_dir

[root/tmp/tmp]# ll -d 0022_dir
drwxr-xr-x 2 root root 6 6月   6 11:30 0022_dir/
```

* 将umask值修改为 `0033`
```
[root/tmp/tmp]# umask 0033

[root/tmp/tmp]# umask
0033
```

* 创建文件，默认权限为 `rw-r--r--`
```
[root/tmp/tmp]# touch 0033

[root/tmp/tmp]# ll 0033
-rw-r--r-- 1 root root 0 6月   6 11:32 0033
```

* 创建目录，默认权限为 `rwxr--r--`
```
[root/tmp/tmp]# mkdir 0033_dir

[root/tmp/tmp]# ll -d 0033_dir
drwxr--r-- 2 root root 6 6月   6 11:33 0033_dir/
```

## 四、chmod 修改文件权限

### 1. 语法

    chmod [选项] [模式] [文件名]

#### **选项**

> `-R`：递归

#### **模式**

> 1. [ugoa] [+-=] [rwx]
> 2. [421]

### 2. 权限的数字表示

* r => 4 （读权限）
* w => 2 （写权限）
* x => 1 （执行权限）

例如，`rwxrw-r--` 权限使用数字表示文 `764`

### 3. 实例

#### **为文件所有者（u）添加（+）执行权限（x）**

```
[root/tmp/tmp]# ll
总用量 4.0K
-rw-r--r-- 1 root root 43 6月   5 11:13 file

[root/tmp/tmp]# chmod u+x file

[root/tmp/tmp]# ll
总用量 4.0K
-rwxr--r-- 1 root root 43 6月   5 11:13 file*
```

#### **为文件所属组（g）添加（+）写（w）和执行（x）权限，删除（-）其他人（o）的读权限（r）**
```
[root/tmp/tmp]# chmod g+wx,o-r file

[root/tmp/tmp]# ll
总用量 4.0K
-rwxrwx--- 1 root root 43 6月   5 11:13 file*
```

#### **将文件所属组（g）权限设为（=）可读可写（rw），将其他人（o）权限设为（=）只读（r）**
```
[root/tmp/tmp]# chmod g=rw,o=r file

[root/tmp/tmp]# ll
总用量 4.0K
-rwxrw-r-- 1 root root 43 6月   5 11:13 file*
```

#### **递归地将目录及目录下所有文件权限设为 `755`**
```
[root/tmp/tmp]# mkdir dir
[root/tmp/tmp]# touch dir/file1
[root/tmp/tmp]# touch dir/file2

[root/tmp/tmp]# ll -d dir
drwxr-xr-x 2 root root 30 6月   6 10:23 dir/
[root/tmp/tmp]# ll dir
总用量 0
-rw-r--r-- 1 root root 0 6月   6 10:23 file1
-rw-r--r-- 1 root root 0 6月   6 10:23 file2

[root/tmp/tmp]# chmod -R 755 dir/

[root/tmp/tmp]# ll -d dir
drwxr-xr-x 2 root root 30 6月   6 10:23 dir/
[root/tmp/tmp]# ll dir
总用量 0
-rwxr-xr-x 1 root root 0 6月   6 10:23 file1*
-rwxr-xr-x 1 root root 0 6月   6 10:23 file2*

```


> #### **递归修改目录下所有目录的权限为755,目录下所有文件权限为644**

```
find <directory> -type d -exec chmod 755 {} \;
find <directory> ! -type d -exec chmod 644 {} \;
```

## 五、chown 修改文件所有者

### 1. 语法

    # 只修改文件的所有者
    chown [选项] [所有者] [文件名]

    # 同时修改文件的所有者和所属组
    chown [选项] [所有者]:[所属组] [文件名]
    
#### **选项**

> `-R`：递归

### 2. 实例

#### **只修改文件的所有者**
```
[root/tmp/tmp]# ll
总用量 4.0K
drwxr-xr-x 2 root root 30 6月   6 10:23 dir/
-rwxrw-r-- 1 root root 43 6月   5 11:13 file*

[root/tmp/tmp]# chown wang file

[root/tmp/tmp]# ll file
-rwxrw-r-- 1 wang root 43 6月   5 11:13 file*
```

#### **递归地修改目录及目录下所有文件的所有者和所属组**
```
[root/tmp/tmp]# chown -R wang:users dir/

[root/tmp/tmp]# ll -d dir
drwxr-xr-x 2 wang users 30 6月   6 10:23 dir/
[root/tmp/tmp]# ll dir
总用量 0
-rwxr-xr-x 1 wang users 0 6月   6 10:23 file1*
-rwxr-xr-x 1 wang users 0 6月   6 10:23 file2*
```

## 六、chgrp 修改文件的所属组

### 1. 语法

    chgrp [选项] [所属组] [文件名]
    
#### **选项**

> `-R`：递归

### 2. 实例

#### **递归地修改目录及目录下所有文件的所有者和所属组**
```
[root/tmp/tmp]# chgrp -R root dir/

[root/tmp/tmp]# ll -d dir
drwxr-xr-x 2 wang root 30 6月   6 10:23 dir/

[root/tmp/tmp]# ll dir
总用量 0
-rwxr-xr-x 1 wang root 0 6月   6 10:23 file1*
-rwxr-xr-x 1 wang root 0 6月   6 10:23 file2*
```