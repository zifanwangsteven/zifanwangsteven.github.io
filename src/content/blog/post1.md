---
title: "For the last time, what is git rebase really?"
description: "The same article you've read 1000 times but never remembered"
pubDate: "Oct 16 2024"
heroImage: "/post_img.webp"
tags: ["dev-peripheral"]
---

If you are like me, you've looked up what `git rebase` does at least once a month and never seem to fully understand its might and glory; you only occasionally use the command when it's suggested by none other but git itself, when you try to push a foxtrot commit to a remote branch and got rejected. Then you realized you just recently opened your personal blog, and this seems like a light-hearted enough topic to be the first official post. So here goes.

### The 2 Modes of git rebase
People generally use git rebase for 2 purposes. (1) to clean up their local commit history and (2) to sync commits in their feature branch with a base branch (typically `main` or `master`). 


