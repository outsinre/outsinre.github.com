---
layout: post
title: Let's Encrypt
---

1. toc
{:toc}

# [ACME and Certbot](https://www.linuxbabe.com/security/letsencrypt-webroot-tls-certificate)

In order to manage [Let's Encrypt](https://letsencrypt.org/) certificates, we need a client tool that supports the [RFC 8555 Automated Certificate Management Environm (ACME) protocol](https://tools.ietf.org/html/rfc8555). When managing certificates, the client communicates with Let's Encrypt server using ACME protocol.

Let's Encrypt officially recommends the [Certbot](https://certbot.eff.org/docs/using.html) client.

```bash
~ # yum search certbot
~ # yum install certbot

~ # certbot -h
~ # certbot certificates # List existing certificates
```

To run *certbot*, we supply a subcommand like *certonly* (obtain a certificate), *install* (update vhost), *run* (both) etc. A subcommand accepts different types of plugin, namely *authenticator* plugins and *installer* plugins. The general usage looks like:

```
certbot subcommand --plugin-name ...
```

The *certonly* subcommand and *install* subcommand accept *authenticator* plugins and *installer* plugins respectivelly, while the *run* subcommand accepts plugins belonging to both types.

*authenticator* plugins challenge you whether you are eligible for a certificate by verifying the domain owership. *installer* plugins automatically modify web server's configuration with specified certificate. Most of the time, we firstly use *authenticator* plugins to obtain a certificate and then manually update configurations of the web server.

Examples of *authenticator* plugins are `--webroot` and `--standalone`. There does not exist plugins exclusively belonging to the *installer* type. However, `--apache` and `--nginx` belong to both types of *authenticator* and *installer*, and can be used with the *run* subcommand to both obtain and install a certificate.

The following table simply presents their relationship:

| --- | --- | --- |
| subcommand | type | plugin |
| :--- | :--- | :--- |
| certonly | authenticator | webroot, standalone, dns-cloudflare, dns-digitalocean |
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

We will show you how to get a certificate by the *certonly* subcommand. Run the Certbot client where you host the web server.

```bash
~ # certbot certonly --webroot -w /var/www/example.com -d example.com,www.example.com -w /var/www/b.example.com -d b.example.com --dry-run --email "name@example.com"
```

As the name implies, *certonly* only obtains certificates but does not install them. When requesting a certificate for multiple domains, each domain will use the most recently specified `--webroot-path, -w`. The `--email` option is to receive notification like expiration message.

Use the `--webroot` authenticator if you have full control over the *running* web server and the domain. Let's Encrypt's ACME server tells the Certbot client to write *unique* files under the root of your web server (i.e. */usr/share/nginx/html/*). This step is challenge you whether you do own the web server (i.e. write access). The file URL takes the form:

```
http://domain/.well-known/acme-challenge/<file>
```

Afterwards, the ACME server tries to fetch the URL, verifying you own the domain. Therefore, make sure the domain name is finally resolved to the web server IP and the web server is running on HTTP 80. You can cover your web server with CDN, as the challenge method is to download the unique file.

Once downloaded, the ACME server also compares the file hashes of the fetched copy with its local store.

```bash
~ # certbot certonly --standalone -d www.example.com,blog.example.com --dry-run
```

To the contrary, use the `--standalone` authenticator uses a different challenge method. It obtains a certificate when there is *no* web server running on the host. It starts a *temporary standalone web server* to talk to Let’s Encrypt. Therefore, it does not verify web server. You must make sure the server port 80 is not occupied by any services. You may have to turn down existing web servers to release port 80.

Recall that `--webroot` challenge domain onwership by HTTP, GETting an unique URL. Then how does `--standalone` challenge the domain onwership? By HTTP too! You must make sure the domain name is *directly* resolved to the host IP where you apply for certificates, namely where run the *certbot* client. If the domain name is resolved to the host IP, it means you manage the domain. You cannot cover your domain with CDN!

The `--standalone` authenticator is usually used when the host you use to apply for certificates is not the one you would like to host your web server.

No matter which plugins you use, the generated certificates are placed under */etc/letsencrypt/*. You will find a certificate has two versions, namely the *fullchain.pem* and the *cert.pem*. The formmer one contains intermediate certificates, and provide full validation chain. So use *fullchain.pem* [whenever possible](https://github.com/v2ray/v2ray-core/issues/509#issuecomment-319321002).

# Wildcard Certificate #

With the *certbot* client, the only way to obtain a wildcard certificate from Let's Encrypt is using DNS Plugins. DNS plugins belong to the *authenticator* type but challenge you by DNS protocol.

For each DNS platform (i.e. [Cloudflare](https://certbot-dns-cloudflare.readthedocs.io/en/stable/)), *certbot* provodes a corresponding DNS plugin. DNS plugins are not installed by default.

Basically, a DNS plugin uses a API token from the DNS platform to first add a TXT record and then remove that record, such that you are proved to own the domain name.

# Install a Certificate

Update your web server's vhost configuration accordingly.

```bash
~ # nginx -t
~ # systemctl reload nginx
```

# Expand a Certificate

Add new domains to an existing certificate.

```bash
~ # certbot certonly --expand --cert-name example.com --webroot -w /var/www/log.example.com -d blog.example.com --dry-run
```

# Renew a Certificate

A certificate can be manually renewed or automatically renewed. To manually renew a certificate, use the *renwe* subcommand.

```bash
~ # cerbot renew --deploy-hook "systemctl reload nginx.service" --dry-run
```

The `--deploy-hook` tells Nginx to reload if the certificate is successfully renewed. We can configure the hook under */etc/letsencrypt/renewal/* directory:

```
# /etc/letsencrypt/renewal/www.example.com.conf
 
[renewalparams]
renew_hook = systemctl reload nginx.service
```

Renewal of [standalone certificates requires port 80 to be available. In other words, Nginx should not listen on HTTP 80. If that's the case, you can stop Nginx first. Alternatively, we use `--pre-hook` and `--post-hook` arguments.

```bash
~ # cerbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx" --dry-run
```

Attention please; the two hooks will be executed no matter of sucess or failure.

Manual renewal is inefficient. We can automate the process by *crontab* or *systemd* timer.

```bash
# /var/spool/cron/root

~ # crontab -u root -e
~ # crontab -u root -l

30 0,12 * * * /usr/bin/certbot renew --deploy-hook "/usr/bin/systemctl reload nginx.service" --quiet
```

