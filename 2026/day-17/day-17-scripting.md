Task 1: For Loop

1) Create for_loop.sh that:
Loops through a list of 5 fruits and prints each one

        #!/bin/bash

        for fruits in apple banana watermelon mango orange
        do
                echo "$fruits"
        done

        OUTPUT:
             ./for_loop.sh 
          apple
          banana
          watermelon
          mango
          orange

2) Create count.sh that:
Prints numbers 1 to 10 using a for loop

        #!/bin/bash
        
        for i in {1..10}
        do
            echo $i
        done
  
        OUTPUT:
        1
        2
        3
        4
        5
        6
        7
        8
        9
        10


Task 2: While Loop

1) Create countdown.sh that:
Takes a number from the user

        read -p "Enter a number: " number

Counts down to 0 using a while loop
  
      while [ $number -ge 0 ] 
      do
          echo "iteration: $number"
          ((number--))
          
Prints "Done!" at the end

      echo "Done!"

      OUTPUT:
      Enter a number: 10
      Iteration: 10
      Iteration: 9
      Iteration: 8
      Iteration: 7
      Iteration: 6
      Iteration: 5
      Iteration: 4
      Iteration: 3
      Iteration: 2
      Iteration: 1
      Iteration: 0
      Done!


Task 3: Command-Line Arguments

1) Create greet.sh that:

Accepts a name as $1
Prints Hello, <name>!
If no argument is passed, prints "Usage: ./greet.sh "

        #!/bin/bash

        name=$1
        if [ $1 ]
        then
                echo "Hello, $name"
        else
                echo "Usage: ./greet.sh"
        fi

        OUTPUT:
        ./greet.sh Chetan
        Hello, Chetan
        
        ./greet.sh 
        Usage: ./greet.sh

2) Create args_demo.sh that:

Prints total number of arguments ($#)
Prints all arguments ($@)
Prints the script name ($0)

      #!/bin/bash
      
      echo "Total number of argumemts: $#"
      echo "All arguments: $@"
      echo "script name: $0"
      
      OUTPUT:
      ./args_demo.sh ek din pyaar
      Total number of argumemts: 3
      All arguments: ek din pyaar
      script name: ./args_demo.sh
