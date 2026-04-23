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

cut — extract columns by delimiter

sort — alphabetical, numerical, reverse, unique

uniq — deduplicate, count

tr — translate/delete characters

wc — line/word/char count

head / tail — first/last N lines, follow mode
