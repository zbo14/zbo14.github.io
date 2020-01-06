---
layout: post
title: Detect and confirm desync vulnerabilities
---

I recently reread [this post](https://portswigger.net/research/http-desync-attacks-request-smuggling-reborn) on HTTP/S desync vulnerabilities. I like the way the author breaks down the methodology of addressing the vulnerability into multiple stages (i.e. detect, confirm, explore, exploit). It got me wondering whether I could write a barebones tool to automate at least some of these steps.

For those unfamiliar, an HTTP/S desync vulnerability arises when a frontend server (e.g. CDN) handles web requests before sending them to a backend server and the two servers disagree where the requests begin and end. As a result, the servers process different requests. This is problematic because an attacker could "smuggle" a malicious request past the frontend server that would normally be blocked and the backend server processes the request and responds.

How can an attacker get the frontend and backend servers to disagree on request boundaries? One way is to send an HTTP/S request with a [Content-Length](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Length) header *and* a [Transfer-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding) header. In some cases, the frontend refers to one header (e.g. Content-Length) and the backend refers to the other (Transfer-Encoding). The frontend server might see the following.

**Note:** CRLFs are omitted.

```
===========================
POST / HTTP/1.1
Host: www.foobar.com
Content-Length: 51
Transfer-Encoding: chunked

0

GET /secret HTTP/1.1
Host: www.foobar.com

===========================
```

But the backend server would see the following.

```
===========================
POST / HTTP/1.1
Host: www.foobar.com
Content-Length: 51
Transfer-Encoding: chunked

0

===========================
GET /secret HTTP/1.1
Host: www.foobar.com

===========================
```

When the frontend receives the byte stream it sees one request. When the backend receives the byte stream it sees *two* requests. The frontend normally responds to a request for `/secret` stuff with a 403. Since it doesn't see the request - it thinks it's part of the first request's body - it happily forwards the octets to the backend server. Suppose the backend server doesn't provide any protections for the `/secret` stuff. It happily fetches the sensitive content and responds to the second request.

Yes, this is a contrived example but hopefully it conveys the concept of desync vulnerabilities and how they can be exploited to smuggle HTTP/S requests. Reading through the aforementioned post, I noticed the process for confirming and detecting these vulnerabilities is fairly uniform. I created a [proof-of-concept](https://github.com/zbo14/desync) that uses desync trickery to perform these two steps (no exploitation (yet)). To detect a vulnerability we use request length discrepancy to force a timeout. Upon timeout, we confirm the vulnerability by smuggling a request to an endpoint we're fairly certain doesn't exist. A 404 response or redirect means the backend processed the smuggled request and is vulnerable.

The [readme](https://github.com/zbo14/desync/blob/master/README.md) has some more information. Otherwise, if you're interested check out the source. If you have an idea to make the tool better, let me know in an issue!
