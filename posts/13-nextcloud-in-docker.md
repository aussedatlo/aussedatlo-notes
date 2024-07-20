---
title: Nextcloud in Docker
tags:
  - nextcloud
  - docker
  - caddy
date: 2024-06-23
description: How to configure your pesonal Home server with Nextcloud, Caddy and Docker
icon: ☁️
---
> [!warning] Work in progress

---

## Intro

### What is nextcloud ?


---
## Prerequisite

- Docker


---
## Installation

Create the `nextcloud` folder

```bash
mkdir nextcloud && cd nextcloud
```

Create the `.env` file and add the content below:

```
MYSQL_ROOT_PASSWORD=<root_password>
MYSQL_USER=nextcloud
MYSQL_PASSWORD=<mysql_password>
MYSQL_DATABASE=nextcloud
MYSQL_HOST=db
REDIS_HOST=redis
```

make sure to replace `<root_password>`, `<mysql_password>` and `<hostname>`.

Create a `docker-compose.yml` file with the content :
```yml
services:
  db:
    image: mariadb:10.11
    container_name: mariadb-database
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - ./data/db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
    restart: unless-stopped

  redis:
    image: redis:alpine
    container_name: redis-dbcache
    restart: unless-stopped
      
  nextcloud:
    image: nextcloud
    container_name: nextcloud-app
    ports:
      - 8080:80
    volumes:
      - ./data/app:/var/www/html:z
    environment:
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
      - MYSQL_HOST
      - REDIS_HOST
    restart: unless-stopped
    depends_on:
      - db
      - redis

  cron:
    image: nextcloud
    container_name: nextcloud-cron
    volumes:
      - ./data/app:/var/www/html:z
    entrypoint: /cron.sh
    restart: unless-stopped
    depends_on:
      - db
      - redis
```

Start the container by running
```bash
docker compose up -d
```

You can now access the setup page at the url https://localhost:8080

---
## Configure HTTPS

To configure HTTPS protocol, and access securely our server, we will use [caddy]() as a reverse proxy.

Check out my guide [[01-caddy-in-docker]] to see how to configure [caddy]() with docker.

You can add the basic configuration below in your `Caddyfile` :

```text
cloud.domain.name {
    reverse_proxy http://nextcloud-app:80
}
```

If you use a wildcard certificate like explained in this guide [[09-caddy-wildcard-certificates]], you can use this configuration :

```text
@nextcloud host cloud.domain.name
handle @nextcloud {
    reverse_proxy http://nextcloud-app:80
}
```

Add the required environment variable used for reverse proxying http requests :

```txt {7-10}
MYSQL_ROOT_PASSWORD=<root_password>
MYSQL_USER=nextcloud
MYSQL_PASSWORD=<mysql_password>
MYSQL_DATABASE=nextcloud
MYSQL_HOST=db
REDIS_HOST=redis
OVERWRITEPROTOCOL=https
TRUSTED_PROXIES=<caddy_container_ip>
APACHE_DISABLE_REWRITE_IP=1
OVERWRITEHOST=cloud.domain.name
```

> [!note]
> You can get the ip of the caddy container ip with the command `docker exec -it caddy ifconfig eth0 | awk -F ' *|:' '/inet addr/{print $4}'`

, edit the 

Then, edit the the nextcloud `docker-compose.yml` to add the  `caddy` network :

```yml {5-6} {20-21} {27-29} {38-41} {50-51} {60-64}
services:
  db:
    image: mariadb:10.11
    container_name: mariadb-database
    networks:
      - internal
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - ./data/db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
    restart: unless-stopped

  redis:
    image: redis:alpine
    container_name: redis-dbcache
    networks:
      - internal
    restart: unless-stopped

  nextcloud:
    image: nextcloud
    container_name: nextcloud-app
    networks:
      - caddy
      - internal
    volumes:
      - ./data/app:/var/www/html:z
    environment:
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
      - MYSQL_HOST
      - REDIS_HOST
      - OVERWRITEPROTOCOL
      - TRUSTED_PROXIES
      - APACHE_DISABLE_REWRITE_IP
      - OVERWRITEHOST
    restart: unless-stopped
    depends_on:
      - db
      - redis

  cron:
    image: nextcloud
    container_name: nextcloud-cron
    networks:
      - internal
    volumes:
      - ./data/app:/var/www/html:z
    entrypoint: /cron.sh
    restart: unless-stopped
    depends_on:
      - db
      - redis

networks:
  caddy:
    name: caddy
    external: true
  internal: {}
```

