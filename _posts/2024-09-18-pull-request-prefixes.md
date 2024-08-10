---
title: "Prefix Pull Requests to make them measurable"
categories: [ Code tips ]
tags: [ git, productivity ]
---

## Motivation:

The goal is to more effectively measure your performance and impact on a project by categorizing your work in a
consistent way.

## Background:

In the past, I used to prefix my commits and Pull Requests with a category or the relevant product feature.
Then, I discovered [Conventional Commits](https://www.conventionalcommits.org/), and I started applying that approach to
my commits.

## How to use it:

I will be using the following prefixes:

```
Improv:    # Improvements to an existing feature
Feat:      # New features
Fix:       # Bug fixes
Cleanup:   # Code cleanup or refactoring
Test:      # Test-related changes
```

If something is particularly important or impactful, I'll add a `!` to the prefix.  
For example:  
`Improv! comments: display commenter country`