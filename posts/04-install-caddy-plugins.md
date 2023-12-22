---
title: Caddy Plugins
tags:
  - caddy
  - plugins
  - docker
---
# 📦 How to Install Caddy Plugins

> [!warning] Work in progress

---
## CLI for local instance

```bash
xcaddy build \
    --with github.com/caddyserver/transform-encoder
```

---
## Custom Docker image

Create a `Dockerfile` file.

```docker
FROM caddy:builder AS builder

RUN xcaddy build \
	--with github.com/caddyserver/transform-encoder \
	--with github.com/abiosoft/caddy-exec

FROM caddy:latest

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```

---
## Ressources
- [Caddy's Modules](https://caddyserver.com/docs/modules/)