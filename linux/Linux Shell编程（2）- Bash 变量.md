Linux Shell编程（2）- Bash 变量
---

## 一、简介与分类

### 1. Bash 变量的命名规则

> * 变量名只能包含字母、数字、下划线
> * 变量名不能以数字作为开头
> * 变量名区分大小写
> * 变量名长度不超过255个字符
> * 变量名在有效范围内必须唯一

### 2. 变量按照存储的数据类型分类

> 在Bash中，变量的默认类型都是字符串型

* 字符串型
* 整型
* 浮点型
* 日期型

### 3. 变量的分类

#### **用户自定义变量**

* 用户自己定义的变量名

#### **环境变量**

* 环境变量主要保存的是和系统操作环境相关的数据。
* 变量可以自定义，但是对系统生效的环境变量名和变量作用是固定的。

#### **位置参数变量**

* 位置参数变量主要是用来向脚本当中传递参数或数据的。
* 变量名不能自定义，变量作用是固定的。

#### **预定义变量**

* 预定义变量是 bash 中已经定义好的便令。
* 变量名不能自定义，变量作用是固定的。


## 二、用户自定义变量

### 1. 定义变量

#### **语法**

    变量名="变量值"

#### **说明**

* 等号两边不能有空格
* 当变量值中间有空格时，要用引号包裹变量值
* 变量的默认类型都是字符串型
* 变量只在当前的 bash 中生效

#### **实例**
```
[root~]# age=18
[root~]# name="zhang san"
```

### 2. 变量调用

#### **语法**

    $变量名

#### **说明**

* 调用变量需要在变量名前加 `$`
* 变量的默认类型都是字符串型，所以无法直接做 `+`, `-` 等运算操作
* 赋值时引用变量，可使用 `"$变量名"` 或 `${变量名}`
* 赋值时引用变量，若要在值两边使用引号，则必须使用`双引号`，若使用单引号，则单引号内的变量不会转换成变量值

#### **实例**
* 调用变量需要在变量名前加 `$`
```
[root~]# echo $name
zhang san
```
* 变量的默认类型都是字符串型，所以无法直接做 `+`, `-` 等运算操作
```
[root~]# x=1
[root~]# y=2
[root~]# z=$x+$y
[root~]# echo $z
1+2
```
* 赋值时引用变量，可使用 `"$变量名"` 或 `${变量名}`
```
[root~]# a=hel
[root~]# b="$a"lo
[root~]# echo $b
hello
```
* 当变量值中间有空格时，要用引号包裹。赋值时引用变量，必须使用`双引号`
```
[root~]# c="${b} world"
[root~]# echo $c
hello world
```
* 赋值时引用变量，若使用单引号，则单引号内的变量不会转换成变量值
```
[root~]# d='${b} world'
[root~]# echo $d
${b} world
```

### 3. 变量叠加

#### **实例**

```
[root~]$x=123
[root~]$echo $x
123

[root~]$x=${x}456
[root~]$echo $x
123456

[root~]$x="$x"789
[root~]$echo $x
123456789
```

### 4. 变量查看

#### **语法**

    set [选项]

#### **选项**

> `-u`：如果设定此项，调用未声明的变量时会报错（默认无任何提示）

#### **实例**
* `m` 赋值为空字符串，`n` 未定义。默认情况下，echo `$m` 和 `$n`，都没有任何提示
```
[root~]# m=''
[root~]# echo m
m
[root~]# echo $m

[root~]# echo $n
```
* `set -u` 后，`echo $n` 会报错
```
[root~]# set -u
[root~]# echo $m

[root~]# echo $n
bash: n: unbound variable
```

### 5. 变量删除

#### **语法**

    unset 变量名

#### 实例
```
[root~]# set -u

[root~]# x=hello
[root~]# echo $x
hello

[root~]# unset x
[root~]# echo $x
bash: x: unbound variable
```

## 三、环境变量

### 1. 环境变量和用户自定义变量的区别

> 环境变量是全局变量；
> 用户自定义变量是局部变量。

> 用户自定义变量只在当前shell中生效；
> 环境变量在当前shell和这个shell的所有自shell中生效。

### 2. 设置环境变量

    export 变量名=变量值

或

    变量名=变量值
    export 变量名

#### **实例**

```
[root~]# export xxx=env
[root~]# env | grep xxx
xxx=env

[root~]# yyy=env
[root~]# export yyy
[root~]# env | grep yyy
yyy=env
[root~]# yyy=yyy
[root~]# env | grep yyy
yyy=yyy
```

### 3. 查看环境变量

    set：查看所有变量

    env：只查看环境变量

#### **实例**

