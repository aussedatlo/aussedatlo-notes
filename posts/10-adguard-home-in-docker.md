---
title: AdGuard Home in Docker
tags:
  - adguard
  - dns
  - dhcp
  - caddy
  - docker
  - privacy
date: 2024-01-24
---
> [!warning] Work in progress
# 👁️‍🗨️ AdGuard Home in Docker

![[bogomil-mihaylov-OHxTNeAtNRs-unsplash.jpg]]

---

## Intro

[AdGuard Home](https://adguard.com/fr/adguard-home/overview.html) is a software that protect you from **advertising** and **tracking** enhancing your **privacy**. It work as a **DNS** filter so can work on **all your devices** without the need to install client, no matter the type or the operating system of the device.

[AdGuard Home](https://adguard.com/fr/adguard-home/overview.html)  is free, open-source, self-hostable and open to customization. The official documentation and source code can be found on the [github](https://github.com/AdguardTeam/AdGuardHome)

## Prerequisite

- Docker
- A static IP

---

## Run AdGuard Home in Docker

### Docker compose file

Create a `adguard` folder:

```bash
mkdir adguard
cd adguard
```

Create a `docker-compose.yml` file:

```yml
version: "2"
services:
   adguardhome:
     image: adguard/adguardhome
     container_name: adguardhome
     restart: unless-stopped
     ports:
       - 53:53/tcp
       - 53:53/udp
       - 443:443/tcp
       - 784:784/udp
       - 853:853/tcp
       - 3000:3000/tcp
       - 5443:5443/tcp
       - 5443:5443/udp
       - 80:80/tcp
     volumes:
       - ./work:/opt/adguardhome/work
       - ./conf:/opt/adguardhome/conf
```

AdGuard need a lot of port mapping:
- `3000` for initial setup
- `53` for the DNS server
- `80` for the web interface default port
- `443` for [DNS-over-TLS](https://github.com/AdguardTeam/Adguardhome/wiki/Encryption)
- `784` for [DNS-over-QUIC](https://github.com/AdguardTeam/Adguardhome/wiki/Encryption)
- `853` for [DNS-over-TLS](https://github.com/AdguardTeam/Adguardhome/wiki/Encryption)
- `5443` for a [DNSCrypt](https://github.com/AdguardTeam/Adguardhome/wiki/DNSCrypt) server

For this example we will only use the basic DNS server and will select port `3000` for admin interface only instead of the default port `80`.

```yml
version: "2"
services:
   adguardhome:
     image: adguard/adguardhome
     container_name: adguardhome
     restart: unless-stopped
     ports:
       - 53:53/tcp
       - 53:53/udp
       - 3000:3000/tcp
     volumes:
       - ./work:/opt/adguardhome/work
       - ./conf:/opt/adguardhome/conf
```


On linux systems, a DNS is already integrated with systemd. You can disable it with the command:
```bash
systemctl stop systemd-resolved.service
```

Start the service:
```bash
docker compose up -d
```

### Web Setup

Go to `http://127.0.0.1:3000/install.html`

![[10-welcome.png|500]]

![[10-welcome-step-2.png|500]]

![[10-welcome-step-3.png|500]]

![[10-login.png|500]]

![[10-dashboard.png|500]]

### Router configuration
### Testing

Test the DNS with:
```bash
nslookup google.com 127.0.0.1
```

---
## Configure DHCP


---
## Configure Caddy


---
## Ressources

- Photo by [Bogomil Mihaylov](https://unsplash.com/@bogomi?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/red-and-white-stop-road-sign-near-green-tree-OHxTNeAtNRs?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)