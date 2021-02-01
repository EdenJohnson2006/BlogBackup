---
layout: post
title: 在Android设备上原生运行Debian（翻译）
date: 2021-01-14 16:37:58
tags:
---

原文链接（需要科学Networking） https://medium.com/@quantvc/running-debian-on-android-device-natively-73545c9b0757

**注意:本博客下翻译内容遵照WTFPL协议许可,原文可访问: http://www.wtfpl.net/about/**

如果你看不懂,我拿白话跟你解释一下:

> **你他妈的想干嘛就干嘛我不会管你的**

---

去年，在佛罗里达州时，我买了一个不错的中端且相对“开放”的LG Optimus手机。 我root了这部机器，安装了具有最新安全补丁的第三方ROM，以各种可行的方式对其进行了强化，并尽可能将其保留为FLOSS。 它非常适合我的需求，并且陪伴了我很长时间。 几周前，我得到了一个新的OnePlus One，于是我决定将LG手机变成开发设备/橡皮泥（意义未知）。

“为什么不把这款手机变成一台成熟的Linux服务器，并像Raspberry Pi那样在其上运行Web服务？（RPi2：md我是64Bit！）” 这个念头不断涌入我的脑海。 因此，我在9月第一周的大部分时间里都在研究可能性。

关于如何在Android设备上的chroot环境中运行Linux操作系统的教程很多，并且有无数的应用程序（例如Servers Ultimate，PAW Server）可让您在Android上直接运行网络服务器（例如VNC，VPN，SIP， FTP，代理）。 但是，我想要一台真正成熟的Linux服务器（例如Debian）在我的Android手机上无缝运行（chroot模式会限制部分功能比如systemd），这样我可以不受限制地从Debian访问Android操作系统，同时不对Android系统本身进行任何修改（反 向 C h a n g e R o o t）。

我当时在想，也许我可以编写一个新的init程序，在启动时安装一个新的/（指Debian的root），然后在chroot环境中将控制权转移到Android init。 这种方法相对于其他方法的优点： 

- 完整的Debian系统，包括很多轻松可获取的软件包
- 从Debian完全控制Android
- 同时使用Debian和Android
- 可以通过SSH/SFTP从电脑访问Android文件系统
- 通过SSH/SFTP直接访问SD卡，无需卸载
- 更加容易备份整个系统
- 不对Android文件系统造成影响，Android本身对修改一无所知（SafetyNet：我觉得我不行了）
- Android文件系统不再回滚更改，重启后对文件系统的改变不会被回滚（@SELinux）
- Debian的关键文件系统保存在SD卡上，以便在发生重大事故时仍可以轻松访问（比如哪天你闲的没事搞了个rm -rf /）
- 原生的X11UI支持
- 无性能损耗
- 可以针对性的更改Android系统，无需刷机
- 像管理其他Linux一样管理Android

以下是我在这个项目上的笔记：

# 概述

设备：LG Optimus L90 D415 w7 （TM定制（确信））

状况：已解锁BL，已root

SoC：高通骁龙400（MSM8226 S4Series）

 ```bash
~ # uname -a
Linux localhost 3.4.1-AeroKernel+ #1 SMP PREEMPT Tue Oct 21 20:19:09 EDT 2014 armv7l GNU/Linux
~ # cat /proc/cpuinfo
Processor : ARMv7 Processor rev 3 (v7l)
processor : 0
BogoMIPS : 38.40
processor : 1
BogoMIPS : 38.40
processor : 2
BogoMIPS : 38.40
processor : 3
BogoMIPS : 38.40
Features : swp half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt
CPU implementer : 0x41
CPU architecture: 7
CPU variant : 0x0
CPU part : 0xc07
CPU revision : 3
Hardware : Qualcomm MSM 8226 (Flattened Device Tree)
Revision : 0006
Serial : 0000000000000000
 ```



# 步骤

## 给SD卡分区

在Linux电脑上，我将SD卡（至少8GB）分区为两个： 

一个~~胖文件系统~~FAT分区

以及 一个用于Linux的ext3 / ext4分区。 

```bash
＃fdisk -cu /dev/sdc
＃mkfs -t vfat /dev/sdc1
＃mkfs -t ext4 /dev/sdc2 
```

​	

##  创建一个新的initramfs和boot.img

修改后，将Android的Boot.img中的initramfs替换。 

