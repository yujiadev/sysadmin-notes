# Storage and Filesystem Adminstration

## Primary Partition, Extended Partition, and Logical Partition
### Primary Partition
- Primary partition is a bootable partition that can contain an opeartion system.
- A disk can have up to four primary partitions (MBR limitation)
- One of the primary partitions cna be marked as active (bootable)

### Extended Partition
- An extended partition is a special type of primary partition that acts as 
"container" for logical partitions.
- Since MBR disks allow only four primary partitions, an extended partition is 
used to overcome this limitation.
- A disk can have only one extended partition. 
- The extended partition itself does not store date; it just holds logical 
partitions inside it.
- Example: /dev/sda3 (an extended partition that contains /dev/sda5, /dev/sda6)

### Logical Partition
- A logical partition exists inside an extended partition and is used to create 
more than four partitions on a disk.
- Logical partitions are numbered starting from 5(since 1-4 are reserved for 
primary/extended partitions).
- They are used for storing data, but usually **cannot be booted directly**.
- Example: **/dev/sda5**, **/dev/sda6** (logical partitions inside an extend
 partition)

Example Partition Layout (MBR Disk):
```bash 
/dev/sda1 → Primary (Bootable, e.g., `/boot`)
/dev/sda2 → Primary (e.g., `/`)
/dev/sda3 → Extended Partition (container)
   ├─ /dev/sda5 → Logical (e.g., `/home`)
   ├─ /dev/sda6 → Logical (e.g., `/var`)
   └─ /dev/sda7 → Logical (swap)
```

## Partition
A partition or a logical volume can be referred to generically as a volume.

Before using the fdisk, gdisk, or parted utility to create or modify a 
partition, do check currently available free space along with currently 
mounted system. (df and mount command)

```bash 
[yujia@localhost ~]$ df
Filesystem                 1K-blocks    Used Available Use% Mounted on
devtmpfs                        4096       0      4096   0% /dev
tmpfs                        3933748       0   3933748   0% /dev/shm
tmpfs                        1573500    9400   1564100   1% /run
/dev/mapper/rhel_vbox-root  37273600 5580980  31692620  15% /
/dev/sda1                     983040  357944    625096  37% /boot
/dev/mapper/rhel_vbox-home  18165760  184452  17981308   2% /home
tmpfs                         786748     104    786644   1% /run/user/1000
```

```bash 
[yujia@localhost ~]$ df -h
Filesystem                  Size  Used Avail Use% Mounted on
devtmpfs                    4.0M     0  4.0M   0% /dev
tmpfs                       3.8G     0  3.8G   0% /dev/shm
tmpfs                       1.6G  9.2M  1.5G   1% /run
/dev/mapper/rhel_vbox-root   36G  5.4G   31G  15% /
/dev/sda1                   960M  350M  611M  37% /boot
/dev/mapper/rhel_vbox-home   18G  181M   18G   2% /home
tmpfs                       769M  104K  769M   1% /run/user/1000
```

Note that tmpfs and devtmpfs are temporary filesystems stored in RAM, their 
contents are removed after a system reboot.

Command **mount** lists the format and mount options for each filesystem.

```bash 
/dev/mapper/rhel_vbox-home on /home type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
```

Command **findmnt** prints all mounted filesystems in a tree-like format.

```bash 
[yujia@localhost ~]$ findmnt
TARGET                       SOURCE     FSTYPE    OPTIONS
/                            /dev/mapper/rhel_vbox-root
│                                       xfs       rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32
├─/proc                      proc       proc      rw,nosuid,nodev,noexec,relatime
│ └─/proc/sys/fs/binfmt_misc systemd-1  autofs    rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5
├─/sys                       sysfs      sysfs     rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/kernel/security     securityfs securityf rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup           cgroup2    cgroup2   rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memo
│ ├─/sys/fs/pstore           pstore     pstore    rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/fs/bpf              bpf        bpf       rw,nosuid,nodev,noexec,relatime,mode=700
│ ├─/sys/fs/selinux          selinuxfs  selinuxfs rw,nosuid,noexec,relatime
│ ├─/sys/kernel/debug        debugfs    debugfs   rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/kernel/tracing      tracefs    tracefs   rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/fs/fuse/connections fusectl    fusectl   rw,nosuid,nodev,noexec,relatime
│ └─/sys/kernel/config       configfs   configfs  rw,nosuid,nodev,noexec,relatime
├─/dev                       devtmpfs   devtmpfs  rw,nosuid,seclabel,size=4096k,nr_inodes=975312,mode=755,
│ ├─/dev/shm                 tmpfs      tmpfs     rw,nosuid,nodev,seclabel,inode64
│ ├─/dev/pts                 devpts     devpts    rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmo
│ ├─/dev/mqueue              mqueue     mqueue    rw,nosuid,nodev,noexec,relatime,seclabel
│ └─/dev/hugepages           hugetlbfs  hugetlbfs rw,relatime,seclabel,pagesize=2M
├─/run                       tmpfs      tmpfs     rw,nosuid,nodev,seclabel,size=1573500k,nr_inodes=819200,
│ ├─/run/credentials/systemd-tmpfiles-setup-dev.service
│ │                          none       ramfs     ro,nosuid,nodev,noexec,relatime,seclabel,mode=700
│ ├─/run/credentials/systemd-sysctl.service
│ │                          none       ramfs     ro,nosuid,nodev,noexec,relatime,seclabel,mode=700
│ ├─/run/credentials/systemd-tmpfiles-setup.service
│ │                          none       ramfs     ro,nosuid,nodev,noexec,relatime,seclabel,mode=700
│ └─/run/user/1000           tmpfs      tmpfs     rw,nosuid,nodev,relatime,seclabel,size=786748k,nr_inodes
│   ├─/run/user/1000/gvfs    gvfsd-fuse fuse.gvfs rw,nosuid,nodev,relatime,user_id=1000,group_id=1000
│   └─/run/user/1000/doc     portal     fuse.port rw,nosuid,nodev,relatime,user_id=1000,group_id=1000
├─/boot                      /dev/sda1  xfs       rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32
└─/home                      /dev/mapper/rhel_vbox-home
                                        xfs       rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32
```

