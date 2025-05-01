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

---
WIP
#### 🧠 Understanding the Init Process

Just as I was about to jump into reversing the web server, a little voice in my head whispered:

> *"Wait... what if there's something even more interesting than the web interface? Like a hidden Telnet port for technicians that lets you read, write, and execute whatever you want — all by just typing a username and password?"*

That idea was too tempting to ignore.

To investigate whether such a backdoor exists, I decided to look into what happens during the system’s initialization. As usual, I summoned my ever-helpful assistant, ChatGPT, and asked:

> "Where is the init executable usually located?"

I was pointed to:

```bash
/sbin/init
```

So I ran a quick `ls`:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ ls -la sbin/init
lrwxrwxrwx 1 t_omersas t_omersas 14 Apr 12 09:49 sbin/init -> ../bin/busybox
```

No surprises here — it’s just a symlink to BusyBox. Time to peek inside:

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

Yup — `init` is definitely there. Let’s try to run it:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox init
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
init: must be run as PID 1
```

Expected behavior. `init` refuses to run unless it’s process ID 1. At least we know it’s working.

But then I noticed another gem: `login`. Well, you know what I had to do...

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox login
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
PC login: a
Password:
Login incorrect
PC login: ^C
```

Tried the classics — `admin:admin`, `root:root`, `1234`, `toor`... nothing worked. 😢  
I decided to shelve this part for now and turn my attention back to `init`.

That error message earlier — *"must be run as PID 1"* — gave me a lead. Searching for it in IDA landed me in a very interesting place. Say hello to:

```c
int __fastcall init_handler(int a1, _DWORD *a2)
{
  ...
}
```

(This function is... enormous. Scroll up if you want the full thing.)

I didn’t go too deep into analyzing it because:
- It relies on a ton of helper functions like `sub_435A50`, `sub_42ABA8`, etc.
- I don’t have access to this modified BusyBox source.
- Some strings I saw here don’t even exist in the official BusyBox repo.

But here’s the cool discovery: when I looked at the cross-references (xrefs) to `init_handler`, I saw it was part of what looked like a **function pointer table** — a list of all BusyBox command handlers.

And sure enough, that list appeared to mirror exactly what we saw earlier in the BusyBox function output:

```bash
arping, ash, brctl, cat, chmod, cp, date, df, echo, free, getty, halt,
ifconfig, init, insmod, ipcrm, ipcs, kill, killall, linuxrc, login, ls,
lsmod, mkdir, mount, netstat, pidof, ping, ping6, poweroff, ps, reboot,
rm, rmmod, route, sh, sleep, taskset, tftp, top, umount, vconfig
```

This led me to manually start naming functions in the table. Here's a sample:

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

And just like that, I had my next objective:  
🔐 *Find out how `login` works and where the credentials are checked...*

Stay tuned.

#### 🔐 Login Credentials

Alright, time to zoom in on the authentication process.

First off, some context: `/bin/login` in this firmware is just a symlink to BusyBox. That means when we invoke `/bin/login`, we’re effectively calling `busybox login`. I didn’t prove this explicitly, but it’s a pretty safe assumption — and IDA seems to agree.

So, I asked myself: *"Who’s actually calling `/bin/login`?"*

A quick string reference search pointed me to `getty_handler`. According to ChatGPT (and also the real world), `getty` is the utility that manages physical or virtual terminals. It listens on a tty, and when someone connects, it hands over to `login`.

Cool. That means login only gets triggered when someone connects via serial or similar interface.

But the juiciest part came next.

I searched IDA for XRefs to the `"Password:"` string — and landed on this absolute beauty:

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

This is gold.

Depending on the value of `is_adminqq`, the login prompt expects either `"sohoadmin"` (for admins) or `"aa"` (for regular users).

Naturally, I fired up QEMU and tried `"sohoadmin"` — but it just printed `Login incorrect`.

Then I tried `"aa"`... and the whole program *crashed*.

Weird. Time to find out:
- Why the crash?
- And more importantly — how do I become `is_admin == true`?

Turns out, the `is_admin` flag is coming from `sub_438728`, which internally calls:

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

Looks like it opens `/etc/passwd`, reads from it, and uses that to determine privileges.

Then I remembered something from earlier during firmware extraction...

```warning
WARNING: Symlink points outside of the extraction directory: /home/t_omersas/temp/_TPLINK.bin-0.extracted/squashfs-root/etc/passwd -> /var/passwd; changing link target to /dev/null for security purposes.
```

Ah. That might explain everything. If `/etc/passwd` is pointing to `/dev/null`, then no user will ever be found, and the `is_admin` check might misbehave — or worse, crash.

So I fixed the symlink manually:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root/etc $ rm -rf passwd
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root/etc $ sudo ln -s /var/passwd passwd
```

Then I tried creating a chroot environment and running the login from inside:

