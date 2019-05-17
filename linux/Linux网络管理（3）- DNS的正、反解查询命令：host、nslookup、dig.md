Linux 网络管理 - DNS的正、反解查询命令：host、nslookup、dig
---

## 一、host

> 解析域名对应的IP地址和别名等信息

### 1. 语法

    host [选项] [主机名或IP] [server]

### 2. 常用选项

> `-a`：列出该主机详细的各项主机名称设定资料

### 3. 常用参数
    
> `server`：host命令默认是使用 `/etc/resolv.conf` 文件中的 DNS 主机来查询的，若设置该参数，则使用这里设置的 DNS 主机进行查询。

### 4. 应用

#### **解析域名对应的IP地址等信息**

* `host 域名`

```
[root@10 vagrant]# host www.baidu.com
www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 61.135.169.125
www.a.shifen.com has address 61.135.169.121
```

* `host -a 域名`
```
[root@10 vagrant]# host -a www.baidu.com
Trying "www.baidu.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29562
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 5, ADDITIONAL: 5

;; QUESTION SECTION:
;www.baidu.com.                 IN      ANY

;; ANSWER SECTION:
www.baidu.com.          1000    IN      CNAME   www.a.shifen.com.

;; AUTHORITY SECTION:
baidu.com.              52656   IN      NS      ns7.baidu.com.
baidu.com.              52656   IN      NS      ns3.baidu.com.
baidu.com.              52656   IN      NS      ns2.baidu.com.
baidu.com.              52656   IN      NS      ns4.baidu.com.
baidu.com.              52656   IN      NS      dns.baidu.com.

;; ADDITIONAL SECTION:
dns.baidu.com.          52853   IN      A       202.108.22.220
ns2.baidu.com.          65473   IN      A       61.135.165.235
ns3.baidu.com.          52760   IN      A       220.181.37.10
ns4.baidu.com.          65473   IN      A       220.181.38.10
ns7.baidu.com.          53740   IN      A       180.76.76.92

Received 228 bytes from 10.0.2.3#53 in 9 ms
```

> 题外话：从上面可以看出 `www.baidu.com` 通过 CNAME 映射到 `www.a.shifen.com`，但是为什么我们无法直接访问 `www.a.shifen.com` 呢？

Web应用防火墙或高防IP生产的CNAME域名，是用于DNS解析的，不能直接访问。

![此处输入图片的描述][1]

#### **使用自定义的 DNS主机 解析域名对应的IP地址等信息**

* `host 域名 DNS主机名或IP`

```
[root@10 vagrant]# host www.baidu.com 168.95.1.1
Using domain server:
Name: 168.95.1.1
Address: 168.95.1.1#53
Aliases:

www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 180.97.33.108
www.a.shifen.com has address 180.97.33.107

[root@10 vagrant]# host www.baidu.com dns.hinet.net
Using domain server:
Name: dns.hinet.net
Address: 168.95.1.1#53
Aliases:

www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 180.97.33.108
www.a.shifen.com has address 180.97.33.107

[root@10 vagrant]# host www.baidu.com 8.8.8.8
Using domain server:
Name: 8.8.8.8
Address: 8.8.8.8#53
Aliases:

www.baidu.com is an alias for www.a.shifen.com.
www.a.shifen.com has address 61.135.169.121
www.a.shifen.com has address 61.135.169.125
```

## 二、nslookup

> 域名解析工具，就是查DNS信息用的命令。使用 /etc/resolv.conf 这个文件作为 DNS 服务器的来源选择。

### 1. 语法

    nslookup [主机名或IP]

### 2. 应用

#### **解析域名对应的IP地址**

* `nslookup 域名`

```
[root@10 vagrant]# nslookup www.baidu.com
Server:         10.0.2.3
Address:        10.0.2.3#53

Non-authoritative answer:
Name:   www.baidu.com
Address: 61.135.169.121
Name:   www.baidu.com
Address: 61.135.169.125
```

#### **解析IP地址对应的主机名**

> 并不是所有的IP地址都能解析成功

* `nslookup IP`

