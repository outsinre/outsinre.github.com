---
layout: post
title: [Let's Encrypt Certificate](https://www.linuxbabe.com/security/letsencrypt-webroot-tls-certificate)
---

1. toc
{:toc}

# ACME and Certbot

In order to manage [Let's Encrypt](https://letsencrypt.org/) certificates, we need a client tool that supports the [RFC 8555 Automated Certificate Management Environm (ACME) protocol](https://tools.ietf.org/html/rfc8555). When managing certificates, the client communicates with Let's Encrypt server using ACME protocol.

Let's Encrypt officially recommends the [Certbot](https://certbot.eff.org/docs/using.html) client.

```bash
~ # yum search certbot
~ # yum install certbot

~ # certbot -h
~ # certbot certificates # List existing certificates
```

To run *certbot*, we supply a subcommand like *certonly*, *install*, *run* etc. A subcommand accepts different types of plugin, namely *authenticator* plugins and *installer* plugins. The general usage looks like:

```
certbot subcommand --plugin-name ...
```

The *certonly* subcommand and *install* subcommand accept *authenticator* plugins and *installer* plugins respectivelly, while the *run* subcommand can accept both type of plugins.

*authenticator* plugins verify domain owership and issue a certificate - to challenge you when applying for certificates. *installer* plugins automatically modify web server's configuration with specified certificate. Most of the time, we firstly use *authenticator* plugins to obtain a certificate and then manually update configurations of the web server.

Examples of *authenticator* plugins are `--webroot` and `--standalone`. There does not exist plugins exclusively belonging to the *installer* type. However, `--apache` and `--nginx` are intersections of *authenticator* and *installer*, and can be used with the *run* subcommand to automate the management process.

The following table simply presents their relationship:

| --- | --- | --- |
| subcommand | type | plugin |
| :--- | :--- | :--- |
| certonly | authenticator | webroot, standalone |
| install | installer | n/a |
| run | both | apache, nginx |
| --- | --- | --- |

# Nginx Template

A simple Nginx HTTP template.


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

# Obtain a Certificate

We will show you how to get a certificate by *authenticator* plugins. Run the Certbot client on your web server host.

```bash
~ # certbot certonly --webroot -w /usr/share/nginx/html -d www.example.com --dry-run
# -or-
~ # certbot certonly --standalone -d irc.example.com --dry-run
```

Use the `--webroot` authenticator if you have full control over the *running* web server and the domain. Let's Encrypt's ACME server tells the Certbot client to write *unique* files under the root of your web server (i.e. */usr/share/nginx/html/*). This step is challenge you whether you do own the web server (i.e. write access). The file URL takes the form:

```
http://domain/.well-known/acme-challenge/<file>
```

Afterwards, the ACME server tries to fetch the URL, verifying you own the domain. Therefore, make sure the domain name is finally resolved to the web server IP and the web server is running on HTTP 80. You can cover your web server with CDN, as the challenge method is to download the unique file.

Once downloaded, the ACME server also compares the file hashes of the fetched copy with its local store.

To the contrary, use the `--standalone` authenticator uses a different challenge method. It obtains a certificate when there is *no* web server running on the host. It starts an *temporary* standalone web server to talk to Let’s Encrypt. Therefore, it does not verify web server. You must make sure the server port 80 is not occupied by any services. You may have to turn down existing web servers to release port 80.

Recall that `--webroot` challenge domain onwership by HTTP URL. Then how does `--standalone` challenge the domain onwership? By DNS! You must make sure the domain name is directly resolved to the host IP where you apply for certificates and run the *certbot* client. If the domain name is resolved to the host IP, it means you manage the domain.

The `--standalone` authenticator is usually used when the host you use to apply for certificates is not the one you would like to host your web server.

No matter which plugins you use, the generated certificates are placed under */etc/letsencrypt/*. You will find a certificate has two versions, namely the *fullchain.pem* and the *cert.pem*. The formmer one contains intermediate certificates, and provide full validation chain. So use *fullchain.pem* [whenever possible](https://github.com/v2ray/v2ray-core/issues/509#issuecomment-319321002).

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

