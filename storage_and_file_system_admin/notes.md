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

## Basic Linux Filesytems and Directories
|    Directory    | Description           |
| :-------------: | :-------------------- |
| /     | The root directory, the top-level directory in FHS. All other directories are subdirectories of root, which is always ;mounted on some volume |
| /bin  | Essential command-line utilites. Shoudl not be mounted separately; othewise, it could be difficult to get these utilities when user a rescue disk. On RHEL 9, it is a symbolic link to /usr/bin |
| /boot | Linux startup files, including the linux kernel. The default, 1 GB, is usually sufficient for a typical modular kernel and additional kernels. |
| /dev  | Hardware and software device drivers for everything from floppy drives to terminals. Do not mount this directory on a separate volume. |
| /etc  | Most basic configuration files. Do not mount this directory on a separate volume. |
| /home | Home directories for almost every user. |
| /lib  | Program libraries for the kernel and various command-line utilities. Do not mount this directory on a separate volume. On RHEL 9, this is a symbolic link to /usr/lib. |
| /lib64 | Same as /lib, but includes 64-bit libraries. On RHEL 9, this is a symbolic link to /usr/lib64 |
| /media | The mount point for removeable media, including DVDs and USB disk drives. |
| /misc  | The standard mount point for local directory mounted via the automounter. |
| /mnt   | A mount point for temporarily mounted filesystes. |
| /net   | The stanadrd mount point for network directories mounted via the automounter. |
| /opt   | Common location for third-path application files |
| /proc  | A virtual filesystem listing information for currently running kernel-related processes, including device assignments such as IRQ ports, I/O addresses, and DMA channels, as well as kernel-configuration settings such as IP forwarding. As a virtual filesystem, Linux automatically configures it as a separate filesystem in RAM. |
| /root | The home directory of the root user. Do not mount this directory on a separate volume. |
| /run  | A tmpfs filesystem for files that should not persist after a reboot. On RHEL 9, this filesystem replace /var/run, which is a symbolic link to /run |
| /sbin | System administration commands. Don't mount this directory separately. On RHEL 9, this is a symbolic link to /usr/sbin. |
| /smb  | The standard mount point for remote shared Microsoft network directories mounted via the automounter. |
| /srv  | Commonly used by various network servers non-Red Hat distributions. |
| /sys  | Similar to the /proc filesystem. Used to expose information about devices, drivers, and some kernel features. |
| /tmp | Temporary files. By default, RHEL deletes all fiels in this directory periodically. |
| /usr | Programs and read-only data. Includes many system administration commands, utilities, and libraries |
| /var | Variable data, including log files and printer spools. |

## Logical Volume Manager (LVM)
Logical volume manager (LVM) creates an abstractionlayer between physical devices, such as diska and partitions, and volumes that are formatted with a filesystem. 

On LVM, volume groups are like storage pools, and they aggregate together that capacity of multiple storage devices. Logical volumes reside on volume groups and can span multiple physical disks. 

1. Use **fdisk**, **gdisk**, and **parted** to create partitions and configure to the LVM partition type.
2. Set those partitions or disk devices as **physical volumes (PV)**
3. Create **volume groups (VG)** from one or more physical volumes. Volume groups organize the physical storage in a collection of manageable disk chunks known as **physical extents (PEs)**
4. Use command, those PEs can be organized into **logical volumes (LVs)**
5. Logcial volumes ar emade of **logcial extents (LE) **, which map to the underlying PEs. Then format and mount the LVs. 

| Volume Type   | Description |
| ------------- | ----------- |
| Physical volume (PV) | A PV is a partition or a disk drive initialized to be used by LVM |
| Physical extent (PE) | A PE is a small uniform segment of disk space. PVs are split into PEs. |
| Volume group (VG)    | A VG is a storage oool, made of one or more PVs. |
| Logical extent (LE)  | Every PE is associated with an LE, and these PEs can be combined into a logcial volume |
| Logical volume (LV) | An LV is a part of a VG and is made of LEs. An LV can be formmated with a filesystem and the mounted on the directory of your choice. |

