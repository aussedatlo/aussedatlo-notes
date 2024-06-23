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
- Caddy

TEMPORARY


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
OVERWRITEPROTOCOL=https
TRUSTED_PROXIES=caddy
APACHE_DISABLE_REWRITE_IP=1
OVERWRITEHOST=<hostname>
```

make sure to replace `<root_password>`, `<mysql_password>` and `<hostname>`.

Create `caddy` network:
```bash
docker network create caddy
```


docker compose 
```yml
services:
  db:
    image: mariadb:10.11
    container_name: mariadb-database
    command: --transaction-isolation=READ-COMMITTED --log-bin=binlog --binlog-format=ROW
    volumes:
      - ./db_data:/var/lib/mysql
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
      - 8080:8080
    volumes:
      - ./data:/var/www/html:z
    environment:
      - MYSQL_USER
      - MYSQL_PASSWORD
      - MYSQL_DATABASE
      - MYSQL_HOST
      - REDIS_HOST
      - OVERWRITEPROTOCOL
      - OVERWRITEHOST
      - TRUSTED_PROXIES
      - APACHE_DISABLE_REWRITE_IP
    restart: unless-stopped
    depends_on:
      - db
      - redis

  cron:
    image: nextcloud:stable-fpm
    container_name: nextcloud-cron
    networks:
      - caddy
    volumes:
      - ./data:/var/www/html:z
    entrypoint: /cron.sh
    restart: unless-stopped
    depends_on:
      - db
      - redis
```

---
## Ressources
