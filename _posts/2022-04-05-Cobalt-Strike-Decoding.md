---
categories:
  - blog
title: Cobalt Strike Decoding
subtitle: Quick wins on cobalt strike config.
tags: [Malware, Windows, DFIR]
comments: true
header:
  teaser: /img/cs.png
---

# Introduction
This post aims to bring together some resources for quick wins to get cobalt beacons.

## Not reinventing the wheel

The below Sophos post does such a great job at explaining the process. 

[Sophos - Decoding malicious powershell](https://community.sophos.com/sophos-labs/b/blog/posts/decoding-malicious-powershell)


## TLDR Quick Wins

Check your base64 against this cheatsheet [base64 cheatsheet](https://gist.github.com/Neo23x0/6af876ee72b51676c82a2db8d2cd3639)

Some beacons like to use a %COMSPEC% service with encoded powershell that looks something like 
`%COMSPEC% /b /c start /b /min powershell -nop -w hidden -encodedcommand <Base64Here>`

Plugging this into cyberchef with the following recipie should show the next stage of the config
[Recipie1](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)Remove_null_bytes())

In the case there is some more obfuscation here, and it is compressed try the following recipie
[Recipe2](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)Gunzip())

Now you should see the config with a `$DoIt = @` at the top after `Set-StrictMode version 2`

Scrolling down, you may see `[Byte[]]$var_code = <another base64 string>`

Copy out this base64 string and place into this recipie (Take note of the `bxor <number>` below the base64 command, this contains the decimal xor string you might need)
[recipie3](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)XOR(%7B'option':'Decimal','string':'35'%7D,'Standard',false))

This will output the URL or a Named Pipe, if you save this file (Windows Defender will block it) as a .bin file you can use [scdbg](http://sandsprite.com/CodeStuff/scdbg.zip) to see the windows API Calls

`scdbg /f download.bin`
