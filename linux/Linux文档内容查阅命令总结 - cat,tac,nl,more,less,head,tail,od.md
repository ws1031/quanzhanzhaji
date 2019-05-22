Linux文档内容查阅命令总结 - cat,tac,nl,more,less,head,tail,od
---

# Linux文档内容查阅命令总结

    cat  由第一行开始显示文档内容
    tac  从最后一行开始显示文档内容，可以看出 tac 是 cat 的倒着写！
    nl   显示的时候，顺道输出行号！
    more 一页一页的显示档案内容
    less 与 more 类似，但是比 more 更好的是，他可以往前翻页！
    head 只看头几行
    tail 只看尾巴几行
    od   以二进位的方式读取档案内容！
    
## 一、直接查阅文档内容

    直接查阅一个档案的内容可以使用 cat/tac/nl 这几个指令。
    
### 1. `cat` (由第一行开始显示文档内容)

    cat [选项] [文件名]
    
#### 选项

    -A  ：相当于 -vET 的整合选项，可列出一些特殊字符而不是空白而已；
    -b  ：列出行号，仅针对非空白行做行号显示，空白行不标行号！
    -E  ：将结尾的断行字元 $ 显示出来；
    -n  ：列印出行号，连同空白行也会有行号，与 -b 的选项不同；
    -T  ：将 [tab] 按键以 ^I 显示出来；
    -v  ：列出一些看不出来的特殊字符
    
#### 实例

* 查看文档内容

```
[vagrant/tmp/abc] ]$cat hosts
127.0.0.1       localhost
127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

* 输出时附带行号，空号不编号

```
[vagrant/tmp/abc] ]$cat -b hosts
     1  127.0.0.1       localhost
     2  127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty

     3  # The following lines are desirable for IPv6 capable hosts
     4  ::1     localhost ip6-localhost ip6-loopback
     5  ff02::1 ip6-allnodes
     6  ff02::2 ip6-allrouters
```

* 输出时附带行号，空号也编号

```  
[vagrant/tmp/abc] ]$cat -n hosts
     1  127.0.0.1       localhost
     2  127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
     3
     4  # The following lines are desirable for IPv6 capable hosts
     5  ::1     localhost ip6-localhost ip6-loopback
     6  ff02::1 ip6-allnodes
     7  ff02::2 ip6-allrouters
```

* 显示文档中的空白字符，将结尾的断行字元 $ 显示出来，将 [tab] 按键以 ^I 显示出来

```
[vagrant/tmp/abc] ]$cat -A hosts
127.0.0.1^Ilocalhost$
127.0.1.1^Ivagrant-ubuntu-trusty.vagrantup.com^Ivagrant-ubuntu-trusty$
$
# The following lines are desirable for IPv6 capable hosts$
::1     localhost ip6-localhost ip6-loopback$
ff02::1 ip6-allnodes$
ff02::2 ip6-allrouters$
```

### 2. `tac` (从最后一行开始显示文档内容，tac 是 cat 的倒着写)

    tac [文件名]
    
#### 实例

* 从最后一行开始显示文档内容

```
[vagrant/tmp/abc] ]$tac hosts
ff02::2 ip6-allrouters
ff02::1 ip6-allnodes
::1     localhost ip6-localhost ip6-loopback
# The following lines are desirable for IPv6 capable hosts

127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
127.0.0.1       localhost
```

### 3. `nl` (输出附带行号的文档内容)

nl 可以将输出文件内容自动加上行号！其默认结果与 `cat -n` 有点不太一样， nl 可以将行号做比较多的显示设计，包括位数与是否自动补 0 等功能。

    nl [选项] [文件名]
    
#### 选项

    -b  ：指定行号指定的方式，主要有两种：
          -b t ：如果有空行，空的那一行不列出行号；(默认)
          -b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)；
    -n  ：列出行号表示的方法，主要有三种：
          -n ln ：行号在荧幕的最左方显示；
          -n rn ：行号在自己栏位的最右方显示，且不加 0 ；(默认)
          -n rz ：行号在自己栏位的最右方显示，且加 0 ；
    -w  ：行号栏位的占用的位元数。
    
#### 实例

* 输出文档内容，显示行号，空行不编号。类似于 `cat -b`
```
[vagrant/tmp/abc] ]$nl hosts
     1  127.0.0.1       localhost
     2  127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty

     3  # The following lines are desirable for IPv6 capable hosts
     4  ::1     localhost ip6-localhost ip6-loopback
     5  ff02::1 ip6-allnodes
     6  ff02::2 ip6-allrouters
