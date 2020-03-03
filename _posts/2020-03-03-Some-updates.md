---
layout: post
title: Some updates
---

So instead of writing about one thing, I'm going to write a little about multiple things.

In my [previous post](/2020/02/10/Passing-passwords-to-myself.html), I talked about running my own password manager with [pass](https://www.passwordstore.org/) and a private Git server. I promised code for the Git server and [here](https://github.com/zbo14/services/tree/master/services/git) it is! You'll notice the repo contains Dockerfiles, config files, and scripts for running other services (e.g. an Unbound DNS server that does upstream DNS-over-TLS). I thought it would be nice to put all (or most of) the Dockerized services I use into a repo. That way if I'm on a new machine, I can install Docker, clone the repo, and then run any service I want. I'm hoping to add more Dockerized services on a rolling basis!

In addition, I revisited my work on [DIY VPNs](https://zbo14.github.io/2019/11/13/DIY-VPNs.html) and decided to dig deeper into [WireGuard](https://www.wireguard.com/). I created another family of repos under the umbrella title, [wirevpn](/projects/wirevpn). Why create more repos and another VPN name? Well, [diy-vpn](/projects/diy-vpn) outlined minimal proofs-of-concept for [OpenVPN](https://openvpn.net/) and WireGuard VPNs. I wanted to create a project that focused on WireGuard and went beyond the proof-of-concept. This involved encrypting private keys on client and server boxes, using pre-shared keys for an extra layer of encryption, and forwarding client DNS queries from the VPN server upstream over TLS.

On the operational side of things, I deployed the `wirevpn` server on DigitalOcean and am using it with my phone and desktop. I even convinced my roommate to join the party and now he's using it with his phone! And all of this fun only costs me $10 a month!
