---
layout: post
title: 群輝 NFS for ESXi
date: 2017-08-11
tags: 自群輝 NFS ESXi
---
要將 NFS 協定打開
<img src="https://echochio-tw.github.io/images/posts/nas_for_esxi/1.png">
建立一個目錄叫 NFS :
<img src="https://echochio-tw.github.io/images/posts/nas_for_esxi/2.png">
共用資料夾權限 :
<img src="https://echochio-tw.github.io/images/posts/nas_for_esxi/3.png">
設定那個 IP 可連這 NFS :
<img src="https://echochio-tw.github.io/images/posts/nas_for_esxi/4.png">
ESXi 設定 : (資料夾設 /volume1/NFS )
<img src="https://echochio-tw.github.io/images/posts/nas_for_esxi/5.png">
這樣就連上去了
<img src="https://echochio-tw.github.io/images/posts/nas_for_esxi/6.png">
