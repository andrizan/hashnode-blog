---
title: "Server Config Laravel & Lumen"
datePublished: Mon Aug 14 2023 08:33:15 GMT+0000 (Coordinated Universal Time)
cuid: cllamavm9000i0ahz48rk4yzc
slug: server-config-laravel-lumen

---

## Laravel

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

## Lumen

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