这个替换会使用init从SD卡的Linux分区挂载新的根文件系统，并将控制权转移到该分区。

```bash
$ adb shell 
shell@w7:/ $ mount 
...
/dev/block/vold/public:179_65 on /mnt/media_rw/F409-DD80 type vfat (rw,dirsync,nosuid,nodev,noexec,relatime,uid=1023,gid=1023,fmask=0007,dmask=0007,allow_utime=0020,codepage=cp437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro)
...
/dev/block/vold/public:179_66 on /mnt/media_rw/e0c17d6f-efcd-47eb-9f4e-bc5530f76269 type ext4 (rw,dirsync,context=u:object_r:sdcard_posix:s0,nosuid,nodev,noatime,data=ordered)
...
shell@w7:/ $ ls -la /dev/block
 ...
brw------- 1 root root 179, 64 1970-02-14 03:33 mmcblk1 
brw------- 1 root root 179, 65 1970-02-14 03:33 mmcblk1p1 
brw------- 1 root root 179, 66 1970-02-14 03:33 mmcblk1p2
...
```

在LG L90上，FAT分区/mnt/media_rw/F409-DD80是设备179_65，因此下一个分区应该是为179_66，并且他的名字是mmcblk1p2。 这是我的新initramfs中/init的示例。 它必须命名为/ init，因为它被硬编码到Android内核中才能在启动时执行。

```sh
#!/sbin/busybox sh
# initramfs pre-boot init script
# Mount the /proc and /sys filesystems
/sbin/busybox mount -t proc none /proc
/sbin/busybox mount -t sysfs none /sys
/sbin/busybox mount -t tmpfs none /dev
# Something (what?) needs a few cycles here
/sbin/busybox sleep 1
# Populate /dev
/sbin/busybox mdev -s
# Mount the root filesystem, second partition on micro SDcard
/sbin/busybox mount -t ext4 -o noatime,nodiratime,errors=panic /dev/mmcblk1p2 /mnt/root
# Clean up
/sbin/busybox umount /proc
/sbin/busybox umount /sys
/sbin/busybox umount /dev
# Transfer root to SDcard
exec /sbin/busybox switch_root /mnt/root /etc/init
```

