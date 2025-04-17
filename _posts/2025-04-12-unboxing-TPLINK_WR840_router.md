# 🧪 Hacking Routers on Passover: Because Why Not?
## Introduction

> *“While you were out there finding friends, I was studying the firmware.”* — Me, probably

---

Passover arrived earlier than expected this year, and with it came a tough decision:

- 👨👩👧👦 Be social, find friends, and celebrate like a normal human being.
- 🔧 Unbox some dusty hardware and try to exploit it for fun.

If you've read even one of my blog posts, you already know which route I took. Spoiler: my social skills are still stuck in bootloader mode.

Before diving into the binary abyss, I needed to find a worthy target. And who better to ask than my digital partner-in-crime, ChatGPT?

---

**👤 Me:**  
> I want to research an embedded device for fun and try to exploit it (all for educational purposes, of course). But I'm on a budget. Any cheap and interesting suggestions?

**🤖 ChatGPT:**  
> 🔥 Popular Old Routers for Hacking and Research:  
>  1. **TP-Link TL-WR841N**

---

And just like that, I had my victim:  
✅ Cheap  
✅ Interesting  
✅ Practically begging to be hacked

Let the chaos begin. 🧨
## 🔬 Research

### 🛒 Pre-Purchase Intel Gathering

#### 🧠 Find the Firmware, Find the Fun

Before spending even a single shekel, I wanted to peek inside the firmware and get a feel for what I’d be up against. Naturally, I turned once again to my loyal research assistant — ChatGPT.

> *“Show me the bits, O wise one.”*

And voilà — ChatGPT delivered. It led me to a **firmware treasure trove** straight from TP-Link’s official website:

