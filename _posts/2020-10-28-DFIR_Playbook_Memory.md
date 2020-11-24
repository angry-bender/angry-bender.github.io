---
categories:
  - blog
title: DFIR Playbook - Memory Analysis
subtitle: An extract from my Physical Playbook
tags: [dfir, memory, process, malware, rootkit, windows, linux]
comments: false
header:
  teaser: /img/mem/mem.jpg
---

# Introduction
This post aims to replicate my physical playbook on Memory Analysis and includes the following tools

    - Volatility
    - Strings -el

# Contents
- [Introduction](#introduction)
- [Contents](#contents)
- [Windows Overlay Updates](#windows-overlay-updates)
- [Analysis Tasks](#analysis-tasks)
  - [Determine profile](#determine-profile)
  - [Quick IOC Wins (Get the files, dump the files, scan the files)](#quick-ioc-wins-get-the-files-dump-the-files-scan-the-files)
  - [Analyse processes](#analyse-processes)
  - [Analyse System](#analyse-system)
  - [Analyse Network](#analyse-network)
  - [Code Injection](#code-injection)
  - [Volatility Timelines](#volatility-timelines)
- [Generating a Linux profile](#generating-a-linux-profile)
  - [Pre-Reqs](#pre-reqs)
  - [Create vtypes / dwarf module](#create-vtypes--dwarf-module)
  - [Get the system.map](#get-the-systemmap)
  - [Making a profile](#making-a-profile)
  - [Basic Linux volatility commands](#basic-linux-volatility-commands)

# Windows Overlay Updates
1.  By default, the packaged version of volatility does not come with the latest volatility profiles, to fix this conduct the following steps;
- `git clone https://github.com/volatilityfoundation/volatility.git` for the latest version to your home directory
- `cd ~/volatility/volatility/plugins/overlays/windows`
- use the following command to copy the profiles across `sudo cp -r * /usr/local/lib/python2.7/dist-packages/volatility/plugins/overlays/windows`
- confirm you have the latest profiles with `vol.py --info | grep -i win10`
- - You should at least have the `Win10x64_18362        - A Profile for Windows 10 x64 (10.0.18362.0 / 2019-04-23)` profile

[*Back to table of contents*](#contents)

# Analysis Tasks

## Determine profile
1. If you cannot determine the profile, by checking the `HKLM\Software\Microsoft\Windows NT\CurrentVersion\BuildLab` registry key, then use the following commands

Order|Command|Description
-|-----|-----
1 | `vol.py -f <imagename> imageinfo` | Gives the image info for a given file
2 | `vol.py -f <imagename> kbdgscan` | Scans the image for matching memory headers

2. Once you have confirmed your profile test it with a simple psscan `vol.py -f <imagename> --profile <profile> psscan`. If this command runs free from error's you have a successful profile. If you are having issues, check and search [Volatility Issues](https://github.com/volatilityfoundation/volatility/issues) to see if others are having the same issue

[*Back to table of contents*](#contents)

## Quick IOC Wins (Get the files, dump the files, scan the files)
1. firstly, make a dump directory `mkdir dump`
2. conduct the following steps

Order|Command|Description
-|-----|-----
1 | `vol.py -f <imagename> --profile <profile> procdump --dump-dir=<DUMPFILEDIR>` | Dumps the running processes
2 | `vol.py -f <imagename> --profile <profile> dlldump --dump-dir=<DUMPFILEDIR>` | Dumps called DLL's from processes
3 | `vol.py -f <imagename> --profile <profile> moddump --dump-dir=<DUMPFILEDIR>` | Dumps drivers called from processes

4. Once this has been done, you can use either sophos for linux (seems to work the best) or clamav, ensuring you have updated your patterns with the most recent update `clamscan -r --bell -i <DUMPFILEDIR>`

[*Back to table of contents*](#contents)

## Analyse processes
1. All the commands below use `volatility -f <imagename> --profile <profile>` as a prefix, the table below, describes each option used for command line

Option|Description
-----|-----
`psscan` | Shows all running processes, PID, PPID, Time Created, Time Exited
`pstree` | Shows all parent processes visually
`malprocfind -x` | shows malicious processes, `-x` includes closed processes
`malfind` | Finds hidden an injected code (You can add `--dump-dir=<dir>` for quick wins with a virus scanner)
`psslist` | Shows name, ppid, started, handle counts but **Rootkits are invisible**
`dlllist` | Shows loaded dll's, can also be useful for quickly seeing the command line uses for a process
`psxview` | Comapre output to find hidden process **Hidden processes shows `false` in the first 2 columns**
`shimcachemem` | Pulls the shimcache (*Windows application compatibility database*) from memory
`autoruns` | Scans memory for persistance
`handles -s -t <Any combination of file,key,mutant,event,thread> -p <pid>` | Allows you to view all the handles of a given process
`svcsan` | Shows windows services
`threads` | Shows threads and loaded DLL's
`memdump -p <pid>` | dumps the memory of a single process
`getsids -p <pid>` | Lists the users, groups permissions and type of process, to ascertain what permissions its running as, or, who has launched it
`cmdline -p <pid>` | shows the command line used for an application 

[*Back to table of contents*](#contents)

## Analyse System
1. All the commands below use `volatility -f <imagename> --profile <profile>` as a prefix, the table below, describes each option used for command line

Option|Description
-----|-----
`cmdscan` | Shows command history
`consolescan` | shows console information or history
`file-scan` | shows files opened in memory
`dumpregistry -D <outputdirectory>` | dumps the windows registry from memory

1. If all else fails, you can also use `strings -el <filename>` accross the image to find a given string with `grep` etc

[*Back to table of contents*](#contents)

## Analyse Network
1. All the commands below use `volatility -f <imagename> --profile <profile>` as a prefix, the table below, describes each option used for command line
  
Option|Description
-----|-----
`netscan | egrep -i 'CLOSE|ESTABLISHED|OFFSET'` | Shows network IP, Ports, filter for active and closed TCP Connections
`sockets` | Shows running network sockets
`conscan` | Shows TCP Connections

[*Back to table of contents*](#contents)

## Code Injection
1. All the commands below use `volatility -f <imagename> --profile <profile>` as a prefix, the table below, describes each option used for command line

Option|Description
`malfind --dumpdir=<outputdir>` | common yara rule to dump malware with common IOC's
`ldrmodules -p<pid>` | detect unlinked dll's and non-memory mapped files
`hollowfind` | detect evidence of known memory hollowing techniques
`threadmap` | Detect threads to identify hollowing countermeasures
`SSDT` | identifies hooking kernel modules outside of the norm
`psxview` | Review hidden process
`modscan` | Loaded drivers and kernel modules
`apihooks -p <pid>` Can show api hooks used by espionage malware or rootkits see [Rootkit-Investigation-Procedures](https://www.sans.org/score/checklists/rootkits-investigation-procedures)

[*Back to table of contents*](#contents)

## Volatility Timelines
1. `vol.py -f <file> --profile=<profile> <timeliner|mftparser|shellbags> --output=body > bodyfile.txt`
2. `mactime -b bodyfile.txt -d -y > timeline.csv`

# Generating a Linux profile

## Pre-Reqs
1. Firstly, ensure you have `dwarfdump` installed on your system
2. `sudo apt-get install build-essential kernel-devel linux-headers-generic` *You will need to match the kernel to the source system*

## Create vtypes / dwarf module
Vtypes are the kernels data structures in memory.
1. `cd <volatilitydir>/tools/linux`
2. `make`
3. `head module.dwarf`

## Get the system.map
This should be located on the boot drive as `/boot/System.map<Version>`

## Making a profile
1. Copy the module.dwarf file and system.map into a zip file `zip <LinuxRelease-Version>.zip module.dwarf system.map`
2. copy the zip file to `<volatilitydir>/plugins/overlays/linux`
3. See if the profile is working with `vol.py --info | grep Linux` and see if your names version is there

## Basic Linux volatility commands
```
linux_arp           - Print the ARP table
linux_bash          - Recover bash history from bash process memory
linux_check_afinfo  - Verifies the operation function pointers of network protocols
linux_check_creds   - Checks if any processes are sharing credential structures
linux_check_fop     - Check file operation structures for rootkit modifications
linux_check_idt     - Checks if the IDT has been altered
linux_check_modules - Compares module list to sysfs info, if available
linux_check_syscall - Checks if the system call table has been altered
linux_cpuinfo       - Prints info about each active processor
linux_dentry_cache  - Gather files from the dentry cache
linux_dmesg         - Gather dmesg buffer
linux_dump_map      - Writes selected memory mappings to disk
linux_find_file     - Recovers tmpfs filesystems from memory
linux_ifconfig      - Gathers active interfaces
linux_iomem         - Provides output similar to /proc/iomem
linux_lsmod         - Gather loaded kernel modules
linux_lsof          - Lists open files
linux_memmap        - Dumps the memory map for linux tasks
linux_mount         - Gather mounted fs/devices
linux_mount_cache   - Gather mounted fs/devices from kmem_cache
linux_netstat       - Lists open sockets
linux_pidhashtable  - Enumerates processes through the PID hash table
linux_pkt_queues    - Writes per-process packet queues out to disk
linux_proc_maps     - Gathers process maps for linux
linux_psaux         - Gathers processes along with full command line and start time
linux_pslist        - Gather active tasks by walking the task_struct->task list
linux_pslist_cache  - Gather tasks from the kmem_cache
linux_pstree        - Shows the parent/child relationship between processes
linux_psxview       - Find hidden processes with various process listings
linux_route_cache   - Recovers the routing cache from memory
linux_sk_buff_cache - Recovers packets from the sk_buff kmem_cache
linux_slabinfo      - Mimics /proc/slabinfo on a running machine
linux_tmpfs         - Recovers tmpfs filesystems from memory
linux_vma_cache     - Gather VMAs from the vm_area_struct cache
```

[*Back to table of contents*](#contents)
