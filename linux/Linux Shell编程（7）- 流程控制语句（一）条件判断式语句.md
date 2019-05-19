Linux Shell编程（7）- 流程控制语句（一）条件判断式语句
---


## 一、两种判断格式

#### **test 判断式**

#### **[ 判断式 ] (常用)**


## 二、按照文件类型进行判断

测试选项|作用
-|-
`-b 文件`|判断该文件是否存在，并且是块设备文件
`-c 文件`|判断该文件是否存在，并且是字符设备文件
**`-d 文件`**|**判断该文件是否存在，并且是目录**
**`-e 文件`**|**判断该文件是否存在**
**`-f 文件`**|**判断该文件是否存在，并且是普通文件**
`-L 文件`|判断该文件是否存在，并且是符号链接文件
`-p 文件`|判断该文件是否存在，并且是管道文件
`-s 文件`|判断该文件是否存在，并且非空
`-S 文件`|判断该文件是否存在，并且是套接字文件

### 1. 利用 `$?` 判断文件是否存在
```
[root/tmp]# [ -e /etc/passwd ]
[root/tmp]# echo $?
0

[root/tmp]# [ -e /etc/passwds ]
[root/tmp]# echo $?
1
```

### 2. 利用 `&&` 和 `||` 判断文件是否是目录
```
[root/tmp]# [ -d /etc/passwd ] && echo 'yes' || echo 'no'
no

[root/tmp]# [ -d /etc/ ] && echo 'yes' || echo 'no'
yes
```

## 三、按照文件权限进行判断

测试选项|作用
-|-
**`-r 文件`**|**判断该文件是否存在，并且当前用户拥有读权限**
**`-w 文件`**|**判断该文件是否存在，并且当前用户拥有写权限**
**`-x 文件`**|**判断该文件是否存在，并且当前用户拥有执行权限**
`-u 文件`|判断该文件是否存在，并且拥有SUID权限
`-g 文件`|判断该文件是否存在，并且拥有SGID权限
`-k 文件`|判断该文件是否存在，并且拥有SBit权限

### **实例**

* 切换回普通用户，创建一个文件 rwx.md，默认权限为 `rw-rw-r--`
```
[vagrant~/rwx]$ touch rwx.md
[vagrant~/rwx]$ ll
总用量 0
-rw-rw-r-- 1 vagrant vagrant 0 6月   1 04:23 rwx.md
```

* 判断文件权限，有读、写权限，没有执行权限
```
[vagrant~/rwx]$ [ -r rwx.md ] && echo yes || echo no
yes

[vagrant~/rwx]$ [ -w rwx.md ] && echo yes || echo no
yes

[vagrant~/rwx]$ [ -x rwx.md ] && echo yes || echo no
no
```
* 将文件所有者修改为 root ，并修改组权限为可读，不可写，可执行
```
[vagrant~/rwx]$ sudo chown root rwx.md

[vagrant~/rwx]$ sudo chmod 654 rwx.md

[vagrant~/rwx]$ ll
总用量 0
-rw-r-xr-- 1 root vagrant 0 6月   1 04:23 rwx.md*
```

* 此时用户应用的是组权限，即可读，不可写，可执行
```
[vagrant~/rwx]$ [ -w rwx.md ] && echo yes || echo no
no

[vagrant~/rwx]$ [ -x rwx.md ] && echo yes || echo no
yes
```

* 将文件的所有者改回 vagrant
```
[vagrant~/rwx]$ sudo chown vagrant rwx.md

[vagrant~/rwx]$ ll
总用量 0
-rw-r-xr-- 1 vagrant vagrant 0 6月   1 04:23 rwx.md*
```
* 此时用户应用的是所有者权限，即可读、可写、不可执行
```
[vagrant~/rwx]$ [ -w rwx.md ] && echo yes || echo no
yes

[vagrant~/rwx]$ [ -x rwx.md ] && echo yes || echo no
no

[vagrant~/rwx]$ ./rwx.md
-bash: ./rwx.md: 权限不够
```

## 四、两个文件之间进行比较

测试选项| 作用
-|-
`文件1 -nt 文件2`|判断文件1的修改时间是否比文件2的新
`文件1 -ot 文件2`|判断文件1的修改时间是否比文件2的旧
`文件1 -ef 文件2`|判断文件1和文件2的Inode号是否一致，可以理解为两个文件是否为同一个文件。这适用于判断硬链接很好的方法

### 1. 比较两个文件的最后修改时间

* 相隔1秒，创建file1和file2两个文件
```
[root/tmp/tmp]# touch file1; sleep 1; touch file2
[root/tmp/tmp]# ll
总用量 0
-rw-r--r-- 1 root root 0 6月   1 06:52 file1
-rw-r--r-- 1 root root 0 6月   1 06:52 file2
```
* 比较两个文件的最后修改时间
```
[root/tmp/tmp]# [ file1 -ot file2 ] && echo yes || echo no
yes

[root/tmp/tmp]# [ file1 -nt file2 ] && echo yes || echo no
no
```

### 2. 判断两个文件的Inode号是否一致

* 创建 file 和 file2，并创建一个 file 的硬链接 file_ln
```
[root/tmp/tmp]# touch file

[root/tmp/tmp]# touch file2

[root/tmp/tmp]# ln file file_ln
```

* 由 `ll -i` 的第一列可以看出 file 和 file_ln 的i节点是一致的
```
[root/tmp/tmp]# ll -i
总用量 0
34117777 -rw-r--r-- 2 root root 0 6月   1 06:57 file
34117781 -rw-r--r-- 1 root root 0 6月   1 06:58 file2
34117777 -rw-r--r-- 2 root root 0 6月   1 06:57 file_ln
```

