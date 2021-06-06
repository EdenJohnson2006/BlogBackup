---
layout: post
title: 在任意有TWRP 有SD卡的手机上安装Debian（在Android设备上原生运行Debian实操）
date: 2021-01-15 07:07:12
tags:
hide: true
---

# 概述

最近在404上搜索到了这篇文章: [在Android设备上原生运行Debian](https://medium.com/@quantvc/running-debian-on-android-device-natively-73545c9b0757) （需要科学上网，当然你也可以看翻译）

然后我就准备对这部OPPO R9开刀（）

## 基本信息

电脑：

​	系统：ArchLinux

```
den@eden-arch  ~  neofetch 
                   -`                    eden@eden-arch 
                  .o+`                   -------------- 
                 `ooo/                   OS: Arch Linux x86_64 
                `+oooo:                  Host: TM1704 XMAKB3M0P130D 
               `+oooooo:                 Kernel: 5.4.87-1-lts 
               -+oooooo+:                Uptime: 8 hours, 13 mins 
             `/:-:++oooo+:               Packages: 1406 (pacman) 
            `/++++/+++++++:              Shell: zsh 5.8 
           `/++++++++++++++:             Resolution: 1920x1080 
          `/+++ooooooooooooo/`           DE: Plasma 5.20.5 
         ./ooosssso++osssssso+`          WM: KWin 
        .oossssso-````/ossssss+`         WM Theme: Aritim-Dark 
       -osssssso.      :ssssssso.        Theme: Aritim-Dark [Plasma], Breeze [GTK2/3] 
      :osssssss/        osssso+++.       Icons: breeze [Plasma], breeze [GTK2/3] 
     /ossssssss/        +ssssooo/-       Terminal: konsole 
   `/ossssso+/:-        -:/+osssso+-     CPU: Intel i3-8130U (4) @ 3.400GHz 
  `+sso+:-`                 `.-/+oso:    GPU: Intel UHD Graphics 620 
 `++:.                           `-/+/   Memory: 3181MiB / 7873MiB 
 .`                                 `/
```

手机：

​	设备：OPPO R9tm

​	状况：已root

​	SoC：联发科曦力P10（MT6755）

```
shell@R9:/ $ uname -a                                                      
Linux localhost 3.18.22+ #1 SMP PREEMPT Sat Oct 21 16:28:31 CST 2017 aarch64
shell@R9:/ $ cat /proc/cpuinfo                                                 
Processor       : AArch64 Processor rev 2 (aarch64)
processor       : 0
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

processor       : 1
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

processor       : 2
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

processor       : 3
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

processor       : 4
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

processor       : 5
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

processor       : 6
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

processor       : 7
model name      : AArch64 Processor rev 2 (aarch64)
BogoMIPS        : 26.00
Features        : fp asimd evtstrm aes pmull sha1 sha2 crc32
CPU implementer : 0x41
CPU architecture: 8
CPU variant     : 0x0
CPU part        : 0xd03
CPU revision    : 2

Hardware        : MT6755V/C
```

# 步骤

## 分区SD卡

```
 eden@eden-arch  ~  sudo fdisk -l
Identified face as eden
Disk /dev/sda: 119.24 GiB, 128035676160 bytes, 250069680 sectors
Disk model: SAMSUNG MZNLN128
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: CD1AC5E7-52DD-6548-BAD4-5B55ABDB9549

所用裝置     Start      結束      磁區  Size 類型
/dev/sda1     2048    526335    524288  256M EFI System
/dev/sda2   526336   6817791   6291456    3G Linux swap
/dev/sda3  6817792 250069646 243251855  116G Linux filesystem


Disk /dev/loop0: 310.8 MiB, 325902336 bytes, 636528 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdb: 14.84 GiB, 15931539456 bytes, 31116288 sectors
Disk model: STORAGE DEVICE  
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

所用裝置   可開機 Start     結束     磁區  Size Id 類型
/dev/sdb1          2048 31116254 31114207 14.8G  c W95 FAT32 (LBA)
eden@eden-arch  ~  sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.36.1).                                                                              
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
命令 (m 以獲得說明)：o
Created a new DOS disklabel with disk identifier 0x7cb3cc48.

命令 (m 以獲得說明)：n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
分割區編號 (1-4, default 1): 
First sector (2048-31116287, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-31116287, default 31116287): +512MiB

Created a new partition 1 of type 'Linux' and of size 512 MiB.
Partition #1 contains a vfat signature.

Do you want to remove the signature? [Y]es/[N]o: y

The signature will be removed by a write command.

命令 (m 以獲得說明)：n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
分割區編號 (2-4, default 2): 
First sector (1050624-31116287, default 1050624): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-31116287, default 31116287): 

Created a new partition 2 of type 'Linux' and of size 14.3 GiB.
Partition #2 contains a xfs signature.

Do you want to remove the signature? [Y]es/[N]o: y

The signature will be removed by a write command.

命令 (m 以獲得說明)：w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
eden@eden-arch  ~  sudo mkfs.vfat /dev/sdb1
mkfs.fat 4.1 (2017-01-24)
 eden@eden-arch  ~  sudo mkfs.ext4 /dev/sdb2
mke2fs 1.45.6 (20-Mar-2020)
Creating filesystem with 3758208 4k blocks and 940240 inodes
Filesystem UUID: d6d4d8f4-bbb4-49d4-8e87-8dab26da7712
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done   

```

## 创建一个新的boot.img

先把SD卡插到手机里看信息：

```
root@R9:/ # ls -la /dev/block                                                  
...
brw------- root     root     179,  96 2021-01-13 17:19 mmcblk0rpmb
brw------- root     root     179, 128 2021-01-15 08:00 mmcblk1
brw------- root     root     179, 129 2021-01-15 08:00 mmcblk1p1
brw------- root     root     179, 130 2021-01-15 08:00 mmcblk1p2
...
```

ext4分区是179,130，名称是mmcblk1p2

解包boot

```
eden-arch% chmod +x unmkbootimg 
eden-arch% ./unmkbootimg boot.img 
unmkbootimg version 1.2 - Mikael Q Kuisma <kuisma@ping.se>
Kernel size 7840560
Kernel address 0x40080000
Ramdisk size 2210866
Ramdisk address 0x45000000
Secondary size 0
Secondary address 0x40f00000
Kernel tags address 0x44000000
Flash page size 2048
Board name is "1508577311"
Command line "bootopt=64S3,32N2,64N2"

*** WARNING ****
This image is built using NON-standard mkbootimg!
OFF_KERNEL_ADDR is 0xFC080100
OFF_RAMDISK_ADDR is 0x01000100
OFF_SECOND_ADDR is 0xFCF00100
Please modify mkbootimg.c using the above values to build your image.
****************

Extracting kernel to file zImage ...
Extracting root filesystem to file initramfs.cpio.gz ...
All done.
---------------
To recompile this image, use:
  mkbootimg --kernel zImage --ramdisk initramfs.cpio.gz --base 0x43ffff00 --cmdline 'bootopt=64S3,32N2,64N2' --board '1508577311' -o new_boot.img
---------------

```

额...？

非 标 准 m k b o o t i m g

好吧我得编译一份mkbootimg出来了

草我同步的CM12.1源码呢草

...

在神奇的 [GitHub](https://github.com/osm0sis/mkbootimg) 上我找到了从CM14.1中fork出来的代码，让我们开始编译吧：

**注意！注意！注意！！！因为我这里的基址有问题所以需要重新编译，如果提示是：```This image is built using standard mkbootimg```请忽略这个步骤！你可以下载prebuilt的mkbootimg！**

