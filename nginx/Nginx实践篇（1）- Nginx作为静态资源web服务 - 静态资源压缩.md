## 一、静态资源web服务

![静态资源web服务](http://md.ws65535.top/xsj/2018_7_10_2018-07-10_182440.jpg)

### 1. 静态资源类型

类型|文件类型
-|-
浏览器端渲染|HTML、CSS、JS
图片|JEPG、GIF、PNG
视频|FLV、MPEG
文件|TXT等其他下载文件

### 2. 静态资源服务场景-CDN

![静态资源服务场景-CDN](http://md.ws65535.top/xsj/2018_7_10_2018-07-10_184108.jpg)


## 二、静态资源核心配置

### 1. 文件读取 sendfile

sendfile 是一种高效传输文件的模式.
sendfile设置为on表示启动高效传输文件的模式。sendfile可以让Nginx在传输文件时直接在磁盘和tcp socket之间传输数据。如果这个参数不开启，会先在用户空间（Nginx进程空间）申请一个buffer，用read函数把数据从磁盘读到cache，再从cache读取到用户空间的buffer，再用write函数把数据从用户空间的buffer写入到内核的buffer，最后到tcp socket。开启这个参数后可以让数据不用经过用户buffer。

![未开启sendfile](http://md.ws65535.top/xsj/2018_7_11_2018-07-11_170606.jpg)

![开启sendfile](http://md.ws65535.top/xsj/2018_7_11_2018-07-11_170543.jpg)

#### 语法

    Syntax:		sendfile on | off;
    Default:	sendfile off;
    Context:	http, server, location, if in location


### 2. tcp_nopush

在 sendfile 开启的情况下，提高网络数据包的传输效率。
tcp_nopush指令，在连接套接字时启用Linux系统下的TCP_CORK。该选项告诉TCP堆栈附加数据包，并在它们已满或当应用程序通过显式删除TCP_CORK指示发送数据包时发送它们。 这使得发送的数据分组是最优量，并且因此提高了网络数据包的传输效率。
也就是说 tcp_nopush=on 时，结果就是数据包不会马上传送出去，等到数据包最大时，一次性的传输出去，这样有助于解决网络堵塞，虽然有一点点延迟。

#### 语法

    Syntax:		tcp_nopush on | off;
    Default:	tcp_nopush off;
    Context:	http, server, location


### 3. tcp_nodelay

在 keepalive 连接下，提高网络数据包的传输实时性。
tcp_nodelay选项和tcp_nopush正好相反，数据包不等待，实时发送给用户。

#### 语法

    Syntax:		tcp_nodelay on | off;
    Default:	tcp_nodelay off;
    Context:	server, location
	
	
### 4. 压缩

开启压缩，可以加快资源响应速度，同时节省网络带宽资源。

![压缩和解压缩](http://md.ws65535.top/xsj/2018_7_11_2018-07-11_174549.jpg)

#### [ngx_http_gzip_module](http://nginx.org/en/docs/http/ngx_http_gzip_module.html)

#### 语法

开启关闭压缩

    Syntax:		gzip on | off;
    Default:	gzip off;
    Context:	http, server, location, if in location
	
压缩等级配置（压缩等级越高，越消耗服务器资源）

	Syntax:	gzip_comp_level level;
	Default:	gzip_comp_level 1;
	Context:	http, server, location

gzip协议版本配置

	Syntax:	gzip_http_version 1.0 | 1.1;
	Default:	gzip_http_version 1.1;
	Context:	http, server, location

#### 压缩扩展模块

预读gzip功能 [ngx_http_gzip_static_module](http://nginx.org/en/docs/http/ngx_http_gzip_static_module.html)

	Syntax:	gzip_static on | off | always;
	Default:	gzip_static off;
	Context:	http, server, location

应用支持gunzip的压缩方式 [ngx_http_gunzip_module](http://nginx.org/en/docs/http/ngx_http_gunzip_module.html)

	Syntax:	gunzip on | off;
	Default:	gunzip off;
	Context:	http, server, location

	Syntax:	gunzip_buffers number size;
	Default:	gunzip_buffers 32 4k|16 8k;
	Context:	http, server, location

## 三、静态资源压缩实例

#### **1. vim /etc/nginx/conf.d/static.conf**

```
server {
    #开启sendfile，提高网络包的传输效率
    sendfile on;

    #配置txt|xml资源的路径
    location ~ .*\.(txt|xml)$ {
        #开启压缩
        gzip on;
        gzip_http_version 1.1;
        gzip_comp_level 1;
        gzip_types text/plain application/xml;
        root /vagrant/doc;
    }
}
```

#### **2. nginx -s reload 重新载入nginx配置文件**

#### **3. 创建 `/vagrant/doc/a.txt` 文件，并查看文件大小**

```
[root/etc/nginx]# curl http://www.sina.com.cn/ > /vagrant/doc/a.txt
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  557k  100  557k    0     0   488k      0  0:00:01  0:00:01 --:--:--  488k

[root/etc/nginx]# ll /vagrant/doc/a.txt
-rwxrwxrwx 1 vagrant vagrant 558K 7月  11 10:57 /vagrant/doc/a.txt*
```
可见，a.txt 文件大小为 558K

#### **4. 通过curl访问 192.168.33.88/a.txt，查看http响应头信息**

```
[root/etc/nginx]# curl -I 192.168.33.88/a.txt -H Accept-Encoding:gzip,defalte
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Wed, 11 Jul 2018 11:01:43 GMT
Content-Type: text/plain
Last-Modified: Wed, 11 Jul 2018 10:57:22 GMT
Connection: keep-alive
ETag: W/"5b45e292-8b47f"
Content-Encoding: gzip
```
从响应头信息中可看出服务器使用了gzip压缩

#### **5. 通过浏览器访问 192.168.33.88/a.txt，使用开发者工具查看请求文件的大小**

![请求文件信息](http://md.ws65535.top/xsj/2018_7_11_2018-07-11_190500.jpg)

可见，经过gzip压缩，请求文件由558K被压缩到148K，压缩比例很高。

#### **6. 另外还可以通过nginx的access.log日志查看传输文件的大小**

```
[root/etc/nginx]# tail /var/log/nginx/access.log
192.168.33.1 - - [11/Jul/2018:11:02:46 +0000] "GET /a.txt HTTP/1.1" 200 151549 "-" "Chrome/67.0.3396.99" "-"
```

可看出传输文件大小为 151549，单位是B，换算成KB约为148KB。
