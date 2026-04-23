Task 1: Basics

Document the following with short descriptions and examples:

1) Shebang (#!/bin/bash) — what it does and why it matters

        #!/bin/bash
        
        why: it ensures portability by telling system which interpreter to use.

2) Running a script — chmod +x, ./script.sh, bash script.sh

        chmod +x - GIVES EXECUTE FOR ALL (USER, GROUP, OTHERS)
        EX: chmod +x script.sh

        ./script.sh - RUNS THE SCRIPT FILE
        EX: ./addition.sh

        bash script.sh - TO EXECUTE A SHELL SCRIPT USING BASH 
        EX: bash addition.sh
        
3) Comments — single line (#) and inline

        # - IS USE TO WRITE ONE LINE COMMENTS
        <<COMMENT
         SAAAAIIOAAIJAJDOAJHFOSAHFIOH
        COMMENT 
        THE ABOVE IS USED FOR MULTIPLE LINES COMMENT  
    
4) Variables — declaring, using, and quoting ($VAR, "$VAR", '$VAR')

       var="value" {syntax} the equalto should be like this no spaces in between and the value should be in double quotes
       name="Chetan"  

       $VAR 
       file="my file.txt"
       echo $file 

       OUTPUT: 
       my file.txt   # treated as 2 arguments

        "$VAR"
        echo "$file"
        BEST PRACTICE
        ✔ Preserves spaces
        ✔ Safe
        ✔ Prevents word splitting

        '$VAR'            
        echo '$file' 
        OUTPUT:
        $file
  
5) Reading user input — read

        read -p "Enter your name: " name
        in the above what exactly is written is
        read - this is a keyword for taking a input  
        p - prompting
        in double quotes - the prompt/sentence to tell user and get an input
        name - this is a variable where the input will be stored
         
6) Command-line arguments — $0, $1, $#, $@, $?

        $0 - prints the script name
        $1 - prints the first argument
        $# - total number of arguments
        $@ - prints all the arguments
        $? - stores the exit status (return code)


Task 2: Operators and Conditionals

Document with examples:

1) String comparisons — =, !=, -z, -n

        = : equal to
        != : not equal to
        -z : to check if the string is empty
        -n : number of lines 

2) Integer comparisons — -eq, -ne, -lt, -gt, -le, -ge

        -eq : equal to a number
        -ne : not equal to a number
        -lt : less than a number
        -gt : greater than a number
        -le : less than or equal to a number
        -ge : greater than or equal to a number 

3) File test operators — -f, -d, -e, -r, -w, -x, -s

        -f : file
        -d : directory
        -e : exists
        -r : readable
        -w : writeable
        -x : executeable
        -s : non-empty file

4) if, elif, else syntax

        if [ condition ]; then
              echo ""
        elif [ condition ]; then 
              echo ""
        else
              echo ""
        fi

        OUTPUT:
        num=0
        
        if [ "$num" -gt 0 ]; then
            echo "Positive number"
        elif [ "$num" -lt 0 ]; then
            echo "Negative number"
        else
            echo "Zero"
        fi
   
5) Logical operators — &&, ||, !

         && : AND      
         || : OR
         ! : NOT 
  
6) Case statements — case ... esac

        case "$variable" in
            pattern1)
                # commands
                ;;
            pattern2)
                # commands
                ;;
            *)
                # default case
                ;;
        esac

        ;; → ends each case block
        * → acts like else (default case)
        esac → ends the case statement

        EXAMPLE:
        fruit="apple"
        
        case "$fruit" in
            apple)
                echo "It's an apple"
                ;;
            banana)
                echo "It's a banana"
                ;;
            orange)
                echo "It's an orange"
                ;;
            *)
                echo "Unknown fruit"
                ;;
        esac

Task 3: Loops

Document with examples:

1) for loop — list-based and C-style

        SYNTAX:
        for var in list
        do
            # commands
        done
        
        EXAMPLE:
        for fruit in apple banana mango
        do
            echo "$fruit"
        done

2) while loop

        SYNTAX:
        while [ condition ]
        do
            # commands
        done
        
        EXAMPLE:
        i=1
        
        while [ "$i" -le 5 ]
        do
            echo "Number: $i"
            ((i++))
        done

