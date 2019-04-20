上一篇文章 [为七牛云存储空间绑定自定义域名，并使用七牛云提供的免费SSL证书，将自定义加名升级为HTTPS](https://segmentfault.com/a/1190000015919603) 我们提到利用七牛的免费SSL证书，将自定义加名升级为HTTPS的方法。

不知道有没有小伙伴会像我一样担心一年七牛的SSL证书不免费了怎么办？每个域名每年都要几千块的支出对于个人和小企业来说还是一笔不小的数目。

如果绑定七牛云空间的域名能使用 [lets‘encrypt](https://letsencrypt.org/) 等这类免费的网址那么就完美了。
然而七牛目前并不支持 lets'encrypt 这类短期的免费证书。

下面我教大家一种利用 Nginx + lets'encrypt 实现以https的方式访问七牛资源的方法。

## 一、准备工作

1. 首先声明，使用这种方法相当于主动放弃了七牛云存储的CDN优势，只适合访问量不高的个人和小公司。
2. 要有一个域名。
3. 七牛云空间应该已经绑定了自定义的域名，不懂如何绑定的请查看前一篇文章。笔者绑定的域名是 md.ws65535.top。
4. 有一台带公网IP的Linux服务器。笔者服务器IP为 54.191.48.61，Linux环境为 ubuntu14.04。其他发行版原理相同，只不过软件安装方式和目录结构略有不同。

## 二、安装 Nginx
### 1. 安装nginx
```
ubuntu@ip-172-31-27-111:~$ sudo apt-get install nginx
```
### 2. 查看nginx版本
```
ubuntu@ip-172-31-27-111:~$ nginx -v
nginx version: nginx/1.4.6 (Ubuntu)
```
### 3. 启动nginx
```
ubuntu@ip-172-31-27-111:~$ sudo service nginx start

ubuntu@ip-172-31-27-111:~$ ss -tln
State      Recv-Q Send-Q               Local Address:Port                 Peer Address:Port
LISTEN     0      128                              *:80                    *:*
LISTEN     0      128                              *:22                    *:*
LISTEN     0      128                             :::80                    :::*
LISTEN     0      128                             :::22                    :::*
```

### 4. 查看nginx是否安装成功
```
ubuntu@ip-172-31-27-111:~$ curl http://54.191.48.61
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 三、配置Nginx反向代理，将所有访问 `qiniu-ssl.ws65535.top` 的请求全部转发到 `md.ws65535.top`

### 1. **sudo vim /etc/nginx/sites-enabled/qiniu-ssl**
```
server {
    server_name qiniu-ssl.ws65535.top;

    location / {
        proxy_pass http://md.ws65535.top;
    }
}
```
编辑完成后使用 `nginx -s reload` 重新载入Nginx配置文件。

### 2. 登录域名服务商（这里以阿里云为例）的控制台，添加域名解析。
记录类型为 A，主机记录为 qiniu-ssl.ws65535.top，服务器IP为 54.191.48.61
![](http://md.ws65535.top/xsj/2018_8_6_2018-08-06_195839.jpg)

### 3. 此时可以使用 qiniu-ssl.ws65535.top 替换 md.ws65535.top 来访问七牛空间资源
例如
`http://qiniu-ssl.ws65535.top/xsj/2018_8_6_2018-08-06_181854.jpg`
可以访问到下面的资源
`http://md.ws65535.top/xsj/2018_8_6_2018-08-06_181854.jpg`

## 四、安装 HTTPS 证书 【[参考](https://certbot.eff.org/lets-encrypt/ubuntutrusty-nginx)】

> 此处只记录ubuntu14.04安装方法

### 1. 安装 Certbot
```
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install python-certbot-nginx 
```
### 2. 安装HTTPS证书
```
$ sudo certbot --nginx
```

#### 实例
```
ubuntu@ip-172-31-27-111:~$ sudo certbot --nginx
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
-------------------------------------------------------------------------------
1: agency.ws65535.xyz
2: qiniu-ssl.ws65535.top
-------------------------------------------------------------------------------
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 2 #此处选择将 qiniu-ssl.ws65535.top 设为https
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for qiniu-ssl.ws65535.top
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/qiniu-ssl

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2 #是否强制将http方式访问的请求跳转到以HTTPS方式访问
Redirecting all traffic on port 80 to ssl in /etc/nginx/sites-enabled/qiniu-ssl

-------------------------------------------------------------------------------
Congratulations! You have successfully enabled https://qiniu-ssl.ws65535.top

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=qiniu-ssl.ws65535.top
-------------------------------------------------------------------------------

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/qiniu-ssl.ws65535.top/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/qiniu-ssl.ws65535.top/privkey.pem
   Your cert will expire on 2018-11-04. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le

```

### 3. 此时再查看 配置文件 /etc/nginx/sites-enabled/qiniu-ssl，已经被 certbot 做了修改
```
ubuntu@ip-172-31-27-111:~$ cat /etc/nginx/sites-enabled/qiniu-ssl
server {
    server_name qiniu-ssl.ws65535.top;

    location / {
        proxy_pass http://md.ws65535.top;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/qiniu-ssl.ws65535.top/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/qiniu-ssl.ws65535.top/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = qiniu-ssl.ws65535.top) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    server_name qiniu-ssl.ws65535.top;
    listen 80;
    return 404; # managed by Certbot
}
```

### 4. 此时再使用 `http://qiniu-ssl.ws65535.top/xsj/2018_8_6_2018-08-06_181854.jpg` 访问七牛云空间的资源，会被强制跳转到 `https://qiniu-ssl.ws65535.top/xsj/2018_8_6_2018-08-06_181854.jpg`