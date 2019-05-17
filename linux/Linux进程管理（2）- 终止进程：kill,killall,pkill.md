Linux进程管理（2）- 终止进程：kill,killall,pkill
---

## 一、`kill` 命令

    根据进程ID终止指定的进程

### 1. 命令格式

    kill [参数] [进程id]

### 2. `kill -l`（查看可用的进程信号）

```
[vagrant~] ]$kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

* 进程信号说明

信号代号|信号名称|说明
-|-|-
**1**|**SIGHUP**|**该信号让进程立即关闭，然后重新读取配置文件之后重启（常用）**
2|SIGINT|程序终止信号，用于制止前台进程。相当于输出ctrl+c快捷键
8|SIGFPE|在发生致命的算数错误时发出，不仅包含浮点运算错误，还包含溢出及除数为0等其他所有的算术错误
**9**|**SIGKILL**|**用来立即结束程序的运行，本信号不能被阻塞、处理和忽略。一般用于强制终止进程。**
14|SIGALRM|时钟定时信号，计算的是实际的时间或时钟时间。alarm函数使用该信号。
15|SIGTERM|正常结束进程的信号，kill命令的默认信号。有时如果进程已经发生问题，这个信号是不能正常终止进程的，我们才会尝试SIGKILL信号。
18|SIGCONT|该信号可以让暂停的进程恢复执行，本信号不能被阻断。
19|SIGSTOP|该信号可以暂停前台进程，相当于输入ctrl+z快捷键。本信号不能被阻断。

### 3. 重启进程

    kill -1 2235

### 4. 强制杀死进程

    kill -9 2236


## 二、`killall` 命令

    按照进程名杀死进程

### 1. 命令格式

    killall [选项] [信号] 进程名

### 2. 选项

> `-i`：交互式，询问是否要杀死某个进程
> `-I`：忽略进程名的大小写

## 三、`pkill` 命令

    按照进程名杀死进程

### 1. 命令格式

    killall [选项] [信号] 进程名

### 2. 选项

> `-t` 终端号：按照终端号踢出用户

### 3. 按照终端号踢出用户

* `w`
使用 `w` 命令查询本机已经登录的用户

* `pkill -9 -t pst/1`
强制杀死从 `pst/1` 虚拟终端登录的进程