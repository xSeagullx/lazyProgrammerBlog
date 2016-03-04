---
layout:     post
title:      "Surround selection in IntelliJ IDEA"
date:       2016-03-03 23:49:00
summary:    "This tiny post is all about one little feature of IDEA: surrounding selection with quotes or braces."

categories: intellij tips micropost
---

It's pretty simple:

1. You select text.
2. You type opening brace or quote.
3. You see your selection wrapped in that quote.

If your selection is already enclosed by quotes or braces, they will be replaced automatically.
It plays nice with IntelliJ's smart selection, and saves you from tedious routine job when you need to convert single quotes to quotes and vice-versa.

Here is the list of supported characters:

1. Quotes: `'` `"`, `` ` ``.
2. Braces: `(`, `[`, `{`, `<`.

Yes, backtick and angle-brackets too!

### Setup:
![setup](/assets/2016-03-03-intellij-surround-selection/img.png)

I have warned you!
