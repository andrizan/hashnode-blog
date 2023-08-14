---
title: "Install Nginx, PHP for  Laravel/Lumen Ubuntu Server"
datePublished: Mon Aug 14 2023 09:04:19 GMT+0000 (Coordinated Universal Time)
cuid: cllanetsc001708jqhu4af9mw
slug: install-nginx-php-for-laravellumen-ubuntu-server
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1692006595680/7d39ef6d-90c0-4b28-9559-3d68c6545294.png
tags: laravel, ubuntu-server, laravel-lumen, lnpp

---

## Install Latest Nginx Ubuntu Server

### Official Debian/Ubuntu packages

The available NGINX Ubuntu release support is listed at [this distribution page](https://nginx.org/packages/ubuntu/dists/). For a mapping of Ubuntu versions to release names, please visit the [Official Ubuntu Releases page](https://wiki.ubuntu.com/Releases).

Append the appropriate stanza to `/etc/apt/sources.list`. If there is concern about persistence of repository additions (i.e. DigitalOcean Droplets), the appropriate stanza may instead be added to a different list file under `/etc/apt/sources.list.d/`, such as `/etc/apt/sources.list.d/nginx.list`.

```bash
## Replace $release with your corresponding Ubuntu release.
deb https://nginx.org/packages/ubuntu/ $release nginx
deb-src https://nginx.org/packages/ubuntu/ $release nginx
```

e.g. Ubuntu 20.04 (Focal Fossa):

```bash
deb https://nginx.org/packages/ubuntu/ focal nginx
deb-src https://nginx.org/packages/ubuntu/ focal nginx
```

To install the packages, execute in your shell:

```bash
sudo apt update
sudo apt install nginx
```

If a `W: GPG error:` [`https://nginx.org/packages/ubuntu`](https://nginx.org/packages/ubuntu) `focal InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY $key` is encountered during the NGINX repository update, execute the following:

```bash
## Replace $key with the corresponding $key from your GPG error.
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys $key
sudo apt update
sudo apt install nginx
```

You have now nginx installed on your server but not ready to serve web pages. you have to start the nginx. You can do this by using this command:

```bash
sudo systemctl start nginx
```

---

## Install PHP-FPM PostgreSQL Ubuntu Server

**ALL IN ONE CODE**

```bash
sudo apt update && apt upgrade -y
sudo add-apt-repository ppa:ondrej/php #private repo
sudo apt update
sudo apt-get install -y php8.2-fpm php8.2-cli php8.2-common php8.2-zip php8.2-gd php8.2-mbstring php8.2-curl php8.2-xml php8.2-bcmath php8.2-pgsql
```

### 1\. Run system updates

The first thing to do in a new system is to update our repositories in order to make them up to date. Run upgrade command also.

```bash
sudo apt update && apt upgrade -y
```

### 2\. Add Ondrej sury PPA repository

To run PHP 8.2 on Ubuntu 22.04, we need to add Ondrej sury PPA into our system. This is the maintainer of the PHP repository at the moment. This PPA is not currently checked so installing from it will not be guaranteed 100% results.

To add this PPA use the following command on our terminal.

```bash
sudo add-apt-repository ppa:ondrej/php
```

After installation is complete we need to update the repositories again for the changes to take effect.

```bash
sudo apt update
```

### 3\. Install PHP 8.2 on Ubuntu Server

We should now be able to install PHP 8.2 on Ubuntu 22.04 Linux machine. The commands to run are as shared below:

```bash
sudo apt install php8.2-fpm -y
```

Check for the currently active version of PHP with the following command:

```bash
php --version
```

### 4\. Install PHP 8.2 Extensions

Besides PHP itself, you will likely want to install some additional PHP modules. You can use this command to install additional modules, replacing `PACKAGE_NAME` with the package you wish to install:

```bash
sudo apt-get install php8.2-PACKAGE_NAME
```

You can also install more than one package at a time. Here are a few suggestions of the most common modules you will most likely want to install:

```bash
sudo apt-get install -y php8.2-cli php8.2-common php8.2-zip php8.2-gd php8.2-mbstring php8.2-curl php8.2-xml php8.2-bcmath php8.2-pgsql
```

This command will install the following modules:

* `php8.2-cli` – command interpreter, useful for testing PHP scripts from a shell or performing general shell scripting tasks
    
* `php8.2-common` – documentation, examples, and common modules for PHP
    
* `php8.2-zip` – for working with compressed files
    
* `php8.2-gd` – for working with images
    
* `php8.2-mbstring` – used to manage non-ASCII strings
    
* `php8.2-curl` – lets you make HTTP requests in PHP
    
* `php8.2-xml` – for working with XML data
    
* `php8.2-bcmath` – used when working with precision floats
    
* `php8.2-pgsql` – for working with PostgreSQL databases
    

---

## Install Composer Ubuntu Server

```bash
sudo apt-get install curl php8.2-cli php8.2-mbstring git unzip
curl –sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer -V
```

---

## Server Config Laravel & Lumen

### Laravel

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }

    server_tokens off; # for production

    access_log  /var/log/nginx/app_access.log;
    error_log  /var/log/nginx/app_error.log crit;
}
```

### Lumen

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /var/www/lumen/public;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    server_tokens off; # for production

    access_log  /var/log/nginx/app_access.log;
    error_log  /var/log/nginx/app_error.log crit;
}
```

---

## Set Permission

```bash
cd /var/www/
sudo chown www-data:www-data <DIR>
sudo chmod g+w <DIR>
```

set user nginx

```bash
sudo nano /etc/nginx/nginx.conf
```

Before

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;
...
```

After

```nginx
user  www-data;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;
...
```

restart `nginx` and `php-fpm`

```bash
sudo systemctl restart nginx
sudo systemctl restart php8.2-fpm
```

src:

* [https://www.nginx.com/resources/wiki/start/topics/tutorials/install/](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)
    
* [https://josuamarcelc.com/install-php-8-2-on-ubuntu-with-nginx/](https://josuamarcelc.com/install-php-8-2-on-ubuntu-with-nginx/)