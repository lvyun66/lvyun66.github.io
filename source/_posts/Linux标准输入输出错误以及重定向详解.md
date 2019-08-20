---
title: Linux标准输入输出错误以及重定向详解
date: 2018-10-15 22:35:58
tags:
    - Linux
catgeoires:
    - Linux
comments: true
---

这两天在用`Swoole`写一个简单的聊天室的时候，需要从标准输入读取数据，并且输出到屏幕，在用的时候对这些概念有些模糊，因此特地去查找了一下资料。
<!-- more -->
## 什么是标准输入(stdin)、标准输出(stdout)、标准错误(stderr)？

Linux中，在创建进程时，会默认创建三个文件描述法，fd为0、1、2，分别对应标准输入、标准输出、标准错误。

在Linux中一切皆文件，每当我们打开一个文件，就创建一个文件描述符，fd就会+1.

1. 标准输入就是从键盘读取、程序让用户输入的信息；
2. 标准输出指的是在命令行下，终端打印出来的信息，和反馈；
3. 标准错误与标准输出相差不多，只不过错误是程序出错时反馈的内容；

标准输入没什么好说的，就是读取用户从终端输入的信息。但标准输出和标准错误之间在终端上是很难看出有什么区别。

## 标出输出和标准错误区别

`stderr`是没有缓冲的，会立即输出；

`stdout`默认是行缓冲，也就是遇到`\n`才会向外输出内容。
```c
#include<stdio.h>

int main(int argc, char const *argv[]) {
    fprintf(stdout, "%s", "Hello ");
    fprintf(stderr, "%s", "World!");
    return 0;
}

// 10:47:39 › gcc -c test.c
// 10:47:46 › gcc test.o -o test
// 10:47:54 › ./test
// World!Hello %            结果
```

要想`stdout`实时输出，可以加上`\n`或者是在输出语句后加上`fflush(stdout);`
```c
#include <stdio.h>

int main(int argc, char const *argv[]) {
    fprintf(stderr, "%s\n", "Hello ");
    fprintf(stderr, "%s\n", "World!");
    return 0;
}
// 10:48:39 › gcc -c test.c
// 10:48:46 › gcc test.o -o test
// 10:48:54 › ./test
// Hello                结果
// World!

int main(int argc, char const *argv[]) {
    fprintf(stderr, "%s", "Hello ");
    fflush(stdout);
    fprintf(stderr, "%s", "World!");
    return 0;
}
// 10:53:34 › gcc -c test.c
// 10:53:40 › gcc test.o -o test
// 10:53:47 › ./test
// Hello World!%
```

## 重定向

重定向就是把输出存储或者输出到另外一个地方。

下面是一些常用的重定向方式：

### 输入重定向

在没有使用重定向时，终端的输出如下：
```shell
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd                  // stdin 0
find: '/etc/dhcp': Permission denied                          // stderr 2
/etc/passwd                                                   // stdout 1
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
/etc/pam.d/passwd
find: '/etc/audisp': Permission denied
```
会把所有的反馈都打印到屏幕上（标准输出和标准错误）

1. command >filename
```shell
// 将stdout重定向到文件filename，如果文件不存在则创建，存在则覆盖。

[lvyun@lvyun-cn ~]$ find /etc/ -name passwd >find.out
find: '/etc/dhcp': Permission denied
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
find: '/etc/audisp': Permission denied

[lvyun@lvyun-cn ~]$ cat find.out
/etc/passwd
/etc/pam.d/passwd
```
2. command 1>filename
```shell
// 效果同上， > 默认输入为1
```

3. command 2>filename
```shell
// 将stderr重定向到filename，如果文件不存在则创建，存在则覆盖。
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd 2>find.out.2
/etc/passwd
/etc/pam.d/passwd

[lvyun@lvyun-cn ~]$ cat find.out.2
find: '/etc/dhcp': Permission denied
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied
```

4. command &>filename
```shell
// 将stdout、stderr重定向到文件filename，如果文件不存在则创建，存在则覆盖。
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd  &>find.out

[lvyun@lvyun-cn ~]$ cat find.out
find: '/etc/dhcp': Permission denied
/etc/passwd
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
/etc/pam.d/passwd
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied
```

5. command M>N
```shell
// 将M(stdout、stderr)输出到文件N，如果重定向的右边没有 & 符号，则表示文件
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd  2>1
/etc/passwd
/etc/pam.d/passwd

[lvyun@lvyun-cn ~]$ cat 1
find: '/etc/dhcp': Permission denied
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied
```

6. command M>&N
```shell
// 将M重定向绑定到N，例如 2>&1，将stderr重定向绑定到stdout
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd  2>&1
find: '/etc/dhcp': Permission denied
/etc/passwd
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
/etc/pam.d/passwd
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied

// 这里看着是和没有使用重定向的时候屏幕显示的是一样的，但是其实是有很大的差别的。
```

