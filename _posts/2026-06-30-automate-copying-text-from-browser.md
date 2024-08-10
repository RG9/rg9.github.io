---
categories:
- tooling
devto_id: 4120577
tags:
- tampermonkey
title: Automate copying text from web browser using Bookmarklet or Tampermonkey
---

We all have those tiny repetitive tasks that take only a few seconds but somehow add up over time.

For me, one of them is **copying GitHub issues into my Obsidian notes**. I usually want the following Markdown format:

```markdown
[Issue title](https://github.com/org/repo/issues/123)
```

Doing this manually means selecting the title, copying it, copying the URL, and formatting everything. Not difficult—just repetitive.

## Option 1: Bookmarklet

A bookmarklet is simply a browser bookmark that **runs JavaScript instead of opening a webpage**.

Simply select from main menu: `Bookmarks -> Manage Bookmarks (Ctrl+Shit+O)`. 
Then right click on `Bookmars Toolbar` and add new bookmark titled, e.g. "Copy to Obsidian", in URL paste:

```javascript
javascript:(async()=>{
  const title = document.querySelector('bdi')?.textContent?.trim()
             || document.title.replace(/ · .*$/, '');

  await navigator.clipboard.writeText(
    `[${title}](${location.href})`
  );
})();
```

 Click the bookmark in toolbar while viewing a GitHub issue or pull request, and the formatted Markdown link is immediately copied to your clipboard.

The nice thing is that it works even for private repositories because it operates on the page you've already opened.

## Option 2: Tampermonkey

Using Tampermonkey, you can bind the action to a keyboard shortcut.

Here is full script that works for me:

```javascript
// ==UserScript==
// @name         GitHub -> Obsidian Task
// @namespace    obsidian
// @version      1.0
// @match        https://github.com/*/*/issues/*
// @match        https://github.com/*/*/pull/*
// @grant        GM_setClipboard
// ==/UserScript==

(function () {
    'use strict';

    function getTitle() {
        return document.querySelector("bdi")?.textContent.trim();
    }

    function copyTask() {
        const title = getTitle();
        const url = location.href;

        if (!title) {
            alert("Couldn't find issue title");
            return;
        }

        GM_setClipboard(`[${title}](${url})`);
    }

    document.addEventListener("keydown", e => {
        if (e.ctrlKey && e.altKey && e.key.toLowerCase() === "c") {
            e.preventDefault();
            copyTask();
        }
    });
})();
```

Now pressing `Ctrl+Alt+C` copies exactly what I need.

## Why I Prefer Tampermonkey

I like bookmarklets because they're simple and require no extensions.

However, I prefer Tampermonkey for small personal automations because:

- scripts are easier to organize and maintain,
    
- they can be limited to specific websites,
    
- adding shortcuts or UI elements is straightforward,
    
- they can grow with your workflow without becoming browser bookmark clutter.
    

One thing I particularly appreciate is that **Tampermonkey is a Recommended Extension in Firefox**. Mozilla reviews recommended extensions for security, code quality, and privacy, which gives me more confidence than installing a random browser add-on.