---
layout: post
title: win10 remove app (移除市集程式)
date: 2018-07-03
tags: Win10 powershell
---

通常公司都沒有用，全移除
管理者權限 powershell 

```
Get-AppxPackage *.* | Remove-AppxPackage
```

以下是一個一個移除 ........

Uninstall 3D Builder:
```
Get-AppxPackage *3dbuilder* | Remove-AppxPackage
```

Uninstall Alarms and Clock:
```
Get-AppxPackage *windowsalarms* | Remove-AppxPackage
```
Uninstall Calculator:
```
Get-AppxPackage *windowscalculator* | Remove-AppxPackage
```
Uninstall Calendar and Mail:
```
Get-AppxPackage *windowscommunicationsapps* | Remove-AppxPackage
```
Uninstall Camera:
```
Get-AppxPackage *windowscamera* | Remove-AppxPackage
```

Uninstall Get Office:
```
Get-AppxPackage *officehub* | Remove-AppxPackage
```

Uninstall Get Skype:
```
Get-AppxPackage *skypeapp* | Remove-AppxPackage
```

Uninstall Get Started:
```
Get-AppxPackage *getstarted* | Remove-AppxPackage
```

Uninstall Groove Music:
```
Get-AppxPackage *zunemusic* | Remove-AppxPackage
```

Uninstall Maps:
```
Get-AppxPackage *windowsmaps* | Remove-AppxPackage
```

Uninstall Microsoft Solitaire Collection:
```
Get-AppxPackage *solitairecollection* | Remove-AppxPackage
```

Uninstall Money:
```
Get-AppxPackage *bingfinance* | Remove-AppxPackage
```

Uninstall Movies & TV:

```
Get-AppxPackage *zunevideo* | Remove-AppxPackage
```

Uninstall News:
```
Get-AppxPackage *bingnews* | Remove-AppxPackage
```

Uninstall OneNote:
```
Get-AppxPackage *onenote* | Remove-AppxPackage
```

Uninstall People:
```
Get-AppxPackage *people* | Remove-AppxPackage
```

Uninstall Phone Companion:
```
Get-AppxPackage *windowsphone* | Remove-AppxPackage
```

Uninstall Photos:
```
Get-AppxPackage *photos* | Remove-AppxPackage
```

Uninstall Store:
```
Get-AppxPackage *windowsstore* | Remove-AppxPackage
```

Uninstall Sports:
```
Get-AppxPackage *bingsports* | Remove-AppxPackage
```

Uninstall Voice Recorder:
```
Get-AppxPackage *soundrecorder* | Remove-AppxPackage
```

Uninstall Weather:
```
Get-AppxPackage *bingweather* | Remove-AppxPackage
```

Uninstall Xbox:
```
Get-AppxPackage *xboxapp* | Remove-AppxPackage
```

還原安裝的  APP
```
Get-AppxPackage -AllUsers| Foreach {Add-AppxPackage -DisableDevelopmentMode -Register "$($_.InstallLocation)\AppXManifest.xml"}
```
