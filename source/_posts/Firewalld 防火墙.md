---
title: Firdwalld防火墙
date: 2017/11/29 上午9:29:24
tags:
    - Linux
    - Firewall
categories:
    - Firewall
comments: true
---

# Centos7防火墙firewalld

## 1. Firewalld介绍

`firewalld`支持动态更新和zone概念。`firewalld` 有图形界面和命令行界面，图形界面简单粗暴，但是`server`没有

`firewalld`命令行管理工具是`firewall-cmd`.

## 2. 服务相关

```
// 启动防火墙
$ systemctl start firewalld

// 关闭防火墙
$ syatemctl stop firewalld

// 重启防火墙
$ systemctl reload firewalld

// 禁用防火墙
$ syatemctl disable firewalld

// 开启防火墙
$ systemctl enable firewalld

// 查看防火墙状态
$ systemctl status firewalld
```

## 3. 客户端命令
```
firewalld 客户端管理命令是 firewall-cmd.

--state 查看防火墙状态,和 syatemctl status fiewalld一样
--reload 重启防火墙
--complate-reload 重启防火墙,两者区别在此方式会关闭防火墙
--parmenet 常驻,永久生效
--set-default-zone=zone 设置默认空间
--zone  设置空间,默认有8中方式
--add-port  增加开放的端口
--list-all 列出所有已经开放的信息
--list-ports 列出已经开放的端口
--remove-port 移除开放的端口
```

## 4. 端口管理
```
// 开放3306端口
$ firewall-cmd --permanent --zone=public --add-port=3306/tcp
success

// 开放端口之后需要重新启动防火墙, 以便于马上生效
$ firewall-cmd --reload
success

// 查看端口已经开放端口
$ firewall-cmd --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports: 3306/tcp 80/tcp
  protocols:
  masquerade: no
  forward-ports:
  sourceports:
  icmp-blocks:
  rich rules:

// 关闭端口
$ firewall-cmd --zone=public --permanent --remove-port=8080/tcp
success
$ firewall-cmd --reload
success
```

## 5. 服务管理
```
$ firewall-cmd --get-services
ssh synergy syslog syslog-tls telnet tftp tftp-client tinc

// 允许SSH服务通过
$ firewall-cmd --enable service=ssh

// 禁止SSH服务通过
$ firewall-cmd --disable service=ssh
```

## 6. 端口转发

```

```
