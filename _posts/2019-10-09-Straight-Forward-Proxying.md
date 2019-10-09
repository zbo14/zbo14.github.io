---
layout: post
title: Straight forward proxying
---

A couple weeks ago, I gave a presentation on forward proxies at my team's weekly meeting. I created a [repo](https://github.com/zbo14/forward-proxying) for it - if you think there's something I should add or change, let me know in an issue! For the presentation, I read the `README.md` aloud, adding an extra detail here and there. Then I ran the code samples, configured my browser to use different types of forward proxies, and ran `tcpdump` to see where browser traffic was going.

Overall, I felt like the presentation went pretty well. I presented to my team (6 people) so it wasn't much pressure but was still a helpful exercise. There were a few things I took away, both technical and non-technical.

One thing I didn't originally consider, and my boss pointed out, was that forward proxies can be used to block outgoing traffic to certain sites. I had considered the opposite use case, e.g. using a forward proxy to circumvent restrictive firewall rules or to access a particular site. So, giving a formal presentation can be a good way to solicit feedback and fill in the gaps you missed.

Another takeaway was to choose a more narrow topic in the future. I chose to cover HTTP/S proxies, HTTP tunneling proxies, and SOCKS proxies, which all fall under the umbrella of "forward proxies". Choosing a single topic instead of context switching from subtopic to subtopic might make it easier for you, as the presenter, to really get into the weeds and for the audience to ask questions/make comments.

The last takeaway was that I enjoyed the process of preparing and giving the presentation! It involved a nice mix of writing code and prose. And when you're required to explain a topic to someone else, rather than just understand it yourself, you come away with a portable comprehension. Portable in that you can convey it to others and they can clarify it back to you. Also portable in the sense that you're translating from code or unarticulated thoughts inside your head to spoken language.

I'm looking forward to the next presentation!
