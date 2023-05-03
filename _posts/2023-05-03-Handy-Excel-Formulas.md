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

`=TEXT(A1,"YYYY-MM-DDTHH:MM:SS")`

## Convert from ISODate (ISO 8601) to DATE 

`=DATEVALUE(MID(A1,1,10))+TIMEVALUE(MID(A1,12,8))`

## Convert from DATE to ISODate (ISO 8601)

`=TEXT(A1,"yyyy-mm-ddThh:MM:ss")`

## Change timezone

`=A1+TIME(<TZ>,0,0)`

## Join two columns together with spaces between
`=CONCAT(A1," "A2).`

## Lookup Value from another CSV/XLSX
`=XLOOKUP(A1,['sheet2.xlsx]!$B:$B,['sheet2.xlsx]!$C:$C,"Not in Sheet2")`

### Values Defintion

Value | Description
-------|--------
`A1`	| Lookup Cell
`Sheet2.xlsx`	| XLSX to search
`$B` | Column to query
`$C` | Column to return

## Min and Max date lookups based on a given field (Using M365/Excel 2019+)

Earliest Date

`=MIN(FILTER(<DateArray>,<LookupArray>=<CellToLookUp>))`

Latest Date

`=MAX(FILTER(<DateArray>,<LookupArray>=<CellToLookUp>))`

*Note: you must have all the date types as text for this to work*

 
