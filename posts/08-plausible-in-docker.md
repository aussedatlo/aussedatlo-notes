---
title: Plausible in Docker
tags:
  - docker
  - caddy
  - plausible
  - analytics
date: 2024-01-04
---

# 📊 What is Plausible Analytics

> [!warning] Work in progress

[Plausible](https://plausible.io) is a lightweight, privacy focusing, opensource, self hostable web analytics tool that is compliant with RGPD and require no cookie.

---

## Run Plausible with Docker

### Clone the hosting repo

The [plausible/hosting] Github repository contains all the resources to start a Plausible docker container.

```bash
git clone https://github.com/plausible/hosting plausible
cd plausible
```

### Configure Plausible

#### Environments

Generate a `SECRET_KEY_BASE` random value using the command:

```bash
openssl rand -base64 64 | tr -d '\n' ; echo
```

Then, in the file `plausible-conf.env` that will contains all the Docker variable environment, add the following:

```txt
BASE_URL=https://plausible.domain.name:443
SECRET_KEY_BASE=<SECRET_KEY_BASE>
```

> [!note] Note
> `BASE_URL` is the hosting URL of the server, used for URL generation. By default the `BASE_URL` port is `8000`. In my case with the future reverse proxy configuration, the port need to be `443`.

You can now start your service using the command:

```bash
docker-compose up -d
```

You can check the log using:

```
docker-compose logs -f
```

#### Reverse Proxy

To access your service securely using `HTTPS`, you can use a reverse proxy.

In my case, I'll be using **Caddy** (check out [[01-caddy-in-docker]] to see how to install **Caddy**).

Edit the `docker-compose.yml` file and add the external `caddy` network:

```yml {5-8}
version: "3.3"
services:
	...

networks:
 default:
   external:
     name: caddy
```

> [!note] Note
> Since the **Plausible** app will be using the `caddy` external network, you can also remove the port mapping from the configuration.

Add the configuration in your `Caddyfile`:

```txt
plausible.domain.name {
	reverse_proxy http://plausible_plausible_1:8000
}
```

Now, with the subdomain `plausible.domain.name` pointing on your server, you should be able to access **Plausible** with the URL [https://plausible.domain.name]()

---

## Create an Account

To create an account, visit your **Plausible** URL, and complete the form:

![[08-create-account.png]]

Since **Plausible** is self-hosted, no need to activate the account.

---

## Add a Website

Include the URL of the website from which you intend to collect information.

![[08-add-site.png]]

Copy the JavaScript snippet and add it to your site.

![[08-snippet.png]]

After reloading your site, the initial connection data should be visible on your Plausible dashboard.

![[08-dashboard.png]]

Enjoy.

---

## Security Tips

To enhance the protection of your **Plausible** instance and referring to the post [[02-caddy-hardening]], you can allow only your local network for accessing the **Plausible** app.

You can modifying the **Caddy** configuration like below:

```txt
plausible.domain.name {  
    import common  
    @allowed remote_ip 192.168.0.0/24  
  
    handle /api {  
        reverse_proxy http://plausible_plausible_1:8042  
    }  
  
    handle /js {  
        reverse_proxy http://plausible_plausible_1:8042  
    }  
  
    handle @allowed {  
        reverse_proxy http://plausible_plausible_1:8042  
    }  
  
    handle {  
        respond 401  
    }  
}
```

---

## Ressources

- Plausible [website](https://plausible.io)
- Plausible [github](https://github.com/plausible)
