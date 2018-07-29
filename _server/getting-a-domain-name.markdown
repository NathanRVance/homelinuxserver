---
title: "Getting a Domain Name"
desc: "Grab a free domain name from FreeDNS"
sortable: 0
---

The most exclusive part of web hosting tends to be the domain name. It can cost a significant amount to rent a good name from places like [GoDaddy](https://www.godaddy.com/). For some, the expense might be worth it. I am not one of those people.

In this guide, we will obtain a domain name from [FreeDNS][freedns]. Not only is this service free (as the name implies), it also provides dynamic DNS, which is vital when hosting from home.
1. Create an account at [FreeDNS][freedns].
2. Browse the [Registry](https://freedns.afraid.org/domain/registry/) for a domain that appeals to you, and come up with a subdomain. Determining the domain and the subdomain is the hardest part of setting up a home linux server. It took me several days of intense thought and consideration. But, it's worth it! When this task is completed, you will have assembled a Fully Qualified Domain Name (FQDN) for your server. We'll refer to it as `subdomain.domain.com` everywhere else on this website.
3. Navigate to the [Subdomains](https://freedns.afraid.org/subdomain/) section and register `subdomain.domain.com` as an [A record](https://support.dnsimple.com/articles/a-record/). You may wish to customize the domain list in [Preferences](https://freedns.afraid.org/profile/) for ease of using FreeDNS.
4. While you're at it, also nab `www.subdomain.domain.com` as an A record. Ideally, this should have been a CNAME record pointing to your main record, but FreeDNS doesn't support CNAMEs pointing to dynamic domains. Alternatively, if you spring the $60/yr to get a Premium account, then you can make `subdomain.domain.com` a [wildcard domain](https://en.wikipedia.org/wiki/Wildcard_DNS_record) that matches `*.subdomain.domain.com`. The tutorials in this site do not assume that you did this, but should work regardless.
5. Optionally, grab [AAAA (IPv6) records](https://en.wikipedia.org/wiki/IPv6_address#Domain_Name_System) corresponding to your two A records.
6. Set up Dynamic DNS using the [updated interface](https://freedns.afraid.org/dynamic/v2/). Enable dynamic DNS for your domains, generate the cron scripts, and add them to your crontab.
7. On your server machine, edit `/etc/hosts` to include `subdomain.domain.com` as your hostname. When you're done, executing `hostname -s` should return the hostname that you set up for your machine on installation, and `hostname -f` should return `subdomain.domain.com`.

[freedns]: https://freedns.afraid.org/
