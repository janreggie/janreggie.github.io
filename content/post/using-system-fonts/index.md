---
title: "Using System Fonts and Adding Emoji"
type: post
date: 2021-06-20T16:34:44+08:00
publishdate: 2021-06-20
lastmod: 2021-06-20
draft: false
description: "In this blogpost I talk about falling back to using system fonts, as well as (incidentally) adding Emoji support"
tags:
  - "hugo"
  - "web development"
categories:
  - "blog"
---

The "System Font Stack" is a font stack which emphasizes on built-in fonts
in order to boost performance and use the "best" font available for a system.
[CSS-Tricks](https://css-tricks.com/snippets/css/system-font-stack/) has talked about it in detail.
I have decided to do the same in this blog as well as in future projects I'll be making.

<!--more-->

## Former configuration: Google Fonts and hosting

My previous font stack was as follows:

- titles: [Overlock by Dario Manuel Muhafara](https://fonts.google.com/specimen/Overlock).
  Hosted via Google Fonts.
- text body: [Kumbh Sans by Saurabh Sharma](https://fonts.google.com/specimen/Kumbh+Sans)
  Hosted likewise via Google Fonts.
- monospace: [Fantasque Sans Mono](https://github.com/belluzj/fantasque-sans)
  by [Jany Belluz (belluzj)](https://github.com/belluzj).
  Hosted manually in this repository.

These fonts contribute to the general "aesthetic" of this website.
But I don't really care about aesthetics, honestly.

## The new stack

This font stack is based on [this Meta Stackexchange post](https://meta.stackexchange.com/q/364048):

```css
body {
  font-family: 
    "Kumbh Sans", /* Desired font */
    system-ui, -apple-system, BlinkMacSystemFont, /* San Francisco on macOS and iOS */
    "Segoe UI", /* Windows */
    "Ubuntu", /* Ubuntu */
    "Roboto", "Noto Sans", "Droid Sans", /* Chrome OS and Android with fallbacks */
    sans-serif, /* The final fallback for rendering in sans-serif. */
    "Emoji"; /* Emoji support */
}

pre, code {
  font-family: 
    ui-monospace, /* San Francisco Mono on macOS and iOS */
    "Cascadia Mono", "Segoe UI Mono", /* Newer Windows monospace fonts that are optionally installed. Most likely to be rendered in Consolas */
    "Ubuntu Mono", /* Ubuntu */
    "Roboto Mono", /* Chrome OS and Android */
    Menlo, Monaco, Consolas, /* A few sensible system font choices */
    monospace; /* The final fallback for rendering in monospace. */
}
```

Note that I added a "Kumbh Sans" font at the start of `body $ font-family`.
This is so that it will be used *provided that* Kumbh Sans exists in the visitor's font library.
It's just a damn good font to use.

Note as well that I decided to deprecate "Fantasque Sans Mono",
not necessarily because I fell out of love with it.
I still use it in my editor.
Instead, I think that a lot of people might be put off by [ligatures, for good reason](https://practicaltypography.com/ligatures-in-programming-fonts-hell-no.html).

## So, emoji

At the very end of the `font-family` list, I added common fonts for emoji,
something which I never thought of until now.
And upon adding it, I was surprised that the :sun_behind_large_cloud: emoji for my previous blog post
has been rendered "properly". That's amazing :grin:

I'm currently using a Linux system so I don't know how this will look like in Windows, however...
I should check it when I reboot.
