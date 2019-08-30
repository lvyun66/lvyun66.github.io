---
title: Linux 环境变量
date: 2017-09-13 19:43:47
tags:
- Linux
- 环境变量
categories:
- Linux
comments: true
---

## Linux 环境变量

### 设置用户自定义变量

1. 设置局部用户自定义变量
<!-- more -->
	```
	lvyun@yunlyz-mbp ~ $ echo $my_variable

	lvyun@yunlyz-mbp ~ $ my_variable=Hello
	lvyun@yunlyz-mbp ~ $ echo $my_variable
	Hello
	```

	如果设置含有空格的字符串,需要使用引号来界定字符的开始和结尾.没有引号, bash shell 会以为下一个词是另一个要执行的命令;

	```
	lvyun@yunlyz-mbp ~ $ my_variable=Hello World
	-bash: World: command not found
	lvyun@yunlyz-mbp ~ $ my_variable="Hello World"
	lvyun@yunlyz-mbp ~ $ echo $my_variable
	Hello World
	```

	当前方法设置的变量只能在当前进程中使用,如果另外一个 shell 进程则不可使用

	*注:所有的环境变量都使用大写字母;如果是自定义的局部变量或者 shell 脚本,则建议使用小写;这能够避免重定义系统环境变量可能带来的灾难.*


2. 设置全局环境变量

	设置全局环境变量可以使用 export 这个命令来完成;

	```
	lvyun@yunlyz-mbp ~ $ my_variable="Hello World"
	lvyun@yunlyz-mbp ~ $ export my_variable
	lvyun@yunlyz-mbp ~ $ echo $my_variable
	Hello World
	```

	export 创建的环境变量是全局变量,能够在子进程中使用:

	```
	lvyun@yunlyz-mbp ~ $ my_variable="Hello World"
	lvyun@yunlyz-mbp ~ $ export my_variable
	lvyun@yunlyz-mbp ~ $ echo $my_variable
	Hello World
	lvyun@yunlyz-mbp ~ $ bash
	lvyun@yunlyz-mbp ~ $ echo $my_variable
	Hello World
	lvyun@yunlyz-mbp ~ $ exit
	exit
	lvyun@yunlyz-mbp ~ $ echo $my_variable
	Hello World
	```
	** *特别说明:在使用 export 创建全局环境变量导入的变量不需要加$**

	** *注:如果在子进程中修改了改环境变量,那么这种修改只在子进程中有效,父进程中的值不受影响; 甚至子 shell 无法通过 export 来改变父 shell 中的全局环境变量.**

### 删除环境变量

#### 1. unset
```
lvyun@yunlyz-mbp ~ $ unset my_variable
lvyun@yunlyz-mbp ~ $ echo $my_variable
```
如果在子 shell 中删除了环境变量,父 shell 的环境变量依然可用;

### 设置 PATH 环境变量

当你在 shell 输入一个外部命令时, shell 必须搜索系统来找到对应的程序; Path 环境变量定义了用于进行命令和程序查找的目录;

```
lvyun@yunlyz-mbp ~ $ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:
/Users/lvyun/tool/bin:/Users/lvyun/.composer/vendor/bin:
/Users/lvyun/pear/bin
```

**注意: PATH 环境变量中的目录使用冒号(:)分割;**

如果命令或者程序的位置没有在 PATH 中,而没有使用绝对路径访问, shell 是没有办法找到命令的;会提示命令没有找到;

```
lvyun@yunlyz-mbp ~ $ myCommand
-bash: myCommand: command not found
```

如何设置 PATH 环境变量,只需要引用原来的 PATH 值,然后再给这个字符串添加新目录即可.

```
lvyun@yunlyz-mbp ~ $ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/Users/lvyun/tool/bin:/Users/lvyun/.composer/vendor/bin:/Users/lvyun/pear/bin
lvyun@yunlyz-mbp ~ $ PATH=$PATH:/Users/lvyun/bin
lvyun@yunlyz-mbp ~ $ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin:/Users/lvyun/tool/bin:/Users/lvyun/.composer/vendor/bin:/Users/lvyun/pear/bin:/Users/lvyun/bin
```

