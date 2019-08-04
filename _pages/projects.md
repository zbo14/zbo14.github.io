---
title: "Projects"
permalink: /projects/
layout: default
---
# Projects

## p1ng <a href="https://github.com/zbo14/p1ng"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

This is a little utility that sends a single ICMP echo request with some options. These include spoofing the source address at the IP layer and including payload data at the ICMP layer. Since the payload section can carry arbitrary data, ICMP is sometimes used to tunnel traffic for other protocols ([ptunnel](http://www.mit.edu/afs.new/sipb/user/golem/tmp/ptunnel-0.61.orig/web/)'s a good example). I didn't implement any tunneling here- just wanted to see if I could ping a host with my payload and have it echoed back.

## passworld <a href="https://github.com/zbo14/passworld"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

The goal for this project was to make a proof-of-concept library/CLI that takes a filesystem path and password and encrypts/decrypts whatever's on the path.

## SSH-ing over reverse SSH tunnels

### rtunnel <a href="https://github.com/zbo14/rtunnel"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

`rtunnel` contains two components, a host and a client.

The host, which is running behind a NAT let's say, opens a reverse tunnel to a publicly accessible host (i.e. `rtunneld` instance) and starts an SSH daemon listening on the local end of the tunnel.

The client component, which is behind another NAT, authenticates with the publicly accessible host, requests port forwarding to the reverse tunnel, and SSH-es into the `rtunnel` host over the local socket.

### rtunneld <a href="https://github.com/zbo14/rtunneld"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

This is the publicly-accessible SSH daemon that facilitates port-forwarding for `rtunnel` instances so they can open reverse tunnels and SSH into each other.

## SOCKS proxying

I wanted to run a SOCKS proxy on my local network by connecting/authenticating to an SSH daemon running somewhere else. Then I could tunnel my web traffic through SSH via the SOCKS proxy. The purpose of this project suite was first and foremost a learning exercise but also an attempt to make this process easier by automating certain steps with scripts and containerizing the services that need to run. Partway through, I realized some applications support HTTP proxies but not SOCKS proxies (who would've thunk). So I added an HTTP proxy component that translates CONNECT requests for a SOCKS proxy running on the same network. This way, clients can encrypt their web traffic even if they don't speak SOCKS.

### socksd <a href="https://github.com/zbo14/socksd"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

A Dockerized SSH daemon for your SOCKS proxy.

### socksproxy <a href="https://github.com/zbo14/socksproxy"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

A Dockerized SOCKS proxy that tunnels your traffic through SSH.

### http2socks <a href="https://github.com/zbo14/http2socks"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

A Dockerized HTTP proxy that routes traffic through `socksproxy`.
