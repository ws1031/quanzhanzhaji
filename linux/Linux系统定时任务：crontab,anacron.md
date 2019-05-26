Linux 系统定时任务：crontab,anacron
---

## 一、Cron 服务

### 1. 启动服务

    service cron start

### 2. 关闭服务

    service cron stop

### 3. 重启服务

    service cron restart

### 4. 重新载入配置

    service cron reload

### 5. 查看服务状态

    service cron status 

## 二、用户定时任务

### 1. 选项

> `-e`：执行文字编辑器来设定定时任务
> `-l`：列出目前所有定时任务
> `-r`：删除目前所有定时任务（慎用）

> 要经常备份定时任务。因为键盘上 `r` 和 `e` 是挨着的，很可能会按错导致删除所有定时任务。

### 2. crontab 格式

> 分 时 日 月 周 command

代表意义|	分	|时|	日	|月	|周|	command
-|-|-|-|-
数字范围|	0-59|	0-23|	1-31	|1-12|	0-7（0和7都表示周日）	| 需要执行的命令

特殊字符|	代表意义
-|-
`*`|	代表任何时刻。比如第一个 `*` 代表一个小时中每一分钟都执行一次。
`,`|	代表不连续的时间。比如 `0 8,12,16 * * * command` 表示每天8点，12点，16点执行一次
`-`|	代表连续时间范围。比如 `0 8-12 * * * command` 表示每天8点到12点，每小时都执行一次
`*/n`|	那个 n 代表数字，代表‘每隔 n 单位时间执行一次’。例如 `*/5 * * * * command` 表示每五分钟进行一次

> 注意事项
> 1. 当 `日` 和 `周` 都不为 `*` 时，任意一个满足条件，都会运行定时任务。、
> 2. `*/n` 并不是真正意义上的 ‘每隔 n 单位时间执行一次’，详情：https://segmentfault.com/q/1010000010290756

### 3. 实例

* 每1分钟执行一次command
```
* * * * * command
```
* 每小时的第3和第15分钟执行
```
3,15 * * * * command
```
* 在上午8点到11点的第3和第15分钟执行
```
3,15 8-11 * * * command
```
* 每隔两天的上午8点到11点的第3和第15分钟执行
```
3,15 8-11 */2 * * command
```
* 每个星期一的上午8点到11点的第3和第15分钟执行
```
3,15 8-11 * * 1 command
```
* 在 12 月内, 每天的早上 6 点到 12 点，每隔 3 个小时 0 分钟执行一次 /usr/bin/backup
```
0 6-12/3 * 12 * /usr/bin/backup
```
* 周一到周五每天下午 5:00 寄一封信给 alex@domain.name
```
0 17 * * 1-5 mail -s "hi" alex@domain.name < /tmp/maildata
```
* 每月每天的午夜 0 点 20 分, 2 点 20 分, 4 点 20 分....执行 echo "haha"
```
20 0-23/2 * * * echo "haha"
```
* 每两个小时重启一次apache
```
0 */2 * * * /sbin/service httpd restart
```
* 每天7：50开启ssh服务
```
50 7 * * * /sbin/service sshd start
```
* 每天22：50关闭ssh服务 
```
50 22 * * * /sbin/service sshd stop
```
* 每月1号和15号检查/home 磁盘 
```
0 0 1,15 * * fsck /home
```
* 每小时的第一分执行 /home/bruce/backup这个文件 
```
1 * * * * /home/bruce/backup
```
* 每周一至周五3点钟，在目录/home中，查找文件名为*.xxx的文件，并删除4天前的文件。
```
0 3 * * 1-5 find /home "*.xxx" -mtime +4 -exec rm {} \;
```
* 每月的1、11、21、31日是的6：30执行一次ls命令
```
30 6 */10 * * ls
```

## 三、系统定时任务

### 1. `/etc/crontab` 配置文件

> `crintab -e` 是编辑当前用户执行的命令，也就是不同的用户身份可以执行自己的定时任务。可是有些定时任务需要系统执行，这时我们就需要编辑 `/etc/crontab` 这个配置文件了。

#### `/etc/crontab` 文件内容（ubuntu14.04）

```
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

# 一些环境变量
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# 默认的任务，定期执行
# /etc/cron.hourly
# /etc/cron.daily
# /etc/cron.weekly
# /etc/cron.monthly
# 文件夹下的脚本文件。
#
# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

    只需要将定时任务按照格式添加到 `/etc/crontab` 文件中，`service cron reload` 重新载入配置即可。

### 2. `/etc/cron.d` 目录

    将定时任务添加到 `/etc/cron.d` 目录下，按照 `m h dom mon dow user  command` 的格式编写定时任务，`service cron reload` 重新载入配置即可。

```
[vagrant/etc/cron.d] ]$ll
total 8K
-rw-r--r-- 1 root root 712 Feb  5 12:44 php
-rw-r--r-- 1 root root 102 Feb  9  2013 .placeholder
[vagrant/etc/cron.d] ]$cat php
# /etc/cron.d/php@PHP_VERSION@: crontab fragment for PHP
#  This purges session files in session.save_path older than X,
#  where X is defined in seconds as the largest value of
#  session.gc_maxlifetime from all your SAPI php.ini files
#  or 24 minutes if not defined.  The script triggers only
#  when session.save_handler=files.
#
#  WARNING: The scripts tries hard to honour all relevant
#  session PHP options, but if you do something unusual
#  you have to disable this script and take care of your
#  sessions yourself.

