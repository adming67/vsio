
#### Switch to root user

```bash
mohsin@ubsvr:~$ sudo su
[sudo] password for mohsin:
root@ubsvr:/home/mohsin#
```

#### Partition Status

```bash
root@ubsvr:/home/mohsin# fdisk -l | grep /dev/sd

Disk /dev/sda: 127 GiB, 136365211648 bytes, 266338304 sectors
/dev/sda1     2048      4095      2048    1M BIOS boot
/dev/sda2     4096   4198399   4194304    2G Linux filesystem
/dev/sda3  4198400 266336255 262137856  125G Linux filesystem
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
```
#### Partitioning Using fdisk

```bash
root@ubsvr:/home/mohsin# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
```

#### Create New Partition

```bash
Command (m for help): n
Partition number (1-128, default 1):
First sector (34-209715166, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715166, default 209715166):

Created a new partition 1 of type 'Linux filesystem' and of size 100 GiB.

Command (m for help): p
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: Virtual Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: EEFEE8B7-6A5D-48A1-9AED-4C76751468C4

Device     Start       End   Sectors  Size Type
/dev/sdb1   2048 209715166 209713119  100G Linux filesystem
```

#### Set Partition Type to LVM (using option 8E or GUID)

```bash
Command (m for help): t
Selected partition 1
Partition type or alias (type L to list all): E6D6D379-F507-44C2-A23C-238F2A3DF928
Changed type of partition 'Linux filesystem' to 'Linux LVM'.

Command (m for help): p
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Disk model: Virtual Disk
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: EEFEE8B7-6A5D-48A1-9AED-4C76751468C4

Device     Start       End   Sectors  Size Type
/dev/sdb1   2048 209715166 209713119  100G Linux LVM

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

#### List Partitions

```bash
root@ubsvr:/home/mohsin# fdisk -l | grep /dev/sd

Disk /dev/sda: 127 GiB, 136365211648 bytes, 266338304 sectors
/dev/sda1     2048      4095      2048    1M BIOS boot
/dev/sda2     4096   4198399   4194304    2G Linux filesystem
/dev/sda3  4198400 266336255 262137856  125G Linux filesystem
Disk /dev/sdb: 100 GiB, 107374182400 bytes, 209715200 sectors
/dev/sdb1   2048 209715166 209713119  100G Linux LVM
```

### LVM Commands
#### List Physical Volumes

```bash
root@ubsvr:/home/mohsin# pvs
  PV         VG  Fmt  Attr PSize    PFree
  /dev/sda3  vgm lvm2 a--  <125.00g    0
```

#### Create PV

```bash
root@ubsvr:/home/mohsin# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
```

```Result
root@ubsvr:/home/mohsin# pvs
  PV         VG  Fmt  Attr PSize    PFree
  /dev/sda3  vgm lvm2 a--  <125.00g       0
  /dev/sdb1      lvm2 ---  <100.00g <100.00g
```
#### Create New Volume Group

```bash
root@ubsvr:/home/mohsin# vgcreate -s 16M vgn /dev/sdb1
  Volume group "vgn" successfully created
```

```Result
root@ubsvr:/home/mohsin# pvs
  PV         VG  Fmt  Attr PSize    PFree
  /dev/sda3  vgm lvm2 a--  <125.00g     0
  /dev/sdb1  vgn lvm2 a--    99.98g 99.98g
```

```Result
root@ubsvr:/home/mohsin# vgs
  VG  #PV #LV #SN Attr   VSize    VFree
  vgm   1   3   0 wz--n- <125.00g     0
  vgn   1   0   0 wz--n-   99.98g 99.98g
```

```Before
root@ubsvr:/home/mohsin# lvs
  LV   VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0 vgm -wi-ao----   8.00g
  lv-1 vgm -wi-ao----  50.00g
  lv-2 vgm -wi-ao---- <67.00g
```

#### Create a Logical Volume of 10G

```bash
root@ubsvr:/home/mohsin# lvcreate -L 10G -n library vgn
  Logical volume "library" created.
```

```Result
root@ubsvr:/home/mohsin# lvs
  LV      VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0    vgm -wi-ao----   8.00g
  lv-1    vgm -wi-ao----  50.00g
  lv-2    vgm -wi-ao---- <67.00g
  library vgn -wi-a-----  10.00g
```

```Result
root@ubsvr:/home/mohsin# vgs
  VG  #PV #LV #SN Attr   VSize    VFree
  vgm   1   3   0 wz--n- <125.00g     0
  vgn   1   1   0 wz--n-   99.98g 89.98g
```

#### Extend the Logical Volume

```bash
root@ubsvr:/home/mohsin# lvextend -l +100%FREE /dev/vgn/library
  Size of logical volume vgn/library changed from 10.00 GiB (640 extents) to 99.98 GiB (6399 extents).
  Logical volume vgn/library successfully resized.
```

```Result
root@ubsvr:/home/mohsin# lvs
  LV      VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv-0    vgm -wi-ao----   8.00g
  lv-1    vgm -wi-ao----  50.00g
  lv-2    vgm -wi-ao---- <67.00g
  library vgn -wi-a-----  99.98g
```

#### Create ext4 File System with Label

```bash
root@ubsvr:/home/mohsin# mkfs.ext4 -L library /dev/vgn/library
mke2fs 1.46.5 (30-Dec-2021)
Discarding device blocks: done
Creating filesystem with 26210304 4k blocks and 6553600 inodes
Filesystem UUID: d5cab94a-839c-4aa5-b642-a3f05ca7dde0
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872

Allocating group tables: done
Writing inode tables: done
Creating journal (131072 blocks): done
Writing superblocks and filesystem accounting information: done
```

#### Create Mount Point

```bash
root@ubsvr:/home/mohsin# mkdir /library
```

#### Add Mount Point to /etc/fstab

```bash
root@ubsvr:/home/mohsin# nano /etc/fstab
```

```bash
##### Add the following entry to /etc/fstab ######
/dev/mapper/vgn-library                                /library ext4 defaults 0 1
```
#### Mount the Partition

```bash
root@ubsvr:/home/mohsin# mount -a
```

#### Verify Partition Size

```Result
root@ubsvr:/home/mohsin# df -h
Filesystem               Size  Used Avail Use% Mounted on
tmpfs                    391M  2.6M  389M   1% /run
/dev/mapper/vgm-lv--1     49G  3.9G   43G   9% /
tmpfs                    2.0G     0  2.0G   0% /dev/shm
tmpfs                    5.0M     0  5.0M   0% /run/lock
/dev/sda2                2.0G  145M  1.7G   8% /boot
/dev/mapper/vgm-lv--2     66G  5.5G   57G   9% /media
tmpfs                    391M  4.0K  391M   1% /run/user/1000
/dev/mapper/vgn-library   98G   24K   93G   1% /library
```

#### Add a New Partition to an existing LVM

```bash
pvcreate /dev/sdc1
```

```bash
vgextend vgn /dev/sdc1
```

```bash
lvextend -r -L +50G /dev/vgn/library

OR

lvextend -r -l +100%FREE /dev/vgm/var
```
- Option -r with resize the ext4 file system as well.
