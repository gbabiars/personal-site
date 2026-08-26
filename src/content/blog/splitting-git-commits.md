---
title: "Splitting Git Commits"
description: "It is very important to keep a project's Git history clean. There are several huge advantages to this like making it easier to track down bugs and understand why technical…"
pubDate: "2015-05-04T00:00:00"
---

It is very important to keep a project's Git history clean. There are several huge advantages to this like making it easier to track down bugs and understand why technical decisions were made. To this end, it is often times needed to refine your working history before pushing it to remote.

Let's take the following example: You are working on some functionality and you make some changes to one area of the application. While in there, you notice something else and make a quick change. You commit all of this in one commit. But when going back and look at the commit, you realize they touch two different areas of the application and would make more sense as two distinct commits. Git provides us with an easy way to go back and edit this history.

To demo how this is done, we'll take a look at a simple history where we have two files (file1.txt and file2.txt) and three total commits. We will split the second commit into separate commits: one that touches file1.txt and one that touches file2.txt.

To start with our Git history looks like:

```
ebe053a Added line to file2
f2504cd Added line to file1 and added file2
3064113 Add file1.txt
```

We want to edit the second commit, `f2504cd` and split it into two commits. First we need to rebase.

```
git rebase -i HEAD~2
```

Then in the editor, we need to select `edit` for the first commit in the list.

```
edit f2504cd Added line to file1 and added file2
pick ebe053a Added line to file2
```

At this point, the rebase will be stopped with the last commit being the one we want to edit. If you want the existing commit message, it is a good idea here to use `git log` and copy the last message. We then need to reset our head to before the commit.

```
git reset HEAD~
```

This will keep the changes from the commit we want to modify in the working directory. Now we can commit each part individually.

```
git add file1.txt
git commit -m 'Added line to file1'
git add file2.txt
git commit -m 'Added file2.txt'
```

This will create two new commits rather than the original one. To finish off, we just need to complete the rebase.

```
git rebase --continue
```

Now if we look at our history, we have four commits instead of three:

```
a5b0a6b Added line to file2
3011d73 Added file2.txt
44e23d7 Added line to file1
3064113 Add file1.txt
```

This highlights how simple it is to split commits in Git using rebase and edit.
