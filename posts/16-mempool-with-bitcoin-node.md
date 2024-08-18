---
title: Mempool Integration with Bitcoin Node
tags:
  - bitcoin
  - network
  - analytics
  - explorer
  - docker
  - caddy
date: 2024-08-18
description: Step-by-step guide to connecting Mempool to your Bitcoin node
icon: ⛓️
---
> [!warning] Work in progress

---
## Intro

Mempool is a real-time web interface for visualizing Bitcoin's mempool, where unconfirmed transactions are stored. It provides detailed insights into transaction fees, mempool congestion, and network activity. Users can view transaction history, fee estimates, and the current state of the Bitcoin network.

Self-hosting a mempool site alongside a Bitcoin node enhances your privacy by keeping transaction data within your control and not relying on third-party servers.

Additionally, operating your own instance eliminates dependency on external services, ensuring continuous access and reliability even if external servers experience issues or downtime.

---
## Prerequisite

Before we start, ensure you have the following prerequisites:

- A server or machine with Docker installed.
- Basic understanding of Docker concepts such as containers, images, and volumes.
- Docker Compose.
- A Bitcoin node with an Electrum server, see [[15-zero-to-bitcoin-node]] to see how to install both.

---
## Configuration

### Networking

Assuming you have followed the previous guide [[15-zero-to-bitcoin-node]] and want to use a Caddy reverse proxy (see [[01-caddy-in-docker]] or [[09-caddy-wildcard-certificates]]) to access the Mempool web server, the network configuration should be as follows:

![[mempool-network.excalidraw.svg]]

The `mempool-api` service is on the `bitcoin` network to access both Elects and Bitcoin node and in the `internal` network to access the `mempool-db` service.

The `mempool-web` is on the `caddy` network to allow Caddy to reverse proxy to the server, and on the `internal` network to access the `mempool-api` backend.

The `mempool-db` only needs access to the `mempool-api` and `mempool-web`, so it is configured to use only the `internal` network.

### Docker compose file

Create a folder `mempool`:
```bash
mkdir mempool && cd mempool
```

Create a `docker-compose.yml` file with the content:
```yml
services:
  web:
    environment:
      FRONTEND_HTTP_PORT: "8080"
      BACKEND_MAINNET_HTTP_HOST: "api"
    image: mempool/frontend:latest
    user: "1000:1000"
    restart: unless-stopped
    stop_grace_period: 1m
    command: "./wait-for db:3306 --timeout=720 -- nginx -g 'daemon off;'"
    networks:
      - internal
      - caddy

  api:
    environment:
      MEMPOOL_BACKEND: "electrum"
      ELECTRUM_HOST: "192.168.10.3"
      ELECTRUM_PORT: "50001"
      ELECTRUM_TLS_ENABLED: "false"
      CORE_RPC_HOST: "192.168.10.2"
      CORE_RPC_PORT: "8332"
      CORE_RPC_USERNAME: "bitcoin_rpc_user"
      CORE_RPC_PASSWORD: "bitcoin_rpc_password"
      DATABASE_ENABLED: "true"
      DATABASE_HOST: "db"
      DATABASE_DATABASE: "mempool"
      DATABASE_USERNAME: "mempool"
      DATABASE_PASSWORD: "mempool"
      STATISTICS_ENABLED: "true"
    image: mempool/backend:latest
    user: "1000:1000"
    restart: on-failure
    stop_grace_period: 1m
    command: "./wait-for-it.sh db:3306 --timeout=720 --strict -- ./start.sh"
    networks:
      internal:
      bitcoin-network:
        ipv4_address: 192.168.10.4
    volumes:
      - ./data:/backend/cache

  db:
    environment:
      MYSQL_DATABASE: "mempool"
      MYSQL_USER: "mempool"
      MYSQL_PASSWORD: "mempool"
      MYSQL_ROOT_PASSWORD: "admin"
    image: mariadb:10.5.21
    user: "1000:1000"
    restart: unless-stopped
    stop_grace_period: 1m
    networks:
      - internal
    volumes:
      - ./mysql/data:/var/lib/mysql

networks:
  caddy:
    name: caddy
    external: true
  bitcoin-network:
    name: bitcoin-network
    external: true
  internal: {}
```

> [!note] Note
> Be sure to replace `bitcoin_rpc_user` and `bitcoin_rpc_password` with your actual credentials.


Start the service using the command:
```bash
docker compose up -d
```

### Caddy reverse proxy

Add the Caddy configuration:
```text
mempool.domain.name {
    reverse_proxy http://mempool-web-1:8080
}
```

or if you are using wildcard certificates as described in the [[09-caddy-wildcard-certificates]] guide:
```text
@mempool host mempool.domain.name
handle @mempool {
    reverse_proxy http://mempool-web-1:8080
}
```

Restart Caddy and visit the URL `mempool.domain.name`. You should see your Mempool site initializing and loading.

---
## Conclusion

With Caddy handling the reverse proxy, and the appropriate access set up for `mempool-api`, `mempool-web`, and `mempool-db`, your Mempool instance should initialize smoothly, providing an efficient and reliable Bitcoin mempool monitoring experience.

---
## Resources

- Photo by [Shubham Dhage](https://unsplash.com/@theshubhamdhage?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/photos/a-very-large-amount-of-chocolate-squares-a29VlbgH4wo?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)
- Mempool Github: https://github.com/mempool/mempool
