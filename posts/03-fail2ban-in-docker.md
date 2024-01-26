---
title: Fail2Ban in Docker
tags:
  - docker
  - fail2ban
  - security
date: 2023-12-21
description: Defend your services from potential threats using Fail2Ban, an advanced log monitoring, in a Docker environment.
icon: 📛
---

---

## Intro

Fail2Ban is an open-source intrusion prevention framework that helps protect Linux servers from various types of attacks. It works by monitoring log files for patterns that may indicate malicious activity, such as repeated failed login attempts, and then takes action to block the source of the suspicious activity.

---

## Run Fail2Ban in Docker

Create a folder `fail2ban` and navigate into it:

```bash
mkdir fail2ban && cd fail2ban
```

Create a `.env` file with the following content:

```env
TZ=Europe/Paris

F2B_LOG_TARGET=STDOUT
F2B_LOG_LEVEL=INFO
F2B_DB_PURGE_AGE=1d
```

Create a `docker-compose.yml` with the following content:

```yml
version: "3.9"

services:
  fail2ban:
    image: crazymax/fail2ban:latest
    container_name: fail2ban
    network_mode: "host"
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - "./data:/data"
    restart: unless-stopped
```

**fail2ban** container need access to `NET_ADMIN` and `NET_RAW` to be able to manipulate the firewall that will block all undesired IPs.

To initiate **fail2ban**, execute the following command:

```bash
docker compose up -d
```

For halting **fail2ban**, execute:

```bash
docker compose down
```

To monitor the logs, employ:

```bash
docker logs -f caddy
```

---

## Add a Jail

In this example, I will show how to protect your **SSH** service.

> [!note] Predefined filters
> **fail2ban** docker image is shipped with already predefined filters, that can be used to filter logs and detect malicious behavior of common softwares like `sshd`. They are located in the `/etc/fail2ban/filter.d` folder.
>
> You can check it using the command:
>
> ```bash
> docker exec -it fail2ban cat /etc/fail2ban/filter.d/sshd.conf
> ```

Firstly, enable the `sshd` `jail` by creating a file in `data/jail.d/sshd.conf` with the following content:

```yml
[sshd]
enable = true
```

Since **fail2ban** watch logs to detect intrusive behavior, you will need to give access to the `sshd` log file `/var/log/auth.log` (may vary from your distribution).

In the `docker-compose.yml` file, add the line:

```yml {13}
version: "3.9"

services:
  fail2ban:
    image: crazymax/fail2ban:latest
    container_name: fail2ban
    network_mode: "host"
    cap_add:
      - NET_ADMIN
      - NET_RAW
    volumes:
      - "./data:/data"
      - "/var/log/auth.log:/var/log/auth.log:ro"
    restart: unless-stopped
```

Now, you can restart your **fail2ban** service with:

```bash
docker compose restart
```

---

## Monitoring

You can check the status of **fail2ban** using the command:

```bash
docker exec -it fail2ban fail2ban-client status
```

```txt
Status
|- Number of jail:      1
`- Jail list:   sshd
```

You can also specify a `jail`:

```bash
docker exec -it fail2ban fail2ban-client status sshd
```

```txt
Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     0
|  `- File list:        /var/log/auth.log
`- Actions
  |- Currently banned: 0
  |- Total banned:     0
  `- Banned IP list:
```

---

## Ressources

- [Docker image from crazy-max](https://github.com/crazy-max/docker-fail2ban/tree/master)
- Photo by [Marco Chilese](https://unsplash.com/@chmarco?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/gray-metal-frame-2sMbKyQvom4?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)
