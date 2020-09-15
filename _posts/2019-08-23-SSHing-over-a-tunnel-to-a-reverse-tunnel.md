---
layout: post
title: SSHing over a tunnel to a reverse tunnel
---

You're running an SSH server on your rpi at home and you want to SSH into it from work. However, the rpi's behind a [NAT](https://en.wikipedia.org/wiki/Network_address_translation) router so it doesn't have a public IP address (i.e. it's not open to the world). You could forward a port to the rpi but you're reluctant to expose a port on your home router. You look at the [SSH manpage](https://linux.die.net/man/1/ssh) and see that SSH has `-L`ocal *and* `-R`emote port forwarding options. Hmmm...

You could do remote port forwarding and establish a reverse tunnel between the rpi and *something else*. That *something else* can then forward certain connections to the rpi through the NAT router. Great, you'll just create a reverse tunnel to your work laptop! Except... you remember your office network is NATed as well. GAHH, your work laptop doesn't have a public IP either so your rpi has no immediate way of reaching out to it and saying, "Excuse me, I want a reverse SSH tunnel please".

Suddenly, you remember you have an instance languishing in the cloud. Languishing yes, but it has a public IP! That means the rpi (and your work laptop) can talk to it. Ok, so you create a reverse SSH tunnel between the rpi and the cloud instance. The cloud can talk to the rpi and your laptop can talk to the cloud. But how can we get the laptop to talk to the rpi? If only there was some way to forward the laptop's traffic over the reverse tunnel to the rpi...

This is where local port forwarding comes in. The laptop authenticates with the cloud and tells it to forward traffic to the reverse tunnel. In other words, the laptop establishes a (forward) tunnel to the rpi's reverse tunnel. Then we can SSH into the rpi through our tunnel system. The pretty diagram I drew below somewhat depicts the scenario I've described.

<script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/8.8.0/mermaid.min.js"></script>

<div class="mermaid">
  graph LR
    subgraph cloud
    FPORT
    EPORT
    end
    subgraph rpi
    PORT1-->|1|EPORT
    FPORT-. 2 .-> 22
    end
    subgraph laptop
    PORT2-->|3|EPORT
    LPORT-. 4 .-> FPORT
    style cloud fill:gray,stroke:gray
    style rpi fill:gray,stroke:gray
    style laptop fill:gray,stroke:gray
    style EPORT stroke-width:4px
    linkStyle default stroke-width:2px,fill:none,stroke:#e6e6e6
    linkStyle 1 stroke-width:2px,fill:none,stroke:#e6e6e6,stroke-dasharray:3
    linkStyle 3 stroke-width:2px,fill:none,stroke:#e6e6e6,stroke-dasharray:3
    end
</div>

Couple things to note here: the exposed port `EPORT` is emboldened because it's `E`xposed to the world. Solid lines indicate actual connections (e.g. the laptop connecting to the cloud). Dotted lines indicate conceptual connections. The laptop doesn't establish a connection from its `LPORT` to the cloud's `FPORT`. It couldn't- `FPORT` isn't emboldened. However, traffic on the laptop's `LPORT` travels through the SSH tunnel (3), is sent to the cloud's `FPORT`, and is then `F`orwarded through the reverse tunnel. This can only happen after the laptop's established a solid line to the cloud.

This design might seem convoluted or more complex than it needs to be. Why does the laptop use local port forwarding after the rpi's opened the reverse tunnel? Can't we just SSH into the daemon and then SSH into the rpi? Well, assuming the daemon was authorized to SSH into the rpi, yes we could. But is that something we want? Suppose it is and the cloud instance is compromised. The attacker could SSH into the rpi and any other host that's connected to the daemon, assuming we don't unauthorize the daemon in time. In the diagram above, the rpi and laptop can SSH into the daemon but the daemon shouldn't be able to SSH into them. The daemon has one job: serve as a tunnel bridge between the laptop and rpi and other hosts/clients.

I threw together [some scripts and a Dockerfile](https://github.com/zbo14/rtunnel), which I'm collectively calling `rtunnel`. In the ensuing walkthrough, I'll recreate the scenario described at the beginning of this post. Familiarity with SSH certainly helps, but don't be dissuaded if you're new to it! You're welcome to install the tool and try it yourself (please refer to the repo for install/usage instructions). Also, I totally encourage you to build your own solution or improve upon the one here!

First, you'll need to run a publicly-accessible daemon. Create an account on your cloud-hosting platform of choice, spin up an instance (hopefully for free), and complete the necessary configuration steps (hopefully well-documented).

```bash
# on cloud (daemon)
# assuming you've installed rtunnel
$ rtunnel build
$ rtunnel init
$ rtunnel start 12345
```

This starts the daemon on port 12345. You'll need to open this port on your cloud instance. Still preferable to opening a port on your home router though. If your home/office networks have static public IPs, you should only allow traffic from those addresses instead of the entire world.

Then start an SSH server and configure `rtunnel` on the device you ultimately want to SSH into. In the scenario, this was the rpi but it could be another device. Change `PasswordAuthentication` to `no` in the SSH server configuration file (usually `/etc/ssh/sshd_config`) since we only want to allow clients with authorized keys.

```bash
# on rpi (host)
# assuming you've installed rtunnel
$ rtunnel init
$ cat ~/.ssh/rtunnel.pub
<rpi-pubkey>
```

Now we need to authorize the rpi on the cloud. We do this by adding the rpi's public key to the `authorized_keys` file at `<path-to-rtunnel>/home/rtunnel/.ssh/authorized_keys`. No need to restart the daemon, the rpi can now open a reverse tunnel!

```bash
# on rpi
# completes steps (1) and (2) in the diagram
$ ADDR="1.2.3.4" EPORT=12345 FPORT=12346 rtunnel open
```

We specify the address of the cloud with the `ADDR` parameter and its exposed port with the `EPORT` parameter. Connections to `FPORT` on the cloud will be forwarded to the reverse tunnel.

Let's check out the client side of things.

```bash
# on laptop (client)
# assuming you've installed rtunnel
$ rtunnel init
$ cat ~/.ssh/rtunnel.pub
<laptop-pubkey>
```

We need to authorize the laptop on the cloud *and* the rpi because it will SSH into both. First, let's add the laptop's public key to the cloud's `authorized_keys` file. We already did this with the rpi's public key, so do what you did before just with the laptop's public key. On the rpi, we have to find the specific user's `authorized_keys` file. Suppose we want to SSH into the rpi as user `pie`. In that case, we add the laptop's public key to `/home/pie/.ssh/authorized_keys` (create the file first if it's not there).

Ok, we've established the reverse tunnel and authorized the laptop. Time to SSH into the rpi!

```bash
# on laptop
# completes step (3) and (4) in diagram
$ ADDR="1.2.3.4" EPORT=12345 FPORT=12346 LPORT=12347 USER=pie rtunnel ssh
```

Let's break this command down...

(1) We SSH into the cloud at `<ADDR>:<EPORT>` as user `rtunnel`.

(2) We request port forwarding from `LPORT` on the laptop to `FPORT` on the cloud. Remember, connections to `FPORT` on the cloud are forwarded to the reverse tunnel (i.e. the rpi), which is what we want.

(3) We SSH into the rpi as user `pie` over the allocated socket on `LPORT`.

You should now have a shell on the rpi! Go wild.

##### *Shoutout to [ngriffiths21](https://medium.com/@ngriffiths21) for helping with this post!*
