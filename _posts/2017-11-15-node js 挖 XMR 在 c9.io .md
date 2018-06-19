---
layout: post
title: node js 挖 XMR 在 c9.io.md
date: 2017-11-15
description: node js 挖 XMR 在 c9.io.md
tag: node.js XMR c9.io
--- 


<img src="https://echochio-tw.github.io/images/posts/c9.io/1.png">

----------
```
```
----------

<img src="https://echochio-tw.github.io/images/posts/c9.io/2.png">

----------
```
testuser:~/workspace $ node -v
v6.11.2
testuser:~/workspace $ nvm install 8
Downloading https://nodejs.org/dist/v8.9.1/node-v8.9.1-linux-x64.tar.xz...
######################################################################## 100.0%
Now using node v8.9.1 (npm v5.5.1)
testuser:~/workspace $ node -v
v8.9.1
testuser:~/workspace $ npm install -g coin-hive
/home/ubuntu/.nvm/versions/node/v8.9.1/bin/coin-hive -> /home/ubuntu/.nvm/versions/node/v8.9.1/lib/node_modules/coin-hive/bin/coin-hive

> puppeteer@0.10.2 install /home/ubuntu/.nvm/versions/node/v8.9.1/lib/node_modules/coin-hive/node_modules/puppeteer
> node install.js

Downloading Chromium r497674 - 92.5 Mb [====================] 100% 0.0s 
+ coin-hive@1.9.3
added 185 packages in 12.846s
testuser:~/workspace $ coin-hive CwgD5lA9Z10D9eDGlBfj2Hh2Svl0Mhwf

Site key: CwgD5lA9Z10D9eDGlBfj2Hh2Svl0Mhwf

   ┌───────────────────┬───────────────────┬───────────────────┐
   │     Hashes/s      │       Total       │     Accepted      │
   ├───────────────────┼───────────────────┼───────────────────┤
   │        6.5        │        235        │        256        │
   └───────────────────┴───────────────────┴───────────────────┘

⠏ | Listening on localhost:3002 | 8 Threads

s: Start/Stop | +/-: Threads | a: Auto threads | q/Ctrl-C: Quit

```
----------


c9.io 測試可以耶 .....XD

看得懂就在 c9.io 開個 node js 的專案 ....挖寶 .....

搞了老半天原來是 c9.io 上的 node.js 版本太舊換版就可以了 ....XD
