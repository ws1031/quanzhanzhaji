Nginx的快速搭建和基本参数
---

## 一、Nginx简介

### 1. Nginx简述

Nginx是一个开源且高性能、可靠的HTTP中间件、代理服务。

### 2. 常见的HTTP服务

* httpd - Apache
* IIS - 微软
* GWE - Google
* tomcat - Sun

## 二、为什么选择Nginx

### 1. IO多路复用epoll

#### **什么是IO多路复用**

多个描述符的I/O操作都能在一个线程内并发交替地顺序完成，这就叫I/O多路复用，这里的“复用”指的是复用同一个线程。

#### **什么是epoll**

> IO多路服用的实现方式：select、poll、epoll

##### **select**

**基本原理：**
select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。

**select缺点：**
1.能够监视文件描述符的数量存在最大限制。
2.线性扫描效率低下。

##### **epoll**

**基本原理：**
epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次。还有一个特点是，epoll使用“事件”的就绪通知方式，通过epoll_ctl注册fd，一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd，epoll_wait便可以收到通知。
 
**epoll的优点：**
1.没有最大并发连接的限制，能打开的FD的上限远大于1024（1G的内存上能监听约10万个端口）。
2.效率提升，不是轮询的方式，不会随着FD数目的增加效率下降。
3.内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销。

### 2. 轻量级

#### **功能模块少**

#### **代码模块化**

### 3. CPU亲和性（affinity）好

CPU亲和性（affinity）是一种把CPU核心和Nginx工作进程绑定方式，把每个worker进程固定在一个CPU上执行，减少CPU的cache miss，获得更好的性能。

### 4. sendfile

sendfile可以让Nginx在传输文件时直接在磁盘和tcp socket之间传输数据。如果这个参数不开启，会先在用户空间（Nginx进程空间）申请一个buffer，用read函数把数据从磁盘读到cache，再从cache读取到用户空间的buffer，再用write函数把数据从用户空间的buffer写入到内核的buffer，最后到tcp socket。

