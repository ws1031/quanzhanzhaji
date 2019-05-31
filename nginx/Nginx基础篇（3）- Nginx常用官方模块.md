Nginx常用官方模块
---

    Nginx采用模块化的架构，Nginx中大部分功能都是通过模块方式提供的，比如HTTP模块、Mail模块等。

[Nginx官方模块文档](http://nginx.org/en/docs/)

### 1. [ngx_http_stub_status_module](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html)

#### **编译选项**

    --with-http_stub_status_module

#### **作用**

提供Nginx当前处理连接等基本状态信息的访问

#### **语法**

    Syntax:		stub_status;
    Default:	—
    Context:	server, location

#### **用法**

* 在nginx配置文件中的 server 下配置
```nginxconf
server {
    # 添加的配置
    location /nginx_status {
        stub_status;
    }
    
    ...其它代码省略...
}
```
* 修改后重新载入配置文件`nginx -s reload`
* 在浏览器中访问 `http://<ip>/nginx_status`，会返回如下内容


    Active connections: 3 
    server accepts handled requests
     7 7 16 
    Reading: 0 Writing: 1 Waiting: 2 

> Active connections: Nginx当前活跃链接数
> accepts: 接收客户端连接的总次数
> handled: 处理客户端连接的总次数。一般来说，这个参数值与accepts相同，除非已经达到了一些资源限制（例如[worker_connections](http://nginx.org/en/docs/ngx_core_module.html#worker_connections)限制）
> requests: 客户端请求的总次数
> Reading: 当前nginx正在读取请求头的连接数
> Writing: 当前nginx正在写入响应的连接数
> Reading: 当前正在等待请求的空闲客户端连接数。一般是在nginx开启长连接（keep alive）情况下出现。

### 2. [ngx_http_random_index_module](http://nginx.org/en/docs/http/ngx_http_random_index_module.html)

#### **编译选项**

    --with-http_random_index_module

#### **作用**

在主目录中选择一个随机文件作为主页

#### **语法**

    Syntax:		random_index on | off;
    Default:	random_index off;
    Context:	location

#### **用法**

* 在nginx配置文件中的 server 下配置
```nginxconf
server {
    location / {
        root /usr/share/nginx/html;
        #添加这一行开启随机主页模块
        random_index on;
        #把指定的主页注释掉
        #index index.html index.htm;
    }
    
    ...其它代码省略...
}
```

### 3. [ngx_http_sub_module](http://nginx.org/en/docs/http/ngx_http_sub_module.html)

#### **编译选项**

    --with-ngx_http_sub_module

#### **作用**

通过替换一个指定的字符串来修改响应

#### **语法**

* 指定被替换的字符和替代字符

		Syntax:    sub_filter string replacement;
		Default:   —
		Context:	http, server, location


* Last-Modified，用于校验服务端内容是否更改，主要用于缓存场景

		Syntax:	   sub_filter_last_modified on | off;
		Default:   sub_filter_last_modified off;
		Context:	http, server, location

* 默认只替换找到的第一个字符串，若替换文本中的所有匹配的字符串，则置为off

		Syntax:	   sub_filter_once on | off;
		Default:	sub_filter_once on;
		Context:	http, server, location

* 除了“text/html”之外，还可以用指定的MIME类型替换字符串。特殊值‘*’匹配任意MIME类型

		Syntax:	   sub_filter_types mime-type ...;
		Default:	sub_filter_types text/html;
		Context:	http, server, location

#### **用法**

* 在nginx配置文件中的 server 下配置
```
server {
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
        # 将首页的nginx替换为home
        sub_filter 'nginx' 'home';
        # 不止替换第一个，而是替换response中所有的nginx
        sub_filter_once off;
    }
    
    ...其它代码省略...
}
```
* 修改后重新载入配置文件`nginx -s reload`
* `curl localhost`，返回如下内容，会发现响应中所有nginx已经替换为home
```shell
[vagrant/etc/nginx]$ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to home!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to home!</h1>
<p>If you see this page, the home web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://home.org/">home.org</a>.<br/>
Commercial support is available at
<a href="http://home.com/">home.com</a>.</p>

<p><em>Thank you for using home.</em></p>
</body>
</html>
```