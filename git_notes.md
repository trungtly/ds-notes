# Git notes

## Simple workflow
- On `github`: create a new repo under your account or use an existing repo
- On local:
```bash
# Step 0: Clone an existing remote repo (first time only) or pull latest changes 
git clone https://github.com/user-name/project-name.git
cd project-name

# Step 1: Create a feature branch, make some changes
git checkout main
git pull
git checkout -b feat_branch
echo "more testing" >> testing.txt
git add testing.txt
git status
git commit -m "more testing"

# Step 3: Push the changes (--set-upstream first time only)
git push --set-upstream origin feat_branch
git push

# Step 3: On github: Create a Pull Request (PR) to merge to another branch (eg main)
# Once the PR is merged, the main branch (remote only) is updated with our changes. 

# Step 4: Pull those changes to local and clean up the feature branch
git checkout main
git pull
git branch -d feat_branch
```
## Useful git commands
```bash
# Undo the latest commit
git commit -m "some wrong change"
git reset --hard HEAD~1
git push --force

# Reorder/remove/combine/split commits that are not in main (before push or merge)
git checkout feat_branch
git rebase -i origin/main # work in editor
git push --force
```

## Typing less with git alias
- put the `alias` in your `$HOME/.gitconfig`
```bash
[alias]
    # View abbreviated SHA, description, and history graph of the latest 20 commits
    l = log --pretty=oneline -n 20 --graph --abbrev-commit

    # View the current working tree status using the short format
    s = status -s

    # Show the diff between the latest commit and the current state
    d = !"git diff-index --quiet HEAD -- || clear; git --no-pager diff --patch-with-stat"

    # Add all changes and commit
    ac = !git add -A && git commit -av

    # Show all alias
    la = !git config -l | grep alias | cut -c 7-
```
