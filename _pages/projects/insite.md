---
title: ""
permalink: /projects/insite/
layout: default
---

# insite <a href="https://github.com/zbo14/insite"><svg class="svg-icon" style="vertical-align:middle"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a>

A library and CLI for website crawling and enumeration. First, you tell `insite` to visit a URL and it finds more URLs under the same hostname. It visits those URLs and continues its search until it runs out of unvisited URLs or it hits a time limit. When it's done crawling, `insite` returns a result that contains the subdomains, paths, search parameters, and entry points found.
