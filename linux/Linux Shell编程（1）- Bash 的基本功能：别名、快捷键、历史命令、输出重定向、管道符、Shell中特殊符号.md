Linux Shell编程（1）- Bash 的基本功能
---

## 一、命令别名

### 1. 命令生效的顺序

1. 执行使用绝对路径或相对路径执行的命令
2. 执行别名
3. 执行Bash内部命令
4. 执行按照 `$PATH` 环境变量定义的目录顺序查找到的第一个命令

### 2. 查看别名

#### 命令格式

    alias [别名]
    
#### 实例

```
[vagrant/tmp] ]$alias
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -AlhF --color=auto'
alias ls='ls --color=auto'
alias vi='vim'
[vagrant/tmp] ]$alias ls
alias ls='ls --color=auto'
[vagrant/tmp] ]$alias cp
-bash: alias: cp: not found
```

### 3. 设置别名

#### 命令格式

    alias 别名='命令 参数'
    
#### 实例

```
[vagrant/tmp] ]$alias cat='cat -n'
[vagrant/tmp] ]$alias less='less -mN'
[vagrant/tmp] ]$alias
alias cat='cat -n'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias less='less -mN'
alias ll='ls -AlhF --color=auto'
alias ls='ls --color=auto'
alias vi='vim'
```

#### 设置别名永久生效

在上面的命令行中那样设置别名，别名只能在当前bash中使用，且一旦退出登录，别名便会失效。
若要使别名永久生效，需要将该别名添加到 `~/.bashrc` 配置文件中。

* `~/.bashrc` 文件
```
# some more ls aliases
alias grep='grep --color=auto'
alias ll='ls -AlhF --color=auto'
alias la='ls -A'
alias l='ls -CF'
alias vi='vim'
alias cat='cat -n'
alias less='less -mN'
```

### 4. 删除别名

#### 命令格式

    unalias 别名
    
#### 实例
```
[vagrant/tmp] ]$alias
alias cat='cat -n'
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
alias less='less -mN'
alias ll='ls -AlhF --color=auto'
alias ls='ls --color=auto'
alias vi='vim'
[vagrant/tmp] ]$unalias grep
[vagrant/tmp] ]$unalias cat
[vagrant/tmp] ]$alias
alias l='ls -CF'
alias la='ls -A'
alias less='less -mN'
alias ll='ls -AlhF --color=auto'
alias ls='ls --color=auto'
alias vi='vim'
```

#### 删除别名永久生效

与设置别名一样，若要永久删除别名，将该别名从 `~/.bashrc` 配置文件中删除即可。

## 二、常用快捷键

> `Ctrl + c` ：强制停止当前命令
> `Ctrl + l` ：清屏
> `Ctrl + a` ：光标移到命令行首
> `Ctrl + e` ：光标移到命令行尾
> `Ctrl + u` ：从光标所在位置删除到行首
> `Ctrl + z` ：发命令放入后台执行
> `Ctrl + r` ：在命令历史中搜索

## 三、历史命令

### 1. 命令格式

	history [选项] [历史命令保存文件]
    
### 2. 选项

> `-c` 清空历史命令
> `-w` 把缓存中的历史命令写入历史命令保存文件 `~/bash_history`

### 3. 历史保存条数

* 历史命令默认会保存1000条，可以在环境变量配置文件 `~/.bashrc` 中进行修改
```
# for setting history length see HISTSIZE and HISTFILESIZE in bash(1)
HISTSIZE=1000
HISTFILESIZE=2000
```
### 4. 历史命令的调用

* 使用上、下箭头调用以前的历史命令
* 使用 `!n` 重复执行第n条历史命令
* 使用 `!!` 重复执行上一条命令
* 使用 `!字符串` 重复执行最后一条以该字符串开头的命令

## 四、输出重定向

### 1. 标准输入与输出

设备|设备文件名|文件描述符|类型
-|-|-|-
键盘|`/dev/stdin`|0|标准输入
显示器|`/dev/stdout`|1|标准输出
显示器|`/dev/stderr`|2|错误输出

### 2. 输出重定向

#### 标准输出重定向

* `命令 > 文件`

以覆盖的方式，把命令的正确输出存储到指定的文件或设备中。

* `命令 >> 文件`

以追加的方式，把命令的正确输出存储到指定的文件或设备中。

#### 错误输出重定向

* `错误命令 2> 文件`

以覆盖的方式，把命令的错误输出存储到指定的文件或设备中。

* `错误命令 2>> 文件`

以追加的方式，把命令的错误输出存储到指定的文件或设备中。

#### 正确输出与错误输出同时保存

* `命令 > 文件 2>&1`

以覆盖的方式，把命令的正确输出和错误输出都存储到指定的文件中。

* **`命令 >> 文件 2>&1`** **（常用）**

以追加的方式，把命令的正确输出和错误输出都存储到指定的文件中。

* `命令 &> 文件`

以覆盖的方式，把命令的正确输出和错误输出都存储到指定的文件中。

* **`命令 &>> 文件`** **（常用）**

以追加的方式，把命令的正确输出和错误输出都存储到指定的文件中。

* **`命令 >> 文件1 2>> 文件2`** **（常用）**

