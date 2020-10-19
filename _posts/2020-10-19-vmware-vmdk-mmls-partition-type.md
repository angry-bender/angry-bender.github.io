---
categories:
  - blog
title: Vmware VMDK mmls partition type
subtitle: For when VMWARE Cannot detect the partition type
tags: [oscp, offensive]
comments: false
header:
  teaser: /img/vmv/disk.jpg
---

# Introduction
How to convert vmdk's that might be compressed when you get the error
```bash
abender@sift:~$ mmls sample.vmdk
Cannot determine partition type
abender@sift:~$ mmls -afflib sample.vmdk
Cannot determine partition type
```

# Credits
Credits for this discovery go to dave... you know who you are ;-)


# Instructions
-  It appears that the hex value of `ffff ffff ffff ffff`will show under xxd if a vmdk is compressed, but file, shows know difference
abender@sift:~$ xxd sample.vmdk | head -n 4

```bash
... # output suppressed ...
00000030: 0000 0000 0000 0000 ffff ffff ffff ffff
```
- So if the `ffff ffff ffff ffff`is present, you can convert it with virtualbox (you will need to install it first)
   
```bash
abender@sift:~$ vboxmanage clonehd --format vmdk sample.vmdk sample-flat.vmdk
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Clone medium created in format 'vmdk'. UUID 12844321-9000-4a2c-0701ff223241
abender@sift:~$ 
```
-  Now try mmls again
   
```bash
abender@sift:~$ mmls sample.vmdk
Cannot determine partition type
abender@sift:~$ mmls -afflib sample.vmdk
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot      Start               End                Length
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  ---------   0000000000   0068388704   0068388705   Unallocated
02:  00:00   0068388705   0078156224   0009767520   Linux (0x83)
03:  ---------   0078156225   0078165359   0000009135   Unallocated
```
-  If this doesnt work, you can also try `qemu-img` to convert to raw