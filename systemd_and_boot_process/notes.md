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