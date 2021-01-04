服务端面试宝典-HTTP协议知识点整理
---

### 一、HTTP协议简介

#### 1. 含义

HTTP 协议是 **Hyper Text Transfer Protocol（超文本传输协议）** 的缩写，是用于浏览器与服务器之间传输文字、图片、音频、视频等超文本数据的约定和规范。
当前主流的HTTP协议版本为 **HTTP/1.1**。

#### 2. 特点


* 无状态，每个请求都是互相独立、毫无关联的
* 应用层协议
* 支持B/S及C/S模式
* 灵活可扩展，可以任意添加首部字段实现任意功能
* 可靠性强，基于 TCP/IP 协议“尽量”保证数据的送达
* 通信开销小、简单快速、传输成本低，节省传输时间

#### 3. 工作原理

HTTP协议采用了请求/响应模型。客户端向服务器发送一个请求报文，请求报文包含请求的方法、URL、协议版本、请求首部和请求数据。服务器以一个状态行作为响应，响应的内容包括协议的版本、成功或者错误代码、服务器信息、响应首部和响应数据。

![HTTP协议工作原理](https://md.s1031.cn/xsj/2021_1_4_Http_Request_&_Response.png)

##### 客户端与服务器建立TCP连接

客户端发送请求给服务器，创建一个TCP链接，指定服务器通信端口号，默认80。

##### 发送HTTP请求报文

客户端通过TCP连接向服务器发送HTTP请求报文。请求报文由请求行、请求首部、空行和请求数据4部分组成。

##### 服务器接收请求报文并返回响应报文

服务器持续监听80端口，一旦监听到请求数据，马上对请求报文进行解析，定位请求资源。
服务器将请求资源组成响应报文，发回给客户端。响应报文由状态行、响应首部、空行和响应数据4部分组成。

##### 断开TCP连接

若 Connection 首部为 close，则服务器主动关闭 TCP 连接，客户端被动关闭连接，释放TCP连接。
若 Connection 首部为 keep-alive，则该连接会保持一段时间，在该时间内可以继续接收请求。

#### 客户端接收并解析响应报文

客户端首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应首部字段，获取响应数据的长度、字符集、编码等信息。
浏览器读取响应数据 HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示。


---


### 二、HTTP协议请求方法

#### 1. GET：获取资源

GET 方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容。

#### 2. POST：传输实体主体

POST 方法用来传输实体的主体。  
虽然用 GET 方法也可以传输实体的主体，但一般不用 GET 方法进行传输，而是用 POST 方法。虽说 POST 的功能与 GET 很相似，但POST 的主要目的并不是获取响应的主体内容。

#### 3. PUT：传输文件

PUT 方法用来传输文件。就像 FTP 协议的文件上传一样，要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。  
但是，鉴于 HTTP/1.1 的 PUT 方法自身不带验证机制，任何人都可以上传文件 , 存在安全性问题，因此一般的 Web 网站不使用该方法。若配合 Web 应用程序的验证机制，或架构设计采用 REST（REpresentationalState Transfer，表征状态转移）标准的同类 Web 网站，就可能会开放使用 PUT 方法。

#### 4. HEAD：获得报文首部

HEAD 方法和 GET 方法一样，只是不返回报文主体部分。用于确认URI 的有效性及资源更新的日期时间等。

#### 5. DELETE：删除文件

DELETE 方法用来删除文件，是与 PUT 相反的方法。DELETE 方法按请求 URI 删除指定的资源。  
但是，HTTP/1.1 的 DELETE 方法本身和 PUT 方法一样不带验证机制，所以一般的 Web 网站也不使用 DELETE 方法。当配合 Web 应用程序的验证机制，或遵守 REST 标准时还是有可能会开放使用的。

#### 6. OPTIONS：询问支持的方法

OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法。

#### 7. TRACE：追踪路径

TRACE 方法是让 Web 服务器端将之前的请求通信环回给客户端的方法。  
发送请求时，在 Max-Forwards 首部字段中填入数值，每经过一个服务器端就将该数字减 1，当数值刚好减到 0 时，就停止继续传输，最后接收到请求的服务器端则返回状态码 200 OK 的响应。  
客户端通过 TRACE 方法可以查询发送出去的请求是怎样被加工修改 / 篡改的。这是因为，请求想要连接到源目标服务器可能会通过代理中转，TRACE 方法就是用来确认连接过程中发生的一系列操作。
但是，TRACE 方法本来就不怎么常用，再加上它容易引发 XST（Cross-Site Tracing，跨站追踪）攻击，通常就更不会用到了。

---

### 三、HTTP常用状态码

状态码的职责是当客户端向服务器端发送请求时，描述返回的请求结果。  
借助状态码，用户可以知道服务器端是正常处理了请求，还是出现了错误。

#### 1. 1XX Informational（信息性状态码， 接收的请求正在处理）

#### 2. 2XX Success（成功状态码，表示请求正常处理完毕）

##### 200 OK

表示从客户端发来的请求在服务器端被正常处理了。

##### 204 No Content

该状态码代表服务器接收的请求已成功处理，但在返回的响应报文中不含实体的主体部分。另外，也不允许返回任何实体的主体。比如，当从浏览器发出请求处理后，返回 204 响应，那么浏览器显示的页面不发生更新。    
一般在只需要从客户端往服务器发送信息，而对客户端不需要发送新信息内容的情况下使用。

##### 206 Partial Content

该状态码表示客户端进行了范围请求，而服务器成功执行了这部分的 GET 请求。响应报文中包含由 Content-Range 指定范围的实体内容。

#### 3. 3XX Redirection（重定向状态码，需要进行附加操作以完成请求）

##### 301 Moved Permanently

永久性重定向。该状态码表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI。也就是说，如果已经把资源对应的 URI保存为书签了，这时应该按 Location 首部字段提示的 URI 重新保存。

##### 302 Move temporarily

临时性重定向。该状态码表示请求的资源已被分配了新的 URI，希望用户（本次）能使用新的 URI 访问。  
和 301 Moved Permanently 状态码相似，但 302 状态码代表的资源不是被永久移动，只是临时性质的。换句话说，已移动的资源对应的URI 将来还有可能发生改变。比如，用户把 URI 保存成书签，但不会像301 状态码出现时那样去更新书签，而是仍旧保留返回 302 状态码的页面对应的 URI。

##### 304 Not Modified

该状态码表示客户端发送附带条件的请求时，服务器端允许请求访问资源，而文档的内容（自上次访问以来或者根据请求的条件）并没有改变。


#### 4. 4XX Client Error（客户端错误状态码，服务器无法处理请求）

##### 400 Bad Request

该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。

##### 401 Unauthorized

该状态码表示发送的请求需要有通过 HTTP 认证（BASIC 认证、DIGEST 认证）的认证信息。另外若之前已进行过 1 次请求，则表示用户认证失败。  
返回含有 401 的响应必须包含一个适用于被请求资源的 WWW-Authenticate 首部用以质询（challenge）用户信息。当浏览器初次接收到 401 响应，会弹出认证用的对话窗口。

##### 403 Forbidden

该状态码表明对请求资源的访问被服务器拒绝了。  
未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发送源 IP 地址试图访问）等情况都可能是发生 403 的原因。

##### 404 Not Found
该状态码表明服务器上无法找到请求的资源。
 
##### 405 Method Not Allowed 

表明当前请求使用的 HTTP 方法不被服务器允许。  
例如使用GET方法请求需要POST方法的数据。

#### 5. 5XX Server Error（服务器错误状态码，服务器处理请求出错）

##### 500 Internal Server Error

该状态码表明服务器端在执行请求时发生了错误。有可能是 Web 应用存在的 bug 或某些临时的故障。

##### 502 Bad Gateway

该状态码表明服务器和网关/代理通信出错。
 
##### 503 Service Unavailable

该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

##### 504 Gateway Timeout 网关超时

该状态码表明服务器作为网关或代理，但是没有及时从上游服务器收到请求。

---

### 四、HTTP协议常见首部字段

#### 0. End-to-end 首部和 Hop-by-hop 首部

HTTP 首部字段将定义成缓存代理和非缓存代理的行为，分成 2 种类型。

##### 逐跳首部（Hop-by-hop Header）

逐跳首部 只对与当前浏览器产生直接连接的服务器（比如Nginx反向代理）有效。通过缓存或代理服务器之后，逐跳首部 将不会被转发给 upstream 服务器。
比如 Keep-Alive 首部，仅仅表示浏览器尝试和直连Nginx之间保持连接，而不管Nginx和后续服务器之间的连接。代理服务器要处理这些首部，并按照自己的需要来修改这些首部。

**HTTP/1.1** 版本之后，如果要使用逐跳首部，必须需提供 **Connection** 首部字段。

下面列举了 **HTTP/1.1** 中的逐跳首部字段。

	Connection
	Keep-Alive
	Proxy-Authenticate
	Proxy-Authorization
	Trailer
	TE
	Transfer-Encoding
	Upgrade

上面 8 个首部字段作用在第四节有做介绍。
除这 8 个首部字段之外，其他所有字段都属于端到端首部（End-to-end Header）。

##### 端到端首部（End-to-end Header）

端到端首部用来传递这个浏览器和最终处理请求的服务器之间的信息。
比如 **Accept** 首部，表示客户端想从后端服务器得到的数据类型，和中间的代理服务器无关。代理服务器不能修改这些首部。



#### 1. 通用首部字段

通用首部字段是指，请求报文和响应报文双方都会使用的首部字段。

##### Cache-Control

控制缓存行为的工作机制。
指令的参数是可选的，多个指令之间通过 `,` 分隔。

> Cache-Control: max-age=0, no-cache
> Cache-Control: private, must-revalidate

**缓存请求指令**

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

**缓存响应指令**

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


##### Connection

Connection 可以承载 3 种不同类型的标签。

* 管理持久连接（keep-alive/close）
* 设置不应被代理服务器转发的首部字段（逐跳首部）
* 任意标签值，用于描述此连接的非标准选项

HTTP/1.1 版本的默认连接都是持久连接(keep-alive)。因此，客户端会在持久连接上连续发送请求。当服务器端想明确断开连接时，则指定 Connection 首部字段的值为 close。

	Connection: clone

HTTP/1.1 之前的版本的默认连接都是非持久连接。因此，如果想在旧版本的 HTTP 协议上维持长连接，则需要指定 Connection 首部字段的值为 keep-alive。

	Connection: keep-alive

Connection还可以用来设置逐跳首部，比如 Connection: Upgrade，就表示在这个连接中，Upgrade 是一个逐跳首部，不应当被代理服务器原样传递给 upstream 服务器。

	Connection: Upgrade


##### Date

记录创建 HTTP 报文的日期和时间。
 
	Date: Thu, 31 Dec 2020 14:42:39 GMT

##### Trailer

声明在报文主体后记录了哪些首部字段。该首部字段一般应用在 HTTP/1.1 分块传输编码时。

	HTTP/1.1 200 OK
	...
	Trailer: Expires
	Expires: Tue, 28 Sep 2004 23:59:59 GMT

##### Transfer-Encoding

指定报文主体的传输编码方式。
HTTP/1.1 的传输编码方式仅对分块传输编码有效。

	Transfer-Encoding: chunked

##### Upgrade

用于检测 HTTP 协议及其他协议是否可使用更高的版本进行通信，其参数值可以用来指定一个完全不同的通信协议。

	Upgrade: TLS/1.0
	Connection: Upgrade

首部字段 Upgrade 指定的值为 `TLS/1.0`。
Upgrade 首部字段产生作用的 Upgrade 对象仅限于客户端和邻接服务器之间。因此，使用首部字段 Upgrade 时，还需要额外指定 `Connection: Upgrade`。
对于附有首部字段 Upgrade 的请求，服务器可用 `101 Switching Protocols` 状态码作为响应返回。

##### Via

记录通信中间代理服务器的相关信息。
报文经过代理或网关时，会先在首部字段 Via 中附加该服务器的信息，然后再进行转发。

	Via: 1.0 gw.hackr.jp（Squid/3.1）,1.1 a1.example.com（Squid/2.7）

##### Keep-Alive

设置连接的超时时长和最大请求数。

	Keep-Alive: timeout=10, max=1000


#### 2. 实体首部字段

实体首部字段是指包含在请求报文和响应报文中的实体部分所使用的首部，包含内容的更新时间等与实体相关的信息。

##### Allow

声明服务器针对当前请求所支持的HTTP方法。
当服务器接收到不支持的 HTTP 方法时，会以状态码 `405 Method Not Allowed` 作为响应返回。与此同时，还会把所有能支持的 HTTP 方法写入首部字段 Allow 后返回。

	Allow: GET, HEAD, DELETE

##### Content-Type

声明请求体或响应体的MIME类型。

	Content-Type: application/x-www-form-urlencoded
	Content-Type: text/html; charset=utf-8
	
##### Content-Encoding

设置数据使用的编码类型。

	Content-Encoding: gzip

##### Content-Language

请求主体使用的自然语言（指中文或英文等语言）

	Content-Language: zh-CN

##### Content-Length

请求体或响应体的字节长度。

	Content-Length: 3148

##### Expires

设置响应数据的缓存过期时间。
缓存服务器在接收到含有首部字段 Expires 的响应后，会以缓存来应答请求，在Expires 字段值指定的时间之前，响应的副本会一直被保存。当超过指定的时间后，缓存服务器在请求发送过来时，会转向源服务器请求资源。
源服务器不希望缓存服务器对资源缓存时，最好在 Expires 字段内写入与首部字段 Date 相同的时间值。
但是，当首部字段 Cache-Control 有指定 max-age 指令时，会按照 max-age 设置缓存过期时间。

	Expires: Thu, 01 Dec 2021 16:00:00 GMT

##### Last-Modified

设置响应数据最后一次的修改时间。

	Last-Modified: Tue, 15 Nov 2021 12:45:26 GMT
	Last-Modified: Wed, 21 Oct 2015 07:28:00 GMT


#### 3. 请求首部字段

从客户端向服务器端发送请求报文时使用的首部。包含请求的附加内容、客户端信息、响应内容相关优先级等信息。

##### Host

指明了请求将要发送到的服务器主机名和端口号，如果使用的是服务请求标准端口号，端口号可以省略。

	Host: www.abc.org:8080
	Host: www.abc.org
	
##### Accept

告知（服务器）客户端可以处理的内容的MIME类型。
	
	Accept: text/plain
	Accept: text/html
	Accept: image/*
	
##### Accept-Charset

告知（服务器）客户端可以处理的字符集类型。

	Accept-Charset: utf-8
	Accept-Charset: iso-8859-1
	Accept-Charset: utf-8, iso-8859-1;q=0.5, *;q=0.1
	

##### Accept-Encoding

告知（服务器）客户端可以处理的内容编码格式

	Accept-Encoding: gzip, deflate
	Accept-Encoding: gzip, compress, br
	Accept-Encoding: br;q=1.0, gzip;q=0.8, *;q=0.1

##### Origin

标识跨域资源请求（请求服务端设置Access-Control-Allow-Origin响应字段）

	Origin: http://www.abc.com

##### User-Agent

HTTP 客户端程序的信息，包括应用类型、操作系统、软件开发商以及版本号等。

	User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36

##### Cookie 

由服务器通过 Set-Cookie  首部发送并存储到客户端的 HTTP cookies 信息。

	Cookie: name=value; name2=value2; name3=value3

##### Authorization

设置HTTP身份验证的凭证。

	Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==

##### Referer

当前请求的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。
服务端一般使用 Referer 请求头识别访问来源，可能会以此进行资源防盗链、统计分析、日志记录以及缓存优化等。

	Referer: http://www.abc.org/wiki

##### X-Forwarded-For

客户端访问服务器的过程中如果需要经过HTTP代理或者负载均衡服务器，可以被用来获取最初发起请求的客户端的IP地址。

	X-Forwarded-For: client1, proxy1, proxy2
	X-Forwarded-For: 201.1.213.125, 72.42.33.28, 110.132.218.178

##### Forwarded

包含代理服务器的客户端的信息，即由于代理服务器在请求路径中的介入而被修改或丢失的信息。

	Forwarded: by=<identifier>; for=<identifier>; host=<host>; proto=<http|https>
	Forwarded: for=192.0.2.60; proto=http; by=203.0.113.43

##### TE

告知服务器客户端能够处理响应的传输编码方式及相对优先级。它和首部字段 Accept-Encoding 的功能很相像，但是用于传输编码。
除指定传输编码之外，还可以指定伴随 trailer 字段的分块传输编码的方式。应用后者时，只需把 trailers 赋值给该字段值。

	TE: gzip, deflate;q=0.5
	TE: trailers
	
##### Proxy-Authorization

包含了用户代理提供给代理服务器的用于身份验证的凭证。这个首部通常是在服务器返回了 `407 Proxy Authentication Required` 响应状态码及 `Proxy-Authenticate` 首部后发送的。

	Proxy-Authorization: <type> <credentials>
	Proxy-Authorization: Basic dGlwOjkpNLAGfFY5


#### 4. 响应首部字段

从服务器端向客户端返回响应报文时使用的首部。包含响应的附加内容，也会要求客户端附加额外的内容信息。

##### Age

响应主体在代理缓存中暂存的秒数。

	Age: 3600

##### ETag

特定版本资源的标识符。
可以近似理解成响应主体数据的MD5值，当响应主体没有发生改变时，不需要完整返回响应数据，只需要判断该首部即可。

	ETag: "737060cd8c284d8af7ad3082f209582d"

##### Location

令客户端重定向至指定URI。

	Location: http://www.abc.org/pub

##### Access-Control-Allow-Origin

指定哪些站点可以参与跨站资源共享。

	Access-Control-Allow-Origin: *
	Access-Control-Allow-Origin: <origin>

##### Set-Cookie 设置HTTP Cookie

	Set-Cookie: UserID=123; Max-Age=3600; Version=1

##### Proxy-Authenticate

代理服务器对客户端的验证信息。
Proxy-Authenticate 首部需要与 `407 Proxy Authentication Required` 响应一起发送。

	Proxy-Authenticate: Basic realm="Usagidesign Auth"

---

![欢迎关注公众号【全栈札记】](https://md.s1031.cn/xsj/2021_1_4_扫码_搜索联合传播样式-白色版.png)