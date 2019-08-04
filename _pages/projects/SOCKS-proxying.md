---
title: ""
permalink: /projects/SOCKS-proxying/
layout: default
---
# SOCKS proxying

I wanted to run a SOCKS proxy on my local network by connecting/authenticating to an SSH daemon running somewhere else. Then I could tunnel my web traffic through SSH via the SOCKS proxy. The purpose of this project suite was first and foremost a learning exercise but also an attempt to make this process easier by automating certain steps with scripts and containerizing the services that need to run. Partway through, I realized some applications support HTTP proxies but not SOCKS proxies (who would've thunk). So I added an HTTP proxy component that translates CONNECT requests for a SOCKS proxy running on the same network. This way, clients can encrypt their web traffic even if they don't speak SOCKS.

## socksproxy <a href="https://github.com/zbo14/socksproxy"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

A Dockerized SOCKS proxy that tunnels your traffic over SSH.

## socksd <a href="https://github.com/zbo14/socksd"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

A Dockerized SSH daemon for `socksproxy`.

## http2socks <a href="https://github.com/zbo14/http2socks"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

A Dockerized HTTP proxy that routes traffic through `socksproxy`.