```
[root@10 vagrant]# nslookup 168.95.1.1
Server:         10.0.2.3
Address:        10.0.2.3#53

Non-authoritative answer:
1.1.95.168.in-addr.arpa name = dns.hinet.net.

Authoritative answers can be found from:
95.168.in-addr.arpa     nameserver = ans1.hinet.net.
95.168.in-addr.arpa     nameserver = ans2.hinet.net.
ans1.hinet.net  internet address = 168.95.192.15
ans1.hinet.net  has AAAA address 2001:b000:168::1:100:1
ans2.hinet.net  internet address = 168.95.1.15
ans2.hinet.net  has AAAA address 2001:b000:168::2:100:1
```

#### **查看本机DNS服务器**

* `nslookup server`
```
[root@10 vagrant]# nslookup server
Server:         10.0.2.3
Address:        10.0.2.3#53

** server can't find server: NXDOMAIN
```

## 三、dig

> 域名查询工具，可以用来测试域名系统工作是否正常。
    
> 功能与 `nslookup` 类似，建议使用 `dig` 来取代 `nslookup`

### 1. 安装

若系统默认没有 `dig` 命令，则使用下面命令进行安装。

    yum install bind-utils

### 2. 语法

    dig [选项] [主机名]

### 3. 常用选项

> `@<DNS服务器IP>`：dig命令默认使用 `/etc/resolv.conf` 文件中的 DNS 主机来解析域名，若设置该参数，则使用这里设置的 DNS 主机进行解析。
> `-b <IP地址>`：当主机具有多个IP地址，指定使用本机的哪个IP地址向域名服务器发送域名查询请求。

### 4. 应用

#### **解析域名对应的IP地址等信息**
```
[root@10 tmp]# dig www.baidu.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 50280
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 5, ADDITIONAL: 6

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.baidu.com.                 IN      A

;; ANSWER SECTION:
www.baidu.com.          1096    IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       290     IN      A       61.135.169.121
www.a.shifen.com.       290     IN      A       61.135.169.125

;; AUTHORITY SECTION:
a.shifen.com.           34      IN      NS      ns3.a.shifen.com.
a.shifen.com.           34      IN      NS      ns4.a.shifen.com.
a.shifen.com.           34      IN      NS      ns1.a.shifen.com.
a.shifen.com.           34      IN      NS      ns5.a.shifen.com.
a.shifen.com.           34      IN      NS      ns2.a.shifen.com.

;; ADDITIONAL SECTION:
ns1.a.shifen.com.       411     IN      A       61.135.165.224
ns2.a.shifen.com.       435     IN      A       180.149.133.241
ns3.a.shifen.com.       431     IN      A       61.135.162.215
ns4.a.shifen.com.       431     IN      A       115.239.210.176
ns5.a.shifen.com.       435     IN      A       119.75.222.17

;; Query time: 11 msec
;; SERVER: 10.0.2.3#53(10.0.2.3)
;; WHEN: Wed May 16 08:40:42 UTC 2018
;; MSG SIZE  rcvd: 271
```

* 在这个范例当中，我们可以看到输出的信息包括以下几个部分：

> HEADER(标题)：显示查询的内容有哪些，包括1个 QUERY, 3个 ANSWER 及5个AUTHORITY。
> QUESTION(问题)：显示所要查询的内容。
> ANSWER(回答)：依据刚刚的 QUESTION 去查询所得到的结果。
> AUTHORITY(验证)：从这里我们可以知道 www.baidu.com 是由 哪些DNS服务器提供的ANSWER。

#### **使用自定义的 DNS服务器 解析域名对应的IP地址等信息**
```
[root@10 tmp]# dig @168.95.1.1 www.baidu.com

; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> @168.95.1.1 www.baidu.com
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 48040
;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 3072
;; QUESTION SECTION:
;www.baidu.com.                 IN      A

;; ANSWER SECTION:
www.baidu.com.          1034    IN      CNAME   www.a.shifen.com.
www.a.shifen.com.       241     IN      A       180.97.33.107
www.a.shifen.com.       241     IN      A       180.97.33.108

;; Query time: 70 msec
;; SERVER: 168.95.1.1#53(168.95.1.1)
;; WHEN: Wed May 16 08:39:13 UTC 2018
;; MSG SIZE  rcvd: 101
```


  [1]: http://docs-aliyun.cn-hangzhou.oss.aliyun-inc.com/assets/pic/42195/cn_zh/1497335927583/TB1MvlIKFXXXXX5XFXXXXXXXXXX%20%281%29.png