3) until loop

        SYNTAX:
        until [ condition ]
        do
            # commands
        done
        
        EXAMPLE:
        i=1
        
        until [ "$i" -gt 5 ]
        do
            echo "Number: $i"
            ((i++))
        done

4) Loop control — break, continue

        break → exit the loop completely
        continue → skip current iteration and move to next


5) Looping over files — for file in *.log

        SYNTAX:
        for file in *.log
        do
            echo "Processing $file"
        done

        EXAMPLE:
        for file in *.log
        do
            echo "Reading $file"
            cat "$file"
        done

6) Looping over command output — while read line

        SYNTAX:
        command | while read line
        do
            echo "$line"
        done
        
        EXAMPLE:
        ls | while read line
        do
            echo "File: $line"
        done


Task 4: Functions

Document with examples:

1) Defining a function — function_name() { ... }

        SYNTAX:
        function_name() {
            # commands
        }
        
        EXAMPLE:
        greet() {
            echo "Hello, World!"
        }
        
        greet
        
        OUTPUT:
        Hello, World!

2) Calling a function

        SYNTAX:
        greet

3) Passing arguments to functions — $1, $2 inside functions

        add() {
            sum=$(($1 + $2))
            echo "Sum: $sum"
        }
        
        add 5 10

4) Return values — return vs echo
        
        RETURN:
        check_even() {
            if (( $1 % 2 == 0 )); then
                return 0   # success
            else
                return 1
            fi
        }
        
        check_even 4
        echo $?   # prints return status
        
        ECHO:
        get_date() {
            echo "$(date)"
        }
        
        today=$(get_date)
        echo "Today is $today"

5) Local variables — local

        my_func() {
            local var="inside"
            echo "$var"
        }
        
        my_func


Task 5: Text Processing Commands

Document the most useful flags/patterns for each:

1) grep — search patterns, -i, -r, -c, -n, -v, -E

        grep is a powerful command used to search for patterns in text/files.
        
        Simple:
        grep "error" app.log
        
        -i (ignore case)
        grep -i "error" app.log
        
        -r (recursive search)
        grep -r "error" /var/log/
        
        -c (count matches)
        grep -c "error" app.log
        
        -n (line numbers)
        grep -n "error" app.log
        
        -v (invert match)
        grep -v "error" app.log
        
        -E (extended regex)
        grep -E "error|fail|critical" app.log

2) awk — print columns, field separator, patterns, BEGIN/END

        awk is a powerful text-processing tool—great for column extraction, filtering, and reporting.
        
        SYNTAX:
        awk 'pattern { action }' file
        
        pattern → condition (optional)
        action → what to do (e.g., print)
        
        Print Columns
        awk '{print $1}' file.txt
        
        Variable	Meaning
        $1, $2	  Fields (columns)
        $0	      Entire line
        NF	      Number of fields
        NR	      Line number
        
        Field Separator (-F)
        awk -F "," '{print $1, $2}' data.csv
        
        Patterns (Filtering)
        awk '/ERROR/ {print $0}' app.log
        
        BEGIN and END Blocks
        BEGIN (before processing starts)
        awk 'BEGIN {print "Start"} {print $0}' file.txt
        
        END (after processing finishes)
        awk '{sum += $1} END {print "Total:", sum}' numbers.txt

3) sed — substitution, delete lines, in-place edit

        sed (stream editor) is used to modify text non-interactively—perfect for quick edits, filtering, and transformations.
        
        SYNTAX:
        sed 'command' file
        
        Substitution (s)
        sed 's/error/ERROR/' app.log
        
        Delete Lines (d)
        sed '3d' file.txt
        
        In-Place Editing (-i)
        sed -i 's/error/ERROR/g' app.log

4) cut — extract columns by delimiter

        cut is a simple and fast command to extract specific columns (fields) or characters from text.
        
        SYNTAX:
        cut [options] file
        
        Extract by Delimiter (-d + -f)
        syntax:
        cut -d "delimiter" -f field_number file
        
        Default Delimiter (TAB)
        cut -f 1 file.txt
        
        Extract Characters (-c)
        cut -c 1-5 file.txt
        
        Extract Bytes (-b)
        cut -b 1-10 file.txt

