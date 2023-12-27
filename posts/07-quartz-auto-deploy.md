---
title: Quartz Auto Deploy
tags:
  - caddy
  - webhook
  - github
  - quartz
  - docker
---

# 🚀 Auto Deploy Quartz Website

> [!warning] Work in progress

**Quartz**, **Obsidian** with the [Obsidian Git plugin](https://github.com/denolehov/obsidian-git) and [GitHub webhooks](https://docs.github.com/en/webhooks), deployment becomes instantaneous and entirely automated.

---
## Prerequisite
To proceed with this guide, ensure you have:
- A **Caddy** instance (check out [[01-caddy-in-docker]]) configured for **Quartz** (follow the post [[06-caddy-for-quartz]])
- Your **Quartz** sources on [Github](https://github.com)

---
## Create Update Script

Create a `shell` script named `update-quartz` with the following content:

```bash
#!/bin/sh
# /path/to/script/update-quartz
  
(cd /path/to/quartz && git pull origin main && npx quartz build)
```

This script will update the git repository then rebuild the **Quartz** static files from your notes.

---
## Update Caddy Docker Image

### Caddy Exec Plugin

To be able to execute shell script in background, **Caddy** need the `caddy-exec` plugin.

You can use the `Dockerfile` from [[04-install-caddy-plugins]]:

```dockerfile {2-3}
FROM caddy:builder AS builder
RUN xcaddy build \
	--with github.com/abiosoft/caddy-exec
	
FROM caddy:latest
COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

### Script Dependencies

The `update-quartz` also have dependencies that needs to be installed:

```dockerfile {6}
FROM caddy:builder AS builder  
RUN xcaddy build \
       --with github.com/abiosoft/caddy-exec  
  
FROM caddy:latest  
COPY --from=builder /usr/bin/caddy /usr/bin/caddy  
RUN apk add --update nodejs npm git
```

### Environment

To be able to execute the `update-quartz` command, **Caddy** need the script to be in the `PATH` variable. Edit the `Dockerfile` as follow:

```dockerfile {7}
FROM caddy:builder AS builder  
RUN xcaddy build \
       --with github.com/abiosoft/caddy-exec
        
FROM caddy:latest  
COPY --from=builder /usr/bin/caddy /usr/bin/caddy  
RUN apk add --update nodejs npm git  
ENV PATH="${PATH}:/path/to/script"
```

And add a `volume` mapping the `update-quartz` script on the `docker-cpopose.yml` of **Caddy**:

```yml {16}
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
      - /path/to/script:/path/to/script

networks:
  default:
    external:
      name: caddy
```


---
## Create Caddy /update Endpoint

Next, we will create an `/update` endpoint that will be used by a webhook to trigger the `update` script.

From the **Caddy** config of [[06-caddy-for-quartz]], add the configuration below:

```yml {3-5}
domain.name {

	route /update {
		exec update-quartz
	}
	
    root * /path/to/quartz/public
    try_files {path} {path.html}
    file_server

    handle_errors {  
        @404-410 expression `{err.status_code} in [404, 410]`  
        handle @404-410 {  
            root * /path/to/quartz/public  
            rewrite * {err.status_code}.html  
            file_server  
        }  
    }
}
```

This endpoint is currently accessible by all the internet. To create a `basicauth`, generate a password hash using the command:

```bash
docker exec -it caddy caddy hash-password
```

This command will output a string in the format `$2a$14$S57sqa8RAHPBTRYvy.GqYOQOoPBeip.zZ9W.yvmQKck61thG72bKy` (hash generated from `1234` password).

Complete the **Caddy** config file with `basicauth` directive, specifying an `user` and `hash`:

```yml {4-6}
domain.name {

	route /update {
		basicauth {
			user $2a$14$S57sqa8RAHPBTRYvy.GqYOQOoPBeip.zZ9W.yvmQKck61thG72bKy
		}

		exec update-quartz
	}
	
    root * /path/to/quartz/public
    try_files {path} {path.html}
    file_server

    handle_errors {  
        @404-410 expression `{err.status_code} in [404, 410]`  
        handle @404-410 {  
            root * /path/to/quartz/public  
            rewrite * {err.status_code}.html  
            file_server  
        }  
    }
}
```

With this configuration, the endpoint `/route` will be protected with credentials `user`/`1234`.

---
## Setup Github Webhook

Go to Github in `Settings` => `Webhooks` => `Add webhook`.
On the `Payload URL`, set the URL `https://user:1234@domain.name/update`.

When you will push your code on Github, this webhook will be triggered and will automatically update your server repository.

---
## Improve Security

[[05-fail2ban-for-caddy]]

## Ressources