把命令的正确输出追加到文件1中，把错误输出追加到文件2中。

```
# 将 shell.sh 运行的正确输出存储到 access.log 文件，错误输出存储到 error.log 文件
shell.sh >> access.log 2>> error.log
```

### 3. 输入重定向

* `命令 < 文件`

把文件内容作为命令的输入

```
# 在mysql中执行sql文件中的语句
mysql -uroot -p < db.sql

# 统计 access.log 文件的行数，单词书，字符数
wc < access.log
  4  24 130
# 实际上该命令不加 < 也可以执行
wc access.log
  4  24 130 access.log
```

* `命令 << 标识符 ... 标识符`
```
命令 << 标识符
...
标识符
```

将两个相同标识符之间的内容作为命令的输入。
类似PHP中的`heredoc`语法。

## 五、多命令顺序执行

### 1. 多命令顺序执行

多命令执行符|格式|作用
-|-|-
`;`|命令1 ; 命令2|多命令顺序执行，命令之间没有任何逻辑关系
`&&`|命令1 && 命令2|逻辑与<br>当命令1正确执行时，命令2才会执行<br>当命令1执行不正确时，命令2不会执行。
&#124;&#124; | 命令1 &#124;&#124; 命令2|逻辑或<br>当命令1正确执行不正确时，命令2才会执行<br>当命令1正确执行时，命令2不会执行。

```
# 根据两次日期输出的差值，计算中间压缩命令执行的时间
date; tar -zcvf etc.tar.gz /etc; date

# 根据输出 yes 还是 no，判断第一条命令是否正确执行
ls && echo yes || echo no
```

### 2. 管道符

#### 命令格式

    命令1 | 命令2

将命令1的正确输出作为命令2的操作对象

```
# 使用 less 命令查看 /etc/ 下目录或文件信息
ll /etc/ | less -mN

# 查看当前建立连接的端口数量
netstat -an | grep ESTABLISHED | wc -l

# 去掉配置文件中的注释和空行，并生成一个新的配置文件
cat /etc/redis/redis.conf | grep -v "#" | grep -v "^$" > /etc/redis/redis6379.conf

# 在/home目录下查找包含“max_children”的文件
sudo find /home -type f -name '*' | xargs grep "max_children"
```

## 六、Shell中特殊符号

### 1. 通配符

通配符|作用
-|-
`？`|匹配一个任意字符
`*`|匹配0个或任意过个任意字符，也就是可以匹配任何内容
`[]`|匹配中括号中任意一个字符。例如：`[abc]`代表匹配a/b/c中的任意一个字符
`[-]`|匹配中括号中任意一个字符。`-` 代表一个范围。例如：`[a-z]`代表匹配任意一个小写字母
`[^]`|逻辑非，匹配不是中括号中任意一个字符。例如：`[^0-9]`代表匹配任意一个不是数字的字符

```
[vagrant/tmp] ]$ll
total 0
-rw-rw-r-- 1 vagrant vagrant 0 May  3 02:21 ab1
-rw-rw-r-- 1 vagrant vagrant 0 May  3 02:21 ab2
-rw-rw-r-- 1 vagrant vagrant 0 May  3 02:21 ab3
-rw-rw-r-- 1 vagrant vagrant 0 May  3 02:21 abc
-rw-rw-r-- 1 vagrant vagrant 0 May  3 02:21 abc.log
-rw-rw-r-- 1 vagrant vagrant 0 May  3 02:21 abd
-rw-rw-r-- 1 vagrant vagrant 0 May  3 02:21 abe
[vagrant/tmp] ]$ls abc
abc
[vagrant/tmp] ]$ls abc*
abc  abc.log
[vagrant/tmp] ]$ls ab?
ab1  ab2  ab3  abc  abd  abe
[vagrant/tmp] ]$ls ab[0-9]
ab1  ab2  ab3
[vagrant/tmp] ]$ls ab[0-9a-z]
ab1  ab2  ab3  abc  abd  abe
[vagrant/tmp] ]$ls ab[^a-z]
ab1  ab2  ab3
```

### 2. Bash中其他特殊符号

符号|作用
-|-
`''`|单引号。<br>单引号中所有特殊符号，如 `$` 和 ` (反引号) 都没有特殊含义
`""`|双引号。<br>双引号中特殊符号都没有特殊含，但是 `$` 、 `` ` `` (反引号) 、和 `\` 是例外，分别拥有“调用变量的值”、“引用命令”、“转义符”的特殊含义
```` `` ````|反引号。<br>反引号中的内容是系统命令，在Bash中会先执行它。
`$()`|和反引号作用一样，用来引用系统命令。不过推荐使用`$()`，因为反引号非常容易看错。
`#`|在Shell脚本中，`#`开头的行代表注释
`$`|用于调用变量的值，如需要调用变量name的值时，需要用$name的方式得到变量的值
`\`|转义符。<br>跟在`\`之后的特殊符号将失去特殊含义，变为普通字符。如`\$`将输出`$`符号，而不是当做变量引用

---


![欢迎关注公众号【全栈札记】](https://md.s1031.cn/xsj/2021_1_4_扫码_搜索联合传播样式-白色版.png)