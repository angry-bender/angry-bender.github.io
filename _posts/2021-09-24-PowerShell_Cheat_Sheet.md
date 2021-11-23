---
categories:
  - blog
  - 
title: PowerShell Cheat Sheet
subtitle: Handy tips for Powershell I use all the time
tags: [windows, sysadmin, automation]
comments: false
header:
  teaser: /img/ps.jpg
---

## Introduction
Script blocks i find myself using in powershell all the time

## Contents

- [Introduction](#introduction)
- [Contents](#contents)
- [Creating your own object](#creating-your-own-object)
- [Getting FilePath and parents for a file type](#Getting-FilePath-and-parents-for-a-file-type)

[*Back to table of contents*](#contents)

## Creating your own object

This creates the following table

```PowerShell
$object = @()
$object += New-Object -TypeName psobject -Property @{Name1="Value1"; Name2="Value2"}
$object += New-Object -TypeName psobject -Property @{Name1="Value3"; Name2="Value4"}

```

Name1 | Name2
----- |  -----
Value1 | Value2
Value3 | Value4

[*Back to table of contents*](#contents)


## Getting FilePath and parents for a file type

This can be used for bulk computer collections and processing with other tools.
```PowerShell
$SourceDir = C:\
$FileDir = Get-ChildItem -Recurse -Path $SRUMDir -Filter '<FileSearchingFor>'

foreach ($dir in $FileDir){
  $ParentDirectory = (Get-Item FileDir.Directory).parent.FullName
  #You would do something with the file here, in this case I'll use a write-host
  Write-Host "Doing something with $dir.FullName
 }
```

[*Back to table of contents*](#contents)
