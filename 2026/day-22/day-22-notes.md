Task 1: Install and Configure Git

1) Verify Git is installed on your machine

        git --version

2) Set up your Git identity — name and email

        git config --global user.name "<name>"
        git config --global user.email "<email>"

3) Verify your configuration

        git config --list : Displays the current git configuration


Task 2: Create Your Git Project

1) Create a new folder called devops-git-practice

        mkdir devops-git-practice

2) Initialize it as a Git repository

        git init

3) Check the status — read and understand what Git is telling you

        git status
        On branch master
        
        No commits yet

        nothing to commit (create/copy files and use "git add" to track)

4) Explore the hidden .git/ directory — look at what's inside

        to see .git : ls -a
        
        inside .git
        HEAD  branches  config  description  hooks  info  objects  refs

Task 3: Create Your Git Commands Reference

1) Create a file called git-commands.md inside the repo

2) Add the Git commands you've used so far, organized by category:

    Setup & Config

    Basic Workflow

    Viewing Changes

4) For each command, write:
    What it does (1 line)
    An example of how to use it

Task 4: Stage and Commit

1) Stage your file
        
        git add raj.txt
        git add simran.txt

2) Check what's staged

        git status
        On branch master
        
        No commits yet
        
        Changes to be committed:
          (use "git rm --cached <file>..." to unstage)
                new file:   raj.txt
                new file:   simran.txt

3) Commit with a meaningful message

        git commit -m "initial commit"

4) View your commit history

        git log


Task 5: Make More Changes and Build History

1) Edit git-commands.md — add more commands as you discover them

2) Check what changed since your last commit

3) Stage and commit again with a different, descriptive message

4) Repeat this process at least 3 times so you have multiple commits in your history
View the full history in a compact format


Task 6: Understand the Git Workflow
Answer these questions in your own words (add them to a day-22-notes.md file):

1) What is the difference between git add and git commit?

        git add : is used to staged from untracked (file/directory)
        git commit : is used to commit(tracked) from staged

2) What does the staging area do? Why doesn't Git just commit directly?

        staging area : instead of commiting all the changes you can select specific files which you wanna prepare for commit
        
        Why Git doesn’t commit directly
        If Git skipped the staging area and committed everything instantly, you’d lose a lot of control.

3) What information does git log show you?

        git log : it shows all the commit history made 

4) What is the .git/ folder and what happens if you delete it?

        .git/ : this is the folder where all the stuffs are stored ig
        
        if we delete that folder : it deletes the the git repo entirely

5) What is the difference between a working directory, staging area, and repository?

        WORKING DIRECTORY : this is the file system directory (if we haven't commited to the git and we delete the file it is completely deleted and can't be recovered)
        
        STAGING AREA : this is were we staged the files which we want before commit ( this is imp as we select which files are ready for commit )
        
        REPOSITORY : this is the repo/directory of git (where we can stored our files/directories and can recover if they are deleted from the file system)
