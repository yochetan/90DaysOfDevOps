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
