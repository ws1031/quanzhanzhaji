Linux Shell编程（5）- 正则表达式
---

## 一、正则表达式简介

### 1. 正则表达式是什么

    正则表达式用于描述字符排列和匹配模式的一种语法规则。
    它主要用于字符串的模式分割、匹配、查找及替换操作。

### 2. 正则表达式与通配符

 / |正则表达式|通配符
-|-|-
匹配主体|文件中的内容|文件名
匹配规则|包含匹配|完全匹配
常用命令|`grep`,`awk`,`sed`|`ls`,`find`,`cp`

### 3. 通配符

> `*`   ：匹配任意0到多个字符
> `?`   ：匹配任意一个字符
> `[]`   ：匹配括号中的一个字符

#### **`*` 匹配任意0到多个字符**

* 目录下有5个文件，1个子目录
```
[root/tmp/tmp]# ll
总用量 0
-rw-r--r-- 1 root root  0 5月  31 03:37 abc.ini
-rw-r--r-- 1 root root  0 5月  31 03:37 a.conf
-rw-r--r-- 1 root root  0 5月  31 03:37 a.md
-rw-r--r-- 1 root root  0 5月  31 03:37 bbb.md
-rw-r--r-- 1 root root  0 5月  31 03:37 bc.ini
drwxr-xr-x 2 root root 58 5月  31 03:39 sh/
```
* 列出目录下所有文件，包含子目录中的文件
```
[root/tmp/tmp]# ll *
-rw-r--r-- 1 root root  0 5月  31 03:37 abc.ini
-rw-r--r-- 1 root root  0 5月  31 03:37 a.conf
-rw-r--r-- 1 root root  0 5月  31 03:37 a.md
-rw-r--r-- 1 root root  0 5月  31 03:37 bbb.md
-rw-r--r-- 1 root root  0 5月  31 03:37 bc.ini

sh:
总用量 0
-rw-r--r-- 1 root root 0 5月  31 03:39 asd.sh
-rw-r--r-- 1 root root 0 5月  31 03:38 qwe.sh
-rw-r--r-- 1 root root 0 5月  31 03:39 rty.sh
-rw-r--r-- 1 root root 0 5月  31 03:39 zxc.sh
```
* 列出以 `a` 开头和以 `sh` 开头的文件
```
[root/tmp/tmp]# ll a*
-rw-r--r-- 1 root root 0 5月  31 03:37 abc.ini
-rw-r--r-- 1 root root 0 5月  31 03:37 a.conf
-rw-r--r-- 1 root root 0 5月  31 03:37 a.md

[root/tmp/tmp]# ll sh*
总用量 0
-rw-r--r-- 1 root root 0 5月  31 03:39 asd.sh
-rw-r--r-- 1 root root 0 5月  31 03:38 qwe.sh
-rw-r--r-- 1 root root 0 5月  31 03:39 rty.sh
-rw-r--r-- 1 root root 0 5月  31 03:39 zxc.sh
```
* 因为通配符是完全匹配，所以 `a*c` 匹配不到文件，必须用 `a*c*`
```
[root/tmp/tmp]# ll a*c
ls: 无法访问a*b: 没有那个文件或目录

[root/tmp/tmp]# ll a*c*
-rw-r--r-- 1 root root 0 5月  31 03:37 abc.ini
-rw-r--r-- 1 root root 0 5月  31 03:37 a.conf
```
* 匹配字目录中的文件
```
[root/tmp/tmp]# ll s*a*
ls: 无法访问s*a*: 没有那个文件或目录
[root/tmp/tmp]# ll s*/a*
-rw-r--r-- 1 root root 0 5月  31 03:39 sh/asd.sh
```


#### **`?` 匹配任意一个字符**

```
[root/tmp/tmp]# ll
总用量 0
-rw-r--r-- 1 root root  0 5月  31 03:59 aaa.ini
-rw-r--r-- 1 root root  0 5月  31 03:37 aaa.md
-rw-r--r-- 1 root root  0 5月  31 03:37 abc.ini
-rw-r--r-- 1 root root  0 5月  31 03:37 bbb.md
-rw-r--r-- 1 root root  0 5月  31 03:37 cbc.ini
drwxr-xr-x 2 root root 58 5月  31 03:39 sh/

[root/tmp/tmp]# find . -name '?bc.ini'
./abc.ini
./cbc.ini

[root/tmp/tmp]# find . -name '???.md'
./bbb.md
./aaa.md

[root/tmp/tmp]# find . -name 'a??.ini'
./abc.ini
./aaa.ini

[root/tmp/tmp]# find . -name 's?'
./sh
```


