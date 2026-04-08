Task 1: Your First Script

1) Create a file hello.sh

       vim hello.sh

       creates and open in editor

2) Add the shebang line #!/bin/bash at the top

       #!/bin/bash

       why: it ensures portability by telling system which interpreter to use.

3) Print Hello, DevOps! using echo

       echo " Hello, DevOps!"

4) Make it executable and run it

       chmod +x hello.sh [this gives executable to all (user,group,other)] OR chmod 764 hello.sh [this gives executable to just user]
       ./hello.sh [by writing this it get runs]

Document: What happens if you remove the shebang line?

      If you remove the shebang line (#!), the behavior of your script depends entirely on how you attempt to execute it:
      1. Direct Execution (./script.sh)
      2. Explicit Execution (python3 script.py or bash script.sh)
      3. Sourcing the Script (source script.sh or . script.sh)


Task 2: Variables

1) Create variables.sh with:

      A variable for your NAME

        NAME=Chetan

     A variable for your ROLE (e.g., "DevOps Engineer")

       ROLE="DevOps Engineer"

     Print: Hello, I am <NAME> and I am a <ROLE>

       echo "Hello, I am $NAME and I am a $ROLE"

2) Try using single quotes vs double quotes — what's the difference?

        with ' '
        ./variables.sh 
        Hello, I am $NAME and I am a $ROLE

        with " "
        ./variables.sh 
        Hello, I am Chetan and I am a DevOps Engineer


Task 3: User Input with read

1) Create greet.sh that:

   Asks the user for their name using read

          read -p "Enter Your Name: " name

   Asks for their favourite tool

          read -p "Your Favourite Tool Is: " tool

   Prints: Hello <name>, your favourite tool is <tool>

          echo "Hello $name, your favourite tool is $tool"


Task 4: If-Else Conditions
       
1) Create check_number.sh that:

   Takes a number using read

          read -p "Enter a number: " number
   
   Prints whether it is positive, negative, or zero

          if [ $number -gt 0 ]; then
                 echo "$number is positive"
          elif [ $number -lt 0 ]; then
                 echo "$number is negative"
          else
                 echo "$number is zero"
          fi

2) Create file_check.sh that:

   Asks for a filename

          read -p "Enter the filename: " filename

   Checks if the file exists using -f

          if [ -f $filename ]; then

   Prints appropriate message

          if [ -f $filename ]; then
                 echo "File exists"
          else
                 echo "File doesn't exists"
          fi


Task 5: Combine It All

Create server_check.sh that:

1) Stores a service name in a variable (e.g., nginx, sshd)

              read -p "Enter the service: " service

2) Asks the user: "Do you want to check the status? (y/n)"

              read -p "Do you want to check the status? (y/n) " yn

3) If y — runs systemctl status <service> and prints whether it's active or not



4) If n — prints "Skipped."