```

* 输出文档内容，显示行号，空行也编号。类似于 `cat -n`
```
[vagrant/tmp/abc] ]$nl -b a hosts
     1  127.0.0.1       localhost
     2  127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
     3
     4  # The following lines are desirable for IPv6 capable hosts
     5  ::1     localhost ip6-localhost ip6-loopback
     6  ff02::1 ip6-allnodes
     7  ff02::2 ip6-allrouters
```

* 行号靠最左边显示
``` 
[vagrant/tmp/abc] ]$nl -b a -n ln hosts
1       127.0.0.1       localhost
2       127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
3
4       # The following lines are desirable for IPv6 capable hosts
5       ::1     localhost ip6-localhost ip6-loopback
6       ff02::1 ip6-allnodes
7       ff02::2 ip6-allrouters
```

* 行号靠最右方显示，且前面补 0
```
[vagrant/tmp/abc] ]$nl -b a -n rz hosts
000001  127.0.0.1       localhost
000002  127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
000003
000004  # The following lines are desirable for IPv6 capable hosts
000005  ::1     localhost ip6-localhost ip6-loopback
000006  ff02::1 ip6-allnodes
000007  ff02::2 ip6-allrouters
```

* 指定行号栏位的占用的位数为 3
```
[vagrant/tmp/abc] ]$nl -b a -n rz -w 3 hosts
001     127.0.0.1       localhost
002     127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
003
004     # The following lines are desirable for IPv6 capable hosts
005     ::1     localhost ip6-localhost ip6-loopback
006     ff02::1 ip6-allnodes
007     ff02::2 ip6-allrouters
```

## 二、翻页查看文档内容

more 和 less 都有翻页查看文档内容的功能，且都能搜索。
但是 less 的功能要比 more 更强大，而且 less 搜索字符串时会高亮显示要搜索的字符串（more竟然不会高亮）。

### 1. `more`

    more [文件名]
    
#### 操作

    空格键 (Space) ：代表向下翻一页；
    Enter         ：代表向下翻‘一行’；
    /字符串         ：代表在这个显示的内容当中，向下搜寻‘字符串’这个关键字，按回车开始查找，按 n 查找下一个匹配；
    :f            ：立刻显示出文件名以及目前显示的行数；
    q             ：代表立刻离开 more ，不再显示该档案内容。
    b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管道无用。
    
### 2.  `less`

    less [文件名]
    
#### 参数

    -N ：显示行号
    -m ：显示类似more命令的百分比
    
#### 操作

    空格键 (Space)  ：向下翻动一页；
    [pagedown]      ：向下翻动一页；
    [pageup]        ：向上翻动一页；
    /字符串         ：向下搜寻‘字符串’的功能；
    ?字符串         ：向上搜寻‘字符串’的功能；
    n               ：重复前一个搜寻 (与 / 或 ? 有关！)
    N               ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
    q               ：离开 less 这个程式；
    
## 三、查看文档部分内容

我们可以将输出的数据做一个最简单的选取，那就是取出前面 (head) 与取出后面 (tail) 文字的功能。 不过，要注意的是， head 与 tail 都是以‘行’为单位来进行数据选取的！

### 1. `head` (取出前面几行)

    head [-n number] 文件名
    
#### 参数

    -n  ：后面接数字，代表显示几行的意思。默认显示前10行。
    
#### 实例

* 显示前 5 行， `head -n 5` 与 `head -5`
```
[vagrant/tmp/abc] ]$head -n 5 hosts
127.0.0.1       localhost
127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
[vagrant/tmp/abc] ]$head -5 hosts
127.0.0.1       localhost
127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
```

* 显示除去后5行的所有行（文档一共7行，即只显示前2行）
```
[vagrant/tmp/abc] ]$head -n -5 hosts
127.0.0.1       localhost
127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
[vagrant/tmp/abc] ]$cat -n hosts
     1  127.0.0.1       localhost
     2  127.0.1.1       vagrant-ubuntu-trusty.vagrantup.com     vagrant-ubuntu-trusty
     3
     4  # The following lines are desirable for IPv6 capable hosts
     5  ::1     localhost ip6-localhost ip6-loopback
     6  ff02::1 ip6-allnodes
     7  ff02::2 ip6-allrouters
