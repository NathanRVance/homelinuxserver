---
title: "SSL with Let's Encrypt"
desc: "SSL keys are used to encrypt web pages in https connection, and to encrypt email."
sortable: 1
---

The only thing better than a free domain name is a free SSL certificate to go with it. According to Let's Encrypt's [website][letsencrypt], "Let's Encrypt is a free, automated, and open Certificate Authority."

The client for Let's Encrypt is called [certbot](https://certbot.eff.org/), which is the software that will actually generate your keys and send the public key back to Let's Encrypt.
1. Install certbot.
	```
	$ sudo apt install certbot
	```
2. Download standalone keys. You may need to open up ports in your firewall to make it work.
	```
	$ sudo certbot certonly --standalone -d subdomain.domain.com -d www.subdomain.domain.com
	```
3. Automate the renewal process by adding the following to your crontab:
	```
	0 2 * * 1 certbot renew
	```
	This will attempt a renewal at 2:00AM every Monday.

[letsencrypt]: https://letsencrypt.org/
