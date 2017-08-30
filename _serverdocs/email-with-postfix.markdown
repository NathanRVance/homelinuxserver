---
title: "Email with Postfix"
desc: "Securely send and receive email with Postfix"
sortable: 3
---

Before we start, make sure that your internet service provider isn't blocking tcp port 25 (smtp). I found that [AT&T will unblock it for free](http://www.ka9q.net/Uverse/port25.html); I just had to call and ask. Once that is done, we can proceed.
1. If you haven't done so already, [install ssl certificates](ssl-with-letsencrypt.html). We will use the same certs for both https and for encrypting email.
2. Allow smtp through the firewall. If you use firewalld,
	```
	# firewall-cmd --permanent --add-service=smtp
	# firewall-cmd --permanent --add-port=587/tcp
	# firewall-cmd --permanent --add-service=imap
	# firewall-cmd --reload
	```
3. Install postfix.
	```
	# apt install postfix
	```
4. Change or add the following options in `/etc/postfix/main.cf`:
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
	alias_maps = hash:/etc/aliases
	alias_database = hash:/etc/aliases
	smtpd_sasl_type = dovecot
	smtpd_sasl_path = private/auth
	smtpd_sasl_auth_enable = yes
	local_recipient_maps = proxy:unix:passwd.byname $alias_maps
	```
5. Set up aliases in `/etc/aliases`. This isn't strictly required if you only want to receive mail for explicit system users, but may be helpful if you want, well, aliases. Mine looks like this:
	```
	postmaster:     root
	root:           nathan
	```
	This configuration makes mail addressed to postmaster go to root, and root's mail goes to me. So that this change takes effect, execute:
	```
	$ sudo newaliases
	```
5. Uncomment (or modify) the following lines in `/etc/postfix/master.cf`:
	```
	submission inet n       -       -       -       -       smtpd
	 -o syslog_name=postfix/submission
	 -o smtpd_tls_security_level=encrypt
	 -o smtpd_sasl_auth_enable=yes
	 -o mtpd_sasl_type=dovecot
	 -o smtpd_sasl_path=private/auth
	 -o smtpd_reject_unlisted_recipient=no 
	 -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
	 -o milter_macro_daemon_name=ORIGINATING
	```
6. Restart postfix.
	```
	# systemctl restart postfix.service
	```
7. If all is working properly, you should be able to send an email.
	```
	$ mail -s "subject" name@example.com
	body of message
	[CTRL+d]
	```
	If all is not working properly, inspect `/var/log/mail.info` for details.

When you receive the email (perhaps at gmail), you should be able to verify that it is encrypted with TLS. However, if you use gmail, you'll find that things aren't entirely hunky dory:

![kinda works](/assets/no-dkms.png)

The issue is that question mark. In my case, hovering over it displays the text `Gmail couldn't verify that vance.homelinuxserver.org actually sent this message (and not a spammer).`

