---
title: Caddy in Docker
tags:
  - caddy
  - docker
date: 2023-12-16
description: A simple/powerfull server/reverse proxy managing all your services with ease, running in docker.
icon: 🛒
---

---

## Intro

[Caddy](https://caddyserver.com/), a server-of-servers, is a Go application that is **fully open source**,
capable of functioning both as a **server** and a **reverse proxy**.

With no runtime dependencies, it operates seamlessly on all major platforms,
embodying a versatile solution that ensures reliable performance across diverse environments.

[Caddy](https://caddyserver.com/)'s default protocol is **HTTPS**,
it will automatically handle all certificates using issuers like [ZeroSSL](https://zerossl.com/) or
[Let's Encrypt](https://letsencrypt.org/) .

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

> [!note]
> This network can be utilized to connect **Caddy** with others containers.

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

> [!note] TLS Certificates
> Since **Caddy** will automatically handle all TLS certificates with `HTTP-01 challenge`, it need to access the port `80` and `443`.

The configuration will be in the `Caddyfile` config file.

To initiate **Caddy**, execute the following command:

```bash
docker compose up -d
```

For halting **Caddy**, execute:

```bash
docker compose down
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

```text
domain.name {
	root * /var/www/html/website
	file_server
}
```

Add a volume in the `docker-compose.yml` matching the path of the website sources:

```yml {6}
services:
  caddy:
  ...
    volumes:
    ...
    - /var/www/html/website:/var/www/html/website
```

### Using Caddy as a reverse proxy

**Caddy** can also be used as a reverse proxy.

#### Redirect using Docker host name

To redirect your HTTP traffic to another Docker container,
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

- The port mapping is no longer needed
- The **caddy** network is defined as external

Now, you can create the **Caddy** reverse proxy configuration:

```text
sub.domain.name {
  reverse_proxy http://lighttpd:80
}
```

And voila, the **lighttpd** server will now be reachable using `https://subdomain.domain.name`.

![[01-caddy-reverse-example.png]]

#### Redirect using Host IP address

If it's impossible to use the `caddy` external network, such as non-dockerized service scenario, you can also redirect to the host IP.

To get the IP address of the host `<host-ip>` from a Docker container, run:

```bash
docker exec -it caddy /sbin/ip route | awk '/default/ { print $3 }' | head -n1
```

This IP will represent the IP address of the host from the docker container, for example `172.23.0.1`, It's like `localhost` or `127.0.0.1` of your host but from your container.

If you can access the webserver it from `localhost:8080`, you can access it from the **Caddy** container using `<host-ip>:8080`, So re-add the port mapping as follow:

```yml {7-8}
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

Finally, add the configuration in the **Caddy** config file:

```text
sub.domain.name {
  reverse_proxy http://<host-ip>:8080
}
```

You have now, as previously, a redirection from `https://sub.domain.name` to `lighttpd:80` service, passing by `<host-ip>` and not `caddy` external network.

![[01-caddy-reverse-example-2.png]]

---

## Ressources

- [Caddy Documentation](https://caddyserver.com/docs/)
- Photo by [Fabio Bracht](https://unsplash.com/@bracht?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/arranged-blue-grocery-carts-e3oE-l-rtpA?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)
