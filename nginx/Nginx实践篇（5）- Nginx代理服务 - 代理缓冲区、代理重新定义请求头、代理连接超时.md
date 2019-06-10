### 1. 代理缓冲区

> 代理服务器可以缓存一些响应数据，来减少I/O损耗，数据默认存储在内存中，当内存不够时，会存储到硬盘上。

#### **[proxy_buffering](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffering)**
proxy_buffering这个参数用来控制是否打开后端响应内容的缓冲区，如果这个设置为off，那么proxy_buffers和proxy_busy_buffers_size这两个指令将会失效。 但是无论proxy_buffering是否开启，对proxy_buffer_size都是生效的。

proxy_buffering开启的情况下，nignx会把后端返回的内容先放到缓冲区当中，然后再返回给客户端(边收边传，不是全部接收完再传给客户端)。 临时文件由proxy_max_temp_file_size和proxy_temp_file_write_size这两个指令决定的。

如果proxy_buffering关闭，那么nginx会立即把从后端收到的响应内容传送给客户端，每次取的大小为proxy_buffer_size的大小，这样效率肯定会比较低。

注： proxy_buffering启用时，要提防使用的代理缓冲区太大。这可能会吃掉你的内存，限制代理能够支持的最大并发连接数。

	Syntax:	proxy_buffering on | off;
	Default:	proxy_buffering on;
	Context:	http, server, location

#### **[proxy_buffer_size ](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffer_size)**
后端服务器的响应头会放到proxy_buffer_size当中，这个大小默认等于proxy_buffers当中的设置单个缓冲区的大小。 proxy_buffer_size只是响应头的缓冲区，没有必要也跟着设置太大。

	Syntax:	proxy_buffer_size size;
	Default:	proxy_buffer_size 4k|8k;
	Context:	http, server, location

#### **[proxy_buffers](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffers)**

proxy_buffers的缓冲区大小一般会设置的比较大，以应付大网页。 proxy_buffers当中单个缓冲区的大小是由系统的内存页面大小决定的，Linux系统中一般为4k。 proxy_buffers由缓冲区数量和缓冲区大小组成的。总的大小为number*size。

若某些请求的响应过大,则超过_buffers的部分将被缓冲到硬盘(缓冲目录由_temp_path指令指定), 当然这将会使读取响应的速度减慢, 影响用户体验. 可以使用proxy_max_temp_file_size指令关闭磁盘缓冲.

	Syntax:	proxy_buffers number size;
	Default:	proxy_buffers 8 4k|8k;
	Context:	http, server, location

#### **[proxy_busy_buffers_size](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_busy_buffers_size)**

proxy_busy_buffers_size不是独立的空间，他是proxy_buffers和proxy_buffer_size的一部分。nginx会在没有完全读完后端响应的时候就开始向客户端传送数据，所以它会划出一部分缓冲区来专门向客户端传送数据(这部分的大小是由proxy_busy_buffers_size来控制的，建议为proxy_buffers中单个缓冲区大小的2倍)，然后它继续从后端取数据，缓冲区满了之后就写到磁盘的临时文件中。

	Syntax:	proxy_busy_buffers_size size;
	Default:	proxy_busy_buffers_size 8k|16k;
	Context:	http, server, location
	
### 2. 重新定义或添加传递给代理服务器的请求头

#### **[proxy_set_header](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)**

	Syntax:	proxy_set_header field value;
	Default:	
		proxy_set_header Host $proxy_host;
		proxy_set_header Connection close;
	Context:	http, server, location

允许重新定义或添加传递给代理服务器的请求头。value可以包含文本、变量或者它们的组合。 当前配置级别中没有定义proxy_set_header指令时，会从上一级别继承配置。 默认情况下，只有两个请求头会被重新定义：

	proxy_set_header Host $proxy_host;
	proxy_set_header Connection close;

如果启用缓存，来原始请求的请求头 “If-Modified-Since”, “If-Unmodified-Since”, “If-None-Match”, “If-Match”, “Range”, 和 “If-Range” 将不会被代理服务器传递。

可以通过下面的配置使请求头 “Host” 不被代理服务器替换：

	proxy_set_header Host $http_host;

### 3. 代理超时

#### **[proxy_connect_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_connect_timeout)**

	Syntax:	proxy_connect_timeout time;
	Default:	proxy_connect_timeout 60s;
	Context:	http, server, location

定义Nginx作为代理，到后端服务器中间的连接超时时间，默认为60秒。
应该注意的是，这个超时时通常不能超过75秒。


#### **[proxy_read_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout)**

	Syntax:	proxy_read_timeout time;
	Default:	proxy_read_timeout 60s;
	Context:	http, server, location

定义了从代理服务器读取响应的超时时间，默认为60秒。
超时只设置在两个连续的读取操作之间，而不是整个响应的传输。
如果代理服务器在这个时间内没有传输任何数据，那么连接就关闭了。


#### **[proxy_send_timeout](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_send_timeout)**

	Syntax:	proxy_send_timeout time;
	Default:	proxy_send_timeout 60s;
	Context:	http, server, location

定义了将请求发送到代理服务器的超时时间，默认为60秒。
超时只设置在两个连续的写操作之间，而不是整个请求的传输。
如果代理服务器在这个时间内没有收到任何数据，那么连接就关闭了。

### 4. 代理常用配置注解

```nginxconf
location / {
	# 配置反向代理到本机的8080端口
	proxy_pass http://127.0.0.1:8080;

	# 配置请求客户端真实的 Host 信息
	proxy_set_header Host $http_host;
	# 配置请求用户真实的IP信息
	proxy_set_header X-Real-IP $remote_addr;

	# 连接超时时间为30秒
	proxy_connect_timeout 30;
	# 读取响应超时时间为60秒
	proxy_send_timeout 60;
	# 发送请求超时时间为60秒
	proxy_read_timeout 60;

	# 开启代理缓冲区
	proxy_buffering on;
	# 响应头的缓冲区设为32k
	proxy_buffer_size 32k;
	# 网页内容缓冲区个数为4，单个大小为128k
	proxy_buffers 4 128k;
	proxy_busy_buffers_size 256k;
	# 缓冲区临时文件最大为 256k
	proxy_max_temp_file_size 256k;

}
```