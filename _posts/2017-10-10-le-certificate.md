---
layout: post
title: Let's Encrypt Certificate
---

# [Certbot](https://certbot.eff.org/docs/using.html)

```bash
~ # yum search certbot
~ # yum install certbot python-certbot-nginx (optional `--nginx` plugin)
~ # certbot -h/--help all
~ # certbot certificates
```

1. `--nginx` plugin accepts ONLY *tls-sni-01* (443 port) challenge.

   This plugin is shitty! Do not use it unless you have a strong reason.

# Notes

1. Don't load Nginx virtual host before certificate is generated.

   Use [Let's Encrypt template](/2017/04/11/nginx) at first.
2. *run*, *certonly*, *install* are subcommands.
3. `--nginx`, `--webroot`, `--standalone` are plugins.

# webroot/standalone

```bash
~ # certbot certonly --webroot -w /usr/share/nginx/html -d irc.example.com --dry-run
# or
~ # certbot certonly --standalone -d irc.example.com --dry-run
```

1. Use `--webroot` plugin if you have full control over the web server.
1. To be conservative, *certonly* **authenticate**s but does not **install** (modify) web server configurations automatically.

   Default **run** subcommand combines both *certonly* and *install*.
2. Use *standalone* plugin to obtain a certificate if you don’t want to use (or don’t currently have) existing server software. The *standalone* plugin does not rely on any other server software running on the machine where you obtain the certificate.

# expand

```
~ # certbot certonly --expand --cert-name irc.example.com --webroot -w /usr/share/nginx/html/ -d cloud.example.com,irc.example.com --dry-run
```

Add a new domain (cloud) to existing certificate (irc).

# Renewal

```bash
~ # cerbot renew --renew-hook "systemctl reload nginx" --dry-run
```

1. Let's Encrypt certificate expires in 3 months. Please remember to renew. We can use *crontab* or *systemd* timer to automate certificate renewal.
2. If the web server occupies port 80 and/or 443, try to [turn off web serser](https://certbot.eff.org/docs/using.html#standalone) temprarily.
3. `--renew-hook` tells *nginx* to reload updated certificates. Another way is to put the hook into *renew* configuration file:

   ```
   # /etc/letsencrypt/renewal/www.example.conf
   [renewalparams]
   renew_hook = systemctl reload nginx
   ```
