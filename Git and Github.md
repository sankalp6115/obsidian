mkdir ~/.git-templates && cd ~/.git-templates
.git-templates % touch .gitignore
.git-templates % nano .gitignore
.git-templates % git config --global init.templateDir ~/.git-templates

##### This would tell git to use the gitignore file you defined in .git-templates folder, every time you hit git init this gitignore will be made.

Whiteboard Notes
https://app.eraser.io/workspace/P96VaUsW5o0FXVOTDzHY
https://app.eraser.io/workspace/ZNCxQWeQZlJ3BD4Fle26

Cheat Sheet
https://education.github.com/git-cheat-sheet-education.pdf


Clean commit should be done, not too much code pushed at the same time.
### Commands
add
commit
push
status

log (log shows full sha256)
log --oneline (shows commit id as smaller initial char of commit sha256)

show \<commit-id\>

diff

blame \<file-name\>
shows detailed overview of which line in the file was added when and in which commit by whom


revert  \<commit-id\>
What this does is, what code we did in the  \<commit-id\> , it makes a inverted code of that, 
if code was added it removes that, if code was removed it again writes it, makes a new commit and points the head towards it


reset \<commit-id\>
reset --hard \<commit-id\>
moves the **HEAD pointer** (your current branch) to another commit.
Resets head of branch to commit-id mentioned in command
If we go up the history, we completely lose the later commits.

reset vs reset --hard
reset brings head ptr to the specific commit and all commits after that are removed, but the code and file changes in working dir remain and the added changes are still kept in staging area.
“Undo commits, but keep all code changes locally.”

reset --hard brings head ptr to the specific commit and all commits after that are removed, but the code and file changes in working dir are also removed and staging area is completely cleaned.
“Go back in time and wipe everything after that commit.”

remote
git remote add \<any-user-friendly-name\> 
name is default origin
This gives git server an address to sync code with
Can also have multiple remotes, to sync in multiple places

clone
Contributor can clone repo to make their own changes

branch
git branch \<branch-name\>
creates a new branch

checkout
git checkout \<branch-name\>
travel between branches

![](Assets/Pasted%20image%2020260623022355.png)

git merge origin/\<feature-branch\>

merge and rebase

Merging
![](Assets/Pasted%20image%2020260623023518.png)

For a branch:
All of its commits are made into one commit and main head moves to that one commit.
The commits stay clean and under count.
But we lose history of commits in other branch, if we only want to revert changes of ptclr commit of branch we cant do that.

Rebasing
![](Assets/Pasted%20image%2020260623023604.png)
For a branch, 
rebasing will take all commits of the other branch and join them to current head of main and the last commit will become the head of main. 
Increases number of commits
But allows to revert to specific commit of the branch.

(Only used in special cases , mainly merge is used.)


git pull
### Staging Area
An imaginary area where files references are kept after they are added but not commited.


### SSH Key auth
### Contributor Invitation

### Branching
Very essential feature
![](Assets/Pasted%20image%2020260623020621.png)
![](Assets/Pasted%20image%2020260623020647.png)
![](Assets/Pasted%20image%2020260623020656.png)
![](Assets/Pasted%20image%2020260623020744.png)


### Branching Convention
![](Assets/Pasted%20image%2020260623022316.png)


### Stashing
git stash
saves a temporary commit in case we need to pull from remote while working on some non-commitable code. we stash our changes , pull from remote, then
git stash apply
git stash pop

works in stack format