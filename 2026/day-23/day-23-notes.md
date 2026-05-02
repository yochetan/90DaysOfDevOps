Task 1: Understanding Branches
Answer these in your day-23-notes.md:

1) What is a branch in Git?
        
        A branch in Git is a separate workspace to try changes without affecting the main project.
        LIKE making versions of the projects such as bluewhale(V1), bluewhale(V2)

2) Why do we use branches instead of committing everything to main?

        In Git, we use branches so you can work safely on changes without breaking the main code.
        If everyone committed directly to main:
        
        Bugs or unfinished features could break the project
        It becomes messy and hard to manage

3) What is HEAD in Git?

        In Git, HEAD is a pointer to the current commit (or branch) you are working on.

4) What happens to your files when you switch branches?

        In Git, when you switch branches, your files are updated to match the state of that branch.


Task 2: Branching Commands — Hands-On
In your devops-git-practice repo, perform the following:

1) List all branches in your repo

        git branch : shows all branches (local)
        * master
        
        git branch -a : shows all branches (local + remote)
        
        git branch -r : shows only remote branches

2) Create a new branch called feature-1

        git branch feature-1 : this creates a new branch called feature-1

3) Switch to feature-1

        git checkout feature-1 : this swtiches to feature-1

4) Create a new branch and switch to it in a single command — call it feature-2

        git checkout -b feature-2

5) Try using git switch to move between branches — how is it different from git checkout?

        git switch → only for switching branches (simple & clear)
        git checkout → does multiple things (switch branches, restore files, etc.)

6) Make a commit on feature-1 that does not exist on main

        git log
        commit d1603243037e000de3ed9bbdbded0886d8c36e82 (HEAD -> feature-1)
        Author: Ubuntu <ubuntu@ip-172-31-29-59.us-west-2.compute.internal>
        Date:   Sat May 2 19:11:58 2026 +0000
        
            first commit
        
        commit 4f3a1df9ddc3c2d69094cef31fa279091a8112da (master, feature-2)
        Author: Ubuntu <ubuntu@ip-172-31-29-59.us-west-2.compute.internal>
        Date:   Thu Apr 30 18:51:33 2026 +0000
        
            initial commit

7) Switch back to main — verify that the commit from feature-1 is not there

        git log
        commit 4f3a1df9ddc3c2d69094cef31fa279091a8112da (HEAD -> master, feature-2)
        Author: Ubuntu <ubuntu@ip-172-31-29-59.us-west-2.compute.internal>
        Date:   Thu Apr 30 18:51:33 2026 +0000
        
            initial commit

8) Delete a branch you no longer need

        git branch -d feature-2
        Deleted branch feature-2 (was 4f3a1df).

9) Add all branching commands to your git-commands.md

        git branch
        git branch -a
        git branch -r
        git branch feature-1 : this creates a new branch called feature-1
        git checkout feature-1 : this swtiches to feature-1
        git checkout -b feature-2 : Creates a new branch and switch to it
        git switch → only for switching branches (simple & clear)
        git checkout → does multiple things (switch branches, restore files, etc.)
        git branch -d feature-2 : deletes the branch (in this feature-2)


Task 3: Push to GitHub

1) Create a new repository on GitHub (do NOT initialize it with a README)

        demo_repo_for_devops

2) Connect your local devops-git-practice repo to the GitHub remote

        git remote add origin https://github.com/yochetan/demo_repo_for_devops.git
        git remote -v : to verify the connection

3) Push your main branch to GitHub

AFTER ADDING THE ORIGIN WE HAD TO DO THE SHH ONE

CREATED ONE SHH KEYGEN

        cd .shh
        shh-keygen

then updated the git remote

        git remote set-url origin git@github.com:yochetan/demo_repo_for_devops.git

then finally we pushed the main/master branch

        git push origin master

4) Push feature-1 branch to GitHub

first change the branch

        git switch feature-1

then push it

        git push origin feature-1

5) Verify both branches are visible on GitHub

        Yes both branches are visible but idk about compare and pull requests
        lets try it

6) Answer in your notes: What is the difference between origin and upstream?

        origin = your copy (where you push)
        upstream = original repo (where you pull updates from)


Task 4: Pull from GitHub

1) Make a change to a file directly on GitHub (use the GitHub editor)

        hi this is my story : in the raj.txt

2) Pull that change to your local repo
        
        git pull origin master

3) Answer in your notes: What is the difference between git fetch and git pull?

        git fetch → get changes, don’t apply
        git pull → get changes and apply


Task 5: Clone vs Fork

1) Clone any public repository from GitHub to your local machine

        git clone https://github.com/yochetan/test_repo.git
        Cloning into 'test_repo'...
        remote: Enumerating objects: 3, done.
        remote: Counting objects: 100% (3/3), done.
        remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
        Receiving objects: 100% (3/3), done.

2) Fork the same repository on GitHub, then clone your fork



3) Answer in your notes:

What is the difference between clone and fork?

        Fork = copy on GitHub
        Clone = copy on your computer

When would you clone vs fork?

        Clone → work directly on repo
        Fork → create your own copy, then contribute

After forking, how do you keep your fork in sync with the original repo?

Add upstream (one-time setup)

        git remote add upstream https://github.com/original-owner/repo-name.git

Fetch latest changes

        git fetch upstream

Merge into your branch

        git checkout main
        git merge upstream/main
