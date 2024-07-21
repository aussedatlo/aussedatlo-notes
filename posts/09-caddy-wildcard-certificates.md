---
tags:
  - caddy
  - docker
  - wildcard-certificates
  - self-host
  - security
date: 2024-01-17
title: Caddy Wildcard Certificates
description: How to configure your Caddy instance with Wildcard Certificates to quickly setup new services through reverse proxy.
icon: 🔐
---

---

## Intro

Managing **multiple subdomains** within a website can be a **complex task**, especially when it comes to handling security certificates. Each subdomain typically requires its own unique SSL/TLS certificate to ensure a **secure connection** between the user's browser and the server. As the number of subdomains grows, so does the challenge of keeping track of and renewing individual certificates, leading to increased administrative overhead and potential security vulnerabilities.

In response to these challenges, **wildcard certificates** have emerged as a valuable solution. Wildcard certificates provide a way to secure **not just a single subdomain but all of its subdomains** under a common domain with a **single certificate**. 

## Prerequisite

- A Caddy Instance.
- A domain name with  `*` and `@` type `A` DNS configuration pointing on your machine.

---
## What is a Wildcard Certificate

A **wildcard certificate** is a type of SSL/TLS certificate that is designed to secure a domain and all its subdomains with a single certificate. The wildcard character `*` is used in the domain name field to indicate that the certificate is valid for **any subdomain** under the specified domain.

For example, if you have a wildcard certificate for `*.domain.name` it would be valid for `sub1.domain.name`, `sub2.domain.name`, `sub3.domain.name` and so on. The asterisk acts as a placeholder for any subdomain.

> [!warning] Disclosure
> In the event of a compromise of the root certificate's private key, the security of all your subdomains is jeopardized, as they all share the same certificate.

---
## How to configure Caddy

**Caddy** can be configured to use **wildcard certificates**.

> [!note]
> This guide presupposes that you are utilizing Caddy within a Docker container.

### Add DNS Module

Since [Let's encrypt requires](https://letsencrypt.org/docs/challenge-types/) the `DNS-01 challenge` to obtain wildcard certificate (and not the common `HTTP-01 challenge`) , we will need to add a **DNS module** that allow Caddy to resolve the challenge.

A DNS Challenge asks to prove that **we are in control of the domain** DNS by putting a specific value in a `TXT` record under that domain name. If it finds a match, the issuer can proceed to issue a certificate.

> [!note] Note
> With the `DNS-01 challenge` challenge, there is no requirement for ports `443` and `80` to be accessible from the internet.

A collection of DNS modules for Caddy is accessible through the GitHub [caddy-dns](https://github.com/caddy-dns). For my example I will be using [caddy-dns/ionos](https://github.com/caddy-dns/ionos). If your DNS provider is not in the list, you can refer to the [caddy tutorial](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148) to see how to create a custom DNS module.

Referring to the previous post [[04-install-caddy-plugins]], you can modify your `Dockerfile` as follows:

```docker {6}
FROM caddy:builder AS builder  
  
RUN xcaddy build \  
       --with github.com/caddyserver/transform-encoder \  
       --with github.com/abiosoft/caddy-exec \  
       --with github.com/caddy-dns/ionos  
  
FROM caddy:latest  
  
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

Now rebuild the Docker image with the command:

```bash
docker compose build
```

### Add DNS Configuration

To configure [caddy-dns/ionos](https://github.com/caddy-dns/ionos), add the `tls` directive in your `Caddyfile`:

```txt {2-4}
*.domain.name, domain.name {  
   tls {  
       dns ionos {env.IONOS_API_TOKEN}  
   }
}
```

This will configure Caddy to use `DNS-01 challenge` instead of `HTTP-01 challenge` and provide all the tools to edit your DNS settings from your DNS provider ionos.

Create the `.env` file that will contain the generated `IONOS_API_TOKEN` value from your DNS provider:

```env
IONOS_API_TOKEN=<token-here>
```

> [!note] Note
> Generate the `IONOS_API_TOKEN` via the [ionos developer control panel](https://developer.hosting.ionos.fr/?source=IonosControlPanel).

Then, add the `.env` file to your `docker-compose.yml` file:

```yml {20-21}
version: "3.9"  
  
services:  
 caddy:
   # this is a custom docker image with the Caddy DNS module
   image: caddy-custom:<version>
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
   env_file:  
     - .env  
  
networks:  
 default:  
   external:  
     name: caddy
```
### Create Subdomains

You can now add all the subdomains that you want directly in your `Caddyfile`:

```txt {6-14}
*.domain.name, domain.name {  
    tls {  
        dns ionos {env.IONOS_API_TOKEN}  
    }
   
    @main host domain.name
    handle @main {
        respond "OK from domain.name"
    }

    @sub1 host sub1.domain.name
    handle @sub1 {
        respond "OK from sub1.domain.name"
    }
}
```

Now, you have a subdomain  `sub1.domain.name` sharing the same certificate as the main domain `domain.name`.

---
## Hardening

In the previous post [[02-caddy-hardening]], I discussed a method for restricting subdomains to the local network. With the new configuration using wildcard certificates, there is now a simpler and more efficient method available.

Update your `Caddyfile` with the following changes:

```txt {6-8} {20-24} {26-29}
*.domain.name, domain.name {  
    tls {  
        dns ionos {env.IONOS_API_TOKEN}  
    }

	@outside {
        not remote_ip 192.168.0.0/24 
    }
   
    @main host domain.name
    handle @main {
        respond "OK from domain.name"
    }

    @sub1 host sub1.domain.name
    handle @sub1 {
        respond "OK from sub1.domain.name"
    }

	# All the subdomain below will be accessible from
	# local network only
    handle @outside {  
        respond 401  
    }

    @sub2 host sub2.domain.name
    handle @sub2 {
        respond "OK from sub2.domain.name"
    }
}
```

With this configuration, the main domain `domain.name` and the subdomain `sub1.domain.name` will remain accessible from the internet, similar to the previous setup. However, the subdomain `sub2.domain.name`, declared after the `handle @outside`, will now only be available on the local network only.

>[!note] Note
>Any subdomain not configured before the `handle @outside` directive will result in a `401 Unauthorized` error. This includes cases where the subdomain is not explicitly mentioned in the configuration, such as `nonexistent.subdomain.name` for example.

---
## Ressources

- [Caddy DNS modules](https://github.com/caddy-dns)
- [Caddy DNS documentation](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148)
- [Let's encrypt challenge types](https://letsencrypt.org/docs/challenge-types/)
- Photo by [Franck](https://unsplash.com/@franckinjapan?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/black-iphone-5-on-yellow-textile-DoWZMPZ-M9s?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)
