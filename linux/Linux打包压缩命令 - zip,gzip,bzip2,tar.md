Linux打包压缩命令 - zip,gzip,bzip2,tar
---

## 常用打包压缩格式

> `.zip` `.gz` `.bz2` `.tar`

> **`.tar.gz` `.tar.bz2`**

## 一、`.zip` 格式
### 1. 压缩
#### 压缩文件

    zip 压缩文件名 源文件

#### 压缩目录

    zip -r 压缩文件名 源目录
    
* 实例

```
[vagrant/tmp] ]$zip a.zip a.md
  adding: a.md (stored 0%)
[vagrant/tmp] ]$zip -r abc.zip abc
  adding: abc/ (stored 0%)
  adding: abc/def/ (stored 0%)
  adding: abc/def/ghi/ (stored 0%)
[vagrant/tmp] ]$ll
drwxrwxr-x 3 vagrant       vagrant       4.0K Apr 19 00:53 abc/
-rw-rw-r-- 1 vagrant       vagrant        454 Apr 19 00:55 abc.zip
-rw-rw-r-- 1 vagrant       vagrant          0 Apr 19 00:53 a.md
-rw-rw-r-- 1 vagrant       vagrant        158 Apr 19 00:55 a.zip
```

### 2. 解压缩

    unzip 压缩文件名 [-d <文件解压缩后所要存储的目录>]
   
* 实例 
```
[vagrant/tmp] ]$mkdir zip
[vagrant/tmp] ]$unzip a.zip -d zip
Archive:  a.zip
 extracting: zip/a.md
[vagrant/tmp] ]$unzip abc.zip -d zip
Archive:  abc.zip
   creating: zip/abc/
   creating: zip/abc/def/
   creating: zip/abc/def/ghi/
[vagrant/tmp] ]$ll zip
drwxrwxr-x 3 vagrant vagrant 4.0K Apr 19 00:53 abc/
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 00:53 a.md
```

## 二、`.gz` 格式
### 1. 压缩
#### 压缩文件

    1. gzip 源文件
> 注意：源文件会消失！

    2. gzip -c 源文件 > 压缩文件
> 压缩文件，源文件保留

    3. gzip -r 目录
> 压缩目录下所有子文件，但是不能压缩目录


#### 压缩目录
> gzip 不能压缩目录


* 实例
```
[vagrant/tmp] ]$gzip -c a.md > a.md.gz
[vagrant/tmp] ]$ll
drwxrwxr-x 3 vagrant       vagrant       4.0K Apr 19 00:53 abc/
-rw-rw-r-- 1 vagrant       vagrant          0 Apr 19 00:53 a.md
-rw-rw-r-- 1 vagrant       vagrant         25 Apr 19 04:11 a.md.gz
[vagrant/tmp] ]$ll abc
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:12 a
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:12 b
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:12 c
[vagrant/tmp] ]$gzip -r abc
[vagrant/tmp] ]$ll abc
-rw-rw-r-- 1 vagrant vagrant   22 Apr 19 04:12 a.gz
-rw-rw-r-- 1 vagrant vagrant   22 Apr 19 04:12 b.gz
-rw-rw-r-- 1 vagrant vagrant   22 Apr 19 04:12 c.gz
```

### 2. 解压缩

    1. gzip -d 压缩文件

    2. gunzip 压缩文件
    
* 实例
```
[vagrant/tmp/tmp] ]$gzip -d a.md.gz
[vagrant/tmp/tmp] ]$ll
drwxrwxr-x 3 vagrant vagrant 4.0K Apr 19 04:13 abc/
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:11 a.md
[vagrant/tmp/tmp] ]$gzip -dr abc/
[vagrant/tmp/tmp] ]$ll abc
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:12 a
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:12 b
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:12 c
```

## 三、`.bz2` 格式
### 1. 压缩
#### 压缩文件

    1. bzip2 源文件
> 注意：源文件会消失！

    2. bzip2 -k 源文件
> 压缩文件，源文件保留

#### 压缩目录
> bzip2 不能压缩目录


* 实例
```
[vagrant/tmp/tmp] ]$bzip2 -k a.md
[vagrant/tmp/tmp] ]$ll
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:11 a.md
-rw-rw-r-- 1 vagrant vagrant   14 Apr 19 04:11 a.md.bz2
[vagrant/tmp/tmp] ]$rm a.md.bz2
[vagrant/tmp/tmp] ]$bzip2 a.md
[vagrant/tmp/tmp] ]$ll
-rw-rw-r-- 1 vagrant vagrant   14 Apr 19 04:11 a.md.bz2
```

### 2. 解压缩

    1. bzip2 -d 压缩文件

> 解压缩，默认不保留压缩文件。加 `-k` 可保留压缩文件

    2. gunzip 压缩文件

> 解压缩，默认不保留压缩文件。加 `-k` 可保留压缩文件

* 实例
```
[vagrant/tmp/tmp] ]$bzip2 -dk a.md.bz2
[vagrant/tmp/tmp] ]$ll
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:11 a.md
-rw-rw-r-- 1 vagrant vagrant   14 Apr 19 04:11 a.md.bz2
[vagrant/tmp/tmp] ]$rm a.md
[vagrant/tmp/tmp] ]$bunzip2 -k a.md.bz2
[vagrant/tmp/tmp] ]$ll
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:11 a.md
-rw-rw-r-- 1 vagrant vagrant   14 Apr 19 04:11 a.md.bz2
[vagrant/tmp/tmp] ]$rm a.md
[vagrant/tmp/tmp] ]$bzip2 -d a.md.bz2
[vagrant/tmp/tmp] ]$ll
-rw-rw-r-- 1 vagrant vagrant    0 Apr 19 04:11 a.md
```

