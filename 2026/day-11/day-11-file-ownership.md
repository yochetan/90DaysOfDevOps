Task 1: Understanding Ownership

1) Run ls -l in your home directory
ls -l
total 16
-rwxrw-r-- 1 ubuntu ubuntu   50 Apr  2 20:08 backup.sh
-r--r--r-- 1 ubuntu ubuntu    0 Apr  5 19:56 devops.txt
-rw-r----- 1 ubuntu ubuntu   47 Apr  5 20:02 notes.txt
drwxr-xr-x 2 ubuntu ubuntu 4096 Apr  5 20:30 project
-rwxrw-r-- 1 ubuntu ubuntu   33 Apr  5 20:07 script.sh

2) Identify the owner and group columns
-rwxrw-r-- 1      ubuntu ubuntu   33   Apr  5 20:07 script.sh
file permissions  owner  group   size  date         filename
owner: ubuntu
group: ubuntu

3) Check who owns your files
ubuntu

Document: What's the difference between owner and group?
owner means only one user
group means multiple users


Task 2: Basic chown Operations

1) Create file devops-file.txt
command: touch devops-file.txt

2) Check current owner: ls -l devops-file.txt
ubuntu

3) Change owner to tokyo (create user if needed)
command: sudo chown tokyo devops-file.txt

4) Change owner to berlin
command: sudo chown berlin devops-file.txt

5) Verify the changes
when this command: sudo chown tokyo devops-file.txt
-rw-rw-r-- 1 tokyo ubuntu 0 Apr  5 20:44 devops-file.txt

when this command: sudo chown berlin devops-file.txt
-rw-rw-r-- 1 berlin ubuntu 0 Apr  5 20:44 devops-file.txt


Task 3: Basic chgrp Operations

1) Create file team-notes.txt
command: touch team-notes.txt

2) Check current group: ls -l team-notes.txt
ubuntu

3) Create group: sudo groupadd heist-team
command: sudo groupadd heist-team

4) Change file group to heist-team
command: sudo chgrp heist-team team-notes.txt

5) Verify the change
before step 4:
ls -l team-notes.txt 
-rw-rw-r-- 1 ubuntu ubuntu 0 Apr  5 20:53 team-notes.txt
after step 4:
ls -l team-notes.txt 
-rw-rw-r-- 1 ubuntu heist-team 0 Apr  5 20:53 team-notes.txt


Task 4: Combined Owner & Group Change

1) Create file project-config.yaml
command: touch project-config.yaml

2) Change owner to professor AND group to heist-team (one command)
command: sudo chown professor:heist-team project-config.yaml

3) Create directory app-logs/
command: mkdir app-log/

4) Change its owner to berlin and group to heist-team
command: sudo chown berlin:heist-team app-log/


Task 5: Recursive Ownership

1) Create directory structure:
mkdir -p heist-project/vault
mkdir -p heist-project/plans
touch heist-project/vault/gold.txt
touch heist-project/plans/strategy.conf

2) Create group planners: sudo groupadd planners

