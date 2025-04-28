# systemd and the Boot Process

Overview of the boot process
1. System powers on
2. The BIOS/UEFI firmware searches for a bootable device. Assuming this is a local hard drive, the GUID partition Table (GPT) or Master Boot Record (MBR) that device points to the GRUB 2 loader.
3. the booloader loads initramfs, gives contorl to the Linux kernel, which starts systemd, the first Linux process
4. The systemd process then initializes the system and activates appropriate system units. 

When Linux boots into a specific target, it starts a series of "units" (SSH daemon, NTP, etc)

## GRUB2
To pass a parameter to kernel through GRUB2, press E at the first GRUB2 menu.
Locate the line that starts with directive "linux"

Add systemd.unit=emergency.target to the end of the line and press CTRL-x, Linux starts in a mode of operation called emergency target, which runs a rescue shell.

If need direct access into a recovery shell, add systemd.unit=rescue.target to the end of the kernel command line.

If system won't boot into the resuce target, you need to append systemd.unit=empergency.target to kernel command line.

rescue targets mounts the filesystem in read-write mode. 
emergency targets doesn't mount any filesystems, except the root filesystem in read-only mode.

Both emergency and rescue target require the root passwrod to log in and get full root privileges.

If the root password has lost, need to add rd.break to the end of the kernel command line.

| Boot Option | Description |
| :----------- | :----------- |
| systemd.unit=emergency.target | Emergency shell; only the / filesystem is mounted in read-only mode. The root passwrod is required to log in.
| system.unit=rescue.target | Emergency shell; all filesystems are mounted. The root password is required to log in |
| rd.break | Emergency shell; used to recover the root password |

## Boot into a Different Target
Check the default target "systemctl get-default"
1. Reboot system
2. Access BIOS, load from the disk
3. Presset E to edit the current menu entry
4. Scroll down with the DOWN ARROW KEY to locate the line starting with **linux**. First, delete the kernel option **rhgb quiet**. Then, at the end of the line, type "systemd.unit-multi-user.target" or "systemd.unit=rescue.target" or "systemd.unit=emergency.target" or "rd.break"


## Recover the Root Password
1. Reboot system
2. Access BIOS, load from the disk
3. Presset E to edit the current menu entry
4. Scroll down with the DOWN ARROW KEY to locate the line starting with **linux**. First, delete the kernel option **rhgb quiet**. Then, at the end of the line, append "rd.break". The directive interrupts the boot sequence before the root filesystem is mounted. Confirm this by running "ls /sysroot"
5. Press CTRL-x
6. Remount the root /sysroot filesystem as read-write and change the root directory to /sysroot
```bash 
mount -o remount,rw /sysroot
chroot /sysroot
```
7. Change the root password
```bash 
passwd
```
8. Because SELinux is not running, the passwd commadn won't preserve the context the /etc/passwd file. To ensure /etc/passwd file is label with the correct SELinx context, instruct Linux to relabel all files at the next boot with the following command.
```bash 
touch /.autorelabel
```
9. Type "exit", then "exit" the system


## Modify the System Bootloader
Configuration of GRUB2 is at /etc/grub2.cfg (a symbolic link that points to /boot/grub2/grub.cfg if BIOS, /boot/efi/EFI/redhat/grub.cfg if UEFI)

To generate a new version of /etc/grub2.cfg
```bash 
# Generate a new version of /etc/grub2.cfg based on the /etc/default/grub configuration and on the scriipts in the /etc/grub.d/
grub2-mkconfig

# Once a modfication to /etc/default/grub, generate the new GRUB configuration file by running
# Assume we are dealing with BIOS system, adjust to /boot/efi/EFI/redhat/grub.cfig if UEFI
grub2-mkconfig -o /boot/grub2/grub.cfg
```

```bash 
[root@localhost ~] cat /etc/default/grub 
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume=/dev/mapper/rhel_vbox-swap rd.lvm.lv=rhel_vbox/root rd.lvm.lv=rhel_vbox/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
```

GRUB_TIMEOUT: Time in seconds before GRUB2 automatically boots the default OS
GRUB_DISTRIBUTOR: Can modify this entry to any string of your choice
GRUB_DEFAULT: The value "saved" instruct GRUB2 to look at the saved_entry variable in the file /boot/grub2/grubenv. This variable is updated with the name of the latest kernel every time that a new kernel is installed.

```bash 
# Update the saved_entry variable and instruct GRUB2 to boot a different default kernel
# Points to the first available menu
grub2-set-default 0

# Point to the second menu entry as default kernel and so on 
grub2-set-default 1
```

GRUB_CMDLINE_LINUX: Specifying the options to pass to the Linux kernel.
1. **rd.lvm.lv** tells the name of the logical volumes where the root filesystem and swap partition are located. 
2. The **crashkernel** option is used to reserve some memory for kdump, which is invoked to capture a kernel core dump if the system crashes. 
3. **rhgb quiet** directives enable the Red Hat graphical boot and hide the boot messages by default. If you want to enable verbose boot messages, remove the quiet option from this line

GRUB_DISABLE_SUBMENU: Set to "true" by default, it disables any submenu entries at boot.
GRUB_TERMINAL_OUTPUT: Tell GRUB2 to use a text console as the default output terminal.
GRUB_DISABLE_RECOVERY: Set to "true" by default, disables the generation of recovery menu entries.
GRUB_ENABLE_BLSCFG: Tells GRUB 2 to use a special file format, the Boot Loader Specification (BLS).