## The fdisk Utility
**fdisk** works with partitions created using the traditional Master Boot Record
(MBR) partitioning scheme.

**gdisk** and **parted** work with GUID Partition Table (GPT)

Check disk information with **fdisk -l**
```bash 
[root@localhost yujia]# fdisk -l
Disk /dev/sda: 60 GiB, 64424509440 bytes, 125829120 sectors
Disk model: VBOX HARDDISK   
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x003423eb

Device     Boot   Start       End   Sectors Size Id Type
/dev/sda1  *       2048   2099199   2097152   1G 83 Linux
/dev/sda2       2099200 125829119 123729920  59G 8e Linux LVM


Disk /dev/mapper/rhel_vbox-root: 35.61 GiB, 38235275264 bytes, 74678272 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rhel_vbox-swap: 6 GiB, 6442450944 bytes, 12582912 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/rhel_vbox-home: 17.39 GiB, 18668847104 bytes, 36462592 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

### Using fdisk interactively
```bash
[root@localhost yujia]# fdisk /dev/sda

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

This disk is currently in use - repartitioning is probably a bad idea.
It's recommended to umount all file systems, and swapoff all swap
partitions on this disk.


Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table
```

### Using fdisk: In a Nutshell
At the **fdisk** cli prompt, start with the print command (**p**) to examine the
partition table. This allows you to review the current entries in the partition
table. Assuming free space is available, you can then create a new (**n**) 
partition

With an extended partition, you can create a maximum of 12 logical partitions
on a drive.

### Using fdisk: Create a Partition
```bash
fdisk /dev/sdb
...
n   # Create a new partition
...
p   # Make it bootable (primary)
...
a   # Make it bootable byt toggle a bootable flag
...
1   # Partition number
...
w   # Write the partition information to the disk
```

### Using fdisk: Many Partition Types
**t** command change the partition type.

### Using fdisk: Delete a Partition
**d** command delete  the partition.
**w** command write the change to disk.

### Using fdisk: Create a Swap Partition
n, p, 1, 2048, +900M, p, t, **82**, w

## The gdisk Utility
**gdisk** utility was specifically designed for GPT partition


```bash
[root@localhost yujia] gdisk /dev/sda2
GPT fdisk (gdisk) version 1.0.7

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): ?
b	back up GPT data to a file
c	change a partition's name
d	delete a partition
i	show detailed information on a partition
l	list known partition types
n	add a new partition
o	create a new empty GUID partition table (GPT)
p	print the partition table
q	quit without saving changes
r	recovery and transformation options (experts only)
s	sort partitions
t	change a partition's type code
v	verify disk
w	write table to disk and exit
x	extra functionality (experts only)
?	print this menu

Command (? for help): 
```

## The parted Utility
**Changes are written immediately while part was still running**

At the **parted** CLI prompt, start **print** command to list the current 
partition table. If there is sufficient unallocated space is available,
then make a new (**mkpart**) partition. For more use **help** command to see
the details.

1. makelabel
2. msdos/mbr/gpt
3. makpart
4. primary
5. xfs
6. 1Mib
7. 501Mib

Using **rm** commadn to delete the target partition by nunber.
- Save any data you need from that partition
- Unmount the partition
- Make sure the partition isn't configured in /etc/fstab, so that Linux doesn't
try to mount it the next time you boot.
- After starting **parted**, run the **print** command to identify the partition
you want to delete, as well as its ID number.

1. (parted) mkdir
2. primary
3. linux-swap
4. 501Mib
5. 1001Mib
6. in the terminal run **mkswap /dev/sdb2**, then **swapon /dev/sdb2**

When a partiion is created in **parted**, you can change its type with the
**set** command. 

## Filesystem Formats
### Standard Filesystems
### Journaling Filesytem
Journaling filesystems have two main advantages. First, a filesystem check is
very fast. Second if a crash occurs, a journaling filesystem has a log (also 
known as a journal) that can be used to restore metadata for the file on 
relevant partition.

### Filesystem Format Commands
**mkfs**
**mkfs.ext3**
**mkfs.ext4**
**mkfs.xfs**
**mkfs.fat**
**mkswap**

```bash
mkfs -t xfs /dev/sdb5
mkfs.xfs /dev/sdb5
```

### Swap Volumes

```bash
# To see the swap space curently configuread, run cat/proc/swaps 
```
Swap volumes are formatted with the **mkswap** command. But that’s not enough. 
Swap volumes must also be activated with the **swapon** command. If the new 
swap volume is recognized, you’ll see it in both the /proc/swaps file and the 
output to the top command. Second, you’ll need to make sure to configure the 
new swap volume in the **/etc/fstab** file