Linux（CentOS）软件管理（2）- yum在线安装
---


> 手动安装RPM包时，解决依赖性问题是一个非常大的麻烦。如果所有RPM包都是用手工安装，则RPM包的使用难道较大。因此，Red Hat 系列推出了 “yum 在线安装” 方法。


> yum 在线安装优点：将所有软件包放在官方服务器上，当进行yum在线安装时，可以自动解决依赖性问题。

## 一、yum 源配置文件

### 1. 位置

    /etc/yum.repos.d/CentOS-Base.repo

### 2. 内容实例（CentOS7）
```
[root/etc/yum.repos.d]$cat CentOS-Base.repo
# CentOS-Base.repo
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### 3. 说明

 \ | 说明
 -|-
 [base] | 容器名称，一定要放在[]中
 name|容器说明，可以自己随便写
 mirrorlist|镜像站点
 baseurl|yum源服务器地址
 enabled|此容器是否生效。不写或者enable=1生效，enable=0不生效
 gpgcheck|RPM数字证书是否生效。1：生效；0：不生效
 gpgkey|RPM数字证书公钥文件保存位置。一般不要修改。


#### **关于 baseurl**

    baseurl 默认是CentOS官方的yum源服务器地址。是可以使用的，但是由于是国外网址，可能会出现访问较慢的问题，这时可以替换成其他yum源服务器地址。

#### **常用yum源服务器地址**

\ | 地址
-|-
网易|http://mirrors.163.com/.help/centos.html
搜狐|http://mirrors.sohu.com/help/centos.html
清华大学|https://mirrors.tuna.tsinghua.edu.cn/help/centos/
中国科学技术大学|http://centos.ustc.edu.cn/help/centos.html

> 更多开源镜像见【四、常用的Linux国内开源镜像站整理】

### 4. 清除 yum 缓存

> yum 会下载容器的清单到本机的 `/var/cache/yum` 中，当我们修改了容器的网址却没有修改容器的名称时，可能会造成本机缓存中的列表与yum服务其中的列表不同步，此时就会出现无法更新的问题。
> 这种情况下，就需要清除 yum 缓存。

#### **命令**
    
    yum clean [packages|headers|all]
    
#### **参数**

> `packages`：删除已下载的软件缓存文件
> `headers`：删除已下载的软件文件头缓存
> `all`：删除所有的容器缓存数据

#### **实例**
```
[root/var/cache/yum/x86_64/7]$yum clean packages
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
0 package files removed

[root/var/cache/yum/x86_64/7]$yum clean headers
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
0 header files removed

[root/var/cache/yum/x86_64/7]$yum clean all
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
Cleaning up everything
Maybe you want: rm -rf /var/cache/yum, to also free up space taken by orphaned data from disabled or removed repos
Cleaning up list of fastest mirrors
```

### 5. 列出当前 yum 服务器的所有容器

#### **命令**

    yum repolist all

#### **实例**
```
[vagrant~]$yum repolist all
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.huaweicloud.com
repo id                            repo name                         status
C7.0.1406-base/x86_64              CentOS-7.0.1406 - Base            disabled
C7.0.1406-centosplus/x86_64        CentOS-7.0.1406 - CentOSPlus      disabled
...省略...
base/7/x86_64                      CentOS-7 - Base                   enabled: 9,911
base-debuginfo/x86_64              CentOS-7 - Debuginfo              disabled
base-source/7                      CentOS-7 - Base Sources           disabled
...省略...
extras/7/x86_64                    CentOS-7 - Extras                 enabled:   291
extras-source/7                    CentOS-7 - Extras Sources         disabled
fasttrack/7/x86_64                 CentOS-7 - fasttrack              disabled
updates/7/x86_64                   CentOS-7 - Updates                enabled:   626
updates-source/7                   CentOS-7 - Updates Sources        disabled
repolist: 10,828
```

### 6. `yum-fastestmirror` 插件

    对centos系统管理员来说，yum绝对是个好东西，只可惜，官方yum源的速度实在让人不敢恭维，而非官方的yum源又五花八门，让人难以取舍。
 
    幸运的是，yum-fastestmirror插件弥补了这一缺陷：自动选择最快的yum源。

#### **安装**

    yum -y install yum-fastestmirror

#### **配置文件**

    /etc/yum/pluginconf.d/fastestmirror.conf

    /var/cache/yum/timedhosts.txt #yum镜像的速度测试记录文件

## 二、使用光盘搭建yam源 

> 在没有网络的情况下，我们就无法使用 yum 在线安装。这时，我们可以使用光盘搭建 yum 源。

### 1. 挂载光盘
* 插入光盘
* 建立挂载点
```
[root/dev]$mkdir /mnt/cdrom
```
* 挂载光盘
```
[root/dev]$mount /dev/cdrom /mnt/cdrom/
mount: /dev/sr0 is write-protected, mounting read-only
```
### 2. 使网络yum源失效

* 进入 `/etc/yum.repos.d` 目录，修改网络yum源文件后缀名，使其失效
```
[root~]$cd /etc/yum.repos.d/
[root/etc/yum.repos.d]$mv CentOS-Base.repo CentOS-Base.repo.bak
```
### 3. 使光盘yum源生效

```
[root/etc/yum.repos.d]$vim CentOS-Media.repo
# CentOS-Media.repo