# 每隔30分钟寻找并清除旧的会话
# Look for and purge old sessions every 30 minutes
09,39 *     * * *     root   [ -x /usr/lib/php/sessionclean ] && if [ ! -d /run/systemd/system ]; then /usr/lib/php/sessionclean; fi
```

### 3. `/etc/cron.{hourly,daily,weekly,monthly}`

#### 查看 `/etc/cron.{hourly,daily,weekly,monthly}` 这四个目录及目录下的文件
```
[vagrant/etc] ]$ll -d cron.{hourly,daily,weekly,monthly}
drwxr-xr-x 2 root root 4.0K Mar 29 09:21 cron.daily/
drwxr-xr-x 2 root root 4.0K May 10 01:52 cron.hourly/
drwxr-xr-x 2 root root 4.0K Jul 21  2015 cron.monthly/
drwxr-xr-x 2 root root 4.0K Jul 21  2015 cron.weekly/
[vagrant/etc] ]$ll cron.{hourly,daily,weekly,monthly}
cron.daily:
total 60K
-rwxr-xr-x 1 root root  625 Sep 18  2017 apache2*
-rwxr-xr-x 1 root root  16K Apr 10  2014 apt*
-rwxr-xr-x 1 root root  314 Feb 18  2014 aptitude*
-rwxr-xr-x 1 root root  355 Jun  4  2013 bsdmainutils*
-rwxr-xr-x 1 root root  256 Mar  7  2014 dpkg*
-rwxr-xr-x 1 root root  372 Jan 22  2014 logrotate*
-rwxr-xr-x 1 root root 1.3K Apr 10  2014 man-db*
-rwxr-xr-x 1 root root  435 Jun 20  2013 mlocate*
-rwxr-xr-x 1 root root  249 Feb 17  2014 passwd*
-rw-r--r-- 1 root root  102 Feb  9  2013 .placeholder
-rwxr-xr-x 1 root root 2.4K May 13  2013 popularity-contest*
-rwxr-xr-x 1 root root  322 Apr 11  2014 upstart*

cron.hourly:
total 8.0K
-rwxr-xr-x 1 root root  43 May 10 01:52 date*
-rw-r--r-- 1 root root 102 Feb  9  2013 .placeholder

cron.monthly:
total 4.0K
-rw-r--r-- 1 root root 102 Feb  9  2013 .placeholder

cron.weekly:
total 16K
-rwxr-xr-x 1 root root 730 Feb 23  2014 apt-xapian-index*
-rwxr-xr-x 1 root root 427 Apr 16  2014 fstrim*
-rwxr-xr-x 1 root root 771 Apr 10  2014 man-db*
-rw-r--r-- 1 root root 102 Feb  9  2013 .placeholder
```

    由此可见，`/etc/cron.{hourly,daily,weekly,monthly}` 这四个目录中都是一些有执行权限的脚本文件。

> `cron.hourly` 下的脚本每小时执行一次
> `cron.daily` 下的脚本每天执行一次
> `cron.weekly` 下的脚本每周执行一次
> `cron.monthly` 下的脚本每月执行一次

## 四、`anacron`

### 1. `anacron` 是什么

    anacron 是用来保证在系统关机时错过的定时任务可以在系统开机之后在执行

### 2. `anacron` 的监测周期

* `anacron` 会使用一天、七天、一个月作为监测周期。
* 在系统的 `/var/spool/anacron/` 目录中存在 `cron.{daily,weekly,monthly}` 文件，用于记录上次执行 cron 的时间。
* 将记录的时间与当前时间作比较，若两个时间差超过了 anacron 的指定时间差值，证明有cron任务被漏执行。

### 3. `/etc/anacrontab` 文件

```
[vagrant/etc] ]$cat /etc/anacrontab
# /etc/anacrontab: configuration file for anacron

# See anacron(8) and anacrontab(5) for details.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
HOME=/root
LOGNAME=root

# These replace cron's entries
1       5       cron.daily      run-parts --report /etc/cron.daily
7       10      cron.weekly     run-parts --report /etc/cron.weekly
@monthly        15      cron.monthly    run-parts --report /etc/cron.monthly
```

#### **对比 `/etc/crontab`**

返回查看 `/etc/crontab` 文件中有这样一段：

```
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

* `test -x`

    `test -x /usr/sbin/anacron` 表示 “检测文件 `/usr/sbin/anacron` 是否存在且具有“可执行”权限”

    若 `test -x /usr/sbin/anacron` 为 true，则不执行 `||` 后面的命令，若为 false，则执行 `||` 后面的命令。

    即，判断系统是否安装了 `anacron` , 若安装了，则忽略 `/etc/crontab` 中的这三条定时任务，改为使用 `/etc/anacrontab` 中的配置。

* `run-parts --report`

    遍历目标文件夹，执行第一层目录下具有可执行权限的文件。


#### **`/etc/anacrontab` 计划任务格式**

天数|延迟时间（分）|工作名称|实际执行的命令
-|-|-|-
1  |     5   |    cron.daily    |  run-parts --report /etc/cron.daily
7   |    10  |    cron.weekly   |  run-parts --report /etc/cron.weekly
@monthly   |     15  |    cron.monthly   | run-parts --report /etc/cron.monthly

#### **以 cron.daily 为例说明 anacron 执行过程**

1. 从 `/etc/anacrontab` 分析到 cron.daily 的天数为 1 天。
2. 从 `/var/spool/anacron/cron.daily` 中获取最后一次执行 anacron 的时间。
3. 将上面获取的时间与当前时间做对比，若相差时间为1天以上（含一天），就准备执行 cron.daily
4. 根据 `/etc/anacrontab` 的设置，将延迟5分钟执行 cron.daily。
5. 5分钟后，开始执行 `run-parts --report /etc/cron.daily` 命令，即执行 `/etc/cron.daily` 目录下的所有可执行文件。