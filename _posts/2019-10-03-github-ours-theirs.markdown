---
layout: post
title:  "git - ours & theirs"
date:   2019-10-03 13:12:29 +0500
categories: git checkout
---
## git merge
Let's merge conflicting branch `feature` into `master`


```shell
$ git checkout master 
$ git merge feature 
Auto-merging Document 
CONFLICT (content): Merge conflict in codefile.js 
Automatic merge failed; fix conflicts and then commit the result.
```

either fix the conflict manually by editing codefile.js, or use

`$ git checkout --ours codefile.js` <br><br>to select the changes done in `master` | `$ git checkout --theirs codefile.js` <br><br>to select the changes done in `feature`

then, continue as you would normally merge

```shell
$ git add codefile.js 
$ git merge --continue 
[master 5d01884] Merge branch 'feature' 
```
<br>

## git rebase
Let's rebase conflicting branch `feature` over `master`

```
$ git checkout feature 
$ git rebase master 
First, rewinding head to replay your work on top of it... 
Applying: a commit done in branch feature 
error: Failed to merge in the changes. 
...
```

either fix the conflict manually by editing codefile.js, or use

`$ git checkout --ours codefile.js` <br><br>to select the changes done in master | `$ git checkout --theirs codefile.js` <br><br>to select the changes done in feature

then, continue as you would normally do

```
$ git add codefile.js 
$ git rebase --continue 
Applying: a commit done in branch feature
```
