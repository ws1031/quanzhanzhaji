Elasticsearch7.X 及 IK 中文分词插件安装
---

![](https://md.s1031.cn/xsj/2021_1_16_elasticsearch横图2.png)

### 一、安装Java并配置 JAVA_HOME 环境变量
> 由于Elasticsearch是使用Java构建的，所以首先需要安装 [Java 8 或更高版本](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 才能运行。
> 所有Elasticsearch节点和客户机上都应该使用相同的JVM版本。

#### 1. 安装Java

根据不同的系统，从 https://www.oracle.com/technetwork/java/javase/downloads/index.html 下载相应Java版本进行安装。

##### **CentOS安装Java示例**

* 下载Java RPM安装包，笔者这里下载的是 `jdk-12.0.1_linux-x64_bin.rpm`

* 使用 `rpm -ivh jdk-12.0.1_linux-x64_bin.rpm` 命令进行安装。
```
Preparing...                          ################################## [100%]
Updating / installing...
   1:jdk-12.0.1-2000:12.0.1-ga        ################################## [100%]
```

##### **Ubuntu安装Java示例**

* 下载Java DEB安装包

* 使用 `dpkg -i jdk-12.0.1_linux-x64_bin.deb` 命令进行安装。

Ubuntu还可以参考 [How To Install Java with Apt-Get on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04) 安装Java


#### 2. 配置 JAVA_HOME

##### **定位JDK安装路径**

* which java
```shell
[root/usr/local/src] ]$which java
/usr/bin/java
```
* ls -l /usr/bin/java
```shell
[root/usr/local/src] ]$ls -l /usr/bin/java
lrwxrwxrwx 1 root root 22 Jul  5 17:54 /usr/bin/java -> /etc/alternatives/java
```
* ls -l /etc/alternatives/java
```shell
[root/usr/local/src] ]$ls -l /etc/alternatives/java
lrwxrwxrwx 1 root root 29 Jul  5 17:54 /etc/alternatives/java -> /usr/java/jdk-12.0.1/bin/java
```
此时，我们可以确定java的安装目录为： /usr/java/jdk-12.0.1

##### 2. 配置JAVA_HOME

* `vim /etc/environment` 编辑环境变量配置文件，填入 JAVA_HOME 环境变量，保存并退出
```shell
JAVA_HOME="/usr/java/jdk-12.0.1"
```
* `source /etc/environment` 重新载入配置文件
* `echo $JAVA_HOME` 查看环境变量是否生效
```shell
[root/usr/local/src] ]$echo $JAVA_HOME
/usr/java/jdk-12.0.1
```

### 二、安装 Elasticsearch

> 参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html

#### 1. CentOS安装

> 参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html

##### **方法1：使用 yum 命令安装**
* 在 `/etc/yum.repos.d/` 目录下创建一个名为 `elasticsearch.repo` 的文件，填写如下内容。
```shell
[elasticsearch-7.x]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
* 执行如下命令安装 Elasticsearch
```shell
sudo yum install elasticsearch
```

##### **方法2：手动下载 Elasticsearch RPM 安装包进行安装**

Elasticsearch安装包下载地址：https://www.elastic.co/cn/downloads/elasticsearch   
以 Elasticsearch v7.2.0 为例，其他版本只需要修改链接中的版本号即可。

* 下载 Elasticsearch RPM 安装包
```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.0-x86_64.rpm
```
* 下载 SHA 校验文件，并对下载的 RPM 包进行校验
```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.0-x86_64.rpm.sha512

shasum -a 512 -c elasticsearch-7.2.0-x86_64.rpm.sha512
```
> 若出现 `shasum: command not found`，则通过 `yum -y install perl-Digest-SHA` 命令安装 `shasum` 命令。

若校验成功，则输出 `elasticsearch-7.2.0-x86_64.rpm: OK`

* 执行如下命令安装 Elasticsearch
```shell
rpm -ivh elasticsearch-7.2.0-x86_64.rpm
```
```shell
Preparing...                          ################################## [100%]
Creating elasticsearch group... OK
Creating elasticsearch user... OK
Updating / installing...
   1:elasticsearch-0:7.2.0-1          ################################## [100%]
#### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
 sudo systemctl daemon-reload
 sudo systemctl enable elasticsearch.service
#### You can start elasticsearch service by executing
 sudo systemctl start elasticsearch.service
Created elasticsearch keystore in /etc/elasticsearch

```

#### 2. Ubuntu安装

> 参考：https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

##### **方法1：使用 apt 命令安装**
* 在安装 Elasticsearch 之前，首先要安装 `apt-transport-https` 包
```shell
sudo apt-get install apt-transport-https
```
* 将Elasticsearch仓库定义存储到 `/etc/apt/sources.list.d/elastic-7.x.list` 文件。
```shell
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
```
* 执行如下命令安装 Elasticsearch
```shell
sudo apt-get update && sudo apt-get install elasticsearch
```

##### **方法2：手动下载 Elasticsearch DEB 安装包进行安装**

Elasticsearch安装包下载地址：https://www.elastic.co/cn/downloads/elasticsearch   
以 Elasticsearch v7.2.0 为例，其他版本只需要修改链接中的版本号即可。

* 下载 Elasticsearch DEB 安装包
```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.0-amd64.deb
```
* 下载 SHA 校验文件，并对下载的 RPM 包进行校验
```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.0-amd64.deb.sha512

shasum -a 512 -c elasticsearch-7.2.0-amd64.deb.sha512
```
若校验成功，则输出 `elasticsearch-7.2.0-amd64.deb: OK`

* 执行如下命令安装 Elasticsearch
```shell
sudo dpkg -i elasticsearch-7.2.0-amd64.deb
```

#### 3. 默认安装目录结构

##### **配置文件目录**
```
/etc/elasticsearch
```

* 核心配置文件
```
/etc/elasticsearch/elasticsearch.yml
/etc/elasticsearch/jvm.options
```
##### **数据存储目录**
```
/var/lib/elasticsearch
```
##### **日志文件目录**
```
/var/log/elasticsearch
```
##### **命令文件目录**
```
/usr/share/elasticsearch/bin
```
##### **依赖包目录**
```
/usr/share/elasticsearch/lib
```
##### **模块目录**
```
/usr/share/elasticsearch/modules
```
##### **插件目录**
```
/usr/share/elasticsearch/plugins
```

#### 4. 打开Elasticsearch

##### **使用 systemd 管理 Elasticsearch**

* 将 Elasticsearch 设置为开机自启
```shell
systemctl daemon-reload
systemctl enable elasticsearch.service
```
* 开启和关闭 Elasticsearch
```shell
systemctl start elasticsearch.service
systemctl stop elasticsearch.service
```

##### **使用 SysV init 管理 Elasticsearch**

* 将 Elasticsearch 设置为开机自启
```shell
sudo update-rc.d elasticsearch defaults 95 10
```
* 开启和关闭 Elasticsearch
```shell
service elasticsearch start
service elasticsearch stop
```


### 三、安装 IK 中文分词插件

> IK 插件地址：https://github.com/medcl/elasticsearch-analysis-ik

#### 1. 使用 `elasticsearch-plugin` 安装

```shell
[root~] ]$cd /usr/share/elasticsearch/
[root/usr/share/elasticsearch] ]$

