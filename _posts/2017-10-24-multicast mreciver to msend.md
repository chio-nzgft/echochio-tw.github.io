---
layout: post
title: multicast mreciver to msend
date: 2017-10-27
description: multicast mreciver to msend
tag: multicast mreciver msend
---

之前寫的 ....現在放到 git ....紀錄一下

[![https://github.com/echochio-tw/multicast_mreciver_to_msend](https://github.com/echochio-tw/multicast_mreciver_to_msend)](https://github.com/echochio-tw/multicast_mreciver_to_msend)

利用 linux C 編譯　mcreceive_sent.c


執行  mcreceive_sent 224.2.50.22 5100 224.2.50.24 5101


就是 收 224.2.50.22:5100  打到 224.2.50.24:5101


利用 script 可以切換來源 ......


如  


來源是 224.2.50.22:5100 & 224.2.50.23:5100


先執行


mcreceive_sent 224.2.50.22 5100 224.2.50.24 5101


時間到時



kill -9 "grep mcreceive_sent" 


mcreceive_sent 224.2.50.23 5100 224.2.50.24 5101


----
.....

PS 之前有做影像判斷看何時kill mcreceive_sent .......XD