```bash
cp /usr/bin/qemu-mipsel-static squashfs-root/usr/bin/
sudo chroot squashfs-root /usr/bin/qemu-mipsel-static /bin/sh
```

Once inside, I invoked:

```bash
/usr/bin/qemu-mipsel-static /bin/busybox login
```

...but sadly, this happened:

```bash
/ # /usr/bin/qemu-mipsel-static /bin/busybox login
PC login: a
Password:
qemu: uncaught target signal 11 (Segmentation fault) - core dumped
Segmentation fault (core dumped)
```

So either there’s still something missing… or this rabbit hole goes deeper than I thought. 😵‍💫

---
##### 🧩 Discovering the Source Code

At this point, I was *this close* to giving up and just buying the physical router — but then it hit me:  
*"Wait a minute... what if the BusyBox source code is similar enough to reverse what's going on?"*

So I went back to basics and ran:

```bash
t_omersas@PC ~/projects/_TL_WR840N.bin.extracted/squashfs-root $ sudo qemu-mipsel -L . ./bin/busybox
./bin/busybox: cache '/etc/ld.so.cache' is corrupt
BusyBox v1.19.2 (2024-12-30 15:52:05 CST) multi-call binary.
```

Boom — version `v1.19.2`. That's all I needed.

I grabbed the source from here: 👉 [BusyBox 1.19 Source](https://github.com/mirror/busybox/tree/1_19_stable)

Inside `libbb/correct_password.c`, I found the upstream implementation of BusyBox's password validation:

```c
/* Ask the user for a password.
 * Return 1 if the user gives the correct password for entry PW,
 * 0 if not.  Return 1 without asking if PW has an empty password.
 *
 * NULL pw means "just fake it for login with bad username" */

int FAST_FUNC correct_password(const struct passwd *pw)
{
    char *unencrypted, *encrypted;
    const char *correct;
    int r;
#if ENABLE_FEATURE_SHADOWPASSWDS
    /* Using _r function to avoid pulling in static buffers */
    struct spwd spw;
    char buffer[256];
#endif

    /* fake salt. crypt() can choke otherwise. */
    correct = "aa";
    if (!pw) {
        /* "aa" will never match */
        goto fake_it;
    }
    correct = pw->pw_passwd;
#if ENABLE_FEATURE_SHADOWPASSWDS
    if ((correct[0] == 'x' || correct[0] == '*') && !correct[1]) {
        /* getspnam_r may return 0 yet set result to NULL.
         * At least glibc 2.4 does this. Be extra paranoid here. */
        struct spwd *result = NULL;
        r = getspnam_r(pw->pw_name, &spw, buffer, sizeof(buffer), &result);
        correct = (r || !result) ? "aa" : result->sp_pwdp;
    }
#endif

    if (!correct[0]) /* empty password field? */
        return 1;

 fake_it:
    unencrypted = bb_ask_stdin("Password: ");
    if (!unencrypted) {
        return 0;
    }
    encrypted = pw_encrypt(unencrypted, correct, 1);
    r = (strcmp(encrypted, correct) == 0);
    free(encrypted);
    memset(unencrypted, 0, strlen(unencrypted));
    return r;
}
```

It’s a pretty classic login mechanism — reads from `/etc/shadow`, hashes the input, compares, etc.

But in **our firmware**?

Here’s what the same function looks like:

```c
BOOL __fastcall sub_42CD50(int a1)
{
  const char *v1; // $a1
  int v2; // $v0
  int v3; // $s1
  BOOL v4; // $s0
  int v5; // $v0
  const char *v7; // [sp+18h] [-Ch]

  if ( a1 )
    v1 = "sohoadmin";
  else
    v1 = "aa";
  v7 = v1;
  v2 = sub_437BDC("Password: ");
  v3 = v2;
  v4 = 0;
  if ( v2 )
  {
    v4 = strcmp(v2, v7) == 0;
    v5 = strlen(v3);
    memset(v3, 0, v5);
  }
  return v4;
}
```

Literally just checks if the input is `"sohoadmin"` or `"aa"`. That’s it.

---

##### 🔐 Why `correct = "aa"`?

In upstream BusyBox, `"aa"` is used as a fake hash — a decoy. If the username doesn’t exist, it still asks for a password and compares it against `"aa"` using `crypt()`, which guarantees failure. This helps prevent **timing attacks** — attackers shouldn’t be able to tell whether the username or the password was invalid.

But here? There's no hashing, no `/etc/shadow`, no crypt at all. The firmware just says:

> "If you're admin, password is `'sohoadmin'`; otherwise, it's `'aa'`. Deal with it."

---

Interestingly, it seems the only actual use of `login` in the entire system is in `/etc/inittab`:

```
2:ttyS1::askfirst:/bin/login
```

Meaning: the login flow is only triggered via **serial** connections (like UART) — not over the network.

So unless we have physical access to the router’s serial pins...  
🫠 *this entire login system might just be a decoy.*

---
