---
categories:
  - blog
title: DFIR Playbook - Network Forensics
subtitle: An extract from my Physical Playbook
tags: [dfir, network, pcap, netflow]
comments: false
header:
  teaser: /img/net/network.png
---

# Introduction
This post aims to replicate my physical playbook on Networking and includes the following tools

    - tshark
    - capinfos
    - Network Miner
    - editcap
    - nfdummp
    - bro
    - passivedns
    - zcat

Future editions will include Snort and Live Monitoring

# Contents
- [Introduction](#introduction)
- [Contents](#contents)
- [Converting PCAPs](#converting-pcaps)
- [Analysing PCAPs](#analysing-pcaps)
- [Netflow](#netflow)
- [Snort](#snort)


# Converting PCAPs

From|To|Command
----|--|-------
pcap|netflow|`nfcapd -r <pcapfile> -S 1 -z -l <Outputdirectory>`
pcap|zeek| `bro <profile> -r <pcapfile>` *profiles listed in `/opt/bro/share/bro/site/<name>.bro`*
pcap|dns| `passivedns -r <pcapfile> -l dnslog.txt -L nxdomain.txt` *Not included in SIFT, see [repo](https://github.com/gamelinux/passivedns)*
pcapng|pcap| `tcpdump -r <pcapngfile> -w pcap.pcap`
gz|grep'able text| `zcat <gzfile>`

[*Back to table of contents*](#Contents)

# Analysing PCAPs

Description|Command
-----------|-------
HTTP Packet Counter|`tshark -r <pcapfile> -z http,tree -q`
Info | `capinfos <pcapfile>`
Basic filter output | `tshark -r <pcapfile> -Y '<filters>'` *See [Wireshark wiki](https://wiki.wireshark.org/DisplayFilters) or [Unit42](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/) for filter examples* <br> *Note:tshark uses wireshark filters*
Get files from pcap | `tcpflow -r <pcapfile> -o <outputdirectory>` or `networkminer` <br> *Note: see [Netresec](https://www.netresec.com/?page=Blog&month=2014-02&post=HowTo-install-NetworkMiner-in-Ubuntu-Fedora-and-Arch-Linux) for network miner installation instructions
Splice PCAP by before time | `editcap -B <YYYY-MM-DD HH:MM:SS Before Time> <pcapfile> spliced.pcap`
Splice PCAP by after time | `editcap -A <YYYY-MM-DD HH:MM:SS After Time> <pcapfile> spliced.pcap`
Forensics Analysis | `tshark -r <pcap file> -Y <display filters> -T fields -e <Fields To display>` <br> Can be combined with `|sort|uniq -c | sort -nr` for statistical analysis <br> **Fields** Use one `-e` for each field, examples include `ip.addr` `udp` `frame.number` or to show protocol fields from wireshark use `_ws.col` for example `_ws.col.info` or `_ws.col.dns.query`. To print all available fields use `tshark -G fields` or see [Wireshark documentation](https://www.wireshark.org/docs/dfref/)
Filter pcaps (Reduce them down) | `tcpdump -n -r <pcapfile> -w out.pcap <filter>` <br> filter could be `udp and port 53` for DNS traffic see [TCP Dump filters](http://alumni.cs.ucr.edu/~marios/ethereal-tcpdump.pdf) for more examples
Dump netflow | `nfdump -R <inputdirectory> <options> <filter> -o fmt.<format string>` <br> see [572 Poster](https://digital-forensics.sans.org/media/DFPS_FOR572_v1.6_4-19.pdf) for usage

# Netflow
1. WIP

# Snort
2. WIP

[*Back to table of contents*](#contents)