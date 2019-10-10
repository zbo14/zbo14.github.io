---
title: ""
permalink: /projects/dnsdump/
layout: default
---

# dnsdump <a href="https://github.com/zbo14/dnsdump"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a> <a href="https://www.npmjs.com/package/dnsdump"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#npm' | relative_urll }}"></use></svg></a>

[DNS](https://en.wikipedia.org/wiki/Domain_Name_System) isn't only used to resolve hostnames to IP addresses. It also provides information about mail servers, name servers, aliases, etc. for a domain. Each of these corresponds to a specific type of DNS record. If you want all (or at least some of) the resource records for a domain, you can make a DNS "ANY" query. According to [RFC 8482](https://tools.ietf.org/html/rfc8482) however, DNS servers aren't required to respond with the desired records.

`dnsdump` fetches a bunch of DNS records for a domain without using the "ANY" query. It sends an asynchronous request for each record type and, once every request has completed, spits out the results in a legible format.
