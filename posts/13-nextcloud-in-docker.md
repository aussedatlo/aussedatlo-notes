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
      - ./php-fpm-www.conf:/usr/local/etc/php-fpm.d/www.conf:ro
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

Check out my guides [[01-caddy-in-docker]], [[02-caddy-hardening]] and [[09-caddy-wildcard-certificates]] for all the possible configurations.

You can add the basic configuration below in your `Caddyfile` :

```text
cloud.domain.name {
    reverse_proxy http://nextcloud-app:8080
}
```

Now, add the `nextcloud-app` container to the  `caddy` network

```

```

---
## Ressources
