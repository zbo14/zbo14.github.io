---
layout: post
title: Some updates
---

Decided to write about a bunch of things in this post!

In my [previous post](/2020/02/10/Passing-passwords-to-myself.html), I talked about running my own password manager with [pass](https://www.passwordstore.org/) and a private Git server. I promised code for the Git server and [here](https://github.com/zbo14/services/tree/master/services/git) it is! You'll notice the repo contains Dockerfiles, config files, and scripts for running other services (e.g. an Unbound DNS server that does upstream DNS-over-TLS). I thought it would be nice to put all (or most of) the Dockerized services I use into a repo. That way if I'm on a new machine, I can install Docker, clone the repo, and then run any service I want. I'm hoping to add more Dockerized services on an ongoing basis!

In addition, I revisited my work on [DIY VPNs](https://zbo14.github.io/2019/11/13/DIY-VPNs.html) and decided to dig deeper into [WireGuard](https://www.wireguard.com/). I created another family of repos under the umbrella title, [wirevpn](/projects/wirevpn). Why create more repos and another VPN name? Well, [diy-vpn](/projects/diy-vpn) outlined minimal proofs-of-concept for [OpenVPN](https://openvpn.net/) and WireGuard VPNs. I wanted to create a project that focused on WireGuard and went beyond the proof-of-concept. This involved encrypting private keys on client and server boxes, using pre-shared keys for an extra layer of encryption, and forwarding client DNS queries from the VPN server upstream over TLS.

On the operational side of things, I deployed the `wirevpn` server on DigitalOcean and am using it with my phone and desktop. I even convinced my roommate to join the party and now he's using it with his phone! All of this fun only costs me $10 a month!

Lastly, I created an [npm project template](https://github.com/zbo14/npm-project) using [GitHub templates](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-template-repository). I found that I was repeating the same steps every time I started a Node project. These included setting up the directory structure, creating config files, modifying the `package.json`, and specifying development dependencies. Now I can create a repo from this template and quickly dive in to the code! I might build out the template some more and make it user agnostic so others can easily use it. Anyway, if you find yourself repeating the same steps during project setup, GitHub templates could save you from the tedium.
