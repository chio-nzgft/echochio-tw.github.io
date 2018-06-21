---
layout: post
title: CentOS 7 OS 忘記 root 密碼
date: 2017-04-06
tags: Centos root
---
話說不小心把金鑰砍了 ....登出後進不了 OS .....

 開機 選 e 進入 GRUB 編輯

找到 linux16 那一行將 ro 改成 
```
rw init=/sysroot/bin/sh
```
並按 Ctrl+x 開機

```
chroot /sysroot
passwd root
tocuh /.autorelabel
exit
reboot
```

<img src="/images/posts/Server/p50.png">
