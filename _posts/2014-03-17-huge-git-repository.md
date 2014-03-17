---
layout: post
title:  "Huge git repository, anyone?"
date:   2014-03-17 12:00:00
comments: true
categories: system
image: /images/007-git.png

---

Today I started working on existing Rails project; nothing realy special. I got access to git repository and everything that comes with it. After cloning repository to my local drive I found out that repository is bigger than 300MB.

So the challenge was to move repository into [GitHub](https://github.com), but before doing that reduce the size of it. This is what I did.

# Finding biggest files

The easies way to find files was with ```find``` command. This is how you find files bigger than 10MB on OSX.

```
find . -size +10MB -print
```

You can either remove this files; add them to ```.gitignore``` and instruct developers to download them separately.

# Cleaning Git repository

Git comes with handy "tool" called ```git-gc```. Command runs several housekeeping tasks within the current repository, such as compressing file revisions (to reduce disk space and increase performance) and removing unreachable objects.

```
git gc --aggressive --prune=now
```

Command options are

```
--aggressive
# [..] This option will cause
# git gc to more aggressively optimize the
# repository at the expense of
# taking much more time.

--prune=<date>
# [..] Prune loose objects
# older than specific date
```

# Removing everything?

After removing everything from repository that could be moved and running ```git-gc```; I found out that possibly the best way would be to just remove ```.git``` and start project again - just with plain code.

```
rm -rf .git
git init
```

This is how I trimmed a few MB from the project. :)