[root/usr/share/elasticsearch] ]$./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.0/elasticsearch-analysis-ik-7.2.0.zip
-> Downloading https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.0/elasticsearch-analysis-ik-7.2.0.zip
[=================================================] 100%  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@     WARNING: plugin requires additional permissions     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
* java.net.SocketPermission * connect,resolve
See http://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html
for descriptions of what these permissions allow and the associated risks.

Continue with installation? [y/N]y
-> Installed analysis-ik
```

#### 2. IK 中文分词插件目录

```shell
[root/usr/share/elasticsearch] ]$cd plugins/

[root/usr/share/elasticsearch/plugins] ]$ll
total 4.0K
drwxr-xr-x 2 root root 4.0K Jul  8 16:51 analysis-ik/

[root/usr/share/elasticsearch/plugins] ]$ll analysis-ik/
total 1.4M
-rw-r--r-- 1 root root 258K Jul  8 16:50 commons-codec-1.9.jar
-rw-r--r-- 1 root root  61K Jul  8 16:50 commons-logging-1.2.jar
-rw-r--r-- 1 root root  54K Jul  8 16:50 elasticsearch-analysis-ik-7.2.0.jar
-rw-r--r-- 1 root root 720K Jul  8 16:50 httpclient-4.5.2.jar
-rw-r--r-- 1 root root 320K Jul  8 16:50 httpcore-4.4.4.jar
-rw-r--r-- 1 root root 1.8K Jul  8 16:50 plugin-descriptor.properties
-rw-r--r-- 1 root root  125 Jul  8 16:50 plugin-security.policy
```


---


![欢迎关注公众号【全栈札记】](https://md.s1031.cn/xsj/2021_1_4_扫码_搜索联合传播样式-白色版.png)