```
[root~]# env
XDG_SESSION_ID=2
HOSTNAME=10.0.2.15
SHELL=/bin/bash
TERM=cygwin
HISTSIZE=1000
OLDPWD=/home/vagrant
USER=root
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00 ... 36:*.xspf=00;36:
SUDO_USER=vagrant
SUDO_UID=1001
USERNAME=root
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAIL=/var/spool/mail/vagrant
PWD=/root
LANG=en_US.UTF-8
SHLVL=1
SUDO_COMMAND=/bin/su
HOME=/root
LOGNAME=root
LESSOPEN=||/usr/bin/lesspipe.sh %s
SUDO_GID=1001
_=/bin/env
```

### 4. 删除环境变量

    unset 变量名

> 环境变量在当前shell和这个shell的所有自shell中生效。在子shell中删除或者修改环境变量，并不能影响父shell中的环境变量

#### **实例**
* 定义环境变量 `xxx` 和 `yyy`
```
[root~]# export xxx=xxx
[root~]# export yyy=yyy
[root~]# env
xxx=xxx
yyy=yyy
```
* 进入子shell，环境变量 `xxx` 和 `yyy` 依然有效
```
[root~]# bash
[root~]# env
xxx=xxx
yyy=yyy
... 省略其他环境变量 ...
```
* 在子shell中删除 `xxx` ，并修改 `yyy` 的值 
```
[root~]# unset xxx
[root~]# yyy=zzz
[root~]# env
yyy=zzz
... 省略其他环境变量 ...
```
* 回到父shell，`xxx` 和 `yyy` 的值没受影响
```
[root~]# exit
[root~]# env
xxx=xxx
yyy=yyy
... 省略其他环境变量 ...
```

### 5. 常用的环境变量

环境变量|说明
-|-
HOSTNAME|主机名
SHELL|当前的shell
TERM|终端环境
HISTSIZE|历史命令条数
SSH_CLIENT|当前操作环境是使用ssh连接的，这里记录客户端IP
SSH_TTY|ssh连接的终端 pts/1
USER|当前登录的用户
PATH|系统查找命令的路径
PS1|命名提示符设置。env命令不会输出该环境变量
```
HOSTNAME=10.0.2.15
SHELL=/bin/bash
TERM=cygwin
HISTSIZE=1000
SSH_CLIENT=10.0.2.2 8432 22
SSH_TTY=/dev/pts/0
USER=vagrant
PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin
```

#### **PATH 环境变量**

    系统查找命令的路径

* 查看 PATH 环境变量

```
[root~]# echo $PATH
/sbin:/bin:/usr/sbin:/usr/bin
```

* 增加 PATH 环境变量
```
[root~]# PATH="$PATH":/usr/share/bin
[root~]# echo $PATH
/sbin:/bin:/usr/sbin:/usr/bin:/usr/share/bin
```
> 注意这种修改方法只在当前shell中生效，若想永久生效，必须修改环境变量配置文件。

#### **PS1 环境变量**

    命令提示符设置

* 常用提示符说明

> `\d`：显示日期，格式为 “星期 月 日”
> `\H`：显示完整的主机名，如 “localhost”
> `\t`：显示24小时制时间，格式为 “HH:MM:SS”
> `\A`：显示24小时制时间，格式为 “HH:MM”
> `\u`：显示当前用户名
> `\w`：显示当前所在目录的完整名称
> `\W`：显示当前所在目录的最后一个目录
> `\$`：提示符，如果是root用户会显示“#”，普通用户会显示“$”

* 实例

```
[root~]# PS1='[\d]'
[Mon May 28]PS1='[\H]'
[10.0.2.15]PS1='[\t]'
[09:51:09]PS1='[\A]'
[09:51]PS1='[\u]'
[root]PS1='[\w]'
[~]PS1='[\W]'
[~]PS1='[\$]'
[#]PS1='[\[\e[1;31m\]\u\[\e[1;34m\]\w\[\e[0m\]]\$ '
[root~]# 
```
> 注意这种修改方法只在当前shell中生效，若想永久生效，必须修改环境变量配置文件。

### 6. 语系变量

#### **查看当前系统的语系**

* `locale`

> LANG：定义系统主语系的变量
> LC_ALL：定义整体语系的变量

* 实例
```
[root~]# locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

#### **语系变量 `LANG`**

* 查看系统当前语系
```
[root~]# echo $LANG
en_US.UTF-8
```
* 查看系统支持的所有语系

```
[root~]# locale -a | more
aa_DJ
aa_DJ.iso88591
aa_DJ.utf8
aa_ER
...省略n行...
ar_SY
ar_SY.iso88596
ar_SY.utf8
ar_TN
ar_TN.iso88596
--More--
```

#### **查询系统默认语系**

    /etc/locale.conf    # CentOS7
    
    /etc/sysconfig/i18n  # CentOS6

* 实例
```
[root~]# cat /etc/locale.conf
LANG="en_US.UTF-8"
```

## 四、位置参数变量

### 1. 常用位置参数变量

位置参数变量|作用
-|-
`$n`|n为数字，`$0`代表命令本身，`$1`-`$9`代表第一个到第九个参数，十以上的参数需要用大括号括起来，如 `${10}`
`$*`|这个变量代表命令行中所有的参数，`$*`把所有的参数看成一个整体
`$@`|这个变量代表命令行中所有的参数，`$@`把每个参数区分对待
`$#`|这个变量代表命令行中参数的个数

