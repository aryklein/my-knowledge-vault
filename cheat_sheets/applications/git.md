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
