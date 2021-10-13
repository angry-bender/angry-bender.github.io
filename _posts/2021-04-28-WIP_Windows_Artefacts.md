---
categories:
  - blog
title: DFIR Playbook - Windows Forensics(WIP APR21)
subtitle: An extract from my Physical Playbook
tags: [dfir, windows]
comments: false
header:
  teaser: /img/zt.jpg
---
# Last Updated

15JUL21

# Introduction

*note this post is incomplete, apr 2021, this is quite a large playbook to replicate*

This post aims to replicate my physical playbook on windows. Unlike other playbooks, it is not tool centric, rather it is concept/artefact centric.

# Scripted Environment Setup

All these tools are self contained in

Linux VM - [SANS SIFT](https://digital-forensics.sans.org/community/downloads)

Windows VM - [WINSIFT](https://github.com/angry-bender/forensicssetup)

# Contents

- [Last Updated](#last-updated)
- [Introduction](#introduction)
- [Scripted Environment Setup](#scripted-environment-setup)
- [Contents](#contents)
- [Fundamentals](#fundamentals)
  - [Event log Parsing](#event-log-parsing)
  - [Registry Parsing or locations](#registry-parsing-or-locations)
  - [OST Files](#ost-files)
- [Malware Discovery](#malware-discovery)
  - [Static Executable / Script Searches](#static-executable--script-searches)
    - [Packed or encrypted](#packed-or-encrypted)
    - [Unsigned](#unsigned)
    - [Base64 Searching](#base64-searching)
  - [Persistance](#persistance)
    - [WMI Event Consumers](#wmi-event-consumers)
      - [WMI Data](#wmi-data)
      - [WMI Events](#wmi-events)
      - [Non-Normal WMI Activity](#non-normal-wmi-activity)
    - [Normal WMI Event consumers](#normal-wmi-event-consumers)
    - [Registry](#registry)
  - [Non-Standard Behavior](#non-standard-behavior)
  - [Useful Eventlogs](#useful-eventlogs)
    - [System](#system)
    - [Security](#security)
  - [Email Attachments](#email-attachments)
    - [Officemacros](#officemacros)
- [Activity Discovery](#activity-discovery)
  - [Process Activity](#process-activity)
    - [Prefetch Files](#prefetch-files)
    - [Application Experience Service (Amcache)](#application-experience-service-amcache)
  - [User Activity](#user-activity)
    - [Shellbags](#shellbags)
    - [Shimcache](#shimcache)
    - [Windows 10 Notifications](#windows-10-notifications)

# Fundamentals

## Event log Parsing

- Log Locations
  - Vista+\2008+ `%SYSTEMROOT%\winevt\logs`
  - Log Location XP\2003- `%SYSTEMROOT%\config

- Tools
  - Event Log explorer
  - Evtxexport (linux)
  - `EvtxEcmd.exe -f <filename> --csv <outputdirectory>`

## Registry Parsing or locations

- Tools
  - Registry Explorer
  - rip.pl
  - `.\RECmd.exe --bn .\BatchExamples\RECmd_Batch_MC.reb --csv <outputdirectory> -d '<ntuser.dat location>` | Decodes all user artefacts

- Location
  - `%SYSTEMROOT%\config`
- Correlating hives

Filename | Hive | Notes
---------|------|------
Sam | HKLM\SAM | -
Security | HKLM\SECURITY | -
Software | HKLM\SOFTWARE | -
System | HKLM\SYSTEM | Current control set at HKLM\SYSTEM\Select\Current

- Location
  - `%USERPROFILE%\Ntuser.dat`
- Correlating hives

Filename | Hive
----------|----------
Ntuser.dat | HKCU (Of current profile)  
%USERPROFILE%\Local Settings\Application Data\Microsoft\Windows\Usrclass.dat | XP\2003- HKCU All Users
%USERPROFILE%\AppData\Local\Microsoft\Windows\Usrclass.dat | Vista\2008+  HKCU All Users

## OST Files

- Location
  - C:/Users/<Username>/AppData/Local/Microsoft/Outlook/<User>@<Domain>.ost
- Tools
  - Windows VM - Kernel OST Viewer (View only without paid version)
  - Windows VM/SFIT - pffexport - Text `pffexport -q -m all <file>

# Malware Discovery

## Static Executable / Script Searches

### Packed or encrypted

- Tools
  - DensityScout - `densitycout -r -s exe,dll,sys -P0.1 -o <outputfile> <scandirectory>`

This tool is useful to find the entropy of a file, however is likely to output a lot of false positives. Often malicious executables use packing or encryption to make them harder to reverse engineer in a static environment.

One point to note, regardless of tools is entropy values. In short, these can be defined as below;

Description | Value
------------|------
Encrypted | >= 7
Compressed | == 6.7
Native Program | 5.0
English Text | 4.0

### Unsigned

In the case a vendor hasnt been pwned by a supply chain attack, a good hunting mechanism could be to check for unsigned file signatures.

From my point of view, the most value here would be to check for any executable in the windows directory and scan those. This could be acheived through powershell

1. Mount you're image
2. cd to <Systemroot>:\Windows
3. Execute the following mini script

```$exes Get-ChildItem -Recurse -Include *.exe
foreach($i in $exes)
{
    cp $i.fullname <destination Directory>
}

```

4. run sigcheck -c -e -u -h -vrs <destination directory> > results.csv

If you didnt want to copy each file, you could try the below (untested)

```$exes Get-ChildItem -Recurse -Include *.exe
foreach($i in $exes)
{
    sigcheck -c -e -u -h -vrs $i.fullname >> results.csv
}
```

### Base64 Searching

- Tools
  - `recmd.exe -d <windows root> --base64 <MinLength (Reccomended 300)>`

A quick win for finding powershell or other types of scripts. Often, powershell scripts are executed by base64 either to obfuscate their true purpose or to send bits over a text only channel such as a webserver, where the use of symbols may cut a malicious script short.

## Persistance

- Tools
  - `recmd.exe -d <windows root> -bn <Registry Explorer root\batchexamples\registryASEPS.reb --ml --csv <outputdiretory>`

  Note: If using [WINSIFT](https://github.com/angry-bender/forensicssetup) this directory is contained at `C:\NonChocoTools\ZimmermanTools\Registry Explorer\`

  This particular command extracts 500 of the most common persistance keys in windows. Its also likely used by KAPE if thats you're prefered method

  - `autorunsc.exe /accepteula -a * -c -h -s '*' -nobanner` - Note: In my experience this often doesnt catch WMI Event comsumers
  
  - the Kroll persitance script for registry from <https://github.com/EricZimmerman/RECmd/blob/master/BatchExamples/Kroll_Batch.reb>
  --`recmd.exe -d <windows root> -bn <Registry Explorer root\batchexamples\kroll_batch.reb --ml --csv <outputdiretory>`

### WMI Event Consumers

#### WMI Data

Ref - <https://blackhat.com/docs/us-14/materials/us-14-Kazanciyan-Investigating-Powershell-Attacks-WP.pdf>

Ref - <https://cyberforensicator.com/2019/07/13/using-mitre-attck-for-forensics-wmi-event-subscription-t1084/>

- Location
  - C:\WINDOWS\system32\wbem\Repository

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

#### WMI Events

- Location
  - C:\Windows\system32\winevet\logs\Microsoft-Windows-WMI-Activity-Operational
- evtx for the following event id's may be useful to find persistance or execution of wmi

Event ID | Description | Interpretation Notes
---------|-------------|---------------------
5857 | Time of wmiprvse execution and dll usage | -
5860 or 5861 | Registration of event consumers | Event Filter - condition (method of execution) <br>  Event Consumer - The action (what action to take) <br>  Bound-filter - The link  (the selection is active)

#### Non-Normal WMI Activity

1. There is a wmiprvse.exe running without a PPID of svchost.exe
2. scrons.exe is running
  
### Normal WMI Event consumers

Ref - <https://support.sophos.com/support/s/article/KB-000038535?language=en_US&c__displayLanguage=en_US>
  
The SCM Event consumer, is commonly present in most enterprise environments. However, is also modified by attackers. Here is an example of a known good, which can be found in objects.data or EID `5861`
  
  ```
           CreatorSID={1,2,0,0,0,0,0,5,32,0,0,0,32,2,0,0}
           EventAccess=
           EventNamespace=root\cimv2
           Name=SCM Event Log Filter
           Query=select * from MSFT_SCMEventLogEvent
           QueryLanguage=WQL

          Category=0
          CreatorSID={1,2,0,0,0,0,0,5,32,0,0,0,32,2,0,0}
          EventID=0
          EventType=1
          InsertionStringTemplates={""}
          MachineName=
          MaximumQueueSize=
          Name=SCM Event Log Consumer
          NameOfRawDataProperty=
          NameOfUserSIDProperty=sid
          NumberOfInsertionStrings=0
          SourceName=Service Control Manager
          UNCServerName=
 
          Consumer="NTEventLogEventConsumer.Name="SCM Event Log Consumer""
          CreatorSID={1,2,0,0,0,0,0,5,32,0,0,0,32,2,0,0}
          DeliverSynchronously=FALSE
          DeliveryQoS=
          Filter="__EventFilter.Name="SCM Event Log Filter""
          MaintainSecurityContext=FALSE
          SlowDownProviders=FALSE

  ```

### Registry

Some of the most common keys are as follows

1. `HKLM` or `HKCU` (See [Registry Parsing](#registry-parsing)) Microsoft\CurrentVersion\Run
2. `HKCU` Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders
3. `HKLM` or `HKCU` Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run

## Non-Standard Behavior

This is *highly* dependent on you're environment but some IOC's could be

1. Using syswow64 for execution of cmd or powershell on a 64 bit system
   1. Particularly within scheduled tasks
2. Using powershell v2

## Useful Eventlogs

### System

- Location C:\Windows\system32\winevet\logs\system.evtx

Event ID | Description | Type
---------|-------------|---------------------
7034 | Crashed unexpectedly | Services
7035 | Start Stop Control | Services
7040 | Start Type Changed | Services
7045 | New Service | Services
1 | Kernel General | Time Manipulation

### Security

- Location C:\Windows\system32\winevet\logs\security.evtx
  
Event ID | Description | Type
---------|-------------|---------------------
4672 | Successful login (Check for Type 3 or 10) <br> Requirement for $admin, $ipc, $c share access and rdp | Network
4616 | System Time Changed | Time Maniupulation

## Email Attachments

### Officemacros

- Tools
  - List all oledump.py <filename>
  - Deobfuscate compression oledump.py -v <filename> -s <matching index from oledump>

# Activity Discovery

## Process Activity

### Prefetch Files

- Location
  - C:\Windows\Prefetch
- Tools
  - WinPrefetchView
  - Volatility - PrefetchParser
  - `PECmd.exe -d <Extracted Directory> --csv <outputdirectory>`
  - `PECmd.exe -f <prefetchfile>`
- Malicious IOC's
  - Syswow64 on a 64 bit system
  - Suspicious names (svxhost.exe)
  - Unusual Etensions (malz.pdf.exe)

### Application Experience Service (Amcache)

Try to use this befre using the app compatability cache, as it may provide better results

- Location
  -C:\windows\appcompat\programs\amcache.hve
- Tools
  - `amcacheparser.exe -f <hive file> --csv <output file>`
  - Registry Explorer

## User Activity

### Shellbags

Can use Ntuser.dat, but, userclass.dat is more verbose.

### Shimcache

Location \%SystemRoot%\AppCompat\Programs\Amcache.hve
HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatibility\AppCompatCache

### Windows 10 Notifications

Credit - <https://www.forensafe.com/blogs/win10notifications.html>

Location %Userproile%\AppData\Local\Microsoft\Windows\Notifications

This location is stored as an Sqlite database, you will need to use Sqlitebrowser to decode it.

Microsoft also stores information about notifications in the following registry key: HKCU\Software\Microsoft\Windows\CurrentVersion\PushNotifications
