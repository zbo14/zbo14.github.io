---
layout: post
title: URLs can grow on trees
---

The past few months, I've been working on a [library](/projects/web-tree) that organizes URLs into a hierarchical tree structure. This is useful if you're exploring a website and you'd like a visual summary of your navigation/request history.

The top-level domain (e.g. ".com") is the root of the tree and each subdomain is a branch. Subdomains of a subdomain form branches off that branch, and so on and so forth. Each path under a domain forms a different kind of branch. The subpaths form branches off that branch, and so on and so forth. Following this recursive procedure, we end up with a tree containing all of our URLs.

After writing the first version of the library, I created a [Firefox add-on](https://addons.mozilla.org/en-US/firefox/addon/web-tree/) that uses it to construct a tree of URLs you've visited/sent requests to. The add-on builds the tree in the background as you navigate and your browser fires off requests.

There's a minimal UI in devtools where you can filter/show part of the tree under a particular domain or wipe the tree state clean. If you filtered for "foobar.com" with the first version of the add-on, it would spit out a text representation of the *entire* subtree under "foobar.com" (if any). This subtree could be pretty big/difficult to visually parse all at once. The newer version of the add-on renders an HTML tree. You can click a domain or path to expand/collapse the corresponding branch, so you can visually parse the tree in digestible chunks.

The UI still needs work, but I'm fairly happy with the proof-of-concept. The library is open-source so if you find a bug, would like a feature added, or would like to improve the UI (hint hint) let me know in an issue! Below's a URL tree I generated for this website. 

<style>
  .web-tree-btn {
    background-color: #eee;
    color: #444;
    cursor: pointer;
    padding: 4px;
    width: 100%;
    border: none;
    text-align: left;
    outline: none;
    font-size: 12px;
  }

  .active, .web-tree-btn:hover {
    background-color: #ccc;
  }

  .web-tree-div {
    padding: 0 4px;
    display: none;
    overflow: hidden;
    background-color: #f1f1f1;
  }
</style>

<script>
  const btns = document.getElementsByClassName('web-tree-btn')

  for (const btn of btns) {
    btn.addEventListener('click', () => {
      btn.classList.toggle('active')

      const div = btn.nextElementSibling

      if (!div.style.display || div.style.display === 'none') {
        div.style.display = 'block'
        return
      }

      div.style.display = 'none'

      const btns = div.querySelectorAll('.web-tree-btn')
      const divs = div.querySelectorAll('.web-tree-div')

      for (const btn of btns) {
        btn.classList.remove('active')
      }

      for (const div of divs) {
        div.style.display = 'none'
      }
    })
  }
</script>

<button class="web-tree-btn">.io</button>
<div class="web-tree-div">
  <button class="web-tree-btn">.github</button>
  <div class="web-tree-div">
    <button class="web-tree-btn">.zbo14</button>
    <div class="web-tree-div">
      <button class="web-tree-btn">/2019</button>
      <div class="web-tree-div">
        <button class="web-tree-btn">/08</button>
        <div class="web-tree-div">
          <button class="web-tree-btn">/23</button>
          <div class="web-tree-div">
            <button class="web-tree-btn">/SSHing-over-a-tunnel-to-a-reverse-tunnel</button>
            <div class="web-tree-div">
            </div>
            <button class="web-tree-btn">/SSHing-over-a-tunnel-to-a-reverse-tunnel.html</button>
            <div class="web-tree-div">
            </div>
          </div>
        </div>
        <button class="web-tree-btn">/10</button>
        <div class="web-tree-div">
          <button class="web-tree-btn">/09</button>
          <div class="web-tree-div">
            <button class="web-tree-btn">/Straight-Forward-Proxying</button>
            <div class="web-tree-div">
            </div>
            <button class="web-tree-btn">/Straight-Forward-Proxying.html</button>
            <div class="web-tree-div">
            </div>
          </div>
          <button class="web-tree-btn">/11</button>
          <div class="web-tree-div">
            <button class="web-tree-btn">/SOCKS-proxying</button>
            <div class="web-tree-div">
            </div>
            <button class="web-tree-btn">/SOCKS-proxying.html</button>
            <div class="web-tree-div">
            </div>
          </div>
        </div>
        <button class="web-tree-btn">/11</button>
        <div class="web-tree-div">
          <button class="web-tree-btn">/13</button>
          <div class="web-tree-div">
            <button class="web-tree-btn">/DIY-VPNs</button>
            <div class="web-tree-div">
            </div>
            <button class="web-tree-btn">/DIY-VPNs.html</button>
            <div class="web-tree-div">
            </div>
          </div>
          <button class="web-tree-btn">/14</button>
          <div class="web-tree-div">
            <button class="web-tree-btn">/URLs-can-grow-on-trees</button>
            <div class="web-tree-div">
            </div>
            <button class="web-tree-btn">/URLs-can-grow-on-trees.html</button>
            <div class="web-tree-div">
            </div>
          </div>
        </div>
      </div>
      <button class="web-tree-btn">/about</button>
      <div class="web-tree-div">
      </div>
      <button class="web-tree-btn">/assets</button>
      <div class="web-tree-div">
        <button class="web-tree-btn">/audio</button>
        <div class="web-tree-div">
          <button class="web-tree-btn">/I'll-Try-Anything-Once.m4a</button>
          <div class="web-tree-div">
          </div>
          <button class="web-tree-btn">/I-Know-It's-Over.m4a</button>
          <div class="web-tree-div">
          </div>
          <button class="web-tree-btn">/Time-Again.m4a</button>
          <div class="web-tree-div">
          </div>
          <button class="web-tree-btn">/Untitled.m4a</button>
          <div class="web-tree-div">
          </div>
        </div>
        <button class="web-tree-btn">/main.css</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/minima-social-icons.svg</button>
        <div class="web-tree-div">
          <p>#github</p>
          <p>#linkedin</p>
          <p>#mozilla</p>
          <p>#npm</p>
        </div>
      </div>
      <button class="web-tree-btn">/feed.xml</button>
      <div class="web-tree-div">
      </div>
      <button class="web-tree-btn">/memos</button>
      <div class="web-tree-div">
      </div>
      <button class="web-tree-btn">/projects</button>
      <div class="web-tree-div">
        <button class="web-tree-btn">/bencode-hs</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/diy-vpn</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/dnsdump</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/genalg</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/nerv</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/p1ng</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/passworld</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/rad-tree</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/recrypt</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/socks</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/this-website</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/tidepool</button>
        <div class="web-tree-div">
        </div>
        <button class="web-tree-btn">/web-tree</button>
        <div class="web-tree-div">
        </div>
      </div>
    </div>
  </div>
</div>
