---
title: "Install Latest Nginx Ubuntu Server"
datePublished: Mon Aug 14 2023 07:40:04 GMT+0000 (Coordinated Universal Time)
cuid: cllakehfp000509mnf1158dqo
slug: install-latest-nginx-ubuntu-server
canonical: https://www.nginx.com/resources/wiki/start/topics/tutorials/install/
tags: nginx

---

## Official Debian/Ubuntu packages

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

doc : [https://www.nginx.com/resources/wiki/start/topics/tutorials/install/](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)