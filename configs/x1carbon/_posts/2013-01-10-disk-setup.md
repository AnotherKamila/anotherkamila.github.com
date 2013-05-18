---
title: Disk setup
tagline: partitions, filesystems, UEFI+BIOS boot
layout: configs_post
excerpt: |
    What I wanted:

    - a UEFI boot without a separate bootloader (i.e. bootstrap by the kernel EFISTUB loader)
    - a backup BIOS boot with GRUB(2) just in case (it has proved useful a few times already)
    - Btrfs (yeah, I will write about my backup scheme one day :D)

    What I did not want:

    - dualboot ;-)
---

#### This poor draft of an article is not finished yet! I am just publishing it in order to force myself into writing it before I forget everything.

My X1 Carbon has a 256GB SanDisk SD5SG2256G1052E SSD (and it's blazing fast :P).

What I wanted:

- a UEFI boot without a separate bootloader (i.e. bootstrap by the kernel EFISTUB loader)
- a backup BIOS boot with GRUB(2) just in case (it has proved useful a few times already)
- Btrfs (yeah, I will write about my backup scheme one day :D)

What I did not want:

- dualboot ;-)

Partition Table
---------------

For UEFI boot, the GPT partition scheme is necessary. Therefore one needs to use `gdisk` instead of
`fdisk` to setup partitions.

Here is how I set up mine:

    Number  Start (sector)    End (sector)  Size       Code  Name
       1            2048          264191   128.0 MiB   EF00  EFI System
       2          264192          266239   1024.0 KiB  EF02  BIOS boot partition
       3          266240       482611199   230.0 GiB   8300  Linux filesystem
       4       482611200       500118158   8.3 GiB     8200  Linux swap



`sda1` is the EFI boot partition (type `EF00`) and `sda2` the BIOS/GRUB one (see below for the boot
setup), `sda4` is swap (so large because I want to use hibernation), and all the remaining space
went to `sda3`, which was then set up as a Btrfs raw storage pool.


EFI + BIOS Boot
---------------

### EFI

First of all, reading ArchWiki's articles on [UEFI](https://wiki.archlinux.org/index.php/UEFI) and
[UEFI bootloaders](https://wiki.archlinux.org/index.php/UEFI_Bootloaders) or something comparably
useful is sort of required in order to figure out what I am talking about here. I just provide a
quick summary of what I did, not a thorough explanation of all the UEFI stuff.

When booting in EFI mode, I use the EFISTUB bootloader built into the kernel (stock Arch kernels
have had it compiled in since fall 2012). This means that it "Just Works" (unless it doesn't).
[The ArchWiki recommends](https://wiki.archlinux.org/index.php/Beginners%27_Guide/Installation#For_UEFI_motherboards)
using rEFInd to get a nice menu and set the kernel commandline parameters, but I have decided to
bake the commandline into the firmware and boot the kernel directly instead.[^boot_parameters]

To edit the EFI boot menu, one needs to boot in EFI mode[^boot_chicken]. Then I modprobe'd `efivars`
and used `efibootmgr` to add the boot menu entry (including the kernel commandline, since I have no
intermediate bootstrapping something in between the kernel and the firmware).

Note for the future me: this is the `efibootmgr` command:

    efibootmgr -c -g -d /dev/sda -p1 -L ArchLinux -l "\EFI\arch\vmlinuz-linux.efi" -u "`cat /boot/efi/EFI/arch/linux.conf`"

(Note for the past me: Thanks! :D)

I also added an [EFI shell](https://wiki.archlinux.org/index.php/UEFI#UEFI_Shell) just in case. It
was a good idea :P

A side note: If a new kernel is installed but does not get copied into `/boot/efi`, it will not
boot and there will be no error message. Therefore,
[this](https://wiki.archlinux.org/index.php/UEFI_Bootloaders#Sync_EFISTUB_Kernel_in_UEFISYS_partition_using_Systemd)
(or other methods listed below) is a great idea.

### BIOS+GRUB

I somehow installed GRUB2 onto the BIOS boot partition (`sda2` in my case), and it somehow works --
the computer happily boots with GRUB in BIOS mode and with EFISTUB in UEFI mode. Unfortunately I
can't remember the details of how exactly I got it to work. But that probably signifies that it
wasn't hard, so it might actually be a good thing O:-)

[^boot_parameters]: There is a patch to get the kernel to read its commandline from a file named
`linux.conf` in the same directory as the kernel image instead of from the boot menu in the
firmware, but this works, too.

[^boot_chicken]: Does that smell like the good old chicken-and-egg problem? I needed to
[create an EFI-capable liveUSB](https://wiki.archlinux.org/index.php/UEFI#Create_UEFI_bootable_USB_from_ISO)
first.

Filesystem
----------

My whole system besides `/boot/{EFI,GRUB}` resides on one [btrfs](https://btrfs.wiki.kernel.org/‎)
partition, with different subvolumes for `/`, `/home` and `/var`. With the awesome snapshot
capabilities of btrfs, this allows for neat rollback in case an update breaks something, makes
online backup terribly simple and just generally rocks.[^btrfs_scripts]

I chose LZO compression, as some benchmark out there claimed btrfs with LZO was the fastest option
on SSDs similar to mine. I might run my own benchmarks one day.

[^btrfs_scripts]: I will release my fancy scripts at some point in the future.

Here is my `/etc/fstab`:

    # <file system> <dir>     <type>  <options>             <dump>  <pass>

    LABEL=btr       /         btrfs   rw,noatime,ssd,space_cache,compress=lzo,subvol=root       0 1
    LABEL=btr       /var      btrfs   rw,noatime,ssd,space_cache,compress=lzo,subvol=var        0 2
    LABEL=btr       /home     btrfs   rw,noatime,ssd,space_cache,compress=lzo,subvol=home       0 2
    tmpfs           /tmp      tmpfs   nodev,nosuid,size=5G                                      0 0
    LABEL=swap      none      swap    defaults                                                  0 0

    LABEL=btr       /btrroot  btrfs   noauto,rw,noatime,ssd,space_cache,compress=lzo,subvolid=0 0 2

    UUID=4ECD-9824  /boot/efi vfat    rw,noatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro 0 2
