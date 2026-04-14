Task 1: Basic Functions

1) Create functions.sh with:

A function greet that takes a name as argument and prints Hello, <name>!

    greet () {
            echo "Hello, $1"
    }

A function add that takes two numbers and prints their sum

    add () {
            echo "sum of $1 + $2 is $(($1 + $2))" 
    }

Call both functions from the script

    greet Chetan
    add 2 5

    OUTPUT:
    Hello, Chetan
    sum of 2 + 5 is 7


Task 2: Functions with Return Values

1) Create disk_check.sh with:

A function check_disk that checks disk usage of / using df -h

    check_disk () {
        df -h / | tail -1
    }
    
A function check_memory that checks free memory using free -h
    
    check_memory () {
        free -h
    }
    
A main section that calls both and prints the results

    main () {
        check_disk
        check_memory
    }
    main

    OUTPUT:
    /dev/root       6.8G  2.6G  4.2G  38% /
                   total        used        free      shared  buff/cache   available
    Mem:           911Mi       372Mi       102Mi       2.7Mi       596Mi       539Mi
    Swap:             0B          0B          0B


Task 3: Strict Mode — set -euo pipefail

1) Create strict_demo.sh with set -euo pipefail at the top

        set -euo pipefail

2) Try using an undefined variable — what happens with set -u?

        $age
        echo $age

3) Try a command that fails — what happens with set -e?
    
        false
        echo "This will not run"
    
4) Try a piped command where one part fails — what happens with set -o pipefail?

        false | true
        echo $?


set -e → Stops script on any failure
set -u → Crashes on undefined variable
set -o pipefail → Makes pipelines fail properly


Task 4: Local Variables

1) Create local_demo.sh with:

A function that uses local keyword for variables
Show that local variables don't leak outside the function
Compare with a function that uses regular variables

      #!/bin/bash
      
      local_function () {
              local my_var="value"
              echo "LOCAL VARIABLE: $my_var"
      }
      
      normal_function () {
              var="v1"
              echo "GLOBAL VARIABLE: $var"
      }
      
      local_function
      normal_function
      
      echo "OUTSIDE FUNC_LOCAL: $my_var"
      echo "OUTSIDE FUNC_GLOBAL: $var"
      
      OUTPUT:
      LOCAL VARIABLE: value
      GLOBAL VARIABLE: v1
      OUTSIDE FUNC_LOCAL: 
      OUTSIDE FUNC_GLOBAL: v1


Task 5: Build a Script — System Info Reporter

Create system_info.sh that uses functions for everything:

1) A function to print hostname and OS info

        system_info() {
            echo "===== SYSTEM INFO ====="
            echo "Hostname: $(hostname)"
            echo "OS Information:"
            cat /etc/os-release | grep PRETTY_NAME
            echo
        }

        >>The PRETTY_NAME variable is commonly used in Linux systems within the /etc/os-release file to provide a human-readable description of the operating                 system. 
        
2) A function to print uptime: The uptime command is a standard utility on Unix-like operating systems (including Linux and macOS) that provides a snapshot of how long the system has been running and its current performance load.

        check_uptime() {
            echo "===== UPTIME ====="
            uptime -p
            echo
        }

3) A function to print disk usage (top 5 by size)

        disk_usage() {
            echo "===== TOP 5 DISK USAGE ====="
            du -ah / 2>/dev/null | sort -rh | head -n 5 || true
            echo
        }

        The du (disk usage) command measures the disk space occupied by files or directories.
        -n	Numeric Sort: Sorts strings based on their numerical value.
        -r	Reverse: Reverses the order of the sort.

4) A function to print memory usage

           memory_usage() {
            echo "===== MEMORY USAGE ====="
            free -h
            echo
        }

        free -h command is used in Linux and Unix-like operating systems to display a summary of system memory usage in a human-readable format.

5) A function to print top 5 CPU-consuming processes

        cpu_usage() {
            echo "===== TOP 5 CPU PROCESSES ====="
            ps -eo pid,comm,%cpu --sort=-%cpu | head -n 6 || true
            echo
        }

        ps -eo pid,comm,%cpu

        ps = shows running processes
        
        Options:
        
        -e → show all processes
        -o → custom output format
        
        Fields:
        
        pid → Process ID
        comm → Command name (process name)
        %cpu → CPU usage

        --sort=-%cpu

        Sorts processes by CPU usage
        
        %cpu → sort by CPU
        - → descending order (highest first)
        
        So top CPU-consuming processes come first
        
6) A main function that calls all of the above with section headers

        main() {
            system_info
            check_uptime
            disk_usage
            memory_usage
            cpu_usage
        }
        
        main

7) Use set -euo pipefail at the top

       set -euo pipefail


OUTPUT:

        ./system_info.sh 
        ===== SYSTEM INFO =====
        Hostname: ip-172-31-29-59
        OS Information:
        PRETTY_NAME="Ubuntu 24.04.4 LTS"
        
        ===== UPTIME =====
        up 20 minutes
        
        ===== TOP 5 DISK USAGE =====
        3.7G    /
        1.7G    /usr
        1017M   /snap
        957M    /usr/lib
        870M    /var
        
        ===== MEMORY USAGE =====
                       total        used        free      shared  buff/cache   available
        Mem:           911Mi       417Mi        99Mi       2.7Mi       598Mi       494Mi
        Swap:             0B          0B          0B
        
        ===== TOP 5 CPU PROCESSES =====
            PID COMMAND         %CPU
            549 snapd            0.1
              1 systemd          0.1
            543 amazon-ssm-agen  0.1
            121 systemd-journal  0.0
             54 kswapd0          0.0
