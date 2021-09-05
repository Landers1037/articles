---
title: CentOS测试开启bbr
name: bbr-enable
date: 2020-01-21
id: 0
tags: [linux]
categories: []
abstract: ""
---


在CentOS系统里开启BBR加速


<!--more-->


在CentOS系统里开启BBR加速

<!--more-->

# 查看内核

```bash
uname -r
```

bbr要求内核的版本高于4.9

## 查看可用内核

```bash
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```



## 更新仓库

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org

yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm -y
```

针对CentOS6，使用

```bash
yum install https://www.elrepo.org/elrepo-release-6-9.el6.elrepo.noarch.rpm -y
```

## 安装内核

```bash
yum --enablerepo=elrepo-kernel install kernel-ml -y
```

使用新的内核启动

```bash
grub2-set-default 0
```

如果该内核参数不为0，则

```bash
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg 
```

重新生成grab配置文件

### 查看是否配置成功

```bash
grub2-editenv list# saved_entry=CentOS Linux (5.12.4-1.el7.elrepo.x86_64) 7 (Core)
```

然后重启系统即可

### 启用

在内核版本高于4.9的机器上，直接修改配置文件

```bash
echo 'net.core.default_qdisc=fq' >> /etc/sysctl.confecho 'net.ipv4.tcp_congestion_control=bbr' >> /etc/sysctl.confsysctl -p
```

### 查看bbr运行状态

```bash
lsmod | grep tcp_bbr
```

