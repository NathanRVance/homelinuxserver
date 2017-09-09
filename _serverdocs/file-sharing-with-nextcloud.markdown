---
title: "File Sharing with Nextcloud"
desc: "Run a cushy replacement for Google Drive"
sortable: 4
---

[Nextcloud](https://nextcloud.com/) is a really nice drop in replacement for most of the functionality of Google Drive.

1. Install required PHP and SQL software.
   ```
   $ sudo apt install mariadb-server php7.0-fpm php7.0-xml php7.0 php7.0-cgi php7.0-cli php7.0-gd php7.0-curl php7.0-zip php7.0-mysql php7.0-mbstring wget unzip
   ```
2. Configure MariaDB.
   ```
   # mysql_secure_installation
   ```
   Add a database for nextcloud. In the following lines, you may replace "nextclouddb" with any name for the cloud database, "nextcloud" with any username for that database, and you should replace "password" with an actual password.
   ```
   # mysql -u root -p
   CREATE DATABASE nextclouddb;
   CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'password';
   GRANT ALL PRIVILEGES ON nextclouddb.* TO 'nextcloud'@'localhost';
   FLUSH PRIVILEGES;
   \q
   ```
3. Configure php. In the file `/etc/php/7.0/fpm/php.ini`, edit the section `[opcache]` to contain the following:
   ```
   opcache.enable=1
   opcache.enable_cli=1
   opcache.interned_strings_buffer=8
   opcache.max_accelerated_files=10000
   opcache.memory_consumption=128
   opcache.save_comments=1
   opcache.revalidate_freq=1
   ```
   In the file `/etc/php/7.0/fpm/pool.d/www.conf`, uncomment the following lines:
   ```
   env[HOSTNAME] = $HOSTNAME
   env[PATH] = /usr/local/bin:/usr/bin:/bin
   env[TMP] = /tmp
   env[TMPDIR] = /tmp
   env[TEMP] = /tmp
   ```
   Finally, restart php7.0-fpm:
   ```
   $ sudo systemctl restart php7.0-fpm.service
   ```
4. So that we don't pollute `nginx.conf`, create the file `/etc/nginx/conf.d/nextcloud.conf` containing the following:
   ```
   add_header X-Content-Type-Options nosniff;
   add_header X-XSS-Protection "1; mode=block";
   add_header X-Robots-Tag none;
   add_header X-Download-Options noopen;
   add_header X-Permitted-Cross-Domain-Policies none;

   # The following 2 rules are only needed for the user_webfinger app.
   # Uncomment it if you're planning to use this app.
   # rewrite ^/.well-known/host-meta /nextcloud/public.php?service=host-meta
   # last;
   #rewrite ^/.well-known/host-meta.json
   # /nextcloud/public.php?service=host-meta-json last;

   location = /.well-known/carddav {
   	return 301 $scheme://$host/nextcloud/remote.php/dav;
   }
   location = /.well-known/caldav {
   	return 301 $scheme://$host/nextcloud/remote.php/dav;
   }

   location /.well-known/acme-challenge { }

   location ^~ /nextcloud {
   	# set max upload size
   	client_max_body_size 512M;
   	fastcgi_buffers 64 4K;

   	# Enable gzip but do not remove ETag headers
   	gzip on;
   	gzip_vary on;
   	gzip_comp_level 4;
   	gzip_min_length 256;
   	gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
   	gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

   	location /nextcloud {
   		rewrite ^ /nextcloud/index.php$uri;
   	}

   	location ~ ^/nextcloud/(?:build|tests|config|lib|3rdparty|templates|data)/ {
   		deny all;
   	}
   	location ~ ^/nextcloud/(?:\.|autotest|occ|issue|indie|db_|console) {
   		deny all;
   	}

   	location ~ ^/nextcloud/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+)\.php(?:$|/) {                                                                       
   		fastcgi_split_path_info ^(.+\.php)(/.*)$;
   		include fastcgi_params;              
   		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
   		fastcgi_param PATH_INFO $fastcgi_path_info;
   		fastcgi_param HTTPS on;              
   		#Avoid sending the security headers twice
   		fastcgi_param modHeadersAvailable true;
   		fastcgi_param front_controller_active true;
   		fastcgi_pass php-handler;
   		fastcgi_intercept_errors on;
   		fastcgi_request_buffering off;
   	}

   	location ~ ^/nextcloud/(?:updater|ocs-provider)(?:$|/) {
   		try_files $uri/ =404;
   		index index.php;
   	} 

   	# Adding the cache control header for js and css files
   	# Make sure it is BELOW the PHP block
   	location ~ \.(?:css|js|woff|svg|gif)$ {
   		try_files $uri /nextcloud/index.php$uri$is_args$args;
   		add_header Cache-Control "public, max-age=15778463";
   		# Add headers to serve security related headers  (It is intended
   		# to have those duplicated to the ones above)
   		# Before enabling Strict-Transport-Security headers please read
   		# into this topic first.
      		# add_header Strict-Transport-Security "max-age=15768000;
   		# includeSubDomains; preload;";
   		add_header X-Content-Type-Options nosniff;
   		add_header X-XSS-Protection "1; mode=block";
   		add_header X-Robots-Tag none;
   		add_header X-Download-Options noopen;
      		add_header X-Permitted-Cross-Domain-Policies none;
   		access_log off;
   	}

   	location ~ \.(?:png|html|ttf|ico|jpg|jpeg)$ {
   		try_files $uri /nextcloud/index.php$uri$is_args$args;
   		access_log off;
   	}
   }
   ```
   In the http block of `/etc/nginx/nginx.conf`, add the following:
   ```
   upstream php-handler {
   	server unix:/var/run/php/php7.0-fpm.sock;
   }
   ```
   Also in `/etc/nginx/nginx.conf`, move the following line to the server block:
   ```
   include /etc/nginx/conf.d/*.conf;
   ```
   Finall, restart nginx.
   ```
   $ sudo systemctl restart nginx
   ```
5. Download and extract [nextcloud](https://nextcloud.org/install/) server. Replace X.X.X with the current version:
   ```
   $ curl -O https://download.nextcloud.com/server/releases/nextcloud-X.X.X.tar.bz2
   $ tar xjf nextcloud-X.X.X.tar.bz2
   $ sudo cp -a nextcloud/ /var/www/nextcloud/
   $ sudo chown -R www-data:www-data /var/www/nextcloud/
   ```
6. In your web browser, go to `https://subdomain.domain.com/nextcloud`. When configuring the database, use the username, password, and database name that you configured in step 2.
7. In the upper right corner, click the gear icon, and in the dropdown click `+ Apps`. Under the `Organization` tab you can find Calendar and oh so much more! Install Calendar and whatever else you want.
8. Navigate in your web browser to whatever calendar app you used to use (for me it's [Google Calendar](https://calendar.google.com/)) and export it. In Google Calendar, you have to go under Settings -> Calendars -> Export calendars.
9. Back in Nextcloud, import your calendar.
10. If you want to mount your cloud directory with webdav, connect to:
    ```
    davs://subdomain.domain.com/nextcloud/remote.php/webdav
    ```

Before ammassing a significant amount of stuff, configure [backups](backup-everything-always.html).
