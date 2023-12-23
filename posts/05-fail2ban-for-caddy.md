---
title: Fail2Ban for Caddy
tags:
  - caddy
  - fail2ban
  - security
  - docker
date: 2023-12-23
---
# 🛡️ How to configure Fail2Ban for Caddy

> [!warning] Work in progress

## Prerequisite

To proceed with this guide, ensure you have:
- A functioning **fail2ban** instance (refer to [[03-fail2ban-in-docker]])
- A **Caddy** instance (check out [[01-caddy-in-docker]])
- Plugin `access-formatted` for **Caddy** (look [[04-install-caddy-plugins]] to know how to add the plugin).

---
## Return an Unauthorized Error

Referencing the earlier post [[02-caddy-hardening]], update the configuration to permit only the local network, resulting in an unauthorized error (401) response instead of an abort.

```yml {8}
(safe) {
    # 192.168.0.0/24: local ip range    
    @allowed remote_ip 192.168.0.0/24 
  
    handle {
        # return unauthorized error if
        # the remote IP isn't in the specified range
        respond 401
    }
}
```


---
## Format Caddy Logs

Add a global configuration on top of your `Caddyfile` configuration:

```yml
{  
    debug  
    log access-formatted {  
        output file /data/access.log
    } 
}
```

Configured in this manner, all **Caddy** logs will be written to the `/data/access.log` file, serving as the reference for **Fail2Ban** to monitor and identify potentially suspicious access.

The **Caddy** logfile is rich in information. For improved readability and parsing, leverage the `transform` logging encoder available in the `access-formatted` plugin.

```yml {5-7}
{  
    debug  
    log access-formatted {  
        output file /data/access.log
        format transform `{ts} {request>headers>X-Forwarded-For>[0]:request>remote_ip} {request>host} {request>method} {request>uri} {status}` {
            time_format "02/Jan/2006:15:04:05"
        }
    } 
}
```

This will standardize the format of each log entry to look like:

```txt
23/Dec/2023:17:53:24 192.168.0.24 domain.name GET /index 200
```

---
## Create Fail2Ban filter

on the `data/filter.d/` folder, create a file `caddy-custom.conf` with the content:

```yml
# /etc/fail2ban/filter.d/caddy-custom.conf  
  
[Definition]  
failregex = <HOST> \S+ \S+ \S+ (401)$  
ignoreregex =
```

---
## Ressources