### 2. `$n`

* 创建一个加法运算的shell脚本

```
[root/tmp]# vim eg1.sh

#!/bin/bash

num1=$1
num2=$2

# 变量sum是num1加num2的和
sum=$(( $num1 + $num2 ))

# 输出变量sum的值
echo $sum
```

* 运行shell脚本
```
[root/tmp]# chmod a+x eg1.sh

[root/tmp]# ./eg1.sh 3 2
5
```
### 3. `$0`，`$*`，`$@`，`$#`

```
[root/tmp]# vim eg2.sh

#!/bin/bash

echo "\$0 命令是：$0"

echo "\$* 参数是：$*"

echo "\$@ 参数是：$@"

echo "\$# 参数个数是：$#"

```

* 运行shell脚本
```
[root/tmp]# chmod a+x eg2.sh

[root/tmp]# ./eg2.sh 1 2 3 4 5
$0 命令是：./eg2.sh
$* 参数是：1 2 3 4 5
$@ 参数是：1 2 3 4 5
$# 参数个数是：5

[root/tmp]# ./eg2.sh 7 8 9
$0 命令是：./eg2.sh
$* 参数是：7 8 9
$@ 参数是：7 8 9
$# 参数个数是：3
```
### 4. `$*` 与 `$@` 的区别

```
[root/tmp]# vim eg3.sh

#!/bin/bash

echo "\$*中的所有参数看成是一个整体，所以这个for循环只会循环一次"

for m in "$*"
    do
        echo "参数是：$m"
    done

echo -e "\n--------------------------------------------------------\n"

echo "\$@中的每个参数都看成是独立的，所以“$@”中有几个参数，就会循环几次"

for n in "$@"
    do
        echo "参数是：$n"
    done

```

* 运行shell脚本
```
[root/tmp]# chmod a+x eg3.sh

[root/tmp]# ./eg3.sh 1 2 3 4 5
$*中的所有参数看成是一个整体，所以这个for循环只会循环一次
参数是：1 2 3 4 5

--------------------------------------------------------

$@中的每个参数都看成是独立的，所以“1 2 3 4 5”中有几个参数，就会循环几次
参数是：1
参数是：2
参数是：3
参数是：4
参数是：5
```

## 五、预定义变量

### 1. 常用预定义变量

预定义变量|作用
-|-
`$?`|最后一次执行的命令的返回状态。<br>如果这个变量的值为0，说明上一个命令正确执行；<br>如果这个变量的值为非0（具体数值，有命令自己决定），说明上一个命令执行不正确。
`$$`|当前进程的进程号（PID）
`$!`|后台运行的最后一个进程的进程号（PID）

### 2. `$?`：最后一次执行的命令的返回状态

```
[root/tmp]# ls
eg1.sh  eg2.sh  eg3.sh
[root/tmp]# echo $?
0

[root/tmp]# cat eg
cat: eg: No such file or directory
[root/tmp]# echo $?
1

[root/tmp]# lll
bash: lll: command not found
[root/tmp]# echo $?
127
```

### 3. `$$`：当前进程的进程号（PID）

```
[root/tmp]# echo $$
4311
```

```
[root/tmp]# vim bl.sh

#!/bin/bash

echo "当前进程PID：$$"


[root/tmp]# chmod a+x bl.sh

[root/tmp]# ./bl.sh
当前进程PID：4382

```


### 4. `$!`：后台运行的最后一个进程的进程号（PID）



```
[root/tmp]# find / -name hello &
[1] 4408

[root/tmp]# echo $!
4408

[root/tmp]# echo $!
4408
[1]+  Done                    find / -name hello
```

## 六、`read` 接收键盘输入

### 1. 命令

    read [选项] [变量名]

### 2. 选项

> `-p 提示信息`：在等待用户输入时，输出的提示信息
> `-t 秒数`：指定等待用户输入的时长（秒），不设置的话会一直等待用户输入
> `-n 字符数`：最多输入字符数，当用户输入指定字符数时，就会自动执行
> `-s`：隐藏用户输入的数据，适用于保密信息的输入

### 3. 实例

```
[root/tmp]# vim read.sh

#!/bin/bash

# 等待30秒
read -p '请输入你的用户名：' -t 30 name
echo $name

# 隐藏输入内容
read -p '请输入你的密码：' -s passwd
echo -e '\n'
echo $passwd

# 最多只能输入一个字符
read -p '请输入你的性别 [M/F]：' -n 1 sex
echo -e '\n'
echo $sex
```

```
[root/tmp]# ./read.sh
请输入你的用户名：Mr.wang
Mr.wang
请输入你的密码：

123456
请输入你的性别 [M/F]：M

M
```