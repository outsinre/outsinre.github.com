---
layout: post
title: Let's Encrypt Certificate
---

1. toc
{:toc}

# [Certbot](https://certbot.eff.org/docs/using.html)

```bash
~ # yum search certbot
~ # yum install certbot

~ # certbot -h
~ # certbot certificates # List existing certificates
```

# [Outline](https://certbot.eff.org/docs/using.html#getting-certificates-and-choosing-plugins)

To run *certbot*, we supply a subcommand like *certonly*, *install*, *run* etc. A subcommand accepts different types of plugin, namely *authenticator* plugins and *installer* plugins.

*authenticator* plugins verify domain owership and issue a certificate under (*/etc/letsencrypt/*). *installer* plugins modify webserver's configuration automatically. Most of the time, we want *authenticator* plugins and then update web server configuration manually.

The *certonly* subcommand and *install* subcommand accept *authenticator* plugins and *installer* plugins respectivelly, while the *run* subcommand can accept both type of plugins.

Examples of *authenticator* plugins are `--webroot` and `--standalone`. There does not exist plugins exclusively belonging to the *installer* type. However, `--apache` and `--nginx` are intersections of *authenticator* and *installer*, and can be used with the *run* subcommand.

# Nginx Template

Template for `--webroot` plugin to obtain a certificate.


```
server {
    listen 80;
    listen [::]:80;

    server_name cloud.example.com;
    root /usr/share/nginx/html;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;
}
```

Use *fullchain.pem* instead of *cert.pem* [whenever possible](https://github.com/v2ray/v2ray-core/issues/509#issuecomment-319321002).

# Obtain a Certificate

We show how to get a certificate by *authenticator* plugins.

```bash
~ # certbot certonly --webroot -w /usr/share/nginx/html -d www.example.com --dry-run
# or
~ # certbot certonly --standalone -d irc.example.com --dry-run
```

1. Use `--webroot` plugin if you have full control over the web server. It tries to write additional files under the root of your web server. The file URL is `http://domain/.well-known/acme-challenge/<file>`. It then tries to download the file by that URL.

   Therefore, make sure the domain name is resolved to current server IP.
2. Use `--standalone` plugin to obtain a certificate if you don't want to use (or don't currently have) existing web server. It does not rely on any web servers running on the machine where you obtain the certificate.

# Expand a Certificate

Add a new domain to an existing certificate.

```bash
~ # certbot certonly --expand --cert-name irc.example.com --webroot -w /usr/share/nginx/html/ -d blog.example.com,www.example.com --dry-run
```

# Renew a Certificate

Manually renew a certificate:

```bash
~ # cerbot renew --deploy-hook "systemctl reload nginx.service" --dry-run
```

The `--deploy-hook` tells Nginx to reload if the certificate is successfully renewed. We can configure the hook under */etc/letsencrypt/renewal/* directory:

```
# /etc/letsencrypt/renewal/www.example.conf
 
[renewalparams]
renew_hook = systemctl reload nginx.service
```

Renewal of [standalone certificates](https://certbot.eff.org/docs/using.html#standalone) requires either port 80 or 443 to be available. In other words, Nginx should not listen on both 80 and 443. If that's the case, we can stop Nginx first. Alternatively, we use `--pre-hook` and `--post-hook`.

```bash
~ # cerbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx" --dry-run
```

The two hooks will be executed no matter of sucess or failure.

Manual renewal is undesirable. We can automate the process by *crontab* or *systemd* timer.

```bash
# /var/spool/cron/root

~ # crontab -u root -e
~ # crontab -u root -l

30 0,12 * * * /usr/bin/certbot renew --deploy-hook "/usr/bin/systemctl reload nginx.service" --quiet
```

