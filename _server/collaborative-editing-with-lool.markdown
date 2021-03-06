---
title: "Collaborative Editing with LibreOffice Online"
desc: "Scroogle docks? Micro$oft 360? We can do that trick too!"
sortable: 5
---

[LibreOffice Online](https://wiki.documentfoundation.org/Development/LibreOffice_Online) is a web client for the wildly popular LibreOffice. The document foundation collaborates with [Collabora Office](http://collaboraoffice.com) in the development of LOOL (Libre Office On Line), and both sources distribute the software. Since it is currently under heavy development and suffers a heafty amount of cludge, we we cover the installation using docker.

1. Install Docker (instructions from [here](https://www.itzgeek.com/how-tos/linux/debian/how-to-install-docker-on-debian-9.html)).
   ```
   # apt install apt-transport-https ca-certificates wget software-properties-common
   # wget https://download.docker.com/linux/debian/gpg
   # apt-key add gpg
   # echo "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee -a /etc/apt/sources.list.d/docker.list
   # apt update
   # apt-cache policy docker-ce
   # apt install docker-ce
   ```
2. Use Docker to install Collabora CODE (stable) **XOR** Libreoffice Online master branch (cutting edge)
   ```
   # docker pull collabora/code
   OR
   # docker pull libreoffice/online:master
   ```
3. Start Collabora **XOR** LibreOffice Online.
   ```
   # docker run -t -d -p 127.0.0.1:9980:9980 -e 'domain=subdomain\\.domain\\.com' --restart always --cap-add MKNOD collabora/code
   OR
   # docker run -t -d -p 127.0.0.1:9980:9980 -e 'domain=subdomain\\.domain\\.com' --restart always --cap-add MKNOD libreoffice/online:master
   ```
   Notice the double-escaped .'s in the domain name. It's so that it'll be escaped by a [real backslash](https://www.xkcd.com/1638/).
4. Create the file `/etc/nginx/sites-available/lool` containing the following:
   ```
   server {
      listen       9979 ssl;
      server_name  subdomain.domain.com;

      ssl_certificate         /etc/letsencrypt/live/subdomain.domain.com/fullchain.pem;
      ssl_certificate_key     /etc/letsencrypt/live/subdomain.domain.com/privkey.pem;
    
      # static files
      location ^~ /loleaflet {
          proxy_pass https://localhost:9980;
          proxy_set_header Host $http_host;
      }

      # WOPI discovery URL
      location ^~ /hosting/discovery {
          proxy_pass https://localhost:9980;
          proxy_set_header Host $http_host;
      }

      # main websocket
      location ~ ^/lool/(.*)/ws$ {
          proxy_pass https://localhost:9980;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "Upgrade";
          proxy_set_header Host $http_host;
          proxy_read_timeout 36000s;
      }
   
      # download, presentation and image upload
      location ~ ^/lool {
          proxy_pass https://localhost:9980;
          proxy_set_header Host $http_host;
      }
   
      # Admin Console websocket
      location ^~ /lool/adminws {
          proxy_pass https://localhost:9980;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "Upgrade";
          proxy_set_header Host $http_host;
          proxy_read_timeout 36000s;
      }
   }
   ```
   The port number 9979 in the above block is deeply arbitrary (chosen because it's one less than 9980). The way this works is that LOOL cannot handle being installed to a subdirectory; it instead vomits up a css-free, js-free, office-free non-experience. Therefore, we install it as a reverse-proxy under nginx, so whenever someone visits port 9979 on your website, it is passed through by nginx to the proxy server listening on port 9980, which is itself inside a docker container. Nice and cozy, insulated in many layers of glop. And if you're installing all of this in a VM, that would be rather special now, wouldn't it!
   5. Symlink `/etc/nginx/sites-available/lool` to `/etc/nginx/sites-enabled/lool`
   ```
   # cd /etc/nginx/sites-enabled
   # ln -s ../sites-available/lool .
   ```
   6. Open tcp port 9979 up to the outside world (I like firewalld so that's what I support).
   ```
   # firewall-cmd --add-port=9979/tcp --permanent
   # firewall-cmd --reload
   ```
   7. Within your [nextcloud installation](file-sharing-with-nextcloud.html), install the Collabora Online app.
   8. Under the Collabora Online settings in your nextcloud settings pannel, set the Collabora Online server field to `https://subdomain.domain.com:9979`
   9. Open a document in the Nextcloud files app. Happy online editing!

If the above steps worked flawlessly, then you are far luckier than I. If not, and this isn't your first stab at getting collabora working, I would recommend deleting the autogenerated richdocuments xml:
```
# rm /path/to/nextcloud/data/appdata_*/richdocuments/richdocuments/discovery.xml
```
In addition to the above trick, I have experienced some [erratic feedback](https://www.xkcd.com/1457/) from re-applying the Collabora url:port settings in Nextcloud. But the main reason for my frustrations in getting LOOL fully operational has been that it takes some time to ititialize. If loolforkit and/or docker is sipping CPU cycles, that's a sign that it's busy doing... stuff. During this time it'll give you a bad gateway at best, and a nondescript server error at worst. If your approach to debugging is to restart the docker container, then you're in for a long fight.
