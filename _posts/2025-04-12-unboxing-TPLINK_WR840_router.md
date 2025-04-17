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

