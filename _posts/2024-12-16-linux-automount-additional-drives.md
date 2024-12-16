---
layout: post
title: "Use fstab to automount additional drives"
date: 2024-12-16 15:32:00 +0000
categories: linux fstab admin
published: true
---
Some additional notes after reading the Arch wiki entry for [fstab](https://wiki.archlinux.org/title/Fstab).

The following example relates to an installation of [EndevourOS](https://endeavouros.com/) with the following storage hardware configuration:
- M.2 NVMe as the boot drive, which also contains the root and swap partitions
- SATA SSD as an additional drive
- SATA HDD (x2) as an additional drive, configured as RAID-1 with [mdadm](https://wiki.archlinux.org/title/RAID#Implementation)

For reference, the EndevourOS installation created `/etc/fstab` (note: the UUIDs shown in the example that follows have been shortened for presentation purposes)

```sh
# <file system>  <mount point>  <type>  <options>  <dump>  <pass>
UUID=DE64-2685   /efi           vfat    fmask=0137,dmask=0027 0 2
UUID=d3afc3ed    /              ext4    noatime    0 1
UUID=9e0e8169    swap           swap    defaults   0 0
tmpfs            /tmp           tmpfs   defaults,noatime,mode=1777 0 0
```


### 1. Create mount points

This is a single user computer so the mount points will be made inside this user's home directory. Both additional storage devices are formatted with a single partition. The RAID device will be mounted as `~/Storage` and the SSD device as `~/Recordings`. Mount points are just directories and are created with [mkdir](https://man.archlinux.org/man/mkdir.1)

```sh
mkdir ~/Storage
mkdir ~/Recordings
```


### 2. Determine device id information

Block device information can be listed with [lsblk](https://man.archlinux.org/man/lsblk.8).  

```sh
lsblk -f
```

With this hardware, following report is produced (note: in the example that follows, not all of output columns are shown, and the UUIDs have again been shortened for presentation purposes)

```sh
NAME        FSTYPE            FSVER LABEL          UUID                   
sda                                                                       
└─sda1      linux_raid_member 1.2   Storage        cc32d3aa               
  └─md127   ext4              1.0                  165b24ba
sdb                                                                       
└─sdb1      linux_raid_member 1.2   Storage        cc32d3aa               
  └─md127   ext4              1.0                  165b24ba
sdc                                                                       
└─sdc1      ext4              1.0                  61bd98ee
nvme0n1                                                                   
├─nvme0n1p1 vfat              FAT32                DE64-2685              
├─nvme0n1p2 ext4              1.0   endevouros     d3afc3ed
└─nvme0n1p3 swap              1     swap           9e0e8169
```


### 3. Add entries to fstab and test

Entries can identify storage devices by name and label, but for consistency with the installation UUIDs will be used here. For the RAID device, the UUID required is the one assigned to the ext4 partition. Editing `/etc/fstab` requires `sudo` or to else to be root. The new `/etc/fstab` (note: the UUIDs have been shortened, etc.)

```sh
# <file system>  <mount point>  <type>  <options>  <dump>  <pass>
UUID=DE64-2685   /efi           vfat    fmask=0137,dmask=0027 0 2
UUID=d3afc3ed    /              ext4    noatime    0 1
UUID=9e0e8169    swap           swap    defaults   0 0
UUID=165b24ba    /home/kdenshi/Storage ext4 defaults,noatime 0 2
UUID=61bd98ee    /home/kdenshi/Recordings ext4 defaults,noatime 0 2
tmpfs            /tmp           tmpfs   defaults,noatime,mode=1777 0 0
```

Any whitespace delineates columns so precisely tab spacing the entries is not necessary. Test the new fstab with [mount](https://man.archlinux.org/man/mount.8) - with `sudo` or as root

```sh
mount -a
```

This will mount the devices. For ext4 formatted devices it may be required to set filesystem permissions for the user. This should only be required once (i.e. not every time the drive is mounted). If the device is not empty it may be better to not do this recursively (omit the `-R` switch) to change only the ownership of the mount point and leave the rest of the device's filesystem permissions intact.
<br />
**Note that the following command will recursively set permissions across the whole of the device, so one must think before making copypasta if the device already contains data.**
<br />
With `sudo` or as root

```sh
chown -R kdenshi:kdenshi /home/kdenshi/Storage
chown -R kdenshi:kdenshi /home/kdenshi/Recordings
```

All drives should now automatically mount to the mount points specified and be fully accessible to the user.   