The solution: Use [DKIM](http://www.dkim.org/) to authenticate email. Ah, yes, yet another technology to throw into the mix. Here we go.
1. Install opendkim.
	```
	# apt install opendkim opendkim-tools
	```
2. Change or add the following options in `/etc/opendkim.conf`:
	```
	UMask                   007
	Domain                  subdomain.domain.com
	KeyFile                 /etc/dkimkeys/dkim.key
	Selector                mail
	Canonicalization        relaxed/relaxed
	```
3. In `/etc/default/opendkim`, uncomment the line:
	```
	SOCKET=inet:12345@localhost
	```
	Disclaimer: This doesn't actually do anything in Debian 9. You'll also have to modify the service file, `/lib/systemd/system/opendkim.service`, which passes the socket in as an argument:
	```
	ExecStart=/usr/sbin/opendkim -P /var/run/opendkim/opendkim.pid -p inet:12345@localhost
	```
	As always when modifying a .service file, reload the daemons with `systemctl daemon-reload`.
4. Edit `/etc/postfix/main.cf` to account for DKIM.
	```
	# DKIM
	milter_default_action = accept
	milter_protocol = 2
	smtpd_milters = inet:localhost:12345
	non_smtpd_milters = inet:localhost:12345
	```
5. Generate dkim keys and move the private one to the proper location.
	```
	# opendkim-genkey -t -s mail -d subdomain.domain.com
	# mv mail.private /etc/dkimkeys/dkim.key
	# chown opendkim:opendkim /etc/dkimkeys/dkim.key
	```
6. The file mail.txt (generated along with mail.private in the previous step) contains the public key information. Mine looks like this:
	```
	mail._domainkey	IN	TXT	( "v=DKIM1; h=sha256; k=rsa; t=y; "
		"p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA9VI66HU8kitypUPGrZbajuWM1tqCDhLuPafk5JMItKIspdBhTO8YxXj2we01Yba21H0JCjROroy23I0r2XT2tmkfsY4q3KhsZlV/UUEH1RohLjmEOYqtRJATUtEdJxG0Pp3KW96L4fTHaabzXsDcdVyJE++I/OvU9NL+UUw4izbYzadBOrU0IDqDoD86Kv5OO3WMK1VNQ97qss"
		"iqAmhBQHrrEsFHmrjxhdaTfBpqxjqZdfvolGkzqEHInJMHmd54msFxJOdLYfvqn8lp6B5+J0islBisAd36wpNxWry8AwB7McxtK9nn+z4s/FYKGaqykaaAYXyer52FPhuM1u1SDwIDAQAB" )  ; ----- DKIM key mail for vance.homelinuxserver.org
	```
	Go to [FreeDNS](https://freedns.afraid.org/) and create a [TXT record](https://en.wikipedia.org/wiki/TXT_record). For the subdomain, enter `mail._domainkey.subdomain`, and for the destination, enter the parts of `mail.txt` in quotes:
	```
	"v=DKIM1; h=sha256; k=rsa; t=y; " "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA9VI66HU8kitypUPGrZbajuWM1tqCDhLuPafk5JMItKIspdBhTO8YxXj2we01Yba21H0JCjROroy23I0r2XT2tmkfsY4q3KhsZlV/UUEH1RohLjmEOYqtRJATUtEdJxG0Pp3KW96L4fTHaabzXsDcdVyJE++I/OvU9NL+UUw4izbYzadBOrU0IDqDoD86Kv5OO3WMK1VNQ97qss" "iqAmhBQHrrEsFHmrjxhdaTfBpqxjqZdfvolGkzqEHInJMHmd54msFxJOdLYfvqn8lp6B5+J0islBisAd36wpNxWry8AwB7McxtK9nn+z4s/FYKGaqykaaAYXyer52FPhuM1u1SDwIDAQAB"
	```
	Of course, it won't work if you use my public key, so use your own :). The reason why the key is split into 3 strings is that TXT records can't contain chunks larger than 255 bytes.
7. Start opendkim and restart postfix.
	```
	# systemctl start opendkim
	# systemctl restart postfix
	```

Now when you send an email, gmail can verify that it was you who sent it.

![works](/assets/with-dkms.png)

You can stop there if you prefer to do everything on the command line over ssh. However, if you wish to use an email client, you must set up imap with [Dovecot](https://dovecot.org/).
1. Install Dovecot.
	```
	$ sudo apt install dovecot-core dovecot-imapd
	```
2. Wipe out `/etc/dovecot/dovecot.conf` and replace it with:
	```
	disable_plaintext_auth = no
	mail_privileged_group = mail
	mail_location = mbox:~/mail:INBOX=/var/mail/%u
	userdb {
	  driver = passwd
	}
	passdb {
	  args = %s
	  driver = pam
	}
	protocols = " imap"
	protocol imap {
	  mail_plugins = " autocreate"
	}
	plugin {
	  autocreate = Trash
	  autocreate2 = Sent
	  autosubscribe = Trash
	  autosubscribe2 = Sent
	}
	service auth {
	  unix_listener /var/spool/postfix/private/auth {
	    group = postfix
	    mode = 0660
	    user = postfix
	  }
	}
	ssl=required
	ssl_cert = </etc/letsencrypt/live/subdomain.domain.com/cert.pem
	ssl_key = </etc/letsencrypt/live/subdomain.domain.com/privkey.pem
	```

3. Restart postfix and dovecot.
	```
	$ sudo systemctl restart postfix
	$ sudo systemctl restart dovecot
	```
4. Connect to your server using imap. The connection is authenticated with an unencrypted password, but it's tunneled using STARTTLS.
