Part 1: Linux File System Hierarchy

Core Directories

/ (root) - The starting point of everything
in this directory these all directories/files contains and this "/" is the base directory of the linux
ls -l /
total 76
lrwxrwxrwx   1 root root     7 Apr 22  2024 bin -> usr/bin
drwxr-xr-x   2 root root  4096 Feb 26  2024 bin.usr-is-merged
drwxr-xr-x   5 root root  4096 Mar 13 22:12 boot
drwxr-xr-x  16 root root  3300 Mar 31 18:58 dev
drwxr-xr-x 108 root root  4096 Mar 31 18:58 etc
drwxr-xr-x   3 root root  4096 Mar 31 18:58 home
lrwxrwxrwx   1 root root     7 Apr 22  2024 lib -> usr/lib
drwxr-xr-x   2 root root  4096 Apr  8  2024 lib.usr-is-merged
lrwxrwxrwx   1 root root     9 Apr 22  2024 lib64 -> usr/lib64
drwx------   2 root root 16384 Mar 13 22:08 lost+found
drwxr-xr-x   2 root root  4096 Mar 13 22:05 media
drwxr-xr-x   2 root root  4096 Mar 13 22:05 mnt
drwxr-xr-x   2 root root  4096 Mar 13 22:05 opt
dr-xr-xr-x 172 root root     0 Mar 31 18:58 proc
drwx------   4 root root  4096 Mar 31 18:58 root
drwxr-xr-x  28 root root   900 Mar 31 19:20 run
lrwxrwxrwx   1 root root     8 Apr 22  2024 sbin -> usr/sbin
drwxr-xr-x   2 root root  4096 Mar 31  2024 sbin.usr-is-merged
drwxr-xr-x   6 root root  4096 Mar 13 22:12 snap
drwxr-xr-x   2 root root  4096 Mar 13 22:05 srv
dr-xr-xr-x  13 root root     0 Mar 31 18:58 sys
drwxrwxrwt  13 root root  4096 Mar 31 19:20 tmp
drwxr-xr-x  12 root root  4096 Mar 13 22:05 usr
drwxr-xr-x  13 root root  4096 Mar 31 18:58 var

/home - User home directories
in this users home directories are located for now we only have one user "ubuntu"
ls -l /home
total 4
drwxr-x--- 4 ubuntu ubuntu 4096 Mar 31 19:17 ubuntu

/root - Root user's home directory
to view this directory we need to use sudo(superuserdo) and this directory has access for the root user itself and the home user can't access 
sudo ls -l /root
total 4
drwx------ 3 root root 4096 Mar 31 18:58 snap

/etc - Configuration files
In this there are many files (config files) and directories [so far i know this much only]

/var/log - Log files (very important for DevOps!)
In this path there are several log files including systemlogs etc...
this is really imp for devops but for now i don't have much idea

/tmp - Temporary files
so far in this directory there are temp files but don't know for now what are they

Additional Directories (Good to Know):

/bin - Essential command binaries
ls -l /bin
lrwxrwxrwx 1 root root 7 Apr 22  2024 /bin -> usr/bin

/usr/bin - User command binaries
in this there are user commands which we use!!

/opt - Optional/third-party applications
For now there are none but maybe in future we will install one for some good motive (who knows)

# Find the largest log file in /var/log
du -sh /var/log/* 2>/dev/null | sort -h | tail -5
48K     /var/log/dmesg
60K     /var/log/kern.log
132K    /var/log/cloud-init.log
140K    /var/log/syslog
17M     /var/log/journal

# Look at a config file in /etc
cat /etc/hostname
ip-172-31-29-59

# Check your home directory
ls -la ~
total 40
drwxr-x--- 4 ubuntu ubuntu 4096 Mar 31 19:35 .
drwxr-xr-x 3 root   root   4096 Mar 31 18:58 ..
-rw-r--r-- 1 ubuntu ubuntu  220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu 3771 Mar 31  2024 .bashrc
drwx------ 2 ubuntu ubuntu 4096 Mar 31 18:58 .cache
-rw------- 1 ubuntu ubuntu   20 Mar 31 19:17 .lesshst
-rw-r--r-- 1 ubuntu ubuntu  807 Mar 31  2024 .profile
drwx------ 2 ubuntu ubuntu 4096 Mar 31 18:58 .ssh
-rw-r--r-- 1 ubuntu ubuntu    0 Mar 31 19:35 .sudo_as_admin_successful
-rw------- 1 ubuntu ubuntu  916 Mar 31 19:02 .viminfo
-rw-rw-r-- 1 ubuntu ubuntu   21 Mar 31 19:15 notes.txt