![一般web server传输数据方式](http://md.ws1031.cn/xsj/2018_7_11_2018-07-11_170606.jpg)

![sendfile](http://md.ws1031.cn/xsj/2018_7_11_2018-07-11_170543.jpg)


## 三、Nginx的快速搭建和基本参数（CentOS7）

### 1. yum方式安装【[参考](http://nginx.org/en/linux_packages.html#stable)】

#### **创建`/etc/yum.repos.d/nginx.repo`文件，并输入如下内容**

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/OS/OSRELEASE/$basearch/
gpgcheck=0
enabled=1
```
> `OS` 可选值有 `centos` 和 `rhel`。
> `OSRELEASE` 为系统版本，例如 `6` 和 `7` 分别代表 6.x 和 7.x 的版本。

#### **运行 `yum install -y nginx` 安装nginx**

#### **运行 `nginx -v` 查看nginx版本**
```
[root~]# nginx -v
nginx version: nginx/1.14.0
```

### 2. 编译参数详解

#### **查看nginx安装时的编译参数**

    nginx -V

```shell
[root~]# nginx -V
nginx version: nginx/1.14.0
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled

configure arguments: 
--prefix=/etc/nginx 
--sbin-path=/usr/sbin/nginx 
--modules-path=/usr/lib64/nginx/modules 
--conf-path=/etc/nginx/nginx.conf 
--error-log-path=/var/log/nginx/error.log 
--http-log-path=/var/log/nginx/access.log 
--pid-path=/var/run/nginx.pid 
--lock-path=/var/run/nginx.lock 
--http-client-body-temp-path=/var/cache/nginx/client_temp 
--http-proxy-temp-path=/var/cache/nginx/proxy_temp 
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp 
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp 
--http-scgi-temp-path=/var/cache/nginx/scgi_temp 
--user=nginx 
--group=nginx 
--with-compat 
--with-file-aio 
--with-threads 
--with-http_addition_module 
--with-http_auth_request_module 
--with-http_dav_module 
--with-http_flv_module 
--with-http_gunzip_module 
--with-http_gzip_static_module 
--with-http_mp4_module 
--with-http_random_index_module 
--with-http_realip_module 
--with-http_secure_link_module 
--with-http_slice_module 
--with-http_ssl_module 
--with-http_stub_status_module 
--with-http_sub_module 
--with-http_v2_module 
--with-mail 
--with-mail_ssl_module 
--with-stream 
--with-stream_realip_module 
--with-stream_ssl_module 
--with-stream_ssl_preread_module 
--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -fPIC' 
--with-ld-opt='-Wl,-z,relro -Wl,-z,now -pie'
```

#### **安装编译参数详解【[参考](http://nginx.org/en/docs/configure.html)】**

编译选项|作用
-|-
`--prefix=/etc/nginx` | 配置文件目录
`--sbin-path=/usr/sbin/nginx` | 可执行文件名称和所在目录
`--modules-path=/usr/lib64/nginx/modules` |nginx动态模块的安装目录
`--conf-path=/etc/nginx/nginx.conf`  | 主配置文件名称和所在目录
`--error-log-path=/var/log/nginx/error.log` | 全局错误日志文件名称和所在目录
`--http-log-path=/var/log/nginx/access.log` | HTTP服务器的主请求日志文件的名称和所在目录
`--pid-path=/var/run/nginx.pid` |nginx.pid所在目录，这是储存主进程的进程ID文件
`--lock-path=/var/run/nginx.lock` | nginx.lock所在目录
-|-
`--http-client-body-temp-path=/var/cache/nginx/client_temp`<br> `--http-proxy-temp-path=/var/cache/nginx/proxy_temp`<br> `--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp`<br> `--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp`<br> `--http-scgi-temp-path=/var/cache/nginx/scgi_temp` <br>|执行对应模块时nginx所保留的临时文件
-|-
`--user=nginx`<br> `--group=nginx` |设定Nginx进程启动的用户和用户组
-|-
`--with-http_random_index_module`|目录中随机选择一个随机主页
`--with-http_stub_status_module`|Nginx客户端状态
`--with-http_sub_module`|	HTTP内容替换
`--with-cc-opt=<parameters>`| 设置额外的参数将被添加到CFLAGS变量
`--with-ld-opt=<parameters>`|设置附加参数，链接系统库

### 3. 安装目录详解

#### **查看nginx所有文件的安装位置**

    rpm -ql nginx

```shell
[root~]# rpm -ql nginx
/etc/logrotate.d/nginx
/etc/nginx
/etc/nginx/nginx.conf
/etc/nginx/conf.d
/etc/nginx/conf.d/default.conf
/etc/nginx/fastcgi_params
/etc/nginx/scgi_params
/etc/nginx/uwsgi_params
/etc/nginx/koi-utf
/etc/nginx/koi-win
/etc/nginx/win-utf
/etc/nginx/mime.types
/etc/sysconfig/nginx
/etc/sysconfig/nginx-debug
/usr/lib/systemd/system/nginx-debug.service
/usr/lib/systemd/system/nginx.service
/usr/lib64/nginx
/usr/lib64/nginx/modules
/etc/nginx/modules
/usr/sbin/nginx
/usr/sbin/nginx-debug
/usr/share/doc/nginx-1.14.0
/usr/share/doc/nginx-1.14.0/COPYRIGHT
/usr/share/man/man8/nginx.8.gz
/usr/share/nginx
/usr/share/nginx/html
/usr/share/nginx/html/50x.html
/usr/share/nginx/html/index.html
/var/cache/nginx
/var/log/nginx
/usr/libexec/initscripts/legacy-actions/nginx
/usr/libexec/initscripts/legacy-actions/nginx/check-reload
/usr/libexec/initscripts/legacy-actions/nginx/upgrade
```

默认路径|类型|作用
-|-|-
`/etc/logrotate.d/nginx`|配置文件|Nginx日志轮转，用于logrotate服务的日志切割
`/etc/nginx`<br>`/etc/nginx/nginx.conf`<br>`/etc/nginx/conf.d`<br>`/etc/nginx/conf.d/default.conf`  |目录、配置文件|nginx主配置文件
`/etc/nginx/fastcgi_params`<br>`/etc/nginx/uwsig_params`<br>`/etc/nginx/scgi_params`|配置文件|cgi配置相关，fastcgi配置
`/etc/nginx/koi-utf`<br>`/etc/nginx/koi-win`<br>`/etc/nginx/win-utf`|配置文件|编码转换映射转化文件
`/etc/nginx/mime.types`|配置文件|设置http协议的Content-Type与扩展名对应关系
`/usr/lib/systemd/system/nginx-debug.service`<br>`/usr/lib/systemd/system/nginx.service`<br>`/etc/sysconfig/nginx`<br>`/etc/sysconfig/nginx-debug`|配置文件|用于配置出系统守护进程管理器管理方式
`/usr/lib64/nginx/modules`<br>`/etc/nginx/mudules`|目录|Nginx模块目录
`/usr/sbin/nginx`<br>`/usr/sbin/nginx-debug`|命令|Nginx服务的启动管理的终端命令
`/usr/share/doc/nginx-1.14.0`<br>`/usr/share/doc/nginx-1.14.0/COPYRIGHT`<br>`/usr/share/man/man8/nginx.8.gz`|文件、目录|Nginx手册和帮助文件
`/var/cache/nginx`|目录|Nginx缓存目录
`/var/log/nginx`|目录|Nginx日志目录


### 4. Nginx常用命令

命令|解释
-|-
`nginx [-c <配置文件>]` |以指定的配置文件启动nginx
`nginx -s quit`|正常停止nginx，Nginx在退出前完成已经接受的连接请求。
`nginx -s stop`|快速停止nginx，不管有没有正在处理的请求。
`nginx -s reload [-c <配置文件>]`|重载配置文件
`nginx -s reopen`|重新打开日志文件
`nginx -v` |查看版本
`nginx -V` |查看安装时的编译参数
`nginx -t [-c <配置文件>]` | 检查配置文件语法是否正确

#### **`nginx -s  reload` 命令加载修改后的配置文件,命令下达后发生如下事件**

> 1. Nginx的master进程检查配置文件的正确性，若是错误则返回错误信息，nginx继续采用原配置文件进行工作（因为worker未受到影响）
> 2. Nginx启动新的worker进程，采用新的配置文件
> 3. Nginx将新的请求分配新的worker进程
> 4. Nginx等待以前的worker进程的全部请求都返回后，关闭相关worker进程
> 5. 重复上面过程，直到全部旧的worker进程都被关闭掉