## 四、`.tar` 格式
### 1. 打包

    tar -cvf 打包文件名 源文件或目录

* 选项
> `-c` : 打包
> `-v` : 显示打包过程
> `-f` : 指定打包后的文件名

* 实例
```
[vagrant/tmp/tmp] ]$tar -cvf abc.tar abc
abc/
abc/def/
abc/def/ghi/
abc/a
abc/b
abc/c
[vagrant/tmp/tmp] ]$ll
drwxrwxr-x 3 vagrant vagrant 4.0K Apr 19 04:22 abc/
-rw-rw-r-- 1 vagrant vagrant  10K Apr 19 07:02 abc.tar
```

### 2. 解打包

    tar -xvf 打包文件名
    
* 选项
> `-x` : 解打包

* 实例
```
vagrant/tmp/tmp] ]$tar -xvf abc.tar
abc/
abc/def/
abc/def/ghi/
abc/a
abc/b
abc/c
[vagrant/tmp/tmp] ]$ll
drwxrwxr-x 3 vagrant vagrant 4.0K Apr 19 04:22 abc/
-rw-rw-r-- 1 vagrant vagrant  10K Apr 19 07:02 abc.tar
```

## 五、`.tar.gz` 格式

> 其实，`.tar.gz` 格式是先将文件或目录打包文 `.tar` 格式，再压缩为 `.gz` 格式

### 1. 压缩

    tar -zcvf 压缩包名.tar.gz 源文件
    
* 选项
> `-z` : 压缩为 .tar.gz 格式

### 2. 解压缩

    tar -zxvf 压缩包名.tar.gz
    
* 选项
> `-x` : 解压缩
> `-t` : 查看压缩保内文件，但是不解压缩

* 实例
```
[vagrant/tmp/tmp] ]$tar -zcvf abc.tar.gz abc
abc/
abc/def/
abc/def/ghi/
abc/a
abc/b
abc/c
[vagrant/tmp/tmp] ]$ll
total 8.0K
drwxrwxr-x 3 vagrant vagrant 4.0K Apr 19 04:22 abc/
-rw-rw-r-- 1 vagrant vagrant  204 Apr 19 07:27 abc.tar.gz
[vagrant/tmp/tmp] ]$rm -rf abc
[vagrant/tmp/tmp] ]$tar -ztvf abc.tar.gz
drwxrwxr-x vagrant/vagrant   0 2018-04-19 04:22 abc/
drwxrwxr-x vagrant/vagrant   0 2018-04-19 00:52 abc/def/
drwxrwxr-x vagrant/vagrant   0 2018-04-19 00:52 abc/def/ghi/
-rw-rw-r-- vagrant/vagrant   0 2018-04-19 04:12 abc/a
-rw-rw-r-- vagrant/vagrant   0 2018-04-19 04:12 abc/b
-rw-rw-r-- vagrant/vagrant   0 2018-04-19 04:12 abc/c
[vagrant/tmp/tmp] ]$ll
total 4.0K
-rw-rw-r-- 1 vagrant vagrant 204 Apr 19 07:27 abc.tar.gz
[vagrant/tmp/tmp] ]$tar -zxvf abc.tar.gz
abc/
abc/def/
abc/def/ghi/
abc/a
abc/b
abc/c
[vagrant/tmp/tmp] ]$ll
total 8.0K
drwxrwxr-x 3 vagrant vagrant 4.0K Apr 19 04:22 abc/
-rw-rw-r-- 1 vagrant vagrant  204 Apr 19 07:27 abc.tar.gz
```

## 六、`.tar.bz2` 格式

> 其实，`.tar.bz2` 格式是先将文件或目录打包文 `.tar` 格式，再压缩为 `.bz2` 格式

### 1. 压缩

    tar -jcvf 压缩包名.tar.bz2 源文件
    
* 选项
> `-j` : 压缩为 .tar.bz2 格式

### 2. 解压缩

    tar -jxvf 压缩包名.tar.bz2
    
* 选项
> `-x` : 解压
> `-t` : 查看压缩保内文件，但是不解压缩
> `-C` : 指定解压的目录（注意，该选项必须放在后面）

* 实例
```
[vagrant/tmp/tmp] ]$tar -jcvf abc.tar.bz2 abc
abc/
abc/def/
abc/def/ghi/
abc/a
abc/b
abc/c
[vagrant/tmp/tmp] ]$ll
total 8.0K
drwxrwxr-x 3 vagrant vagrant 4.0K Apr 19 04:22 abc/
-rw-rw-r-- 1 vagrant vagrant  210 Apr 19 07:33 abc.tar.bz2
[vagrant/tmp/tmp] ]$tar -jtvf abc.tar.bz2
drwxrwxr-x vagrant/vagrant   0 2018-04-19 04:22 abc/
drwxrwxr-x vagrant/vagrant   0 2018-04-19 00:52 abc/def/
drwxrwxr-x vagrant/vagrant   0 2018-04-19 00:52 abc/def/ghi/
-rw-rw-r-- vagrant/vagrant   0 2018-04-19 04:12 abc/a
-rw-rw-r-- vagrant/vagrant   0 2018-04-19 04:12 abc/b
-rw-rw-r-- vagrant/vagrant   0 2018-04-19 04:12 abc/c
[vagrant/tmp/tmp] ]$tar -jxvf abc.tar.bz2 -C /tmp
abc/
abc/def/
abc/def/ghi/
abc/a
abc/b
abc/c
[vagrant/tmp/tmp] ]$ll /tmp/
drwxrwxr-x 3 vagrant       vagrant       4.0K Apr 19 04:22 abc/
drwxrwxr-x 3 vagrant       vagrant       4.0K Apr 19 07:33 tmp/
```