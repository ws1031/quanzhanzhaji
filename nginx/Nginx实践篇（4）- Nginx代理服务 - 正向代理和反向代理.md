## 一、代理简介

### 1. 代理
![代理](http://md.ws1031.cn/xsj/2018_8_2_2018-07-17_173831.jpg)

### 2. Nginx代理服务
![Nginx代理服务](http://md.ws1031.cn/xsj/2018_8_2_2018-07-17_174046.jpg)

### 3. 正向代理和反向代理

区别在于代理的对象不一样。

#### **正向代理代理的对象是客户端**

![正向代理代理的对象是客户端](http://md.ws1031.cn/xsj/2018_8_2_2018-07-17_174149.jpg)

#### **反向代理代理的对象是服务端**

![反向代理代理的对象是服务端](http://md.ws1031.cn/xsj/2018_8_2_2018-07-17_174219.jpg)

### 4. Nginx代理模块 [ngx_http_proxy_module](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)

#### **语法**

	Syntax:	proxy_pass URL;
	Default:	—
	Context:	location, if in location, limit_except
	
URL支持：
1. http：`http://localhost:8000/uri/`
2. https：`https://192.168.1.111:8000/uri/`
3. socket：`http://unix:/tmp/backend.socket:/uri/`


## 二、反向代理实例

#### **1. 创建真实要访问的服务配置：`vim conf.d/real_server.conf`**

```nginxconf
server {
	# 监听8080端口
    listen 8080;

    location / {
		# 配置访问根目录为 /vagrant/proxy
        root /vagrant/proxy;
    }
}
```

#### **2. 创建反向代理配置 `vim conf.d/fx_proxy.conf`**

```nginxconf
server {
	# 监听80端口
    listen 80;
    server_name localhost;

    location ~ /fx_proxy.html {
		# 设置反向代理，将访问 /fx_proxy.html 的请求转发到 http://127.0.0.1:8080
        proxy_pass http://127.0.0.1:8080;
    }
}
```

#### **3. nginx -s reload 重新载入nginx配置文件**

#### **4. 创建 `/vagrant/proxy/fx_proxy.html` 文件**

* vim /vagrant/proxy/fx_proxy.html
```
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="utf-8">
		<title>反向代理</title>
	</head>
	<body>
		<h1>反向代理</h1>
	</body>
</html>
```

#### **5. 使用 `ss -tln` 查看 80 端口和 8080 端口全部开启**
```
[root~]# ss -tln
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port
LISTEN      0      128               *:8080                          *:*
LISTEN      0      128               *:80                            *:*
LISTEN      0      128               *:22                            *:*
LISTEN      0      10        127.0.0.1:25                            *:*
LISTEN      0      128              :::22                           :::*
```

#### **6. 使用 curl进行访问测试**

* `http://127.0.0.1/fx_proxy.html`可以正常访问
```
[root~]# curl http://127.0.0.1/fx_proxy.html
<!DOCTYPE html>
<html lang="en">
        <head>
                <meta charset="utf-8">
                <title>反向代理</title>
        </head>
        <body>
                <h1>反向代理</h1>
        </body>
</html>
```

* `http://127.0.0.1:8080/fx_proxy.html`可以正常访问
```
[root~]# curl http://127.0.0.1:8080/fx_proxy.html
<!DOCTYPE html>
<html lang="en">
        <head>
                <meta charset="utf-8">
                <title>反向代理</title>
        </head>
        <body>
                <h1>反向代理</h1>
        </body>
</html>
```


## 三、正向代理实例

> 正向代理须在有公网IP的正式的服务器上测试。
> 笔者远程服务器的IP地址为：39.106.178.166，测试用的域名为 `zx_proxy.ws65535.top`

#### **1. 在服务器创建真实要访问的服务配置：`vim conf.d/real_server.conf`**

```nginxconf
server {
	# 监听80端口
    listen 80;
	# 域名为 zx_proxy.ws65535.top;
    server_name  zx_proxy.ws65535.top;

    location / {
		# $http_x_forwarded_for 可以记录客户端及所有中间代理的IP
		# 判断客户端IP地址是否是 39.106.178.166，不是则返回403
        if ($http_x_forwarded_for !~* "^39\.106\.178\.166") {
            return 403;
        }
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
```

#### **2. nginx -s reload 重新载入nginx配置文件**
#### **3. 在本地使用浏览器访问 `http://zx_proxy.ws65535.top/`，返回 `403 Forbidden`，说明访问被拒绝**

![enter description here](http://md.ws1031.cn/xsj/2018_8_3_2018-08-03_104422.jpg)

#### **4. 在服务器创建代理服务配置：`vim conf.d/zx_proxy.conf`**
```
server {
	# 代理服务监听的端口（注意，一定要看服务器供应商控制台的安全组是否开启了该端口）
    listen 3389;

	# 配置DNS，223.5.5.5是阿里云的DNS
    resolver 223.5.5.5;
	
	# 正向代理配置
    location / {
        proxy_pass http://$http_host$request_uri;
    }
}
```
#### **5. nginx -s reload 重新载入nginx配置文件**
#### **6. 浏览器配置代理（以下是Windows10的代理配置方式，其他操作系统自行配置）**

* 控制面板 -> 网络和Internet -> 代理 -> 手动设置代理

![enter description here](http://md.ws1031.cn/xsj/2018_8_3_2018-08-03_112426.jpg)

#### **7. 设置代理后在本地使用浏览器访问 `http://zx_proxy.ws65535.top/`，可以正常访问**

![enter description here](http://md.ws1031.cn/xsj/2018_8_3_2018-08-03_112735.jpg)
