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

Want instant notifications for new SSH logins? Integrating Signal chat app allows you to receive immediate alerts whenever someone logs into your server via SSH. This enhances security by keeping you informed of access attempts in real-time, enabling prompt action if unauthorized access is detected.

---
## Prerequisite

- Docker installed

---
## Signal

To communicate from the shell environment to Signal, we will use the `signal-cli-rest-api`. This REST API is designed to call the `signal-cli` binary under the hood.

I prefer to use this REST application to avoid installing `signal-cli` directly on my server and to enable integration with other applications, such as `Home Assistant`.

To configure the service, create a `docker-compose.yml` file on a `signal` folder.

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

To start the container, simply run:

```bash
docker compose up -d
```

The server should be starting. You can now link it to your Signal account using the `qrcodelink` endpoint: `http://localhost:8080/v1/qrcodelink?device_name=signal-api`. Scan the generated QR code with the Signal application by navigating to `Settings` -> `Linked devices`.


---
## Pam

To trigger a script while logging into SSH, we can use the `pam` configuration. It provides a flexible and centralized way to manage authentication-related tasks across various services. Edit the `/etc/pam.d/sshd` file by adding the lines below:

```bash
# Execute script on login
session optional pam_exec.so seteuid /opt/login.sh
```

You can now create the script  `/opt/login.sh` with the lines below:

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

This will automatically send a message through `signal-cli-rest-api` directly to your `Note to self` personal Signal channel.

---
## Ressources
