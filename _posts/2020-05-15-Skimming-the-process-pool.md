---
layout: post
title: Skimming the process pool
---

I've been working on a few [bug bounty](https://en.wikipedia.org/wiki/Bug_bounty_program) programs recently. For a given program, there may be several assets or domains the company deems "fair game". Hoyouver, they might not all be *explicity* listed. Suppose the wildcard domain `*.foo.com` is in-scope but no subdomains are listed. This puts the onus on bug hunters to enumerate in-scope subdomains.

There are many tools for subdomain enumeration, including [amass](https://github.com/OWASP/Amass), [sublist3r](https://github.com/aboul3la/Sublist3r), and [knock](https://github.com/guelfoweb/knock). You can use one or more of these tools to compile a list of subdomains, but what next? You need to check whether these domain names actually resolve to IP addresses (or have CNAME records). Often times, subdomain enumeration tools return subdomains that no longer resolve (or never resolved) to an IP address. If you visit one of these domains in your browser you'll get a message along the lines of, "Cannot resolve hostname" or "Cannot find server".

You can resolve domain names at the command line with `dig` or `host`. If you have *a lot* of domain names, you can use an optimized tool like [massdns](https://github.com/blechschmidt/massdns). Sometimes the subdomain enumeration tool will actually try to resolve the domain names it discovers (e.g. `amass enum` does this by default).

Now you should have a list of resolvable domain names. Next you might want to scan these domains for web paths. Unsuprisingly, there are many tools to do this - [dirb](https://tools.kali.org/web-applications/dirb), [dirsearch](https://github.com/maurosoria/dirsearch), and [gobuster](https://github.com/OJ/gobuster) to name a few. Suppose you have 200 subdomains to scan. This might sound like a lot but it's not unrealistic for a relatively large company with a permissive scope. Scanning the subdomains in serial will take a while. You could scan each subdomain in its own OS process or its own thread if you're using a programming language/runtime with that concurrency model. Since you're leveraging other CLIs for enumerating subdomains and scanning web paths, it would be more convenient to script a solution.

You write a quick-and-dirty Bash script to enumerate subdomains and then scan each subdomain in its own process. You run it and it works! Then you run a quick `ps -a` at the command line and see *a lot* more processes than before. Ok, your computer is humming along fine and you're nowhere near the pid limit. Still, you find it unsettling that your script spawns a process for *each* subdomain it scans, a magnitude that's not configurable or known beforehand unless you're specifying the subdomains. Furthermore, the pid limit isn't the only factor here. Each process is opening sockets and making network requests. You probably won't hit the file descriptor limit, but there are still I/O concerns and issues with network throughput to consider.

Ideally, you'd specify the number of processes beforehand. Then each process would scan one subdomain and, after it's finished, scan the next (unscanned) subdomain. When a process finishes scanning a subdomain and there are no more subdomains to scan, it exits. When all processes have exited, you're done. How do you script this "pool" of processes? How do you implement a multiprocess queue of subdomains such that every subdomain is only scanned once?

Using process pools and multiprocess queues can be relatively simple. For instance, Python has a [multiprocessing package](https://docs.python.org/3.8/library/multiprocessing.html) with "Pool" and "Queue" objects. Rolling your own solution in Bash is a bit trickier, but I'll outline an approach here.

Suppose you want to kick off 20 processes. You can easily do this with a for-loop and the Bash ampersand operator:

>If a command is terminated by the control operator &, the shell executes the command in the background in a subshell. The shell does not wait for the command to finish, and the return status is 0.


```bash
#!/bin/bash

for _ in $(seq 1 20); do
  scanner & # dont wait!
done
```

Ok cool, you can easily spin up a bunch of process in our pool. But what is `scanner`?

```bash
#!/bin/bash

while true; do
  while true; do
    mkdir lock 2> /dev/null && break
    sleep 1."$RANDOM" # didn't get lock, let's wait for 1.? seconds
  done

  # acquired lock!

  subdomain="$(head -1 subdomains.txt)"
  subdomains="$(tail -n +2 subdomains.txt)"
  echo "$subdomains" > subdomains.txt

  rm -rf lock # release lock

  [[ "$subdomain" =~ [A-Za-z]+ ]] || exit

  scan "$subdomain"
done
```

This code doesn't look promising (evidence: nested while-loops with condition `true`!). First things first, `subdomains.txt` contains the subdomains you'd like to scan. The inner while-loop implements a locking mechanism such that one `scanner`  process is reading from/writing to `subdomains.txt` at a time. Remember you have 20 of these processes running! Earlier I posed the question:
> How do you implement a multiprocess queue of subdomains such that every subdomain is only scanned once?

The locking mechanism is here to help with that. The way it works is each process repeatedly tries to create the `lock/` directory. If the `lock/` directory already exists, the command fails with exit code 1. This means the process *won't* reach the `break` statement after the `&&` operator. **Note:** `2> /dev/null` redirects stderr to The Void™️ so you don't see the error message. It sleeps for a little bit and tries create the directory again. If the directory doesn't exist and it succeeds in creating the directory, it `break`s the loop. At this point, the other processes shouldn't be able to create the directory since *this* process already created it.

This process pulls the first subdomain from `subdomains.txt` using the `head` command. Then it gets the rest of the subdomains with the `tail` command and writes those to `subdomains.txt`. Next, it removes the `lock/` directory so another process can recreate it and pull the next subdomain from `subdomains.txt`. This way you don't have multiple processes reading and scanning the same subdomain!

If the subdomain doesn't have any letters, the process assumes there are no subdomains left and exits. Otherwise, it passes the subdomain as an argument to `scan`. Don't worry about the specifics of `scan` - it's just a command that takes a single subdomain as an argument, scans it for web paths, and writes the results somewhere. When a process finishes scanning a subdomain, it goes back to the beginning of the outer while-loop.

Not the most elegant solution, but it seems to work! [Here's](https://github.com/zbo14/dirscour) a more fleshed-out version that actually scans web paths. I'd be interested to see other approaches (I experimented with Unix named pipes but wasn't able to build a working POC). **Sidenote:** if you're building bug bounty tools or looking to collaborate, let me know at zmbalder<<at>>gmail<<dot>>com!
