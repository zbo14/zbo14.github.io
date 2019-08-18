---
layout: post
title: SSHing over a tunnel to a reverse tunnel
---

You're running an SSH server on your raspi at home and you want to SSH into it from work. However, the raspi's behind a [NAT](https://en.wikipedia.org/wiki/Network_address_translation) router so it doesn't have a public IP address (i.e. it's not open to the world). You could forward a port to the raspi but you're reluctant to expose a port on your home router. You look at the [SSH manpage](https://linux.die.net/man/1/ssh) and see that SSH has `-L`ocal *and* `-R`emote port forwarding options. Hmmm...

You could do remote port forwarding and establish a reverse tunnel between the raspi and *something else*. That *something else* can then forward traffic to the raspi through the NAT router. Great, you'll just create a reverse tunnel to your work laptop! Except... you remember your office network is NATed as well. GAHH, your work laptop doesn't have a public IP either so your raspi has no immediate way of reaching out to it and saying, "Excuse me, I want a reverse SSH tunnel please".

Suddenly, you remember you have an instance languishing in the cloud. Languishing yes, but it has a public IP! That means the raspi (and your work laptop) can talk to it. Ok, so you create a reverse SSH tunnel between the raspi and the cloud instance. The cloud can talk to the raspi and your laptop can talk to the cloud. But how can we get the laptop to talk to the raspi? If only there was some way to forward the laptop's traffic over the reverse tunnel to the raspi...

This is where local port forwarding comes in. The laptop authenticates with the cloud and says, "I want you to forward my traffic to that port.. Ya know, the one for the reverse tunnel to the raspi?". The laptop allocates a socket so that when a connection is made and data is sent over it (to the cloud), the cloud forwards those octets through the reverse tunnel. In other words, the laptop establishes a forward tunnel to the reverse tunnel.

But what are these octets exactly? We're ultimately trying to SSH into the raspi. What payload should we send to do that? Once we establish the tunnel to the reverse tunnel, data traveling through is opaque to our Frankenstein tunnel. That means we can send any data we want and use any protocol. So how about SSH then? Yes, we can SSH over the SSH tunnel to the reverse SSH tunnel. The pretty diagram I drew below attempts to depict the scenario I've described.

<script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/8.2.3/mermaid.min.js"></script>

<div class="mermaid">
  graph LR
    subgraph cloud
    FORWARD_PORT
    REMOTE_PORT
    style REMOTE_PORT stroke-width:4px
    end
    subgraph raspi
    SOME_PORT1-->|1|REMOTE_PORT
    FORWARD_PORT-. 2 .-> SSH_PORT
    end
    subgraph laptop
    SOME_PORT2-->|3|REMOTE_PORT
    LOCAL_PORT-. 4 .-> FORWARD_PORT
    end
</div>

Couple things to note here. The `REMOTE_PORT` is emboldened because it's open to the world. Solid lines indicate actual connections (e.g. the laptop connecting to the cloud). Dotted lines indicate conceptual connections. The laptop doesn't establish a connection from its `LOCAL_PORT` to the cloud's `FORWARD_PORT`. It couldn't- the `FORWARD_PORT` isn't emboldened. However, traffic on the laptop's `LOCAL_PORT` travels through the SSH tunnel and is eventually forwarded to the cloud's `FORWARD_PORT`. But this can only happen after the laptop's established a solid line to the cloud.

I've been working on two projects, [rtunnel](https://github.com/zbo14/rtunnel) and [rtunneld](https://github.com/zbo14/rtunneld), which contain scripts and Dockerfiles to hopefully achieve the scenario above. The ensuing walkthrough uses these tools. You're welcome to install them and follow along (refer to the repos for install/usage instructions). Also, I totally encourage you to build your own solution or improve upon the one here!

First, you'll need to run a publicly-accessible `rtunneld` instance (e.g. on a cloud-hosting platform).

```bash
# on cloud
# assuming you've installed rtunneld and built the image
$ rtunneld init
$ rtunneld start 12345
$ rtunnel get-key
<cloud-pubkey>
```

This starts `rtunneld` on port 12345. You'll need to open this port to the world on your cloud instance. Still preferable to opening a port on your home router.

Then configure `rtunnel` on the device you ultimately want to SSH into. This was the raspi in the scenario but it could be another device.

```bash
# on raspi
# assuming you've installed rtunnel and built the image
$ rtunnel init
$ HOST="1.2.3.4" PORT=12345 PUBKEY=<cloud-pubkey> rtunnel add-host
$ rtunnel get-key
<raspi-pubkey>
```

`rtunnel` performs strict host-checking- i.e. hosts must be whitelisted, so we add the cloud's address, port, and public key with the `add-host` command.

Now we tell the cloud that the raspi is authorized to open reverse SSH tunnels.

```bash
# on cloud
# rtunneld is already running but that's ok!
$ rtunneld add-client <raspi-pubkey>
```

Let's start `rtunnel` on the raspi!

```bash
# on the raspi
# completes steps (1) and (2) in the diagram
$ HOST="1.2.3.4" FORWARD_PORT=12346 REMOTE_PORT=12345 rtunnel start
```

This command opens a reverse SSH tunnel from the raspi to the cloud. We specify the address of the cloud instance with the `HOST` parameter and its exposed port with the `REMOTE_PORT` parameter. Connections to `FORWARD_PORT` on the cloud will be "forwarded" to the reverse tunnel.

Let's check out the client side of things.

```bash
# on laptop
# assuming you've installed rtunnel
$ rtunnel init
$ HOST="1.2.3.4" PORT=12345 PUBKEY=<cloud-pubkey> rtunnel add-host
$ HOST="127.0.0.1" PORT=12347 PUBKEY=<raspi-pubkey> rtunnel add-host
$ rtunnel get-key
<laptop-pubkey>
```

Notice how we add two hosts- the cloud and the loopback address with the raspi's public key and a new port number. We'll see why shortly.

Before that, we tell the cloud and the raspi our laptop is an authorized client.

```bash
# on cloud
$ rtunneld add-client <laptop-pubkey>

# on raspi
$ rtunnel add-client <laptop-pubkey>
```

Ok, we've established the reverse SSH tunnel between the raspi and cloud and added the laptop as an authorized client. It looks like we're ready to SSH into the raspi!

```bash
# on laptop
# completes step (3) and (4) in diagram
$ HOST="1.2.3.4" FORWARD_PORT=12346 LOCAL_PORT=12347 REMOTE_PORT=12345 rtunnel ssh
```

Let's break this command down...

(1) We SSH into the cloud at `<HOST>:<REMOTE_PORT>`

(2) We request port forwarding from `LOCAL_PORT` on the laptop to `FORWARD_PORT` on the cloud. Remember, connections to `FORWARD_PORT` on the cloud are forwarded to the reverse tunnel (i.e. the raspi), which is what we want.

(3) We SSH into the raspi over the allocated socket on `LOCAL_PORT` (on the laptop). This is why we previously added a host with loopback address, `LOCAL_PORT`, and the raspi's public key.

You should now have a shell on the raspi (actually, on a container on the raspi)! Go wild.
