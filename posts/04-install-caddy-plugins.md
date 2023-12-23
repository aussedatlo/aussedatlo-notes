---
title: Caddy Plugins
tags:
  - caddy
  - plugins
  - docker
date: 2023-12-22
---
# 📦 How to Install Caddy Plugins

**Caddy** is designed with a user-friendly architecture that facilitates the creation of plugins. The community actively contributes numerous plugins, which can be explored in the comprehensive list available [here](https://caddyserver.com/docs/modules/). These plugins enhance functionality, allowing users to, for instance, format logs and execute shell commands.

---
## Add a Plugin with CLI

**Caddy** is equipped with a command-line interface (**CLI**) that enables the seamless integration of plugins. To add a plugin, execute the following command:

```bash
xcaddy build \
    --with github.com/caddyserver/transform-encoder
```

> [!hint]- transform-encoder
> **transform-encoder** is a plugin that provides the capability to format entry logs with a distinct and customizable structure, useful for tools like **fail2ban** that will parse logs.

---
## Add a Plugin for Docker

To incorporate plugins into a Docker **Caddy** instance, the most effective approach is to rebuild the **Caddy** image by including the desired plugin.

Craft a `Dockerfile` based on `caddy:builder` and run `xcaddy`:

```docker
FROM caddy:builder AS builder

RUN xcaddy build \
	--with github.com/caddyserver/transform-encoder \
	--with github.com/abiosoft/caddy-exec

FROM caddy:latest

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

From my previous blog post [[01-caddy-in-docker]], you can edit the `docker-compose.yml` file:

```yml {5-8}
version: "3.9"
  
services:
 caddy:
   image: caddy-custom:1.0.0
   build:
     context: .
     dockerfile: ./Dockerfile
   container_name: caddy
   restart: unless-stopped
   ports:
     - "80:80"
     - "443:443"
     - "443:443/udp"
   volumes:  
     - ./Caddyfile:/etc/caddy/Caddyfile
     - ./data:/data
     - ./config:/config
  
networks:
 default:
   external:
     name: caddy
```

With these modifications, the `docker-compose.yml` file will now employ the customized image `caddy-custom:1.0.0`, which includes the desired plugins, in lieu of `caddy:latest`.

To restart the container with the new image, just run:

```bash
docker-compose down && docker-compose up -d
```

If you want to rebuild the image, use:

```bash
docker-compose rebuild
```

---
## Ressources
- [Caddy's Modules](https://caddyserver.com/docs/modules/)