---
tags:
  - Git
  - tips
  - snippets
title: Git commands test
created: 2024-03-17
date: 2024-04-07
---
Show the diff between the current branch and the target branch and show file names only.
```shell
git diff --name-status tgt_branch
```

Replay the commits from the current branch onto the `tgt_branch`. In this case, the current branch is the "incoming branch"
```shell
git rebase tgt_branch
```

Fetch latest changes from the remote repo.
```shell
git fetch
```

Push local branch onto remote repo with the name `tgt_branch`
```shell
git push --set-upstream origin tgt_branch
```

Revert changes from the last commit into the working tree
```shell
git reset HEAD .
```

Force push 
```shell
git push -f origin local_branch:remote_branch
```