5) sort — alphabetical, numerical, reverse, unique

        sort is used to order lines of text—alphabetically, numerically, or in custom ways. It’s often paired with tools like uniq, cut, and awk.
        
        SYNTAX:
        sort [options] file
        
        Alphabetical Sort
        sort names.txt
        
        Numerical Sort (-n)
        sort -n numbers.txt
        
        Reverse Sort (-r)
        sort -r names.txt
        
        Unique Sort (-u)
        sort -u file.txt

6) uniq — deduplicate, count

        uniq is used to filter or count duplicate lines—but there’s a catch: it only works on adjacent (consecutive) duplicates, so you usually pair it with sort.
        
        SYNTAX:
        uniq [options] file
        
        Remove Duplicates
        uniq file.txt
        
        Count Occurrences (-c)
        sort file.txt | uniq -c
        
        Show Only Duplicates (-d)
        sort file.txt | uniq -d
        
        Show Only Unique Lines (-u)
        sort file.txt | uniq -u

7) tr — translate/delete characters

        tr (translate) is used to transform or delete characters from input. It works with standard input/output, so it’s usually used with pipes.
        
        SYNTAX:
        tr [options] SET1 [SET2]
        SET1 → characters to match
        SET2 → characters to replace with
        
        Translate Characters
        Convert lowercase → uppercase
        echo "hello" | tr 'a-z' 'A-Z'
        OUTPUT: HELLO
        
        Convert uppercase → lowercase
        echo "HELLO" | tr 'A-Z' 'a-z'
        OUTPUT: hello
        
        Delete Characters (-d)
        echo "hello123" | tr -d '0-9'
        OUTPUT: hello

8) wc — line/word/char count

        wc (word count) is used to count lines, words, and characters in files or input.
        
        SYNTAX:
        wc [options] file
        
        Default Output
        wc file.txt
        
        Output format:
        lines  words  bytes  filename
        
        Example:
        10  50  300 file.txt
        
        -l (line count)
        wc -l file.txt
        
        -w (word count)
        wc -w file.txt
        
        -m (character count)
        wc -m file.txt

9) head / tail — first/last N lines, follow mode

        head — First N Lines
        
        SYNTAX:
        head file.txt
        
        tail — Last N Lines
        
        SYNTAX:
        tail file.txt


Task 6: Useful Patterns and One-Liners

Include at least 5 real-world one-liners you find useful. Examples:

Find and delete files older than N days
Count lines in all .log files
Replace a string across multiple files
Check if a service is running
Monitor disk usage with alerts
Parse CSV or JSON from command line
Tail a log and filter for errors in real time

Task 7: Error Handling and Debugging

Document with examples:

1) Exit codes — $?, exit 0, exit 1

        $? → last command status
        0 → success
        1 → failure
        exit N → set script exit status

2) set -e — exit on error

        set -e is a Bash option that makes your script exit immediately if any command fails (i.e., returns a non-zero exit code).

3) set -u — treat unset variables as error

        set -u (aka set -o nounset) makes Bash error out when you use an unset variable. It helps catch typos and missing inputs early instead of silently using empty strings.

4) set -o pipefail — catch errors in pipes

        set -o pipefail fixes a subtle but important problem with pipelines: by default, Bash only returns the exit status of the last command in a pipe.

5) set -x — debug mode (trace execution)

        set -x turns on debug (trace) mode in Bash. It prints each command before it runs, after variable expansion—great for understanding what your script is actually doing.

6) Trap — trap 'cleanup' EXIT

        trap lets you run commands automatically when a script exits or receives signals. It’s commonly used for cleanup (temp files, locks, background processes).
        
        SYNTAX:
        trap 'commands' SIGNAL


Task 8: Bonus — Quick Reference Table

Create a summary table like this at the top of your cheat sheet:

        Topic	        Key Syntax	        Example
        Variable	 VAR="value"	         NAME="DevOps"
        Argument	$1, $2	                 ./script.sh arg1
        If	        if [ condition ]; then	    if [ -f file ]; then
        For loop	for i in list; do	   for i in 1 2 3; do
        Function	name() { ... }	         greet() { echo "Hi"; }
        Grep	       grep pattern file	 grep -i "error" log.txt
        Awk	     awk '{print $1}' file	 awk -F: '{print $1}' /etc/passwd
        Sed 	     sed 's/old/new/g' file   	 sed -i 's/foo/bar/g' config.txt
