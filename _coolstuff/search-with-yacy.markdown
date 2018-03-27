---
title: "Search with YaCy"
desc: "A distributed, open source search portal"
sortable: 1
---

[YaCy](http://yacy.net) is fully decentralized, open source, web search powered by the users. It is a piece of software that runs on your computer which you use to perform web searches, accessing indexing information from other YaCy instances across the world. Here, I provide instructions for setting YaCy up on a Debian 9 server.

1. Install java.
   ```
   # apt install openjdk-8-jdk
   ```
2. The easiest installation route is through a package. The official mirror is currently incompatible with Debian 9 (instructions for when they fix it are on their [wiki](http://www.yacy-websuche.de/wiki/index.php/En:DebianInstall)), but we have options.
   ```
   # wget -qO - https://bintray.com/user/downloadSubjectPublicKey?username=bintray | apt-key add -
   # echo "deb https://dl.bintray.com/luccioman/yacy_search_server stretch main" > /etc/apt/sources.list.d/yacy.list
   ```
3. Install YaCy.
   ```
   # apt update
   # apt install yacy
   ```
4. The installer should prompt you for some details, and afterward, YaCy should be automatically started.
   ```
   # systemctl status yacy.service
   ● yacy.service - LSB: Distributed web search engine
      Loaded: loaded (/etc/init.d/yacy; generated; vendor preset: enabled)
      Active: active (running) since Thu 2017-12-21 08:41:18 EST; 19min ago
   <omitted for brevity>
   ```
5. Punch a hole through your firewall for tcp port 8090. If you use firewalld:
   ```
   # firewall-cmd --permanent --add-port=8090/tcp
   # firewall-cmd --reload
   ```
6. You should now be able to access YaCy locally on `localhost:8090`, or remotely from `subdomain.domain.com:8090`. Note that remote administration requires logging in to your YaCy instance using the username "admin" and the password you set up when installing.
7. Specifying ports in urls is lame. If you have already set up an [nginx server](../server/https-with-nginx.html), then you can simply set up a proxy. Inside the server block in `/etc/nginx/nginx.conf`, add:
   ```
   location /search/ {
   	rewrite ^/search/?(.*)$ /$1 break;    
   	proxy_pass  http://127.0.0.1:8090;
   	proxy_set_header Host $host;
   	proxy_set_header X-Real-IP $remote_addr;
   	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   }
   ```
   Restart nginx with `systemctl restart nginx.service`. Now, navigating to `subdomain.domain.com/search/` should bring up the YaCy search page!
8. In the YaCy admin pages there are many things that can be customized. You can set YaCy to share indexing data with the decentralized network, crawl pages, change the asthetics, and much more.
