Linux Shell编程（4）- 环境变量配置文件
---

## 一、环境变量配置文件简介

### 1. 环境变量的作用

    环境变量配置文件主要是定义对系统操作环境生效的系统默认环境变量，如PATH、HISTSIZE、PS1、HOSTNAME等。

### 2. source 命令

> 修改配置文件后，注销重新登录之后才会生效，使用source命令可以不用重新登录，令配置文件生效。

#### **语法**

* `source 配置文件`
或
* `. 配置文件`

#### **实例**

```
[root~]# source .bashrc
[root~]# . .bashrc
```

### 3. 主要的环境变量配置文件

> /etc/profile
> /etc/profile.d/*.sh
> ~/.bash_profile
> ~/.bashrc
> /etc/bashrc

### 4. 环境变量配置文件加载顺序

#### **正常登录**
```
`/etc/profile` ——> `~/.bash_profile` ——> `~/.bashrc` ——> `/etc/bashrc` ——> 命令提示符
    |
    |——> `/etc/profile.d/*.sh` ——> `/etc/profile.d/lang.sh` ——> `/etc/locale.conf`
```

#### **非正常登录（使用 `su` 命令切换用户）**

```
`/etc/bashrc` ——> 命令提示符
    |
    |——> `/etc/profile.d/*.sh` ——> `/etc/profile.d/lang.sh` ——> `/etc/locale.conf`
```

## 二、环境变量配置文件功能

### 1. `/etc/profile` 文件的作用

> USER 变量
> LOGNAME 变量
> MAIL 变量
> PATH 变量
> HOSTNAME 变量
> HISTSIZE 变量
> umask
> 遍历调用 `/etc/profile.d/*.sh` 文件

#### **实例**

* `/etc/profile` 文件，省略了部分内容
```
# /etc/profile

# PATH 变量
pathmunge () {
    case ":${PATH}:" in
        *:"$1":*)
            ;;
        *)
            if [ "$2" = "after" ] ; then
                PATH=$PATH:$1
            else
                PATH=$1:$PATH
            fi
    esac
}

if [ -x /usr/bin/id ]; then
    if [ -z "$EUID" ]; then
        # ksh workaround
        EUID=`/usr/bin/id -u`
        UID=`/usr/bin/id -ru`
    fi
    # USER 变量
    USER="`/usr/bin/id -un`"
    # LOGNAME 变量
    LOGNAME=$USER
    # MAIL 变量
    MAIL="/var/spool/mail/$USER"
fi

# HOSTNAME 变量
HOSTNAME=`/usr/bin/hostname 2>/dev/null`
# HISTSIZE 变量
HISTSIZE=1000

# 声明为环境变量
export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

# /usr/share/doc/setup-*/uidgid file
# 定义 umask
if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
    umask 002
else
    umask 022
fi

# 遍历调用 `/etc/profile.d/*.sh` 文件
for i in /etc/profile.d/*.sh /etc/profile.d/sh.local ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done
```

### 2. `/etc/bashrc` 文件的作用

> PS1 变量
> PATH 变量
> umask
> 遍历调用 `/etc/profile.d/*.sh` 文件

#### **实例**

* `/etc/bashrc` 文件，省略了部分内容
```
# /etc/bashrc

# are we an interactive shell?
# PS1 变量
if [ "$PS1" ]; then
  if [ -z "$PROMPT_COMMAND" ]; then
    case $TERM in
    xterm*|vte*)
      if [ -e /etc/sysconfig/bash-prompt-xterm ]; then
          PROMPT_COMMAND=/etc/sysconfig/bash-prompt-xterm
      elif [ "${VTE_VERSION:-0}" -ge 3405 ]; then
          PROMPT_COMMAND="__vte_prompt_command"
      else
          PROMPT_COMMAND='printf "\033]0;%s@%s:%s\007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
      fi
      ;;
    screen*)
      if [ -e /etc/sysconfig/bash-prompt-screen ]; then
          PROMPT_COMMAND=/etc/sysconfig/bash-prompt-screen
      else
          PROMPT_COMMAND='printf "\033k%s@%s:%s\033\\" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
      fi
      ;;
    *)
      [ -e /etc/sysconfig/bash-prompt-default ] && PROMPT_COMMAND=/etc/sysconfig/bash-prompt-default
      ;;
    esac
  fi
  # Turn on parallel history
  shopt -s histappend
  history -a
  # Turn on checkwinsize
  shopt -s checkwinsize
  [ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\h \W]\\$ "
fi

# ----------只有非正常登录的shell才会执行下面脚本----------------
if ! shopt -q login_shell ; then # We're not a login shell
    # Need to redefine pathmunge, it get's undefined at the end of /etc/profile
    # PATH 变量
    pathmunge () {
        case ":${PATH}:" in
            *:"$1":*)
                ;;
            *)
                if [ "$2" = "after" ] ; then
                    PATH=$PATH:$1
                else
                    PATH=$1:$PATH
                fi
        esac
    }

    # /usr/share/doc/setup-*/uidgid file
    # umask
    if [ $UID -gt 199 ] && [ "`/usr/bin/id -gn`" = "`/usr/bin/id -un`" ]; then
       umask 002
    else
       umask 022
    fi

    SHELL=/bin/bash
    # 遍历调用 `/etc/profile.d/*.sh` 文件
    for i in /etc/profile.d/*.sh; do
        if [ -r "$i" ]; then
            if [ "$PS1" ]; then
                . "$i"
            else
                . "$i" >/dev/null
            fi
        fi
    done
fi
```

### 3. `~/.bash_profile` 文件的作用

> 调用了 `~/.bashrc` 文件
> 在PATH变量后加入 `:$HOME/bin` 这个目录

#### 实例
* `~/.bash_profile` 文件
```
# .bash_profile

# 调用了 `~/.bashrc` 文件
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# 在PATH变量后加入 `:$HOME/bin` 这个目录
PATH=$PATH:$HOME/bin

export PATH
```

### 4. `~/.bashrc` 文件的作用

> 定义别名
> 调用 `/etc/bashrc` 文件

#### 实例

* `~/.bashrc` ，省略了部分内容
```
# .bashrc

# 调用 `/etc/bashrc` 文件
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# 定义别名
alias ll='ls -AlhF --color=auto'
alias la='ls -A'
alias l='ls -CF'
alias vi='vim'
```

## 三、其他文件

### 1. 注销时生效的环境变量配置文件

    ~/.bash_logout

### 2. 历史命令存储文件

    ~/.bash_history

### 3. shell登录信息文件

> 登录时显示的欢迎信息

#### **`/etc/moted`**

    不管是本地登录，还是远程登录，都可以显示此文件内容信息。

#### **`/etc/issue.net`**

    远程终端欢迎信息

> 要显示此欢迎信息，由ssh的配置文件 `/etc/ssh/sshd_config` 决定。需要在ssh配置文件中加入"Banner /etc/issue.net" 行，并重启ssh服务才会生效。

#### **`/etc/issue`**

    本地终端欢迎信息

> 因为服务器大都采用远程登录，本地终端欢迎信息设置的意义不大。

* 本地终端欢迎信息支持转义符

转义符|作用
-|-
\\d|显示当前系统日期
\\s|显示操作系统名称
\\l|显示登录终端号
\\m|显示硬件体系结构，如i386/i686等
\\n|显示主机名
\\o|显示域名
\\r|显示内核版本
\\t|显示当前系统时间
\\u|显示当前登录用户的序号
