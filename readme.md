# ICassava Wiki

This document contains all the necessary information to create icassava wiki. 📚

- [Dependencies](#Dependencies)
- [Configurations](#Configurations)
- [Backup](#Backup)
- [Help](#Help)

## Dependencies

### Core

| Name            | Version                                                                   | Note                                                                             |
| --------------- | ------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Mediawiki       | [1.36.1](https://releases.wikimedia.org/mediawiki/1.36/)                  | Not compatible with PHP 7.3.0 - 7.3.18 and 7.4.0 - 7.4.8 due to an upstream bug. |
| php with mysqli | [Use this Docker Image](https://hub.docker.com/r/petekaik/php-fpm-mysqli) | Run php with docker                                                              |
| Nginx           | Any Version (Assumed to be installed)                                     | Current configuration: [nginx.conf](#TODO add link here)                         |
| MySql           | Any Version (Assumed to be installed)                                     |

### Extensions

| Name            | Version                                                                                 | LocalSettings.php                                                                 |
| --------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Visual Editor   | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:VisualEditor)    | wfLoadExtension( 'VisualEditor' );                                                |
| Cite This Page  | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:CiteThisPage)    | wfLoadExtension( 'CiteThisPage' );                                                |
| Cite            | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:Cite)            | wfLoadExtension( 'Cite' );                                                        |
| ParserFunctions | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:ParserFunctions) | wfLoadExtension( 'ParserFunctions' ); $wgPFEnableStringFunctions = true;          |
| Scribunto       | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:Scribunto)       | wfLoadExtension( 'Scribunto' ); <br> $wgScribuntoDefaultEngine = 'luastandalone'; |
| TemplateData    | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:TemplateData)    | wfLoadExtension( 'TemplateData' );                                                |
| TextExtracts    | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:TextExtracts)    | wfLoadExtension( 'TextExtracts' );                                                |
| PageImages      | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Extension:PageImages)      | wfLoadExtension( 'PageImages' );                                                  |
| Popups          | [ddf574a](https://www.mediawiki.org/wiki/Extension:Popups)                              | wfLoadExtension( 'Popups' );                                                      |
| MobileFrontend  | [2.3.0](https://www.mediawiki.org/wiki/Extension:MobileFrontend)                        | wfLoadExtension( 'MobileFrontend' );                                              |

## Skins

| Name         | Version                                                                   | LocalSettings.php                                                        |
| ------------ | ------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| Vector       | [Comes with Mediawiki 1.36.1](https://www.mediawiki.org/wiki/Skin:Vector) | wfLoadSkin( 'Vector' ); <br> $wgDefaultSkin = "Vector";                  |
| Minerva Neue | [20711d8](https://www.mediawiki.org/wiki/Skin:Minerva_Neue)               | wfLoadSkin( 'MinervaNeue' ); <br> $wgMFDefaultSkinClass = 'SkinMinerva'; |

## Backup Tools

| Name   | Version                                     | Note                                         |
| ------ | ------------------------------------------- | -------------------------------------------- |
| gdrive | [any](https://github.com/prasmussen/gdrive) | For uploading database backup to googledrive |

# Installation

## Wiki

1. Download Mediawiki to the server (named as wiki at `/var/www/html` for the current project )

2. Serve the wiki with webserver (see nginx configuration)

3. Follow mediawiki setup page then download the LocalSettings.php

4. Put the LocalSettings.php in mediawiki folder (PATH in the current project: `/var/www/html/wiki/`)

## Extensions

1. Download and place at ${PATH TO WIKI}/extensions (current path is `/var/www/html/wiki/extensions`)

2. Add `wfLoadExtension( 'Extension Name' );` to LocalSettings.php

## Skins

1. Download and place at ${PATH TO WIKI}/skins (current path is `/var/www/html/wiki/skins`)

2. Add `wfLoadSkin( 'Skin Name' );` to LocalSettings.php

## GDrive

1. Download gdrive

2. Run `gdrive about` then follow the login process to get the verification code

3. Enter verification code

Note: Only first time

# Configurations

## Docker-Compose

- Current docker-compose.yaml file

  ```
  version: '3'
  services:
    frontend:
        container_name: bpbiz_nginx
        restart: always
        image: staticfloat/nginx-certbot
        ports:
            - 80:80/tcp
            - 443:443/tcp
            - 9000:9000/tcp
        depends_on:
            - php-wiki
        environment:
            CERTBOT_EMAIL: YOUR EMAIL
            ENVSUBST_VARS: FQDN
            FQDN: YOUR DOMAIN
        volumes:
            - ./conf.d:/etc/nginx/user.conf.d:ro
            - letsencrypt:/etc/letsencrypt
            - /var/www/html:/var/www/html
            - /var/log/nginx:/var/log/nginx

    mysql5:
        container_name: bpbiz_mysql5
        restart: always
        image: mysql:5.7.24
        command: --default-authentication-plugin=mysql_native_password
        volumes:
            - /var/mysql5:/var/lib/mysql
            - /home:/home
        ports:
            - 3306:3306
        environment:
            - MYSQL_ROOT_PASSWORD=ROOTPASSWORD
            - MYSQL_DATABASE=DBNAME
            - MYSQL_USER=USER
            - MYSQL_PASSWORD=PASSWORD

    php:
        container_name: bpbiz_php
        restart: always
        build: php
        depends_on:
            - mysql5
        expose:
            - "9000"
        volumes:
            - /var/www/html:/var/www/html

    php-wiki:
        container_name: php-wiki
        restart: always
        image: petekaik/php-fpm-mysqli
        volumes:
            - /var/www/html:/var/www/html
        expose:
            - "9000"
        stdin_open: true
        tty: true

  volumes:
    letsencrypt:
  ```

- Current Nginx Config

  ```
  server {
    listen              443 ssl;
    server_name         uknowcoe.com;
    ssl_certificate     /etc/letsencrypt/live/uknowcoe.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/uknowcoe.com/privkey.pem;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;

    client_max_body_size 100M;

    root /var/www/html;
    index       index.php;

    location / {
        # Redirect everything that isn't a real file to index.php
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location /dev {
        try_files $uri $uri/ /dev/index.php;
    }

    location ~ /wiki/rest.php/ {
        try_files $uri $uri/ /wiki/rest.php?$query_string;
    }

    location /wiki {
       try_files $uri $uri/ /wiki/index.php;
    }

    location ~ /wiki/(.+)\.php(/|$) {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass   php-wiki:9000;
        try_files $uri =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass   php:9000;
        try_files $uri =404;
    }
  }
  ```

  The wiki relevant parts are

  ```
  location ~ /wiki/rest.php/ {
      try_files $uri $uri/ /wiki/rest.php?$query_string;
  }

  location /wiki {
      try_files $uri $uri/ /wiki/index.php;
  }

  location ~ /wiki/(.+)\.php(/|$) {
      include fastcgi_params;
      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
      fastcgi_pass   php-wiki:9000;
      try_files $uri =404;
  }
  ```

- Current Extensions and Skins in LocalSettings.php file

  ```
  #Skin

  ##Desktop Skin
  wfLoadSkin( 'Vector');
  $wgDefaultSkin = "Vector";

  ##Mobile Skin
  wfLoadSkin( 'MinervaNeue' );
  $wgMFDefaultSkinClass = 'SkinMinerva';

  #Extension

  ##Editor
  wfLoadExtension( 'VisualEditor' );

  ## Citation
  wfLoadExtension( 'Cite' );
  wfLoadExtension( 'CiteThisPage' );
  wfLoadExtension( 'Scribunto' );
  $wgScribuntoDefaultEngine = 'luastandalone';
  wfLoadExtension( 'ParserFunctions' );
  wfLoadExtension( 'TemplateData' );

  ## Mobile Page
  wfLoadExtension( 'MobileFrontend' );

  ## Content Preview
  wfLoadExtension( 'TextExtracts' );
  wfLoadExtension( 'PageImages' );
  wfLoadExtension( 'Popups' );
  ```

- Mail (gmail) Setting in LocalSettings.php file
  ```
  $wgSMTP = [
    'host' => 'ssl://smtp.gmail.com', // hostname of the email server
    'IDHost' => 'gmail.com',
    'port' => 465,
    'username' => 'email', // user of the email account
    'password' => 'app password', // app password of the email account
    'auth' => true
  ];
  ```

## Backup

- Database Backup

  Backup Script

  ```
  #!/bin/bash

  docker exec mysql-container-name mysqldump --user=username --password=password databasename > $(date +%d-%m-%y)-dbwiki-backup.sql

  gdrive upload --parent folder-ID $(date +%d-%m-%y)-dbwiki-backup.sql
  ```

  `1Uwtc42RsZ7oGFBCx6I05hcMevx1\_-iPD` is the ID of google drive database-backup folder (kuicassava@gmail.com's google drive)

  Cron Expression  
   Automatically backup at midnight of 1st and 16th of every month

  ```
  0 0 1 * * /path to script/backup-wiki-database.sh
  0 0 16 * * /path to script/backup-wiki-database.sh
  ```

- Image Backup

  Backup Script

  ```
  #!/bin/bash

  gdrive upload -r --parent folder-ID /path to wiki/images

  ```

  `1mCWFJq7WO-wtz2hkT0AVsxJohO9YOBZB` is the ID of google drive wiki-images-backup folder (kuicassava@gmail.com's google drive)

  Cron Expression  
  Automatically backup at midnight every 3 months

  ```
  0 0 01 Jan,Apr,Jul,Oct \* /home/korn/backup-wiki-image.sh
  ```

## Help

For more information, contact siwatponpued@gmail.com.
