---
title: Caddy for Quartz
tags:
  - caddy
  - quartz
  - reverse-proxy
  - webhook
date: 2023-12-26
---
# ✨ What is Quartz

From the official [Quartz Github](https://github.com/jackyzha0/quartz)

> [!quote]
> Quartz is a set of tools that helps you publish your [digital garden](https://jzhao.xyz/posts/networked-thought) and notes as a website for free. Quartz v4 features a from-the-ground rewrite focusing on end-user extensibility and ease-of-use.

Personally, I love **Quartz** for its seamless integration with [Obsidian](https://obsidian.md/), enabling me to craft this blog effortlessly using my favorite note editor.

---
## Prerequisite
To proceed with this guide, ensure you have:
- A static website generated with **Quartz** (read the [Quartz documentation](https://quartz.jzhao.xyz/))
- A **Caddy** instance (check out [[01-caddy-in-docker]])
- Plugin `caddy-exec` to auto deploy (look [[04-install-caddy-plugins]] to know how to add the plugin).

---
## Host Quartz with Caddy

### Basic Configuration

To host your Quartz website, execute the command below to generate all static files within the public folder (refer to [build documentation](https://quartz.jzhao.xyz/build) for more details):

```bash
npx quartz build
```

Following the execution of this command, a `public` folder will be generated, containing all the `.html` necessary files.

On the `Caddyfile`, add the following configuration:

```yml
domain.name {  
    root * /path/to/quartz/public
    file_server 
}
```

This will tell **Caddy** to serve the `public` folder as a static website.

> [!note] Caddy and path handling
> The debug server in **Quartz** automatically redirects the path to **HTML** files. However, this behavior differs in **Caddy**; if you request `/post,` it searches for a `/post` file rather than `/post.html`.
>
>This configuration is effective only for `/index` because **Caddy** automatically searches for HTML index file.

To ensure accurate path handling, incorporate a `try_files` directive:

```yml {3}
domain.name {
    root * /path/to/quartz/public
    try_files {path} {path.html}
    file_server 
}
```

> [!note] Caddy and errors handling
> Similar to path handling, it's crucial to specify error handling in the Caddy file. This ensures the correct rendering of the `401.html` file when a `not found` error occurs.

To appropriately capture errors, employ a `handle_errors` and `rewrite` directive:

```yml {6-13}
domain.name {
    root * /path/to/quartz/public
    try_files {path} {path.html}
    file_server

    handle_errors {  
        @404-410 expression `{err.status_code} in [404, 410]`  
        handle @404-410 {  
            root * /path/to/quartz/public  
            rewrite * {err.status_code}.html  
            file_server  
        }  
    }
}
```

With this configuration, any errors ranging from `401` to `410` will be displayed using the corresponding `.html` file.

---
## Ressources
- Quartz
	- [Quartz Github](https://github.com/jackyzha0/quartz)
	- [Quartz Documentation](https://www.quartz-scheduler.org/documentation/)
- Caddy
	- [file_server directive](https://caddyserver.com/docs/caddyfile/directives/file_server)
	- [try_files directive](https://caddyserver.com/docs/caddyfile/directives/try_files#try-files)
	- [rewrite directive](https://caddyserver.com/docs/caddyfile/directives/rewrite#rewrite)
- Obsidian
	- [Obsidian website](https://obsidian.md/)
	- [Obsidian Git plugin](https://github.com/denolehov/obsidian-git)