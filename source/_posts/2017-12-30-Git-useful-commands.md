---
layout: post
title: Git useful commands
author: Auxcoder
tags:
  - Git
category:
  - Misc
  - Tools
date: 2017-12-30 16:40:12
---

## Create a new repository on the command line

```
echo "# borrar" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/<user>/<repo>.git
git push -u origin master
```

## Forking from GitHub to Bitbucket

### Setup remotes

Setup local repo

```sh
mkdir myrepository
cd myrepository
git init
```

Add bitbucket remote as "origin"

```sh
git remote add origin ssh://git@bitbucket.org/aleemb/myrepository.git
```

Add github remote as "sync"

```sh
git remote add sync https://github.com/aleemb/laravel.git
```

Check remotes

```sh
git remote -v
# origin	git@bitbucket.org/<user>/<repo>.git (fetch)
# origin	git@bitbucket.org/<user>/<repo>.git (push)
# sync	https://github.com/<user>/<repo>.git (fetch)
# sync	https://github.com/<user>/<repo>.git (push)
```

### Setup branches

First pull from github using the "sync" remote

```sh
git pull sync
```

Setup local "github" branch to track "sync" remote's "master" branch

```sh
git branch --track github sync/master
```

Switch to the new branch

```sh
git checkout github
```

Create new master branched out of github branch

```sh
git checkout -b master
```

Push local "master" branch to "origin" remote (bitbucket)

```sh
git push -u origin master
```

---

## Check for submodule

Git still think mysubmodule is a submodule, because it is recorded in the index with a special mode "160000".

```sh
git ls-tree HEAD my/module
```

## Remove submodule

```sh
git rm --cached my/module
```

## Remove `index.lock`

When git thow the error: _fatal: Unable to create '/path/to/project/.git/index.lock': File exists.index.lock': File exists._

```sh
rm -f ./.git/index.lock
```

## Fix error

When git thow the error: `fatal: refusing to merge unrelated histories`

```sh
git pull origin branchname --allow-unrelated-histories
```

### Refs

- [Forking from GitHub to Bitbucket](https://stackoverflow.com/questions/8137997/forking-from-github-to-bitbucket)
