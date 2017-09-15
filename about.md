---
layout: page
title: About
permalink: /about/
---

# This Site
This server, vance.homelinuxserver.org, runs Debian 9 and was configured entirely using free and open source technology. It is part of a larger system of free and open source technologies that makes it possible for me, the primary user of this server, to cut ties with large tech companies like Google. The goal of this site is to document the process of setting up this server so that others can do the same.

Unfortunately, this server isn't enough to make me 100% self sufficient. The scope of this server covers the following:
 * [Postfix](/serverdocs/email-with-postfix.html) to replace Gmail.
 * [Nextcloud](/serverdocs/file-sharing-with-nextcloud.html) to replace Google Drive (or Backup and Sync, whatever).
   * Pair it with [LibreOffice](https://www.libreoffice.org/) to replace Google Docs, Sheets, and Slides.
   * Install [a plugin](https://apps.nextcloud.com/apps/calendar) to replace Google Calendar.
   * [Another plugin](https://apps.nextcloud.com/apps/mail) replaces the gmail web client.
   * And [another](https://apps.nextcloud.com/apps/spreed) does video calls.
   * You can even [stream music](https://apps.nextcloud.com/apps/music), thus replacing Spotify and Pandora.

There are many more services that can't be hosted by a home linux server (at least not as easily). I can still recommend a few open source and/or freedom loving alternatives:
 * [LBRY](/clientdocs/videos-with-lbry.html) to replace YouTube.
 * [DuckDuckGo](https://duckduckgo.com/), while not open source, at least prioritizes privacy. It's a reasonable replacement for Google Search.
 * [Mastodon](https://mastodon.social/about) to replace social media.

Shamefully, there are several things that I have no good recommendations for:
 * Google Voice. Maybe [Matrix](https://matrix.org/) someday? If it bridged SMS, then yes. But it doesn't (yet), so no.
 * Android phones, and the Google Play ecosystem that goes with them. Right now, nothing. But [Librem 5](https://puri.sm/shop/librem-5/) looks like it will be awesome when it ships in 2019!
   * If all you need is a handheld, there are [many linux handhelds](https://www.reddit.com/r/linux/comments/4biamr/a_list_of_handheldpocket_linux_computers/) out there.
 * Google Maps. For reference, the privacy loving DuckDuckGo defaults to Bing Maps. There isn't much out here, at least not much decent.

# My Philosophy Concerning Open Source
Throughout this site, I stress the importance of decentralization in communication services. The reason for this is:

>Power tends to corrupt, and absolute power corrupts absolutely.

-Lord Acton

This situation has historically been seen in both politics and economics, but can be seen in software as well. The basic question of where power should rest can be summarized as follows:
 * Who gets to create legislation, a leader or the people who live under it?
 * Who gets to design the economy, the state or the people who work and do commerse in it?
 * Who gets to design software, a company or the people who use it?
 * In general, who should have power? A trusted entitiy? Or the people who are vulnerable to that power?

This question has been answered different ways throughout history. Here are a few contrasting examples:

|---
| Field | Definition | Implication |
|:-:|-|-
| Centralized Politics (Dictatorship) | Absolute power to legislate is consolidated in one leader | The single source of authoritarian rule will be corrupted |
| Decentralized Politics (Democracy) | Legislative power is distributed among many people | It's more difficult to corrupt (buy out) the masses |
| Centralized Economics <br> (Communism or Statism) | Power to produce is consolidated in the commune (communism) or a big business (statism) | A single source for goods and services tends to overcharge for bad products |
| Decentralized Economics (Capitalism) | Production is distributed among many individuals and businesses | The freedom to choose suppliers keeps prices down and quality up |
| Centralized Software (Proprietary) | Control of software is consolidated in the company that wrote it | Proprietary software companies tend to sell user data to increase profits |
| Decentralized Software <br> (Open Source) | Control of software is distributed among the people that use it | Because projects can be forked, only data that end users want shared ends up getting shared\* |
{:.mbtablestyle}
\* If someone steals it, it's on the sysadmin who hosted the open source software.

Different people lean towards either centralized or decentralized systems. Being a [classic liberal](https://www.sciencedaily.com/terms/classical_liberalism.htm), I believe that centralizing power is always dangerous when people (as opposed to God) are involved. I believe authoritarian legislation (dictatorships), planned economies (communism, statism), and centrally controlled software (proprietary software) will tend toward a corrupt, ineffective, and insecure society.

On the other hand, when the power to legislate is shifted to the people (democracy), and the power to produce is held by individuals (free market capitalism), and the power to modify software is held by users (open source software), then society will tend toward being just, efficient, and secure.

This clash of ideas can best be seen though two opposing worldviews: Society is either a machine that must be engineered, or it is an ecosystem that must be conserved.

Every machine requires a centralized system so that it can function efficiently and remain stable. An engineer (dictator) must design the system, a mechanic (commune) must maintain it, and the owner (proprietor) must use it.

In contrast, every ecosystem will tend toward efficiency and stability if it is allowed to do so. The ecosystem will design itself (democracy), provide for itself (capitalism), and make decisions for itself (open source) without the heavy hand of an outside power messing with it, thankyouverymuch.

It would be great if these worldviews were complimentary. That way, we could all have our political differences, shrug it off as a matter of opinion, and carry on. Unfortunately, they're mutually exclusive, and in quite a disasterous way. Leave a machine be and it will fall apart. Try to engineer an ecosystem and you'll destroy it. If society is one of these two things, and it is mistreated as the other, then very bad things happen.

For what it's worth, I for one am convinced that society is an ecosystem. This worldview was influenced by what I've read, experienced, observed, and reasoned out. If you disagree, feel free to shoot me an email (the address is in the footer). I'd love to chat.

Whether or not you agree with my philosophy, I sincerely hope that you find this web site useful for participating in the open source software ecosystem.
