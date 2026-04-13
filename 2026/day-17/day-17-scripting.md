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


Task 4: Install Packages via Script

1) Create install_packages.sh that:

Defines a list of packages: nginx, curl, wget
Loops through the list
Checks if each package is installed (use dpkg -s or rpm -q)
Installs it if missing, skips if already present
Prints status for each package

        #!/bin/bash

        install_packages () {
                for package in nginx curl wget 
                do
                        echo "Checking $package"
                        if dpkg -s "$package" >/dev/null 2>&1; then
                                echo "$package is already installed"
                        else
                                echo "$package is not installed"
                                echo "Installing package $package"
                                sudo apt-install -y $package
                                if [ $? -eq 0 ]; then
                                        echo "$package is successfully installed"
                                else
                                        echo "failed to install $package"
                                fi
                        fi
                done
        }
        install_packages

        OUTPUT:
        Checking nginx
        nginx is already installed
        Checking curl
        curl is already installed
        Checking wget
        wget is already installed


Task 5: Error Handling

1) Create safe_script.sh that:

   Uses set -e at the top (exit on error)
   Tries to create a directory /tmp/devops-test
   Tries to navigate into it
   Creates a file inside
   Uses || operator to print an error if any step fails

        mkdir /tmp/devops-test || echo "Directory already exists"
        no output when directory it was run for the first time

        output after it was created:
        mkdir: cannot create directory ‘/tmp/devops-test’: File exists
        Directory already exists

2) Modify your install_packages.sh to check if the script is being run as root — exit with a message if not.

        if [ "$EUID" -ne 0 ]; then echo "Run as root"; exit 1; fi

           the output "Run as root" is when we run as a user
           and no output when it is runned as root
