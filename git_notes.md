# Git notes

## Setup a repo

- Step 1: On `github.com`: create an emtpy repo `test-repo` under your account (here `trungtly`)
- Step 2: On local machine
```bash
git init test-repo
cd test-repo
echo "# test-repo" >> README.md
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/trungtly/test-repo.git
git push -u origin main
```

## Developing

```bash
# clone the remote repo (first time only) or pull latest changes to local 
git clone https://github.com/trungtly/test-repo.git .
git pull

# create a new branch, make some changes
git checkout -b feat_branch
git branch
echo "#### more testing" >> README.md
git add .
git status
git commit -m "more testing"

# push the changes (`--set-upstream` only for the first time)
git push --set-upstream origin feat_branch
git push

# Create a Pull Request (PR) to merge to another branch (eg `main`)
# Once the PR is approved and merged, the `main` branch (remote only) is updated with our changes
# need to pull those changes to local
git checkout main
git pull

# Delete a branch once done
git branch -d feat_branch # local
git push origin --delete feat_branch
```

## Useful git alias
- put the `alias` in your `$HOME/.gitconfig`
```bash
[alias]
l = log --pretty=format:"%C(yellow)%h\\ %ad%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate --date=short
s = status -s
# show all alias
la=!git config -l | grep alias | cut -c 7-
```