OSI七层模型与TCP/IP五层模型简述
---

![OSI七层模型总览图](http://md.s1031.cn/xsj/2019_4_29_OSI七层模型总览图.gif)

#### **应用层**

OSI参考模型中最靠近用户的一层，是为计算机用户提供应用接口和各种网络服务，并规定应用程序中通信相关的细节。

包含协议有：

* HTTP（超文本传输协议，HyperText Transfer Protocol），默认端口80。
* HTTPS（超文本传输安全协议，Hypertext Transfer Protocol Secure），默认端口443。
* FTP（文件传输协议，File Transfer Protocol），默认端口20和21，其中20用于传输数据，21用于传输控制信息。
* SMTP（简单邮件传输协议，Simple Mail Transfer Protocol），端口25。
* DNS（域名系统，Domain Name System），端口53。
* DHCP（动态主机设置协议，Dynamic Host Configuration Protocol）。
* SSH（安全Shell协议，Secure Shell），默认端口22。
* TELNET，默认端口23。

#### **表示层**

数据的表示、安全、压缩。  
将应用处理的信息转换为适合网络传输的格式，或将来自下一层的数据转换为上层能够处理的格式。因此它主要负责数据格式的转换。
具体来说，就是将设备固有的数据格式转换为网络标准传输格式。不同设备对同一比特流解释的结果可能会不同。因此，使它们保持一致是这一层的主要作用。

格式有：JPEG、ASCll、DECOIC、加密格式等。

#### **会话层**

建立、管理、终止通信连接（数据流动的逻辑通路），以及数据的分割等数据传输相关的管理。  
该层的通信由不同设备中的应用程序之间的服务请求和响应组成。 

#### **传输层**

定义传输数据的协议端口号，以及流控和差错校验。数据包一旦离开网卡即进入网络传输层。

数据传输单位：数据段（Segments）。

常用设备有：四层交换机、四层路由器。

协议有：

* TCP（传输控制协议，Transmission Control Protocol）
* UDP（无连接的传输层协议）

#### **网络层**

进行IP地址寻址，实现不同网络之间的路由选择。

数据传输单位：数据包（Packet）。

常用设备有：路由器、三层交换机。

协议有：

* ICMP（互联网控制报文协议，Internet Control Message Protocol。我们通常使用的 `ping` 命令就是使用ICMP协议）
* IGMP（互联网组管理协议，Internet Group Management Protocol）
* IP（互联网协议地址，Internet Protocol Address。包含IPV4和IPV6两大类） 
* ARP（地址解析协议，Address Resolution Protocol。在OSI模型中ARP协议属于链路层；而在TCP/IP模型中，ARP协议属于网络层）

#### **数据链路层**

建立逻辑连接、进行硬件地址寻址、差错校验等功能。（由底层网络定义协议）。  
将比特组合成字节进而组合成帧，用MAC地址访问介质，错误发现但不能纠正。

数据传输单位：帧（Frame）。

常用设备有：网桥、以太网交换机、网卡。

协议有：

* ARP（地址解析协议，Address Resolution Protocol。在OSI模型中ARP协议属于链路层；而在TCP/IP模型中，ARP协议属于网络层）
* PPP（点对点协议，Point to Point Protocol。为在点对点连接上传输多协议数据包提供了一个标准方法）

#### **物理层**

建立、维护、断开物理连接。  
实际最终信号的传输是通过物理层实现的。通过物理介质传输比特流。  

数据传输单位：比特（Bit）。

常用设备有：集线器、中继器、调制解调器、网线、双绞线、同轴电缆。

#### **OSI七层模型通信流程**

![OSI七层模型通信流程](http://md.s1031.cn/xsj/2019_4_29_OSI七层模型通信流程.png)

#### **TCP/IP五层协议和OSI的七层协议对应关系**

![TCP/IP五层协议和OSI的七层协议对应关系](http://md.s1031.cn/xsj/2019_4_29_OSI七层模型与TCP-IP五层模型对应关系.png)


参考：
[OSI七层模型详解](https://www.jianshu.com/p/69fe871c2d0e)
[OSI七层模型与TCP/IP五层模型](https://www.cnblogs.com/qishui/p/5428938.html)
图解TCP/IP