Linux Shell编程（9）- 流程控制语句（三）循环语句
---


## 一、for循环

### 1. 语法

#### **语法1**
```
for 变量 in 值1 值2 值3 ... 值n
    do
        程序
    done
```

#### **语法2**
```
for (( 初始值;循环控制条件;变量变化 ))
    do
        程序
    done
```

### 2. 应用

#### **批量解压缩脚本**

* for_tar.sh
```
#!/bin/bash

cd /tmp/tmp

for i in $(ls *.tar.gz)
    do
        tar -zxf $i & > /dev/null
    done
```

* 为 for_tar.sh脚本添加执行权限
```
[root/tmp]# chmod +x for_tar.sh
```

* 运行脚本前
```
[root/tmp]# ll tmp/
总用量 20K
-rw-r--r-- 1 root root 167 6月   5 11:13 file1.tar.gz
-rw-r--r-- 1 root root 119 6月   5 11:12 file2.tar.gz
-rw-r--r-- 1 root root 120 6月   5 11:12 file3.tar.gz
-rw-r--r-- 1 root root 119 6月   5 11:12 file4.tar.gz
-rw-r--r-- 1 root root 120 6月   5 11:12 file5.tar.gz
```

* 运行脚本后
```
[root/tmp]# ./for_tar.sh

[root/tmp]# ll tmp/
总用量 40K
-rw-r--r-- 1 root root  43 6月   5 11:13 file1
-rw-r--r-- 1 root root 167 6月   5 11:13 file1.tar.gz
-rw-r--r-- 1 root root   5 6月   5 11:11 file2
-rw-r--r-- 1 root root 119 6月   5 11:12 file2.tar.gz
-rw-r--r-- 1 root root   5 6月   5 11:12 file3
-rw-r--r-- 1 root root 120 6月   5 11:12 file3.tar.gz
-rw-r--r-- 1 root root   5 6月   5 11:12 file4
-rw-r--r-- 1 root root 119 6月   5 11:12 file4.tar.gz
-rw-r--r-- 1 root root   5 6月   5 11:12 file5
-rw-r--r-- 1 root root 120 6月   5 11:12 file5.tar.gz
```

#### **批量创建用户**

* for_useradd.sh
```
#!/bin/bash

#批量添加指定数量的用户实例
read -t 30 -p "input user name:" name
read -t 30 -p "input password:" pass
read -t 30 -p "input user number:" num

#检查输入内容是否都为非空
if [ -z "$name" -o -z "$pass" -o -z "$num" ]
    then
        echo "Error,must be input name,password,number"
        exit 1
fi

#检查输入的用户数量是否为纯数字 
chknum=$( echo "$num" | sed 's/[0-9]//g' )
if [ -n "$chknum" ]
    then
        echo "Error,the number must be number"
        exit 2
fi

for (( i=1;i<="$num";i=i+1 ))
    do
        #添加用户
        /usr/sbin/useradd $name$i
        #添加用户密码，passwd 的--stdin参数是非交互输入，直接传入密码，不需要第二次确认
        echo $pass | /usr/bin/passwd --stdin $name$i
        echo "add $i"
    done
```

* 为 for_useradd.sh脚本添加执行权限
```
[root/tmp]# chmod +x for_useradd.sh
```

* 运行脚本
```
[root/tmp]# ./for_useradd.sh
input user name:name
input password:name
input user number:10
更改用户 name1 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 1
更改用户 name2 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 2
更改用户 name3 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 3
更改用户 name4 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 4
更改用户 name5 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 5
更改用户 name6 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 6
更改用户 name7 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 7
更改用户 name8 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 8
更改用户 name9 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 9
更改用户 name10 的密码 。
passwd：所有的身份验证令牌已经成功更新。
add 10
```

* 查看新增加的用户
```
[root/tmp]# cat /etc/passwd | grep ^name
name1:x:1002:1002::/home/name1:/bin/bash
name2:x:1003:1003::/home/name2:/bin/bash
name3:x:1004:1004::/home/name3:/bin/bash
name4:x:1005:1005::/home/name4:/bin/bash
name5:x:1006:1006::/home/name5:/bin/bash
name6:x:1007:1007::/home/name6:/bin/bash
name7:x:1008:1008::/home/name7:/bin/bash
name8:x:1009:1009::/home/name8:/bin/bash
name9:x:1010:1010::/home/name9:/bin/bash
name10:x:1011:1011::/home/name10:/bin/bash
```

#### **批量删除所有普通用户**

* for_userdel.sh
```
#!/bin/bash

user=$(cat /etc/passwd | grep /bin/bash | grep -v root | cut -d ":" -f1)

for i in $user
    do
        userdel -r $i
    done
```

* 为 for_userdel.sh脚本添加执行权限
```
[root/tmp]# chmod +x for_userdel.sh
```

* 运行脚本
```
./for_userdel.sh
```

* 查看用户
```
[root/tmp]# cat /etc/passwd | grep /bin/bash
root:x:0:0:root:/root:/bin/bash
```

#### **从 1 加到 100**

* for_sum.sh
```
#!/bin/bash

s=0
for(( i=1;i<=100;i=i+1))
    do
        s=$(($s+$i))
    done
    
echo "sum is: $s"
```

* 为 for_sum.sh 脚本添加执行权限
```
[root/tmp]# chmod +x for_sum.sh
```

* 运行脚本
```
[root/tmp]# ./for_sum.sh
sum is: 5050
```

## 二、while循环

> while 循环是不定循环，又称作条件循环。只要条件判断式成立，循环就会一直继续，直到条件判断式不成立，循环才会停止。这就和 for 的固定循环不太一样了。

> 写 while 循环时要特别注意不要写成死循环。

### 1. 语法
```
while [ 条件判断式 ]
    do
        程序
    done
```

### 2. 应用


#### **从 1 加到 100**

* while_sum.sh
```
#!/bin/bash

i=1
s=0

while [ $i -le 100 ]
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
    
echo "sum is: $s"
```

* 为 while_sum.sh 脚本添加执行权限
```
[root/tmp]# chmod +x while_sum.sh
```

* 运行脚本
```
[root/tmp]# ./while_sum.sh
sum is: 5050
```

## 三、until循环

> until 循环和 while 循环相反，until 循环时只要条件判断式不成立则进行循环，并执行循环程序。一旦循环条件成立，则终止循环。

> 写 until 循环是要特别注意不要写成死循环。

### 1. 语法
```
until [ 条件判断式 ]
    do 
        程序
    done
```

### 2. 应用

#### **从 1 加到 100**

* until_sum.sh
```
#!/bin/bash

i=1
s=0

until [ $i -gt 100 ]
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
    
echo "sum is: $s"
```

* 为 until_sum.sh 脚本添加执行权限
```
[root/tmp]# chmod +x until_sum.sh
```

* 运行脚本
```
[root/tmp]# ./until_sum.sh
sum is: 5050
```