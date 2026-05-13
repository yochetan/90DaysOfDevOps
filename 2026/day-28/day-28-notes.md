Task 1: Self-Assessment Checklist
Go through the checklist below. For each item, mark yourself honestly:

* Can do confidently
* Need to revisit
* Haven't done yet

Linux

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

Shell Scripting

1. Script with Variables, Arguments, and User Input

Example Script

        #!/bin/bash
        
        name=$1
        
        echo "Enter your city:"
        read city
        
        echo "Hello $name from $city"

Run Script
        
        chmod 764 script.sh
        ./script.sh Chetan

Important Variables   
        
        |---------------------|--------------------------|
        | Variable            | Meaning                  |
        | $0                  | Script name              |
        | $1                  | First argument           |
        | $2                  | Second argument          |
        | $#                  | Number of arguments      |
        | $@                  | All arguments            |
        | $?                  | Last command exit status |

2. if / elif / else

Example

        #!/bin/bash
        
        num=10
        
        if [ $num -gt 5 ]; then
            echo "Greater than 5"
        elif [ $num -eq 5 ]; then
            echo "Equal to 5"
        else
            echo "Less than 5"
        fi

3. case Statement

Example

        #!/bin/bash
        
        echo "Enter option:"
        read option
        
        case $option in
            start)
                echo "Starting service"
                ;;
            stop)
                echo "Stopping service"
                ;;
            restart)
                echo "Restarting service"
                ;;
            *)
                echo "Invalid option"
                ;;
        esac

4. for Loop

Example

        #!/bin/bash
        
        for i in 1 2 3 4 5
        do
            echo "Number: $i"
        done

5. while Loop

Example

        #!/bin/bash
        
        count=1
        
        while [ $count -le 5 ]
        do
            echo $count
            ((count++))
        done

6. until Loop

Example

        #!/bin/bash
        
        count=1
        
        until [ $count -gt 5 ]
        do
            echo $count
            ((count++))
        done

7. Functions

Example
        
        #!/bin/bash
        
        greet() {
            echo "Hello $1"
        }
        
        greet Chetan

Function Return Value

        #!/bin/bash
        
        add() {
            return $(($1 + $2))
        }
        
        add 2 3
        echo $?

8. grep

Search Text

        grep "ERROR" logfile.txt

Ignore Case

        grep -i "error" logfile.txt

9. awk

Print Columns

        awk '{print $1, $3}' file.txt

Sum Values

        awk '{sum += $1} END {print sum}' numbers.txt

10. sed

Replace Text

        sed 's/apple/orange/g' file.txt

Delete Line

        sed '2d' file.txt

11. sort

Alphabetical Sort

        sort names.txt

Reverse Sort

        sort -r names.txt

12. uniq

Remove Duplicate Lines

        uniq file.txt

Count Duplicates

        uniq -c file.txt

13. Combined Text Processing Example

Find Top Errors

        grep "ERROR" logfile.txt | awk '{print $5}' | sort | uniq -c | sort -rn

14. Error Handling

set -e - Exit script if any command fails.

        set -e

set -u - Treat unset variables as errors.

        set -u

set -o pipefail - Catch errors in pipelines.

        set -o pipefail
15. trap

Cleanup on Exit

        trap 'echo Cleanup done' EXIT

16. Full Safe Script Example
        
        #!/bin/bash
        
        set -e
        set -u
        set -o pipefail
        
        trap 'echo Script failed' ERR
        
        echo "Running safely"

17. Schedule Scripts with Crontab

Open Crontab

        crontab -e

Run Every Day at 2 AM

        0 2 * * * /home/ubuntu/backup.sh

Run Every 5 Minutes

        */5 * * * * /home/ubuntu/check.sh

Cron Format

        * * * * *
        - - - - -
        | | | | |
        | | | | +---- Day of Week
        | | | +------ Month
        | | +-------- Day of Month
        | +---------- Hour
        +------------ Minute