Here, only the `nextcloud-app` need to be accessed by caddy. All the other services are isolated into an internal network, to avoid confrontation with other services. For example an other `redis` instance.

> [!note] Note
> You can also remove the port mapping since the service will be accessible with the subdomain.

Now you can restart the nextcloud service using the command :

```
docker compose down && docker compose up -d
```

You can now access your nextcloud instance using `https://cloud.domain.name`!

## Add SMTP

SMTP settings are used to send mail to users using the nextcloud service.

> [!note] Note
> In this example I will be using SMTP service with a gmail address. You should be able to use it with a different email provider compatible with SMTP.


To configure the mail service, add in the `.env` file the environment variable below :

```txt {11-17}
MYSQL_ROOT_PASSWORD=<root_password>
MYSQL_USER=nextcloud
MYSQL_PASSWORD=<mysql_password>
MYSQL_DATABASE=nextcloud
MYSQL_HOST=db
REDIS_HOST=redis
OVERWRITEPROTOCOL=https
TRUSTED_PROXIES=<caddy_container_ip>
APACHE_DISABLE_REWRITE_IP=1
OVERWRITEHOST=cloud.domain.name
SMTP_HOST=smtp.gmail.com
SMTP_SECURE=ssl
SMTP_PORT=465
SMTP_NAME=address@gmail.com
SMTP_PASSWORD=secure password
MAIL_FROM_ADDRESS=from
MAIL_DOMAIN=cloud.domain.name
```

> [!warning] Warning
> If you have 2FA enabled on your gmail account, you can't use your regular password with gmail. You will need to generate an app password like described here: https://support.google.com/accounts/answer/185833?hl=en


The final docker-compose.yml file should look like this :

```yml {44-50}
services:
  db:
    image: mariadb:10.11
    container_name: mariadb-database
    networks:
      - internal
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - ./data/db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
    restart: unless-stopped

  redis:
    image: redis:alpine
    container_name: redis-dbcache
    networks:
      - internal
    restart: unless-stopped

  nextcloud:
    image: nextcloud
    container_name: nextcloud-app
    networks:
      - caddy
      - internal
    ports:
      - 8080:80
    volumes:
      - ./data/app:/var/www/html:z
    environment:
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
      - MYSQL_HOST
      - REDIS_HOST
      - OVERWRITEPROTOCOL
      - TRUSTED_PROXIES
      - APACHE_DISABLE_REWRITE_IP
      - OVERWRITEHOST
      - SMTP_HOST
      - SMTP_SECURE
      - SMTP_PORT
      - SMTP_NAME
      - SMTP_PASSWORD
      - MAIL_FROM_ADDRESS
      - MAIL_DOMAIN
    restart: unless-stopped
    depends_on:
      - db
      - redis

  cron:
    image: nextcloud
    container_name: nextcloud-cron
    networks:
      - internal
    volumes:
      - ./data/app:/var/www/html:z
    entrypoint: /cron.sh
    restart: unless-stopped
    depends_on:
      - db
      - redis

networks:
  caddy:
    name: caddy
    external: true
  internal: {}

```

You should see the correct settings in the admin panel here `https://cloud.domain.name/settings/admin`.

---
## Ressources

- Docker image: https://hub.docker.com/_/nextcloud/
- Caddy documentation: https://caddyserver.com/docs/
- Google App password: https://support.google.com/accounts/answer/185833?hl=en
- Photo by [CHUTTERSNAP](https://unsplash.com/@chuttersnap?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/blue-clouds-under-white-sky-9AqIdzEc9pY?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)