# nova-install

## Linux versions supported
**5.1 (current master), 5.0, 4.19, 4.18, 4.14, 4.13. Checkout each branch if you are interested.

## Environment
```
#lsb_release -a
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.4 LTS
Release:	18.04
Codename:	bionic

#uname -r 
Kernel : 4.15
```
## Setup
```
git clone ‘https://github.com/NVSL/linux-nova.git’
unzip linux-nova-5.1.zip
cd linux-nova-5.1
make menuconfig
```
*/bin/sh: 1: gcc: not found
  HOSTCC  scripts/basic/fixdep
/bin/sh: 1: gcc: not found
scripts/Makefile.host:92: recipe for target 'scripts/basic/fixdep' failed
make[1]: *** [scripts/basic/fixdep] Error 127
Makefile:489: recipe for target 'scripts_basic' failed
make: *** [scripts_basic] Error 2, then install Gcc && ncurses (ncurses-devel) && flex && bison as below:

* sudo apt-get update
* sudo apt-get upgrade
* sudo apt-get install build-essential
* sudo apt-get install ncurses-dev
* apt-get install flex
* apt-get install bison

**Kernel Configuration**
```
Device Drivers  --->
        <*> NVDIMM (Non-Volatile Memory Device) Support  --->
            <*>   PMEM: Persistent memory block device support (NEW)
            <*>   BLK: Block data window (aperture) device support (NEW)
            [*]   BTT: Block Translation Table (atomic sector updates)
    Processor type and features  --->
        <*> Support non-standard NVDIMMs and ADR protected memory
    Memory Management options  --->
        [*] Allow for memory hot-add
        [*]   Online the newly added memory blocks by default
        [*]   Allow for memory hot remove
        [*] Device memory (pmem, HMM, etc...) hotplug support
    File systems  --->
        [*] Direct Access (DAX) support
        <M>   NOVA: log-structured file system for non-volatile memories
```
```
make -j4 
make modules
make modules_install 
make install
```
*fatal error: openssl/opensslv.h: No such file or directory, then install lib as below:
* apt-get install libssl-dev

## Emulate Persistent Memory and Mount a folder
```
vim /etc/default/grub

*Add `memmap=3G!4G` in the line of `GRUB_CMDLINE_LINUX`
*How To Choose the Correct memmap Option for Your System

dmesg | grep BIOS
[    0.000000] BIOS-provided physical RAM map:
[    0.000000] BIOS-e820: [mem 0x0000000000000000-0x000000000009fbff] usable
[    0.000000] BIOS-e820: [mem 0x000000000009fc00-0x000000000009ffff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000000f0000-0x00000000000fffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000000100000-0x00000000dffeffff] usable
[    0.000000] BIOS-e820: [mem 0x00000000dfff0000-0x00000000dfffffff] ACPI data
[    0.000000] BIOS-e820: [mem 0x00000000fec00000-0x00000000fec00fff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fee00000-0x00000000fee00fff] reserved
[    0.000000] BIOS-e820: [mem 0x00000000fffc0000-0x00000000ffffffff] reserved
[    0.000000] BIOS-e820: [mem 0x0000000100000000-0x000000021fffffff] usable
//4096MB(4GB)- 8703MB(8.7GB) usable
//reserve the 3GiB usable space between 4GiB and 7GiB as emulated persistent memory.

grub-mkconfig -o /boot/grub/grub.cfg
update-grub
reboot
*after reboot machine, you can select the ubuntu 5.1.0 in GRUB boot menu

modprobe nova
mkdir /mnt/ramdisk
mount -t NOVA -o init /dev/pmem0 /mnt/ramdisk
```


