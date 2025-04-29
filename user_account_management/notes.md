# User Account Management

```bash 
[yujia@localhost Desktop]$ last
yujia    pts/1        192.168.1.5      Tue Apr 29 09:46   still logged in
yujia    tty2         tty2             Tue Apr 29 09:34   still logged in
yujia    seat0        login screen     Tue Apr 29 09:34   still logged in
reboot   system boot  5.14.0-503.38.1. Tue Apr 29 09:32   still running
yujia    pts/0        192.168.1.5      Mon Apr 28 20:25 - crash  (13:06)
yujia    tty2         tty2             Mon Apr 28 14:59 - crash  (18:32)
yujia    seat0        login screen     Mon Apr 28 14:59 - crash  (18:32)
reboot   system boot  5.14.0-503.38.1. Mon Apr 28 14:59   still running
yujia    tty2         tty2             Mon Apr 28 14:31 - down   (00:00)
```

```bash 
# id show the identity of the current user
id
uid=1000(yujia) gid=1000(yujia) groups=1000(yujia),10(wheel)context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# show the identity of given user
id yujia
uid=1000(yujia) gid=1000(yujia) groups=1000(yujia),10(wheel)

```

## The Shadow Passwrod Suite
Default settings for user accountss are stored in **/etc/login/defs**
Four configuration file os the shadow password suite:
1. /etc/passwd
2. /etc/group
3. /etc/shadow
4. /etc/gshadow

### /etc/passwd
/etc/passwd file contans basic info about every user.

```bash 
[yujia@localhost Desktop]$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:998:996:User for polkitd:/:/sbin/nologin
avahi:x:70:70:Avahi mDNS/DNS-SD Stack:/var/run/avahi-daemon:/sbin/nologin
rtkit:x:172:172:RealtimeKit:/:/sbin/nologin
pipewire:x:997:994:PipeWire System Daemon:/run/pipewire:/usr/sbin/nologin
libstoragemgmt:x:992:992:daemon account for libstoragemgmt:/:/usr/sbin/nologin
geoclue:x:991:990:User for geoclue:/var/lib/geoclue:/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
cockpit-wsinstance:x:990:989:User for cockpit-ws instances:/nonexisting:/sbin/nologin
flatpak:x:989:988:User for flatpak system helper:/:/sbin/nologin
colord:x:988:987:User for colord:/var/lib/colord:/sbin/nologin
setroubleshoot:x:987:986:SELinux troubleshoot server:/var/lib/setroubleshoot:/usr/sbin/nologin
clevis:x:986:985:Clevis Decryption Framework unprivileged user:/var/cache/clevis:/usr/sbin/nologin
gdm:x:42:42::/var/lib/gdm:/sbin/nologin
sssd:x:985:984:User for sssd:/:/sbin/nologin
gnome-initial-setup:x:984:983::/run/gnome-initial-setup/:/sbin/nologin
stapunpriv:x:159:159:systemtap unprivileged user:/var/lib/stapunpriv:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
dnsmasq:x:983:982:Dnsmasq DHCP and DNS server:/var/lib/dnsmasq:/usr/sbin/nologin
chrony:x:982:981:chrony system user:/var/lib/chrony:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
yujia:x:1000:1000:yujia:/home/yujia:/bin/bash
```

| Field       |   Example    |                  Description                            |
| :---------- | :----------- | :------------------------------------------------------ |
| Username    | yujia        | The user logs in with this name                         |
| Password    | x            | The password, should be an *x*, means actual encrypted password stored in /etc/shardow/. An asterisk means that the user will not able to log in |
| User ID     | 1000         | The unique numeric UID for that user. By default, RHEL creates starts regular UID at 1000 |
| Group ID    | 1000 | The primary GID associated with that user. By default RHEL creates a new group for every new user, and the number matches the UID, if the corresponding GID is available, some other Linux system assign all users to a default users group |
| User info   | yujia        | Information about the user |
| Home directory | /home/yujia | By default, RHEL places new home directory in /home/*username*/ |
| Login shell | /bin/bash | By default, RHEL assigns users to bash as their default shell |

### /etc/group
Every user is assigned to a *primary group*, which is the grop correponding to the GID defined in /etc/passwd.
By default, every user gets their own private group with the same name as their username. 
A user may also be a member of other gorup, as *supplementary group*.

The purpose of a primary group is to assign a group owner to a new file.
All files are owned by the UID and primary group GID of the user who created them.

wheel group gives the privilege to run administrative commands via sudo.

```bash 
[yujia@localhost Desktop]$ cat /etc/group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:yujia
....
yujia:x:1000:
```

| Field       |   Example    |                  Description                            |
| :---------- | :----------- | :------------------------------------------------------ |
| Group name  |  wheel        | Each user gets his own primary group, with the same name as his username. You can also create supplementary group  |
| Group password | x | The password, should be *x*, which points to /etc/gshadow for the actual password |
| Group ID | 2000 | The numeric GID associated with the group. If you want to create a supplementary group, you should assign a GID outside the standard range; otherwise, Red Hat GIDs and UIDs would probably get out of sequence. |
| Group members | yujia | Lists the usernames that are members of the group. |


