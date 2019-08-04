---
title: ""
permalink: /projects/SSHing-over-reverse-SSH-tunnels/
layout: default
---

# SSHing over reverse SSH tunnels

## rtunnel <a href="https://github.com/zbo14/rtunnel"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

`rtunnel` contains two components, a host and a client.

The host, which is running behind a NAT let's say, opens a reverse tunnel to a publicly accessible `rtunneld` instance and starts an SSH daemon listening on the local end of the tunnel.

The client component, which is behind another NAT, authenticates with the `rtunneld` instance, requests port forwarding to the reverse tunnel, and SSHes into the `rtunnel` host over the local socket.

## rtunneld <a href="https://github.com/zbo14/rtunneld"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

This is the publicly-accessible SSH daemon that facilitates port-forwarding for `rtunnel` instances so they can open reverse tunnels/SSH into each other.
