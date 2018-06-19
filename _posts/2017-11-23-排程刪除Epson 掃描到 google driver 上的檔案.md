---
layout: post
title: 排程刪除Epson 掃描到 google driver 上的檔案
date: 2017-11-23
description: 排程刪除Epson 掃描到 google driver 上的檔案
tag: Epson google driver 掃描
--- 

以前的IT 交代要去定期刪除Epson 掃描到 google driver 上的檔案 ....XD

這事不是排程就好了嗎 ? 挖哩

下載 gdriver 

```

https://github.com/prasmussen/gdrive

```
我的是 linux x64 版本

```

[root@test ~]# wget -O /root/gdriver https://docs.google.com/uc?id=0B3X9GlR6EmbnQ0FtZmJJUXEyRTA&export=download
[root@test ~]# chmod +x /root/gdriver

```

取得google 帳號認證 ....

```

[root@test ~]# ./gdriver list
Authentication needed
Go to the following url in your browser:
https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=367116221053-7n0vf5akeru7on6o2fjinrecpdoe99eg.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&response_type=code&scope=https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fdrive&state=state

Enter verification code:

```

在網頁打入那連結後 .....取得 key 再輸入

```

Enter verification code: 4/s0ybtPXMr10eIfw9tVgka2ilWYH7TT3Kd7tHJNT-YGU

```

再次 list 就有資訊了
```

[root@test ~]# ./gdriver list
Id                                  Name                       Type   Size       Created
1N3OkCfr0cVeC0TFyLZL01DP-ewvhDdAM   Epson_20171123113109.pdf   bin    1.4 MB     2017-11-20 11:32:13
15h_sXKQ8A92YYKFsYBk1RxgsAve3XyxP   Epson_20171122173553.pdf   bin    166.9 KB   2017-11-20 17:36:15
1eSubPyuwUAFnLa3HBY9Lquhz8SbyMAfu   Epson_20171122160610.pdf   bin    1.1 MB     2017-11-20 16:07:29
1owuN8PvvHKDQomHkw_lL1rk0cKIoYs4p   Epson_20171122160500.pdf   bin    1.1 MB     2017-11-20 16:06:03
1wzGT8Mca91EoUxcL005PfOGJp8H0mDkt   Epson_20171122160255.pdf   bin    1.0 MB     2017-11-20 16:03:59
1uRvUfaCH7BTZYDdYCV-Y-bcU0_-bkCGc   Epson_20171121165452.pdf   bin    1.3 MB     2017-11-20 16:55:56
1wUv0iKnbPuZjqAh9E4Z8AIoMg3W4fskM   Epson_20171120181534.pdf   bin    1.5 MB     2017-11-20 18:16:38
164FYCYn4PyrxzgKPa-nuWiqz47rgmIUK   Epson_20171120144024.pdf   bin    1.0 MB     2017-11-20 14:41:29
1vOjj2ODZ3CwizFnjmaQMzYe4EfXaxp-C   Epson_20171120115905.jpg   bin    608.8 KB   2017-11-20 12:00:09
1H2ggWHEfbZk2mi6kNPW2HZVgNv5JSIgZ   Epson_20171120112017.pdf   bin    1.9 MB     2017-11-20 11:21:21
16yhI4YPPvkj2uOGq5PwqXSfOkKhtUvgQ   Epson_20171120111844.pdf   bin    2.0 MB     2017-11-20 11:19:49
1_BOGxZ0M__7FEQzRt_vlXOzwIPAQaMU6   Epson_20171120102646.jpg   bin    793.4 KB   2017-11-20 10:27:51
1Msy7P_gLJelaWxCuupY9Az9mr_KBbWBx   Epson_20171116173740.jpg   bin    1.0 MB     2017-11-20 17:38:43


```

要刪除三天前的檔案 .... 用 date 取得

```

date --date='3 days ago' "+%Y-%m-%d"

```

測試刪除的script 產生
```

[root@test ~]# /root/gdriver list |grep Epson_20 |grep `date --date='3 days ago' "+%Y-%m-%d"` |awk '{print "/root/gdriver delete "$1"}'
/root/gdriver delete 1N3OkCfr0cVeC0TFyLZL01DP-ewvhDdAM
/root/gdriver delete 15h_sXKQ8A92YYKFsYBk1RxgsAve3XyxP
/root/gdriver delete 1eSubPyuwUAFnLa3HBY9Lquhz8SbyMAfu
/root/gdriver delete 1owuN8PvvHKDQomHkw_lL1rk0cKIoYs4p
/root/gdriver delete 1wzGT8Mca91EoUxcL005PfOGJp8H0mDkt
/root/gdriver delete 1uRvUfaCH7BTZYDdYCV-Y-bcU0_-bkCGc
/root/gdriver delete 1wUv0iKnbPuZjqAh9E4Z8AIoMg3W4fskM
/root/gdriver delete 164FYCYn4PyrxzgKPa-nuWiqz47rgmIUK
/root/gdriver delete 1vOjj2ODZ3CwizFnjmaQMzYe4EfXaxp-C
/root/gdriver delete 1H2ggWHEfbZk2mi6kNPW2HZVgNv5JSIgZ
/root/gdriver delete 16yhI4YPPvkj2uOGq5PwqXSfOkKhtUvgQ
/root/gdriver delete 1_BOGxZ0M__7FEQzRt_vlXOzwIPAQaMU6
/root/gdriver delete 1Msy7P_gLJelaWxCuupY9Az9mr_KBbWBx

```

看來沒錯 .....就可寫 script

```

[root@test ~]# /root/gdriver list |grep Epson_20 |grep `date --date='3 days ago' "+%Y-%m-%d"` |awk '{print "/root/gdriver delete "$1";sleep 10"}' > /tmp/delgdriver ;chmod +x /tmp/delgdriver;/tmp/delgdriver
Deleted 'Epson_20171123113109.pdf'
Deleted 'Epson_20171122173553.pdf'
Deleted 'Epson_20171122160610.pdf'
Deleted 'Epson_20171122160500.pdf'
Deleted 'Epson_20171122160255.pdf'
Deleted 'Epson_20171121165452.pdf'
Deleted 'Epson_20171120181534.pdf'
Deleted 'Epson_20171120144024.pdf'
Deleted 'Epson_20171120115905.jpg'
Deleted 'Epson_20171120112017.pdf'
Deleted 'Epson_20171120111844.pdf'
Deleted 'Epson_20171120102646.jpg'
Deleted 'Epson_20171116173740.jpg'

```

如何寫 crontab 就不必寫了吧 .......XD