👉 [TL-WR840N Firmware Downloads](https://www.tp-link.com/ae/support/download/tl-wr840n/?utm_source=chatgpt.com#Firmware)

Without overthinking (or reading anything, to be honest), I downloaded the first shiny thing that caught my eye:

> 🧾 **TL-WR840N(EU)\_V6.20\_241230**

Because who needs documentation when you have curiosity and reckless optimism?

#### 🔧 Extracting the Firmware

Let’s crack this thing open. The goal? Dive deep into the firmware and see what treasures are buried inside. Spoiler alert: it was surprisingly easy.

Just toss the binary into `binwalk` and let the magic happen:

```bash
t_omersas@PC /tmp $ binwalk -e TL_WR840N.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
54592         0xD540          U-Boot version string, "U-Boot 1.1.3 (Dec 30 2024 - 15:49:18)"
66560         0x10400         LZMA compressed data, properties: 0x5D, dictionary size: 8388608 bytes, uncompressed size: 2986732 bytes
...
1049088       0x100200        Squashfs filesystem, little endian, version 4.0, compression:xz, ...
```

You’ll also see some friendly warnings about symlinks being redirected. Just binwalk things.

Now let’s dig into what got extracted:

```bash
t_omersas@PC /tmp $ cd _TL_WR840N.bin.extracted
t_omersas@PC /tmp/_TL_WR840N.bin.extracted $ ls
100200.squashfs  10400  10400.7z  squashfs-root  squashfs-root-0
```

Curious, I ran `binwalk` again on `10400` and voilà — kernel time:

```bash
t_omersas@PC /tmp/_TL_WR840N.bin.extracted $ binwalk 10400

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
2240616       0x223068        Linux kernel version 2.6.36
...
2982244       0x2D8164        ASCII cpio archive (SVR4 with no CRC), file name: "TRAILER!!!"
```

Yep, we’re staring right at the Linux kernel. Classic 2.6.x vintage.

But the real gem is in the SquashFS directory. Whether you check `squashfs-root` or `squashfs-root-0` (they’re twins), or extract `100200.squashfs` directly, you’ll end up with the same tasty filesystem:

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
...
```

It’s a beautiful mess of classic embedded Linux goodies:

- 🧰 BusyBox tools
- 🧠 `/etc` configs
- 🕸 Web server content in `/web`
- 🧱 Full rootfs including `/bin`, `/sbin`, `/lib`, `/proc`, and more

We officially have ourselves a functioning Linux system stuffed inside a little plastic router. The fun begins now 🧨

#### 🧪 Attempting Kernel Emulation (Spoiler: It Failed)

At this point in the journey, I thought: *"Why not emulate the Linux kernel? Sounds fun!"*  
The reasons behind this brave but naive decision:

- 🧼 I had no proper setup yet.
- 🧠 I had never touched QEMU or any emulation tool before.
- 🤷‍♂️ I love learning by diving into the deep end.

Looking back... would I do it again?  
**Absolutely not.** Kernel emulation for this device turned out to be inefficient and unnecessary. Instead, it's far more rational to focus on user-mode applications (especially the ones lovingly crafted by TP-Link's devs).

But let’s walk through the attempt anyway — for science.

---

##### 🧭 Discover the Architecture

First, I needed to figure out the target architecture. Fortunately, `binwalk` delivered clear results:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted $ binwalk -A 10400

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
50140         0xC3DC          MIPSEL instructions, function epilogue
...
```

Yep — it’s MIPS, little endian (MIPSEL). Let's try to run this thing with QEMU!

---

##### 🧨 Try to Boot It

```bash
qemu-mips 10400
```

Result:

```
Error while loading /tmp/_TL_WR840N.bin.extracted/10400: Exec format error
```

Turns out QEMU expects an ELF file, not a raw binary blob.  
So, I took the advice of ChatGPT and tried to turn this thing into an ELF:

```bash
mipsel-linux-gnu-objcopy -I binary -O elf32-tradlittlemips -B mips 10400 10400.elf
```

Created a `link.ld` file:

```ld
SECTIONS {
  . = 0x80000000;
  .text : { *(.text) }
  .data : { *(.data) }
  .bss  : { *(.bss) }
}
```

Linked it:

```bash
mipsel-linux-gnu-ld -T link.ld -o 10400_exec.elf 10400.elf
```

And ran it with QEMU:

```bash
qemu-system-mipsel -M malta -kernel 10400_exec.elf -initrd initrd.img \
-drive file=rootfs.sqfs,format=raw,if=mtd \
-append "root=/dev/mtdblock0 bash=ttyS0" -nographic
```

The result? 🧨
```
qemu-system-mipsel: Trap-and-Emul kernels (Linux CONFIG_KVM_GUEST) are not supported
```

---

##### 🔧 The Verdict

After chasing ghosts and grepping through forum threads, I discovered this heartbreaking note in the official QEMU docs:

> **MIPS Trap-and-Emul kernels (since QEMU 6.0)**  
> The MIPS Trap-and-Emul KVM host and guest support has been removed from Linux upstream kernel.  
> It is now deprecated.

So, I even downloaded **QEMU 5.0.0** in an act of desperation:

```bash
wget https://download.qemu.org/qemu-5.0.0.tar.xz
tar -xvf qemu-5.0.0.tar.xz
cd qemu-5.0.0
```

But that didn’t help either. QEMU spit out another cryptic error and slammed the door on my dreams.

```
rom: requested regions overlap (rom prom. free=0x00000000002d92ec, addr=0x0000000000002000)
qemu-system-mipsel: rom check and register reset failed
```

---

##### 💡 New Plan: Focus on Userland

So, I gave up kernel emulation and did something smarter:
> Focus on what **really matters** — the filesystem and, more specifically, the **web server**.

Because let’s be honest... that’s where the juicy bugs live.

#### 🌐 Finding the Web Server (a.k.a. "Where Are You Hiding, Little HTTPd?")

At this point, I was ready to find some real vulnerabilities — the kind that live in user-mode programs. Since embedded web servers are often **custom-written for each device**, I figured this would be the most promising attack surface.

> **💡 Tip:** Router web servers are almost always **user-mode binaries**, stored as regular files in the filesystem. They’re not baked into the Linux kernel — they’re started by an init script after the root filesystem is mounted.

So, I began the hunt in the `bin/` directory:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ ls -la bin
total 3636
drwxr-xr-x  2 t_omersas t_omersas    4096 Apr 12 12:56 .
...
-rwxr-xr-x  1 t_omersas t_omersas  262100 Apr 12 09:49 busybox
-rw-r--r--  1 t_omersas t_omersas 3449390 Apr 12 12:56 busybox.i64
...
```

Looks like everything here is just **symlinks to BusyBox** (classic).  
> **💡 Tip:** BusyBox provides a minimal implementation of `init`, `sh`, and dozens of core Unix tools in a single binary.  

I popped `busybox` into IDA, but didn't find anything web-related — no `index.htm`, no `http`, nothing that screams "I'm a web server!"

Still curious, I tried running it:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox
...
Currently defined functions:
        arping, ash, brctl, cat, chmod, cp, date, df, echo, free, getty, halt,
        ifconfig, init, insmod, ipcrm, ipcs, kill, killall, linuxrc, login, ls,
        lsmod, mkdir, mount, netstat, pidof, ping, ping6, poweroff, ps, reboot,
        rm, rmmod, route, sh, sleep, taskset, tftp, top, umount, vconfig
```

Nice! It works in QEMU. I even tried arping Google DNS for fun:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox arping 8.8.8.8
...
ARPING to 8.8.8.8 from 172.20.123.158 via eth0
^CSent 25 probe(s) (25 broadcast(s))
Received 0 reply (0 request(s), 0 broadcast(s))
```

(No response — probably expected in emulation.)

Still, no sign of a web server. So I dug deeper and found a potential lead:

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

Nothing useful — no `httpd` or any web server start script here. Back to poking around...

Finally, jackpot:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ ls usr/bin
afcd     cmxdns  dhcpd     dropbearkey    ebtables  igmpd      ipcrm   iptables  killall  pwdog      taskset  tdpd  top         wanType        xtables-multi
arping   cos     dnsProxy  dropbearmulti  free      ip         ipcs    iwconfig  noipdns  rt2860apd  tc       tftp  traceroute  wlNetlinkTool
ated_tp  dhcpc   dropbear  dyndns         httpd     ip6tables  ipping  iwpriv    ntpc     scp        tddp     tmpd  upnpd       wscd
```

**Boom. There it is.**  
Not only that — there’s also `tftp`, `dnsProxy`, and other networking tools. Let’s confirm `httpd` is really the one serving `/web`:

```
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ strings usr/bin/httpd | grep htm
...
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
```

Confirmed: this binary contains all the HTML files we saw under `/web/main`.

The web server has been found. Next step? Tear it apart 🧠🔍.

