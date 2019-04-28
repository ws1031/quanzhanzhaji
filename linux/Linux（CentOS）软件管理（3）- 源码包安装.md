## 一、源码包的特点

### 1. 优点
* 开源，如果能力足够，可以修改源代码
* 可以自定义选择所需的功能
* 软件是编译安装，所以更加适合自己的系统，更加稳定，效率更高
* 卸载方便，直接删除安装目录即可，不会有任何残留

### 2. 缺点
* 安装过程步骤较多，尤其是安装较大的软件集合时（例如LAMP环境搭建）
* 编译过程时间较长，安装比二进制安装时间长
* 因为是编译安装，安装过程中一旦报错，新手很难解决

### 3. 源码包与RPM包的区别

#### **安装前的区别：概念上的区别**

\ |源码包|RPM包
-|-|-
包类型|未经过编译，源码可见|编译之后的二进制包
定制性|可以自定义选择所需的功能|自定义功能选择不灵活
安装、升级|步骤多，编译时间长|使用rpm命令直接安装、升级，简单，速度快
卸载|没有卸载命令，直接删除安装目录即可，无残留|使用rpm卸载命令卸载

#### **安装后的区别：安装位置不同**

**RPM包默认安装位置（仅供参考）**

> rpm 包默认安装位置由该软件包的开发者决定
> rpm 包也可以通过 `--prefix=<dir>` 选项指定安装目录

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

**源码包安装位置**

> 源码包安装时需要指定安装位置，一般是 `/usr/local/软件名/`

#### **安装位置不同带来的影响**

> RPM包安装的服务可以使用系统服务管理命令（service）来管理。

* 例如RPM包安装的nginx启动方法
```
service nginx start

/etc/rc.d/init.d/nginx start
```

> 源码包安装的服务不能被服务管理命令管理，因为没有安装到默认路径中，所有只能使用绝对路径今次那个服务管理。
* 例源码包安装的nginx启动方法
```
/usr/local/nginx/sbin/nginx
```

## 二、源码包安装软件

### 0. 安装注意事项

#### **源码包保存位置：`/usr/local/src`**

#### **软件安装位置：`/usr/local`**

#### **如何确定安装过程报错**

1. 安装过程终止
2. 并出现error、warning或no的提示

> 以 nginx 为例，演示源码包安装过程

### 1. 安装准备

#### **安装C语言编译器**

    yum -y install gcc

#### **下载源码包**

* `wget http://nginx.org/download/nginx-1.14.0.tar.gz`

```
[root/usr/local/src]$wget http://nginx.org/download/nginx-1.14.0.tar.gz
--2018-05-23 09:16:19--  http://nginx.org/download/nginx-1.14.0.tar.gz
Resolving nginx.org (nginx.org)... 95.211.80.227, 206.251.255.63, 2606:7100:1:69::3f, ...
Connecting to nginx.org (nginx.org)|95.211.80.227|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1016272 (992K) [application/octet-stream]
Saving to: ‘nginx-1.14.0.tar.gz’

100%[=================================================>] 1,016,272    164KB/s   in 8.8s

2018-05-23 09:16:29 (113 KB/s) - ‘nginx-1.14.0.tar.gz’ saved [1016272/1016272]
```

#### **解压源码包**

```
[root/usr/local/src]$tar -zxf nginx-1.14.0.tar.gz
```

#### **进入解压缩目录**

```
[root/usr/local/src]$cd nginx-1.14.0

[root/usr/local/src/nginx-1.14.0]$ll
total 732K
drwxr-xr-x 6 vagrant vagrant 4.0K May 23 10:43 auto/
-rw-r--r-- 1 vagrant vagrant 281K Apr 17 15:22 CHANGES
-rw-r--r-- 1 vagrant vagrant 428K Apr 17 15:22 CHANGES.ru
drwxr-xr-x 2 vagrant vagrant 4.0K May 23 10:43 conf/
-rwxr-xr-x 1 vagrant vagrant 2.5K Apr 17 15:22 configure*
drwxr-xr-x 4 vagrant vagrant   68 May 23 10:43 contrib/
drwxr-xr-x 2 vagrant vagrant   38 May 23 10:43 html/
-rw-r--r-- 1 vagrant vagrant 1.4K Apr 17 15:22 LICENSE
drwxr-xr-x 2 vagrant vagrant   20 May 23 10:43 man/
-rw-r--r-- 1 vagrant vagrant   49 Apr 17 15:22 README
drwxr-xr-x 9 vagrant vagrant   84 May 23 10:43 src/
```

### 2. 开始安装

#### **`./configure` 软件配置与检查**

    ./configure 

    - 定义需要的功能选项
    - 监测系统环境是否符合安装要求
    - 把定义好的功能选项和检测系统环境的信息都写入`Makefile`文件，用于后续的编辑

