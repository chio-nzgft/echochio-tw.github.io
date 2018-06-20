---
layout: post
title: 用 powercli 建立排程定時做快照
date: 2017-06-06
tags: Esxi powercli
---

這要配合 vCenter .... XD

1. 要寫個 BAT 叫 ......vm-snap.bat  好了

```
#echo off

Title,Report Script &color 9e
for /f "usebackq delims=$" %%a in (`cd`) do (
  set SCRIPTDIR=%%a
)

(Set ScriptFile=C:\script\VM_SNAP.ps1 )

C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -psc "C:\Program Files (x86)\VMware\Infrastructure\vSphere PowerCLI\vim.psc1" -noe -c ". \"C:\Program Files (x86)\VMware\Infrastructure\vSphere PowerCLI\Scripts\Initialize-PowerCLIEnvironment.ps1\"";%ScriptFile%"
```

2. 寫 VM_SNAP.ps1

```
# snap VM
# [ON LINE] VMH
# Eat Params
$doVMname = '`[ON LINE`] VMH'
$vCenterName = "192.168.0.200"
$filenameFormat = "VMH" + (get-date -Format "yyyy-MM-dd")
$delfilenameFormat = "VMH" + (get-date (get-date).addDays(-3) -Format "yyyy-MM-dd")

# Load Powershell snapin from VMware
# ignore all errors
Add-PSSnapin VMware.VimAutomation.Core -ErrorAction SilentlyContinue

# Connect ESX/vCenter Server
connect-viserver $vCenterName

$VMF = Get-VM $doVMname

# juet test get snapshot list for $doVMname
get-snapshot -vm $VMF

# take snapshoot
New-snapshot -VM $VMF -name $filenameFormat -Quiesce -Memory

# del old snapshoot
get-snapshot -VM $VMF -name $delfilenameFormat |  remove-snapshot  -confirm:$false
```

3. vm-snap.bat  放到排程 ...設成不登入與否都要執行 ....