```

### 2. `tail` (取出后面几行)

    tail [-n number] 文件名
    
#### 参数

    -n  ：后面接数字，代表显示几行的意思。默认显示前10行。
    -f  ：表示持续监控文件内容的修改，按 Ctrl+c 可退出监控。
    
#### 实例

* 显示后 5 行， `tail -n 5` 与 `tail -5`
```
[vagrant/tmp/abc] ]$tail -n 5 hosts

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
[vagrant/tmp/abc] ]$tail -5 hosts

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

* 显示从第5行开始一直到文件结尾的所有行（文档一共7行，即只显示后3行）
```
[vagrant/tmp/abc] ]$tail -n +5 hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

* 显示后5行，并且持续监控文件更新
```
[vagrant/tmp/abc] ]$tail -n 5 -f hosts

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
╚
```

## 四、读取非文本内容

### 1. `od`

上面提到的命令，都是在查阅纯文本文件的内容。如果我们想要查阅非文本文件的内容，例如 /usr/bin/passwd 这个可执行文件，使用上面提到的指令来读取他的内容时， 会输出类似乱码的数据！这时，我们可以使用 od 这个指令。

#### 参数

    -t  ：后面可以接各种‘类型 (TYPE)’的输出，例如：
          a       ：利用默认的字符来输出；
          c       ：使用 ASCII 字符来输出
          d[size] ：利用十进位(decimal)来输出资料，每个整数占用 size bytes ；
          f[size] ：利用浮点数值(floating)来输出资料，每个数占用 size bytes ；
          o[size] ：利用八进位(octal)来输出资料，每个整数占用 size bytes ；
          x[size] ：利用十六进位(hexadecimal)来输出资料，每个整数占用 size bytes ；
          
#### 实例

* 将/usr/bin/passwd的内容使用ASCII方式输出
```
[vagrant/tmp/abc] ]$od -t c /usr/bin/passwd
0000000 177   E   L   F 002 001 001  \0  \0  \0  \0  \0  \0  \0  \0  \0
0000020 002  \0   >  \0 001  \0  \0  \0 337   8   @  \0  \0  \0  \0  \0
0000040   @  \0  \0  \0  \0  \0  \0  \0 270 260  \0  \0  \0  \0  \0  \0
0000060  \0  \0  \0  \0   @  \0   8  \0  \t  \0   @  \0 034  \0 033  \0
0000100 006  \0  \0  \0 005  \0  \0  \0   @  \0  \0  \0  \0  \0  \0  \0
... 后面省略 n 行 ...

# 最左边第一栏是以八进制来表示bytes数。
# 以上面范例来说，第二栏0000020代表开头是第 16 个 byes (2x8) 的内容之意。
```

* 将/etc/issue这个文件的内容以八进位列出储存值与ASCII的对照表
```
[vagrant/tmp/abc] ]$od -t oCc /etc/issue
0000000 125 142 165 156 164 165 040 061 064 056 060 064 040 114 124 123
          U   b   u   n   t   u       1   4   .   0   4       L   T   S
0000020 040 134 156 040 134 154 012 012
              \   n       \   l  \n  \n
0000030

# 如上所示，可以发现每个字符对应的数值。
# 例如b对应的记录数值为142，转成十进制：1x8^2+4x8+2=98。
```