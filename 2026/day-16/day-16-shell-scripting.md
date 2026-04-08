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

       
