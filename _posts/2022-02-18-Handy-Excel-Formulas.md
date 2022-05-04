---
categories:
  - blog
title: Handy DFIR Excel Formulas
subtitle: Excel formulas for DFIR.
tags: [Timeline, DFIR]
comments: true
header:
  teaser: /img/excel.png
---

# Introduction

We all know DFIR loves spreadsheets for timelines, but copying out times, dates and converting data types can be a pain. This post aims to show the formulas I commonly use in excel to help this out

## Date wont convert when doing formulas, or sort not working

`=TEXT(A2,"YYYY-MM-DDTHH:MM:SS")`

## Convert from ISODate (ISO 8601) to DATE 

`=DATEVALUE(MID(A1,1,10))+TIMEVALUE(MID(A1,12,8))`

## Convert from DATE to ISODate (ISO 8601)

`=TEXT(A1,"yyyy-mm-ddThh:MM:ss")`

## Change timezone

`=MOD(A2+(<Zone Offset>/24),1)`