7. 附上`stackoverflow`上`Byte Commander`的一个[回答](https://askubuntu.com/questions/420981/how-do-i-save-terminal-output-to-a-file)

![image](https://i.loli.net/2018/10/18/5bc7e697c132e.png)

### 高级用法
1. command >filename 2>&1
```shell
// 将 stdout 重定向到文件 filename 中，stderr 被重定向到当前stdout 的位置
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd >find.out  2>&1

[lvyun@lvyun-cn ~]$ cat find.out
find: '/etc/dhcp': Permission denied
/etc/passwd
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
/etc/pam.d/passwd
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied
```

2. command 2>&1 >filename
```shell
// 将 stderr 重定向到当前 stdout 的位置，然后将 stdout 重定向到文件filename
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd 2>&1 >find.out
find: '/etc/dhcp': Permission denied
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied

[lvyun@lvyun-cn ~]$ cat find.out
/etc/passwd
/etc/pam.d/passwd
```

3. 两者的区别
```shell
// 我们可以通过下面的表达式来理解两者区别

// 前提：
stdin = 0
stdout = 1
stderr = 2
filename = 3

// >filename 2>&1
stdout = filename  // stdout = 3
stderr = stdout    // stderr = 3

// 2>&1 >filename
stderr = stdout    // stderr = 1
stdout = filename  // stdout = 3

这就是他们之间的区别，也就是为什么 >filename 2>&1 会把标准输出、标准错误全部重定向文件filename中。
```

4. command >/dev/null
/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。那么执行了>/dev/null之后，标准输出就会不再存在，没有任何地方能够找到输出的内容。
```shell
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd >/dev/null
find: '/etc/dhcp': Permission denied
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied
```

5. nohup结合
我们经常会用`nohup command &`的组合方式来启动一个后台任务
```shell
nohup php filename.php &
```

为了不让执行过程中输出的日志或者错误打印到前台（屏幕），我们就需要用到重定向绑定。
```shell
nohup php filename.php >filename.out 2>&1 &

// 或者是
nohup php filename.php >/dev/null 2>&1 &
```

## command >filename 2>&1 与 command >filename 2>filename

为什么要用重定向绑定，而不用`command >filename 2>filename`这种方式呢？我们想先看下这两者的输出结果区别。

```shell
// command >filename 2>&1
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd >find.out 2>&1
[lvyun@lvyun-cn ~]$ cat find.out
find: ‘/etc/dhcp’: Permission denied
/etc/passwd
find: ‘/etc/ntp/crypto’: Permission denied
find: ‘/etc/sudoers.d’: Permission denied
find: ‘/etc/selinux/final’: Permission denied
find: ‘/etc/selinux/targeted/active’: Permission denied
/etc/pam.d/passwd
find: ‘/etc/audisp’: Permission denied
find: ‘/etc/polkit-1/rules.d’: Permission denied
find: ‘/etc/polkit-1/localauthority’: Permission denied
find: ‘/etc/pki/rsyslog’: Permission denied
find: ‘/etc/pki/CA/private’: Permission denied
find: ‘/etc/docker’: Permission denied
find: ‘/etc/grub.d’: Permission denied
find: ‘/etc/audit’: Permission denied
find: ‘/etc/lvm/backup’: Permission denied
find: ‘/etc/lvm/archive’: Permission denied
find: ‘/etc/lvm/cache’: Permission denied
find: ‘/etc/firewalld’: Permission denied

// command >filename 2>filename
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd 2>find.out 1>find.out
[lvyun@lvyun-cn ~]$ cat find.out
/etc/passwd
/etc/pam.d/passwd
ion denied                                                              // 这段错误被截断
find: '/etc/ntp/crypto': Permission denied
find: '/etc/sudoers.d': Permission denied
find: '/etc/selinux/final': Permission denied
find: '/etc/selinux/targeted/active': Permission denied
find: '/etc/audisp': Permission denied
find: '/etc/polkit-1/rules.d': Permission denied
find: '/etc/polkit-1/localauthority': Permission denied
find: '/etc/pki/rsyslog': Permission denied
find: '/etc/pki/CA/private': Permission denied
find: '/etc/docker': Permission denied
find: '/etc/grub.d': Permission denied
find: '/etc/audit': Permission denied
find: '/etc/lvm/backup': Permission denied
find: '/etc/lvm/archive': Permission denied
find: '/etc/lvm/cache': Permission denied
find: '/etc/firewalld': Permission denied
```

因为`command >filename 2>filename`标准输出和标准错误会同时抢占filename的管道，所以导致在输出内容的时候会出现缺失、覆盖或者是乱码等现象。因此这种写法最后的结果是无法预估的。

`command >filename 2>filename`不像`command >filename 2>&1`共用filename文件的管道，文件会被打开两次，这也就是为什么会出现抢占文件管道的问题了。
