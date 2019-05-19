Linux Shell编程（8）- 流程控制语句（二）分支条件语句
---

## 一、单分支if语句

### 1. 语法
```
if [ 条件判断式 ];then
    程序
fi
```
或者
```
if [ 条件判断式 ]
    then
    程序
fi
```

#### **单分支条件语句需要注意**

* `if` 语句使用 `fi` 结尾，和一般语言使用大括号结尾不同
* `[ 条件判断式 ]` 就是使用 `test` 命令判断，所以中括号和条件判断式之间必须有空格
* `then` 后面跟符合条件之后执行的程序，可以放在 `[]` 之后，用 `;` 分隔。也可以换行输入，就不需要 `;` 了

### 2. 应用

#### **判断登录用户是否是root**

* 创建if1.sh脚本文件，文件内容如下

```
#!/bin/bash

#env是linux的一个外部命令，可以显示当前用户的环境变量，其中一行显示当前用户
currentUser=$(env | grep "^USER=" | cut -d "=" -f 2)

if [ "$currentUser" == "root" ]
    then
    echo "Current user is root."
fi
```

* 为 if1.sh脚本添加执行权限
```
[root/tmp]# chmod +x if1.sh
```

* root 用户执行此脚本
```
[root/tmp]# ./if1.sh
Current user is root.
```

* 非 root 用户执行此脚本
```
[vagrant/tmp]$ ./if1.sh
```

#### **判断分区使用率**
```
#!/bin/bash

# 统计根分区使用率
# 把根分区使用率作为变量值赋予 rate
rate=$(df -h | grep '/dev/sda1' | awk '{print $5}' | cut -d '%' -f 1)

# 判断使用率是否大于 80
if [ "$rate" -gt 80 ]
    then
    echo "Warning! 分区使用率过高！"
fi
```

## 二、双分支if语句
### 1. 语法

```
if [ 条件判断式 ]
    then
        条件成立时执行的程序
    else
        条件不成立时执行的程序
fi
```

### 2. 应用

#### **判断输入内容是否是一个目录**

* 创建if2.sh脚本文件，文件内容如下

```
#!/bin/bash

#定义输入的变量 dir 用read -t 等待时间 -p "提示信息" 变量名 定义输入的变量
read -t 30 -p "Please input you dir :" dir

#[ -d "$dir" ] 判断变量是否是目录
if [ -d "$dir" ]
    then
        #如果是目录则执行程序
        echo "Your input is a dir"
    else
        #如果不是目录这执行这个程序
        echo "Your input is not a dir"
fi #结束if
```

* 为 if2.sh脚本添加执行权限
```
[root/tmp]# chmod +x if2.sh
```

* 判断输入内容是否是一个目录
```
[root/tmp]# ./if2.sh
Please input you dir :/root
Your input is a dir

[root/tmp]# ./if2.sh
Please input you dir :abc
Your input is not a dir
```

#### **判断 apache 是否启动，没启动则启动它**
```
#!/bin/bash

# 截取httpd进程，并把结果赋予变量httpd
httpd=$(ps aux | grep 'httpd' | grep -v 'grep')

# 判断 httpd 是否为空
if [ -n "$httpd" ]
    then
        # 若不为空，则说明apache正常运行
        echo "$(date) httpd is ok" >> /tmp/autohttpd_ok.log
    else
        # 若为空，则说明apache停止，启动 apache
        /etc/rc.d/init.d/httpd start &>> /tmp/autohttpd_err.log
        echo "$(date) restart httpd" >> /tmp/autohttpd_err.log
fi
```

## 三、多分支if语句

### 1. 语法

```
if [ 条件判断式1 ]
    then
        条件判断式1成立时执行的程序
elif [ 条件判断式2 ]
    then
        条件判断式2成立时执行的程序
    
    ... 省略更多条件 ...
    
else
    当所有条件都不成立时，最后执行的程序
fi
```

### 2. 应用

#### **判断用户输入的是什么类型的文件**

