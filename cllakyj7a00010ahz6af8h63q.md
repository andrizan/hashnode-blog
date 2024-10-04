---
title: "Install PHP-FPM PostgreSQL Ubuntu Server"
datePublished: Mon Aug 14 2023 07:55:40 GMT+0000 (Coordinated Universal Time)
cuid: cllakyj7a00010ahz6af8h63q
slug: install-php-fpm-postgresql-ubuntu-server
tags: postgresql, php, fpm

---

## Install the PostgreSQL Database Server Package

1. Add the PostgreSQL repository to your server's APT sources.
    
    ```bash
    $ sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
    ```
    
2. Import the PostgreSQL repository key to your server using the `wget` utility.
    
    ```bash
    $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo tee /etc/apt/trusted.gpg.d/postgresql.asc > /dev/null
    ```
    
3. Update the server packages to synchronize the new PostgreSQL repository.
    
    ```bash
    $ sudo apt update
    ```
    
4. Install PostgreSQL on your server.
    
5. ```bash
       $ sudo apt install postgresql-15
    ```
    
6. Start the PostgreSQL database server.
    
    ```bash
    $ sudo systemctl restart postgresql
    ```
    
7. View the PostgreSQL service status and verify that it's active.
    
    ```bash
    $ sudo systemctl status postgresql
    ```
    
    Output:
    
    ```bash
    ● postgresql.service - PostgreSQL RDBMS
         Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
         Active: active (exited) since Sun 2024-04-21 16:08:10 UTC; 13s ago
        Process: 6756 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
       Main PID: 6756 (code=exited, status=0/SUCCESS)
            CPU: 1ms
    ```
    

## PHP

**ALL IN ONE CODE**

```bash
sudo apt update && sudo apt upgrade -y
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

Besides PHP itself, you will likely want to install some additional PHP modules. You can use this command to install additional modules, replacing `PACKAGE_NAME` with the package you wish to install:

```bash
sudo apt-get install php8.2-PACKAGE_NAME
```

You can also install more than one package at a time. Here are a few suggestions of the most common modules you will most likely want to install:

```bash
sudo apt-get install -y php8.2-cli php8.2-common php8.2-zip php8.2-gd php8.2-mbstring php8.2-curl php8.2-xml php8.2-bcmath php8.2-pgsql
```

This command will install the following modules:

* `php8.2-cli` – command interpreter, useful for testing PHP scripts from a shell or performing general shell scripting tasks
    
* `php8.2-common` – documentation, examples, and common modules for PHP
    
* `php8.2-zip` – for working with compressed files
    
* `php8.2-gd` – for working with images
    
* `php8.2-mbstring` – used to manage non-ASCII strings
    
* `php8.2-curl` – lets you make HTTP requests in PHP
    
* `php8.2-xml` – for working with XML data
    
* `php8.2-bcmath` – used when working with precision floats
    
* `php8.2-pgsql` – for working with PostgreSQL databases
    

---

**src** :

* [https://josuamarcelc.com/install-php-8-2-on-ubuntu-with-nginx/](https://josuamarcelc.com/install-php-8-2-on-ubuntu-with-nginx/)
    
* [https://docs.vultr.com/how-to-install-postgresql-database-server-on-ubuntu-22-04](https://docs.vultr.com/how-to-install-postgresql-database-server-on-ubuntu-22-04)