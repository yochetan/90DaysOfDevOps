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

         command: sudo gpasswd - a tokyo developers
                  sudo gpasswd - a berlin developers
                  sudo gpasswd - a berlin admins
                  sudo gpasswd - a professor admins
         
Use appropriate command to check group membership:

         command: cat /etc/group
         developers:x:1004:tokyo,berlin
         admins:x:1005:berlin,professor

Task 4: Shared Directory

1) Create directory: /opt/dev-project

            command: sudo mkdir /opt/dev-project

2) Set group owner to developers

         command: sudo chgrp developers dev-project/

3) Set permissions to 775 (rwxrwxr-x)

         command: sudo chmod 775 dev-project/

4) Test by creating files as tokyo and berlin

         command: sudo touch tokyo
         sudo touch berlin
         not sure

Task 5: Team Workspace

1) Create user nairobi with home directory

         command: sudo useradd -m nairobi

2) Create group project-team

         command: sudo groupadd project-team

3) Add nairobi and tokyo to project-team

         command: sudo gpasswd -a nairobi project-team
         sudo gpasswd -a tokyo project-team

4) Create /opt/team-workspace directory

         command: sudo mkdir /opt/team-workspace

5) Set group to project-team, permissions to 775

         command: sudo chgrp project-team team-workspace/
         command: sudo chmod 775 team-workspace/

6) Test by creating file as nairobi
      
         command: sudo touch nairobi /opt/team-workspace/
         not sure
