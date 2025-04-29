# Command Line

Redirection Operators
| Operator | Description |
| -------- | ----------- |
| < *file* | Redirect stdin from file |
| > *file* | Redirect stdout to file |
| >> *file* | Append stdout to file |
| 2> *file* | Redirect stderr to file |
| &> *file* | Redirect stdout and stderr to file |
| *command1* \| *commnad2* | Redirect the stdout of command1 as stdin to command2 |

the tide (~) is $PATH

```bash 
[yujia@localhost Desktop]$ ls -l
total 8
-rwxr-xr-x. 1 yujia yujia 28 Apr 29 09:47 auto.sh
-rw-r--r--. 1 yujia yujia 24 Apr 29 09:47 dns.txt
drwxr-xr-x. 2 yujia yujia  6 Apr 29 09:47 notes
# d: directory, rwx: owner read,write,execut, group read, execute and other execute
```

File creation
```bash 
touch *file*
```

Copy file
```bash 
# -a supports recusive changes and preserves all file attributes
cp -a /home/yujia/. /mnt/backup

cp file1 file2 file3 /mnt/backup
```

Make links between files
softlink: redirect
hardlink: names that share the same inode (must on the same filesystem)

```bash 
[yujia@localhost Desktop]$ ln orignal-hardlink-file.txt hardlink-test.txt 
[yujia@localhost Desktop]$ ln -s orignal-softlink-file.txt softlink-test.txt 
[yujia@localhost Desktop]$ ls -li
total 20
50338719 -rwxr-xr-x. 1 yujia yujia 28 Apr 29 09:47 auto.sh
50338720 -rw-r--r--. 1 yujia yujia 24 Apr 29 09:47 dns.txt
50338718 -rw-r--r--. 2 yujia yujia 34 Apr 29 10:14 hardlink-test.txt # hardlink
50338715 drwxr-xr-x. 2 yujia yujia  6 Apr 29 09:47 notes
50338718 -rw-r--r--. 2 yujia yujia 34 Apr 29 10:14 orignal-hardlink-file.txt # hardlink
50338721 -rw-r--r--. 1 yujia yujia 31 Apr 29 10:15 orignal-softlink-file.txt
50338722 lrwxrwxrwx. 1 yujia yujia 25 Apr 29 10:17 softlink-test.txt -> orignal-softlink-file.txt
```

Create/Delte subdirectory
```bash 
mkdir -p /sub1/sub2/sub3
rmdir -p /sub1/sub2/sub3
```
Alias
```bash 
[yujia@localhost Desktop]$ alias
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias xzegrep='xzegrep --color=auto'
alias xzfgrep='xzfgrep --color=auto'
alias xzgrep='xzgrep --color=auto'
alias zegrep='zegrep --color=auto'
alias zfgrep='zfgrep --color=auto'
alias zgrep='zgrep --color=auto'
```

Find
```bash 
find / -name named.conf
find / -name "*.conf"
```
## File Permissions

```bash 
[yujia@localhost Desktop]$ ln orignal-hardlink-file.txt hardlink-test.txt 
[yujia@localhost Desktop]$ ln -s orignal-softlink-file.txt softlink-test.txt 
[yujia@localhost Desktop]$ ls -li
total 20
50338719 -rwxr-xr-x. 1 yujia yujia 28 Apr 29 09:47 auto.sh
50338720 -rw-r--r--. 1 yujia yujia 24 Apr 29 09:47 dns.txt
50338718 -rw-r--r--. 2 yujia yujia 34 Apr 29 10:14 hardlink-test.txt # hardlink
50338715 drwxr-xr-x. 2 yujia yujia  6 Apr 29 09:47 notes
50338718 -rw-r--r--. 2 yujia yujia 34 Apr 29 10:14 orignal-hardlink-file.txt # hardlink
50338721 -rw-r--r--. 1 yujia yujia 31 Apr 29 10:15 orignal-softlink-file.txt
50338722 lrwxrwxrwx. 1 yujia yujia 25 Apr 29 10:17 softlink-test.txt -> orignal-softlink-file.txt
```

### Special Permission Bit

| Special Permission | On an Executable File | On a Directory |
| :----------------- | :-------------------- | :------------- |
| SUID | When the file is executed, the effective user ID of the process is that of the file | No effect |
| SGID | When the file is executed, the effective group ID of the process is that of the file | Give files created in the directory the same group ownership as that of the directory |
| Sticky bit | No effect | Files in the directory can be renamed or removed only by their owners |

The **s** in the execuate bit for the user owner of the file is the SUID bit, which means the file can be executed by other users with the authority of the file owner (root in this case)
```bash 
[yujia@localhost Desktop]$ ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 32648 Aug 10  2021 /usr/bin/passwd
```

The **s** in the executate bit for the group owner of the file is the SGID.
```bash 
[yujia@localhost Desktop]$ ls -l /usr/bin/locate
-rwx--s--x. 1 root slocate 41032 Aug 10  2021 /usr/bin/locate
```

The **t** in the execute bit for other users is the sticky bit.
```bash 
[yujia@localhost Desktop]$ ls -ld /tmp/
drwxrwxrwt. 18 root root 4096 Apr 29 10:48 /tmp/
```

## Commands to Change Permissions and Ownership
ugo: user, group, other
r = 4, w = 2, x = 1
ugo/rwx
permission 640: owner rw, group r, other none

SUID = 4, SGID = 2, sticky bit = 1

chmod
```bash 
chmod u+x script.sh
chmod go-w document.txt
chmod g=rw document.txt
chmod +x script.sh

chmod 4764 testfile
chmod g+s testscript
chmod o+t /test
```
chown
```bash 
chown new-owner filename
chown new-owner:new-group filename
```
chgrp
```bash 
chgrp new-group filename
```

## Special File Attributes
lsattr: list the current file attributes
chattr: change file attributes

| Attribute | Description |
| --------- | ----------- |
| append only (a) | Prevent deletion, but allows appending to a file |
| no dump (d) | Disallows backups of the configured file with the **dump** command |
| immutable (i) | Prevents deletion or any other kind of change to a file (read-only) |

```bash 
chattr -i /etc/fstab
```

## Manage Text Files
cat, less, more
head, tail
sort, grep
diff, wc
sed, awk

grep
```bash 
ls -l | grep '\.sh'

# -v reverses the matching, do not match a regexp
grep -v '^$' /etc/nsswitch.conf | grep -v '^#'
```

sed
```bash 
sed 's/Windows/Linux/' opsys > newopsys
sed 's/Windows/Linux/g' opsys > newopsys
```

awk
```bash 
awk -F : '/mike/ {print $4}' /etc/passwd
```