* 通过 `[ file -ef file_ln ]` 判断只有 file 和 file_ln 的i节点是一致的
```
[root/tmp/tmp]# [ file -ef file_ln ] && echo yes || echo no
yes

[root/tmp/tmp]# [ file2 -ef file_ln ] && echo yes || echo no
no

[root/tmp/tmp]# [ file -ef file2 ] && echo yes || echo no
no
```

## 五、两个整数之间比较

测试选项| 作用
-|-
`整数1 -eq 整数2`|判断整数1是否**等于**整数2
`整数1 -ne 整数2`|判断整数1是否**不等于**整数2
`整数1 -gt 整数2`|判断整数1是否**大于**整数2
`整数1 -lt 整数2`|判断整数1是否**小于**整数2
`整数1 -ge 整数2`|判断整数1是否**大于等于**整数2
`整数1 -le 整数2`|判断整数1是否**小于等于**整数2

### 1. `-eq`：判断整数1是否**等于**整数2
```
[root~]# [ 1 -eq 1 ] && echo yes || echo no
yes
[root~]# [ 1 -eq 2 ] && echo yes || echo no
no
[root~]# [ 2 -eq 1 ] && echo yes || echo no
no
```

### 2. `-ne`：判断整数1是否**不等于**整数2
```
[root~]# [ 1 -ne 1 ] && echo yes || echo no
no
[root~]# [ 1 -ne 2 ] && echo yes || echo no
yes
[root~]# [ 2 -ne 1 ] && echo yes || echo no
yes
```

### 3. `-gt`：判断整数1是否**大于**整数2
```
[root~]# [ 1 -gt 1 ] && echo yes || echo no
no
[root~]# [ 1 -gt 2 ] && echo yes || echo no
no
[root~]# [ 2 -gt 1 ] && echo yes || echo no
yes
```

### 4. `-lt`：判断整数1是否**小于**整数2
```
[root~]# [ 1 -lt 1 ] && echo yes || echo no
no
[root~]# [ 1 -lt 2 ] && echo yes || echo no
yes
[root~]# [ 2 -lt 1 ] && echo yes || echo no
no
```

### 5. `-ge`：判断整数1是否**大于等于**整数2
```
[root~]# [ 1 -ge 1 ] && echo yes || echo no
yes
[root~]# [ 1 -ge 2 ] && echo yes || echo no
no
[root~]# [ 2 -ge 1 ] && echo yes || echo no
yes
```

### 6. `-le`：判断整数1是否**小于等于**整数2
```
[root~]# [ 1 -le 1 ] && echo yes || echo no
yes
[root~]# [ 1 -le 2 ] && echo yes || echo no
yes
[root~]# [ 2 -le 1 ] && echo yes || echo no
no
```

## 六、字符串的判断

测试选项| 作用
-|-
`-z 字符串`|判断字符串是否为**空**
`-n 字符串`|判断字符串是否为**非空**
`字符串1 == 字符串2`|判断字符串1和字符串2是否**相等**
`字符串1 != 字符串2`|判断字符串1和字符串2是否**不相等**

### 1. `-z 字符串`：判断字符串是否为**空**
```
[root~]# name=''
[root~]# [ -z "$name" ] && echo yes || echo no
yes

[root~]# name=Wang
[root~]# [ -z "$name" ] && echo yes || echo no
no

[root~]# unset name
[root~]# [ -z "$name" ] && echo yes || echo no
yes
```

### 2. `-n 字符串`：判断字符串是否为**非空**
```
[root~]# name=''
[root~]# [ -n "$name" ] && echo yes || echo no
no

[root~]# name=Wang
[root~]# [ -n "$name" ] && echo yes || echo no
yes

[root~]# unset name
[root~]# [ -n "$name" ] && echo yes || echo no
no
```

### 3. `字符串1 == 字符串2`：判断字符串1和字符串2是否**相等**
```
[root~]# a=123
[root~]# b=123
[root~]# c=1234

[root~]# [ "$a" == "$c" ] && echo yes || echo no
no

[root~]# [ "$a" == "$b" ] && echo yes || echo no
yes
```

### 4. `字符串1 != 字符串2`：判断字符串1和字符串2是否**不相等**

```
[root~]# a=123
[root~]# b=123
[root~]# c=1234

[root~]# [ "$a" != "$b" ] && echo yes || echo no
no

[root~]# [ "$a" != "$c" ] && echo yes || echo no
yes
```

### 七、多重条件判断

测试选项|作用
-|-
`判断1 -a 判断2`|逻辑与，判断1 和 判断2都为真，最终结果才为真
`判断1 -o 判断2`|逻辑或，判断1 和 判断2有一个为真，最终结果就为真
`! 判断`|逻辑非，使原始的判断式取反

### 1. **`判断1 -a 判断2`：逻辑与，判断1 和 判断2都为真，最终结果才为真**
```
[root~]# [ a == a -a b == b ] && echo yes || echo no
yes

[root~]# [ a == a -a a == b ] && echo yes || echo no
no

[root~]# [ a == b -a b == b ] && echo yes || echo no
no

[root~]# [ a == b -a b == a ] && echo yes || echo no
no
```

### 2. **`判断1 -o 判断2`：逻辑或，判断1 和 判断2有一个为真，最终结果就为真**
```
[root~]# [ a == a -o b == b ] && echo yes || echo no
yes

[root~]# [ a == a -o a == b ] && echo yes || echo no
yes

[root~]# [ a == b -o b == b ] && echo yes || echo no
yes

[root~]# [ a == b -o b == a ] && echo yes || echo no
no
```

### 3. **`! 判断`：逻辑非，使原始的判断式取反**

```
[root~]# [ ! a == a ] && echo yes || echo no
no
[root~]# [ ! a == b ] && echo yes || echo no
yes

```