#### **`[]` 匹配括号中的一个字符**

```
[root/tmp/tmp]# ll
总用量 0
-rw-r--r-- 1 root root  0 5月  31 03:59 aaa.ini
-rw-r--r-- 1 root root  0 5月  31 03:37 aaa.md
-rw-r--r-- 1 root root  0 5月  31 03:37 abc.ini
-rw-r--r-- 1 root root  0 5月  31 03:37 bbb.md
-rw-r--r-- 1 root root  0 5月  31 03:37 cbc.ini
drwxr-xr-x 2 root root 58 5月  31 03:39 sh/

[root/tmp/tmp]# ll [a-z]bc.ini
-rw-r--r-- 1 root root 0 5月  31 03:37 abc.ini
-rw-r--r-- 1 root root 0 5月  31 03:37 cbc.ini

[root/tmp/tmp]# ll [ab]bc.ini
-rw-r--r-- 1 root root 0 5月  31 03:37 abc.ini
```

## 二、基础正则表达式

元字符|作用
-|-
`*`|前一个字符匹配0次或任意多次
`.`|匹配除了换行符以外的任意一个字符
`^`|匹配行首。例如：`^hello` 会匹配以 hello 开头的行
`$`|匹配行尾。例如：`hello$` 会匹配以 hello 结尾的行
`[]`|匹配括号中任意一个字符，只匹配一个字符。例如：<br>`[aeiou]`匹配任意一个元音字符<br>`[0-9]`匹配任意一个数字<br>`[a-z]`匹配任意一个小写字母
`[^]`|取反，匹配括号中字符以外的任意一个字符。例如：<br>`[^0-9]`匹配任意一个非数字字符<br>`[^a-z]`匹配任意一个非小写字母字符
`\`|转义符，用于取消特殊符号的含义。例如：`\.` 匹配符号 '.' 
`\{n\}`|表示其前面的字符恰好出现n次。
`\{n,\}`|表示其前面的字符至少出现n次。
`\{n,m\}`|表示其前面的字符至少出现n次，最多出现m次。

### 0. 测试用文档

> 使用 `grep -n '[正则表达式]' [文件]` 命令来输出匹配的行，并显示行号。

* test.md
```
 1  a
 2  aa
 3  aaa
 4  aaaa
 5  aaaaa
 6
 7  b
 8  bb
 9  bbb
