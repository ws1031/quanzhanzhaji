## 一、什么是跨域

跨域是指从一个域名的网页去请求另一个域名的资源。比如从 www.a.com 页面去请求 www.b.com 的资源。

![跨域访问](http://md.ws1031.cn/xsj/2018_7_17_2018-07-11_194120.jpg)

浏览器一般默认会禁止跨域访问。因为不安全，容易出现 CSRF（跨站请求伪造）攻击。

## 二、Nginx控制浏览器允许跨域访问

Nginx通过添加 Access-Control-Allow-Origin、Access-Control-Allow-Methods、Access-Control-Allow-Headers 等HTTP头信息的方式控制浏览器缓存。

> "Access-Control-Allow-Origin" 设置允许发起跨域请求的网站
> "Access-Control-Allow-Methods" 设置允许发起跨域请求请求的HTTP方法
> "Access-Control-Allow-Headers" 设置允许跨域请求包含 Content-Type头

### **[ngx_http_headers_module](http://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header)**

### **语法**

	Syntax:	add_header name value [always];
	Default:	—
	Context:	http, server, location, if in location

### 应用实例

#### **1. vim conf.d/cross_site.conf**

```nginxconf
# 配置网站www.a.com
server {
    server_name www.a.com;
    root /vagrant/a;
	
	# 允许 http://www.b.com 使用 GET,POST,DELETE HTTP方法发起跨域请求
    add_header Access-Control-Allow-Origin http://www.b.com;
    add_header Access-Control-Allow-Method GET,POST,DELETE;
}

# 配置网站www.b.com
server {
    server_name www.b.com;
    root /vagrant/b;
}

# 配置网站www.c.com
server {
    server_name www.c.com;
    root /vagrant/c;
}

```

#### **2. nginx -s reload 重新载入nginx配置文件**

#### **3. 创建 `/vagrant/a/a.txt`、`/vagrant/b/index.html`、`/vagrant/c/index.html` 文件**

* vim /vagrant/a/a.txt
```
Hello,I'm a!
```

* /vagrant/b/index.html
```html
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="utf-8">
		<title>Ajax跨站访问b</title>
	</head>
	<body>
		<h1>Ajax跨站访问b - </h1>
	</body>
	<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
	<script>
	$(function(){
		$.ajax({
			url: "http://www.a.com/a.txt",
			type: "GET",
			success: function (data) {
				$('h1').append(data);
			},
			error: function (data) {
				$('h1').append('请求失败！');
			}
		});
	})
	</script>
</html>
```

* /vagrant/c/index.html
```html
<!DOCTYPE html>
<html lang="en">
	<head>
		<meta charset="utf-8">
		<title>Ajax跨站访问c</title>
	</head>
	<body>
		<h1>Ajax跨站访问c - </h1>
	</body>
	<script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
	<script>
	$(function(){
		$.ajax({
			url: "http://www.a.com/a.txt",
			type: "GET",
			success: function (data) {
				$('h1').append(data);
			},
			error: function (data) {
				$('h1').append('请求失败！');
			}
		});
	})
	</script>
</html>
```

#### **4. 配置客户端的hosts文件（使用真是域名的可以忽略）**

windows: `C:\Windows\System32\drivers\etc\hosts`
linux: `/etc/hosts`

添加如下内容，并保存（192.168.33.88为笔者虚拟机的IP，需自行替换为自己的IP）：
```
192.168.33.88 www.a.com
192.168.33.88 www.b.com
192.168.33.88 www.c.com
```

#### **5. 浏览器分别访问 `http://www.b.com/index.html` 和 `http://www.c.com/index.html`**

* http://www.b.com/index.html
```
Ajax跨站访问b - Hello,I'm a!
```

* http://www.c.com/index.html
```
Ajax跨站访问c - 请求失败！
```
打开浏览器的开发者模式Console，还可以发现 http://www.c.com/index.html 的页面出现报错:
```
Failed to load http://www.a.com/a.txt: The 'Access-Control-Allow-Origin' header has a value 'http://www.b.com' that is not equal to the supplied origin. Origin 'http://www.c.com' is therefore not allowed access.
```
