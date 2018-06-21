---
layout: post
title: kill long run mysql process
date: 2013-08-13
tags:  linux shell sql mysql
---

紀錄起來還常用到的

``` 
mysql -e "show processlist" | awk '{ $6 = $6 +1; if ($6 >= 60 && $5 == "Query") system("mysqladmin kill "$1); print strftime("%c") "] query Killed. [account: "$2"] [database: "$4"] [query runtime: "$6" seconds]"; }' >> /var/log/mysql_kills.log
```

