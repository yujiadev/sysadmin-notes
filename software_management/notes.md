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

## Dependencies and the dnf Command
Extra packages, things like transmission
```bash 
$ sudo dnf install epel-release
```

### Red Hat Subscription Management
```bash 
# Use --force command option to re-register the machine
subscription-manager register 

# --auto option finds the most appropriate subscription
subscription-manager attach --auto

# List the available subscriptions. Please take note of the pool IDs.
subscription-manager list --available
subscription-manager attach --pool=<your-pool-id-goes-herer>

# Review the current settings
subscription-manager list

# Show all available repos
subscription-manager repos

# Show the enabled repos
subscription-manager repos --list-enabled

# Enable additional repo
subscription-manager repos --enable=<repo-id-goes-here>

# Confirm the subscribed BaseOS and AppStream
dnf repolist
```

## dnf Configuration
dnf configurations are under /etc/dnf/dnf.conf, /etc/dnf, and /etc/yum.repos.d

```bash 
# List the current dnf configurations
dnf config-manager --dump
```

### dnf.conf
```bash 
[root@localhost Packages] cat /etc/dnf/dnf.conf 
[main]                                # Indicate all the following directives applies to globally
gpgcheck=1                            # Check the GPG
installonly_limit=3                   # number of the package listed in the installonlypkgs option can be installed at the same time
clean_requirements_on_remove=True     # Go through the each package dependency when removing packages
best=True
skip_if_unavailable=False             # Try to install the highest version available 
```

### /etc/dnf/plugins
The default files in the /etc/dnf/plugins configure a connection between dnf and Red Hat Portal or a local Satellite server.

### /etc/yum.repos.d
Configuration files in the /etc/yum.repos.d are designed to connect systems to actual dnf repos. Doc available on "man dnf.conf" command

### Create Your Own /etc/yum.repos.d Configuration File
Create a file name <your-repos-name>.repo in /etc/yum.repos.d

```bash 
dnf config-manager --add-repo=https://myrepos.example.com
```
In the newly create repo file, configurute the name directive for the repo and based url

```bash 
[test]
name=myRHtest
baseurl=http://192.168.122.1/inst
gpgcheck=0     # it is easier to set to 0 in the test setting but not in production setting!!!
```

Steps of Creating a new repos config
```bash 
# Assuming that there is RHEL DVD are mounted at the system
mount /dev/cdrom /media

cd /etc/yum.repos.d

# Add the following to the vim (adjust the baseurl accordingly)
#[test]
#name=BaseOS
#baseurl=file:///run/media/yujia/RHEL-9-5-0-BaseOS-x86_64/BaseOS/
#enabled=1
#gpgkey=0
vim rhel9.repo

dnf clean all
dnf update
```

For deletion, just remove the .repo file under /etc/yum.repos.d and run "dnf clean all" and "dnf update"


## dnf Commands

```bash 
dnf list
dnf list | grep package-name
dnf repolist all
dnf info samba
dnf install package-name

# Keep the packages on the system up to date
dnf update package-name

# Remove a package
dnf remove package-name

# List available updates
# dnf check-update
dnf list updates

# Not sure what to install?
dnf provides "*/evince*"

# Search for all instances of file with the .repos extension
# It lists all instances of the packages with files that end with .repo extension, with the associate RPM package.
# Wildcare is require because "provides" option requires the full path to the file.
# dnf provides "/etc/systemd/*"
dnf provides "*/*.repo"

# Download the package
dnf download cups
```
View all directive with dnf config-manager
```bash 
#  dnf config-manager --dump | less
dnf config-manager --dump
```

## Package Groups and dnf
dnf command can install and remove package in groups.

```bash 
# Display the available package groups from the configured repos
[root@localhost yum.repos.d] dnf group list
Updating Subscription Management repositories.
Last metadata expiration check: 0:17:02 ago on Sun 27 Apr 2021 10:27:37 PM CST.
Available Environment Groups:
   Server
   Minimal Install
   Workstation
   Virtualization Host
   Custom Operating System
Installed Environment Groups:
   Server with GUI
Installed Groups:
   Container Management
   Headless Management
Available Groups:
   RPM Development Tools
   .NET Development
   Console Internet Tools
   Legacy UNIX Compatibility
   Network Servers
   Smart Card Support
   Scientific Support
   System Tools
   Development Tools
   Graphical Administration Tools
   Security Tools
```

```bash 
# Display the available package groups from the configured repos
dnf group info "Server"

dnf group install "Printing Client"

# Exclude paps and gutenprint-cups packages
dnf group install "Printing Client" -x paps -x gutentprint-cups

# Remove package group
dnf group remove "Printing Client"
```

## Module Stream(*)
```bash 

# List all modules
dnf module list

# Get the list of RPM packages that are installed by the PostgreSQL module
dnf module info postgresql

dnf module info nodejs:18

# --profile shows thte RPM packages that will be installed on each profile
dnf module info --profile nodejs:18
```

### Install and Removing Module Streams

d: enable, e: disable, i: installed

To insttall a module stream, the corresponding module stream must be enabled first.

```bash 
dnf module enable nodejs:18
dnf module install nodejs

# dnf module install automatically enables a moduels, you can replace the previous commands with the following
dnf module install nodejs:18

# Remvoing a module
dnf moduel remove nodejs:18

# "Reset" the stream, disable the stream and resets the configuration to the default settings:
dnf module reset nodejs
```