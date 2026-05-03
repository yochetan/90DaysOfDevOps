Setup & Config

    git --version : to check which version we have on the system
    
    git config : this gives all the commands that we can put ahead of git
    
    git config --global user.name "<name>" : using this command we can set username for the git in the system
    
    git config --global user.email "<email>" : using this command we can set useremail for the git in the system
    
    git config --list : Displays the current git configuration

 Basic Workflow

    git init : to initialize a repo
  
    git status : tells the current status of the repo (as if there are any things to be committed or untracked/staged)

Viewing Changes

    git add filename.extension : this is run to staged a file/directory
    
    git commit -m "initial commit" : to commit that are staged in repo with a message
    
    git log : to view the commit history

Branch commands

    git branch
    
    git branch -a
    
    git branch -r
    
    git branch feature-1 : this creates a new branch called feature-1
    
    git checkout feature-1 : this swtiches to feature-1
    
    git checkout -b feature-2 : Creates a new branch and switch to it
    
    git switch → only for switching branches (simple & clear)
    
    git checkout → does multiple things (switch branches, restore files, etc.)
    
    git branch -d feature-2 : deletes the branch (in this feature-2)

