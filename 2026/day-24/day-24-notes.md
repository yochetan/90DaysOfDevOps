Task 1: Git Merge — Hands-On

1) Create a new branch feature-login from main, add a couple of commits to it

        git checkout -b feature-login

2) Switch back to main and merge feature-login into main

        git merge feature-login

3) Observe the merge — did Git do a fast-forward merge or a merge commit?

        git did a fast-forward merge

4) Now create another branch feature-signup, add commits to it — but also add a commit to main before merging

        git checkout -b feature-signup

5) Merge feature-signup into main — what happens this time?

        git merge feature-signup

Answer in your notes:
What is a fast-forward merge?

    No divergence between branches
    Git doesn’t create a new merge commit
    It just “fast-forwards” to the latest commit
    
    main:    A---B
    feature:     \---C---D
    
    After fast-forward merge:
    main: A---B---C---D

When does Git create a merge commit instead?

    History has two separate lines
    Git combines them with a merge commit (M)
    
    In short: Git creates a merge commit when branches have diverged and can’t be fast-forwarded.

What is a merge conflict? (try creating one intentionally by editing the same line in both branches)

    Stops the merge
    Marks the conflict in the file
    Asks you to fix it manually


Task 2: Git Rebase — Hands-On

1) Create a branch feature-dashboard from main, add 2-3 commits

        git checkout -b feature-dashboard

2) While on main, add a new commit (so main moves ahead)
        
        git switch main
        vim backend.txt
        git add backend.txt
        git commit -m "feat: added backend file"

3) Switch to feature-dashboard and rebase it onto main

        git switch feature-dashboard
        git rebase main

4) Observe your git log --oneline --graph --all — how does the history look compared to a merge?

        git log --oneline --graph --all
        * 0c72252 (HEAD -> feature-dashboard) feat: added a cool feature
        * a92f25e chore: fixed a minor bug
        * fba52df feat: created dashboard
        * 6b2abb8 (main) feat: added backend file
        *   70a15c8 Merge branch 'feature-signup'
        |\  
        | * 9cf1e4a (feature-signup) feat: adding a signup feature
        * | 48f0793 feat: adding homepage
        |/  
        * 4f0e805 (feature-login) chore: fixed a bug
        * 2d4ec92 feat: adding a login feature

Answer in your notes:
What does rebase actually do to your commits?

    In Git, rebase rewrites your commits by moving them onto a new base.
    
    What actually happens
    Git takes your commits
    Re-applies them one by one on top of another branch
    Creates new commit IDs (they’re rewritten, not the same commits)
    
    Before:
    main:    A---B---C
                 \
    feature:      D---E
    
    After git rebase main:
    main:    A---B---C
                         \
    feature:              D'---E'

How is the history different from a merge?

    Merge history (non-linear)
    Keeps the original branch structure
    Adds a merge commit
    History looks like a branch graph
    Example:
    
    A---B---C-------M
         \         /
          D---E---F
    
    Rebase history (linear)
    Rewrites commits on top of main
    No merge commit
    History becomes a straight line
    Example:
    
    A---B---C---D'---E'---F'

Why should you never rebase commits that have been pushed and shared with others?

    In Git, you should avoid rebasing shared commits because rebase rewrites history.
    
    What goes wrong
    Rebasing changes commit IDs (hashes)
    Others still have the old commits
    This creates duplicate/conflicting histories
    Result
    Confusing commit history
    Merge conflicts when others pull
    Team members may lose or overwrite work
    
    Simple idea
    
    👉 You’re changing the past that others are already using

When would you use rebase vs merge?

    🔹 Use rebase when:
    You want a clean, linear history
    You’re working on your own local branch
    You want to update your branch with latest main before merging
    
    👉 Keeps history neat and easy to read
    
    🔹 Use merge when:
    You’re working on shared/public branches
    You want to preserve full history
    Multiple people are collaborating
    
    👉 Safer, no history rewriting
    
    Simple rule
    Rebase → clean history (private work)
    Merge → safe history (team work)


Task 3: Squash Commit vs Merge Commit

1) Create a branch feature-profile, add 4-5 small commits (typo fix, formatting, etc.)

        git branch feature-profile
        git switch feature-profile
        vim profile.txt
        added 4 commits 

2) Merge it into main using --squash — what happens?

        git merge --squash feature-profile

3) Check git log — how many commits were added to main?



Now create another branch feature-settings, add a few commits
Merge it into main without --squash (regular merge) — compare the history
Answer in your notes:
What does squash merging do?
When would you use squash merge vs regular merge?
What is the trade-off of squashing?
