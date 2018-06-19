---
layout: post
title: CentOS 7上部署 Google BBR
date: 2018-01-17
tags: CentOS
---

BBR 需要將 CentOS 7 機器的內核升級到 4.9.0 以後的版本

不知為啥裝 新版核心我的 CentOS 開不了機 ...可能是 VM 的原因 

應該是 vmware 的問題 .... 只能支援到  kernel 4.12 .....

安裝 kernel 4.13 就  開不了機 ..... XD 

(參考 ....之前文章 .... 

https://echochio-tw.github.io/2017/02/Centos-7-%E6%8F%9B-kernel-4.X/

手動裝 kernel-ml-4.9.9-1.el7.elrepo.x86_64.rpm 可以開機

也裝了  kernel 4.9.9 相關套件

```
kernel-ml-4.9.9-1.el7.elrepo.x86_64.rpm
kernel-ml-headers-4.9.9-1.el7.elrepo.x86_64.rpm
kernel-ml-tools-4.9.9-1.el7.elrepo.x86_64.rpm
kernel-ml-tools-libs-4.9.9-1.el7.elrepo.x86_64.rpm
```

裝好後

```
# uname -a
Linux proxy 4.9.9-1.el7.elrepo.x86_64 #1 SMP Thu Feb 9 11:43:40 EST 2017 x86_64 x86_64 x86_64 GNU/Linux
```

BBR 算法修改 sysctl 配置

```
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

確認啟用 BBR .... 看到顯示 bbr

```
# sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = bbr cubic reno
# sysctl -n net.ipv4.tcp_congestion_control
bbr
# lsmod | grep bbr
tcp_bbr                16384  10
```

重開我的機器 nginx 服務開很久 ...... 我將fireward 服務關閉就好了 .....
用 iptable 去管吧 ...

```
systemctl disable firewalld
systemctl stop firewalld
```
