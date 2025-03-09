---
Created: 2024-08-06T22:41
---
#### Enable `push` from local repository to remote
```bash
# Initialize empty git repository.
# This will create a default branch named `main` or `master`
git init

# Stage and commit current files
git add .
git commit -m 'message'

# Add a pointer from local repo to the remote one
git remote add origin 'https://github.com/<user_name>/<repo_name.git>'

# Turn main branch is named `main`
git branch -M main

# Push commited changes to remote repo
git push -u origin main
```
---
#### Remove `.git` directory
```bash
rm -rf .git
```
---
#### Change git repository `remote url`
```bash
git remote set-url "origin" git@github.com:User/UserRepo.git
```
---
#### `Merge` local branch into local main
```bash
# on local branch
git add .
git commit -m 'message'

# switch to main and merge
git switch main
git merge <branch_name>
```
---
#### Update local branch with commits from `main`
```bash
# When working in a team you may need to update your branch with the most
# recent commits from main to have the actual code version thus avoiding 
# potential conflicts when merging branch to main.
# The `rebase` command will add commits from one branch to another.
git add .

# add new commit on top of the commit stack of your working branch
git commit -m 'message'

# add all commits from <main> branch starting from the moment when the
# local branch was produced from main. Commits will be added right before
# the first <local branch> commit.
git rebase main

# merge new commits from <local branch>
git switch main
git merge <branch_name>
```
---
#### Show one-line `logs`
```bash
git log --oneline
```
---
#### Show commit details
```bash
# Show the last commit's details.
git show HEAD

# Show details of the commit previous to last.
git show HEAD~1
```
---
#### `Reset` last commit
```bash
# Erase the last commit from history, add changes to working tree.
git reset HEAD~1

# After that you can add your files and commit again
git add .
git commit -m 'message'

# Erase the last commit from history, add changes to staged.
# No need to use `git add`, you can commit immediately.
git reset --soft HEAD~1

# Erase the last commit from history, drop changes.
git reset --hard HEAD~1
```
---
#### `Revert` last commit
```bash
# This will preserve the last commit and add a new commit that will perform
# operations opposite to the last commit.
git revert HEAD
```
---
#### Create new `branch` and `switch` to it
```bash
git switch -c <new_branch>
# OR
git checkout -b <new_branch>
```
---
#### `Delete` branch
```bash
git branch -d <branch_name>  # Only works for fully committed branches
git branch -D <branch_name>  # Force delete
```
---
#### `Stash` changes
```bash
# Hide all tracked and uncommitted changes present on a branch.
git stash
# Git will create a stack of stashes. Each new stash will come before the
# previous one.

# View all stashed changes
git stash list

# Pop the last stash from the stash stack. All changes included into this
# stash will be staged again.
git stash pop

# Apply the stash (add all changes to staged area), but do not pop from stack.
git stash apply

# Drop the stash without applying the changes.
git stash drop  # drop latest
# OR
git stash drop <stash_name>  # drop specific stash

# View brief details of the last stash (file names, how many lines were added
# or removed from files)
git stash show

# View detailed info
git stash show -p

# View details for an arbitrary stash
git stash show <stash_name> [-p]
```
---
#### Edit commit history with `rebase --interactive`
```bash
# Git allows manually edit commit history.

# Edit last two commits.
git rebase --interactive HEAD~2

# This will open the text editor (e.g. Nano) with some usefull options about
# the chosen commits written in it.

# At the top of the document will be listed chosed commits in format:
# <action> <commit_name> <commit_message>. Commits will be listed starting
# from the oldest one.
pick c02c12e Updated README.md
pick 9b2e3ec Add new test cases

# After that commit options will be listed:
p, pick <commit> - use (apply) this commit
r, reword <commit> - use this commit, but edit commit message
# Using `r` will  open another text editor to enter updated message.
d, drop <commit> - drop this commit
s, squash <commit> - use this commit but mix it to the previous one
# Using `s` will open another text editor session where you can edit commit
# messages or leave them as the were.

# Rebase all commits from the first one. 
git rebase --interactive --root
```
---
#### Exclude selected files from staging
```bash
# Excluded files should be prefixed with `:!` and enclosed in quotes. 
git add --all -- ':!/src/exclude1.txt' ':!src/exclude2.txt'
```
---
#### Add ssh keys to Gitlab and Github on WSL
```bash
# Run PowerShell in admin mode (via GUI)

# Generate ssh keys both for gitlab and github 
ssh-keygen.exe -t rsa -f C:\Users\user\.ssh\gitlab -C user@pc
ssh-keygen.exe -t rsa -f C:\Users\user\.ssh\github -C user@pc

# Check private keys' properties and ensure that only system and user have access to this file. This is done via GUI. Be sure to deactivate inheritance.

# Check if the ssh-agent is active on PowerShell
Get-Service ssh-agent
# ---------------------||--------------------------
#	  				   \/
Status   Name               DisplayName
------   ----               -----------
Running  ssh-agent          OpenSSH Authentication Agent

# If the status is not running try following commands
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent

# Add private keys to powershell ssh-client
ssh-add C:\Users\user\.ssh\gitlab
ssh-add C:\Users\user\.ssh\github

# Create config file in C:\Users\user\.ssh directory with contents below:
Host gitlab
    HostName gitlab.com
    PreferredAuthentications publickey # this may not be strictly neccessary
    IdentityFile C:/Users/smira/.ssh/gitlab

Host github
    HostName github.com
    IdentityFile /mnt/c/Users/smira/.ssh/github

# Copy the contents of your local windows `.ssh` folder to WSL
sudo cp -r /mnt/c/Users/user/.ssh/* ~/.ssh

# Add private keys to WSL ssh-agent
eval $(ssh-agent -s) # start the agent
ssh-add ~/.ssh/gitlab
ssh-add ~/.ssh/github

# Run to check gitlab and github connections
sudo ssh -Tv git@gitlab.com
sudo ssh -Tv git@github.com

# Don't forget to set user name and email local to each project
git config --local user.name = name
git config --local user.email = email@email.com

################## Update #####################
# You can use Git-Bash utility for a more convinient workflow
# Update the config file to add the private on-prem repository

# Private GitLab instance
Host hostname.gitlab.yandexcloud.net
   PreferredAuthentications publickey
   IdentityFile /c/Users/username/.ssh/gitlab_key

# Add the ssh key named `gitlab_key` to your gitlab account.
# Create new directory and `git init` from it.

# Configure local name and email.
git config --local user.name "name"
git config --local user.email "email@email.com"

# Bind local and remote repositories
git remote add origin 'https://github.com/<user_name>/<repo_name.git>'

# Make your main branch
git branch -M main

# Now every time you make push from this repository, Git-Bash will automatically resolve your commits to remote repo.
git push -u origin main
```
---
#### List git config
```bash
# global
git config --global --list

# local
git config --global --list
```
---
