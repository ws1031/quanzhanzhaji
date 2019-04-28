Linux（CentOS）软件管理（1）- RPM包管理
---

## 软件包管理简介

### 1. 软件包分类

* 源码包
* 二进制包（RPM包、系统默认包）

### 2. 源码包

#### **优点**

* 开源，如果能力足够，可以修改源代码
* 可以自定义选择所需的功能
* 软件是编译安装，所以更加适合自己的系统，更加稳定，效率更高
* 卸载方便，直接删除安装目录即可，不会有任何残留

#### **缺点**

* 安装过程步骤较多，尤其是安装较大的软件集合时（例如LAMP环境搭建）
* 编译过程时间较长，安装比二进制安装时间长
* 因为是编译安装，安装过程中一旦报错，新手很难解决

### 3. 二进制包（RPM包、系统默认包）

#### **优点**

* 包管理系统简单，只通过几个命令即可实现包的安装、升级、查询和卸载
* 安装速度比源码包安装快得多

#### **缺点**

* 经过编译，不再可以看到源代码
* 功能选择不如源码包灵活
* 依赖性解决比较复杂

### 4. 脚本安装包

> 所谓的脚本安装包，就是把复杂的软件包安装过程写成程序的脚本，初学者可以执行程序脚本实现一键安装。但实际安装的还是源码包和二进制包。

#### **优点**

* 安装简单、快捷

#### **缺点**

* 完全丧失了自定义性

#### **不建议使用脚本安装包**

---

# RPM包管理

## 一、RPM 包命名规则

### 1. RPM 包的来源

#### **RPM 包存在于系统光盘中**

> 光盘挂载

> 1. 插入光盘
> 2. 建立挂载点
```
[root/dev]$mkdir /mnt/cdrom
```
> 3. 挂载光盘
```
[root/dev]$mount /dev/cdrom /mnt/cdrom/
mount: /dev/sr0 is write-protected, mounting read-only
```
> 4. 查看光盘下的RPM包
```
[root/dev]$cd /mnt/cdrom/

[root/mnt/cdrom]$ll
total 109K
-rw-rw-r-- 1 root root   14 May  2 11:28 CentOS_BuildTag
-rw-r--r-- 1 root root   29 May  3 20:30 .discinfo
drwxr-xr-x 3 root root 2.0K May  3 20:34 EFI/
-rw-rw-r-- 1 root root  227 Aug 30  2017 EULA
-rw-rw-r-- 1 root root  18K Dec  9  2015 GPL
drwxr-xr-x 3 root root 2.0K May  3 20:45 images/
drwxr-xr-x 2 root root 2.0K May  3 20:34 isolinux/
drwxr-xr-x 2 root root 2.0K May  3 20:34 LiveOS/
drwxrwxr-x 2 root root  70K May  3 21:03 Packages/
drwxrwxr-x 2 root root 4.0K May  3 21:06 repodata/
-rw-rw-r-- 1 root root 1.7K Dec  9  2015 RPM-GPG-KEY-CentOS-7
-rw-rw-r-- 1 root root 1.7K Dec  9  2015 RPM-GPG-KEY-CentOS-Testing-7
-r--r--r-- 1 root root 2.9K May  3 21:07 TRANS.TBL
-rw-r--r-- 1 root root  354 May  3 20:34 .treeinfo

[root/mnt/cdrom]$cd Packages/

[root/mnt/cdrom/Packages]$ll
total 351M
-rw-rw-r-- 1 root root   82K Apr 25 10:52 acl-2.2.51-14.el7.x86_64.rpm
-rw-rw-r-- 1 root root   23K Jul  4  2014 aic94xx-firmware-30-6.el7.noarch.rpm
-rw-rw-r-- 1 root root  134K Aug 10  2017 aide-0.15.1-13.el7.x86_64.rpm
...省略n行
-rw-rw-r-- 1 root root  1.3M Apr 25 11:52 yum-3.4.3-158.el7.centos.noarch.rpm
-rw-rw-r-- 1 root root   28K Jul  4  2014 yum-metadata-parser-1.1.4-10.el7.x86_64.rpm
-rw-rw-r-- 1 root root   33K Apr 25 11:52 yum-plugin-fastestmirror-1.1.31-45.el7.noarch.rpm
-rw-rw-r-- 1 root root  260K Nov 20  2016 zip-3.0-11.el7.x86_64.rpm
-rw-rw-r-- 1 root root   90K Nov 20  2016 zlib-1.2.7-17.el7.x86_64.rpm
```


### 2. RPM 包命名规则

> zip-3.0-11.el7.x86_64.rpm

>    - zip：软件包名
>    - 3.0： 软件版本
>    - 11： 软件发布的次数
>    - el7： 适合的Linux平台
>    - x86_64： 适合的硬件平台
>    - rpm： 扩展名

