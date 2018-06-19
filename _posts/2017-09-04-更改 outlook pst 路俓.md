---
layout: post
title: 更改 outlook pst 路俓
date: 2017-09-04
tags: outlook pst
---

更改 outlook pst 路俓。 例如 D:\Outlook\

OFFICE版本：

Outlook 2007：
```
HKEY_CURRENT_USER\Software\Microsoft\Office\12.0\Outlook
```

Outlook 2010 ：
```
HKEY_CURRENT_USER\Software\Microsoft\Office\14.0\Outlook
```

其他看 是 16 or ........

```
新建字串值：ForcePSTPath，保存.pst的路俓。 例如 D:\Outlook\
```
