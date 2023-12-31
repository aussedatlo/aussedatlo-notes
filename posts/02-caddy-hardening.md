---
title: Caddy Hardening
tags:
  - caddy
  - docker
  - reverse-proxy
  - security
date: 2023-12-18
---

# 🔒 How to secure Caddy instance

In certain case, when you cant your applications to be available no matter where you are, your **Caddy** instance can be opened to the **Internet**.

It can be dangerous because of the numerous bad actors that continuously scan the web looking for vulnerabilities to exploit.

In this post, I will show you how to step-up the security of your **Caddy** server.

---

## Prerequisite

To proceed with this guide, ensure you have:

- A **Caddy** instance (check out [[01-caddy-in-docker]])

---

## Restrict to Home Network

If you have some applications that are served with **Caddy** but you don't want them to be available outside of your home network, it's possible to configure **Caddy** to reject automatically all the non desired HTTPS requests that doesn't match your local IP range.

You can create a snippet on top of your `Caddyfile`:

```text
(safe) {
    # replace 192.168.0.0/24 with your local IP range
    @allowed remote_ip 192.168.0.0/24

	handle {
		abort
	}
}
```

Then, import it on a domain declaration:

```text {2}
sub.domain.name {
	import safe

	handle @allowed {
	reverse_proxy http://app:80
	}
}
```

![[02-ban-local.png]]

In this example, all the traffic from an IP address different from the local network `192.168.0.0/24` and requesting for `sub.domain.name` will be automatically aborted 🤯.

> [!note] Note
> To manage TLS certificates, **Caddy** needs internet accessibility. However, in the current configuration, Caddy is set up to handle TLS certificates for domains that are accessible solely within the local network and are not exposed to the wider internet.


---

## Remove the `Server` Response Header

By default, **Caddy** add a `Server: Caddy` response header that will expose the type of server you are running.

Obviously, for security reasons, we don't want this information to be available in our HTTP responses.

You can add a `common` snippet on top of your `Caddyfile`:

```text
(common) {
	header /* {
	-Server
	}
}
```

And then, include it this way:

```text {3}
sub.domain.name {
	import safe
	import common

	handle @allowed {
		reverse_proxy http://lighttpd:80
	}
}
```

---

## Final Configuration

> [!note] Note
> You can combine `safe` and `common` snippet to reduce imports.

```text
(common) {
    header /* {
        -Server
    }
}

(safe) {
    import common

    # 192.168.0.0/24: local ip range
    @allowed remote_ip 192.168.0.0/24

	handle {
		abort
	}
}

sub.domain.name {
    import safe

	    handle @allowed {
	        reverse_proxy http://lighttpd:80
	    }
}
```

---

## Ressources

- [Caddy Documentation: snippets](https://caddyserver.com/docs/caddyfile/concepts#snippets)
- [Caddy Documentation: headers syntax](https://caddyserver.com/docs/caddyfile/directives/header#syntax)
