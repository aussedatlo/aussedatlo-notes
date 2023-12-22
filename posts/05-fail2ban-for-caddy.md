---
title: Fail2Ban for Caddy
tags:
  - caddy
  - fail2ban
  - security
  - docker
---
# 🛡️ How to configure Fail2Ban for Caddy

> [!warning] Work in progress

From the previous post [[02-caddy-hardening]]

---
## Return an error with Caddy

```yml {6}
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
## Log Caddy errors

This one is complicated:

```
{  
       debug  
       log access-formatted {  
               output file /data/access.log  
               format transform `{ts} {request>headers>X-Forwarded-For>[0]:request>remote_ip} {request>host} {reque  
st>method} {request>uri} {status}` {  
                       time_format "02/Jan/2006:15:04:05"  
               }  
       }  
  
       log access-json {  
               output file /data/access.json  
               format json  
       }  
}
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