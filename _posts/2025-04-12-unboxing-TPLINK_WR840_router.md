## Introduction
Passover has come earlier than I expected, and it means that I have two options:
    * Find friends and go hanging up with them on the holidays like a normal human being.
    * Unobxing devices for fun & try to find vulnerabillities inside them.
I think that if you have read at least one blogpost that I've posted, you might already know what I chose :)

So in order to start my journey, I had to first find a victim. And as a great fan of ChatGPT I did it by ask him for recommendations:

Me:
> I want to research an embedded device for fun purposes and try to exploit it for educational reasons. However, it costs a lot of money, so I want something that will be cheap, yet interesting enough. What can I do?

ChatGPT:
>🔥 Popular Old Routers for Hacking and Research
>        1. TP-Link TL-WR841N

Ok, so I found a victim, now let's start with the real shit.

## Research
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

>The webservers implementations of router almost always a user-mode program stored as a regular file in the filesystem, not part of the Linux kernel. The kernel does not load the web server itself. Instead, a user-space init system or startup script is responsible for launching it after the kernel has booted and mounted the root filesystem.



