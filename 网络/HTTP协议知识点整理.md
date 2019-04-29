HTTP协议知识点整理
---

## 一、HTTP协议的工作特点和工作原理

#### **工作特点**

* 基于B/S模式
* 通信开销小、简单快速、传输成本低
* 使用灵活、可试用超文本传输协议
* 节省传输时间
* 无状态

#### **工作原理**

客户端发送请求给服务器，创建一个TCP链接，指定端口号，默认80，连接到服务器，服务器监听浏览器请求，一旦监听到客户端请求，分析请求类型后，服务器会向客户端放回状态信息和数据内容。

### 二、HTTP协议请求方法

### 1. GET：获取资源

GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容。

### 2. POST：传输实体主体

POST 方法用来传输实体的主体。  
虽然用 GET 方法也可以传输实体的主体，但一般不用 GET 方法进行传输，而是用 POST 方法。虽说 POST 的功能与 GET 很相似，但POST 的主要目的并不是获取响应的主体内容。

### 3. PUT：传输文件

PUT 方法用来传输文件。就像 FTP 协议的文件上传一样，要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。  
但是，鉴于 HTTP/1.1 的 PUT 方法自身不带验证机制，任何人都可以上传文件 , 存在安全性问题，因此一般的 Web 网站不使用该方法。若配合 Web 应用程序的验证机制，或架构设计采用 REST（REpresentationalState Transfer，表征状态转移）标准的同类 Web 网站，就可能会开放使用 PUT 方法。

### 4. HEAD：获得报文首部

HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认URI 的有效性及资源更新的日期时间等。

### 5. DELETE：删除文件

DELETE 方法用来删除文件，是与 PUT 相反的方法。DELETE 方法按请求 URI 删除指定的资源。  
但是，HTTP/1.1 的 DELETE 方法本身和 PUT 方法一样不带验证机制，所以一般的 Web 网站也不使用 DELETE 方法。当配合 Web 应用程序的验证机制，或遵守 REST 标准时还是有可能会开放使用的。

### 6. OPTIONS：询问支持的方法

OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。

### 7. TRACE：追踪路径

TRACE 方法是让 Web 服务器端将之前的请求通信环回给客户端的方法。  
发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器端就将该数字减 1，当数值刚好减到 0 时，就停止继续传输，最后接收到请求的服务器端则返回状态码 200 OK 的响应。  
客户端通过 TRACE 方法可以查询发送出去的请求是怎样被加工修改 / 篡改的。这是因为，请求想要连接到源目标服务器可能会通过代理中转，TRACE 方法就是用来确认连接过程中发生的一系列操作。
但是，TRACE 方法本来就不怎么常用，再加上它容易引发 XST（Cross-Site Tracing，跨站追踪）攻击，通常就更不会用到了。

## 三、HTTP常用状态码

状态码的职责是当客户端向服务器端发送请求时，描述返回的请求结果。  
借助状态码，用户可以知道服务器端是正常处理了请求，还是出现了错误。

### 1. 1XX Informational（信息性状态码， 接收的请求正在处理）

### 2. 2XX Success（成功状态码，表示请求正常处理完毕）

#### **200 OK**

表示从客户端发来的请求在服务器端被正常处理了。

#### **204 No Content**

该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。另外，也不允许返回任何实体的主体。比如，当从浏览器发出请求处理后，返回 204 响应，那么浏览器显示的页面不发生更新。    
一般在只需要从客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用。

#### **206 Partial Content**

该状态码表示客户端进行了范围请求，而服务器成功执行了这部分的 GET 请求。响应报文中包含由 Content-Range 指定范围的实体内容。

### 3. 3XX Redirection（重定向状态码，需要进行附加操作以完成请求）

#### **301 Moved Permanently**

永久性重定向。该状态码表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI。也就是说，如果已经把资源对应的 URI保存为书签了，这时应该按 Location 首部字段提示的 URI 重新保存。

#### **302 Move temporarily**

临时性重定向。该状态码表示请求的资源已被分配了新的 URI，希望用户（本次）能使用新的 URI 访问。  
和 301 Moved Permanently 状态码相似，但 302 状态码代表的资源不是被永久移动，只是临时性质的。换句话说，已移动的资源对应的URI 将来还有可能发生改变。比如，用户把 URI 保存成书签，但不会像301 状态码出现时那样去更新书签，而是仍旧保留返回 302 状态码的页面对应的 URI。

#### **304 Not Modified**

该状态码表示客户端发送附带条件的请求时，服务器端允许请求访问资源，而文档的内容（自上次访问以来或者根据请求的条件）并没有改变。


### 4. 4XX Client Error（客户端错误状态码，服务器无法处理请求）

#### **400 Bad Request**

该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。

#### **401 Unauthorized**
该状态码表示发送的请求需要有通过 HTTP 认证（BASIC 认证、DIGEST 认证）的认证信息。另外若之前已进行过 1 次请求，则表示用户认证失败。  
返回含有 401 的响应必须包含一个适用于被请求资源的 WWW-Authenticate 首部用以质询（challenge）用户信息。当浏览器初次接收到 401 响应，会弹出认证用的对话窗口。

#### **403 Forbidden**

该状态码表明对请求资源的访问被服务器拒绝了。  
未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发送源 IP 地址试图访问）等情况都可能是发生 403 的原因。

#### **404 Not Found**
该状态码表明服务器上无法找到请求的资源。
 
#### **405 Method Not Allowed** 

