---
categories:
  - blog
  - 
title: DFIR - Final result 1 - Powershell telemetry by Windows
subtitle: Documentation on the powershell activity from Microsoft Compatibility Appraiser
tags: [windows, DFIR]
comments: false
header:
  teaser: /img/tel.jpg
---

## TLDR
Heaps of reddit posts and AV posts have discussed this command at length, with general users sometimes stating `powershell.exe -ExecutionPolicy Restricted -Command Write-Host 'Final result: 1'; ` is malicious.

This activity belongs to the opted in Windows telemetry done during windows setup for Windows 10/11 & 2019, so long as the output matches that at [Powershell Script Block](#powershell-script-block) and has the parent process of `C:\Windows\System32\CompatTelRunner.exe ` i don't believe it is malicious.

However, CompatTelRunner can be used for persistence, so if your unsure you should check the contents of the registry key at `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\TelemetryController` or check if the scheduled task has been modified. See more at [Checking for Persistance](#checking-for-persistance).

## Contents

- [TLDR](#tldr)
- [Contents](#contents)
- [Powershell Script Block](#powershell-script-block)
- [Sysmon Logging](#sysmon-logging)
- [CompatTelRunner](#compattelrunner)
- [Checking for Persistance](#checking-for-persistance)
- [Assessment](#assessment)
## Powershell Script Block

When checking your powershell script block commands you might see the following output

```
**********************
Windows PowerShell transcript start
Start time: <time>
Username: WORKGROUP\SYSTEM
RunAs User: WORKGROUP\SYSTEM
Host Application: powershell.exe -ExecutionPolicy Restricted -Command Write-Host 'Final result: 1';
**********************
PS>Write-Host 'Final result: 1';
Final result: 1
PS>$global:?
True
**********************
Windows PowerShell transcript end
End time: <time>
**********************
```

## Sysmon Logging

When we check sysmon, we can see this is standard windows telemetry behavior from the `C:\Windows\System32\CompatTelRunner.exe` process

```
EventData 

  RuleName technique_id=T1059.001,technique_name=PowerShell 
  ProcessGuid {979c8b9b-1901-6188-2c5d-000000000c00} 
  Image C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe 
  OriginalFileName PowerShell.EXE 
  CommandLine powershell.exe -ExecutionPolicy Restricted -Command Write-Host 'Final result: 1'; 
  CurrentDirectory C:\WINDOWS\system32\ 
  User NT AUTHORITY\SYSTEM 
  TerminalSessionId 0 
  IntegrityLevel System 
  Hashes SHA1=EEE0B7E9FDB295EA97C5F2E7C7BA3AC7F4085204,MD5=0E9CCD796E251916133392539572A374,SHA256=C7D4E119149A7150B7101A4BD9FFFBF659FBA76D058F7BF6CC73C99FB36E8221,IMPHASH=BF7A6E7A62C3F5B2E8E069438AC1DD3D 
  ParentImage C:\Windows\System32\CompatTelRunner.exe 
  ParentCommandLine C:\WINDOWS\system32\CompatTelRunner.exe -m:appraiser.dll -f:DoScheduledTelemetryRun -cv:f5xHeCd6QkakkzW0.1 
```

## CompatTelRunner

This process is part of the default Windows10 or 11 installation and some versions of Server 2019. It sends periodic usage and performance data to microsoft. This data appears to be sent to the domain settingsfd-geo.trafficmanager.net by https. So if you didn't want this data being sent to Microsoft, you could choose to disable it at a firewall level.

We can also see form the full commandline of this item `C:\WINDOWS\system32\CompatTelRunner.exe -m:appraiser.dll -f:DoScheduledTelemetryRun -cv:zFhNaBJ2wU+WHopX.1` that the string appears to be randomised. Obviously, microsoft want the public to know exactly what is happening here, however the data that this item is collecting appears completely benign.

Looking at the time this item executed, I have observed that the scheduled task for this exact task is called `Microsoft Compatibility Appraiser` you can get the info for the last runtime for this task from `Get-ScheduledTaskInfo -TaskPath "\Microsoft\Windows\Application Experience" -TaskName "Microsoft Compatibility Appraiser"`. 

This also means you could disable this task if you so wish with `Disable-ScheduledTask -TaskPath "\Microsoft\Windows\Application Experience" -TaskName "Microsoft Compatibility Appraiser"`. This scheduled task has been around since windows 7, and defaults to running at 3am everyday.

The tasks description is `"Collects program telemetry information if opted-in to the Microsoft Customer Experience Improvement Program."` So, if you Opt out by general sysadmin methods this task should go away. 

## Checking for Persistance

If you suspect malicious usage of this script, you should check the following registry keys. You can check this with the following powershell command

```
Get-ChildItem "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\TelemetryController"
$TelemetryController | format-list
```

As of November 2021, the default values in Windows 11 are
```
Name                           Property
----                           --------
Appraiser                      Command     : C:\WINDOWS\system32\CompatTelRunner.exe -m:appraiser.dll -f:DoScheduledTelemetryRun
                               Nightly     : 1
                               Sku         : 1
                               Result      : 0
AppraiserServer                Command     : C:\WINDOWS\system32\CompatTelRunner.exe -m:appraiser.dll -f:DoScheduledTelemetryRun
                               Maintenance : 1
                               Sku         : 2
AvStatus                       Command     : C:\WINDOWS\system32\CompatTelRunner.exe -m:appraiser.dll -f:UpdateAvStatus
                               Nightly     : 1
                               Sku         : 1
                               Result      : 0
DevInv                         Command     : C:\WINDOWS\system32\CompatTelRunner.exe -m:devinv.dll -f:CreateDeviceInventory
                               Oobe        : 1
                               Result      : 0
Encapsulation                  Command     : C:\WINDOWS\system32\CompatTelRunner.exe -m:pcasvc.dll -f:QueryEncapsulationSettings
                               Maintenance : 1
                               Nightly     : 1
                               Oobe        : 1
                               Result      : 0
InvAgent                       Command     : C:\WINDOWS\system32\CompatTelRunner.exe -m:invagent.dll -f:RunUpdate
                               Maintenance : 1
                               Oobe        : 1
                               Result      : 0
```

You should also check that the dll's above have not been modified. Futher documentation on these keys can be found from [Trustedsec](https://www.trustedsec.com/blog/abusing-windows-telemetry-for-persistence/?utm_content=131234033&utm_medium=social&utm_source=twitter&hss_channel=tw-403811306)
## Assessment

Overall, in my opinion this activity is non-malicious, and can safely be ignored, so long as the registry has not been modified.

However, if, in the future, you see non-expected output from this command, this should be further investigated. 

In the case there is privacy concerns for you, or your organisation, you should opt out of Windows telemetry, disable the scheduled task, and monitor the above registry key for changes.