10  bbbb
11  bbbbb
12
13  ab
14  aab
15  abcb
16  abcde
17
18  said
19  soid
20  suud
21  sooooood
22
23  023456
24  1223456
25  123
26  3445
27  425asdf
28  dasd1234ff
29
30  3.1415926
31  hello world.
32
33  2018-05-31
34  2017-02-22
35
36  192.168.33.10
37  255.255.255.255
38  0.0.0.0
```

### 1. `*` 前一个字符匹配0次或任意多次

### 2. `.` 匹配除了换行符以外的任意一个字符

* `.*` 匹配所有内容，包括空白行

*  `s..d` 匹配在 s 和 d 之间有两个字符的行
```
[root/tmp]# grep -n 's..d' test.md
18:said
19:soid
20:suud
```
* `a.c` 匹配在 a 和 c 之间有一个字符的行
```
[root/tmp]# grep -n 'a.c' test.md
15:abcb
16:abcde
```
* `a.*d` 匹配在 a 和 d 之间有任意0到多个字符的行
```
[root/tmp]# grep -n 'a.*d' test.md
16:abcde
18:said
27:425asdf
28:dasd1234ff
```

### 3. `^` 匹配行首，`$` 匹配行尾

* `^s` 匹配以 s 开头的行
```
[root/tmp]# grep -n '^s' test.md
18:said
19:soid
20:suud
21:sooooood
```

* `d$` 匹配以 d 结尾的行
```
[root/tmp]# grep -n 'd$' test.md
18:said
19:soid
20:suud
21:sooooood
```

* `^$` 匹配所有空行
```
[root/tmp]# grep -n '^$' test.md
6:
12:
17:
22:
29:
32:
35:
```

### 4. `[]`|匹配括号中任意一个字符，只匹配一个字符。

* `s[aeiou]id` 匹配 s 和 id 之间只有一个元音字母的行
```
[root/tmp]# grep -n 's[aeiou]id' test.md
18:said
19:soid
```

* `s[aeiou]*d` 匹配 s 和 d 之间只有任意0个或多个元音字母的行
```
[root/tmp]# grep -n 's[aeiou]*d' test.md
18:said
19:soid
20:suud
21:sooooood
27:425asdf
28:dasd1234ff
```

* `[4-6]` 匹配包含 4-6 之间的任意一个数字的行
```
[root/tmp]# grep -n '[4-6]' test.md
23:023456
24:1223456
26:3445
27:425asdf
28:dasd1234ff
30:3.1415926
33:2018-05-31
36:192.168.33.10
37:255.255.255.255
```
* `^[c-z]` 匹配以 c-z 之间的任意一个小写字母开头的行
```
[root/tmp]# grep -n '^[c-z]' test.md
18:said
19:soid
20:suud
21:sooooood
28:dasd1234ff
31:hello world.
```

### 5. `[^]` 取反，匹配括号中字符以外的任意一个字符

* `^[^a-z]` 匹配不以小写字母开头的行
```
[root/tmp]# grep -n '^[^a-z]' test.md
23:023456
24:1223456
25:123
26:3445
27:425asdf
30:3.1415926
33:2018-05-31
34:2017-02-22
36:192.168.33.10
37:255.255.255.255
38:0.0.0.0
```

* `[^a-z0-9]` 匹配包含非数字和小写字母的行
```
[root/tmp]# grep -n '[^a-z0-9]' test.md
30:3.1415926
31:hello world.
33:2018-05-31
34:2017-02-22
36:192.168.33.10
37:255.255.255.255
38:0.0.0.0
```

### 6. `\`|转义符，用于取消特殊符号的含义

* `\.` 匹配包含 '.' 的行
```
[root/tmp]# grep -n '\.' test.md
30:3.1415926
31:hello world.
36:192.168.33.10
37:255.255.255.255
38:0.0.0.0
```
* `\.$` 匹配以 '.' 结尾的行
```
[root/tmp]# grep -n '\.$' test.md
31:hello world.
```

### 7. `\{n\}` 表示其前面的字符恰好出现n次

* `a\{3\}` 匹配 a 连续出现 3 次的行
```
[root/tmp]# grep -n 'a\{3\}' test.md
3:aaa
4:aaaa
5:aaaaa
```

* `[0-9]\{5\}` 匹配包含连续5个数字的行
```
[root/tmp]# grep -n '[0-9]\{5\}' test.md
23:023456
24:1223456
30:3.1415926
```

### 8. `\{n,\}` 表示其前面的字符至少出现n次


* `a\{3,\}` 匹配 a 至少连续出现 3 次的行
```
[root/tmp]# grep -n 'a\{3,\}' test.md
3:aaa
4:aaaa
5:aaaaa
```

* `[0-9]\{5,\}` 匹配包含至少连续5个数字的行
```
[root/tmp]# grep -n '[0-9]\{5,\}' test.md
23:023456
24:1223456
30:3.1415926
```

### 9. `\{n,m\}` 表示其前面的字符至少出现n次，最多出现m次。


* `s[a-z]\{2,5\}d` 匹配 s 和 d 之间至少有2个字母，最多有5个字母的行
```
[root/tmp]# grep -n 's[a-z]\{2,5\}d' test.md
18:said
19:soid
20:suud
```

## 三、几个实例

### 1. 匹配日期格式 'YYYY-MM-DD'

> 并非严格匹配日期格式，实际上是匹配 '9999-99-99'

#### `[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}`

```
[root/tmp]# grep -n '[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}' test.md
33:2018-05-31
34:2017-02-22
```

#### `[0-9]\{4\}\(-[0-9]\{2\}\)\{2\}`
```
[root/tmp]# grep -n '[0-9]\{4\}\(-[0-9]\{2\}\)\{2\}' test.md
33:2018-05-31
34:2017-02-22
```

### 2. 匹配IP地址

> 并非严格匹配IP地址，实际上是匹配 0.0.0.0 - 999.999.999.999，而实际IP地址只到 255.255.255.255 

#### `[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}`
```
[root/tmp]# grep -n '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' test.md
36:192.168.33.10
37:255.255.255.255
38:0.0.0.0
```

#### `[0-9]\{1,3\}\(\.[0-9]\{1,3\}\)\{3\}`
```
[root/tmp]# grep -n '[0-9]\{1,3\}\(\.[0-9]\{1,3\}\)\{3\}' test.md
36:192.168.33.10
37:255.255.255.255
38:0.0.0.0
```