---
layout: post
title: URLs can grow on trees
---

The past few months, I've been working on a [library](/projects/web-tree) that organizes URLs into a hierarchical tree structure. This is useful if you're exploring a website and you'd like a visual summary of your navigation/request history.

The top-level domain (e.g. ".com") is the root of the tree and each subdomain is a branch. Subdomains of a subdomain form branches off that branch, and so on and so forth. Each path under a domain forms a different kind of branch. The subpaths form branches off that branch, and so on and so forth. Following this recursive procedure, we end up with a tree containing all of our URLs.

After writing the first version of the library, I created a [Firefox add-on](https://addons.mozilla.org/en-US/firefox/addon/web-tree/) that uses it to construct a tree of URLs you've visited/sent requests to. The add-on builds the tree in the background as you navigate and your browser fires off requests.

There's a minimal UI in devtools where you can filter/show part of the tree under a particular domain or wipe the tree state clean. If you filtered for "foobar.com" with the first version of the add-on, it would spit out a text representation of the *entire* subtree under "foobar.com" (if any). This subtree could be pretty big/difficult to visually parse all at once. The newer version of the add-on renders an HTML tree. You can click a domain or path to expand/collapse the corresponding branch, so you can visually parse the tree in digestible chunks.

The UI still needs work, but I'm fairly happy with the proof-of-concept. The library is open-source so if you find a bug, would like a feature added, or would like to improve the UI (hint hint) let me know in an issue!
