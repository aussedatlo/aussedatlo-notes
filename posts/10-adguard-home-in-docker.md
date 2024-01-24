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
description: How to run AdGuard Home on your home server to protect you from intrusive advertising and tracking.
icon: 👁️‍🗨️
---


![[bogomil-mihaylov-OHxTNeAtNRs-unsplash.jpg]]
> [!warning] Work in progress

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

AdGuard can require a lot of port mapping:
- `3000` for initial setup
- `53` for the DNS server
- `80` for the web interface default port
- `443` for [DNS-over-TLS](https://github.com/AdguardTeam/Adguardhome/wiki/Encryption)
- `784` for [DNS-over-QUIC](https://github.com/AdguardTeam/Adguardhome/wiki/Encryption)
- `853` for [DNS-over-TLS](https://github.com/AdguardTeam/Adguardhome/wiki/Encryption)
- `5443` for a [DNSCrypt](https://github.com/AdguardTeam/Adguardhome/wiki/DNSCrypt) server

In this example, we will exclusively utilize the **basic DNS server**. We will also maintaining port `3000` for the admin interface. This choice helps **prevent conflicts** with other services, such as reverse-proxy or web-server services, that commonly utilize port `80`.

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


Before initiating the container, verify whether a DNS server is currently running on your system by executing the following command:

```bash
netstat -nlp | grep ":53 "
```

On Ubuntu-like systems, a DNS server is integrated with systemd. You can disable it using the following command:

```bash
systemctl stop systemd-resolved.service
```

Now, you can start the container using the command:

```bash
docker compose up -d
```

### Web Setup

Once the container started, you can visit the URL `http://127.0.0.1:3000` to access the web interface.

> [!note] Note
> For remote installations, you can replace the IP address `127.0.0.1` with the actual IP address of the machine where AdGuard is installed, such as `192.168.0.4` for example.

You should see the **Welcome screen**:

![[10-welcome.png|500]]

Create the basic configuration, changing the **Admin Web Interface** port to `3000` instead of `80`. Maintain the default **DNS port** as `53`.

![[10-welcome-step-2.png|500]]

Create a `username` and `password` that will be used to connect to the **web interface**.

![[10-welcome-step-3.png|500]]

After the **Welcome** process finished, refresh the URL `http://127.0.0.1` to access the **login interface**.

![[10-login.png|500]]

You should be able to access the **Dashboard**.

![[10-dashboard.png|500]]

### Testing

To confirm the correct setup of the DNS, you can use `nslookup` to query the DNS for the resolution of a domain name. If the server is configured properly, the command should return the IP address associated with the specified domain name.

For example, run the `nslookup` command below:

```bash
nslookup google.com 127.0.0.1
```

The command should return a response similar to the following:
```txt
Server:         127.0.0.1
Address:        127.0.0.1#53

Non-authoritative answer:  
Name:   google.com
Address: 142.250.179.110
Name:   google.com
Address: 2a00:1450:4007:818::200e
```

### Router configuration

To configure this custom DNS for all your home devices, you'll need to set up the correct configuration directly on your router. Typically, the DHCP server, responsible for assigning IP addresses, also transmits the DNS address. In many cases, the router handles both DNS and DHCP server functions. The configuration details may vary depending on the type of router you are using.

In certain situations, some routers may not support or provide limited options for custom DNS settings. In such cases, you should have the option to disable the DNS and DHCP services on your router and utilize the DNS and DHCP services provided by AdGuard Home. AdGuard Home does support DHCP services, allowing you to manage both DNS and DHCP functionalities from within the AdGuard Home environment.

---
## Configure DHCP

>[!warning] Warning
>It's crucial to have only one DHCP server active on your network at a time. Running multiple DHCP services simultaneously can lead to issues such as duplicate IP assignments and unpredictable network behavior. If you decide to use AdGuard Home's DHCP service, ensure that the DHCP service on your router is disabled to maintain a stable and well-managed network environment.



---
## Configure Caddy


---
## Ressources

- Photo by [Bogomil Mihaylov](https://unsplash.com/@bogomi?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/red-and-white-stop-road-sign-near-green-tree-OHxTNeAtNRs?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)