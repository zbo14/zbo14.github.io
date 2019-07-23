---
title: "Projects"
permalink: /projects/
layout: default
---
# Projects

## [p1ng](https://github.com/zbo14/p1ng)

This is a little utility that sends a single ICMP echo request with some options. These include spoofing the source address at the IP layer and including payload data at the ICMP layer. Since the payload section can carry arbitrary data, ICMP is sometimes used to tunnel traffic for other protocols ([ptunnel](http://www.mit.edu/afs.new/sipb/user/golem/tmp/ptunnel-0.61.orig/web/)'s a good example). I didn't implement any tunneling here- just wanted to see if I could ping a host with my payload and have it echoed back.

## [passworld](https://github.com/zbo14/passworld)

The goal for this project was to make a proof-of-concept library/CLI that takes a filesystem path and password and encrypts/decrypts whatever's on the path.

## socks*

I wanted to run a SOCKS proxy on my local network by connecting/authenticating to an SSH daemon running somewhere else. Then I could tunnel my web traffic through SSH via the SOCKS proxy. The purpose of this project suite was first and foremost a learning exercise but also an attempt to make this process easier by automating certain steps with scripts and containerizing the services that need to run. Partway through, I realized some applications support HTTP proxies but not SOCKS proxies (who would've thunk). So I added an HTTP proxy component that translates CONNECT requests for a SOCKS proxy running on the same network. This way, clients can encrypt their web traffic even if they don't speak SOCKS.

### [socksd](https://github.com/zbo14/socksd)

A Dockerized SSH daemon for your SOCKS proxy.

### [socksproxy](https://github.com/zbo14/socksproxy)

A Dockerized SOCKS proxy that tunnels your traffic through SSH.

### [http2socks](https://github.com/zbo14/http2socks)

A Dockerized HTTP proxy that routes traffic through `socksproxy`.
