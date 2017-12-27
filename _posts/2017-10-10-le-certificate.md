---
layout: post
title: Let's Encrypt Certificate
---

# [Certbot](https://certbot.eff.org/docs/using.html)

```bash
~ # yum search certbot
~ # yum install certbot [python-certbot-nginx] (optional Nginx plugin)
~ # certbot -h [all]
~ # certbot certificates
```

# [Outline](https://certbot.eff.org/docs/using.html#getting-certificates-and-choosing-plugins)

1. To run certbot, we supply a subcommand like *certonly*, *install*, *run* etc.
2. A subcommand accepts different types of plugin, namely *authenticator* and *installer*.
3. *authenticator* plugins verify domain owership and issue a certificate (*/etc/letsencrypt/*). *installer* plugins modify webserver's configuration automatically. Most of the time, we want authenticator plugins and then update web server configuration manually.
4. *certonly* and *install* accept *authenticator* plugins and *installer* plugins respectivelly while *run* accepts plugins that are of both types.
5. Examples of *authenticator* plugin are `--webroot` and `--standalone`. `--apache` and `--nginx` are intersection of *authenticator* and *installer*.

   There is not exclusive *installer* plugin yet.

# Tips

1. A simple Nginx template:

   Use [Let's Encrypt template](/2017/04/11/nginx).
2. Use *fullchain.pem* instead of *cert.pem* [whenever possible](https://github.com/v2ray/v2ray-core/issues/509#issuecomment-319321002).

# webroot/standalone plugin

```bash
~ # certbot certonly --webroot -w /usr/share/nginx/html -d www.example.com --dry-run
# or
~ # certbot certonly --standalone -d irc.example.com --dry-run
```

1. Use `--webroot` plugin if you have full control over the web server.
2. Use `--standalone` plugin to obtain a certificate if you don't want to use (or don't currently have) existing server software. It does not rely on any other server software running on the machine where you obtain the certificate.

# expand

```bash
~ # certbot certonly --expand --cert-name irc.example.com --webroot -w /usr/share/nginx/html/ -d blog.example.com,www.example.com --dry-run
```

Add a new domain (blog) to existing certificate (www).

# Renewal

```bash
~ # cerbot renew --deploy-hook "systemctl reload nginx.service" --dry-run
```

1. `--deploy-hook` tells *nginx* to reload *if* renewal is successful. Another way is to put the hook into *renew* configuration file:

   ```
   # /etc/letsencrypt/renewal/www.example.conf
   [renewalparams]
   renew_hook = systemctl reload nginx.service
   ```

2. Renewal certificates of [standalone](https://certbot.eff.org/docs/using.html#standalone) plugin requires port 80 and/or 443 to be available. You may need to stop web server during renewal.

   We can do this by `--pre-hook` and `--post-hook`.

   ```bash
   ~ # cerbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx" --dry-run
   ```

   These two hooks will be executed *always* no matter of sucess or failure.
3. We can use *crontab* or *systemd* timer to automate certificate renewal.

   ```bash
   ~ # crontab -u root -e/-l
   # /var/spool/cron/root
   30 0,12 * * * /usr/bin/certbot renew --deploy-hook "/usr/bin/systemctl reload nginx.service" --quiet
   ```

   Twice a day.
4. Make sure relevant ports 80/443 are allowed by firewall.