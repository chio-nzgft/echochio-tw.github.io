---
layout: post
title: linux kill 多個相同 process 方法
date: 2013-10-07
tags:  linux process
---
很多 方法 記錄一下

```
ps -ef | grep PROCESS | grep -v grep | awk '{print $2}' | xargs kill -9
```

或
```
pkill -9 -f PROCESS
```

或
```
ps -ef | awk '/PROCESS/ && !/awk/ {print $2}' | xargs -r kill -9
```

或
```
killall PROCESS
```

或
```
tokill=`ps -ef|grep PROCESS|awk '{ printf $2" "}'`; kill -9 $tokill;
```

或
```
kill `ps -ef | grep PROCESS | grep -v grep | awk ?{print $2}?`
```
 
發現沒pkill 與 xgrgs 與 killall 指令只好用 最後兩個指令。 
