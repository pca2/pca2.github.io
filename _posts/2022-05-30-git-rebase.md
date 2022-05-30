---
title:  "Interactive Git Rebases: A quick and dirty guide"
date:   2022-05-30 15:04:23
tags: [Git, Version Control, rebases]
layout: post
---

Let's say you got the following commits:
```plaintext
A Latest
B Third
C Second
D First
```

And you want to edit the contents of commit C, maybe changing it completely. 

You can only rebase a clean repo, so stash any changes you want to hold on to or just `git checkout`.

1. Rebase starting at a commit that comes _before_ the commit you want to change. In this example, we'll target commit D:

```bash
git rebase -i D
```

You'll be presented with a list of the commits before D. Something like this:

```
pick C Second
pick B Third
pick A Latest

# Rebase c2e5a85..d5e63b8 onto c2e5a85 (2 commands)
```

The latest commit is at the **bottom**. I think that's confusing because typically you see the latest commit on the top.

2. Change `pick` to `e` or `edit` for the commit you want to change and save and quit your text editor.

3. Edit the file(s) as you desire. Here's where you can `git stash pop` any changes you had to stash. You may have to resolve some merge conflicts.

4. `git add` the updated file(s) to stage your changes.

5. `git commit --amend` to edit the old commit you are re-writing. Change the commit message to reflect the current changes.

6. `git rebase --continue` to complete.

If you need to rebase a remote branch you can add a 7th step of forcing a push:

`git push origin <branchname> -f`

### Warning: Git rebase rewrites history

Use caution when doing this type of rebase. It essentially rewrites history, which can have unintentional consequences, especially if other people are working on the same branch as you. 
