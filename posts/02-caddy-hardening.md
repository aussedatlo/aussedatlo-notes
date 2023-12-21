---
title: Caddy Hardening
tags:
  - caddy
  - docker
  - reverse-proxy
  - fail2ban
  - security
---
# 🔒 How to secure Caddy instance

In certain case, when you cant your applications to be available no matter where you are, your **Caddy** instance can be opened to the **Internet**.

It can be dangerous because of the numerous bad actors that continuously scan the web looking for vulnerabilities to exploit.

In this post, I will show you how to step-up the security of your **Caddy** server.

>[!hint]- How to install **Caddy** ?
> Check out how to run **Caddy** in Docker: [[01-caddy-in-docker]]

## Restrict to Home network

If you have some applications that are served with **Caddy** but you don't want them to be available outside of your home network, it's possible to configure **Caddy** to reject automatically all the non desired HTTPS requests that doesn't match your local IP range.

You can create a snippet on your `Caddyfile`:

```yml
(safe) {
  # 192.168.0.0/24: local ip range
  @allowed remote_ip 192.168.0.0/24

  handle {
    abort
  }
}
```

Then, import it on a domain declaration:

```yml
subdomain.domain.name {
  import safe

  handle @allowed {
    reverse_proxy http://lighttpd:80
  }
}
```

> [!hint]
> In this example, all the traffic from an ip address different from the range `192.168.0.0` will be automatically aborted 🤯

## Ban IPs with Fail2Ban

> [!warning]
> WIP