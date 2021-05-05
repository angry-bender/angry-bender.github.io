---
categories:
  - blog
title: DFIR Playbook - Windows Forensics
subtitle: An extract from my Physical Playbook
tags: [dfir, windows]
comments: false
header:
  teaser: [zt](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQaPxzKoMu88RECpi3OkBUhdgq7afyKpPW8gw&usqp=CAU)
---

# Event log Parsing
Log Location  Vista+\2008+ `%SYSTEMROOT%\winevt\logs`
Log Location XP\2003- `%SYSTEMROOT%\config
Event Log explorer
Evtxexport

# Registry Parsing
Location `%SYSTEMROOT%\config`
Registry Explorer
rip.pl
`.\RECmd.exe --bn .\BatchExamples\RECmd_Batch_MC.reb --csv <outputdirectory> -d '<ntuser.dat location>` | Decodes all user artefacts

Correlating hives

----------|-----------|-------
Filename | Hive | Notes
Sam | HKLM\SAM
Security | HKLM\SECURITY
Software | HKLM\SOFTWARE
System | HKLM\SYSTEM | Current control set at HKLM\SYSTEM\Select\Current

Location `%USERPROFILE%\Ntuser.dat`
Correlating hives

----------|-----------
Filename | Hive
Ntuser.dat | HKCU (Of current profile)  
%USERPROFILE%\Local Settings\Application Data\Microsoft\Windows\Usrclass.dat | XP\2003- HKCU All Users
%USERPROFILE%\AppData\Local\Microsoft\Windows\Usrclass.dat | Vista\2008+  HKCU All Users

# Prefetch Files
Location C:\Windows\Prefetch
WinPrefetchView
Volatility - PrefetchParser
`PECmd.exe -d <Extracted Directory> --csv <outputdirectory>`
`PECmd.exe -f <prefetchfile>`

# OST Files
 C:/Users/<Username>/AppData/Local/Microsoft/Outlook/<User>@<Domain>.ost
 
Windows VM - Kernel OST Viewer (View only without paid version)
Windows VM/SFIT - pffexport - Text `pffexport -q -m all <file>

# Officemacros
List all oledump.py <filename>
Deobfuscate compression oledump.py -v <filename> -s <matching index from oledump>

# WMI Event Consumers
Ref - https://blackhat.com/docs/us-14/materials/us-14-Kazanciyan-Investigating-Powershell-Attacks-WP.pdf
Ref - https://cyberforensicator.com/2019/07/13/using-mitre-attck-for-forensics-wmi-event-subscription-t1084/

Location - C:\WINDOWS\system32\wbem\Repository

Objects are stored in objects.data, if the system is disk only, and the analyst has not captured the output of `Get-WMIObject` you're next best place to look is running `strings -le` over objects.data
The following strings are a good start

1. Updater (For powersploit)
2. CommandLineEventConsumer.Name (Excluding CommandLineEventConsumer.Name="BVTConsumer")
3. Powershell.exe
4. Cmd.exe
5. -ExecutionPolicy
6. NonInteractive
7. AMSIBypass

There is also a tool `wmi-parser.exe' -i .\OBJECTS.DATA` This tool will parse the data file for any consumers, that might not be stored as MOF's

