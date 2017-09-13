title: Linux 环境变量
tags: Linux,文件权限
categories: Linux
---
## Linux文件权限

### Linux 安全性

Linux允许用户和组根据每个文件和目录的安全性设置来访问文件;

Linux安全系统的核心是用户账户.每个能进入 Linux 系统的用户都会被分配唯一的用户账户;用户对系统中各个对象的访问取决于他们登录系统时用的账户.

1. /etc/passwd 文件

Linux 使用`/etc/passwd`文件来将用户的登录名匹配到对应的 UID 值;文件部分信息如下

```
lvyun@yunlyz-mbp ~ $ cat /etc/passwd
##
# User Database
#
# Note that this file is consulted directly only when the system is running
# in single-user mode.  At other times this information is provided by
# Open Directory.
#
# See the opendirectoryd(8) man page for additional information about
# Open Directory.
##
nobody:*:-2:-2:Unprivileged User:/var/empty:/usr/bin/false
root:*:0:0:System Administrator:/var/root:/bin/sh
daemon:*:1:1:System Services:/var/root:/usr/bin/false
```

`/etc/passwd`文件包含了以下信息:

	1. 用户登录名
	2. 用户密码
	3. 用户账户的 UID
	4. 用户账户的组 ID(GID)
	5. 用户账户的文本描述
	6. 用户 HOME 目录的位置
	7. 用户默认的 shell

为了保证安全,文件中的密码都被设置成了`*`,但这并不说明所有用户的密码都相同.

root 用户账户是 Linux 系统管理员,固定分配给它的 UID 是`0`;

运行在 Linux 服务器后台的几乎所有服务都有自己的用户账户;这样做的目的在于,及时有人攻入某个服务,也无法访问整个系统.

Linux 为系统预留了500以下的 UID 值;

2. /etc/shadow 文件

/etc/shadow 保存了 Linux 用户账户密码;只有 root 用户才能访问/ etc/shadow 文件.

```
[root@yunlyz-tencent ~]# cat /etc/shadow
root:$1$2msPQtx7$i3n7.qer4xu/RnbfYOZId/:17326:0:99999:7:::
```

`/etc/shadow` 每条记录中都有9个字段:

```
1. 与/etc/passwd文件中登录名对应字段的登录名
2. 加密后的密码
3. 自上次修改密码后过去的加密天数
4. 多少天后可以修改密码
5. 多少天后必须更改密码
6. 密码过期前提前多少天提醒用户
7. 密码过期后多少天禁止用户账户
8. 用户账户被禁止日期
9. 预留字段以备将来使用
```

### 添加新用户

用来向 Linux 添加新用户的主要工具是 `useradd`.这个命令可以一次性创建新用户账户以及设置用户 HOME 目录结构.系统默认值被设置在`/etc/default/useradd`中; 可以使用 -D 查看 Linux 系统中的这些默认值.

```
[root@yunlyz-tencent ~]# useradd -D
GROUP=100				// 新用户会被添加到 GID 为100的公共组
HOME=/home			// 新用户的 HOME 目录会位于/home/loginname
INACTIVE=-1			// 新用户密码过期后不会被禁用
EXPIRE=				// 新用户账户没有设置过期时间
SHELL=/bin/bash		// 将 bash shell 作为默认 shell
SKEL=/etc/skel		// 将/etc/skel目录下的内容复制到 HOME 目录下
CREATE_MAIL_SPOOL=yes	// 系统为改用户在 mail 目录下创建一个用户接受邮件的目录
```

`/etc/skel` 目录下都是一些启动文件:

```
[root@yunlyz-tencent ~]ls -al /etc/skel
total 32
drwxr-xr-x.  3 root root  4096 Jul 18 18:42 .
drwxr-xr-x. 91 root root 12288 Jul 24 23:44 ..
-rw-r--r--   1 root root    18 Dec  7  2016 .bash_logout
-rw-r--r--   1 root root   193 Dec  7  2016 .bash_profile
-rw-r--r--   1 root root   231 Dec  7  2016 .bashrc
drwxr-xr-x   4 root root  4096 Jul  4 20:26 .mozilla
```

`useradd`参数:

```
-c comment		给新用户添加备注
-d home_dir		为主目录指定一个名词(如果不想使用登录名作为主目录名)
-e expire_date	用 YYYY-MM-DD 指定用户过期日期
-f inactive_date	 指定用户账户多少天后禁用
-g initial_group	指定用户登录组 GID 或者组名
-G group			指定用户登录组之外所属的一个或者多个附加组
-k					必须与-m一起使用,将/etc/skel 目录内容复制到用户的 HOME 目录
-m					创建用户 HOME 目录
-M					不创建用户 HOME 目录
-n					创建一个与用户名相同的新组
-r					创建系统账户
-p	passwd			为用户指定默认密码
-s shell			指定默认 shell
-u	uid				 为账户指定唯一 UID
```

如果你想修改新用户默认系统设置,可以使用如下设置:

```
[root@yunlyz-tencent ~]# useradd -D -s /bin/test
[root@yunlyz-tencent ~]# useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/test
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
```

这样创建的新用户默认使用的 shell 就是为`/bin/test`了.

### 删除用户

`userdel`命令只会删除`/etc/passwd`文件中的用户信息,而不会删除系统中属于该账户的任何文件;

加上`-r`,`userdel`会删除用户的 HOME 目录以及邮件目录

### 修改用户

Linux 提供不同工具来修改用户账户信息.

```
usermod	修改用户账户的字段,还可以指定主要组以及附加组的所属关系
passwd	 修改已有用户的密码
chpasswd	 从文件读取登录名和密码对,并更新密码
chage	 修改密码过期时间
chfn	 修改用户账户的备注信息
chsh	 修改用户账户的默认登录 shell
```
#### 1. usermod

usermod 是用户账户修改工具中最强大的一个;能用来修改/etc/passwd文件中的大部分字段,只需要加上对应参数(`-c`修改用户账户备注, `-e`修改用户账户过期时间, `-p`修改用户账户密码, `-g`修改默认的登录组)

```
-U	解除锁定,是用户能够登录
-L	锁定用户,无法登录
```

#### 2. passwd 和 chpasswd

更改用户密码简便方法`passwd`:

```
[root@yunlyz-tencent ~]# passwd test
Changing password for user test.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

如果需要为系统中大量用户修改密码,`chpasswd`命令可以事半功倍;`chpasswd`能从标准输入自动读取登录名和密码对(冒号分割),给密码加密,然后为账户设置;可以用重定向命令将含有 userID:passwd 对的文件重定向给该命令.

```
[root@yunlyz-tencent ~]# chpasswd < passwd.txt
[root@yunlyz-tencent ~]#
```

#### 3. chsh,chcn 和 chage

```
[root@yunlyz-tencent ~]# chsh -s /bin/csh test
Changing shell for test.
Shell changed.
```
### Linux 组
