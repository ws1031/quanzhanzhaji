Linux Shell编程（3）- Shell 运算
---

## 一、declare 命令

> shell 变量是弱类型，且默认都是字符串型
> declare 命令可以给变量设定或取消类型

### 1. 语法

    declare [-/+][选项] [变量名]

### 2. 选项

> `-`：给变量设定类型属性
> `+`：取消变量的类型属性

> `-a`：将变量声明为数组类型（array）
> `-i`：将变量声明为整数类型（integer）

> `-x`：将变量声明为环境变量
> `-r`：将变量声明为证只读变量

> `-p`：显示指定变量的被声明的类型

### 3. 把变量声明为数值型

    declare -i [变量名]

```
[root~]# a=1
[root~]# b=2

[root~]# declare -i c=$a+$b
[root~]# echo $c
3

[root~]# declare -p c
declare -i c="3"
```

### 4. 声明数组变量

#### **定义数组**

    数组名[n]=值
    
    declare -a 数组名[n]=值

    注意，n只能是数字

#### **查看数组**
* ${数组名}：查看数组第一个元素，即array[0]的值

* ${数组名[n]}：n为数字，查看数组指定下标的值

* ${数组名[*]}：查看数组中所有的值

#### **删除数组**

* 删除数组指定下标的值

`unset 数组名[n]`

* 删除整个数组

`unset 数组名`

#### **实例**
* 定义数组下标[2]和下标[3]，没有定义下标[0]
```
[root~]# array[2]=2
[root~]# declare -a array[3]=3
```
* 查看数组第一个元素，即array[0]的值，因为没有，所以输出为空
```
[root~]# echo ${array}

```
* 查看数组下标[3]的值
```
[root~]# echo ${array[3]}
3
```
* 查看数组中所有的值
```
[root~]# echo ${array[*]}
2 3
```
* 定义数组下标[0]的值，并输出
```
[root~]# array[0]=0
[root~]# echo ${array}
0
[root~]# echo ${array[*]}
0 2 3
```
* 删除数组下标[3]
```
[root~]# unset array[3]
[root~]# echo ${array[*]}
0 2
```
* 删除整个数组
```
[root~]# unset array
[root~]# echo ${array[*]}

```

### 5. 声明环境变量

    export 变量名=变量值

    declare -x 变量名=变量值

### 6. 声明变量只读属性

> 具有只读属性的变量不能修改，不能删除，甚至不能取消只读属性！

#### **实例**

* 声明变量只读属性
```
[root~]# declare -r var="I am a readonly variable"
```
* 不能修改
```
[root~]# var=change
-bash: var: readonly variable
```
* 不能删除
```
[root~]# unset var
-bash: unset: var: cannot unset: readonly variable
```
* 不能取消只读属性
```
[root~]# declare +r var
-bash: declare: var: readonly variable
```

### 7. 查询变量的属性

#### **查询所有变量的属性**

    declare -p

#### **查询指定变量的属性**

    declare -p [变量名]

#### **实例**

```
[root~]# declare -p
declare -- BASH="/bin/bash"
...省略...
declare -ir UID="0"
declare -x USER="root"
declare -x USERNAME="root"

[root~]# declare -p BASH
declare -- BASH="/bin/bash"

[root~]# declare -p PATH
declare -x PATH="/sbin:/bin:/usr/sbin:/usr/bin"
```

## 二、数值运算的方法

### 1. `declare -i sum=$num1+$num2`

```
[root~]# num1=1
[root~]# num2=2
[root~]# declare -i sum=$num1+$num2
[root~]# echo $sum
3
```

### 2. `expr`

#### **`expr` 用法**

    sum=$(expr $num1 + $num2)

```
[root~]# num1=1
[root~]# num2=2
[root~]# sum=$(expr $num1 + $num2)
[root~]# echo $sum
3
```
* `expr` 运算符中间必须有空格，否则：
```
[root~]# sum=$(expr $num1+$num2)
[root~]# echo $sum
1+2
```
### 3. `let`

#### **`let` 用法**

    let [变量名]=[运算式]

```
[root~]# let sum=$num1+$num2
[root~]# echo $sum
3
[root~]# let num=(1+2)*3/4+5
[root~]# echo $num
7
```

### 4. `$((运算式))` 或 `$[运算式]` **（常用）**

#### **`$((运算式))`**

```
[root~]# a=1
[root~]# b=2
[root~]# c=$(($a+$b))
[root~]# echo $c
3
```
* 运算符中间也可以加空格
```
[root~]# c=$(( $a + $b ))
[root~]# echo $c
3
```
#### **`$[运算式]`**
```
[root~]# c=$[$a+$b]
[root~]# echo $c
3
```
* 运算符中间也可以加空格
```
[root~]# c=$[ $a + $b ]
[root~]# echo $c
3
```

### 5. 运算符优先级

#### **运算符优先级从高到低排序**

运算符|说明
-|-
`-` , `+`|单目负，单目正
`!` , `~`|逻辑非，按位取反或补码
`*` , `/`, `%`|乘，除，取余
`+` , `-`|加，减
`<<` , `>>`|按位左移，按位右移
`<` , `>` , `<=` , `>=`|小于，大于，小于等于，大于等于
`==` , `!=`|等于，不等于
`&`|按位与
`^`|按位异或
&#124;|按位或
`&&`|逻辑与
&#124;&#124;|逻辑或
`=` , `+=` , `-=` , `*=` , `/=` , `%=` ,<br>`&=` , `^=` , &#124;= , `<<=` , `>>=`|赋值、运算且赋值

> 优先级相同的，从左到右运算。
> 有小括号的，先算小括号里面的。

#### 实例

* 加。减、乘、除、取余、小括号

```
[root~]# a=$(( 4 + 5 * 3 / 2 ))
[root~]# echo $a
11
[root~]# b=$(( (4 + 5) * 3 / 2 ))
[root~]# echo $b
13
[root~]# c=$(( (4 + 5) * 3 % 5 ))
[root~]# echo $c
2
[root~]# d=$(( (4 + 5) * 3 / (2 + 1) ))
[root~]# echo $d
9
```
* 按位与
```
[root~]# e=$(( 1 & 0 ))
[root~]# echo $e
0
```
* 按位或
```
[root~]# f=$(( 1 | 0 ))
[root~]# echo $f
1
```
* 按位异或
```
[root~]# g=$(( 1 ^ 2 ))
[root~]# echo $g
3
```

## 三、变量测试

> 变量测试一般在脚本优化时使用。

> 格式复杂多样，语法简单。

### 1. 变量测试表

![图片描述][1]

### 2. 实例

#### **以变量测试表第一行 `{x=${y-n}}` 为例**

* 变量y没有设置时，x=n
```
[root~]# unset y
[root~]# x=${y-1}
[root~]# echo $x
1
```
* 变量y为空值时，x=空值
```
[root~]# y=''
[root~]# x=${y-1}
[root~]# echo $x
```
* 变量y有值时，x=$y
```
[root~]# y=2
[root~]# x=${y-1}
[root~]# echo $x
2
```


  [1]: /img/bVbbrdM