### 3. RPM 包的依赖性

#### **树形依赖**

    a -> b -> c

#### **环形依赖**

    a -> b -> c -> a

#### **模块依赖**

    模块依赖查询网站：www.rpmfind.net

## 二、RPM包安装

> 包全名与包名
> - 包全名：操作的包是没有安装的软件包时，使用包全名。而且要注意路径。
> - 包名：操作已经安装的软件包时，使用包名。实际上是搜索 `/var/lib/rpm/` 中的数据库。

### 1. 语法

    rpm -ivh [包全名]

### 2. 选项

> `-i`：(install) 安装
> `-v`：(verbose) 显示详细信息
> `-h`：(hash) 显示进度

### 3. 实例
```
[root/mnt/cdrom/Packages]$rpm -ivh zip-3.0-11.el7.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:zip-3.0-11.el7                   ################################# [100%]
```

## 三、RPM包升级

### 1. 语法

    rpm -Uvh [包全名]

### 2. 选项

> `-U`：(upgrade) 升级

### 3. 实例
```
[root/mnt/cdrom/Packages]$rpm -Uvh tar-1.26-34.el7.x86_64.rpm
Preparing...                          ################################# [100%]
        package tar-2:1.26-34.el7.x86_64 is already installed
```

## 四、RPM包卸载

### 1. 语法

    rpm -e [包名]

### 2. 选项

> `-e`：(erase) 卸载

### 3. 实例
```
[root/mnt/cdrom/Packages]$rpm -Uvh tar-1.26-34.el7.x86_64.rpm
Preparing...                          ################################# [100%]
        package tar-2:1.26-34.el7.x86_64 is already installed
```

## 五、RPM包查询

### 1. 检查包是否安装

#### **语法**

    rpm -q 包名

#### **选项**

> `-q`：(query) 查询

#### **实例**

```
[root/mnt/cdrom/Packages]$rpm -q zip
zip-3.0-11.el7.x86_64

[root/mnt/cdrom/Packages]$rpm -q mysql
package mysql is not installed
```

### 2. 查询所有已安装的RPM包

#### **语法**

    rpm -qa

#### **选项**
> `-a`：(all) 所有

#### **实例**

```
[root ~]$rpm -qa
nss-tools-3.34.0-4.el7.x86_64
fipscheck-1.4.1-6.el7.x86_64
curl-7.29.0-46.el7.x86_64
...省略n行
libpipeline-1.2.3-3.el7.x86_64
linux-firmware-20180220-62.git6d51311.el7.noarch
```
* 使用管道符过滤指定名称
```
[root ~]$rpm -qa | grep zip
bzip2-1.0.6-13.el7.x86_64
bzip2-libs-1.0.6-13.el7.x86_64
bzip2-libs-1.0.6-13.el7.i686
zip-3.0-11.el7.x86_64
gzip-1.5-10.el7.x86_64
```

### 3. 查询软件包的详细信息

#### **语法**

    rpm -qi 包名 （查询已安装软件包详细信息）
    rpm -qip 包全名 （查询未安装软件包详细信息）

#### **选项**
> `-i`：(infomation) 查询软件信息
> `-p`：(package) 查询未安装包信息

#### **实例**

* 查询已安装软件包详细信息
```
[root/mnt/cdrom/Packages]$rpm -qi zip
Name        : zip
Version     : 3.0
Release     : 11.el7
Architecture: x86_64
Install Date: Tue 22 May 2018 03:13:50 AM UTC
Group       : Applications/Archiving
Size        : 815173
License     : BSD
Signature   : RSA/SHA256, Sun 20 Nov 2016 09:04:58 PM UTC, Key ID 24c6a8a7f4a80eb5
Source RPM  : zip-3.0-11.el7.src.rpm
Build Date  : Sat 05 Nov 2016 04:49:55 PM UTC
Build Host  : worker1.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://www.info-zip.org/Zip.html
Summary     : A file compression and packaging utility compatible with PKZIP
Description :
The zip program is a compression and file packaging utility.  Zip is
analogous to a combination of the UNIX tar and compress commands and
is compatible with PKZIP (a compression and file packaging utility for
MS-DOS systems).

Install the zip package if you need to compress files using the zip
program.
```
* 查询未安装软件包详细信息
```
[root/mnt/cdrom/Packages]$rpm -qip json-c-0.11-4.el7_0.x86_64.rpm
Name        : json-c
Version     : 0.11
Release     : 4.el7_0
Architecture: x86_64
Install Date: (not installed)
Group       : Development/Libraries
Size        : 65593
License     : MIT
Signature   : RSA/SHA256, Sat 05 Jul 2014 03:25:00 PM UTC, Key ID 24c6a8a7f4a80eb5
Source RPM  : json-c-0.11-4.el7_0.src.rpm
Build Date  : Tue 24 Jun 2014 12:18:58 PM UTC
Build Host  : worker1.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : https://github.com/json-c/json-c/wiki
Summary     : A JSON implementation in C
Description :
JSON-C implements a reference counting object model that allows you to easily
construct JSON objects in C, output them as JSON formatted strings and parse
JSON formatted strings back into the C representation of JSON objects.
```