Usual steps
1. Create a new PV, pvcreate
2. Assgin the space from one or more PVs to a VG, vgcreate
3. Allocate the space from some part of available VGs to an LV, lvcreate
4. To add space to an existing logical volue, lvextend.
5. If no spare space on VG, add more to it, vgextend.

```bash
# Just one PV
pvcreate /dev/sda1

# More than one PVs to be configured
pvcreate /dev/sda1 /dev/sda2 /dev/sdb1 /dev/sdb2

vgcreate volumegroup /dev/sda1 /dev/sda2

vgextend volumegroup /dev/sdb1 /dev/sdb2

# Create a logical volume, 
# This creates a device named /dev/volumegroup/logicalvolume
# You can format this device as if ti were a regular isk partition
lvcreate -l number_of_PEs volumegroup -n logicalvolume

# Use vgdisplay command to display the size of the PEs. or specify the size with
# -s option of the vgcreate commadn when you initalize the VG. 
# Also you cna use the -L option to set a size in Mib, Gib

# Create an LV name flex of 200Mib
lvcreate -L 200M volumegroup -n flex

# Remove a logcial volume
umount /dev/vg_01/lv_01
lvremove /dev/vg_01/lv_01

# Resize logical volumes
vgextend vg_00 /dev/sdd1
vgdisplay vg_00
lvextend -L 200M /dev/vg_00/lv_00
```

## Filesytem Management
To access file in a filesystem, that filesystem must be mounted on a mount point.

Linux normally automate this process using the **/etc/fstab** configuration file.

```bash
[root@localhost yujia]# cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Wed Apr 23 11:58:58 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel_vbox-root /                       xfs     defaults        0 0
UUID=c4026f8d-5b37-4426-966b-43f5260911ee /boot                   xfs     defaults        0 0
/dev/mapper/rhel_vbox-home /home                   xfs     defaults        0 0
/dev/mapper/rhel_vbox-swap none                    swap    defaults        0 0
```

In RHEL 9 the default is to use universally unique IDs (UUIDs) to mount non-LVM
filesystems. UUIDs can represent a partition, a logical volume, or an entire disk drive.

You can verify how partitions are actually mounted in the **/etc/mtab** file.

