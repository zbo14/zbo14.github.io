---
permalink: /projects/p1ng/
layout: default
---

# p1ng <a href="https://github.com/zbo14/p1ng"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

This is a little utility that sends a single ICMP echo request with some options. These include spoofing the source address at the IP layer and including payload data at the ICMP layer. Since the payload section can carry arbitrary data, ICMP is sometimes used to tunnel traffic for other protocols ([ptunnel](http://www.mit.edu/afs.new/sipb/user/golem/tmp/ptunnel-0.61.orig/web/)'s a good example). I didn't implement any tunneling here- just wanted to see if I could ping a host with my payload and have it echoed back.