### 4. 查询包中的文件安装位置

> rpm 包默认安装位置由该软件包的开发者决定

> rpm 包也可以通过 `--prefix=<dir>` 选项指定安装目录

**RPM包默认安装位置（仅供参考）**

目录|说明
-|-
`/etc/init.d/`|启动脚本所在目录
`/etc/sysconfig/`|初始化环境配置文件目录
`/etc/`|配置文件安装目录
`/var/log/`|日志文件目录
`/usr/bin/`|可执行的命令安装目录
`/usr/lib/`|程序所使用的函数库保存目录
`/usr/share/doc/`|基本的软件使用手册保存目录
`/usr/share/man/`|帮助文件保存目录

#### **语法**

    rpm -ql 包名 （查询已安装软件包文件安装位置）
    rpm -qlp 包全名 （查询未安装软件包文件安装位置）

#### **选项**

> `-l`：(list) 文件列表
> `-p`：(package) 查询未安装包信息

#### **实例**

* 查询已安装软件包文件安装位置
```
[root/mnt/cdrom/Packages]$rpm -ql zip
/usr/bin/zip
/usr/bin/zipcloak
/usr/bin/zipnote
/usr/bin/zipsplit
/usr/share/doc/zip-3.0
/usr/share/doc/zip-3.0/CHANGES
/usr/share/doc/zip-3.0/LICENSE
/usr/share/doc/zip-3.0/README
/usr/share/doc/zip-3.0/README.CR
/usr/share/doc/zip-3.0/TODO
/usr/share/doc/zip-3.0/WHATSNEW
/usr/share/doc/zip-3.0/WHERE
/usr/share/doc/zip-3.0/algorith.txt
/usr/share/man/man1/zip.1.gz
/usr/share/man/man1/zipcloak.1.gz
/usr/share/man/man1/zipnote.1.gz
/usr/share/man/man1/zipsplit.1.gz
```
* 查询未安装软件包文件安装位置
```
[root/mnt/cdrom/Packages]$rpm -qlp json-c-0.11-4.el7_0.x86_64.rpm
/usr/lib64/libjson-c.so.2
/usr/lib64/libjson-c.so.2.0.1
/usr/lib64/libjson.so.0
/usr/lib64/libjson.so.0.1.0
/usr/share/doc/json-c-0.11
/usr/share/doc/json-c-0.11/AUTHORS
/usr/share/doc/json-c-0.11/COPYING
/usr/share/doc/json-c-0.11/ChangeLog
/usr/share/doc/json-c-0.11/NEWS
/usr/share/doc/json-c-0.11/README
/usr/share/doc/json-c-0.11/README.html
```

### 5. 查询系统文件属于哪个RPM包

#### **语法**

    rpm -qf [系统文件或目录名]

#### **选项**

> `-f`：(file) 查询系统文件属于哪个软件包

#### **实例**

```
[root/mnt/cdrom/Packages]$rpm -qf /etc/python/
python-libs-2.7.5-68.el7.x86_64
```

### 6. 查询RPM包的依赖性

#### **语法**

    rpm -qR 包名

#### **选项**

> `-R`：(requires) 查询软件包的依赖性
> `-p`：(package) 查询未安装包信息

#### **实例**

* 查询已安装软件包的依赖性
```
[root/mnt/cdrom/Packages]$rpm -qR zip
libbz2.so.1()(64bit)
libc.so.6()(64bit)
libc.so.6(GLIBC_2.14)(64bit)
libc.so.6(GLIBC_2.2.5)(64bit)
libc.so.6(GLIBC_2.3)(64bit)
libc.so.6(GLIBC_2.3.4)(64bit)
libc.so.6(GLIBC_2.4)(64bit)
libc.so.6(GLIBC_2.7)(64bit)
rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(FileDigests) <= 4.6.0-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rtld(GNU_HASH)
rpmlib(PayloadIsXz) <= 5.2-1
```
* 查询未安装软件包的依赖性
```
[root/mnt/cdrom/Packages]$rpm -qRp json-c-0.11-4.el7_0.x86_64.rpm
/sbin/ldconfig
/sbin/ldconfig
libc.so.6()(64bit)
libc.so.6(GLIBC_2.14)(64bit)
libc.so.6(GLIBC_2.2.5)(64bit)
libc.so.6(GLIBC_2.3)(64bit)
libc.so.6(GLIBC_2.3.4)(64bit)
libc.so.6(GLIBC_2.4)(64bit)
libc.so.6(GLIBC_2.8)(64bit)
libjson-c.so.2()(64bit)
rpmlib(CompressedFileNames) <= 3.0.4-1
rpmlib(FileDigests) <= 4.6.0-1
rpmlib(PayloadFilesHavePrefix) <= 4.0-1
rtld(GNU_HASH)
rpmlib(PayloadIsXz) <= 5.2-1
```

