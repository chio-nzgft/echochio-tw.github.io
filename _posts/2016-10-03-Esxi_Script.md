---
layout: post
title: 話說我備份Esxi guest 都是用 Script .重點是免費
date: 2016-10-03
tags: esxi backup Script
---
話說我備份Esxi guest 都是用 Script ....  免錢 .... 哈哈

當然要裝 VMware vSphere PowerCLI ......用 powershell ...

要寫個 BAT ......執行 PBX.DB_backup.ps1 

```
#echo off

Title,Report Script &color 9e
for /f "usebackq delims=$" %%a in (`cd`) do (
  set SCRIPTDIR=%%a
)

(Set ScriptFile=C:\script\PBX.DB_backup.ps1 )

C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -psc "C:\Program Files (x86)\VMware\Infrastructure\vSphere PowerCLI\vim.psc1" -noe -c ". \"C:\Program Files (x86)\VMware\Infrastructure\vSphere PowerCLI\Scripts\Initialize-PowerCLIEnvironment.ps1\"";%ScriptFile%"
```

程式如下 ...當然抄抄改改 

```
# Clone VM
# Script is base on http://communities.vmware.com/message/1506367, LucD’s reply.
# Added args for mulitple use by mark
# [ON LINE] PBX.SQL (Win2012 core)
# Eat Params
$fromVMname = '`[ON LINE`] PBX.SQL (Win2012 core)'
$newVMName =  '`[OFF LINE`] PBX.SQL (Win2012 core) -- BACKUP'
$tgtEsxName = "192.168.1.199"
$tgtDatastoreName = "datastore-2"

# Load Powershell snapin from VMware
# ignore all errors
Add-PSSnapin VMware.VimAutomation.Core -ErrorAction SilentlyContinue

# Connect ESX/vCenter Server
connect-viserver $vCenterName

# remove cloned VM
$VMR = get-vm $newVMName
Remove-vm -VM $VMR -DeleteFromDisk -confirm:($false)

$newVMName =  '[OFF LINE] PBX.SQL (Win2012 core) -- BACKUP'

$VMF = Get-VM $fromVMname

# Doing Clone
$cloneTask = New-VM -Name $newVMName -VM $VMF -VMHost (Get-VMHost $tgtEsxName) -Datastore (Get-Datastore $tgtDatastoreName) -RunAsync

#email back
Wait-Task -Task $cloneTask -ErrorAction SilentlyContinue
Get-Task | where {$_.Id -eq $cloneTask.Id} | %{
$EmailFrom = "chiotest@gmail.com"
$EmailTo = "chiotest@gmail.com" 
$Subject = "Esxi backup " + $fromVMname + " " + $_.State
$Body = "this is a notification from powershell.. " + $event
$SMTPServer = "smtp.gmail.com" 
$SMTPClient = New-Object Net.Mail.SmtpClient($SmtpServer, 587) 
$SMTPClient.EnableSsl = $true
$SMTPClient.Credentials = New-Object System.Net.NetworkCredential("chiotest", "password");
$SMTPClient.Send($EmailFrom, $EmailTo, $Subject, $Body)
}

# powershell -command "& {c: clonevm.ps1 comp-name clone-comp-name esx-ip datastore v}"
```

排程

<img src="/images/posts/Esxi_VCB/p101.png">

結果

<img src="/images/posts/Esxi_VCB/p102.png">

ps :  gmail 要開放 低安全性應用程式 ....才會寄信成功

<img src="/images/posts/Esxi_VCB/p103.png">
