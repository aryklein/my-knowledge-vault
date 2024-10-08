---
tags:
  - git
  - cheatsheet
  - command
---
# Git cheat sheet

## Changing the Most Recent Commit Message

If you have just made a commit and realize you need to change the commit
message,
you can do so with:

```bash
git commit --amend -m "New commit message"
```

If you've already pushed the commit to a remote repository, you'll need to force
push the
amended commit:

```bash
git push --force
```

## WIP

When you're working on code that isn't complete yet but you want to commit it to
Git to save your progress or share with others, you can commit it as a Work In
Progress (WIP). Here's a step-by-step guide on how to do it:

1) **Stage Your Changes:**
   Use the command git add to stage the files you've been working on:
```bash
git add -A
```

2) **Commit Your Changes:**
   It’s a good practice to clearly mark your commit as **WIP** to indicate that
   the work is not complete. You can do this in the commit message:
```bash
git commit -m "WIP: Add initial structure for new feature"
```

3) **Push Your Changes (if necessary):**
   If you are collaborating with others and need to share this `WIP` commit,
   push it to the remote repository:
```bash
git push origin your_branch_name
```

4)  **Continue your work:**
   After committing, you can continue to work on your task. If you need to make
   more commits as WIP, you can keep using similar commit messages with "WIP" in
   them.

5) **Clean Up Your History (Optional):**
   Before merging your changes into a shared branch (like `main` or `master`),
   consider squashing your **WIP** commits into a single commit to maintain a
   clean commit history. This can be done via an interactive rebase.

   Replace `n` with the number of commits you want to squash. Follow the prompts
   to squash the commits.

```bash
git rebase -i HEAD~n
```

## Count the amount of commits in a branch

This command lists each commit with its short SHA and message in a single line,
making it easier to count.

```bash
git log --oneline
```

Or to count all commits on the current branch up to the most recent commit, you
can use:

```bash
git rev-list --count HEAD
```


## Squash all commits in a Git repository into one single commit

This method will squash all commits into one while preserving the changes made
in those commits.
### 1. Reset the branch to the first commit:

Use the following command to interactively rebase your branch starting from the
first commit:

```
git reset --soft $(git rev-list --max-parents=0 HEAD)
```

This command does a "soft" reset, which keeps the changes in the working
directory but clears the commit history.

### 2. Create a new commit:

Once the branch is reset to the first commit, all the changes will be staged.
You can now create a new commit with all the changes:

```bash
git commit -m "Your new commit message"
```

### 3. Force push the new commit (if pushing to a remote):

If this is a branch you are pushing to a remote repository (e.g., origin), you
will need to force push to overwrite the history

```bash
git push --force
```
