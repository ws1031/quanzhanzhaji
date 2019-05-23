Linux搜索命令总结 - whereis,which,locate,find,grep
---

## 一、命令搜索命令 `whereis` 与 `which`

### 1. whereis 命令

* 搜索命令所在路径及帮助文档所在位置

### 选项

* -b: 只查找可执行文件
* -m: 之查找帮助文件

### 2. which 命令

* 搜索命令所在位置及别名

### 3. 引申

* whatis 查询一个命令执行什么功能

## 二、文件搜索命令 `locate` 

### 1. 简介
locate(locate) 命令用来查找文件或目录。 locate命令要比find -name快得多，原因在于它不搜索具体目录，而是搜索一个数据库/var/lib/mlocate/mlocate.db 。这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，因此，我们在用whereis和locate 查找文件时，有时会找到已经被删除的数据，或者刚刚建立文件，却无法查找到，原因就是因为数据库文件没有被更新。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。整个locate工作其实是由四部分组成的:

1. `/usr/bin/locate`     命令文件位置
2. `/usr/bin/updatedb`   更新数据库命令
3. `/etc/updatedb.conf`  updatedb的配置文件
4. `/var/lib/mlocate/mlocate.db`  存放文件信息的文件，即locate命令搜索用到的数据库

### 2. updatedb的配置文件/etc/updatedb.conf

```
[vagrant~] ]$cat /etc/updatedb.conf
PRUNE_BIND_MOUNTS="yes"
# PRUNENAMES=".git .bzr .hg .svn"
PRUNEPATHS="/tmp /var/spool /media /home/.ecryptfs"
PRUNEFS="NFS nfs nfs4 rpc_pipefs afs binfmt_misc proc smbfs autofs iso9660 ncpfs coda devpts ftpfs devfs mfs shfs sysfs cifs lustre tmpfs usbfs udf fuse.glusterfs fuse.sshfs curlftpfs ecryptfs fusesmb devtmpfs"
```

* PRUNE_BIND_MOUNTS="yes" 值为"yes"时开启搜索限制，此时，下边的配置生效；为"no"时关闭搜索限制
* PRUNEFS 搜索时，忽略掉的文件系统
* PRUNENAMES 搜索时，忽略的文件类型
* PRUNEPATHS 搜索时，忽略的文件路径


### 3. 命令格式

* locate [选项] [文件名]

### 4. 常用选项

```
"locate -c" 查询指定文件的数目。(c为count的意思)
"locate -e" 只显示当前存在的文件条目。(e为existing的意思)
"locate -h" 显示"locate"命令的帮助信息。(h为help的意思)
"locate -i" 查找时忽略大小写区别。(i为ignore的意思)
"locate -n" 最大显示条数" 至多显示"最大显示条数"条查询到的内容。
"locate -r" 使用正则运算式做寻找的条件。(r为regexp的意思)
```

## 三、文件查找命令 `find`

### 1. 命令格式

* find [搜索范围] [搜索条件]

### 2. 按照文件名查找文件

    find <path> -name <filename>
    
    #不区分大小写
    find <path> -iname <filename>

* 查找指定文件名的文件，如果需要匹配，使用通配符匹配，通配符是完全匹配
* Linux中的通配符
> `*` 匹配任意多个字符
> `?` 匹配任意一个字符
> `[]` 匹配任意一个中括号内的字符


### 3. 按照所有者查找文件

    find <path> -user <username>
    
### 4. 查找没有所有者的文件

    find <path> -nouser

### 5. 按照时间查找文件

    find <path> -mtime +10

* 参数说明

> `atime` 文件访问时间
> `ctime` 文件属性修改时间
> `mtime` 文件内容修改时间


> `-10` 10天内修改的文件
> `10` 10天当天修改的文件
> `+10` 10天前修改的文件

* 实例：查找30天前的日志文件
```
[vagrant~] ]$find /var/log -mtime +30
/var/log/php5-fpm.log.10.gz
/var/log/redis/redis-server.log.5.gz
/var/log/redis/redis-server.log.6.gz
/var/log/redis/redis-server.log.9.gz
/var/log/redis/redis-server.log.12.gz
/var/log/redis/redis-server.log.10.gz
/var/log/redis/redis-server.log.11.gz
/var/log/redis/redis-server.log.8.gz
/var/log/redis/redis-server.log.7.gz
/var/log/php5-fpm.log.5.gz
/var/log/unattended-upgrades
/var/log/unattended-upgrades/unattended-upgrades-shutdown.log
/var/log/mongodb/mongodb.log.6.gz
/var/log/mongodb/mongodb.log.8.gz
/var/log/mongodb/mongodb.log.5.gz
/var/log/mongodb/mongodb.log.7.gz
/var/log/mongodb/mongodb.log.9.gz
/var/log/mongodb/mongodb.log.10.gz
/var/log/dpkg.log.7.gz
/var/log/dpkg.log.2.gz
```

### 6. 按照文件大小查找文件

    find <path> -size <filesize>

* 参数说明（注意大小写）
> filesize:
> `-25k` 小于25kb的文件
> `25k` 等于25kb的文件
> `+25k` 大于25kb的文件
> `+100M` 大于100M的文件

