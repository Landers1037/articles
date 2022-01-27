---
title: vmware下配置NAT模式静态ip
name: vmware-nat-static
date: 2022-01-27 14:11:33
tags: [linux]
categories: [linux]
abstract: 
---
使用Vmware创建Ubuntu虚拟机时，通过`NAT`模式配置静态IP

<!--more-->

## 环境

- VMware16
- Ubuntu18.04



## 配置本机网卡

### 查询本机IP

```bash
ipconfig

C:\Users\landers>ipconfig

Windows IP 配置


以太网适配器 以太网:

   连接特定的 DNS 后缀 . . . . . . . :
   IPv4 地址 . . . . . . . . . . . . : 192.168.0.100
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 192.168.0.1

以太网适配器 VMware Network Adapter VMnet1:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::4502:2ae9:bdd4:3923%6
   IPv4 地址 . . . . . . . . . . . . : 192.168.158.1
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . :

以太网适配器 VMware Network Adapter VMnet8:

   连接特定的 DNS 后缀 . . . . . . . :
   本地链接 IPv6 地址. . . . . . . . : fe80::8471:5c53:32ba:a9f5%12
   IPv4 地址 . . . . . . . . . . . . : 192.168.100.10
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 0.0.0.0
                                       192.168.100.1
```

配置的子网网段不能和主机的网段冲突

这里选择`192.168.100.1`

首先配置`VMnet8`虚拟网卡

![](/images/vmware-nat-static-1.jpg)

这里的本机网卡网关设置为`192.168.100.1`

### 配置虚拟机网络

![](/images/vmware-nat-static-2.jpg)

进入**虚拟网络编辑器**，编辑VMnet8的网络，取消勾选DHCP分配IP地址

![](/images/vmware-nat-static-3.jpg)

配置NAT的网关地址为`192.168.100.2`

## 进入虚拟机配置虚拟机网卡

```bash
vi /etc/network/interfaces 

# ifupdown has been replaced by netplan(5) on this system.  See
# /etc/netplan for current configuration.
# To re-enable ifupdown on this system, you can run:
#    sudo apt install ifupdown
auto ens33
iface ens33 inet static
address 192.168.100.100
netmask 255.255.255.0
gateway 192.168.100.2
# dns-nameservers 223.5.5.5
dns-nameserver 8.8.8.8
```

`auto ens33`即需要自动挂载的网卡，通过ifconfig查看并填写对应的网卡

配置`address`，`netmask`和`gateway`

子网掩码和网关需要和虚拟网络编辑器中的保持一致

修改完毕后，重启网卡

```bash
service networking restart

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ad:e4:75:1d  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.100.100  netmask 255.255.255.0  broadcast 0.0.0.0
        inet6 fe80::20c:29ff:fe50:599d  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:50:59:9d  txqueuelen 1000  (Ethernet)
        RX packets 21878  bytes 22915568 (22.9 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13016  bytes 1009765 (1.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 19  base 0x2000  

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1148  bytes 81664 (81.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1148  bytes 81664 (81.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

此时查看网卡配置，可以看到IP地址`192.168.100.100`已经配置成功

## 配置dns

可以尝试ping外部网络

```bash
ping baidu.com    
ping: baidu.com: Temporary failure in name resolution
```

因为默认的dns导向localhost导致无法解析域名

临时配置`/etc/resolv.conf`

```bash
➜  cat /etc/resolv.conf 
# This file is managed by man:systemd-resolved(8). Do not edit.
#
# This is a dynamic resolv.conf file for connecting local clients to the
# internal DNS stub resolver of systemd-resolved. This file lists all
# configured search domains.
#
# Run "systemd-resolve --status" to see details about the uplink DNS servers
# currently in use.
#
# Third party programs must not access this file directly, but only through the
# symlink at /etc/resolv.conf. To manage man:resolv.conf(5) in a different way,
# replace this symlink by a static file or a different symlink.
#
# See man:systemd-resolved.service(8) for details about the supported modes of
# operation for /etc/resolv.conf.

nameserver 8.8.8.8
options edns0
```

修改nameserver为`8.8.8.8`即可

但是每次重启后，`resolv.conf`就会被重置为默认值

```bash
apt-get install resolvconf
/etc/resolvconf/resolv.conf.d/base
```

加入内容`nameserver 8.8.8.8`

## 最终效果

可以通过ssh直接连接`root@192.168.100.100:22`

主机ping虚拟机

```bash
C:\Users\landers>ping 192.168.10.100

正在 Ping 192.168.10.100 具有 32 字节的数据:
来自 192.168.10.100 的回复: 字节=32 时间<1ms TTL=128
来自 192.168.10.100 的回复: 字节=32 时间<1ms TTL=128

192.168.10.100 的 Ping 统计信息:
    数据包: 已发送 = 2，已接收 = 2，丢失 = 0 (0% 丢失)，
往返行程的估计时间(以毫秒为单位):
    最短 = 0ms，最长 = 0ms，平均 = 0ms
Control-C
```

虚拟机ping主机

```bash
➜  ping 192.168.0.100
PING 192.168.0.100 (192.168.0.100) 56(84) bytes of data.
64 bytes from 192.168.0.100: icmp_seq=1 ttl=128 time=0.456 ms
64 bytes from 192.168.0.100: icmp_seq=2 ttl=128 time=0.263 ms
64 bytes from 192.168.0.100: icmp_seq=3 ttl=128 time=0.319 ms
64 bytes from 192.168.0.100: icmp_seq=4 ttl=128 time=0.260 ms
64 bytes from 192.168.0.100: icmp_seq=5 ttl=128 time=0.248 ms
^C
--- 192.168.0.100 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4098ms
rtt min/avg/max/mdev = 0.248/0.309/0.456/0.078 ms
```

