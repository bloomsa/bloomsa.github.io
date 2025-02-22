---
title: Markdown reference-style links
description: ""
date: 2025-02-22T19:40:12.358Z
preview: ""
draft: false
tags:
    - hyperlinking
    - Markdown
    - TIL
categories: []
fmContentType: til
---

I recently learned about an alternative syntax for links in Markdown. The most basic syntax for adding a link to Markdown content is inline like `[Markdown links](https://www.markdownguide.org/basic-syntax/#links)` which renders as a hyperlink:
[This link is inline](https://www.markdownguide.org/basic-syntax/#links).

A cleaner syntax for links is reference-style. Both render the same, but reference-style keeps your paragraphs free from long urls. The url can be added at the bottom of your section or at the bottom of the page. [This is a reference link][1]
```markdown
[Markdown links][1]

[1]: https://www.markdownguide.org/basic-syntax/#links
```

[1]: https://www.markdownguide.org/basic-syntax/#links