* file_type.sh
```
#!/bin/bash

# 接收用户输入的文件名，赋值给file
read -p "Please input your file: " file

# 判断 $file 是否为空
if [ -z "$file" ]
    then
        echo "Error,Input is NULL!"
        exit 1
elif [ ! -e "$file" ]
    then
        echo "$file is not a file!"
        exit 2
elif [ -f "$file" ]
    then
        echo "$file is a regular file!"
elif [ -d "$file" ]
    then
        echo "$file is a directory!"
else
    echo "$file is an other file!"
fi
```

* 为 file_type.sh脚本添加执行权限
```
[root/tmp]# chmod +x file_type.sh
```

* 输入为空，返回错误输出，查看错误码为1
```
[root/tmp]# ./file_type.sh
Please input your file:
Error,Input is NULL!

[root/tmp]# echo $?
1
```

* 输入不是文件，返回错误输出，查看错误码为2
```
[root/tmp]# ./file_type.sh
Please input your file: abc
abc is not a file!

[root/tmp]# echo $?
2
```

* 输入是一个普通文件
```
[root/tmp]# ./file_type.sh
Please input your file: file_type.sh
file_type.sh is a regular file!

[root/tmp]# echo $?
0
```

* 输入是一个目录
```
[root/tmp]# ./file_type.sh
Please input your file: /tmp
/tmp is a directory!

[root/tmp]# echo $?
0
```

* 输入是一个其他文件
```
[root/tmp]# ./file_type.sh
Please input your file: /dev/null
/dev/null is an other file!

[root/tmp]# echo $?
0
```

#### **命令行界面加减乘除计算器**

* calculator.sh
```
#!/bin/bash

# 通过read命令接收用户输入要计算的数值，并赋值给 num1 和 num2
read -p "please input num1: " -t 30 num1
read -p "please input num2: " -t 30 num2

# 通过read命令接收用户输入要计算的运算符号，并赋值给 op
read -p "please input operator: " -t 30 op

# 判断 $num1，$num2 和 $op 都不能为空
if [ -z "$num1" -o -z "$num2" -o -z "$op" ]
    then
        echo "Error: the input cannot be null";
        exit 10
fi

# 判断 $num1 和 $num2 是否都为数字
# 将 $num1 和 $num2 的数字部分全部替换为空，将替换结果赋值给 tmp1 和 tmp2
tmp1=$(echo $num1 | sed 's/[0-9]//g')
tmp2=$(echo $num2 | sed 's/[0-9]//g')

# 判断 tmp1 和 tmp2 是否都为空，都为空则全都是数字
if [ -n "$tmp1" -o -n "$tmp2" ]
    then
        echo "Error: the num1 and num2 must be number"
        exit 11
fi

# 加法运算
if [ "$op" == "+" ]
    then
        result=$(($num1 + $num2))
# 减法运算
elif [ "$op" == "-" ]
    then
        result=$(($num1 - $num2))
# 乘法运算
elif [ "$op" == "*" ]
    then
        result=$(($num1 * $num2))
# 除法运算
elif [ "$op" == "/" ]
    then
        # 除法运算中除数不能为0
        if [ $num2 -eq 0 ]
            then
                echo "Error: The divisor cannot be 0"
                exit 12
            else
                result=$(($num1 / $num2))
        fi
# 运算符不是 + | - | * | /
else
    echo "Error: the operator must be + | - | * | /"
    exit 13
fi

# 输出运算结果
echo "$num1 $op $num2 = $result"
```

* 为 calculator.sh 脚本添加执行权限
```
[root/tmp]# chmod +x calculator.sh
```

* 输入num1和num2为空，返回错误输出，查看错误码为10
```
[root/tmp]# ./calculator.sh
please input num1:
please input num2:
please input operator: +
Error: the input cannot be null

[root/tmp]# echo $?
10
```

* 输入num1不是数字，返回错误输出，查看错误码为11
```
[root/tmp]# ./calculator.sh
please input num1: a
please input num2: 1
please input operator: +
Error: the num1 and num2 must be number

[root/tmp]# echo $?
11
```

