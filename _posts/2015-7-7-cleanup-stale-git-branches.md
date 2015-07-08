---
layout: post
title: Clean Unused Git Branches 
---

Git is a powerful tool to help developers work confidently and try
different approaches without losing any of it.

Here is a Git workflow borrowed from [here](http://nvie.com/posts/a-successful-git-branching-model/)
<img src="/images/git-model.png" alt='Git Workflow'>

Its apparent that keeping things handy can lead to lot of git branches
which need to be cleaned up once proved your research and executed.

Here is a script which is extremely apt [script](https://gist.github.com/antonio/4586456)

```bash
#!/bin/sh

echo "Delete branches having last commit before 1st June 2015 -
DRY_RUN=1 to show commands - ignore if execution"
echo "DRY_RUN=1 delete_branches_older_than.sh '2015-06-01' 'pod_rev'"

echo $DRY_RUN
echo $1

date=$1
sprint_prefix = $2

for branch in $(git branch -a | sed 's/^[ ]*//' | sed 's/^remotes\///' | grep -v 'master$'); do
  ## Addition to check a prefix in remote branches while deleting from remote
  if [[ "branch" =~ "sprint_prefix/" ]]; then
    if [[ "$(git log $branch --since $date | wc -l)" -eq 0 ]]; then
      if [[ "$branch" =~ "origin/" ]]; then
        local_branch_name=$(echo "$branch" | sed 's/^origin\///')
        if [[ "$DRY_RUN" -eq 1 ]]; then
          echo "git push origin :$local_branch_name"
        else
          git push origin :$local_branch_name
        fi
      else
        if [[ "$DRY_RUN" -eq 1 ]]; then
          echo "git branch -D $branch"
        else
          git branch -D $branch
        fi
      fi
    fi
  else
    echo "$branch NOT SCOPED"
  fi
done
```

## Walkthrough

- DRY_RUN = 1 can be passed to show commands instead of executing them
- Pass *date* after which commits would be looked in branches
  - Ignore the branch if any commit done after it
- Accepts *sprint_prefix* to clean only branches related to it 
- Iterate over all branches
- Remove whitespace at start of branch name
- Remove *remotes/* string at start of branch name
- Skip `master` branch
- Clean Git branches related to last completed hotfix / feature
  - It's advisable to add a prefix based on sprint/epic to all features
    developed for it
  - Look for decided prefix and follow further steps
- Check, whether any commit done *SINCE* given date
  - Ignore the branch if any commit done after it
- Perform `git push remote <branch_name>` if contains `origin` i.e. remote branch
- Perform `git branch -D <branch_name>` if local branch
- print the `command` if DRY_RUN is set as 1, else execute


## Remote Branch Reference Pruning
- git gc --prune=now
- git remote prune <branch_name>
- git fetch -p


### Useful links

- [Git housekeeping
  tutorial](http://railsware.com/blog/2014/08/11/git-housekeeping-tutorial-clean-up-outdated-branches-in-local-and-remote-repositories/)