[c7-media]
name=CentOS-$releasever - Media
baseurl=file:///mnt/cdrom   # 地址为光盘挂载地址
#        file:///media/CentOS/ # 注释掉不存在的地址
#        file:///media/cdrom/
#        file:///media/cdrecorder/
gpgcheck=1
enabled=1 # 将enabled=0改为enabled=1，让这个yum源文件生效
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### 4. 可以使用 `yum list` 命令查看光盘yum源中的可用软件包

## 三、yum 常用命令

### 1. 查询

    yum [list|info|search|provides|whatprovides] [参数]

#### **查询所有可用软件包列表**

    yum list

#### **使用通配符匹配查询可用软件包列表**

    yum list [pattern]
    
* 实例
```
[root~]$yum list g?nc-[a-e]*
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.sohu.com
Available Packages
gvnc-devel.i686                      0.7.0-3.el7                   base

gvnc-devel.x86_64                    0.7.0-3.el7                   base
```

#### **查询服务器上可供本机进行升级的软件列表**

    yum list updates

* 实例
```
[root/etc/yum.repos.d]$yum list updates
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.huaweicloud.com
Updated Packages
NetworkManager.x86_64                       1:1.10.2-14.el7_5               updates
NetworkManager-libnm.x86_64                 1:1.10.2-14.el7_5               updates
NetworkManager-ppp.x86_64                   1:1.10.2-14.el7_5               updates
...省略...
```

#### **搜索服务器上所有与关键字相关的包**

    yum search [关键字]
    yum search all [关键字]

#### 实例
```
[root~]$yum search fastestmirror
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.huaweicloud.com
==================================== N/S matched: fastestmirror ====================
yum-plugin-fastestmirror.noarch : Yum plugin which chooses fastest repository from a mirrorlist

  Name and summary matches only, use "search all" for everything.
[root~]$yum search all fastestmirror
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.huaweicloud.com
==================================== Matched: fastestmirror ========================
yum-plugin-fastestmirror.noarch : Yum plugin which chooses fastest repository from a mirrorlist
```

### 2. 安装

#### **语法**

    yum -y install [包名]

#### **选项**

> `-y`：提示是否安装依赖时，自动回答yes
> `install`：安装

#### **实例**
```
[root~]$yum install -y gcc
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.aliyun.com
 * updates: mirrors.huaweicloud.com
base                                                                | 3.6 kB  00:00:00
extras                                                              | 3.4 kB  00:00:00
...省略...
Resolving Dependencies
--> Running transaction check
---> Package gcc.x86_64 0:4.8.5-28.el7 will be updated
--> Processing Dependency: gcc = 4.8.5-28.el7 for package: gcc-c++-4.8.5-28.el7.x86_64
...省略...
---> Package libstdc++-devel.x86_64 0:4.8.5-28.el7_5.1 will be an update
--> Finished Dependency Resolution

Dependencies Resolved

=======================================================================================
 Package          Arch          Version     Repository       Size
=======================================================================================
Updating:
 gcc            x86_64    4.8.5-28.el7_5.1    updates     16 M
Updating for dependencies:
 cpp            x86_64    4.8.5-28.el7_5.1    updates     5.9 M
...省略...
Transaction Summary
=========================================================================================
Upgrade  1 Package (+8 Dependent packages)

Total download size: 32 M
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
(1/9): gcc-4.8.5-28.el7_5.1.x86_64.rpm                               |  16 MB  00:00:03
(2/9): libgcc-4.8.5-28.el7_5.1.i686.rpm                              | 108 kB  00:00:00
...省略...
(9/9): gcc-c++-4.8.5-28.el7_5.1.x86_64.rpm                           | 7.2 MB  00:00:06
----------------------------------------------------------------------------------------
Total                                                       4.9 MB/s |  32 MB  00:00:06
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Updating   : libgcc-4.8.5-28.el7_5.1.x86_64                         1/18
  Updating   : libstdc++-4.8.5-28.el7_5.1.x86_64                      2/18
...省略...
  Cleanup    : libgomp-4.8.5-28.el7.x86_64                            18/18
  Verifying  : libgomp-4.8.5-28.el7_5.1.x86_64                        1/18
  Verifying  : libstdc++-4.8.5-28.el7_5.1.i686                        2/18
  ...省略...
  Verifying  : libstdc++-4.8.5-28.el7.i686                            18/18

Updated:
  gcc.x86_64 0:4.8.5-28.el7_5.1

Dependency Updated:
  cpp.x86_64 0:4.8.5-28.el7_5.1         gcc-c++.x86_64 0:4.8.5-28.el7_5.1       libgcc.i686 0:4.8.5-28.el7_5.1                libgcc.x86_64 0:4.8.5-28.el7_5.1     libgomp.x86_64 0:4.8.5-28.el7_5.1
  libstdc++.i686 0:4.8.5-28.el7_5.1     libstdc++.x86_64 0:4.8.5-28.el7_5.1     libstdc++-devel.x86_64 0:4.8.5-28.el7_5.1

Complete!
```

