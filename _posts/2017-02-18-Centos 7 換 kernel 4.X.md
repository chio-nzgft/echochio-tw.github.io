---
layout: post
title: Centos 7 換 kernel 4.X
date: 2017-02-18
tags: CentOS kernel
---
Centos 7 換 kernel 4.X
紀錄一下 :

確定版本

```
# cat /etc/centos-release
CentOS Linux release 7.3.1611 (Core)
```

看一下 kernel 是 3.X 沒錯

```
# uname -a
Linux ocs  3.10.0-514.6.1.el7.x86_64 #1 SMP Wed Feb 15 14:24:04 EST 2017 x86_64 x86_64 x86_64 GNU/Linux
```

抓 kernel 來源

```
# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```

安裝 kernel 4.X

```
# yum --enablerepo=elrepo-kernel install kernel-ml
```

看 kernel

```
# rpm -qa |grep kernel
kernel-ml-4.9.10-1.el7.elrepo.x86_64
kernel-3.10.0-514.6.1.el7.x86_64
kernel-tools-libs-3.10.0-514.6.1.el7.x86_64
kernel-tools-3.10.0-514.6.1.el7.x86_64
kernel-headers-3.10.0-514.6.1.el7.x86_64
```

除了 kernel-3.X 不移除其他  kernel-headers kernel-tools kernel-tools-libs 都移除

```
# yum remove kernel-headers-3.10.0-514.6.1.el7.x86_64 kernel-tools-libs-3.10.0-514.6.1.el7.x86_64 kernel-tools-3.10.0-514.6.1.el7.x86_64
```

改 grub2

```
# awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
# cat /etc/grub2.cfg
# grub2-set-default 0
```

安裝 4.X kernel 的 headers tools libs

```
# yum --enablerepo=elrepo-kernel install kernel-ml-headers kernel-ml-tools kernel-ml-tools-libs
# yum update
# yum updategrade
```

重˙開後看 版本應該是新版的

```
# uname -a
Linux ocs 4.9.10-1.el7.elrepo.x86_64 #1 SMP Wed Feb 15 14:24:04 EST 2017 x86_64 x86_64 x86_64 GNU/Linux
```
