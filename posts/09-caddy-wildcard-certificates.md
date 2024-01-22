---
tags:
  - caddy
  - docker
  - reverse-proxy
  - tls
  - security
  - certificate
date: 2024-01-17
title: Caddy Wildcard Certificates
---

> [!warning] Work in progress
# 🔐 Wildcard Certificates in Caddy server

Managing multiple subdomains within a website can be a complex task, especially when it comes to handling security certificates. Each subdomain typically requires its own unique SSL/TLS certificate to ensure a secure connection between the user's browser and the server. As the number of subdomains grows, so does the challenge of keeping track of and renewing individual certificates, leading to increased administrative overhead and potential security vulnerabilities.

In response to these challenges, wildcard certificates have emerged as a valuable solution. Wildcard certificates provide a way to secure not just a single subdomain but all of its subdomains under a common domain with a single certificate. 

## Prerequisite

- A Caddy Instance
- A domain name with `*` and `@` DNS configuration pointing on your machine

---
## What is a Wildcard Certificate

A wildcard certificate is a type of SSL/TLS certificate that is designed to secure a domain and all its subdomains with a single certificate. The wildcard character `*` is used in the domain name field to indicate that the certificate is valid for any subdomain under the specified domain.

For example, if you have a wildcard certificate for `*.domain.name` it would be valid for `sub1.domain.name`, `sub2.domain.name`, `sub3.domain.name` and so on. The asterisk acts as a placeholder for any subdomain.

> [!warning] Disclosure
> In the event of a compromised private key, the security of all your subdomains is at risk, given that they share the same certificate.

---
## How to configure Caddy

Caddy can be configured to use Wildcard Certificates.

> [!note]
> This guide presupposes that you are utilizing Caddy within a Docker container.

### Add DNS Module

Since [Let's encrypt requires](https://letsencrypt.org/docs/challenge-types/) the `DNS-01 challenge` (and not the common `HTTP-01 challenge`) to obtain wildcard certificate, we will need to add a DNS Module that allow Caddy to resolve the challenge.

A DNS Challenge asks to prove that we are in control of the domain DNS by putting a specific value in a `TXT` record under that domain name. If it finds a match, the issuer can proceed to issue a certificate.

> [!note] Note
> With this challenge, you don't need port `443` of `80` to be accessible from internet

Caddy comes with a variety of DNS modules that are listed on [this github link](https://github.com/caddy-dns). In my example I will be using the [ionos dns module](https://github.com/caddy-dns/ionos). If your DNS provider is not in the module list, you can follow the [caddy tutorial](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148) to see how to create a custom DNS module.

Referring to the previous post [[04-install-caddy-plugins]], you can modify your `Dockerfile` like following:

```Dockerfile {6}
FROM caddy:builder AS builder  
  
RUN xcaddy build \  
       --with github.com/caddyserver/transform-encoder \  
       --with github.com/abiosoft/caddy-exec \  
       --with github.com/caddy-dns/ionos  
  
FROM caddy:latest  
  
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

Now rebuild the Docker image with the command

```bash
docker-compose build
```

### Add DNS Configuration

To configure the new DNS Caddy Module, edit the `Caddyfile` as follow:

```txt {2-4}
*.domain.name, domain.name {  
   tls {  
       dns ionos {env.IONOS_API_TOKEN}  
   }
}
```

This will configure Caddy to use `DNS-01 challenge` instead of `HTTP-01 challenge` and provide all the tools to edit your DNS settings from your DNS provider ionos.

Create the `.env` file that will contains the `IONOS_API_TOKEN` value:

```env
IONOS_API_TOKEN=<token-here>
```

Add the env file to your `docker-compose.yml` file:



```yml {19-20}
version: "3.9"  
  
services:  
 caddy:
   # this is the custom docker image with the Caddy DNS module
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

```txt
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
        respond "OK from sub.domain.name"
    }
}
```

Now, you have a subdomain  `sub1.domain.name` that share the same certificate as `domain.name`.

---
## Hardening

---
## Ressources