## 六、RPM包校验

### 1. 语法

    rpm -V 已安装的包名

### 2. 选项

> `-V`：(verify) 校验指定的RPM包中的文件

### 3. 实例

```
[root/etc]$rpm -V sudo
S.5....T.  c /etc/sudoers
```

> S.5....T.  c /etc/sudoers

>    - S：文件大小发生改变
>    - 5：文件内容发生改变
>    - T：文件的修改时间发生改变
>    - c：文件类型为配置文件


#### **验证内容中的8个信息**

\ | 说明
-|-
`S`|文件大小是否改变
`M`|文件类型或权限（rwx）是否改变
`5`|文件MD5校验和是否改变（可以看出是文件内容是否改变）
`D`|设备的主从代码是否改变
`L`|文件路径是否改变
`U`|文件所有者是否改变
`G`|文件所属组是否改变
`T`|文件的修改时间是否改变

#### **文件类型**

\ | 说明
-|-
`c`|配置文件（config file）
`d`|普通文档（documentation）
`g`|“鬼”文件（ghost file），很少见，就是该文件不应该被这个RPM包包含
`L`|授权文件（license file）
`r`|描述文件（read me）

## 七、RPM 包中的文件提取

> 主要用于恢复被误删的命令文件

### 1. 命令

    rpm2cpio 包全名 | cpio -idv .文件绝对路径

### 2. rpm2cpio

    将RPM包转换为cpio格式的命令

### 3. cpio
    
    是一个标准工具，用于创建软件档案文件和从档案文件中提取文件

* 语法


    cpio [选项] < [文件|设备]

* 选项

> `-i`：copy-in模式，还原
> `-d`：还原时自动新建目录
> `-v`：显示还原过程

### 4. 实例
* 查询 `/usr/bin/zip` 命令属于哪个软件包
```
[root~]$rpm -qf /usr/bin/zip
zip-3.0-11.el7.x86_64
```
* 模拟 `/usr/bin/zip` 命令被误删
```
[root~]$mv /usr/bin/zip /tmp/

[root~]$zip
bash: zip: command not found
```
* 提取RPM包中的 `/usr/bin/zip` 命令到当前目录的 `usr/bin/zip`
```
[root~]$rpm2cpio /mnt/cdrom/Packages/zip-3.0-11.el7.x86_64.rpm | cpio -idv ./usr/bin/zip
./usr/bin/zip
1598 blocks

[root~]$ll
total 52K
drwxr-xr-x  3 root root   16 May 22 05:10 usr/
```

* 把 `zip` 命令复制回 `/usr/bin/` 目录下，修复丢失的文件

```
[root~]$cp usr/bin/zip /usr/bin/

[root~]$zip
Copyright (c) 1990-2008 Info-ZIP - Type 'zip "-L"' for software license.
Zip 3.0 (July 5th 2008). Usage:
zip [-options] [-b path] [-t mmddyyyy] [-n suffixes] [zipfile list] [-xi list]
  The default action is to add or replace zipfile entries from list, which
  can include the special name - to compress standard input.
  If zipfile and list are omitted, zip compresses stdin to stdout.
  -f   freshen: only changed files  -u   update: only changed or new files
  -d   delete entries in zipfile    -m   move into zipfile (delete OS files)
  -r   recurse into directories     -j   junk (don't record) directory names
  -0   store only                   -l   convert LF to CR LF (-ll CR LF to LF)
  -1   compress faster              -9   compress better
  -q   quiet operation              -v   verbose operation/print version info
  -c   add one-line comments        -z   add zipfile comment
  -@   read names from stdin        -o   make zipfile as old as latest entry
  -x   exclude the following names  -i   include only the following names
  -F   fix zipfile (-FF try harder) -D   do not add directory entries
  -A   adjust self-extracting exe   -J   junk zipfile prefix (unzipsfx)
  -T   test zipfile integrity       -X   eXclude eXtra file attributes
  -y   store symbolic links as the link instead of the referenced file
  -e   encrypt                      -n   don't compress these suffixes
  -h2  show more help
```
