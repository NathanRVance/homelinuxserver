---
title: "Email with Postfix"
desc: "Securely send and receive email with Postfix"
sortable: 3
---

Before we start, make sure that your internet service provider isn't blocking tcp port 25 (smtp). I found that AT&T will unblock it for free; I just had to call and ask. Once that is done, we can proceed.
1. If you haven't done so already, install ssl certificates. We will use the same certs for both https and for encrypting email.
2. Allow smtp through the firewall. If you use firewalld,
	```
	# firewall-cmd --permanent --zone=public --add-service=smtp
	# firewall-cmd --permanent --zone=public --add-port=587/tcp
	# firewall-cmd --reload
	```
3. Install postfix
	```
	# apt install postfix
	```
4. Change or add the following options in `/etc/postfix/main.cf`
	```
	myorigin = $myhostname
	inet_interfaces = all
	smtpd_use_tls = yes
	smtpd_tls_auth_only = yes
	smtpd_tls_received_header = yes
	smtpd_tls_cert_file = /etc/letsencrypt/live/subdomain.domain.com/cert.pem
	smtpd_tls_key_file = /etc/letsencrypt/live/subdomain.domain.com/privkey.pem
	tls_random_source = dev:/dev/urandom
	smtpd_tls_security_level = may
	smtp_tls_security_level = may
	inet_protocols = ipv4
	```
5. Uncomment (or modify) the following lines in `/etc/postfix/master.cf`
	```
	submission inet n       -       n       -       -       smtpd
	 -o syslog_name=postfix/submission
	 -o smtpd_tls_security_level=encrypt
	 -o smtpd_sasl_auth_enable=yes
	 -o smtpd_reject_unlisted_recipient=no 
	 -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
	 -o milter_macro_daemon_name=ORIGINATING
	```
6. Restart postfix
	```
	# systemctl restart postfix.service
	```
7. If all is working properly, you should be able to send an email:
	```
	$ mail -s "subject" name@example.com
	body of message
	[CTRL+d]
	```
	If all is not working properly, inspect `/var/log/mail.info` for details.

When you receive the email (perhaps at gmail), you should be able to verify that it is encrypted with TLS. However, if you use gmail, you'll find that things aren't entirely hunky dory:

![kinda works](/assets/no-dkms.png)

The issue is that question mark. In my case, hovering over it displays the text `Gmail couldn't verify that vance.homelinuxserver.org actually sent this message (and not a spammer).`

The solution: Use DKIM to authenticate email. Ah, yes, yet another technology to throw into the mix. Here we go.
1. Install opendkim
	```
	# apt install opendkim opendkim-tools
	```
2. Change or add the following options in `/etc/opendkim.conf`
	```
	UMask                   007
	Domain                  subdomain.domain.com
	KeyFile                 /etc/dkimkeys/dkim.key
	Selector                mail
	Canonicalization        relaxed/relaxed
	```
3. In `/etc/default/opendkim`, uncomment the line
	```
	SOCKET=inet:12345@localhost
	```
	Disclaimer: This doesn't actually do anything in Debian 9. You'll also have to modify the service file, `/lib/systemd/system/opendkim.service`, which passes the socket in as an argument:
	```
	ExecStart=/usr/sbin/opendkim -P /var/run/opendkim/opendkim.pid -p inet:12345@localhost
	```
	and, as always when modifying a .service file, reload the daemons with `systemctl daemon-reload`.
4. Edit `/etc/postfix/main.cf` to account for DKIM
	```
	# DKIM
	milter_default_action = accept
	milter_protocol = 2
	smtpd_milters = inet:localhost:12345
	non_smtpd_milters = inet:localhost:12345
	```
5. Generate dkim keys and move the private one to the proper location
	```
	# opendkim-genkey -t -s mail -d subdomain.domain.com
	# mv mail.private /etc/dkimkeys/dkim.key
	# chown opendkim:opendkim /etc/dkimkeys/dkim.key
	```
6. The file mail.txt (generated with mail.private) contains the public key information. Mine looks like this:
	```
	mail._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; t=y; "
		"p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA9VI66HU8kitypUPGrZbajuWM1tqCDhLuPafk5JMItKIspdBhTO8YxXj2we01Yba21H0JCjROroy23I0r2XT2tmkfsY4q3KhsZlV/UUEH1RohLjmEOYqtRJATUtEdJxG0Pp3KW96L4fTHaabzXsDcdVyJE++I/OvU9NL+UUw4izbYzadBOrU0IDqDoD86Kv5OO3WMK1VNQ97qss"
		"iqAmhBQHrrEsFHmrjxhdaTfBpqxjqZdfvolGkzqEHInJMHmd54msFxJOdLYfvqn8lp6B5+J0islBisAd36wpNxWry8AwB7McxtK9nn+z4s/FYKGaqykaaAYXyer52FPhuM1u1SDwIDAQAB" )  ; ----- DKIM key mail for vance.homelinuxserver.org
	```
	Go to freedns and create a TXT record. For the subdomain, enter `mail._domainkey.subdomain`, and for the destination, enter the parts in quotes:
	```
	"v=DKIM1; h=sha256; k=rsa; t=y; " "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA9VI66HU8kitypUPGrZbajuWM1tqCDhLuPafk5JMItKIspdBhTO8YxXj2we01Yba21H0JCjROroy23I0r2XT2tmkfsY4q3KhsZlV/UUEH1RohLjmEOYqtRJATUtEdJxG0Pp3KW96L4fTHaabzXsDcdVyJE++I/OvU9NL+UUw4izbYzadBOrU0IDqDoD86Kv5OO3WMK1VNQ97qss" "iqAmhBQHrrEsFHmrjxhdaTfBpqxjqZdfvolGkzqEHInJMHmd54msFxJOdLYfvqn8lp6B5+J0islBisAd36wpNxWry8AwB7McxtK9nn+z4s/FYKGaqykaaAYXyer52FPhuM1u1SDwIDAQAB"
	```
	These are split up because TXT records can't contain chunks larger than 255 bytes.
7. Start opendkim and restart postfix
	```
	# systemctl start opendkim
	# systemctl restart postfix
	```

When you send a new email, gmail will be able to verify that it was you who sent it.

![works](/assets/with-dkms.png)
