Task 1: Create Files
1) Create empty file devops.txt using touch
command: touch devops.txt 

2) Create notes.txt with some content using cat or echo
command: touch notes.txt
         echo "Today i made pav bhaji with the help of Unnati" > notes.txt {this added the sentence in the notes.txt}
         cat notes.txt {this read the content of notes.txt}

3) Create script.sh using vim with content: echo "Hello DevOps"
command: vim script.sh {to create the script.sh file and open in editor to edit}
         cat script.sh {to read the contents of script.sh}

ls -l to see permissions


Task 2: Read Files
1) Read notes.txt using cat
command: cat notes.txt

2) View script.sh in vim read-only mode
command: vim script.sh

3) Display first 5 lines of /etc/passwd using head
command: head -n 5 /etc/passwd

4) Display last 5 lines of /etc/passwd using tail
command: tail -n 5 /etc/passwd


Task 3: Understand Permissions
Format: rwxrwxrwx (owner-group-others)

r = read (4), w = write (2), x = execute (1)
Check your files: ls -l devops.txt notes.txt script.sh

Answer: What are current permissions? Who can read/write/execute?

-rw-rw-r-- 1 ubuntu ubuntu  0 Apr  5 19:56 devops.txt
owner - read/write, group - read/write, others - read

-rw-rw-r-- 1 ubuntu ubuntu 47 Apr  5 20:02 notes.txt
owner - read/write, group - read/write, others - read

-rw-rw-r-- 1 ubuntu ubuntu 33 Apr  5 20:07 script.sh
owner - read/write, group - read/write, others - read


Task 4: Modify Permissions
1) Make script.sh executable → run it with ./script.sh
command: chmod 764 script.sh {this modifies permissions [gives rwx:user, rw-:group, r--:others]}
         ./script.sh {this runs the shell file} 
      
2) Set devops.txt to read-only (remove write for all)
command: chmod 444 devops.txt {this modifies permissions [gives r--:user, r--:group, r--:others]}

3) Set notes.txt to 640 (owner: rw, group: r, others: none)
command: chmod 640 notes.txt {this modifies permissions [gives rw-:user, r--:group, ---:others]}

4) Create directory project/ with permissions 755
command: mkdir project/
         chmod 755 project/ {this modifies permissions [gives rwx:user, r-x:group, r-x:others]}

command: ls -l
-r--r--r-- 1 ubuntu ubuntu    0 Apr  5 19:56 devops.txt
-rw-r----- 1 ubuntu ubuntu   47 Apr  5 20:02 notes.txt
drwxr-xr-x 2 ubuntu ubuntu 4096 Apr  5 20:30 project
-rwxrw-r-- 1 ubuntu ubuntu   33 Apr  5 20:07 script.sh


Task 5: Test Permissions
1) Try writing to a read-only file - what happens?
W10: Warning: Changing a readonly file  

2) Try executing a file without execute permission
permission denied

3) Document the error messages
