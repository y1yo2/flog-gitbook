---
description: 本机ssh（固定ip）登录virtual box，虚机可访问外网
---

# 虚机内网ssh+访问外网

### 目的

1. 多虚机搭建中间件集群（固定ip，方便测试）
2. 主机可ssh登录虚机
3. 虚机可访问外网

### 问题

初始方案：虚机网卡桥接主机物理网卡（有线或无线，外网）。

结果：虚机可访问外网。

问题：

* 虚机ip不固定；
* 主机能否ssh连接虚机，取决外部局域网；
* 主机使用有线或无线上网，虚机桥接网卡需切换；

### 解决方案

* 总体思路

1. 桥接机器的物理网卡，自动获取IP、DNS，用于上外网。
2. 桥接一张虚拟网卡，设置固定网段、IP，用于固定IP，主机内SSH访问。

* 使用Hyper-V创建虚拟网卡。

{% embed url="https://docs.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/about/" %}

1. 创建内部类型，虚拟交换机。
2. 通过Windows的控制面板-网络连接，设置IP地址、子网掩码，固定虚拟网卡的网段。（例，IP：192.168.1.1，掩码：255.255.255.0，网段：192.168.1.x）
3. 虚机桥接物理网卡、虚拟网卡。
4. 虚机内设置两张网卡的配置。

物理网卡配置（默认网卡，访问外网）：

```bash
BOOTPROTO=dhcp #开启DHCP服务，自动获取主机IP
ONBOOT=yes #开机自动连接网络

NAME=eth0
DEVICE=eth0
```

虚拟网卡配置（固定IP，内网互访）：

```bash
BOOTPROTO=static #固定IP
ONBOOT=yes #开机自动连接网络
IPADDR=192.168.1.2
NETMASK=255.255.255.0

NAME=eth1
DEVICE=eth1
```

重启网络：

```bash
sudo systemctl restart network
sudo ip addr
```



* 安装 netstat 命令

netstat命令包含在net-tools软件包中，可通过yum在线安装，或rpm安装包安装。

```bash
# yum install net-tools     [On CentOS/RHEL]
# apt install net-tools     [On Debian/Ubuntu]
# zypper install net-tools  [On OpenSuse]
# pacman -S netstat-nat     [On Arch Linux]
```

创建用户并授权sudo（用户，可切换的用户及用户组，可执行命令）

* 添加用户：useradd
* 设置密码：passwd
* 查看当前用户：who am i
* 查看用户详情：id \[username\]
* 查看所有用户：cat /etc/passwd
* 设置sudo权限：vi /etc/sudoers
  * chmod u+w /etc/sudoers
  * root ALL=\(ALL:ALL\) ALL
  * 用户 登录主机=\(切换用户:切换用户组\) 可执行命令
* sudo使用root用户权限：sudo
  * 系统通过 /etc/sudoers 检查用户是否有sudo权限。
  * 有则需输入用户密码（root执行sudo无需密码）。
  * 密码验证成功后执行命令。
* 切换用户：su

### 安装docker

{% embed url="https://docs.docker.com/engine/install/centos/" %}

使用 yum 通过 Docker Repository 安装 Docker Engine。

add-repo: [https://download.docker.com/linux/centos/docker-ce.repo](https://download.docker.com/linux/centos/docker-ce.repo)

安装rzsz命令：sudo yum install lrzsz







## 遗留问题

1.安装yum的Docker Repository，网络不通问题

could not resolve host download.docker.com

2.无法设置DNS

* /etc/sysconfig/network-scripts/ifcfg-enp0s3
  * DNS1=8.8.8.8 DNS2=114.114.114.114 PEERDNS=no（是否允许DHCP获得的DNS覆盖本地的DNS）
  * eth接口启用DHCP后，本地resolv.conf文件将被修改，resolv.conf文件中的DNS地址将被改为从DHCP获取到的地址。
* /etc/systemd/resolved.conf
* /etc/resolved.conf：当需要解析域名时，读取该文件获得DNS 服务器 IP；
* /etc/hosts ：这个是最早的 hostname 对应 IP 的存档；
* /etc/nsswitch.conf：这个档案『决定』先使用 /etc/hosts 还是 /etc/resolv.conf 的设定；

3.服务 systemd-resolved 找不到

linux的init有关



4.主机可ping通虚机，但ssh失败

原因是，虚机的ip设置为网卡的网段ip，造成ip冲突。











