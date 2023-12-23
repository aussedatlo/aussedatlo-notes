---
title: Caddy in Docker
tags:
  - caddy
  - docker
  - reverse-proxy
date: 2023-12-16
---
# 🛒 What is Caddy

**Caddy v2**, a server-of-servers, is a Go application that is **fully open source**,
capable of functioning both as a **server** and a **reverse proxy**.

With no runtime dependencies, it operates seamlessly on all major platforms,
embodying a versatile solution that ensures reliable performance across diverse environments.

**Caddy**'s default protocol is **HTTPS**,
it will automatically handle all certificates using issuers like [ZeroSSL](https://zerossl.com/) or
[Let's Encrypt](https://letsencrypt.org/) for example.

---
## Run Caddy with Docker

**Caddy** is really easy to run with docker.

Create a **caddy** folder and navigate into it

```bash
mkdir caddy && cd caddy
```

Create a **caddy** docker network

```bash
docker network create caddy
```

This network can be utilized to connect **Caddy** with others containers.

Create a `docker-compose.yml` file that will be containing all the configuration below

```yml
version: "3.9"

services:
  caddy:
    image: caddy:latest
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

Since **Caddy** will automatically handle all TLS certificates, it need to access the port `80` and `443`.
The configuration will be in the `Caddyfile` config file.

To initiate **Caddy**, execute the following command:

```bash
docker-compose up -d
```

For halting **Caddy**, execute:

```bash
docker-compose down
```

To monitor the logs, employ:

```bash
docker logs -f caddy
```

---
## Configure Caddy

### Using Caddy as a server

**Caddy** can be used to serve a static website.

In the **Caddy** config file, add the configuration below:

```bash
domain.name {
  root * /var/www/html/website
  file_server
}
```

Add a volume in the `docker-compose.yml` matching the path of the website sources:

```yml
services:
  caddy:
  ...
    volumes:
    ...
    - /var/www/html/website:/var/www/html/website
```

### Using Caddy as a reverse proxy

**Caddy** can also be used as a reverse proxy.

Given that **Caddy** is encapsulated within a Docker container with only ports `80` and `443` accessible,
it is essential to obtain the IP address of the host machine in order to ensure proper redirection.

#### Using Host IP address

To get the IP address of the host, run:

```bash
docker exec -it caddy /sbin/ip route | awk '/default/ { print $3 }' | head -n1
```

In the **Caddy** config file, add the configuration below:

```bash
subdomain.domain.name {
  reverse_proxy http://<host-ip>:8080
}
```

This **Caddy** configuration block sets up a reverse proxy for the domain `subdomain.domain.name`,
directing incoming HTTP traffic to a backend server running on `http://<host-ip>:8080`.

This is a common setup for routing requests from a subdomain to a specific service or application running on a backend server.

#### Using Docker host name

To direct your HTTP traffic to another Docker container,
the most effective approach is to route it through the Docker host name of the target container.

By default, containers are isolated from each other,
but utilizing an external network makes it feasible to access them from another container.

Imagine a docker container serving a website using **lighttpd**

```yml
lighttpd:
  container_name: lighttpd
  image: sebp/lighttpd:latest
  volumes:
    - <home-directory>:/var/www/localhost/htdocs
    - <config-directory>:/etc/lighttpd
  ports:
    - "8080:80"
  tty: true
```

This basic webserver will serve a static website accessible through the URL `http://localhost:8080` on the host machine.

To enable access through the **Caddy** reverse proxy, you can modify the configuration as follows:

```yml {9-12}
lighttpd:
  container_name: lighttpd
  image: sebp/lighttpd:latest
  volumes:
    - <home-directory>:/var/www/localhost/htdocs
    - <config-directory>:/etc/lighttpd
  tty: true

networks:
  default:
    external:
      name: caddy
```

You can observe two modifications:

- The port mapping is no longer defined
- The **caddy** network is defined as default and external


Now, you can create the **Caddy** reverse proxy configuration:

```bash
subdomain.domain.name {
  reverse_proxy http://lighttpd:80
}
```

And voila, the **lighttpd** server will now be reachable using `https://subdomain.domain.name`
and not `http://<host-ip>:8080`.

---
## Ressources

- [Caddy Documentation](https://caddyserver.com/docs/)