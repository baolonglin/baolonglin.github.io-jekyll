---
layout: post
title: "My first emacs package"
date: 2017-05-18
---

After using emacs for serval monthes, come out my first emacs package. It may not such useful, but it solves my daily work problem.

The package is stored at [switch-process-environment](https://github.com/baolonglin/switch-process-environment). It's quiet simple, just manages the emacs process environment, enables user switch the process environments.

In my daily work, I need to switch between different shell environments in one project. Different shell environments provides different toolset, and those environment variables are conflict with each other. Restart emacs and recover the desktop session cost time, different git commit can have different environment variables.

With that package, I can keep one emacs as daemon, don't need to restart the emacs to switch the different environment.
