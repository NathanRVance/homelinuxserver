---
layout: page
title: About
permalink: /about/
---

# This Site
This server, vance.homelinuxserver.org, runs Debian 9 and was configured entirely using free and open source software. It is part of a larger set of free and open source technologies that makes it possible for me, the primary user of this server, to cut ties with large tech companies like Google. The goal of this site is to document the process of setting up this server so that others can do the same.

The scope of this server covers the following:
 * [Postfix](/serverdocs/email-with-postfix.html) to replace Gmail.
 * [Nextcloud](/serverdocs/file-sharing-with-nextcloud.html) to replace Google Drive (or Backup and Sync, whatever).
   * Pair it with [LibreOffice](https://www.libreoffice.org/) to replace Google Docs, Sheets, and Slides.
   * Install [a plugin](https://apps.nextcloud.com/apps/calendar) to replace Google Calendar.
   * [Another plugin](https://apps.nextcloud.com/apps/mail) replaces the Gmail web client.
   * And [another](https://apps.nextcloud.com/apps/spreed) does video calls.
   * You can even [stream music](https://apps.nextcloud.com/apps/music), thus replacing Spotify and Pandora.

There are many more services that are extremely difficult to host at home. I can recommend a few open source and/or freedom loving alternatives:
 * [BitChute](https://www.bitchute.com/) to replace YouTube.
 * [LBRY](/clientdocs/videos-with-lbry.html) is another approach to YouTube replacement.
 * [DuckDuckGo](https://duckduckgo.com/), while not open source, at least prioritizes privacy. It's a good replacement for Google Search.
 * [Mastodon](https://mastodon.social/about) to replace social media.

Shamefully, there are several things that I have no good recommendations for:
 * Google Voice. Maybe [Matrix](https://matrix.org/) someday? If it bridged SMS, then yes. But it doesn't (yet), so no.
 * Android phones, and the Google Play ecosystem that goes with them. Right now, nothing. But [Librem 5](https://puri.sm/shop/librem-5/) looks like it will be awesome when it ships in 2019!
   * If all you need is a handheld, there are [many linux handhelds](https://www.reddit.com/r/linux/comments/4biamr/a_list_of_handheldpocket_linux_computers/) out there.
 * Google Maps. For reference, the privacy loving DuckDuckGo defaults to Bing Maps. The only thing that comes close is [OpenStreetMap](https://www.openstreetmap.org), although it's quite point-of-interest sparse.

# My Philosophy Concerning Open Source
Throughout this site, I stress the importance of decentralization in communication services. The reason for this is:

>Power tends to corrupt, and absolute power corrupts absolutely.

-Lord Acton

This situation has historically been seen in both politics and economics, but is present in software as well. In essence, it's a matter of where power should rest:
 * Who gets to create legislation? A leader? Or the people who live under the law?
 * Who gets to design the economy? The state? Or the people who work and do commerce in the economy?
 * Who gets to control software? A company? Or the people who use the software?
 * In general, who should wield power? A trusted entitiy? Or the people who are vulnerable to that power?

These questions have been answered many different ways. Here are a few contrasting examples (implications are my observations):

|---
| Field | Definition | Implication |
|:-:|-|-
| Centralized Politics (Dictatorship) | Absolute power to legislate is consolidated in one leader | The single source of authoritarian rule will be corrupted |
| Decentralized Politics (Democracy) | Legislative power is distributed among many people | It's more difficult to corrupt (buy out) the masses |
| Centralized Economics <br> (Communism or Statism) | Power to produce is consolidated in the commune (communism) or a big business (statism) | A single source for goods and services tends to overcharge for bad products |
| Decentralized Economics (Capitalism) | Production is distributed among many individuals and businesses | The freedom to choose suppliers keeps prices down and quality up |
| Centralized Software (Proprietary) | Control of software is consolidated in the company that wrote it | The end user is not allowed to know everything that their software is doing or make unapproved modifications to it |
| Decentralized Software <br> (Open Source) | Control of software is distributed among the people that use it | Because source code is available, every function is visible and any modification that can be coded is possible |
{:.mbtablestyle}

In practice, the extremes presented in this table rarely if ever exist. However, many situations lean one way or the other, toward either centralization or decentralization. Being a [classic liberal](https://www.sciencedaily.com/terms/classical_liberalism.htm), I believe that centralizing power is always dangerous when people (as opposed to God) are involved. I believe authoritarian legislation (dictatorships), planned economies (communism, statism), and centrally controlled software (proprietary software) will tend toward a corrupt, ineffective, and restricted society.

On the other hand, when the power to legislate is shifted to the people (democracy), and the power to produce is held by individuals (free market capitalism), and the power to modify software is held by users (open source software), then society will tend toward being just, efficient, and free.

This clash of ideas can best be seen through two opposing worldviews: Society is either a machine that must be engineered, or it is an ecosystem that must be conserved.

Every machine requires a centralized system so that it can function efficiently and remain stable. An engineer (dictator) must design it, a mechanic (commune) must maintain it, and the owner (proprietor) must use it.

In contrast, every ecosystem will tend toward efficiency and stability if and only if it is allowed to do so. The ecosystem will design itself (democracy), provide for itself (capitalism), and engineer itself (open source) without the heavy hand of an outside power messing with it, thankyouverymuch.

It would be great if these worldviews were complimentary. That way, we could all have our political differences, shrug it off as a matter of opinion, and carry on. Unfortunately, they're mutually exclusive, and in quite a disasterous way. Leave a machine be and it will fall apart. Try to engineer an ecosystem and you'll destroy it. If society is one of these two things, and it is mistreated as the other, then very bad things happen.

For what it's worth, I'm convinced that society is an ecosystem. Part of what makes countries like the USA great is decentralized politics and economics. In stark contrast, countries like Noth Korea are humanitarian nightmares because of their authoritarian governments and command economies. Similarly, Linux is great because it is open source and community driven. Windows and OS X suck because they are locked down and locked into their respective companies' visions.

If you disagree with any of this, feel free to shoot me an email (the address is in the footer). I'd love to chat. Whether or not you agree with my philosophy, I sincerely hope that you find this web site useful for participating in the open source software ecosystem.
