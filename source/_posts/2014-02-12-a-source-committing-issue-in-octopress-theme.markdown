---
layout: post
title: "A source committing issue in oscailte theme"
date: 2014-02-12 11:30:54 +0800
comments: true
categories: 
---
I'm using the oscailte theme for Octopress. When I try
to commit the source files to the repository, I come 
across an error. I type `git add .`, and I get:


```
fatal: Not a git repository: sass/inuitcss/../../.git/modules/sass/inuitcss
```

After googling the web, I find the fix: change the `gitdir` in `sass/inuitcss/.git` to


``` plain fixed gitdir http://github.com/coogie/oscailte/issues/23 found at
gitdir: ../../.themes/oscailte/.git/modules/sass/inuitcss
```

Done!
