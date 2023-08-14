---
title: "Install Composer Ubuntu Server"
datePublished: Mon Aug 14 2023 08:04:28 GMT+0000 (Coordinated Universal Time)
cuid: cllal9v2a000109l74wdygb64
slug: install-composer-ubuntu-server
tags: composer

---

```bash
sudo apt-get install curl php8.2-cli php8.2-mbstring git unzip
curl –sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer -V
```