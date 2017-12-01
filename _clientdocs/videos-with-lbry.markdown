---
title: "Videos with LBRY"
desc: "A distributed, open source alternative to YouTube"
sortable: 0
---

According to their website, "[LBRY](https://lbry.io/) is a free, open, and community-run digital marketplace." In LBRY, content producers share videos and set a price for them, where the price is in LBC, LBRY's [blockchain](https://en.wikipedia.org/wiki/Blockchain) backed [cryptocurrency](https://en.wikipedia.org/wiki/Cryptocurrency). For the concerned, I have found that most of the content out there is the wonderful price of free, which is fine by me.

A word of warning - LBRY is (currently) only accesable via a client. If you aren't particularly interested in mucking around with finicky software and just want to watch videos, then I'd recommend checking out [BitChute](https://www.bitchute.com/), a web-based decentralized YouTube alternative that uses [WebTorrent](https://webtorrent.io/).

LBRY packages their software for Debian/Ubuntu, but it's possible to install it on most major distributions. I ran into some pretty major trouble, which I thought made the process worth documenting. If, after installing libsecret, you have an error that looks like one of these:
```
(node:28375) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: Error calling StartServiceByName for org.freedesktop.secrets: Timeout was reached
```
or
```
** Message: Remote error from secret service: org.freedesktop.DBus.Error.ServiceUnknown: The name org.freedesktop.secrets was not provided by any .service files
```
then chances are you don't have a daemon running that implements org.freedesktop.secrets. To fix this on Debian, install gnome-keyring.
```
$ sudo apt install gnome-keyring
```
Depending on your setup, you may need to start the daemon:
```
$ gnome-keyring-daemon
```
