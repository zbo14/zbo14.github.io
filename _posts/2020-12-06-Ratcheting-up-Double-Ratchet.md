---
layout: post
title: Ratcheting up Double Ratchet
---

When it comes to secure, user-friendly messaging apps, it doesn't get much better than [Signal](https://signal.org/docs/). Nowadays, a whole host of messaging apps (e.g. iMessage, Facebook Messenger, WhatsApp) use end-to-end encryption as a default.

Signal (or more specifically the [Signal Protocol](https://en.wikipedia.org/wiki/Signal_Protocol)) was the first to employ end-to-end encryption via the [Double Ratchet Algorithm](https://en.wikipedia.org/wiki/Double_Ratchet_Algorithm). In fact, the Signal Protocol designers wrote the [specification](https://signal.org/docs/specifications/doubleratchet/) for Double Ratchet. This algorithm is important and used by many other messaging apps because it provides three critical properties:
* End-to-end encryption
* Forward security
* Break-in recovery

Before we dig into each of these properties, let's start with a definition of Double Ratchet. [The introduction](https://signal.org/docs/specifications/doubleratchet/#double-ratchet-1) in the spec can help us here.

> The Double Ratchet algorithm is used by two parties to exchange encrypted messages based on a shared secret key.

The two parties communicating, let's call them Alice and Bob, somehow agree on a shared secret key. This key is used to encrypt/decrypt messages they send to/receive from one another. Nobody else should know the secret key, even a server relaying traffic between Alice and Bob. Therefore, Alice and Bob are the only ones who should be able read the plaintext, or decrypted contents, the other sends. This satsifies the first property, "end-to-end encryption".

Unfortunately, there exist attackers who will try to snoop on or interfere with Alice and Bob's conversation. Most of their attempts will fail, but a few might succeed. We don't know which attempts will succeed or why they'll succeed (at least, not immediately). This depends on several varaibles, including network conditions and the particular devices Alice and Bob are using. Thankfully, the Double Ratchet Algorithm packs defensive mechanisms to protect our good friends, Alice and Bob, in case an attacker succeeds. The introduction in the spec nicely summarizes these defensive mechanisms.

> The parties derive new keys for every Double Ratchet message so that earlier keys cannot be calculated from later ones... The results of Diffie-Hellman calculations are mixed into the derived keys so that later keys cannot be calculated from earlier ones. These properties gives some protection to earlier or later encrypted messages in case of a compromise of a party's keys.

There isn't a single secret key that's used to encrypt/decrypt messages. Rather, the secret key changes every time Alice or Bob sends or receives a message. This functionality provides "forward security", meaning *earlier keys cannot be calculated from later ones*. Suppose an attacker compromises a secret key. They won't be able to decrypt previous messages they saw or intercepted because those messages were encrypted with a different secret key that can't be extrapolated from the current (compromised) key.

Can't the attacker decrypt future messages then? Nope, and this is where Double Ratchet lives up to its name!

I mentioned before that secret keys are regularly computed by the sending and receiving parties. The process by which secret keys are computed is called the [symmetric-key ratchet](https://signal.org/docs/specifications/doubleratchet/#symmetric-key-ratchet). This is the first ratchet: it takes the current key and some value and generates the next key. The tricky part is getting Alice and Bob to agree on a value so they end up with the same secret key. Alice and Bob could use a constant value or shared secret, but then it wouldn't be difficult for an attacker who comprises the current key to calculate later keys. By itself, the symmetric-key ratchet doesn't prevent an attacker from computing future keys and decrypting future messages, so we need a second ratchet.

The [Diffie-Hellman ratchet](https://signal.org/docs/specifications/doubleratchet/#diffie-hellman-ratchet) begins with each party generating a key pair with public and private components. Each party communicates its public key to the other, generates new keys, and produces [Diffie-Hellman (DH) outputs](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange#General_overview) that are fed into the symmetric-key ratchet. The specification provides more detail regarding the precise chronology of events. What I want to highlight is the relation between the second ratchet and "break-in recovery". The constant refreshing of keypairs and calculation of DH outputs addresses the problem of Alice and Bob agreeing on a non-constant value that's also difficult for an attacker to ascertain. An attacker would need to compromise encryption keys and DH outputs each time it wanted to compute the next encryption key. This is an incredibly high hurdle since key pairs are constantly regenerated. Therefore, Double Ratchet prevents attackers from predicting future encryption keys from a compromised key by combining these two ratchets.

Hopefully this gives you a better sense why the algorithm is designed the way it is. The protocol flow is pretty tough to wrap your head around- I'm struggling as I write this post despite hand-waving a lot of the details! Several months ago I decided to write my own implementation of Double Ratchet. My motivation was two-fold: I wanted to get a better "feel" of the protocol and implementation is often an effective learning method for me. I also thought it would be cool to create a package that other applications could use for Double-Ratchet end-to-end encryption.

Implementing the algorithm opened my eyes, if only slightly, to its inner workings and other implementation considerations. For instance, if two clients want to talk Double-Ratchet to each other, they'll most likely need a rendezvous server. Furthermore, they'll need to agree on an initial secret before they can start talking Double Ratchet. These considerations are outside the scope of the Double Ratchet specification. Fortunately, there's another specification, [Extended Triple Diffie-Hellman (X3DH)](https://signal.org/docs/specifications/x3dh/), which details how two parties can perform mutual public-key authentication and establish a shared secret key. This secret can then be used to initialize a Double Ratchet session.

My innocent attempt to implement the Double Ratchet Algorithm thus became a larger effort to develop an npm package for end-to-end encrypted communication.

**WARNING:** As of now, I don't recommend using this package in production because it hasn't received a formal security audit. If/when the package is reviewed, I'll be sure to give an update!

Besides implementing Double Ratchet, the package includes a [server component](https://github.com/zbo14/triple-double/blob/master/lib/server.js) that performs initial secret negotation via X3DH to establish a secure WebSocket channel between clients. Once the secure channel's set up, the clients can talk Double Ratchet to each other. A client can simultaneously maintain multiple Double Ratchet sessions, each corresponding to another client. Therefore, a web application could use this package to ensure end-to-end encryption for user communication via WebSockets without having to worry about Double Ratchet or implement X3DH itself.

I added the [project](/projects/triple-double/) to my protfolio page. You can also check it out on [GitHub](https://github.com/zbo14/triple-double)!