### /etc/shadow

/etc/shadow a supplement to /etc/passwd.

| Column | Field       |                  Description                            |
| :----- | :---------- | :------------------------------------------------------ |
| 1      | Username    | Username                                                |
| 2      | Password    | Encrypted password; requires an x in the second column of /etc/passwd |
| 3      | Date of last password change | In number of days after January 1, 1970 |
| 4      | Minimum password age | Minimum number days that a user must keep a pssword before they can change it |
| 5      | Maximum password age | Maximum number of days after which a pssword must be changed |
| 6      | Password warning period | Number of days before password expiration when a warning is given |
| 7      | Password inactivity period | Number of days after password expiration during which the password is still accepted, but the user is prompted to change their password |
| 8      | Account expiration date | Number of days since January 1, 1970 after which the password will be expire
| 9      | Reserved | Reserved for future use |

Note that some user have * or !! in their password field. That means the users will not able to use a password to login.

```bash 
[root@localhost ~]# cat /etc/shadow
root:$6$rounds=100000$6XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX:20206:0:99999:7:::
bin:*:19760:0:99999:7:::
daemon:*:19760:0:99999:7:::
adm:*:19760:0:99999:7:::
lp:*:19760:0:99999:7:::
sync:*:19760:0:99999:7:::
shutdown:*:19760:0:99999:7:::
halt:*:19760:0:99999:7:::
mail:*:19760:0:99999:7:::
operator:*:19760:0:99999:7:::
games:*:19760:0:99999:7:::
ftp:*:19760:0:99999:7:::
nobody:*:19760:0:99999:7:::
apache:!!:20201::::::
systemd-coredump:!!:20201::::::
dbus:!!:20201::::::
polkitd:!!:20201::::::
avahi:!!:20201::::::
rtkit:!!:20201::::::
pipewire:!!:20201::::::
libstoragemgmt:!*:20201::::::
geoclue:!!:20201::::::
tss:!!:20201::::::
cockpit-wsinstance:!!:20201::::::
flatpak:!!:20201::::::
colord:!!:20201::::::
setroubleshoot:!!:20201::::::
clevis:!!:20201::::::
gdm:!!:20201::::::
sssd:!!:20201::::::
gnome-initial-setup:!!:20201::::::
stapunpriv:!*:20201::::::
sshd:!!:20201::::::
dnsmasq:!!:20201::::::
chrony:!!:20201::::::
tcpdump:!!:20201::::::
yujia:$6XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX::0:99999:7:::
```

### /etc/gshadow
/etc/gshadow includes the group administrators, which can add other group members using **gpasswd** command.

```bash 
[root@localhost ~] cat /etc/gshadow
root:::
bin:::
daemon:::
sys:::
adm:::
tty:::
disk:::
lp:::
mem:::
kmem:::
wheel:::yujia
cdrom:::
mail:::
...
stapusr:!*::
stapsys:!*::
stapdev:!*::
stapunpriv:!*::stapunpriv
sshd:!::
dnsmasq:!::
slocate:!::
chrony:!::
tcpdump:!::
yujia:!::
test:!:yujia:yujia,yujiadev
```
| Field | Example | Description |
| :---- | :------ | :---------- |
| Group name | reviewers | The group name |
| Password | ! | Most groups have a !, which means no password |
| Administrator | yujia | A list of user who can change the members of password of the group using the gpasswd command |
| Group members | yujia,yujiadev | A list of usernames that are members of the group |

### /etc/login.defs 
/etc/login.defs provides the baseline for a number of parameters in the shadow password suite.

| Congfiguration Parameter | Descripttion |
| :----------------------- | :----------- |
| PASS_MAX_DAYS            | After thi number of days, the password must be changed |
| PASS_MIN_DAYS            | Password must be kept for at least this number of days |
| PASS_WARN_AGE            | Users are warned this number of days before PASS_MAX_DAYS |

## Command-Line Tools
As the root user, you can add user directly by editing the /etc/passwd.
Both **vipw** and **vigr** can edit the concerning files.

### Add Users at the Command Line

```bash 
# Add user michael in /etc/passwd
# Create the /home/michael home directory
# Add the standard files from the /etc/skel directory
# Assign the default shell /bin/bash
useradd michael 
```
useradd Command Options
| Option | Descritption |
| :----- | :----------- |
| -u UID | Override the default assigned UID |
| -g GID | Override the default GID |
| -c info | Enter the comment of you choice about the user |
| -d dir  | Override the default home directory for the user, /home/username |
| -e YYYY-MM-DD | Set an expiration date for the user account |
| -f num | Number of days after passwrod expiration that the account will be permanently disabled |
| -G group1,group2 | Make a user a member of the supplementary groups *group1* and *group2* based on their current names as defined in the /etc/group file. |
| -s shell | Overrides the default shell for the user, /bin/bash |

