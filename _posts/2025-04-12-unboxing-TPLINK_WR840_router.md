
# 🛠️ Hacking the Holidays: TP-Link WR840N Unboxing & Shenanigans

## Introduction

Ah, Passover — the perfect time to clean your home, reflect on freedom, and... tear apart cheap networking hardware?

Faced with the holiday break, I had two options:

- Go outside, touch grass, and be a normal person with friends.
- Stay indoors, rip open a dusty old router, and see how fast I can make it beg for mercy.

If you've ever read *literally anything* I've written, you know exactly which path I took 😎

But first, I needed a worthy opponent. So I turned to my ever-faithful partner in crime: ChatGPT.

Me:
> I'm looking to hack into a small embedded device for totally innocent, 100% educational reasons. It needs to be cheap, hackable, and not explode. What should I get?

ChatGPT:
> 🔥 **Top Cheap Routers for Hacking Adventures**
> 1. TP-Link TL-WR841N

And just like that, destiny chose my target.

Let's crack this thing open.

---

## 🔍 Research

### Pre-Purchase Research

#### Finding the Firmware

First mission: locate the router’s firmware — because why plug something in when you can tear apart its soul first?

With ChatGPT’s help, I found this golden archive:

👉 [TL-WR840N Firmware Downloads](https://www.tp-link.com/ae/support/download/tl-wr840n/?utm_source=chatgpt.com#Firmware)

I grabbed the first one I saw, because patience is for people who don't void warranties:
> `TL-WR840N(EU)_V6.20_241230`

#### Extracting the Firmware

Out comes the scalpel: `binwalk`.

```bash
t_omersas@PC /tmp $ binwalk -e TL_WR840N.bin
```

Result:
```
- U-Boot bootloader? ✅
- LZMA compressed data? ✅
- SquashFS filesystem? Oh baby, yes. ✅
- A disturbing amount of symlinks to /dev/null? We'll pretend that's fine.
```

Then I navigated into the extracted guts of the firmware:

```bash
t_omersas@PC /tmp $ cd _TL_WR840N.bin.extracted
```

And peeked inside the contents like a digital pathologist.

#### Filesystem Layout

What greeted me was a Linux filesystem, complete with BusyBox utilities and a `/web/` directory that screamed "please hack me."

```bash
/bin
/dev
/etc
/lib
/sbin
/sys
/usr
/var
/web  <-- 🍪 Jackpot?
```

> **💡 Tip:** Router web servers are usually standalone binaries located in `/usr/bin/`, not kernel modules.
> **💡 Tip:** BusyBox is the Swiss Army knife of embedded Linux — it handles init, login, and even pretends to be `sh` on weekends.
> **💡 Tip:** QEMU emulation of MIPS devices is more fragile than your new year’s resolutions. Expect lots of tears.
> **💡 Tip:** The `init` binary is usually just a symlink to BusyBox. Real init magic happens inside scripts like `/etc/init.d/rcS`.
> **💡 Tip:** Default credentials for routers can be shockingly simple — try `admin`, `root`, `1234`, or just yelling at it.

---

The rest of the blog continues with juicy details about failed QEMU emulation attempts, web server spelunking, and the classic “Why won’t this damn login work?” scenario. But don’t worry — I preserved all sections, including TODOs, and I’ll keep punching up the tone in the same fun, technical style.

Now saving this beautified version...


## 
### Pre-Purchase Research
#### Finding The Firmawre
First of all, I decided to find the firmware and see what it contains. Again, just by asking ChatGPT I've found this great website of TP-Link:
[TL-WR840N Firmwares](https://www.tp-link.com/ae/support/download/tl-wr840n/?utm_source=chatgpt.com#Firmware)
And I download the first version that I've seen: 
> TL-WR840N(EU)_V6.20_241230

#### Extracting The Firmware
Now I need to extract the firmware of the device, and it actually quite simple. By running binwalk on it, I actually solved it easily:
```bash
t_omersas@PC /tmp $ binwalk -e TL_WR840N.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
54592         0xD540          U-Boot version string, "U-Boot 1.1.3 (Dec 30 2024 - 15:49:18)"
66560         0x10400         LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 2986732 bytes

WARNING: Symlink points outside of the extraction directory: /tmp/_TL_WR840N.bin.extracted/squashfs-root-0/etc/TZ -> /var/tmp/TZ; changing link target to /dev/null for security purposes.

WARNING: Symlink points outside of the extraction directory: /tmp/_TL_WR840N.bin.extracted/squashfs-root-0/etc/passwd -> /var/passwd; changing link target to /dev/null for security purposes.

WARNING: Symlink points outside of the extraction directory: /tmp/_TL_WR840N.bin.extracted/squashfs-root-0/etc/resolv.conf -> /var/tmp/resolv.conf; changing link target to /dev/null for security purposes.

WARNING: Symlink points outside of the extraction directory: /tmp/_TL_WR840N.bin.extracted/squashfs-root/etc/TZ -> /var/tmp/TZ; changing link target to /dev/null for security purposes.

WARNING: Symlink points outside of the extraction directory: /tmp/_TL_WR840N.bin.extracted/squashfs-root/etc/passwd -> /var/passwd; changing link target to /dev/null for security purposes.

WARNING: Symlink points outside of the extraction directory: /tmp/_TL_WR840N.bin.extracted/squashfs-root/etc/resolv.conf -> /var/tmp/resolv.conf; changing link target to /dev/null for security purposes.
1049088       0x100200        Squashfs filesystem, little endian, version 4.0, compression:xz, size: 2996448 bytes, 553 inodes, blocksize: 262144 bytes, created: 2024-12-30 07:54:17
```

Cool, let's see the content of the files inside it:
```bash
t_omersas@PC /tmp $ cd _TL_WR840N.bin.extracted
t_omersas@PC /tmp/_TL_WR840N.bin.extracted $ ls
100200.squashfs  10400  10400.7z  squashfs-root  squashfs-root-0
```

running binwalk on 10400 lead to:
```bash
t_omersas@PC /tmp/_TL_WR840N.bin.extracted $ binwalk 10400

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
2240616       0x223068        Linux kernel version 2.6.36
2240788       0x223114        CRC32 polynomial table, little endian
2280144       0x22CAD0        CRC32 polynomial table, little endian
2509580       0x264B0C        xz compressed data
2556352       0x2701C0        Neighborly text, "NeighborSolicits6InDatagrams"
2556372       0x2701D4        Neighborly text, "NeighborAdvertisementsorts"
2559839       0x270F5F        Neighborly text, "neighbor %.2x%.2x.%pM lostrename link %s to %s"
2981888       0x2D8000        ASCII cpio archive (SVR4 with no CRC), file name: "/dev", file name length: "0x00000005", file size: "0x00000000"
2982004       0x2D8074        ASCII cpio archive (SVR4 with no CRC), file name: "/dev/bash", file name length: "0x0000000D", file size: "0x00000000"
2982128       0x2D80F0        ASCII cpio archive (SVR4 with no CRC), file name: "/root", file name length: "0x00000006", file size: "0x00000000"
2982244       0x2D8164        ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!", file name length: "0x0000000B", file size: "0x00000000"
```
Cool, it seems like the kernel that the device is running.

Let's also examine the squashfs-root directory (which seems to be equal to squashfs-root-0 directory, and extracting 100200.squashfs file seems to be the same):
```bash
t_omersas@PC /tmp/_TL_WR840N.bin.extracted $ ls squashfs-root/*
squashfs-root/linuxrc

squashfs-root/bin:
ash  busybox  cat  chmod  cp  date  df  echo  kill  login  ls  mkdir  mount  netstat  pidof  ping  ping6  ps  rm  sh  sleep  umount

squashfs-root/dev:
net  pts  shm

squashfs-root/etc:
MT7628_AP_2T2R-4L_V15.BIN   RT2860AP.dat      TZ                  fstab  init.d   iptables-stop  passwd.bak  reduced_data_model.xml  samba
MT7628_EEPROM_20140317.bin  SingleSKU_CE.dat  default_config.xml  group  inittab  passwd         ppp         resolv.conf             services

squashfs-root/lib:
ld-uClibc-0.9.33.2.so  libcmm.so             libcutil.so        libgdpr.so   libm-0.9.33.2.so    libnsl.so.0             libpthread.so.0        librt-0.9.33.2.so  libuClibc-0.9.33.2.so  libutil.so.0
ld-uClibc.so.0         libcrypt-0.9.33.2.so  libdl-0.9.33.2.so  libiw.so.29  libm.so.0           libos.so                libresolv-0.9.33.2.so  librt.so.0         libupnp.so             libxml.so
libc.so.0              libcrypt.so.0         libdl.so.0         libixml.so   libnsl-0.9.33.2.so  libpthread-0.9.33.2.so  libresolv.so.0         libthreadutil.so   libutil-0.9.33.2.so    modules

squashfs-root/mnt:

squashfs-root/proc:

squashfs-root/sbin:
config-mii.sh  getty  halt  ifconfig  init  insmod  lsmod  mii_mgr  mii_mgr_cl45  poweroff  reboot  rmmod  route  switch  vconfig

squashfs-root/sys:

squashfs-root/usr:
bin  sbin

squashfs-root/var:

squashfs-root/web:
MenuRpm.htm  css  domain-redirect.htm  frame  help  img  index.htm  js  main  mainFrame.htm  qr.htm
```

Amazing! it seems like a linux filesystem + some user mode programs like busybox and webserver related files.

#### Emulating The Kernel -> Failed
In this point I decided to try and emulate the linux kernel. The reasons I decided to do this are:
    * At this point I still have no setup.
    * I've never used qemu before nor any other emulation program; So why not trying?
Actually in a retrospective, it is not something I would repeat in the future, since emulating the kernel is not so efficient, and instead I think it is more rational to find bugs in a user-mode applications written by TPLink.

So in order to emulate it I had to discover first, what is the architecture of this kernel. Binwalk seems to be with very clear results:
```bash
t_omersas@PC ~ $ cd ~/projects/_TL_WR840N.bin.extracted/
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted $ vim ~/.tmux.conf
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted $ binwalk -A 10400

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
50140         0xC3DC          MIPSEL instructions, function epilogue
50360         0xC4B8          MIPSEL instructions, function epilogue
50536         0xC568          MIPSEL instructions, function epilogue
50888         0xC6C8          MIPSEL instructions, function epilogue
51444         0xC8F4          MIPSEL instructions, function epilogue
51652         0xC9C4          MIPSEL instructions, function epilogue
51708         0xC9FC          MIPSEL instructions, function epilogue
51776         0xCA40          MIPSEL instructions, function epilogue
51900         0xCABC          MIPSEL instructions, function epilogue
52268         0xCC2C          MIPSEL instructions, function epilogue
...........................
...........................
...........................
```

Ok let's go one step ahead, and try to run it using qemu:
```bash
qemu-mips 10400
Error while loading /tmp/_TL_WR840N.bin.extracted/10400: Exec format error
```

Asking ChatGPT leads to the reason that the file is not elf, which needed by qemu.
I tried to generate an ELF using advices from ChatGPT:
```
mipsel-linux-gnu-objcopy -I binary -O elf32-tradlittlemips -B mips 10400 10400.elf
```
then, creating a link.ld file:
```ld
SECTIONS
{
  . = 0x80000000;
  .text : { *(.text) }
  .data : { *(.data) }
  .bss  : { *(.bss)  }
}
```
and then running:
```bash
mipsel-linux-gnu-ld -T link.ld -o 10400_exec.elf 10400.elf
```
but then I got:
```
qemu-system-mipsel: Trap-and-Emul kernels (Linux CONFIG_KVM_GUEST) are not supported
```
which is very sad. Looking at the official website of qemu, I discovered that support of `Trap-and-Emul` kernels (which I have no idea what does it means) is being depreacted since qemu 6.0 or something like that:

> MIPS Trap-and-Emul KVM support (since 6.0):
> The MIPS Trap-and-Emul KVM host and guest support has been removed from Linux upstream kernel, declare it deprecated.

so I've downloaded qemu-5.0.0 by:
```bash
wget https://download.qemu.org/qemu-5.0.0.tar.xz
tar -xvf qemu-5.0.0.tar.xz
cd qemu-5.0.0
```
and executed something like that:
```bash
qemu-system-mipsel -M malta -kernel 10400_exec.elf -initrd initrd.img -drive file=rootfs.sqfs,format=raw,if=mtd -append "root=/dev/mtdblock0 bash=ttyS0" -nographic
```
but now I've got something like:
```
rom: requested regions overlap (rom prom. free=0x00000000002d92ec, addr=0x0000000000002000)
qemu-system-mipsel: rom check and register reset failed
```

anyway, at this point I thought about it again, and I've got to the point that emulating the kernel is not something that I want to focus, and instead, let's focus on the filesystem, and more specific, let's focus on the web-server

#### Understanding Where The WebServer Implementation Located
So, in order to find my vulnerabillites, I looked at the file system more deeply, and tried to find some code that runs the webserver himself. Since, usually, the webservers are being executed as a user programs in the init of the system, and the webservers are usually have a different implementation to each device, I thought that it has higher precentages to include vulnerabilites.

> **💡 Tip:** The webservers implementations of router almost always a user-mode program stored as a regular file in the filesystem, not part of the Linux kernel. The kernel does not load the web server itself. Instead, a user-space init system or startup script is responsible for launching it after the kernel has booted and mounted the root filesystem.

cool, so based on this, I searched for a relevant file on the bin directory, to see, maybe there is an implementation there.

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ ls -la bin
total 3636
drwxr-xr-x  2 t_omersas t_omersas    4096 Apr 12 12:56 .
drwxr-xr-x 13 t_omersas t_omersas    4096 Apr 12 09:49 ..
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 ash -> busybox
-rwxr-xr-x  1 t_omersas t_omersas  262100 Apr 12 09:49 busybox
-rw-r--r--  1 t_omersas t_omersas 3449390 Apr 12 12:56 busybox.i64
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 cat -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 chmod -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 cp -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 date -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 df -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 echo -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 kill -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 login -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 ls -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 mkdir -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 mount -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 netstat -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 pidof -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 ping -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 ping6 -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 ps -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 rm -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 sh -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 sleep -> busybox
lrwxrwxrwx  1 t_omersas t_omersas       7 Apr 12 09:49 umount -> busybox
```

It seems like there is only one file that is interesting here, and this is the busybox. 
> **💡 Tip:** BusyBox provides a minimal implementation of init and the whole suite of core Unix tools.

The busybox file seems to be an MIPSEL elf, and by opening it on IDA and searching for relevant strings, I didn't find any relevant data for an http server :(
for example, I didn't find a string that includes "index.htm" which I saw in the web directory and appears to be the html file of the main page of the router webserver...

Anyway, I decided to try and emulate this file, and maybe I will discover some other relevant data about him:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
BusyBox v1.19.2 (2024-12-30 15:52:05 CST) multi-call binary.
Copyright (C) 1998-2011 Erik Andersen, Rob Landley, Denys Vlasenko
and others. Licensed under GPLv2.
See source distribution for full notice.

Usage: busybox [function] [arguments]...
   or: busybox --list[-full]
   or: function [arguments]...

        BusyBox is a multi-call binary that combines many common Unix
        utilities into a single executable.  Most people will create a
        link to busybox for each function they wish to use and BusyBox
        will act like whatever it was invoked as.

Currently defined functions:
        arping, ash, brctl, cat, chmod, cp, date, df, echo, free, getty, halt,
        ifconfig, init, insmod, ipcrm, ipcs, kill, killall, linuxrc, login, ls,
        lsmod, mkdir, mount, netstat, pidof, ping, ping6, poweroff, ps, reboot,
        rm, rmmod, route, sh, sleep, taskset, tftp, top, umount, vconfig
```

looks good! seems like an implementation for a several functions... let's try to arping to 8.8.8.8:
```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox arping 8.8.8.8
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
ARPING to 8.8.8.8 from 172.20.123.158 via eth0
^CSent 25 probe(s) (25 broadcast(s))
Received 0 reply (0 request(s), 0 broadcast(s))
```
seems to fail, but it starts well.... maybe it's because of the emulation.

Anyway, let's keep searching for our webserver. By looking at some more directories I saw another suspicious file: /etc/init.d/rcS

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ cat etc/init.d/rcS
#!/bin/sh

mount -a
# added by yangcaiyong for sysfs
mount -t sysfs /sys /sys
# ended add

/bin/mkdir -m 0777 -p /var/https
/bin/mkdir -m 0777 -p /var/lock
/bin/mkdir -m 0777 -p /var/log
/bin/mkdir -m 0777 -p /var/run
/bin/mkdir -m 0777 -p /var/tmp
/bin/mkdir -m 0777 -p /var/Wireless/RT2860AP
/bin/mkdir -m 0777 -p /var/tmp/wsc_upnp
cp -p /etc/SingleSKU_FCC.dat /var/Wireless/RT2860AP/SingleSKU.dat

/bin/mkdir -m 0777 -p /var/tmp/dropbear

/bin/mkdir -m 0777 -p /var/dev
cp -p /etc/passwd.bak /var/passwd
/bin/mkdir -m 0777 -p /var/l2tp

echo 1 > /proc/sys/net/ipv4/ip_forward
#echo 1 > /proc/sys/net/ipv4/tcp_syncookies
echo 1 > /proc/sys/net/ipv6/conf/all/forwarding

echo 30 > /proc/sys/net/unix/max_dgram_qlen

#krammer add for LAN can't continuous ping to WAN when exchenging the routing mode
#bug1126
echo 3 > /proc/sys/net/netfilter/nf_conntrack_icmp_timeout

echo 0 > /proc/sys/net/ipv4/conf/default/accept_source_route
echo 0 > /proc/sys/net/ipv4/conf/all/accept_source_route
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore

echo 2560 > /proc/sys/net/netfilter/nf_conntrack_expect_max
#defined 8192 in nf_conntrack_core.c
echo 5120 > /proc/sys/net/netfilter/nf_conntrack_max

#allow max low mem alloc
echo 2 > /proc/sys/vm/overcommit_memory
echo 100 > /proc/sys/vm/overcommit_ratio
echo 2048 > /proc/sys/vm/min_free_kbytes

insmod /lib/modules/kmdir/kernel/drivers/net/rt_rdm/rt_rdm.ko
insmod /lib/modules/kmdir/kernel/drivers/net/raeth/raeth.ko

#netfilter modules load
insmod /lib/modules/kmdir/kernel/net/netfilter/nf_conntrack_proto_gre.ko
insmod /lib/modules/kmdir/kernel/net/netfilter/nf_conntrack_pptp.ko

#for sfe
[ -d /lib/modules/kmdir/kernel/net/shortcut-fe ] && {
        insmod /lib/modules/kmdir/kernel/net/shortcut-fe/shortcut-fe.ko
        insmod /lib/modules/kmdir/kernel/net/shortcut-fe/shortcut-fe-cm.ko
        echo 512 > /sys/sfe_ipv4/max_connections
}

#ip statisctics
insmod /lib/modules/ipt_STAT.ko
#support tplinklogin.net
insmod /lib/modules/tp_domain.ko



ifconfig lo 127.0.0.1 netmask 255.0.0.0

#for l2tp modules
insmod /lib/modules/pppol2tp.ko
insmod /lib/modules/l2tp_ppp.ko

#config mii for 7628
config-mii.sh

cos &
```

Looking deeply in the file, didn't lead to any lines that looks like running a webserver or something like that. 
Keep on searching....

Looking on the directory bin looks the most promoising result up now:
```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ ls usr/bin
afcd     cmxdns  dhcpd     dropbearkey    ebtables  igmpd      ipcrm   iptables  killall  pwdog      taskset  tdpd  top         wanType        xtables-multi
arping   cos     dnsProxy  dropbearmulti  free      ip         ipcs    iwconfig  noipdns  rt2860apd  tc       tftp  traceroute  wlNetlinkTool
ated_tp  dhcpc   dropbear  dyndns         httpd     ip6tables  ipping  iwpriv    ntpc     scp        tddp     tmpd  upnpd       wscd
```


Cool! there is the implementation for the webserver `httpd` and in addition, it seems like there is also an implementation for a tftp.

Running strings on the http file shows that it really our desired file, as it holds all the files that we saw in the /web/main dir.
```
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ strings usr/bin/httpd | grep htm                                                                                                                frame/login.htm                                                                                                                                                                                                  <html><head><title>%d %s</title></head><body><center><h1>%d %s</h1></center></body></html>                                                                                                                       domain-redirect.htm
qr.htm
index.htm
text/html
"%s.htm",
<html><head></head><body>%s</body></html>
<html><head></head><body>OK</body></html>
/main/status.htm
/main/statusAP.htm
/main/statusClient.htm
/main/stat.htm
/main/wlStats.htm
/main/qsStart.htm
/main/qsType.htm
/main/qsPPP.htm
/main/qsStaIp.htm
/main/qsWl.htm
/main/qsL2tp.htm
/main/qsPptp.htm
/main/qsAuto.htm
/main/qsEnd.htm
/main/qsSave.htm
...
...
...
```

#### Understanding The Init Process
I was about to start and reversing the web server, but before doing it, I was thinking to my self: "maybe there are some more surfaces that will be more interesting than the web server? what if there is, for example, a telnet surface that is reserved for technicians, and allow you to read write execute a things on the device by just giving a username and a password?"

So, in order to discover if it is a thing, I decided to understand what is being done on the init of the system. In order to do it, I asked chatGPT where does the init executable is located, and I was referenced to look at 
```bash
/sbin/init
```

executing ls on it lead to:
```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ ls -la sbin/init
lrwxrwxrwx 1 t_omersas t_omersas 14 Apr 12 09:49 sbin/init -> ../bin/busybox
```

and running busybox with qemu, show that:
```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
BusyBox v1.19.2 (2024-12-30 15:52:05 CST) multi-call binary.
Copyright (C) 1998-2011 Erik Andersen, Rob Landley, Denys Vlasenko
and others. Licensed under GPLv2.
See source distribution for full notice.

Usage: busybox [function] [arguments]...
   or: busybox --list[-full]
   or: function [arguments]...

        BusyBox is a multi-call binary that combines many common Unix
        utilities into a single executable.  Most people will create a
        link to busybox for each function they wish to use and BusyBox
        will act like whatever it was invoked as.

Currently defined functions:
        arping, ash, brctl, cat, chmod, cp, date, df, echo, free, getty, halt,
        ifconfig, init, insmod, ipcrm, ipcs, kill, killall, linuxrc, login, ls,
        lsmod, mkdir, mount, netstat, pidof, ping, ping6, poweroff, ps, reboot,
        rm, rmmod, route, sh, sleep, taskset, tftp, top, umount, vconfig
```
there is a really init in the defined functions. we can even try to run it:
```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox init
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
init: must be run as PID 1
```

Cool!

In addition, as we can see, there is also a login function. Let's try to run it:
```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox login
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
PC login: a
Password:
Login incorrect
PC login: ^C
```

Interesting... maybe the default users, passwords to embedded devices - root root, admin 1234, etc.. but all of this neither worked :( 

Ok, let's put it on the side right now.
Let's look on the init handler, by searching the error message we got when we had run it - "must be run as PID 1".

We arrived to an interesting function, thats really looks like the init process:
```c
int __fastcall init_handler(int a1, _DWORD *a2)
{
  int v2; // $a0
  _DWORD *v3; // $s0
  int v4; // $v0
  int v6; // $a1
  int v7; // $v0
  int v8; // $v0
  int v9; // $s1
  int v10; // $s1
  int v11; // $a0
  _BYTE *v12; // $s1
  int v13; // $v0
  int v14; // $a0
  const char *v15; // $a1
  void (__fastcall *v16)(int, const char *, int); // $t9
  int i; // $a2
  int v18; // $v0
  int v19; // $s1
  int v20; // $s1
  BOOL j; // $a2
  int v22; // $v0
  int v23; // $v0
  _BYTE v24[4]; // [sp+18h] [-2Ch] BYREF
  int v25; // [sp+1Ch] [-28h] BYREF
  int (*v26)(); // [sp+20h] [-24h]
  _DWORD v27[5]; // [sp+24h] [-20h] BYREF
  int v28; // [sp+38h] [-Ch]

  v2 = a2[1];
  v3 = a2;
  if ( !v2 || (v4 = strcmp(v2, "-q"), v2 = 1, v4) )
  {
    if ( getpid(v2) != 1 && *(_BYTE *)dword_44FAE0 != 108 )
      sub_435A50("must be run as PID 1", v6);
    reboot(0);
    dword_44FAEC = 2592000;
    v7 = getenv("CONSOLE");
    if ( v7 || (v7 = getenv("console")) != 0 )
    {
      v8 = open64(v7, 2178);
      v9 = v8;
      if ( v8 >= 0 )
      {
        dup2(v8, 0);
        dup2(v9, 1);
        sub_436E50(v9, 2);
      }
    }
    else
    {
      sub_435C98();
    }
    v10 = getenv("TERM");
    if ( ioctl(0, 22016, v24) )
    {
      if ( !v10 || !strcmp(v10, "linux") )
        putenv("TERM=vt102");
      off_44F654 = 0;
    }
    else if ( !v10 )
    {
      putenv("TERM=linux");
    }
    sub_42A794();
    sub_4370D0("/");
    setsid();
    putenv("HOME=/");
    putenv("PATH=/sbin:/usr/sbin:/bin:/usr/bin");
    putenv("SHELL=/bin/sh");
    putenv("USER=root");
    putenv("TMPDIR=/var/tmp");
    v11 = 4456448;
    if ( v3[1] )
      sub_436FF8("RUNLEVEL");
    v12 = (_BYTE *)v3[1];
    if ( v12 && (!strcmp(v3[1], "single") || !strcmp(v12, "-s") || *v12 == 49 && !v12[1]) )
      sub_42A84C(8, "-/bin/sh", "");
    else
      sub_42ABA8(v11);
    v13 = strlen(*v3);
    v14 = *v3;
    v15 = "init";
    v16 = (void (__fastcall *)(int, const char *, int))&strncpy;
    for ( i = v13; ; i = v18 )
    {
      ++v3;
      v16(v14, v15, i);
      if ( !*v3 )
        break;
      v18 = strlen(*v3);
      v14 = *v3;
      v15 = 0;
      v16 = (void (__fastcall *)(int, const char *, int))&memset;
    }
    sub_434A44(229376, sub_42B6F8);
    signal(3, sub_42B650);
    v25 = 0;
    v26 = 0;
    v27[4] = 0;
    memset(v27, 255, 16);
    sigdelset(v27, 25);
    v26 = sub_42B494;
    sub_4349E4(24, &v25);
    sub_4349E4(23, &v25);
    sub_434AC8(4, sub_4349D8);
    sub_434AC8(2, sub_4349D8);
    sub_42B254(1);
    sub_42B35C();
    sub_42B254(2);
    sub_42B35C();
    sub_42B254(4);
    while ( 1 )
    {
      v19 = sub_42B35C();
      sub_42B254(24);
      v20 = sub_42B35C() | v19;
      sleep(1);
      for ( j = (v20 | sub_42B35C()) != 0; ; j = 1 )
      {
        v22 = waitpid(-1, 0, j);
        if ( v22 <= 0 )
          break;
        v28 = v22;
        v23 = sub_42A74C(v22);
        if ( v23 )
          sub_42AA38(1, "process '%s' (pid %d) exited. Scheduling for restart.", (const char *)(v23 + 41), v28);
      }
    }
  }
  return kill(1, 1);
}
```

I was not going deep into it, since it uses a-lot of functions, and I don't have a source code (actually there is a kind of source code for busybox but it a general codebase and in my opinion it was changed specifically for this router, since I can't find some strings in the codebase, that I see on the IDA.

Anyway, some interesting thing I've found, is that, by looking on the xref to that function, we can see that it stays inside a list of function, that seems to be the handlers of the commands. I verified it by going to other functions and see that they are functions that looks like commands that the busybox implemented. In addition, I saw that this handlers list, sorted just like it appears when running the busybox to see his commands:

```bash
arping, ash, brctl, cat, chmod, cp, date, df, echo, free, getty, halt,
ifconfig, init, insmod, ipcrm, ipcs, kill, killall, linuxrc, login, ls,
lsmod, mkdir, mount, netstat, pidof, ping, ping6, poweroff, ps, reboot,
rm, rmmod, route, sh, sleep, taskset, tftp, top, umount, vconfig
```

So I started to name some functions:
```ida
.data.rel.ro:0044F538 D4 7A 40 00 commands_handlers:.word sub_407AD4       # DATA XREF: sub_404EC0:loc_404F6C↑o
.data.rel.ro:0044F53C 1C 12 42 00                 .word sub_42121C
.data.rel.ro:0044F540 A0 86 40 00                 .word sub_4086A0
.data.rel.ro:0044F544 34 78 42 00                 .word sub_427834
.data.rel.ro:0044F548 7C 79 42 00                 .word sub_42797C
.data.rel.ro:0044F54C F8 7A 42 00                 .word sub_427AF8
.data.rel.ro:0044F550 E8 7C 42 00                 .word sub_427CE8
.data.rel.ro:0044F554 D8 7F 42 00                 .word sub_427FD8
.data.rel.ro:0044F558 6C 83 42 00                 .word echo_handler
.data.rel.ro:0044F55C A4 F7 40 00                 .word free_handler
.data.rel.ro:0044F560 0C 5D 40 00                 .word getty_handler
.data.rel.ro:0044F564 C8 A5 42 00                 .word halt_handler
.data.rel.ro:0044F568 2C 90 40 00                 .word ifconfig_handler
.data.rel.ro:0044F56C 78 B7 42 00                 .word init_handler
.data.rel.ro:0044F570 DC 70 40 00                 .word insmod_handler
.data.rel.ro:0044F574 D4 30 42 00                 .word ipcrm_handler
.data.rel.ro:0044F578 88 47 42 00                 .word ipcs_handler
.data.rel.ro:0044F57C E0 F9 40 00                 .word kill_handler
.data.rel.ro:0044F580 E0 F9 40 00                 .word kill_handler
.data.rel.ro:0044F584 78 B7 42 00                 .word init_handler
.data.rel.ro:0044F588 F8 67 40 00                 .word login_handler
.data.rel.ro:0044F58C 28 90 42 00                 .word sub_429028
```

And that's lead to the next mission, to understand the login creds!

#### Login Credentials
So, first of all, let's understand what's going on. /bin/login is soft linked to busybox meaning it's calls to busybox login (I didn't prove it, but it sounds logically). Let's see in the IDA who calls to /bin/login. Xref to this string found only on - `getty_handler` which is, by chatGPT, manages physical or virtual terminals. It waits for a connection on a terminal line and then invokes login.

Ok, cool, now let's dive deeper to find a username and a password.
I searched for XRefs to "Password:" string, and land on a function that looks very interesting.

```c
BOOL __fastcall check_password(BOOL is_adminqq)
{
  const char *expected_password; // $a1
  int input; // $v0
  int v3; // $s1
  BOOL is_equal; // $s0
  int v5; // $v0
  const char *v7; // [sp+18h] [-Ch]

  if ( is_adminqq )
    expected_password = "sohoadmin";
  else
    expected_password = "aa";
  v7 = expected_password;
  input = inputqq((int)"Password: ");
  v3 = input;
  is_equal = 0;
  if ( input )
  {
    is_equal = strcmp(input, v7) == 0;
    v5 = strlen(v3);
    memset(v3, 0, v5);
  }
  return is_equal;
}
```

seems like depending on the BOOL if we are admin or not, the password that expected is 'sohoadmin' or 'aa'. when I tried sohoadmin, it still wrote me that the password is incorrect. but when trying aa it crashed the program. I tried to understand why it is crashing my program, and more important, how can I be an admin.

I saw that the `is_admin` bool is received from `sub_438728`, which inside it, calls to that function:
```c
int __fastcall sub_438368(int a1, _DWORD *a2, int a3, int a4, _DWORD *a5)
{
  int v9; // $s0
  int v10; // $s7
  int v11; // $v0

  *a5 = 0;
  v9 = open_read("/etc/passwd");
  if ( !v9 )
    return *(_DWORD *)dword_44FAE4;
  while ( 1 )
  {
    v11 = sub_437CFC(sub_438214, a2, a3, a4, v9);
    v10 = v11;
    if ( v11 )
      break;
    if ( !strcmp(*a2, a1) )
    {
      *a5 = a2;
      goto LABEL_7;
    }
  }
  v10 = v11 != 2 ? v11 : 0;
LABEL_7:
  fclose(v9);
  return v10;
}
```
that seems like it opening /etc/passwd. I now remember that when I extracted the BIN file, it wrote me 
```warning
WARNING: Symlink points outside of the extraction directory: /home/t_omersas/temp/_TPLINK.bin-0.extracted/squashfs-root/etc/passwd -> /var/passwd; changing link target to /dev/null for security purposes.
```

So, it might be the problem.

Let's change it manually.
```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root/etc $ rm -rf passwd
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root/etc $ sudo ln -s /var/passwd passwd
```

and, let's try to create a chroot environment using 
```bash
cp /usr/bin/qemu-mipsel-static squashfs-root/usr/bin/
sudo chroot squashfs-root /usr/bin/qemu-mipsel-static /bin/sh
```

Then, running:
```bash
/usr/bin/qemu-mipsel-static /bin/busybox login
```

but it seems to be crashing too:
```bash
/ # /usr/bin/qemu-mipsel-static /bin/busybox login
PC login: a
Password:
qemu: uncaught target signal 11 (Segmentation fault) - core dumped
Segmentation fault (core dumped)
```

maybe, it something else :(


#### Reversing The Web Server
TODO!