你会发现在 PATH 的最后面已经加上了路径,现在你可以目录结构中的任何位置执行程序;如果我们重启了机器,你会发现命令已经不能使用,要想命令持久化,可以把环境变量保存在$ HOME/.bashrc或者 $HOME/.bash_profile 中;

```
vim /Users/lvyun/.bashrc
PATH=$PATH:/Users/lvyun/bin
```

### 定位系统环境变量

当你登入 Linux 系统启动一个 bash shell 时,默认情况下 bash 会在几个文件中查找命令.这些文件称为启动文件或环境文件;

启动 bash shell 有三种方式:

	1. 登录时作为默认登录 shell
	2. 作为非登录 shell 的交互式 shell
	3. 作为运行脚本的非交互 shell

#### 1.登录 shell

当你登录 Linux,bash shell 会作为登录 shell 启动. 登录 shell 会从5个不同的启动文件里读取命令:
```
1. /etc/profile
2. $HOME/.bash_profile
3. $HOME/.bashrc
4. $HOME/.bash_login
5. $HOME/.profile

lvyun@yunlyz-mbp ~ $ ls -al | grep bash
-rw-r--r--   1 lvyun  staff   7706  7 24 00:01 .bash_history
-rw-r--r--@  1 lvyun  staff    832  7 16 09:01 .bash_profile

/etc/profile 是系统默认的 bash shell 启动文件,没有用户登录时都会加载这个启动文件

mac os 中/etc/profile是这个样子的,不同的系统可能会是不同的代码:

lvyun@yunlyz-mbp ~ $ cat /etc/profile
# System-wide .profile for sh(1)

if [ -x /usr/libexec/path_helper ]; then
       	eval `/usr/libexec/path_helper -s`
fi

if [ "${BASH-no}" != "no" ]; then
       	[ -r /etc/bashrc ] && . /etc/bashrc
fi


centos:

[root@yunlyz-tencent ~]# cat /etc/profile
# /etc/profile

# System wide environment and startup programs, for login setup
# Functions and aliases go in /etc/bashrc

# It's NOT a good idea to change this file unless you know what you
# are doing. It's much better to create a custom.sh shell script in
# /etc/profile.d/ to make custom changes to your environment, as this
# will prevent the need for merging in future updates.

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
        EUID=`id -u`
        UID=`id -ru`
    fi
    USER="`id -un`"
    LOGNAME=$USER
    MAIL="/var/spool/mail/$USER"
fi

# Path manipulation
if [ "$EUID" = "0" ]; then
    pathmunge /usr/sbin
    pathmunge /usr/local/sbin
else
    pathmunge /usr/local/sbin after
    pathmunge /usr/sbin after
fi

HOSTNAME=`/usr/bin/hostname 2>/dev/null`
HISTTIMEFORMAT='%F %T '
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi

export PATH USER LOGNAME MAIL HOSTNAME HISTSIZE HISTCONTROL

# By default, we want umask to get set. This sets it for login shell
# Current threshold for system reserved uid/gids is 200
# You could check uidgid reservation validity in
# /usr/share/doc/setup-*/uidgid file
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
    umask 002
else
    umask 022
fi

for i in /etc/profile.d/*.sh ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done

unset i
unset -f pathmunge
export HISTTIMEFORMAT
```
centos中会迭代/etc/profile.d目录下的所有文件,文件如下:

