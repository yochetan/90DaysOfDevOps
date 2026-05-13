Task 1: Self-Assessment Checklist
Go through the checklist below. For each item, mark yourself honestly:

* Can do confidently
* Need to revisit
* Haven't done yet

Linux

* Navigate the file system, create/move/delete files and directories

        mkdir folder 
        touch file
        cp source dest
        mv source dest
        rm file
     
 * Manage processes — list, kill, background/foreground


 
 Work with systemd — start, stop, enable, check status of services
 Read and edit text files using vi/vim or nano
 Troubleshoot CPU, memory, and disk issues using top, free, df, du
 Explain the Linux file system hierarchy (/, /etc, /var, /home, /tmp, etc.)
 Create users and groups, manage passwords
 Set file permissions using chmod (numeric and symbolic)
 Change file ownership with chown and chgrp
 Create and manage LVM volumes
 Check network connectivity — ping, curl, netstat, ss, dig, nslookup
 Explain DNS resolution, IP addressing, subnets, and common ports

1. Navigate the File System

Basic Commands
        
        | Command     | Purpose                      |
        |-------------|------------------------------|
        | pwd         | Show current directory       |
        | ls          | List files                   |
        | cd dir      | Change directory             |
        | mkdir dir   | Create directory             |
        | rmdir dir   | Remove empty directory       |
        | touch file  | Create file                  |
        | cp src dest | Copy files                   |
        | mv src dest | Move/rename files            |
        | rm file     | Delete file                  |
        | rm -r dir   | Delete directory recursively |

Example :

        mkdir project
        cd project
        touch app.txt
        mv app.txt demo.txt
        rm demo.txt

2. Manage Processes

List Processes

        ps aux
        top
        htop

Kill Process

        kill PID
        kill -9 PID

Background Process

        sleep 100 &

Foreground Process

        fg

3. Work with systemd

Start Service

        sudo systemctl start nginx

Stop Service

        sudo systemctl stop nginx

Restart Service

        sudo systemctl restart nginx

Enable at Boot

        sudo systemctl enable nginx

Check Status

        systemctl status nginx

4. Edit Text Files

Vim

        vim file.txt

Vim Basics

        | Key | Action           |
        |-----|------------------|
        | i   | Insert mode      |
        | Esc | Exit insert mode |
        | :w  | Save             |
        | :q  | Quit             |
        | :wq | Save & quit      |
        | :q! | Force quit       |


5. Troubleshoot CPU, Memory, Disk

CPU Usage

        top
        htop

Memory Usage

        free -h

Disk Space

        df -h

Directory Size

        du -sh *

6. Linux File System Hierarchy

        | Directory | Purpose                |
        |-----------|------------------------|
        | /         | Root directory         |
        | /home     | User home folders      |
        | /etc      | Configuration files    |
        | /var      | Logs and variable data |
        | /tmp      | Temporary files        |
        | /bin      | Essential commands     |
        | /usr      | User programs          |
        | /opt      | Optional software      |
        | /dev      | Device files           |
        | /proc     | Process/kernel info    |

Simple Layout

        / 
        ├── home
        ├── etc
        ├── var
        ├── tmp
        ├── usr
        └── dev

7. Users and Groups

Create User

        sudo useradd john
        
Set Password

        sudo passwd john

Create Group
        
        sudo groupadd developers

Add User to Group

        sudo usermod -aG developers john

8. File Permissions

Numeric Permissions

| Number | Permission |
|--------|------------|
| 7      | rwx        |
| 6      | rw-        |
| 5      | r-x        |
| 4      | r--        |

chmod Numeric

        chmod 755 script.sh

Meaning:

        Owner = rwx
        Group = r-x
        Others = r-x

Symbolic chmod

        chmod u+x script.sh

9. Ownership

Change Owner

        sudo chown john file.txt

Change Group

        sudo chgrp developers file.txt

Change Owner + Group

        sudo chown john:developers file.txt

10. LVM (Logical Volume Manager)

Steps

Create Physical Volume

        pvcreate /dev/sdb

Create Volume Group

        vgcreate vgdata /dev/sdb

Create Logical Volume

        lvcreate -L 5G -n lvdata vgdata

Format

        mkfs.ext4 /dev/vgdata/lvdata

Mount

        mount /dev/vgdata/lvdata /mnt

11. Network Connectivity

Ping

        ping google.com

Curl

        curl https://example.com

Check Ports

        netstat -tulnp
        ss -tulnp

DNS Lookup
        
        dig google.com
        nslookup google.com

12. DNS, IP, Subnets, Ports

DNS Resolution

DNS converts:

        google.com → IP Address

IP Address

Example:

        192.168.1.10

* Identifies a device on a network

Subnet

Subnet divides networks into smaller parts.

Example:

        192.168.1.0/24

* /24 means:
* 24 bits for network
* 8 bits for hosts
        
        | Common Ports |            |
        |--------------|------------|
        | Port         | Service    |
        | 22           | SSH        |
        | 80           | HTTP       |
        | 443          | HTTPS      |
        | 3306         | MySQL      |
        | 5432         | PostgreSQL |
        | 6379         | Redis      |