* 查看帮助

> `./configure --help`

```
[root/usr/local/src/nginx-1.14.0]$./configure --help

  --help                             print this message

  --prefix=PATH                      set installation prefix
  --sbin-path=PATH                   set nginx binary pathname
  --modules-path=PATH                set modules path
  --conf-path=PATH                   set nginx.conf pathname
...省略...
  --with-debug                       enable debug logging
```

* 指定安装目录为 `/usr/local/nginx`，指定命令文件所在目录为 `/usr/local/sbin/nginx`

> ./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx

```
[root/usr/local/src/nginx-1.14.0]$./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx
checking for OS
 + Linux 3.10.0-229.el7.x86_64 x86_64
checking for C compiler ... found
 + using GNU C compiler
 + gcc version: 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)
checking for gcc -pipe switch ... found

...省略...

creating objs/Makefile

Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
```

* 查看生成的 `Makefile` 文件

```
[root/usr/local/src/nginx-1.14.0]$ll
total 736K
drwxr-xr-x 6 vagrant vagrant 4.0K May 23 10:43 auto/
-rw-r--r-- 1 vagrant vagrant 281K Apr 17 15:22 CHANGES
-rw-r--r-- 1 vagrant vagrant 428K Apr 17 15:22 CHANGES.ru
drwxr-xr-x 2 vagrant vagrant 4.0K May 23 10:43 conf/
-rwxr-xr-x 1 vagrant vagrant 2.5K Apr 17 15:22 configure*
drwxr-xr-x 4 vagrant vagrant   68 May 23 10:43 contrib/
drwxr-xr-x 2 vagrant vagrant   38 May 23 10:43 html/
-rw-r--r-- 1 vagrant vagrant 1.4K Apr 17 15:22 LICENSE
-rw-r--r-- 1 root    root     370 May 23 11:07 Makefile
drwxr-xr-x 2 vagrant vagrant   20 May 23 10:43 man/
drwxr-xr-x 3 root    root     119 May 23 11:07 objs/
-rw-r--r-- 1 vagrant vagrant   49 Apr 17 15:22 README
drwxr-xr-x 9 vagrant vagrant   84 May 23 10:43 src/

[root/usr/local/src/nginx-1.14.0]$cat Makefile

default:        build

clean:
        rm -rf Makefile objs

build:
        $(MAKE) -f objs/Makefile

install:
        $(MAKE) -f objs/Makefile install

modules:
        $(MAKE) -f objs/Makefile modules

upgrade:
        /usr/local/sbin/nginx -t

        kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
        sleep 1
        test -f /usr/local/nginx/logs/nginx.pid.oldbin

        kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin`

```

#### **`make` 编译**

> 若编译出错，可以使用 `make clean` 命令清除编译结果

```
[root/usr/local/src/nginx-1.14.0]$make
make -f objs/Makefile
make[1]: Entering directory `/usr/local/src/nginx-1.14.0'
cc -c -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g  -I src/core -I src/event -I src/event/modules -I src/os/un
ix -I objs \
        -o objs/src/core/nginx.o \
        src/core/nginx.c

...省略...

objs/ngx_modules.o \
-ldl -lpthread -lcrypt -lpcre -lz \
-Wl,-E
sed -e "s|%%PREFIX%%|/usr/local/nginx|" \
        -e "s|%%PID_PATH%%|/usr/local/nginx/logs/nginx.pid|" \
        -e "s|%%CONF_PATH%%|/usr/local/nginx/conf/nginx.conf|" \
        -e "s|%%ERROR_LOG_PATH%%|/usr/local/nginx/logs/error.log|" \
        < man/nginx.8 > objs/nginx.8
make[1]: Leaving directory `/usr/local/src/nginx-1.14.0'
```

#### **`make install` 编译并安装**

```
[root/usr/local/src/nginx-1.14.0]$make install
make -f objs/Makefile install
make[1]: Entering directory `/usr/local/src/nginx-1.14.0'
test -d '/usr/local/nginx' || mkdir -p '/usr/local/nginx'

...省略...

test -d '/usr/local/nginx/logs' \
        || mkdir -p '/usr/local/nginx/logs'
make[1]: Leaving directory `/usr/local/src/nginx-1.14.0'
```

#### **查看软件安装目录和命令所在目录**

* 软件安装目录
```
[root~]$ll /usr/local/nginx/
total 4.0K
drwxr-xr-x 2 root   root 4.0K May 23 11:22 conf/
drwxr-xr-x 2 root   root   38 May 23 11:22 html/
drwxr-xr-x 2 root   root   55 May 23 11:28 logs/
```
* 命令所在目录
```
[root~]$ll /usr/local/sbin/
total 3.6M
-rwxr-xr-x 1 root root 3.6M May 23 11:22 nginx*
```