Once user is created, add a password is needed

```bash 
passwd <username>
```

### Add or Delete a Group at the Command Line
```bash 
# Without -g otpin, the command takes the next available GID. 
groupadd -g 60001 progject

# Delete a group
groupdel project
```

### Delete a User
```bash 
# Just delete user
userdel <username>

# Delete the user's home directory as well
userdel -r <username>
```

## Modify an Account

### usermod
**usermod** modifies various settings in /etc/passwd, allows you change the settings in /etc/shadow, or add a user to a supplementary group in /etc/group.

```bash 
# Set the account associated with user test1 to expire on June 8, 2023
usermod -e 2023-06-08 test1
```

| Option | Description |
| :----- | :---------- |
| -a -G *group1* | Appends to existing gorup memberships; multiple groups may be specified, split with comma, with no spaces |
| -l *newlogin* | Changes the username to *newlogin*, without changing the home directory |
| -L | Locks a user's password |
| -U | Unlocks a user's password |

```bash 
# Make user test1 a member of a grou named accouting
usermod -G accouting test1
```

### groupmod
```bash 
# Change the GID number of the gorup name project (to 60002)
groupmod -g 60002 project

# Check the group name project to secret
groupmod -n secret project
```

### chage
**chage** used to to manage aging information for a password, by modifying the corresponding fields in the /etc/shadow file.

| Option | Description |
| :----- | :---------- |
| -d YYYY-MM-DD | Set the last change date for a password; the output is shown in /etc/shadow as the number of days after January 1, 1970 |
| -E YYYY-MM-DD | Assigns the expiration date for an account; the output is shown in /etc/shadow as the number of days after January 1, 1970 |
| -I num | Locks an account num days after a password has expired; can be set to -1 to make the account permanent. |
| -l | List all aging information |
| -m num | Set a minimum number of days that a user must keep a password |
| -M num | Set a maximum number of days that a user is allowed to keep a pasword; can set to -1 to remove that limit |
| -W num | Specifies when a user is warned to change their passowrd, in number of days before the password expiration date |

```bash 
[root@localhost ~] chage -l yujia
Last password change				: never
Password expires					: never
Password inactive					: never
Account expires						: never
Minimum number of days between password change		: 0
Maximum number of days between password change		: 99999
Number of days of warning before password expires	: 7
```

## Administrative Control
**su** allows a user to become another user
/etc/etc/sudoers

### The Ability to Login
/etc/security/access.conf regulates access by all users.
By default, in RHEL 9 the file /etc/security/access.conf does not have any effect. To enable access.con, the following line to **/etc/pam.d/password-auth** and **/etc/pam.d/system-auth**:

account required pam_access.so

### The su Command
**su -l <username>** login directly to that account

Assume administrative privileges for one command
```bash 
su -c '/sbin/fdisk /dev/sda
```

### Limit Access to su
In /etc/group
wheel:x:10

You can add select users to the end fo this line directly with usermod -G wheel <useranme>

### The sg Command
**sg** allows executing a command as another group.

```bash 
sg <groupname> -c 'cp important.doc /home/project'
```

### Superuser Access with the sudo Command
Regular users who are authorized in /etc/sudoers can access administrative commands with their own password

To access /etc/sudoers run the **visudo** command

In the /etc/sudoers


```txt
# Allows the root user to run any command as any user
root    ALL=(ALL) ALL

# If you want to allow user boris full administrative access, add the following directive to /etc/sudoers
boris   All=(ALL) ALL

# Alternatively, you can allow special users administrative acccess without a password
%wheel  ALL=(ALL)   NOPASSWD: ALL

# Allow users who are members of the %user group to shut down the local system, activate the following directive:

%users localhost=/sbin/shutdown -h now
```

## User and Shell Configuration
All system-wide shell configuration are kept in /etc 
bashrc
profile
/etc/profile.d

Each user gets a copy of all files from /etc/skel directory

## Special Groups
In RHEL, each user gets their own special private group by default. 

### Shared Directories

```bash 
# Create the shared directory
mkdir /home/accshared

# Create a group for the accountants. Call it accgrp. Give it a group ID that doesn't interfere with exiting group of user IDs.
# One way to donw it add a line such as the following to the /etc/group file
accgrp:x:70000:robertc,alanm,victorb,roberta,alano,charliew

# Set up ownership fo rthe new shared directory. The following commands prevent any specific user from taking control of the directory and assign group ownership to accgrp:

chown nobody:accgrp /home/accshared
chmod 2777 /home/accshared

# This is made possible by the 2770 permissions assigned to the /home/accshared directory.
# Let’s break that down into its component parts. The first digit (2) is the set group ID bit,
# also known as the SGID bit. When an SGID bit is set on a directory, any files created in
# that directory automatically have their group ownership set to be the same as the group
# owner of the directory. In addition, group ownership of files copied from other directories is
# reassigned (in this case, to the group named accgrp).

chmod g+s /home/accshared
```