* 输入除数为0，返回错误输出，查看错误码为12
```
[root/tmp]# ./calculator.sh
please input num1: 2
please input num2: 0
please input operator: /
Error: The divisor cannot be 0

[root/tmp]# echo $?
12
```

* 输入运算符号 #，返回错误输出，查看错误码为12
```
[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 2
please input operator: #
Error: the operator must be + | - | * | /

[root/tmp]# echo $?
13
```

* 正常的加减乘除运算
```
[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 2
please input operator: +
1 + 2 = 3

[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 2
please input operator: -
1 - 2 = -1

[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 2
please input operator: *
1 * 2 = 2

[root/tmp]# ./calculator.sh
please input num1: 6
please input num2: 3
please input operator: /
6 / 3 = 2
```

## 四、多分支case条件语句

> `case` 语句和 `if...elif...else` 语句一样都是多分支条件语句。
> 不过和 `if` 多分支条件语句不同的是，`case` 语句只能判断一种条件关系，而 `if` 语句可以判断多种条件关系。

### 1. 语法

```
case $变量名 in
    "值1")
        变量的值等于值1时，执行程序1
        ;;
    "值2")
        变量的值等于值2时，执行程序2
        ;;
    
    ...省略其他分支...
    
    "值n")
        变量的值等于值n时，执行程序n
        ;;
    *)
        变量的值都不等于以上的值时，执行此程序
        ;;
esac
```

> 注意，最后的 `*)` 中 `*` 不加双引号

### 2. 应用

#### **使用case重写计算器**

* calculator.sh
```
#!/bin/bash

# 通过read命令接收用户输入要计算的数值，并赋值给 num1 和 num2
read -p "please input num1: " -t 30 num1
read -p "please input num2: " -t 30 num2

# 通过read命令接收用户输入要计算的运算符号，并赋值给 op
read -p "please input operator: " -t 30 op

# 判断 $num1，$num2 和 $op 都不能为空
if [ -z "$num1" -o -z "$num2" -o -z "$op" ]
    then
        echo "Error: the input cannot be null";
        exit 10
fi

# 判断 $num1 和 $num2 是否都为数字
# 将 $num1 和 $num2 的数字部分全部替换为空，将替换结果赋值给 tmp1 和 tmp2
tmp1=$(echo $num1 | sed 's/[0-9]//g')
tmp2=$(echo $num2 | sed 's/[0-9]//g')

# 判断 tmp1 和 tmp2 是否都为空，都为空则全都是数字
if [ -n "$tmp1" -o -n "$tmp2" ]
    then
        echo "Error: the num1 and num2 must be number"
        exit 11
fi

case $op in
    # 加法运算
    "+")
        result=$(($num1 + $num2))
        ;;
    # 减法运算
    "-")
        result=$(($num1 - $num2))
        ;;
    # 乘法运算
    "*")
        result=$(($num1 * $num2))
        ;;
    # 除法运算
    "/")
        # 除法运算中除数不能为0
        if [ $num2 -eq 0 ]
            then
                echo "Error: The divisor cannot be 0"
                exit 12
            else
                result=$(($num1 / $num2))
        fi
        ;;
    # 运算符不是 + | - | * | /
    *)
        echo "Error: the operator must be + | - | * | /"
        exit 13
        ;;
esac

# 输出运算结果
echo "$num1 $op $num2 = $result"
```

* 正常的加减乘除运算
```
[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 2
please input operator: +
1 + 2 = 3

[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 2
please input operator: -
1 - 2 = -1

[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 2
please input operator: *
1 * 2 = 2

[root/tmp]# ./calculator.sh
please input num1: 6
please input num2: 3
please input operator: /
6 / 3 = 2
```

* 输入非运算符时，输出错误信息
```
[root/tmp]# ./calculator.sh
please input num1: 1
please input num2: 1
please input operator: %
Error: the operator must be + | - | * | /
```