表明当前请求使用的 HTTP 方法不被服务器允许。  
例如使用GET方法请求需要POST方法的数据。

### 5. 5XX Server Error（服务器错误状态码，服务器处理请求出错）

#### **500 Internal Server Error**

该状态码表明服务器端在执行请求时发生了错误。有可能是 Web 应用存在的 bug 或某些临时的故障。

#### **502 Bad Gateway**

该状态码表明服务器和网关/代理通信出错。
 
#### **503 Service Unavailable**

该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

#### **504 Gateway Timeout 网关超时**

该状态码表明服务器作为网关或代理，但是没有及时从上游服务器收到请求。


## 四、HTTP协议常见请求、响应头

### 1. HTTP/1.1 通用首部字段

通用首部字段是指，请求报文和响应报文双方都会使用的首部。

#### **Cache-Control 设置缓存的工作机制**

指令的参数是可选的，多个指令之间通过 `,` 分隔。

> Cache-Control: private, max-age=0, no-cache

* 缓存请求指令

指令 |参数| 说明
-|-|-
no-cache| 无 |强制向源服务器再次验证
no-store| 无| 不缓存请求或响应的任何内容
max-age = [ 秒]| 必需 |响应的最大Age值
max-stale( = [ 秒]) |可省略 |接收已过期的响应
min-fresh = [ 秒] |必需 |期望在指定时间内的响应仍有效
no-transform |无 |代理不可更改媒体类型
only-if-cached |无| 从缓存获取资源
cache-extension |- |新指令标记（token）

* 缓存响应指令

指令 |参数 |说明
-|-|-
public |无 |可向任意方提供响应的缓存
private |可省略 |仅向特定用户返回响应
no-cache |可省略 |缓存前必须先确认其有效性
no-store |无 |不缓存请求或响应的任何内容
no-transform |无 |代理不可更改媒体类型
must-revalidate |无 |可缓存但必须再向源服务器进行确认
proxy-revalidate |无 |要求中间缓存服务器对缓存的响应有效性再进行确认
max-age = [ 秒] |必需| 响应的最大Age值
s-maxage = [ 秒] |必需| 公共缓存服务器响应的最大Age值
cache-extension |- |新指令标记（token）


#### **Connection**

Connection 首部字段具备如下两个作用。

* 控制不再转发给代理的首部字段
* 管理持久连接

	Connection: keep-alive
	Connection: Upgrade

#### **Date 创建 HTTP 报文的日期和时间。**

	Date: Tue, 03 Jul 2012 04:40:59 GMT
	Date: Tue, 03-Jul-12 04:40:59 GMT
	Date: Tue Jul 03 04:40:59 2012

##### **Content-Type 设置请求体或相应体的MIME类型**

	Content-Type: application/x-www-form-urlencoded
	Content-Type: text/html; charset=utf-8
	

#### **Content-Encoding 设置数据使用的编码类型**

	Content-Encoding: gzip

#### **Content-Length 请求体或响应体的字节长度**

	Content-Length: 348
	
### 2. 常见请求头字段

#### **Host 设置服务器域名和TCP端口号，如果使用的是服务请求标准端口号，端口号可以省略**

	Host: en.wikipedia.org:8080
	Host: en.wikipedia.org
	
#### **Accept 设置接受的内容类型**
	
	Accept: text/plain
	
#### **Accept-Charset 设置接受的字符编码**

	Accept-Charset: utf-8
	

#### **Accept-Encoding 设置接受的编码格式**

	Accept-Encoding: gzip, deflate


#### **Origin 标识跨域资源请求（请求服务端设置Access-Control-Allow-Origin响应字段）**

	Origin: http://www.example-social-network.com

#### **User-Agent 用户代理的字符串值**

	User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/21.0

#### **Cookie 设置服务器使用Set-Cookie发送的http cookie**

	Cookie: $Version=1; Skin=new;

#### **Authorization 设置HTTP身份验证的凭证**

	Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==

#### **Referer 告知服务器请求的原始资源的 URI**

	Referer: http://en.wikipedia.org/wiki/Main_Page

#### **X-Forwarded-For 客户端通过HTTP代理或者负载均衡器连接的web服务器的原始IP地址**

	X-Forwarded-For: client1, proxy1, proxy2
	X-Forwarded-For: 129.78.138.66, 129.78.64.103

#### **Forwarded 客户端通过http代理连接web服务的源信息**

	Forwarded: for=192.0.2.60;proto=http;by=203.0.113.43
	Forwarded: for=192.0.2.43, for=198.51.100.17

### 3. 常见响应头字段

#### **Allow 通知客户端请求所支持的 HTTP 方法**

	Allow: GET, HEAD

#### **Access-Control-Allow-Origin 指定哪些站点可以参与跨站资源共享**

	Access-Control-Allow-Origin: *

#### **Expires 设置响应体的过期时间**

	Expires: Thu, 01 Dec 1994 16:00:00 GMT

#### **Last-Modified 设置请求对象最后一次的修改日期**

	Last-Modified: Tue, 15 Nov 1994 12:45:26 GMT

#### **Age 对象在代理缓存中暂存的秒数**

	Age: 3600

#### **ETag 特定版本资源的标识符，通常是消息摘要**

	ETag: "737060cd8c284d8af7ad3082f209582d"

#### **Refresh 重定向**

	Refresh: 5; url=http://www.w3.org/pub/WWW/People.html

#### **Set-Cookie 设置HTTP Cookie**

	Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1