```
[root@yunlyz-tencent ~]# ls -al /etc/profile.d
total 88
drwxr-xr-x.  2 root root  4096 Jul 18 18:43 .
drwxr-xr-x. 91 root root 12288 Jul 24 23:44 ..
-rw-r--r--   1 root root   771 Apr 13 04:24 256term.csh
-rw-r--r--   1 root root   841 Apr 13 04:24 256term.sh
-rw-r--r--   1 root root  1348 Dec  2  2016 abrt-console-notification.sh
-rw-r--r--.  1 root root   660 Jun 10  2014 bash_completion.sh
-rw-r--r--.  1 root root   196 Apr 29  2015 colorgrep.csh
-rw-r--r--.  1 root root   201 Apr 29  2015 colorgrep.sh
-rw-r--r--   1 root root  1741 Nov  5  2016 colorls.csh
-rw-r--r--   1 root root  1606 Nov  5  2016 colorls.sh
-rw-r--r--   1 root root  1706 Apr 13 04:24 lang.csh
-rw-r--r--   1 root root  2703 Apr 13 04:24 lang.sh
-rw-r--r--.  1 root root   123 Jul 31  2015 less.csh
-rw-r--r--.  1 root root   121 Jul 31  2015 less.sh
-rw-r--r--   1 root root   313 Jun 30  2012 qt-graphicssystem.csh
-rw-r--r--   1 root root   379 Jun 13  2012 qt-graphicssystem.sh
-rw-r--r--   1 root root   105 Dec 22  2016 vim.csh
-rw-r--r--   1 root root   269 Dec 22  2016 vim.sh
-rw-r--r--.  1 root root   164 Jan 28  2014 which2.csh
-rw-r--r--.  1 root root   169 Jan 28  2014 which2.sh
```
我们会发现,有些文件与系统特定程序有关;大部分应用都会创建两个启动文件, .sh 是 bash shell 使用, .csh 是供 c shell 使用的.

$HOME 目录下面的启动文件

* $HOME/.bash_profile
* $HOME/.bashrc
* $HOME/.bash_login
* $HOME/.profile

大多数的 Linux 发行版只用到1-2两个启动文件;会依次按照顺序执行.

#### 2. 交互式 shell 进程

什么叫交互式 shell?如果你的 shell 不是在登录时启动的,那么你启动的 shell 叫做交互式 shell.

如果 bash 作为`交互式 shell`启动,那么不会读取/etc/profile文件,只会检查`$HOME`目录下面的 `.bash_profile` 或者 `.bashrc`.

```
[lvyun@yunlyz-tencent ~]$ cat .bashrc
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
       	. /etc/bashrc
fi

for file in /etc/bash_completion.d/*; do
    source $file
done

# Uncomment the following line if you don't like systemctl's auto-paging feature:
# export SYSTEMD_PAGER=

# User specific aliases and functions
```

`.bashrc`文件有两个作用:

1. 查看`/etc`下通用 bashrc;在这里还可以看到还把bash_completion.d 命令自动补全这些文件给引入了;
2. 为用户提供一个定制的命令别名;

#### 3. 非交互式 shell

### 数组变量

数组是能够存储多个值得`变量`

```
lvyun@yunlyz-mbp ~ $ mytest=(one two three)
lvyun@yunlyz-mbp ~ $ echo $mytest
one
lvyun@yunlyz-mbp ~ $ echo ${mytest[2]}
three
// 显示全部值
lvyun@yunlyz-mbp ~ $ echo ${mytest[*]}
one two three
// 改变某个索引值
lvyun@yunlyz-mbp ~ $ mytest[2]=lvyun
lvyun@yunlyz-mbp ~ $ echo ${mytest[*]}
one two lvyun
// 删除某个索引
lvyun@yunlyz-mbp ~ $ unset mytest[2]
lvyun@yunlyz-mbp ~ $ echo ${mytest[*]}
one two
```

### 技巧

#### 技巧一

关于涉及环境变量名时,什么时候该使用$,什么时候不该使用$?

如果要用到环境变量,使用$;
如果要操作环境变量,不使用$;
但是,在使用 printenv 时例外,不要需要使用$

```
lvyun@yunlyz-mbp ~ $ printenv HOME
/Users/lvyun
```

#### 技巧二

另外需要注意,变量名,等号和值之间没有空格;
如果有空格, bash shell 会把值当成一个单独的命令:

```
lvyun@yunlyz-mbp ~ $ my_variable = "Hello World"
-bash: my_variable: command not found
```
