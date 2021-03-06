---
title: "HTTPS with Nginx"
desc: "Host your own website"
sortable: 2
---

[Nginx](https://nginx.org/) is a free and open source webserver for Linux. It is newer and often easier to configure than [Apache](https://httpd.apache.org/), which is why we are using it in this guide.

1. Install nginx.
   ```
   $ sudo apt install nginx
   ```
2. Generate a dhparams.pem.
   ```
   # cd /etc/nginx
   # openssl dhparam -out dhparams.pem 2048
   ```
3. Configure nginx to automatically redirect http to https, and www to not www (it's best practice to stick with one). Edit `/etc/nginx/nginx.conf`. Inside the http block, add a server block:
   ```
   server {
   	listen                  80;
   	listen                  [::]:80;
   	listen                  443 default_server ssl;
   	server_name             subdomain.domain.com www.subdomain.domain.com;
   	if ($http_host = www.subdomain.domain.com) {
   		return 303 https://subdomain.domain.com$request_uri;
   	}
   	if ($scheme = http) {
   		return 301 https://subdomain.domain.com$request_uri;
   	}
   	ssl_certificate         /etc/letsencrypt/live/subdomain.domain.com/fullchain.pem;
   	ssl_certificate_key     /etc/letsencrypt/live/subdomain.domain.com/privkey.pem;
   	ssl_dhparam             /etc/nginx/dhparams.pem;
   	ssl_protocols           TLSv1 TLSv1.1 TLSv1.2;
   	# The following is the current (Sept 2017) recommendation from ssllabs.com.
   	ssl_ciphers             'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
   }
   ```
   Additionally, set the website root and index file, also in the http block:
   ```
   root /var/www;
   index index.html;
   ```
4. Restart nginx so that the new configuration takes affect.
   ```
   # systemctl restart nginx.service
   ```
5. You'll run into trouble when trying to upgrade Let's Encrypt certificates in your cron job because certbot needs to bind to port 80 for the renewal process. Modify your crontab so that the executed command looks like this:
   ```
   certbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"
   ```
6. Visit [SSL Labs](https://www.ssllabs.com/ssltest/) and test the security of your server. Fix any vulnerabilities it finds.
7. Create a website! Use whatever tools you are comfortable with. This site was generated using [jekyll](https://jekyllrb.com/), which basically turns markdown files (generated by you) and formatting (available [online](http://themes.jekyllrc.org/)) into a static HTML website. I found that, after a brief (1 hour) learning curve, nealy 100% of my energies are now spent on content creation rather than fighting with the technologies. If you need further inspiration, you can find the source for this website on [Github](https://github.com/NathanRVance/homelinuxserver).
