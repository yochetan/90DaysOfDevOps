Create a file named notes.txt

Write 3 lines into the file using redirection (> and >>)

Use cat to read the full file

Use head and tail to read parts of the file

Use tee once to write and display at the same time

Keep it short (8–12 lines total in the file)

Suggested command flow:

    touch notes.txt
    
    echo "Line 1" > notes.txt
    
    echo "Line 2" >> notes.txt
    
    echo "Line 3" | tee -a notes.txt
    tee - read from standard input and write to standard output and files
    
    ubuntu@ip-172-31-29-59:~$ echo "Line 1" > notes.txt 
    ubuntu@ip-172-31-29-59:~$ cat notes.txt 
    Line 1
    ubuntu@ip-172-31-29-59:~$ echo "Line 2" >> notes.txt 
    ubuntu@ip-172-31-29-59:~$ cat notes.txt 
    Line 1
    Line 2
    ubuntu@ip-172-31-29-59:~$ echo "Line 3" | tee -a notes.txt
    Line 3
    ubuntu@ip-172-31-29-59:~$ cat notes.txt 
    Line 1
    Line 2
    Line 3
    
    ubuntu@ip-172-31-29-59:~$ head -n 2 notes.txt 
    Line 1
    Line 2
    ubuntu@ip-172-31-29-59:~$ tail -n 2 notes.txt 
    Line 2
    Line 3

