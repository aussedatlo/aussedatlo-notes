---
title: Signal notifications on SSH login
tags:
  - security
  - notifications
  - signal
  - ssh
date: 2024-06-26
description: How to get instant notifications on new SSH login with Signal chat app
icon: 🔔
---
> [!warning] Work in progress

---

## Intro

How to get instant notifications on new SSH login with Signal chat app

## Prerequisite

- Docker installed

## Signal

```yml
services:
  signal:
    image: bbernhard/signal-cli-rest-api:latest
    container_name: signal-cli-rest-api
    environment:
      - MODE=native
    volumes:
      - ./data://home/.local/share/signal-cli
    ports:
      - 8080:8080
```

Get QRCode at `http://localhost:8080/v1/qrcodelink?device_name=signal-api` and scan it with the Signal application. `Settings` -> `Linked devices`.

## Pam

Pam configuration: `/etc/pam.d/sshd`

```bash
# Execute script on login
session optional pam_exec.so seteuid /opt/login.sh
```


`/opt/login.sh`:
```bash
#!/bin/sh

NUMBER="+33xxxxxxxx"
URL=http://localhost:8080
MESSAGE="New device connection\nDate: $(date +"%d/%m/%Y %H:%M:%S")\nUser: ${PAM_USER}\nIp: ${PAM_RHOST}"

if [ "$PAM_TYPE" != "close_session" ]; then
        curl -X POST -H "Content-Type: application/json" "${URL}/v2/send" \
                -d "{\"message\": \"${MESSAGE}\", \"number\": \"${NUMBER}\", \"recipients\": [ \"${NUMBER}\" ]}"
fi
```

---
## Ressources
