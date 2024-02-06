Nginx请求限制和访问控制
---

## 一、Nginx的请求限制

### 1. HTTP协议的连接与请求


#### HTTP协议版本与连接关系


HTTP协议版本|连接关系
-|-
 HTTP1.0|TCP不能复用
 HTTP1.1|顺序性TCP复用
 HTTP2.0|多路复用TCP复用

> HTTP请求建立在一次TCP连接的基础上。
> 一次TCP连接至少可以产生一次HTTP请求，HTTP1.1版本以后，建立一次TCP连接可以发送多次HTTP请求。

![HTTP协议的连接与请求](http://md.ws1031.cn/xsj/2018_7_9_2018-07-09_174028.jpg)
 
### 1. 连接频率限制

#### **[ngx_http_limit_conn_module](http://nginx.org/en/docs/http/ngx_http_limit_conn_module.html)**

#### **语法**

	Syntax:		limit_conn_zone key zone=name:size;
	Default:	—
	Context:	http

	Syntax:		limit_conn zone number;
	Default:	—
	Context:	http, server, location


#### **用法**

* 在nginx配置文件中的 http 下配置

```nginxconf
http {
    # ...其它代码省略...
	# 开辟一个10m的连接空间，命名为addr
	limit_conn_zone $binary_remote_addr zone=addr:10m;
    server {
        ...
        location /download/ {
			# 服务器每次只允许一个IP地址连接
            limit_conn addr 1;
        }
	}
}
```

### 2. 请求频率限制

#### **[ngx_http_limit_req_module](http://nginx.org/en/docs/http/ngx_http_limit_req_module.html)**

#### **语法**

	Syntax:		limit_req_zone key zone=name:size rate=rate;
	Default:	—
	Context:	http
	
	
	Syntax:		limit_req zone=name [burst=number] [nodelay];
	Default:	—
	Context:	http, server, location

#### **用法**

* 在nginx配置文件中的 http 下配置
```nginxconf
http {

	# ...其它代码省略...
	
	# 开辟一个10m的请求空间，命名为one。同一个IP发送的请求，平均每秒只处理一次
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
	
    server {
   	 	...

		location /search/ {
			limit_req zone=one;
			# 当客户端请求超过指定次数，最多宽限5次请求，并延迟处理，1秒1个请求
			# limit_req zone=one burst=5;
			# 当客户端请求超过指定次数，最多宽限5次请求，并立即处理。
			# limit_req zone=one burst=5 nodelay;

		}
	}
}
```

## 二、Nginx的访问控制

### 1. 基于IP的访问控制

#### [ngx_http_access_module](http://nginx.org/en/docs/http/ngx_http_access_module.html)

#### **语法**

	Syntax:		allow address | CIDR | unix: | all;
	Default:	—
	Context:	http, server, location, limit_except
	
	Syntax:		deny address | CIDR | unix: | all;
	Default:	—
	Context:	http, server, location, limit_except

> `address`：IP地址，例如：192.168.1.1
> `CIDR`：例如：192.168.1.0/24;
> `unix`：Socket方式
> `all`：所有

#### **用法**

* 在nginx配置文件中的 server 下配置
```nginxconf
server {
    # ...其它代码省略...
    location ~ ^/index_1.html {
        root   /usr/share/nginx/html;
        deny 151.19.57.60; # 拒绝这个IP访问
        allow all; # 允许其他所有IP访问
    }

    location ~ ^/index_2.html {
        root   /usr/share/nginx/html;
        allow 151.19.57.0/24; # 允许IP 151.19.57.* 访问
        deny all; # 拒绝其他所有IP访问
    }
}
```

#### **ngx_http_access_module 的局限性**

* 当客户端通过代理访问时，nginx的remote_addr获取的是代理的IP

![当客户端通过代理访问时，nginx的remote_addr获取的是代理的IP](http://md.ws1031.cn/xsj/2018_7_9_2018-07-09_190309.jpg)

* http_x_forwarded_for

	http_x_forwarded_for = Client IP, Proxy1 IP, Proxy2 IP, ...

`remote_addr` 获取的是直接和服务端建立连接的客户端IP。
`http_x_forwarded_for` 可以记录客户端及所有中间代理的IP

![http_x_forwarded_for和remote_addr区别](http://md.ws1031.cn/xsj/2018_7_9_2018-07-09_190639.jpg)

### 2. 基于用户的登录认证

#### [ngx_http_auth_basic_module](http://nginx.org/en/docs/http/ngx_http_access_module.html)


#### **语法**

	Syntax:		auth_basic string | off;
	Default:	auth_basic off;
	Context:	http, server, location, limit_except


	Syntax:		auth_basic_user_file file;
	Default:	—
	Context:	http, server, location, limit_except

#### **用法**

* 要使用 htpasswd 命令，需要先安装httpd-tools
```
[root~]# yum -y install httpd-tools
```

* 使用 htpasswd 命令创建账号密码文件
```shell
[root/etc/nginx]# htpasswd -c ./auth_conf auth_root
New password:
Re-type new password:
Adding password for user auth_root

[root/etc/nginx]# ll auth_conf
-rw-r--r-- 1 root root 48 7月   9 11:38 auth_conf

[root/etc/nginx]# cat auth_conf
auth_root:$apr1$2v6gftlm$oo2LE8glGQWi68MCqtcN90
```

* 在nginx配置文件中的 server 下配置
```nginxconf
server {
    # ...其它代码省略...
	
    location ~ ^/index.html {
        root   /usr/share/nginx/html;
        auth_basic "Auth access! Input your password!";
		auth_basic_user_file /etc/nginx/auth_conf;
    }
}
```

* 修改后重新载入配置文件`nginx -s reload`
* 使用浏览器访问 `http://192.168.33.88/index.html`

![基于用户的登录认证](http://md.ws1031.cn/xsj/2018_7_9_2018-07-09_194739.jpg)

输入正确的用户名和密码，即可正常访问。

#### **ngx_http_auth_basic_module 的局限性**

* 用户信息依赖文件方式
* 操作管理效率低下