这个initramfs非常简单，仅包含/sbin/busybox和/proc，/sys，/dev和/mnt/root。为了安全起见，您可以使用原始的initramfs（从内核中提取的），只需添加/sbin/busybox和一个挂载点/mnt/root，然后将init替换为上面的脚本即可。 您可以在 (此处)[https://busybox.net/downloads/binaries/latest/] 或Internet上的其他位置下载预编译的busybox。 

我们需要系统的基本地址（下文简称基址），即RAM的起始地址。 要从原始内核zImage中获取它，请在运行的内核中检查/proc/config.gz或使用内核二进制文件上的extract-ikconfig脚本。 如果都不存在/不适用，请尝试在Android设备上的/proc/iomem中寻找“系统RAM”以获取基址的线索。

```shell
root@w7:/ # cat /proc/iomem 
00000000-083fffff : System RAM 
    00008000-0108c71b : Kernel code 
    0120c000-014fd9eb : Kernel data 
0c400000-0d1fffff : System RAM 
0f500000-0f9fffff : System RAM 
0ff00000-3f7fffff : System RAM
```

在这里，基址是00000000。

> 非原文内容：这是一个非00000000开始的示例：
>
> ```shell
> root@R9:/ # cat /proc/iomem 
> 40000000-445fffff : System RAM 
>  40080000-41036193 : Kernel code 
>  410c2000-4146dfff : Kernel data 
> 44640000-bcdaffff : System RAM 
> bcdbe000-bcdbffff : System RAM 
> c0000000-ebffffff : System RAM 
> f4c00000-f5ffffff : System RAM 
> f6150000-fe41ffff : System RAM 
> 100000000-13fdfffff : System RAM 
> 13ff00000-13fffffff : System RAM
> ```

现在，创建新的启动映像。 我将获得此手机的原始启动映像并进行修改。 首先，获取L90w7的CM13 ROM并解压。（注意：选择对应系统刷机包解压！）然后使用unmkbootimg工具，该工具可帮助您解包boot.img。(当然如果你有其他的方式可以自由选择）

在Linux上，运行：

```bash
$ wget http://whiteboard.ping.se/uploads/Android/unmkbootimg.gz 
$ gunzip unmkbootimg.gz
```

然后，将unmkbootimg移动到与CM13同样的目录下，要解包boot，请执行：

```bash
./unmkbootimg boot.img
unmkbootimg version 1.2 - Mikael Q Kuisma <kuisma@ping.se>
Kernel size 8019648
Kernel address 0x8000
Ramdisk size 872992
Ramdisk address 0x1000000
Secondary size 0
Secondary address 0xf00000
Kernel tags address 0x100
Flash page size 2048
Board name is ""
Command line "console=ttyHSL0,115200,n8 androidboot.console=ttyHSL0 user_debug=31 msm_rtb.filter=0x37 androidboot.hardware=qcom androidboot.selinux=enforcing"
This image is built using standard mkbootimg
Extracting kernel to file zImage ...
Extracting root filesystem to file initramfs.cpio.gz ...
All done.
To recompile this image, use:mkbootimg --kernel zImage --ramdisk initramfs.cpio.gz --base 0x0 --cmdline 'console=ttyHSL0,115200,n8 androidboot.console=ttyHSL0 user_debug=31 msm_rtb.filter=0x37 androidboot.hardware=qcom androidboot.selinux=enforcing' -o new_boot.img
```

新建一个新的initramfs目录：

```bash
$ mkdir initramfs && cd initramfs
```

将ramdisk解压（先解压gzip然后解压cpio），将完成的文件放入新的initramfs目录

```bash
$ gzip -cd ../initramfs.cpio.gz | cpio -i	
```

这会将来自ramdisk的所有文件放置在当前工作目录中。 现在，您可以如上所述更改init。 

当修改完成，开始重新创建虚拟磁盘。 重新cpio，然后重新gzip这些文件。 请记住，cpio将打包在当前目录的所有文件，因此您大概率会删除该目录中可能存在的任何其他废项。

```bash
$ find . | cpio --quiet -H newc -o | gzip > ../initramfs.cpio.gz
$ cd ..
```

清理initramfs/目录，只保留initramfs.cpio.gz和zImage。 虽然没有用于分割图像的官方工具，并且它非常琐碎（意义未知），但可以使用许多脚本来执行此操作。 该映像基本上只是内核zImage和initramfs.cpio.gz的串联。 使用可在各个站点进行预编译的Android OS构建工具包mkbootimg，将内核和新的ramdisk合并为完整映像。 另外，您可以按如下所示从源代码进行编译：

```
$ cd /path/to/android-src
$ cd system/core/libmincrypt/
$ gcc -c *.c -I../include
$ ar rcs libmincrypt.a *.o
$ cd ../mkbootimg
$ ls -la
total 36
drwxrwxr-x 2 abc abc 4096 Sep 7 16:51 .
drwxrwxr-x 45 abc abc 4096 Sep 7 16:51 ..
-rw-rw-r-- 1 abc abc 1186 Sep 7 16:51 Android.mk
-rw-rw-r-- 1 abc abc 3266 Sep 7 16:51 bootimg.h
-rw-rw-r-- 1 abc abc 9507 Sep 7 16:51 mkbootimg.c
-rw-rw-r-- 1 abc abc 6379 Sep 7 16:51 unpackbootimg.c
$ gcc mkbootimg.c -o mkbootimg -I../include ../libmincrypt/libmincrypt.a
$ cd ../cpio
$ $ ls -la
total 24
drwxrwxr-x 2 abc abc 4096 Sep 7 16:51 .
drwxrwxr-x 45 abc abc 4096 Sep 7 16:51 ..
-rw-rw-r-- 1 abc abc 313 Sep 7 16:51 Android.mk
-rw-rw-r-- 1 abc abc 8946 Sep 7 16:51 mkbootfs.c
$ gcc mkbootfs.c -o mkbootfs -I../include
```

现在将system / core / mkbootimg / mkbootimg和system / core / cpio / mkbootfs复制到路径中的目录（例如〜/ bin）。现在，你可以编译你的新boot了：

```bash
$ mkbootimg --kernel zImage --ramdisk initramfs.cpio.gz --base 0x0 --cmdline 'console=ttyHSL0,115200,n8 androidboot.console=ttyHSL0 user_debug=31 msm_rtb.filter=0x37 androidboot.hardware=qcom androidboot.selinux=enforcing' -o my-boot.img
```

**注意！！！此处所执行指令应与前文To recompile this image, use后相同，切勿照搬以免发生事故！！！！！**

 编译之后，您应该在当前工作目录中仅看到3个文件：

```
$ ls -la
total 17876
drwxrwxr-x 2 abc abc 4096 Sep 7 17:41 .
drwxr-xr-x 5 abc abc 4096 Sep 7 17:40 ..
-rw-rw-r-- 1 abc abc 1125169 Sep 7 17:39 initramfs.cpio.gz
-rw-r--r-- 1 abc abc 9148416 Sep 7 17:41 my-boot.img
-rw-rw-r-- 1 abc abc 8019648 Sep 7 13:42 zImage
```

内核zImage是您的原始内核。 现在，我们将基于my-boot.img，完成其余的工作。 不要现在就刷入这个Boot！（除非你觉得你很闲想救砖）

## 在SD卡上建立Debian的/

在你的Linux电脑上，将SD卡挂载到/mnt/debian

```sh
mount -t ext4 /dev/sdc2 /mnt/debian
```

然后安装debootstrap，这个看各位的发行版了，Debian直接安装，Ubuntu不清楚，ArchLinux有AUR，其他发行版本自行解决（）

```
debootstrap --verbose --arch aarch64 --foreign buster /mnt/debian https://mirrors.bfsu.edu.cn/debian/
```

当我们为与运行debootstrap的x86（或者x86_64）系统创建不同体系结构的Debian系统时，–arch armel参数用于指示debootstrap创建用于ARM体系结构的Debian基本系统。(如果是armv8请改成AArch64) –foreign指示它仅进行初始拆包，稍后将在实际硬件上进行第二阶段安装。 jessie指示它将软件包下载到运行debootstrap的当前目录中名为“ jessie”的目录中。 最后是存储库URL，可从中获取软件包。 您可以在此处使用本地存储库，但请确保它具有用于体系结构框架的软件包。 如果需要更多信息，请参见debootstrap手册页。

完成此操作后，请卸载SD卡，然后将其重新插入Android手机。

```bash
# umount /mnt/debian
```

重启手机到Recovery Mode并打开adb shell：

```bash
root@w7:/ # mount /dev/block/mmcblk1p2 /root
root@w7:/ # export PATH=/usr/local/sbin:/usr/local/bin:/sbin:/usr/sbin:/bin:/usr/bin:/system/bin:$PATH
root@w7:/ # echo 'deb https://mirrors.bfsu.edu.cn/debian/ buster main contrib non-free' > /root/etc/apt/sources.list
root@w7:/ # for f in dev dev/pts proc sys ; do mount -o bind /$f /root/$f ; done
root@w7:/ # export TMPDIR=/tmp
root@w7:/ # export HOME=/root
root@w7:/ # busybox chroot /root /bin/bash -l
bash-4.3# debootstrap/debootstrap --second-stage
...
I: Base system installed successfully.
```

现在刷新repo数据，并安装OpenSSH-Server

```
bash-4.3# apt-get update
bash-4.3# apt-get install openssh-server
...
Setting up openssh-server (1:6.7p1-5+deb8u2) 
...
[ ok ] Starting OpenBSD Secure Shell server: sshd.
bash-4.3# passwd root
bash-4.3# exit
bash-4.3# sync
```

在这里，您实际上已经在Android设备上运行了Debian！ 但是它是chroot于Android之下，我们希望得到相反的结果。 但是，现在我们有了一个带有SSH服务器及所有组件的完整Debian系统。 我们仍然需要进行一些修补。 （如果apt-get更新不起作用，请检查您的/etc/named.conf。）

## 建立新的Android的/

将SD卡再次装回到电脑。将原始boot.img的initramfs解压到SD卡上 ext3/4分区下的Android上。这将成为Debian系统目录结构中的新Android的/。新建名为/android/log的文件夹。注意：由于/并非以挂载点而是以子目录存在，Android将无法挂在它为只读。如果你觉得这会引起问题，可以在SD卡上单独分出Android的/并作为/mnt/root/android挂在到initramfs的init中，但是，这会让/android/log只读，/etc/init可能不会在此目录写入log。您可以通过挂载ramfs解决问题，或从/etc/init中关闭日志。

由于vold的限制，Android通常仅在SD卡上接受4个分区。 如果您不想将其中一个浪费在小型根文件系统上，则可以将挂载（使用–bind选项）从/ android循环到/ mnt / android进行挂载。 然后，您可以使用重新安装将此安装点设置为只读。 请注意，您必须重新安装，因为绑定安装最初不能更改原始文件系统的标志。 在这种情况下，您必须使用/ bin / mount在init.stage2中显式地重新安装。 现在，只要让根文件可写，直到一切就绪并运行即可。 可以稍后再做-或根本不做。

## 收尾工作

新的initramfs将控制权转给iSD卡上ext4分区的/etc/init中。

以下是一个Debian引导初始化脚本的示例：

```
#!/sbin/busybox sh
#
# Debian environment boot init script
#
# Leave all the initialization process to the Android init to handle
#
# Launch delayed init script
/etc/init.stage2 > /android/log/boot.log 2>&1 &
# Transfer control to Android init - never returns
exec /sbin/busybox chroot /android /init
```

确保将busybox复制到/sbin。 请注意，来自init.stage2的日志存储在Android文件树中，因此，您可以从Android访问Debian，比如，由于/etc/rc.local中的某些错误而导致Debian级SSH服务器无法启动时，您可以从Android访问它。

然后创建第二个脚本init.stage2 －在Android初始化完成后，Debian环境将执行第二个延迟脚本的派生。 然后，它将控制权转移到Android的原始init（当然仍以pid 1运行）。

```
#!/sbin/busybox sh
#
# Delayed Debian environment boot init script
# Not really init (not pid 1) but a fork of it.
# The real init is right now executing in Android chroot
#
/sbin/busybox echo "`/sbin/busybox date` Debian init stage2 started"
# Wait for Android init to set up everything
# wait for dev to be mounted by Android init
/sbin/busybox echo "`/sbin/busybox date` Waiting on Android to mount /dev"
while [ ! -e /android/dev/.coldboot_done ]; do
/sbin/busybox sleep 1
done
# wait for Android init to signal all done
/sbin/busybox echo "`/sbin/busybox date` Waiting on Android init to finish"
while [ -e /android/dev/.booting ]; do
/sbin/busybox sleep 1
done
# Mount the /proc, /sys etc filesystems
/sbin/busybox echo "`/sbin/busybox date` Mounting /proc /sys and /dev"
/sbin/busybox mount -t proc none /proc
/sbin/busybox mount -t sysfs none /sys
# Mount /dev from the Android world
/sbin/busybox mount -o bind /android/dev /dev
/sbin/busybox mount -o bind /android/dev/pts /dev/pts
/sbin/busybox mount -o bind /android/dev/socket /dev/socket
# All done, now we can start running stuff
export PATH=/sbin:/usr/sbin:/bin:/usr/bin
/sbin/busybox echo "`/sbin/busybox date` Running /etc/rc.local"
# Start selected servers
/etc/init.d/rc.local start
/sbin/busybox echo "`/sbin/busybox date` All done"
exit 0
```

基本上，这只会等待Android初始化，然后设置Debian所需的所有内容，例如dev，proc和sys mount，并执行/etc/rc.local。 由于我们是从Android根目录挂载/dev循环安装的，因此必须删除/dev中由debootstrap填充的所有设备，否则该挂载将失败。 我的/etc/rc.local看起来像这样：

```
#!/bin/sh -e
#
# rc.local
#
# Executed at the end of each multiuser runlevel.
# Make sure the script will "exit 0" on success or print
# any other value on error.
#
# To enable or disable this script, just change the execution# bits.
#
# By default this script does nothing.
/etc/init.d/hostname.sh start
/etc/init.d/ssh start
exit 0
```

请注意，init确保将此处的所有内容都记录到/android/log/boot.log。 这样的话在ssh服务器无法启动的情况下，您可能会在adb shell到Android的文件/log/boot.log中看到原因。

## 刷写新的Boot文件

如果一切顺利，该安装自定义的启动映像了。 LG L90手机在这里具有解锁的引导程序，支持fastboot。 使用组合键在手机上进入快速启动模式（在手机关闭时按住VolumeUp并通过USB线将其连接到桌面）。在你的Linux电脑上运行：

```
# fastboot devices
# fastboot flash boot my-boot.img
# fastboot reboot
```

全做完了！ 您现在正在运行以Matrix Way与Android集成的Debian。 以root用户身份使用您指定的密码连接到ssh。

---

以下内容属于扩展内容，待补。