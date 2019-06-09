## 一、控制浏览器缓存

### 1. 浏览器缓存简介

浏览器缓存遵循HTTP协议定义的缓存机制（如：Expires;Cache-control等）。

#### **当浏览器无缓存时，请求响应流程**

![当浏览器无缓存时，请求响应流程](http://md.ws65535.top/xsj/2018_7_11_2018-07-11_191944.jpg)

#### **当浏览器有缓存时，请求响应流程**

![当浏览器有缓存时，请求响应流程](http://md.ws65535.top/xsj/2018_7_11_2018-07-11_192534.jpg)

#### **浏览器缓存校验过期机制**

校验是否过期|Cache-Control(max-age)、Expires
-|-
协议中Etag头信息校验|Etag
Last-Modified头信息校验|Last-Modified

#### **浏览器请求流程**
![浏览器请求流程](http://md.ws65535.top/xsj/2018_7_11_2018-07-11_193429.jpg)

### 2. Nginx控制浏览器缓存配置

> Nginx通过添加Cache-Control(max-age)、Expires头信息的方式控制浏览器缓存。

#### **[ngx_http_headers_module](http://nginx.org/en/docs/http/ngx_http_headers_module.html#expires)**

#### **语法**

	Syntax:	expires [modified] time;
			expires epoch | max | off;
	Default:	expires off;
	Context:	http, server, location, if in location

> 本配置项可以控制HTTP响应中的“Expires”和“Cache-Control”头信息，（起到控制页面缓存的作用）。

> “Expires”头信息中的过期时间为当前系统时间与您设定的 time 值时间的和。如果指定了 modified 参数，则过期时间为文件的最后修改时间与您设定的 time 值时间的和。
> “Cache-Control”头信息的内容取决于指定 time 的符号。可以在time值中使用正数或负数。
> 当 time 为负数，“Cache-Control: no-cache”;
> 当 time 为正数或0，“Cache-Control: max-age=time”，单位是秒。
> 
> `epoch` 参数用于指定“Expires”的值为 1 January, 1970, 00:00:01 GMT。
> `max` 参数用于指定“Expires”的值为 “Thu, 31 Dec 2037 23:55:55 GMT”，“Cache-Control” 的值为10 年。
> `off` 参数令对“Expires” 和 “Cache-Control”响应头信息的添加或修改失效。

### 3. 应用实例

#### **1. vim /etc/nginx/conf.d/static.conf**

```nginxconf
server {
    location ~ .*\.(txt|xml)$ {
		# 设置过期时间为1天
        expires 1d;
        root /vagrant/doc;
    }
}
```

#### **2. nginx -s reload 重新载入nginx配置文件**

#### **3. 创建 `/vagrant/doc/hello.txt` 文件**

#### **4. 通过curl访问 192.168.33.88/hello.txt，查看http响应头信息**

```
[root/etc/nginx]# curl -I 192.168.33.88/hello.txt
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Tue, 17 Jul 2018 07:12:11 GMT
Content-Type: text/plain
Content-Length: 12
Last-Modified: Tue, 17 Jul 2018 07:07:22 GMT
Connection: keep-alive
ETag: "5b4d95aa-c"
Expires: Wed, 18 Jul 2018 07:12:11 GMT
Cache-Control: max-age=86400
Accept-Ranges: bytes

```
重点查看 `Expires` 和 `Cache-Control`两个字段，可见，hello.txt 的缓存时间为1天。

## 二、防盗链

> 目的：防止资源被盗用
> 思路：区别哪些请求是非正常的用户请求

### 1. 基于http_refer防盗链配置模块

#### **[ngx_http_referer_module](http://nginx.org/en/docs/http/ngx_http_referer_module.html)**

#### **语法**

	Syntax:	valid_referers none | blocked | server_names | string ...;
	Default:	—
	Context:	server, location
	
> `none`：请求头中没有 Referer 字段
> `blocked`：请求头中虽然存在“Referer”字段，但是它的值已经被防火墙或代理服务器删除；这些值是不以“http://”或“https://”开头的字符串;
> `server_names`：“Referer”请求头字段包含该服务器名称
> 任意字符串：定义一个服务器名称和一个可选的URI前缀。服务器名开始或结尾可以有 “*” 。检查时，“Referer”字段中的服务器端口会被忽略。
> 正则表达式：字符串必须以 `~` 开头，值得注意的是，正则表达式匹配的是在“http://”或“https://”之后的内容。

#### 示例

	valid_referers none blocked server_names *.example.com example.* www.example.org/galleries/ ~\.google\.;

### 2. 应用实例

#### **1. vim conf.d/static.conf**

```nginxconf
server {
    location ~ .*\.(txt|xml)$ {
		
		# 配置防盗链规则
        valid_referers none blocked 192.168.1.110 *.example.com example.* ~\.google\.;

		# 如果不符合防盗链规则，则返回403
        if ($invalid_referer) {
            return 403;
        }

        root /vagrant/doc;
    }
}
```

#### **2. nginx -s reload 重新载入nginx配置文件**

#### **3. 创建 `/vagrant/doc/hello.txt` 文件**

* vim /vagrant/a/a.txt
```
Hello world!
```

#### **4. 使用 curl进行访问测试**

* 不带referer，可以正常访问
```
[root~]# curl -I http://127.0.0.1/hello.txt
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Fri, 03 Aug 2018 01:34:12 GMT
Content-Type: text/plain
Content-Length: 12
Last-Modified: Tue, 17 Jul 2018 07:07:22 GMT
Connection: keep-alive
ETag: "5b4d95aa-c"
Accept-Ranges: bytes
```

* referer为 `http://www.baidu.com`，返回403 
```
[root~]# curl -e "http://www.baidu.com" -I http://127.0.0.1/hello.txt
HTTP/1.1 403 Forbidden
Server: nginx/1.14.0
Date: Fri, 03 Aug 2018 01:34:34 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive
```

* referer为 `http://192.168.1.110`，可以正常访问
```
[root~]# curl -e "http://192.168.1.110" -I http://127.0.0.1/hello.txt
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Thu, 02 Aug 2018 11:31:51 GMT
Content-Type: text/plain
Content-Length: 12
Last-Modified: Tue, 17 Jul 2018 07:07:22 GMT
Connection: keep-alive
ETag: "5b4d95aa-c"
Accept-Ranges: bytes
```

* referer以 `example.`开头或 `.example.com` 结尾，可以正常访问
```
[root~]# curl -e "http://www.example.com" -I http://127.0.0.1/hello.txt
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Thu, 02 Aug 2018 11:33:47 GMT
Content-Type: text/plain
Content-Length: 12
Last-Modified: Tue, 17 Jul 2018 07:07:22 GMT
Connection: keep-alive
ETag: "5b4d95aa-c"
Accept-Ranges: bytes

[root~]# curl -e "http://example.baidu.com" -I http://127.0.0.1/hello.txt
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Thu, 02 Aug 2018 11:33:53 GMT
Content-Type: text/plain
Content-Length: 12
Last-Modified: Tue, 17 Jul 2018 07:07:22 GMT
Connection: keep-alive
ETag: "5b4d95aa-c"
Accept-Ranges: bytes
```

* referer为 `http://192.168.1.110`，可以正常访问
```
[root~]# curl -e "http://192.168.1.110" -I http://127.0.0.1/hello.txt
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Thu, 02 Aug 2018 11:31:51 GMT
Content-Type: text/plain
Content-Length: 12
Last-Modified: Tue, 17 Jul 2018 07:07:22 GMT
Connection: keep-alive
ETag: "5b4d95aa-c"
Accept-Ranges: bytes
```

* referer为 `http://google.com`，返回403 
```
[root~]# curl -e "http://google.com" -I http://127.0.0.1/hello.txt
HTTP/1.1 403 Forbidden
Server: nginx/1.14.0
Date: Thu, 02 Aug 2018 11:37:43 GMT
Content-Type: text/html
Content-Length: 169
Connection: keep-alive
```

* referer为 `http://www.google.com`，可以正常访问
```
[root~]# curl -e "http://www.google.com" -I http://127.0.0.1/hello.txt
HTTP/1.1 200 OK
Server: nginx/1.14.0
Date: Thu, 02 Aug 2018 11:37:50 GMT
Content-Type: text/plain
Content-Length: 12
Last-Modified: Tue, 17 Jul 2018 07:07:22 GMT
Connection: keep-alive
ETag: "5b4d95aa-c"
Accept-Ranges: bytes
```