```bash
[root@localhost yujia]# cat /etc/mtab 
proc /proc proc rw,nosuid,nodev,noexec,relatime 0 0
sysfs /sys sysfs rw,seclabel,nosuid,nodev,noexec,relatime 0 0
devtmpfs /dev devtmpfs rw,seclabel,nosuid,size=4096k,nr_inodes=975312,mode=755,inode64 0 0
securityfs /sys/kernel/security securityfs rw,nosuid,nodev,noexec,relatime 0 0
tmpfs /dev/shm tmpfs rw,seclabel,nosuid,nodev,inode64 0 0
devpts /dev/pts devpts rw,seclabel,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000 0 0
tmpfs /run tmpfs rw,seclabel,nosuid,nodev,size=1573500k,nr_inodes=819200,mode=755,inode64 0 0
cgroup2 /sys/fs/cgroup cgroup2 rw,seclabel,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot 0 0
pstore /sys/fs/pstore pstore rw,seclabel,nosuid,nodev,noexec,relatime 0 0
bpf /sys/fs/bpf bpf rw,nosuid,nodev,noexec,relatime,mode=700 0 0
/dev/mapper/rhel_vbox-root / xfs rw,seclabel,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota 0 0
selinuxfs /sys/fs/selinux selinuxfs rw,nosuid,noexec,relatime 0 0
systemd-1 /proc/sys/fs/binfmt_misc autofs rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=24078 0 0
debugfs /sys/kernel/debug debugfs rw,seclabel,nosuid,nodev,noexec,relatime 0 0
mqueue /dev/mqueue mqueue rw,seclabel,nosuid,nodev,noexec,relatime 0 0
tracefs /sys/kernel/tracing tracefs rw,seclabel,nosuid,nodev,noexec,relatime 0 0
hugetlbfs /dev/hugepages hugetlbfs rw,seclabel,relatime,pagesize=2M 0 0
none /run/credentials/systemd-tmpfiles-setup-dev.service ramfs ro,seclabel,nosuid,nodev,noexec,relatime,mode=700 0 0
fusectl /sys/fs/fuse/connections fusectl rw,nosuid,nodev,noexec,relatime 0 0
configfs /sys/kernel/config configfs rw,nosuid,nodev,noexec,relatime 0 0
none /run/credentials/systemd-sysctl.service ramfs ro,seclabel,nosuid,nodev,noexec,relatime,mode=700 0 0
/dev/mapper/rhel_vbox-home /home xfs rw,seclabel,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota 0 0
/dev/sda1 /boot xfs rw,seclabel,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota 0 0
none /run/credentials/systemd-tmpfiles-setup.service ramfs ro,seclabel,nosuid,nodev,noexec,relatime,mode=700 0 0
tmpfs /run/user/1000 tmpfs rw,seclabel,nosuid,nodev,relatime,size=786748k,nr_inodes=196687,mode=700,uid=1000,gid=1000,inode64 0 0
gvfsd-fuse /run/user/1000/gvfs fuse.gvfsd-fuse rw,nosuid,nodev,relatime,user_id=1000,group_id=1000 0 0
portal /run/user/1000/doc fuse.portal rw,nosuid,nodev,relatime,user_id=1000,group_id=1000 0 0
```
## Universally Unique Identifiers in /etc/fstab
Run **blkid** command to identify the UUID for available volumes. 

```bash
[root@localhost yujia]# blkid /dev/sda2
/dev/sda2: UUID="QVaXWY-Yp7e-Ddcy-Nuoq-eMZT-B0Lc-Vb0W20" TYPE="LVM2_member" PARTUUID="003423eb-02"
```

### The mount Command
**mount** command can be used to attach local and network partitions to specified directories.

```bash
# To mount all filesystem as currently configured in the /etc/fstab
mount -a
```

```bash
# Not sure about a possible change to the /etc/fstab file. 
# It is possible to test it out with mount command. 
# The following command remounts the volume assicate with the /boot directory
# in read-only mode
mount -o remount,ro /boot

mount -o loop rhel-baseos-9.1-x86_64-dvd.iso /mnt
```
In most cases, it's sufficient to set up a new volume in /etc/fstab with the 
associated device file, such as /dev/sda6 partition, a UUID, or an LVM device such as /dev/mapper/NewVol-NewLV or /dev/NewVol/NewLV. 

Configurate a CD drive that can be mounted by regular users, add the following to the **/etc/fstab**
Some common mount options are shown in the following /etc/fstab entry.
Setting up a mount in read-only mode, does not try to mount it automatically during boot process, and supports access by regular users.
`
```bash
/dev/sr0 /cdrom auto ro,noauto,users 0 0
```

Network Filesystems
```bash
mount -f nfs server1.example.com:/pub /share
```

```bash
server1:/pub /share nfs rsize=65536,wsize=65536,hard 0 0 
```
**rsize** and **wsize** variables determine the maimum size (in bytes) of the data to be read and wriiten in each request. (Normally this should not be required, client server will negotiate on their own). The **hard** directive specifies that the client will retry failed NFS requests indefinitely. The **soft** option wil lcause the client to fail after a predefined number of retransmission, but at the cost of risking the integrity of the data.


## The Automounter
The automount daemon, aslo known as the automounter or **autofs**

## Mounting via the Automounter
In REHEL, the relevant configuration files are auto.master, automisc, auto.net, and auto.smb, all in the /etc directory. 

**If you use the automounter, keep the /misc and /net directories free, Red Hat configures automounts on these directories by default, and they won't work if local files or directories are stored there.**

