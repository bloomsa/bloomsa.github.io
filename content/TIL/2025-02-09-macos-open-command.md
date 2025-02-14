---
title: macOS open command
description: ""
date: 2025-02-14T01:08:07.488Z
preview: ""
draft: false
tags:
    - command line
    - Finder
    - macOS
    - TIL
categories: []
fmContentType: til
---

macOS has a neat program for opening files, directories, and urls from the command line.

```
open file.ext
```

will open `file.ext` from your current directory in whatever default app you have configured for the given files' extension. For example `open paper.pdf` will open Preview or another PDF viewer and `open foobar.py` will open the editor you have configured for `.py` files (configurable in Finder).

It can also open directories for viewing in Finder with `open /path/to/my/dir` or even urls with `open https:example.com` in your default web browser. 