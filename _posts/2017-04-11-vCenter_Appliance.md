---
layout: post
title: vCenter Server Appliance 6.5 安裝
date: 2017-04-11
tags: vCenter Esxi
---

抓 ISO 檔 ....找台 windows 執行內的 
```
vcsa-ui-installer\win32\installer
```

<img src="/images/posts/Esxi_VCB/p50.png">

選安裝 ....我用內嵌式 的 vCenter ...

<img src="/images/posts/Esxi_VCB/p51.png">

中間會要求輸入 Esxi_VCB 的 ip & 帳密 ....將資料 ...輸入 ....就開始安裝了.....

<img src="/images/posts/Esxi_VCB/p52.png">

裝好了 ....接下來設定

<img src="/images/posts/Esxi_VCB/p53.png">

<img src="/images/posts/Esxi_VCB/p54.png">

<img src="/images/posts/Esxi_VCB/p55.png">

如果有網域就可加入 ....

<img src="/images/posts/Esxi_VCB/p56.png">

<img src="/images/posts/Esxi_VCB/p57.png">


開起來看到 OS 換成 PHOTON

<img src="/images/posts/Esxi_VCB/p58.png">

console 畫面與 Esxi_VCB 很像

<img src="/images/posts/Esxi_VCB/p59.png">

PS : 原先用 OVA 安裝但裝完 OS 後很難設定 ...用 UI 好設定多了
