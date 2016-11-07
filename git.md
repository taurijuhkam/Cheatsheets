#Git basics
##Basic workflow commands
Description                     |Command
--------------------------------|----------------------------------
Create a new local repository   | `git init`
Clone a repository              | `git clone ssh/https_repo_url`
View current status             | `git status`
View current changes            | `git diff <file>`
Add/(stage) all changes to the next commit | `git add .`
Add/(stage) changes in <file> to the next commit | `git add -p <file>`
Commit the changes | `git commit`
Push the changes to server | `git push`
Fetch the latest changes but don't merge into HEAD | `git fetch`
Fetch the latest changes and merge into HEAD | `git pull`

__Git workflow__ is a bit different from SVN in some aspects:
* In Git, you need to `git add` all the files you want to commit vs. in SVN, when doing a commit, all changed files that are tracked under version control get added to the commit by default
* SVN does commit and push together, in git you can do several commits before pushing them to your repo
* After doing `git commit`, you cannot see the changes you've made in files anymore. This kind of relates to the previous point - it's like doing SVN commit and pushing your changes to your repo.

##History and publishing
Description                              |Command
-----------------------------------------|----------------------------------
Show commits over time                   | `git log`
Show commits over time for a specific file| `git log -p <file>`
Who changed what and when                | `git blame <file>`
Show all existing branches               | `git branch -av`
Create new branch                        | `git branch <branch name>`
Delete a branch                          | `git branch -d <branch name>`
