---
title: "Host your own source code"
desc: "Why use github when you can do it yourself?"
sortable: 6
---

Command line git is great, but it's nice to have a web interface for showing off a project. We install [GitLab](https://about.gitlab.com/). While I have my doubts regarding their [open core](https://en.wikipedia.org/wiki/Open_core) business model and what that may mean for the long term viability of their currently FOSS community edition, GitLab is still an excellent alternative to the Microsoft owned GitHub.

Instructions for setting up the repository are from [here](https://about.gitlab.com/installation/#debian) in case they change.

1. Install some dependencies and gitlab-ce:
   ```
   $ sudo apt install curl openssh-server ca-certificates
   $ curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
   $ sudo EXTERNAL_URL="http://subdomain.domain.com" apt-get install gitlab-ce
   ```
2. Ensure that the file `/etc/gitlab/gitlab.rb` contains the following lines:
   ```
   external_url 'http://subdomain.domain.com/code'
   web_server['external_users'] = ['www-data']
   nginx['enable'] = false
   ```
   If you modifify the subdirectory /code in the external\_url field, make sure you do the same in subsequent instructions. Of worthy note is that we purposefully made the installation http rather than https (it'll still be encrypted on the dangerous end of the proxy server). Also, we disable the built-in nginx server because we will handle it ourselves.
3. Commit the gitlab configuration:
   ```
   $ sudo gitlab-ctl reconfigure
   ```
4. Add the following to `/etc/nginx/conf.d/gitlab.conf`:
   ```
   location /code {
       root /opt/gitlab/embedded/service/gitlab-rails/public;
       client_max_body_size 0;
       gzip off;
       ## https://github.com/gitlabhq/gitlabhq/issues/694
       ## Some requests take more than 30 seconds.
       proxy_read_timeout      300;
       proxy_connect_timeout   300;
       proxy_redirect          off;
       proxy_http_version 1.1;
       proxy_set_header    Host                $host;
       proxy_set_header    X-Real-IP           $remote_addr;
       proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
       proxy_set_header    X-Forwarded-Proto   http;
       proxy_pass http://gitlab-workhorse;
   }
   ```
   AND add the following to `/etc/nginx/nginx.conf`, inside the http block but outside of the server block:
   ```
   upstream gitlab-workhorse {
       server unix:/var/opt/gitlab/gitlab-workhorse/socket fail_timeout=0;
   }
   ```
5. Restart everybody
   ```
   # systemctl restart nginx.service
   # gitlab-ctl restart
   ```
6. Connect to https://subdomain.domain.com/code. It will prompt you to set a password. After setting it, log in with username `root` and the password you configured.