### 3. 升级

> 在生产服务器中，除非当前软件版本有重大的BUG，否则尽量不对软件进行升级
> 使用 `yum -y update [包名]` 命令时要极其谨慎，若未输入 `包名`，命令会变成 `yum -y update`，该命令**升级所有包同时也升级软件和系统内核**，有可能造成系统重启甚至崩溃！

#### **语法**

    yum -y update [包名]
    yum -y upgrade [包名]
    
#### **选项**

> `-y`：提示是否安装依赖时，自动回答yes
> `update`：更新，后面接要更新的软件，否则将会更新所有包，改变软件设置和系统设置，更新系统版本内核
> `upgrade`：升级，后面接要升级的软件，否则将会升级所有包。

### 4. 卸载

> 在生产服务器中，服务器应使用最小化安装，用什么软件装什么软件，尽量不卸载

#### **语法**

    yum -y remove [包名]
    
#### **选项**

> `-y`：提示是否安装依赖时，自动回答yes
> `remove`：卸载软件

#### **实例**
```
[root~]$yum -y remove zip
Loaded plugins: fastestmirror
Resolving Dependencies
--> Running transaction check
---> Package zip.x86_64 0:3.0-11.el7 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

===========================================================================================
 Package        Arch        Version         Repository          Size
===========================================================================================

Removing:
 zip            x86_64      3.0-11.el7      installed           796 k

Transaction Summary
===========================================================================================

Remove  1 Package

Installed size: 796 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Erasing    : zip-3.0-11.el7.x86_64                                1/1
  Verifying  : zip-3.0-11.el7.x86_64                                1/1

Removed:
  zip.x86_64 0:3.0-11.el7

```

### 5. yum 软件组管理命令

#### **列出所有可用软件组列表**

    yum grouplist

* 实例
```
[root~]$yum grouplist
Loaded plugins: fastestmirror
There is no installed groups file.
Maybe run: yum groups mark convert (see man yum)
Loading mirror speeds from cached hostfile
 * base: mirrors.huaweicloud.com
 * extras: mirrors.huaweicloud.com
 * updates: mirrors.sohu.com
Available Environment Groups:
   Minimal Install
   Compute Node
   Infrastructure Server
   File and Print Server
   Basic Web Server
   Virtualization Host
   Server with GUI
   GNOME Desktop
   KDE Plasma Workspaces
   Development and Creative Workstation
Available Groups:
   Compatibility Libraries
   Console Internet Tools
   Development Tools
   Graphical Administration Tools
   Legacy UNIX Compatibility
   Scientific Support
   Security Tools
   Smart Card Support
   System Administration Tools
   System Management
Done
```

#### **安装指定软件组，组名可以有grouplist查询出来**

    yum groupinstall [软件组名]

#### **卸载指定软件组**

    yum groupremove [软件组名]

## 四、常用的Linux国内开源镜像站整理

【阿里云开源镜像站】：https://opsx.alibaba.com/

【华为开源镜像站】：http://mirrors.huaweicloud.com/

【网易开源镜像站】：http://mirrors.163.com/ 

【搜狐开源镜像站】：http://mirrors.sohu.com/

【清华大学开源软件镜像站】：https://mirrors.tuna.tsinghua.edu.cn/

【中国科学技术大学开源镜像站】：http://centos.ustc.edu.cn/

【上海大学开源镜像站】：http://mirrors.shu.edu.cn/

【浙江大学开源镜像站】：http://mirrors.zju.edu.cn/

【上海交通大学开源镜像站】：http://ftp.sjtu.edu.cn/

【重庆大学开源软件镜像站】：http://mirrors.cqu.edu.cn/

【南京大学开源镜像站】：http://mirrors.nju.edu.cn/

【南京邮电大学开源软件镜像站】：http://mirrors.njupt.edu.cn/

【东软信息学院开源镜像站】：http://mirrors.neusoft.edu.cn/

【兰州大学开源社区镜像站】：http://mirror.lzu.edu.cn/
