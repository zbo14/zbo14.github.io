---
title: ""
permalink: /projects/recrypt/
layout: default
---

# recrypt <a href="https://github.com/zbo14/recrypt"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

When I was working on [passworld](/projects/passworld), I wondered if I could achieve the same objective with shell scripts in less time/code. [GPG](https://www.gnupg.org/) offers symmetric encryption with a password, but you receive an error message if you try to encrypt a directory. To support this use case, I check if the filesystem path points to a directory and, if so, `zip` the directory and then encrypt the .zip file. Even though `recrypt` leverages existing tooling and is more lightweight, `passworld` is nice in that it encrypts twice using two different ciphers.
