---
layout: post
title: "Notes for Homebrew"
categories: macintosh apple homebrew
published: true
---
Notes for working with the [Homebrew](https://brew.sh/) package manager
for Macintosh.

#### To remove everything previously installed with homebrew

```sh
brew remove --force $(brew list --formula)
brew remove --cask --force $(brew list)
```