### 7. 按照 i节点查找文件

    find <path> -inum <inode>

### 8. 高级应用

#### find /etc -size +20k -a -size -50k
* 查找 `/etc/` 目录下，大于20kb 并且 小于 50kb 的文件
> `-a` and 逻辑与，两个条件都满足
> `-o` or 逻辑或，两个条件满足一个即可

#### find /etc -size +20k -a -size -50k -exec ls -lh {} \;
* 查找 `/etc/` 目录下，大于20kb 并且 小于 50kb 的文件，并显示这些文件的详细信息
> `--exec ls -lh {} \`，对搜索结果执行 `ls -lh` 操作

#### find /etc -size +20k -a -size -50k -exec rm -rf {} \;
* 查找 `/etc/` 目录下，大于20kb 并且 小于 50kb 的文件，并删除这些文件
> `--exec rm -rf {} \`，对搜索结果执行 `rm -rf` 操作


### 9. 注意事项
* 避免大范围搜索，会非常耗费系统资源

## 四、字符串搜索命令 `grep`
### 1. 命令格式

    grep [选项] 字符串 文件名
    
* 在文件当中匹配符合条件的字符串
* 选项

> `-r` 目录递归搜索
> `-a` 不要忽略二进制数据
> `-i` 忽略大小写
> `-v` 排除制定字符串
> `-E` 将范本样式为延伸的普通表示法来使用，意味着使用能使用扩展正则表达式。

### 2. 搜索文件中匹配 `pattern` 的内容

    grep -E pattern files

#### 实例
* 在以 `.bash` 开头的文件中搜索匹配 `d.?f` 的内容
```
[vagrant~] ]$ll
total 96K
-rw------- 1 vagrant vagrant 9.7K May  8 08:04 .bash_history
-rw-r--r-- 1 vagrant vagrant  220 Jul 21  2015 .bash_logout
-rw-rw-r-- 1 vagrant vagrant   99 Sep 27  2016 .bash_profile
-rw-rw-r-- 1 vagrant vagrant 4.7K May  3  2017 .bashrc
drwx------ 2 vagrant vagrant 4.0K Mar 30 01:17 .ssh/
drwxrwxr-x 4 vagrant vagrant 4.0K Sep 26  2016 .vim/
-rw-rw-r-- 1 vagrant vagrant  15K Apr 25 10:27 .viminfo
-rw-rw-r-- 1 vagrant vagrant 6.1K Oct  9  2016 .vimrc
[vagrant~] ]$grep -aE "d.?f" .bash*
.bash_history:vim default
.bash_history:ps -df
.bashrc:# off by default to not distract the user: the focus in a terminal window
.bashrc:# Alias definitions.
```

### 3. 在文件夹中递归地搜索匹配 `pattern` 的内容

    grep -r pattern dir

#### 实例
* 在 `/etc/php/` 中搜索 "max_children"
```
[vagrant~] ]$sudo grep -r "max_children" /etc/php/
/etc/php/7.2/fpm/pool.d/www.conf:;   static  - a fixed number (pm.max_children) of child processes;
/etc/php/7.2/fpm/pool.d/www.conf:;             pm.max_children      - the maximum number of children that can
/etc/php/7.2/fpm/pool.d/www.conf:;             pm.max_children           - the maximum number of children that
/etc/php/7.2/fpm/pool.d/www.conf:pm.max_children = 5
```

### 4. 搜索命令输出中匹配 `pattern` 的内容

    command | grep -E pattern

#### 实例
* 从 `ps -ef` 的输出结果中匹配 "b.*h"
```
[vagrant~] ]$ps -ef | grep -E "b.*h"
root         9     2  0 00:23 ?        00:00:00 [rcu_bh]
root       431     1  0 00:23 ?        00:00:00 dhclient -1 -v -pf /run/dhclient.eth0.pid -lf /var/lib/dhcp/dhclient.eth0.leases eth0
root      1117     1  0 00:23 ?        00:00:11 /usr/sbin/VBoxService --pidfile /var/run/vboxadd-service.sh
root      1433     1  0 00:23 ?        00:00:00 /usr/sbin/sshd -D
vagrant   2662  2661  0 08:07 pts/0    00:00:00 -bash
vagrant   3695  2662  0 09:23 pts/0    00:00:00 grep --color=auto -E b.*h
```

### 5. grep命令与find命令的区别
* grep命令：在文件当中搜索符合条件的字符串，如果需要匹配，使用正则表达式进行匹配，正则表达式是包含关系。
* find命令：在系统当中搜索符合条件的文件名，如果需要匹配，使用通配符匹配，通配符是完全匹配。

### 6. 实践应用

#### 根据文件内容查找文件

* 在 `/etc/php/` 目录下查找包含“max_children”的文件
```
sudo grep -r "max_children" /etc/php/

sudo find /etc/php/ -type f -name '*' | sudo xargs grep "max_children"
```

#### 去掉配置文件中的注释和空行，并生成一个新的配置文件
```
cat /etc/redis/redis.conf | grep -v "#" | grep -v "^$" > /etc/redis/redis6379.conf
```