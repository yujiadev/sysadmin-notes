# Software Management (RedHat)

## Location of the RPM database: /var/lib/rpm
```bash 
# Verify whether any file from that RPM package has changed
$ rpm -v <some-package>
```

## Repository
BaseOS: The main repo which includes the packages associated with the core RHEL9 system, along with updates.

AppStream: A large group of open-source applications, which includes databases and run-time language

Each Repo includs a database of packages in a repodata/subdirectory. That database includes information on eac package and allows installation requests to include all dependencies.

Access to Red Hat repos is enabled in the product-id.conf and subscription-manager.conf, in the /etc/dnf/plugins.

```bash 
# Try to install the rpm package (may not work due to dependencies incomplete)
$ rpm -i policycoreutils-gui-3.4.4-4.e19.noarch.rpm
```
To get the rpm package, (assume that RHEL 9 DVD is on the hand), mount /dev/cdrom /media. Next under AppStream/Packages/ look for policycoreutils-gui package. Alternatively, download the package directly from the Red Hat Customer Portal.

```bash 
# Try to install the rpm package (may not work due to dependencies incomplete)
# Try to install the rpm package if it isn't already installed
$ rpm -i policycoreutils-gui-3.4.4-4.e19.noarch.rpm

# -vh options, verbose and hash, can help monitor th progress of the installation
$ rpm -ivh policycoreutils-gui-3.4.4-4.e19.noarch.rpm

# Upgrades any existing package or installs it if an earlier version isn't already installed.
$ rpm -U policycoreutils-gui-3.4.4-4.e19.noarch.rpm

# Upgrades only existing packages. It doesn not install a package if it wan't previously installed
$ rpm -F policycoreutils-gui-3.4.4-4.e19.noarch.rpm
```
In a well made RPM package, if upgrade cause a configuration change, the old configurations will be bakcup. The backup file has a "rpmsave" extension in the same directory

```bash 
# Try to install the rpm package (may not work due to dependencies incomplete)
# Try to install the rpm package if it isn't already installed
$ rpm -U policycoreutils-gui-3.4.4-4.e19.noarch.rpm
warning: /etc/someconfig.conf saved as /etc/someconfig.conf.rpmsave
```

### Uninstall an RPM package
```bash 
# Uninstall an RPM package
# First check dependency check to make sure no other packages need the package
$ rpm -e policycoreutils-gui-3.4.4-4.e19.noarch.rpm
```

### Install RPMS from Remote Systems
```bash 
# Via remote FTP server w/o password
$ rpm -ivh ftp://ftp.rpmdownloads.com/pub/foo.rpm

# Via remote FTP server w/ account
$ rpm -ivh ftp://yujiadev:mypassword@hostname:port/path/to/remote/package.rpm
```

### RPM Installation Security
During the installation of RHEL, the Red Hat public GPG keys are imported. 
```bash 
# Mannally import Red Hat's public GPG
rpm --import /etc/pki/rpm-gpg/PRM-GPG-KEY-redhat-release

# The GPG key is also included in the RPM database, which can be verified with the following
rpm -qa gpg-pubkey
```

### Update the Kernel
Don't rpm -U newkernel !!!! It will replace the existing kernel!!! The best option for upgrading to a new kernel is to install it.
```bash 
rpm -ivh newkernel
```
Files in /boot
|   File     |                  Description                   |
| :--------- | :--------------------------------------------- |
| config-*   | Kernel configuration settings; a text file     |
| efi/       | Files required at boot by the UEFI firmware    |
| grub2/     | Directory with GRUB configuration files        |
| initramf-* | The initial RAM disk filesystem, a root filesystem called during the boot process to help load other kernel components, such as block device modules |
| loader/    | Boot configruation options for each kernel that is installed on the system |
| System.map-* | Map of system names for variables and functions, with their location in memory |
| vmlinuz-*  | The actual Linux kernel |

The installation of a new kernel adds options to boot the new kernel in the GRUB configuration file (/boot/grub2/grub.cfg), without erasing existing options.

By default, the system will boot with the newly installed kernel. Therefore, if that kernel does not work, you can restart the system, access the GRUB menu, and then boot from the older, previously work kernel.

### Package Queries
```bash 
# Lists al installed packages
rpm -qa

# Identifies the package assoicated with /path/to/file
rpm -qf /path/to/file

# Lists only the configuration files from packagename
rpm -qc packagename

# Lists only docs files ffrom the packagename
rpm -qd packagename

# Display basic information for packagename
rpm -qi packagename

# List all files from packagename
rpm -ql packagename

# List all dependencies
rpm -qR packagename

# Display change information for packagename
rpm -q --changelog packagename

# Query an PRM package file rather than the local RPM database. with -p switch that specifiy the path or URL
rpm -qlp xxxx.noarch.rpm
```

### Package Signatures
```bash 
# -K is equivalent to --checksig
rpm ---checksig -v pkg-1.2.3.-4.noarch.rpm
```

### File Verification
```bash 
# Verify all files (take a long time)
rpm --verify -a

# Verify all files in a package agianist a downloaded RPM
rpm --verify -p vsftpd-3.0.3-49.e19.x86_64.rpm

# Verify a file associated with a partiuclar package
rpm -verify --file /bin/ls
```

If the integrity of the files or packages is verified, there iwll be no output. Any output means that a file or package is different from the original.

| Failure Code  | Meaning |
| :-----------: | :------- |
| S             | File size |
| M             | Mode (permissions) |
| 5             | MD5 Digest |
| D             | Device file |
| L             | Symbolic link |
| U             | User ownership |
| G             | Group ownership |
| T             | File modification time |
| P             | Capabilities |