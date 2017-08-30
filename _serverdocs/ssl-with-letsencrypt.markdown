---
title: "SSL with Let's Encrypt"
desc: "Use SSL keys to encrypt web pages in https connection, and to encrypt email"
sortable: 1
---

The only thing better than a [free domain name](getting-a-domain-name.html) is a free SSL certificate to go with it. This is possible with Let's Encrypt. According to Let's Encrypt's [website][letsencrypt], "Let's Encrypt is a free, automated, and open Certificate Authority."

The client for Let's Encrypt is called [certbot](https://certbot.eff.org/). This software is used to generate your keys and send the public key back to Let's Encrypt.
1. Install certbot.
	```
	$ sudo apt install certbot
	```
2. Download standalone keys. If you haven't already, you may need to open up tcp port 80 (http) in your firewall so that Let's Encrypt can verify that you own your domain.
	```
	$ sudo certbot certonly --standalone -d subdomain.domain.com -d www.subdomain.domain.com
	```
3. Automate the renewal process by adding the following to your crontab:
	```
	0 2 * * 1 certbot renew
	```
	This configuration will attempt a renewal at 2:00AM every Monday.

[letsencrypt]: https://letsencrypt.org/
