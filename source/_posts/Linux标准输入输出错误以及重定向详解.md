---
title: Linux标准输入输出错误以及重定向详解
date: 2018-10-15 22:35:58
tags:
    - Linux
catgeoires:
    - Linux
comments: true
---

## 什么是标准输入(stdin)、标准输出(stdout)、标准错误(stderr)？

Linux中，在创建进程时，会默认创建三个文件描述法，fd为0、1、2，分别对应标准输入、标准输出、标准错误。

在Linux中一切皆文件，每当我们打开一个文件，就创建一个文件描述符，fd就会+1.

1. 标准输入就是从键盘读取、程序让用户输入的信息；
2. 标准输出指的是在命令行下，终端打印出来的信息，和反馈；
3. 标准错误与标准输出相差不多，只不过错误是程序出错时反馈的内容；

标准输入没什么好说的，就是读取用户从终端输入的信息。但标准输出和标准错误之间在终端上是很难看出有什么区别。

但是标出输出和标准错误有什么区别呢？


## 重定向

重定向就是把输出存储或者输出到另外一个地方。

下面是一些常用的重定向方式：

### 输入重定向

在没有使用重定向时，终端的输出如下：
```shell
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd                  // stdin 0
find: ‘/etc/dhcp’: Permission denied                          // stderr 2
/etc/passwd                                                   // stdout 1
find: ‘/etc/ntp/crypto’: Permission denied
find: ‘/etc/sudoers.d’: Permission denied
find: ‘/etc/selinux/final’: Permission denied
find: ‘/etc/selinux/targeted/active’: Permission denied
/etc/pam.d/passwd
find: ‘/etc/audisp’: Permission denied
```
会把所有的反馈都打印到屏幕上（标准输出和标准错误）

1. >filename
```shell
// 将stdout重定向到文件filename，如果文件不存在则创建，存在则覆盖。

[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd >find.out
find: ‘/etc/dhcp’: Permission denied
find: ‘/etc/ntp/crypto’: Permission denied
find: ‘/etc/sudoers.d’: Permission denied
find: ‘/etc/selinux/final’: Permission denied
find: ‘/etc/selinux/targeted/active’: Permission denied
find: ‘/etc/audisp’: Permission denied

[lvyun@lvyun-cn ~]$ cat find.out
/etc/passwd
/etc/pam.d/passwd
```
2. 1>filename
```shell
// 效果同上， > 默认输入为1
```

3. 2>filename
```shell
// 将stderr重定向到filename，如果文件不存在则创建，存在则覆盖。
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd 2>find.out.2
/etc/passwd
/etc/pam.d/passwd

[lvyun@lvyun-cn ~]$ cat find.out.2
find: ‘/etc/dhcp’: Permission denied
find: ‘/etc/ntp/crypto’: Permission denied
find: ‘/etc/sudoers.d’: Permission denied
find: ‘/etc/selinux/final’: Permission denied
find: ‘/etc/selinux/targeted/active’: Permission denied
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
```

4. &>filename
```shell
// 将stdout、stderr重定向到文件filename，如果文件不存在则创建，存在则覆盖。
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd  &>find.out

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
```

5. M>N
```shell
// 将M(stdout、stderr)输出到文件N，如果重定向的右边没有 & 符号，则表示文件
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd  2>1
/etc/passwd
/etc/pam.d/passwd

[lvyun@lvyun-cn ~]$ cat 1
find: ‘/etc/dhcp’: Permission denied
find: ‘/etc/ntp/crypto’: Permission denied
find: ‘/etc/sudoers.d’: Permission denied
find: ‘/etc/selinux/final’: Permission denied
find: ‘/etc/selinux/targeted/active’: Permission denied
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
```

6. M>&N
```shell
// 将M重定向绑定到N，例如 2>&1，将stderr重定向绑定到stdout
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd  2>&1
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

// 这里看着是和没有使用重定向的时候屏幕显示的是一样的，但是其实是有很大的差别的。
```

### 高级用法
1. >filename 2>&1
```shell
// 将 stdout 重定向到文件 filename 中，stderr 被重定向到当前stdout 的位置
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd >find.out  2>&1
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
```

2. 2>&1 >filename
```shell
// 将 stderr 重定向到当前 stdout 的位置，然后将 stdout 重定向到文件filename
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd 2>&1 >find.out
find: ‘/etc/dhcp’: Permission denied
find: ‘/etc/ntp/crypto’: Permission denied
find: ‘/etc/sudoers.d’: Permission denied
find: ‘/etc/selinux/final’: Permission denied
find: ‘/etc/selinux/targeted/active’: Permission denied
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

4. >/dev/null
/dev/null代表linux的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。那么执行了>/dev/null之后，标准输出就会不再存在，没有任何地方能够找到输出的内容。
```shell
[lvyun@lvyun-cn ~]$ find  /etc/ -name passwd >/dev/null
find: ‘/etc/dhcp’: Permission denied
find: ‘/etc/ntp/crypto’: Permission denied
find: ‘/etc/sudoers.d’: Permission denied
find: ‘/etc/selinux/final’: Permission denied
find: ‘/etc/selinux/targeted/active’: Permission denied
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
```

4. nohup结合
我们经常会用`nohup command &`的组合方式来启动一个后台任务
```shell
nohup php filename.php &
```

为了不让执行过程中输出的日志或者错误打印到前台（屏幕），我们就需要用到重定向绑定。
```shell
nohup php filename.php >filename.out 2>&1 &

// 或者是
nohup php filename.php >、dev/null 2>&1 &
```
