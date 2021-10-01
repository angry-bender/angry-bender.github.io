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

This post aims to consolidate a list of useful smartphone codes

## Contents

- [Introduction](#introduction)
- [Contents](#contents)
- [Creating your own object](#creating-your-own-object)

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