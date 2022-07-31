---
author:
  name: "Minhung Shih"
date: 2021-03-29
linktitle: Setting GOPATH
type:
- post
- posts
title: Setting GOPATH
weight: 10
series:
- Go Setup
---


### The GOPATH and PATH environment variables

The `GOPATH` environment variable specifies the location of your workspace. It defaults to a directory named `go` inside your home directory (`$HOME/go`).

If you really want to change your GOPATH to something else add GOPATH to your shell/bash/zsh initialization file `.bash_profile`, `.bashrc` or `.zshrc`.

```
export GOPATH=/something-else
```

Add `GOPATH/bin` directory to your `PATH` environment variable so you can run Go programs anywhere.

```
export PATH=$PATH:$(go env GOPATH)/bin
```

Make sure to re-source `source .bash_profile` your current session or simply open new tab within iTerm.



#go #path #setup