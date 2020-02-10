---
layout: post
title: Passing passwords to myself
---

Oh, what to do with passwords...

Maybe you have one you use for everything. It's easy for you to remember and maybe difficult for others to guess. However, an attacker that discovers your password can access any number of services you use and your personal information.

A much better alternative is using a password manager that generates unique, hard-to-guess passwords for each service you use. These passwords are typically encrypted with a password you provide and then stored in a database somewhere. When you need to log in to a service, you give the password manager *your* password. It then decrypts the password it generated for that account.

Something about storing this information on somebody else's disk space irks me. Perhaps the paranoia is unfounded - the passwords are encrypted, or at least in theory they are. If an attacker compromises the system and fails to crack the encryption, they could wipe the database. Unlikely, but still a cause for concern. I wanted to design my own password storage solution so I wasn't reliant on external infrastructure and security.

The proof-of-concept looked something like the following:
1. on a computer, use `/dev/urand` to generate pseudorandom bytes
1. base64-encode the output to get new password
1. copy and paste the password into a file on a usb flash drive
1. encrypt the file with GPG symmetric encryption and my own password
1. later, decrypt the file with GPG and my password
1. copy and paste the password to login, then re-encrypt the file

This isn't entirely different from typical password managers. However, the encrypted passwords are stored on hardware I own. I could go further and store each encrypted password in its own file. And if I have a GPG key on the device, I could encrypt passwords with a public key instead of a password.

This worked nicely on my laptop and desktop. I'd insert the flash drive, decrypt the files, access the contents, re-encrypt them, and then remove the flash drive. The wrinkle was I couldn't access passwords on my iPhone. There might be a way to do this by using a compatible flash drive and GPG client.

At this point, I came across [pass](https://www.passwordstore.org/). The CLI implements the proof-of-concept I had in mind, and more. It has an option for copying decrypted passwords to the clipboard for a short amount of time (super handy!). In addition, it has a utility for generating new pseudorandom passwords and immediately encrypting them so there's no need to manually summon `/dev/urand` and `gpg`. And it has mobile clients so I could finally access passwords on my phone! Another plus was I could use git as source control for the passwords, which would be encrypted and stored in a repository. This would enable password synchronization across devices. That is, I could add a new password on my phone and when I pulled from the remote repository on my desktop, the new password would show up in the local password store.

Factoring in these benefits, I decided to use `pass` in lieu of my homegrown solution. But something still bothered me. I wanted source control and password synchronization, but was I going to store my encrypted passwords on GitHub? I could create a private repo but this solution didn't strike me as philosophically different from using a password manager: the encrypted passwords would still be stored on someone else's disk.

What if I ran my own git server then? I could do this on my rpi on my private home network. That way, it could continually run and it wouldn't be exposed to the world. I could use public key authentication so someone else on the network wouldn't be able to access the repo unless they managed to steal and decrypt my private key. There would be one downside: I'd need to sync passwords at home. This didn't bother me too much. If I added a password on my phone, I'd go home before I'd need to access the password on my desktop. And if I added the password on my desktop, I could immediately push the changes and sync my phone.

This sounded pretty good! I spun up a git server on my rpi - pretty much an SSH server in a Docker container with git installed and volumes for the encrypted password repository and SSH credentials/config. Since `pass` encrypts with GPG keys and the git server authenticates SSH keys, I needed some way to transfer the keys on my desktop to my iPhone (I couldn't generate new keys in [passforios](https://mssun.github.io/passforios/)). So I created QR codes for my keys, scanned these with the app, and deleted the images from my desktop. This wasn't too difficult; however, it took some troubleshooting to discover the GPG private key was long enough such that it required 2 QR codes.

Moral of the story, it's not too difficult to run your own password manager! `pass` does a lot of the work for you, so you just have to bootstrap the git server and snap some QR codes. I'll push my code for the git server soon so stay tuned!
