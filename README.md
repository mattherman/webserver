# webserver

This repository includes documentation and files related to the configuration of my web server. The main thing hosted on the server is my [website](https://www.github.com/mattherman/website.git).

## Setup

### Droplet

The server runs Ubuntu 22.04 on a Digital Ocean droplet. I don't remember all of the steps I followed with this initial setup, but I likely followed a guide like [this](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu). If I set this up again, I'll be sure to document it fully.

### NGINX

The server uses NGINX to serve my website as well as to act as a reverse proxy for other applications. Comprehensive documentation can be found on the [NGINX website](https://nginx.org/en/docs/).

The NGINX configuration files are located in `/etc/nginx`. The main configuration file is `/etc/nginx/nginx.conf`, which should include other configuration files in the `sites-available` directory like this:
```
include /etc/nginx/sites-enabled/*;
```

The `sites-enabled` directory includes sub-directories for each specific site. These are symlinks to the `sites-available/*` sub-directories. There is a `default` site and one for my domain, `matthewherman.net`.

The `sites-available/matthewherman.net` file should contain basic configuration to serve my website from `/var/www/matthewherman.net/html` as well as SSL setup and HTTPS redirects added by Certbot / Let's Encrypt. Here is what it currently looks like (with other applications removed for simplicity):
```
server {

        root /var/www/matthewherman.net/html;
        index index.html index.htm index.nginx-debian.html;

        server_name matthewherman.net www.matthewherman.net;

        location / {
                try_files $uri $uri/ =404;
        }

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/matthewherman.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/matthewherman.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = www.matthewherman.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = matthewherman.net) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        listen [::]:80;

        server_name matthewherman.net www.matthewherman.net;
    return 404; # managed by Certbot




}
```

Any sites served directly by NGINX should be in a sub-directory of `/var/www` (eg., `/var/www/matthewherman.net`) which is owned by the `www-data` user.

Routes to other applications can be added using additional `location` blocks, but usually these will be added to a subdomain instead. For any application I've written myself, see those projects' README files for details on configuration.

### SSL/TLS

The server uses [Let's Encrypt](https://letsencrypt.org/) with [Certbot](https://certbot.eff.org/) to provision and manage certificates. I likely followed [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-22-04) for the initial setup.

### Docker

The server has docker installed in order to run containerized applications. I followed [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04) to install it.

## Applications

### Actual Budget

The server hosts an [Actual Budget](https://actualbudget.org/) server for managing personal finance. It uses Docker Compose to run the official Docker container. The [`actual/docker-compose.yml`](actual/docker-compose.yml) file in this repository contains the configuration. The file was originally sourced from the [`actualbudget/actual-server`](https://github.com/actualbudget/actual-server/blob/master/docker-compose.yml) repository and modified.

While not directly served by NGINX, for simplicity I have placed it in the `/var/www/budget` directory.

This directory should also include a `data` directory which is mounted as a volume in the compose file.

To update and run the application, run `docker compose pull && docker compose up -d`. The application is configured to run on port 5006.
