---
layout: post
title: SOCKS proxying
---

Suppose you're at a coffee shop with Wi-Fi and someone is watching your traffic. What do they see when you navigate to "reddit.com" in your browser?

Let's run `tcpdump` in terminal to get a look at the HTTPS traffic:

```
$ sudo tcpdump -nnSX dst port 443

13:52:21.530207 IP 10.12.117.160.50937 > 151.101.249.140.443: Flags [P.], seq 3497648863:3497648989, ack 2992415932, win 15996, options [nop,nop,TS val 828290264 ecr 2300058011], length 126
	0x0000:  18e8 2941 2140 8c85 903b c1d2 0800 4500  ..)A!@...;....E.
	0x0010:  00b2 0000 4000 4006 29a8 0a0c 75a0 9765  ....@.@.)...u..e
	0x0020:  f98c c6f9 01bb d079 e2df b25c a4bc 8018  .......y...\....
	0x0030:  3e7c cfde 0000 0101 080a 315e b4d8 8918  >|........1^....
	0x0040:  199b 1703 0300 2500 0000 0000 0000 b56f  ......%........o
	0x0050:  b542 2fb8 072f 4b2a 57e1 8b44 4565 d2da  .B/../K*W..DEe..
	0x0060:  c25e 8a22 8dee 603d 9d7a 758b 1703 0300  .^."..`=.zu.....
	0x0070:  2500 0000 0000 0000 b6cd aadf cdbb 2672  %.............&r
	0x0080:  e4f4 fb42 b870 f52c ef7d b94e 9b3d 3e9d  ...B.p.,.}.N.=>.
	0x0090:  3fb4 2f97 17b4 1703 0300 2500 0000 0000  ?./.......%.....
	0x00a0:  0000 b75d 1fad 6d02 bbb3 d242 1093 d579  ...]..m....B...y
	0x00b0:  3c85 3af4 fe73 47bd 2b78 5bfa 3181 ec3d  <.:..sG.+x[.1..=
```

On the first line, we see two IP addresses: "10.12.117.160" and "151.101.249.140". The former is my laptop's address on the local network, which I verified with `ifconfig`. The latter is an IP address for reddit (or rather Fastly, the content delivery network (CDN) reddit uses). We can tell it's HTTPS traffic (read: encrypted) since it's destined for reserved port 443. Below, we see this garbled mess:

```
..)A!@...;....E.
....@.@.)...u..e
.......y...\....
>|........1^....
......%........o
.B/../K*W..DEe..
.^."..`=.zu.....
%.............&r
...B.p.,.}.N.=>.
?./.......%.....
...]..m....B...y
<.:..sG.+x[.1..=
```

This is why HTTPS (HTTP over TLS) is so important and why you should be wary when visiting HTTP sites (read: unencrypted). TLS encrypts your traffic and prevents others on the network from seeing the cleartext data (e.g. your passwords). However, they *can* see the IP addresses your traffic is destined for. This by itself might seem like an invasion of privacy. If you're using a virtual private network (VPN), they wouldn't see the destination addresses; they'd see packets destined for the VPN. It's possible the VPN provider is watching where your packets are going. So are we just kicking the can down the road? Yes, but we'd hope the VPN provider is less inclined to watch our traffic/care where packets are going.

There's another thing we should consider. Your browser makes DNS requests to resolve hostnames like "reddit.com" to IP addresses. These requests aren't encrypted unless you're using DNS-over-HTTPS. Let's take a look at DNS traffic when we visit "foobar.com" in our browser.

```
$ sudo tcpdump -nnSX dst port 53

16:23:46.087037 IP 10.12.117.160.65320 > 10.12.115.1.53: 56794+ A? foobar.com. (28)
	0x0000:  18e8 2941 2140 8c85 903b c1d2 0800 4500  ..)A!@...;....E.
	0x0010:  0038 d646 0000 ff11 e8b4 0a0c 75a0 0a0c  .8.F........u...
	0x0020:  7301 ff28 0035 0024 0a07 ddda 0100 0001  s..(.5.$........
	0x0030:  0000 0000 0000 0666 6f6f 6261 7203 636f  .......foobar.co
	0x0040:  6d00 0001 0001
```

Once again, "10.12.117.160" is my laptop's address. The "10.12.115.1" address corresponds to the DNS server on the local network, and we're asking it to resolve "foobar.com". If your web traffic is going through a VPN but your cleartext DNS requests aren't, someone on the network could monitor them to see what sites you're visiting.

How can we prevent the coffee shop Wi-Fi snooper from seeing the destination addresses of our packets and the hostnames we're visiting? We could pay to use a VPN service or run our own VPN, but this is a lot of work. Another option, which I'll discuss in greater detail, is running a SOCKS v5 proxy on our local machine that tunnels our web traffic and DNS requests over SSH (read: encrypted) to a daemon running in the cloud. From the cloud, the DNS requests are sent unencrypted to some DNS server. Once again, kicking the can down the road but I'd like to think we're kicking it to a part of the road where fewer people are watching.

<script src="https://cdnjs.cloudflare.com/ajax/libs/mermaid/8.8.0/mermaid.min.js"></script>

<div class="mermaid">
  graph LR
		subgraph reddit
		443
		end
		subgraph cloud
		EPORT
		style EPORT stroke-width:4px
		EPORT-->|2|443
		end
    subgraph laptop
    * -->|1|EPORT
    LPORT-. 3 .-> 443
    style reddit fill:gray,stroke:gray
    style cloud fill:gray,stroke:gray
    style laptop fill:gray,stroke:gray
    linkStyle default stroke-width:2px,fill:none,stroke:#e6e6e6
    linkStyle 2 stroke-width:2px,fill:none,stroke:#e6e6e6,stroke-dasharray:3
    end
</div>

Alright, let's get into it. Install [socksproxy](https://github.com/zbo14/socksproxy) on your local machine. Since `socksproxy`'s a systemd service, it runs on Linux. At the time of writing, I've tested it on Ubuntu and Raspbian.

After that, install [socksd](https://github.com/zbo14/socksd) on a cloud instance (e.g. AWS, Digital Ocean). `socksd` is a systemd service too. At the time of writing, I've only tested it on Ubuntu. You'll need to authorize `socksproxy` by copy-and-pasting `socksproxy`'s public key `/root/.ssh/socksproxy.pub` on your local machine to `/home/socksproxy/.ssh/authorized_keys` on the cloud instance. Now we can start `socksd`!

```
$ sudo systemctl start socksd
```

Let's make sure it's up and running.

```
$ sudo systemctl status socksd
```

On your local machine, configure `socksproxy` to communicate with `socksd`. You'll need to change `HOST` in `/etc/socksproxy/socksproxy.conf` to the address or hostname of your cloud instance. Then you can start `socksproxy`!

```
$ sudo systemctl start socksproxy
```

And let's check the status.

```
$ sudo systemctl status socksproxy
```

Hopefully that's working for you. Now we need to point our browser at `socksproxy`. I'm using Firefox but if you're using a different browser you should be able to configure a SOCKS proxy in browser or system settings. In Firefox, click the hamburger menu at the top right and then click "Preferences". Scroll down to the bottom and click "Settings" under "Network Settings". Select "Manual proxy configuration" and enter `socksproxy`'s address ("127.0.0.1") and port (default is 17897) next to "SOCKS Host". Scroll down and check "Proxy DNS when using SOCKS v5".

You should be good to go! If you experience problems with `socksproxy`, you can inspect the logs to help troubleshoot.

```
$ sudo journalctl -u socksproxy
```

There may be issues on the `socksd` side so you can check the logs there too.

```
$ sudo journalctl -u socksd
```

Before we finish up, let's make sure our web traffic *and* DNS requests are sent encrypted to `socksd`.

```
$ sudo tcpdump -nnSX dst port <PORT>

13:10:02.703097 IP 192.168.1.16.57076 > <ADDRESS>.<PORT>: Flags [P.], seq 1414475942:1414476174, ack 2549269707, win 18494, options [nop,nop,TS val 3087440208 ecr 2665010525], length 232
	0x0000:  4510 011c 56d8 4000 4006 e87f c0a8 0110  E...V.@.@.......
	0x0010:  0354 3568 def4 45e8 544f 30a6 97f2 c4cb  .T5h..E.TO0.....
	0x0020:  8018 483e be57 0000 0101 080a b806 9950  ..H>.W.........P
	0x0030:  9ed8 d55d 1bb9 6aac 0016 ba88 609f 5f68  ...]..j.....`._h
	0x0040:  3ed8 8cb8 9c49 6c47 514d a5f3 9a46 8722  >....IlGQM...F."
	0x0050:  623b a80c d186 b698 2fa0 1eed bf3b 1a57  b;....../....;.W
	0x0060:  fd05 e2f6 eec4 b039 aba2 5838 c8d7 4e6c  .......9..X8..Nl
	0x0070:  9588 6422 53b4 af77 e82d 21b4 b1b0 a3e7  ..d"S..w.-!.....
	0x0080:  4fd1 8674 5f19 b8cb d8f9 c03e 03d1 bbff  O..t_......>....
	0x0090:  685c 24f5 9969 affa b5bf 1c10 fc96 0404  h\$..i..........
	0x00a0:  c48b 3d5b 2365 cbda bdab 7949 11c1 db77  ..=[#e....yI...w
	0x00b0:  0527 53d1 dc23 edb5 abcb 0103 dbdc 1f23  .'S..#.........#
	0x00c0:  fe21 114a 0bb9 bdb5 fcaf 8c87 6213 b69c  .!.J........b...
	0x00d0:  fd5b 9a7a eb31 8edd db94 c5ea fdd9 8d5b  .[.z.1.........[
	0x00e0:  697b db7b 6919 4926 8129 7a9c c24a 495e  i{.{i.I&.)z..JI^
	0x00f0:  ea33 c808 3b28 a26b 0bc3 2af1 0d25 cc37  .3..;(.k..*..%.7
	0x0100:  f380 3807 061a bf15 e847 62b1 419e e7f6  ..8......Gb.A...
	0x0110:  c0a8 5914 2df1 e438 360d 1a6e            ..Y.-..86..n
```

For this packet capture, I was on a different network and my private address was "192.168.1.16". I was proxying traffic to a cloud instance with address `<ADDRESS>` where `socksd` was listening on port `<PORT>`. I ran `sudo tcpdump -nnSX dst port 53` in my terminal and navigated to "foobaz.com" in my browser. The DNS query didn't appear in the packet dump like it previously did with "foobar.com"! However, I saw other DNS traffic to/from the local DNS server because I only enabled the proxy in browser settings. I could've configured a SOCKS proxy in system settings, but there's no option to proxy DNS requests over SOCKS v5 that way. So we still have DNS traffic going through the local DNS server, but it looks like all web traffic and DNS requests *from the browser* are going through our proxy. That's good enough for me for now.
