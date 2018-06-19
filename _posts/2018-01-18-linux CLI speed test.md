---
layout: post
title: linux CLI speed test
date: 2018-01-18
tags: CentOS speedtest
---

安裝 (python 程式) , 測試 .....

```
# wget -O speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
# chmod +x speedtest-cli
# ./speedtest-cli
Retrieving speedtest.net configuration...
Testing from HiNet (XX.XX.XX.XX)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Taiwan Mobile (Taichung) [1.78 km]: 8.171 ms
Testing download speed................................................................................
Download: 89.00 Mbit/s
Testing upload speed................................................................................................
Upload: 41.81 Mbit/s
```

查看有哪些伺服器測試可測試

```
# ./speedtest-cli --list
```

測試到伺服器 5854 (Taiwan Star Telecom)

```
#  ./speedtest-cli --server 5854
Retrieving speedtest.net configuration...
Testing from HiNet (XX.XX.XX.XX)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Taiwan Star Telecom Co., Ltd. (Taichung) [1.78 km]: 9.891 ms
Testing download speed................................................................................
Download: 93.21 Mbit/s
Testing upload speed................................................................................................
Upload: 41.75 Mbit/s
```
