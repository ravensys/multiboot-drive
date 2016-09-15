# RavenSystems MultiBoot Drive


## Preparing USB Flash Disk

### Create and format partitions

**Hybrid UEFI-GPT and BIOS-GPT/MBR** USB disk requires GPT partition table present on device and at least 3 partitions:

* A BIOS boot partition, of size exactly 1 MiB (partition type EF02, not formatted),
* An EFI system partition, of size at least 50 MiB (partition type EF00, with FAT32 filesystem),
* Data partition, cat take rest of the space of drive (filesystem supported by GRUB).

```
$ sudo gdisk /dev/sdX

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-61457630, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-61457630, default = 61457630) or {+-}size{KMGTP}: +1M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): EF02
Changed type of partition to 'BIOS boot partition'

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-61457630, default = 4096) or {+-}size{KMGTP}: 
Last sector (4096-61457630, default = 61457630) or {+-}size{KMGTP}: +50
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): EF00
Changed type of partition to 'EFI System'

Command (? for help): n
Partition number (3-128, default 3): 
First sector (34-61457630, default = 6144) or {+-}size{KMGTP}: 
Last sector (6144-61457630, default = 61457630) or {+-}size{KMGTP}: 
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdX.
The operation has completed successfully.
```

### Create hybrid MBR partition:

```
$ sudo gdisk /dev/sdX

Command (? for help): r

Recovery/transformation command (? for help): h

WARNING! Hybrid MBRs are flaky and dangerous! If you decide not to use one,
just hit the Enter key at the below prompt and your MBR partition table will
be untouched.

Type from one to three GPT partition numbers, separated by spaces, to be
added to the hybrid MBR, in sequence: 1 2 3
Place EFI GPT (0xEE) partition first in MBR (good for GRUB)? (Y/N): N

Creating entry for GPT partition #1 (MBR partition #1)
Enter an MBR hex code (default EF): 
Set the bootable flag? (Y/N): N

Creating entry for GPT partition #2 (MBR partition #2)
Enter an MBR hex code (default EF): 
Set the bootable flag? (Y/N): N

Creating entry for GPT partition #3 (MBR partition #3)
Enter an MBR hex code (default 83): 
Set the bootable flag? (Y/N): Y

Recovery/transformation command (? for help): x

Expert command (? for help): h

Expert command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): Y
OK; writing new GUID partition table (GPT) to /dev/sdX.
The operation has completed successfully.
```

### Install GRUB2

Data and EFI system partitions created on the USB drive must be mounted.

Install GRUB2 for EFI systems by running:
```
$ sudo grub2-install --target=x86_64-efi --efi-directory=/EFI_MOUNTPOINT --boot-directory=/DATA_MOUNTPOINT/boot --removable --recheck
```

Install GRUB2 for BIOS systems by running:
```
sudo grub2-install --target=i386-pc --boot-directory=/mnt/multiboot/boot --recheck /dev/sdX
```

## Credits
This project is mostly based on ArchLinux wiki article [Multiboot USB drive](https://wiki.archlinux.org/index.php/Multiboot_USB_drive).
