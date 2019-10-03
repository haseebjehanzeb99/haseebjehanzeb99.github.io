---
layout: post
title:  "Using Vimdiff as mergetool"
date:   2019-10-03 13:29:29 +0500
categories: git vimdiff mergetool
---
## Git config
Prior to doing anything, you need to know how to set vimdiff as a git mergetool. That being said:

```
git config merge.tool vimdiff
git config merge.conflictstyle diff3
git config mergetool.prompt false
```

Let’s resolve the conflict:

![Vimdiff Window](/assets/images/three-way-merge-with-vimdiff.png "Vimdiff window")
This looks terrifying at first, but let me explain what is going on.

From left to right, top to the bottom:

LOCAL – this is file from the current branch BASE – common ancestor, how file looked before both changes REMOTE – file you are merging into your branch MERGED – merge result, this is what gets saved in the repo

Let’s assume that we want to keep the “octodog” change (from REMOTE). For that, move to the MERGED file (Ctrl + w, j), move your cursor to a merge conflict area and then:

```:diffget RE```

This gets the corresponding change from REMOTE and puts it in MERGED file. You can also:

```
:diffg RE  " get from REMOTE
:diffg BA  " get from BASE
:diffg LO  " get from LOCAL
```

You can also use `:diffget //2` or `:diffget //3` (the `//2` and `//3` are unique identifiers for the target/master copy and the merge/branch copy file names) to get the content from other window.

    :diffupdate (to remove leftover spacing issues)
    :only (once you’re done reviewing all conflicts, this shows only the middle/merged file)
    :e (refresh content windows)
    :wq (save and quit)
    git add .
    git commit -m “Merge resolved”

If you were trying to do a `git pull` when you ran into merge conflicts, type `git rebase –continue`.

### Vimdiff Commands

    ]c :        - next difference
    [c :        - previous difference
    do          - diff obtain
    dp          - diff put
    zo          - open folded text
    zc          - close folded text
    :diffupdate - re-scan the files for differences

In the middle file (future merged file), you can navigate between conflicts with `]c` and `[c`

Save the file and quit (a fast way to write and quit multiple files is `:wqa`).
