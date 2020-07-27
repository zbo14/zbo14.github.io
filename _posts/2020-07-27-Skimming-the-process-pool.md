---
layout: post
title: Skimming the process pool
---

Say you have a lot of independent tasks to complete. You don't have to finish one before starting the next, and you'd like to parallelize them to speed up collective execution. This means running the tasks simultaneously in separate processes. How many processes do you spin up?

If you have a hundred tasks, you could run each task in its own OS process. If you have millions of tasks though, you won't be able to create this many processes (you'll hit the PID limit well before). Even if the number of tasks is within the PID limit, and supposing you don't hit other limits (e.g. file descriptors), spinning up hundreds of processes could consume a lot of CPU or make a lot of network requests and eat up your bandwidth.

A better option would be to create a "pool" with a fixed number of processes. Each process would run a task and, when it's finished, begin the next task that hasn't been claimed by another process. This way, you get the benefits of parallelization without exhausting resources on your machine or the network.

Fortunately, this is trivial to achieve with the UNIX command, [`xargs`](https://man7.org/linux/man-pages/man1/xargs.1.html). Suppose you have a `file` with a thousand lines where each line is a task input. The command, `do-stuff`, runs the task in question. The following snippet divides the tasks among 10 processess, passing each line in the file as the input to `do-stuff`:

```bash
xargs -a "$file" -n 1 -P 10 do-stuff
```

What if `do-stuff` doesn't exist and you'd like to write your own shell code that performs arbitary processing on each line in the file? The following snippet achieves this:

```bash
xargs -a "$file" -n 1 -P 10 bash -c $'
  # your shell code that does arbitrary processing
  #
  # example - prepends each line with "foo" and prints result:
  echo "foo:$0"'
```
