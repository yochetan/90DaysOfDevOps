Task 1: Create Users
Create three users with home directories and passwords:

tokyo
berlin
professor

create user with home directories

command: sudo useradd tokyo -m
         sudo useradd berlin -m
         sudo useradd professor -m

Check /etc/passwd and /home/ directory

to check /etc/passwd

command: cat /etc/passwd

tokyo:x:1001:1001::/home/tokyo:/bin/sh
berlin:x:1002:1002::/home/berlin:/bin/sh
professor:x:1003:1003::/home/professor:/bin/sh

to check /home/ directory

ls /home/
berlin  professor  tokyo  ubuntu

Task 2: Create Groups
Create two groups:

developers
admins

command: sudo groupadd developers
         sudo groupadd admins

Check /etc/group

command: cat /etc/group

Task 3: Assign to Groups

Assign users:

tokyo → developers
berlin → developers + admins (both groups)
professor → admins

