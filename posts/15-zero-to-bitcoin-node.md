---
title: From zero to Bitcoin Node
tags:
  - bitcoin
  - electrs
  - docker
  - mempool
  - network
  - self-host
date: 2024-07-21
description: Installing Bitcoin Node, Electrs, and Mempool with Docker Containers
icon: 🪙
---
> [!warning] Work in progress

---
## Intro

Bitcoin is a decentralized digital currency that enables peer-to-peer transactions without the need for a central authority. It's possible to participate to the network by installing a Bitcoin node.

A node ensures you have full control over your transactions, don't rely on other entity, and directly participates in the Bitcoin network's security and decentralization. It empowers you to verify transactions independently, enhancing your privacy and trust in the system.

> Don't trust, verify...

This guide will walk you through the process of installing a Bitcoin node, Electrs, and Mempool using Docker. By leveraging Docker, you can streamline the deployment and management of these services. Whether you're a developer or an enthusiast, this step-by-step tutorial will simplify your setup process. Let's dive into creating a robust and efficient Bitcoin node infrastructure.


---
## Prerequisite

Before we start, ensure you have the following prerequisites:

- A server or machine with Docker installed.
- Basic understanding of Docker concepts such as containers, images, and volumes.
- Docker Compose.


---
## Bitcoin node

```bash
docker network create --driver=bridge --subnet=192.168.10.0/28 --gateway=192.168.10.1 bitcoin-network
```

```yml
services:
  tor:
    image: dockurr/tor
    container_name: bitcoind_tor
    networks:
      internal:
        ipv4_address: 10.254.0.2
    volumes:
      - ./tor/config:/etc/tor
      - ./tor/data:/var/lib/tor
    stop_grace_period: 1m
    restart: unless-stopped

  bitcoin:
    container_name: bitcoind
    user: 1000:1000
    image: lncm/bitcoind:v25.0
    volumes:
      - ./data:/data/.bitcoin
    restart: unless-stopped
    stop_grace_period: 15m30s
    networks:
      internal:
        ipv4_address: 10.254.0.3
      bitcoin-network:
        ipv4_address: 192.168.10.2
        #ports:
      #- "8332:8332"
      #- "28332:28332"
      #- "28333:28333"
    deploy:
      resources:
        limits:
          cpus: '2'
    depends_on:
      - tor

networks:
  bitcoin-network:
    name: bitcoin-network
    external: true
  internal:
    ipam:
      config:
        - subnet: 10.254.0.0/29
```
---
## Electrs


---
## Mempool


---
## Conclusion

---
## Resources

- Photo by [Shubham Dhage](https://unsplash.com/@onefifith?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-group-of-blue-plastic-containers-OD793Oi3kEM?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)