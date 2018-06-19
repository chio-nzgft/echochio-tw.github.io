---
layout: post
title: vbscript check gateway
date: 2017-10-26
tags: vbscript check gateway
---

&emsp;&emsp;在主機上寫個 Script 利用排程每五分鐘 判斷 gateway 192.168.0.1 是否可以 ping 到 google 不可以就換 gateway 192.168.0.2 且發信告知 
   
當切換到  192.168.0.2 後 , 下次過 5 分鐘再判斷看看 gateway       192.168.0.1 是否恢復 , 恢復就切回 192.168.0.1 且寄信通知 ....

亂寫就用一條龍寫法 ...不寫副程式了 ....


----------
```
'判斷是否是 Administrator 權限 , 不是Admin 將切換 ... 切換時可能會問 (Y/N)
If Not WScript.Arguments.Named.Exists("elevate") Then
  CreateObject("Shell.Application").ShellExecute WScript.FullName _
    , """" & WScript.ScriptFullName & """ /elevate", "", "runas", 1
  WScript.Quit
End If

'取得 strDefaultIPGateway 
strComputer = "."
Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")
Set colNetAdapters = objWMIService.ExecQuery("Select * from Win32_NetworkAdapterConfiguration where IPEnabled=TRUE")
For Each objNetAdapter in colNetAdapters
    If Not isNull(objNetAdapter.DefaultIPGateway) Then
    strDefaultIPGateway = Join(objNetAdapter.DefaultIPGateway, ",")
    End If 
Next

'設定檢查 hostname
 Dim hostname
 hostname = "www.google.com"

 '判斷 是否 gateway 是否是 192.168.0.2
if strDefaultIPGateway = "192.168.0.2" Then

 ' 如 gateway 是 192.168.0.2 就切換到 192.168.0.1
 Set WshShell3 = WScript.CreateObject("WScript.Shell")
 WshShell3.currentdirectory="c:\"
 WshShell3.Run "route change 0.0.0.0 mask 0.0.0.0 192.168.0.1", 0
 Set WshShell3 = Nothing

 ' 用 gateway 192.168.0.1 ping google
 Set WshShell = WScript.CreateObject("WScript.Shell")
 Ping = WshShell.Run("ping -n 1 " & hostname, 0, True)
 Select Case Ping
 
 ' 當 ping 到 google 寄信通知已切換回來 gateway 192.168.0.1
 Case 0
  Set CDOMail = CreateObject("CDO.Message")
    With CDOMail
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/sendusing") =2
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/smtpserver") = "192.168.0.3"
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/smtpserverport") = 25
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/sendusername") = "echochio"
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/sendpassword") = "echochiopassord"
        .Configuration.Fields.Update
        .From = "echochio@test.com"
        .To = "echochio@test.com"
        .Subject = "Back to gateway to 192.168.0.1"
        .TextBody = "ping www.google.com.tw OK !! Back to gateway to 192.168.0.1"
        .Send
    End With
    Set CDOMail = Nothing

 ' 當 ping 不到 google 切換回去原本的 gateway 192.168.0.2
 Case 1 
  Set WshShell2 = WScript.CreateObject("WScript.Shell")
  WshShell2.currentdirectory="c:\"
  WshShell2.Run "route change 0.0.0.0 mask 0.0.0.0 192.168.0.2", 0
  Set WshShell2 = Nothing
 End Select

' 如 gateway 不是 192.168.0.2 所以是 192.168.0.1 , ping google
Else 
 Set WshShell = WScript.CreateObject("WScript.Shell")
 Ping = WshShell.Run("ping -n 1 " & hostname, 0, True)
 Select Case Ping
 
 '當 ping 不到 google 切換去 192.168.0.2 且寄信通知
 Case 1
  Set WshShell2 = WScript.CreateObject("WScript.Shell")
  WshShell2.currentdirectory="c:\"
  WshShell2.Run "route change 0.0.0.0 mask 0.0.0.0 192.168.0.2", 0
  Set WshShell2 = Nothing
  
  '寄信通知 切換到 192.168.0.2 了
  Set CDOMail = CreateObject("CDO.Message")
    With CDOMail
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/sendusing") =2
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/smtpserver") = "192.168.0.3"
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/smtpserverport") = 25
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/sendusername") = "echochio"
        .Configuration.Fields("http://schemas.microsoft.com/cdo/configuration/sendpassword") = "echochiopassword"
        .Configuration.Fields.Update
        .From = "echochio@test.com"
        .To = "echochio@test.com"
        .Subject = "defult gateway cannot to google"
        .TextBody = "ping www.google.com.tw fail !! Change gateway to 192.168.0.2"
        .Send
    End With
    Set CDOMail = Nothing
 End Select
End If
```



----------
開始是判斷是否是 Administrator 權限


其中 mail server 是 192.168.0.3 , 帳號 echochio 密碼 echochiopassword , 寄信到 echochio@test.com


要設定最高權限執行

<img src="/images/posts/1508824330-712718608_n.png">

還沒寫完 ...要寫 ...當第二條gateway 也掛了 .....那....要如何 ? 發簡訊 ....哈
