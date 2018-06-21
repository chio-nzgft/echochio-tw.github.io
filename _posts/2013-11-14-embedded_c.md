---
layout: post
title: embedded 時區問題， linux 無解只好用 C
date: 2013-11-14
tags: linux c
---

就是將 UTC 時間 加上8  小時(28800  秒) 就OK了 。

找了 EPOCH 轉換 想用 linux date 解，embedded   內無法用，還真是無解。

```

#include <stdio.h>
#include <time.h>
int main(void)
{

time_t t = time(NULL);

t+=28800;

struct tm tm = *gmtime(&t);

printf("%04d%02d%02d%02d%02d%02d", tm.tm_year + 1900, tm.tm_mon + 1, tm.tm_mday, tm.tm_hour, tm.tm_min, tm.tm_sec);

return 0;

}
```
