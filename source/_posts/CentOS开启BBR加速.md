---
title: CentOS开启BBR加速
date: 2019-12-24 15:30:51
categories: 
- Linux
tags: 
- Linux
---

#### 查看系统版本

```shell
cat /etc/redhat-release
```

#### CentOS 7

1. 安装elrepo并升级内核

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel install kernel-ml -y
```

2. 更新grub文件并重启

```
egrep ^menuentry /etc/grub2.cfg | cut -f 2 -d \'
grub2-set-default 0
reboot
```

3. 查看内核版本号，大于4.1即可开启BBR，当前(2019.11)最新为5.3.13

```
uname -r
```

4. 编辑配置文件

```
vim /etc/sysctl.conf
```

5. `sysctl.conf`中添加如下内容

```
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

6. 加载系统参数

```
sysctl -p
```

7. 验证BBR是否开启

```
#方式1：
sysctl net.ipv4.tcp_available_congestion_control
#返回：
net.ipv4.tcp_available_congestion_control = reno cubic bbr

#方式2:
lsmod | grep bbr
#返回：
tcp_bbr                20480  2	//有值即可
```

#### CentOS 8

由于CentOS 8内核版本高于4.1，所以无需更新内核，直接开启BBR即可

1. 简单起见，直接写入到`/etc/sysctl.conf`

```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```

2. 加载系统参数

```
sysctl -p
```

3. 检查开启状态

```
sysctl -n net.ipv4.tcp_congestion_control
lsmod | grep bbr
```