## How to Update GRUB2

```bash 
# Reinstall GRUB2 bootloader
grub2-install

# When configuration file is generated using grub2-mkconfig,no additional command are required
# The pointer from the MBR automatically reads the current version of /boot/grub2/grub.config
grub2-mkconfig
```

### GRUB2 Command Line
An error in grub.cfg can result in an unbootable system. If the GRUB2 configuration file is complete missing, the following is shown

```bash 
grub>
grub> help
```
Access a GRUB2 CMD can be done by pressing c key when menu is displayed.

linux/ and thre press Tab: review teh available fiels in the /boot directory
ls: find all detected hard drives the system 

By default /boot directory is mounted on a separate partition.

```bash
# h0: first hard drive 
# msdos1: the first partition in of the first hard drive (gpt if it is GPT partition)
# msdos2: the second partition in of the first hard drive (gpt if it is GPT partition)
grub> ls
(h0) (h0, msdo1) (hd0, msdo2)

# Display the content of grub.config
grub> cat (h0,msdos1)/grub2/grub.cfg

# Identify the partiion with the /boot directory
grub> search.file /grub2/grub.cfg

hd0,msdos1

# If the top level root filesystem is located on a partition, you may even confirm the contents of the /etc/fstab file
grub> cat (hd0, msdos2)/etc/fstab

# If the root filesystem resides on an LVM volume, load LVM module
grub> insmod lvm
grub> ls
(hd0) (hd0,msdos2) (hd0,msdos1) (lvm/rhel-root) (lvm/rhel-swap)
```

Boot RHEL9 from GRUB2

```bash 
# Insert module lvm (logical volume)
grub> insmod lvm

# List the all the partitions and logical volumes
grub> ls

# Identify the root partition. This may be named something like "lvm/rhel-root". You may need to poke around to find out
grub> cat (lvm/rhel-root)/etc/fstab

# Set the root variable to tthe device that you have identified as that containing tthe root fileysystem
grub> set root=(lvm/rhel-root)

# Enter te linux command, specifies the kernel and root directory partition
grub> linux (hd0, msdos1)/vmlinz-5.14.0-162.6.1.e19_1.x86_64 root=/dev/mapper/rhel-root

# Enter the initrd command, which specifies the inital RAM disk command and file location
grub> initrd (hd0,msdos1)/initramfs-5.14.0-162.6.1.e19_1.x86_64.img

# boot into the system
grub> boot
```

## Between GRUB 2 and Login
The loading for Linux depends on a temporary system, initial RAM disk. (/boot/initramfs)
Once the boot process is complete, control is given to systemd, known as the first process.

1. BIOS
2. GRUB
3. initramfs + kernel
4. load hardware driver
5. start the first process **systemd**
6. **systemd** activates all the system units for the initrd.target and mounts the root filesystem under sysroot
7. **systemd** restarts itself in the new root directory and activates all units for the default target.

## The First Process, Targets, and Units
Units are the basic buiding blocks of systemd. The most common are service units, which have a **.service** extension and activate a system service.

```bash 
systemctl list-units --type=service --all
```

A special type of unit is a **target unit** (suffix with .target), which isused to group together other system units and to transition the system into a different state.

```bash 
systemctl list-units --type=target --all
```

Targets are controlled by units, organized in unit files. Although the default target is defined in **/etc/systemd/system**, you can override the default during the boot process from the GRUB 2 menu.

```bash 
# list target's dependecies
systemctl list-dependencies graphical.target
systemctl list-dependencies rsyslog.service
```

The default target is specified as a symbolic link from the /etc/systemd/system/default.target file to either **multi-user.target** or **graphical.target**

```bash 
systemctl get-default

systemctl set-default multip-user.target
```

The **systemctl isolate** command in systemd is used to transition the system to a specific target unit while stopping all unrelated services. 

It’s a powerful tool for switching between system states (e.g., changing from graphical mode to rescue mode) without rebooting.

```bash 
sudo systemctl isolate multi-user.target  # Switch to CLI mode
sudo systemctl isolate rescue.target     # Enter rescue mode
```
```bash 
systemd-analyze blame
```

### Logging
By default, the journal log files are temporarily stored in RAM in a ring buffer in the /run/log/journal/. To get Linxu to write journal log files persistently on disk run the following
```bash 
mkdir /var/log/journal
journalctl --flush
```

### Control Groups
Control groups (cgroups), a feature of the Linux kernel to group process together and control or limit their resource usage (CPU, memory, etc). In systemd, cgroups are primaryily used to track processes and to ensure that all processes that belong to a service are terminated when a service is stopped.

```bash 
systemd-cgls
```
### systemd Units
systemd relies on teh configuration fiels under **/etc/systemd/system** and **/usr/lib/systemd/system** to start other process

The default configuration files are stored in **/usr/lib/systemd/system**.

Custom files stores in **/etc/systemd/system**

## Time Synchronization (NTP)
The time synchronization daemon included with RHEL9 is chronyd.

Timezone configuration file location: **/etc/localtime** A symbolic link that points to one of the time zone files in /usr/share/zoneinfo.

```bash 
ls -l /etc/localtime
lrwxrwxrwx 1 root root 34 Sep 14  2024 /etc/localtime -> /usr/share/zoneinfo/Asia/Singapore

timedatectl set-timezone America/Los_Angeles
```
The default **chronyd** config file locates /etc/chrony.conf