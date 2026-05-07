Task 1: Git Reset — Hands-On
1) Make 3 commits in your practice repo (commit A, B, C)

        git checkout -b practice-repo
        
        made 3 commits A, B, C

2) Use git reset --soft to go back one commit — what happens to the changes?

        BEFORE :
        git log --oneline
        2270121 (HEAD -> practice-repo) C
        64ef003 B
        39c532b A
        
        AFTER :
        git reset --soft 64ef003
        
        git log --oneline
        64ef003 (HEAD -> practice-repo) B
        39c532b A

        what happens to the changes?
        Commit removed, code stays ready to commit again

3) Re-commit, then use git reset --mixed to go back one commit — what happens now?

        git reset --mixed 64ef003

        what happens now?
        Removes the commit
        Keeps changes in files
        Unstages the changes
        Code stays, but you must git add again

4) Re-commit, then use git reset --hard to go back one commit — what happens this time?

                git reset --hard 64ef003
           what happens this time?
        Removes the commit
        Deletes changes from staging and files

Answer in your notes:
What is the difference between --soft, --mixed, and --hard?
      
      Simple comparison
      Option	Commit Removed	Changes Kept	Staged?
      --soft	✅	✅	✅
      --mixed	✅	✅	❌
      --hard	✅	❌	❌
    
    In short:
    
    --soft → keep staged changes
    --mixed → keep unstaged changes
    --hard → delete everything completely

Which one is destructive and why?

      git reset --hard
    Why it’s dangerous
    
    Once deleted, those changes are usually hard (or impossible) to recover unless they exist in:
    
    another commit
    stash
    remote repo
    
When would you use each one?
    
    git reset --soft
    Use when:
    * You want to undo a commit
    * But keep changes staged and ready to recommit
    
    git reset --mixed
    Use when:
    * You want to undo a commit
    * Keep changes in files
    * But unstage everything
    
    git reset --hard
    Use when:
    * You want to completely discard changes
    * And return to a clean state

Should you ever use git reset on commits that are already pushed?
    
    don’t reset shared pushed commits because it rewrites history and breaks collaboration.


Task 2: Git Revert — Hands-On
1) Make 3 commits (commit X, Y, Z)
        
        git checkout -b hands-on
        made 3 commits (X, Y, Z)

2) Revert commit Y (the middle one) — what happens?

        git revert 75356c9


3) Check git log — is commit Y still in the history?

        yes it is 
        In short: git revert does not remove history — it adds a new commit that reverses the old one.

Answer in your notes:
How is git revert different from git reset?

    git revert
    Undoes changes safely
    Creates a new commit
    Keeps full history intact
    Safe for shared/public branches
    
    git reset
    Moves branch backward
    Can remove commits/history
    May delete changes (--hard)
    Best for local/unshared commits
    
    revert → undo by adding new history
    reset → undo by rewriting history

Why is revert considered safer than reset for shared branches?

    revert → adds a safe “undo” commit
    reset → changes the past

When would you use revert vs reset?

    Revert → safe undo for shared history
    Reset → rewrite/remove local history


Task 3: Reset vs Revert — Summary

Create a comparison in your notes:

      | Feature                              | `git reset`                                      | `git revert`                             |
      | ------------------------------------ | ------------------------------------------------ | ---------------------------------------- |
      | **What it does**                     | Moves branch pointer back and can remove commits | Creates a new commit that undoes changes |
      | **Removes commit from history?**     | ✅ Yes                                            | ❌ No                                     |
      | **Safe for shared/pushed branches?** | ❌ No                                             | ✅ Yes                                    |
      | **When to use**                      | For local/private commits and history cleanup    | For safely undoing pushed/shared commits |


Task 4: Branching Strategies

Research the following branching strategies and document each in your notes with:

How it works (short description)

A simple diagram or flow (text-based is fine)

When/where it's used

Pros and cons
1) GitFlow — develop, feature, release, hotfix branches

2) GitHub Flow — simple, single main branch + feature branches

3) Trunk-Based Development — everyone commits to main, short-lived branches

4) Answer:
Which strategy would you use for a startup shipping fast?

Which strategy would you use for a large team with scheduled releases?

Which one does your favorite open-source project use? (check any repo on GitHub)


Task 5: Git Commands Reference Update

Update your git-commands.md to cover everything from Days 22–25:

Setup & Config

Basic Workflow (add, commit, status, log, diff)

Branching (branch, checkout, switch)

Remote (push, pull, fetch, clone, fork)

Merging & Rebasing

Stash & Cherry Pick

Reset & Revert
