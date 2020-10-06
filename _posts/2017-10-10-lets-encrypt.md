---
layout: post
title: Let's Encrypt
---

1. toc
{:toc}

# [ACME and Certbot](https://www.linuxbabe.com/security/letsencrypt-webroot-tls-certificate)

In order to manage [Let's Encrypt](https://letsencrypt.org/) certificates, we need a client tool that supports the [RFC 8555 Automated Certificate Management Environm (ACME) protocol](https://tools.ietf.org/html/rfc8555). ACME clients communicate with servers of Let's Encrypt using ACME protocol, to automate the following jobs:

1. Generate key pair.
2. Create and send Certificate Signing Request (CSR).
3. Challenge ownership of domain or web server.

Let's Encrypt officially recommends the [Certbot](https://certbot.eff.org/docs/using.html) client.

```bash
~ # yum search certbot
~ # yum install certbot

~ # certbot -h
~ # certbot certificates # List existing certificates
```

To run *certbot*, we supply a subcommand like *certonly* (obtain a certificate), *install* (update vhost), *run* (both) etc. If no subcommand is given, then *run* is assumed. A subcommand accepts different types of plugins, namely *authenticator* plugins and *installer* plugins. The general usage looks like:

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
| certonly | authenticator | webroot, standalone, dns-cloudflare |
| install | installer | n/a |
| run (default) | both | apache, nginx |
| --- | --- | --- |

# Nginx Template

A simple Nginx HTTP template.


```
server {
	listen       80 default_server;
	listen       [::]:80 default_server;
	server_name  _;
	root         /usr/share/nginx/html;

	# Load configuration files for the default server block.
	include /etc/nginx/default.d/*.conf;

	location / {
	}

	error_page 404 /404.html;
		location = /40x.html {
	}

	error_page 500 502 503 504 /50x.html;
		location = /50x.html {
	}
}
```

# Obtain a Certificate

ACME challenge is a complex process, you'd better turn off CDN caching before applying for a certificate.

## Webroot Authenticator ##

```bash
~ # certbot -h certonly
~ # certbot -h webroot

~ # certbot certonly --webroot -w /var/www/example.com -d example.com,www.example.com -w /var/www/b.example.com -d b1.example.com -d b2.example.com --email "name@example.com" --dry-run
```

As the name implies, *certonly* only obtains certificates but does not install them. When requesting a certificate for multiple domains, each domain will use the most recently specified `--webroot-path, -w`. The `--email` option is to receive notification like expiration message. Multiple domain names can be separated by comma in the `-d` argument or by individual `-d` arguments. Always append the `--dry-run` before any real operation.

Use the `--webroot` authenticator if you have full control over the *running* web server and the domain. Let's Encrypt's ACME server tells the Certbot client to write *unique* files under the root of your web server (i.e. */usr/share/nginx/html/*). This step is challenge you whether you do own the web server (i.e. write access). The file URL takes the form:

```
http://domain/.well-known/acme-challenge/<file>
```

Afterwards, the ACME server tries to fetch the URL, verifying you own the domain. Therefore, make sure the domain name is finally resolved to the web server IP and the web server is running on HTTP 80 (NOT HTTPS 443). Though, the *webroot* authenticator use HTTP to challenge the ownership, the security of Certbot itself is guranteed by ACME protocol. We call this kind of challenge *http-01*.

You can cover your web server with CDN (i.e. Cloudflare), as the challenge method is to download the unique file. However, if the CDN enables HSTS, then temporarily turn it off by removing the caching capability in DNS settting.

Once downloaded, the ACME server also compares the file hashes of the fetched copy with its local store.

## Standalone Authenticator ##

```bash
~ # certbot -h standalone
~ # certbot certonly --standalone -d www.example.com,blog.example.com --dry-run
```

To the contrary, use the `--standalone` authenticator to obtain a certificate when there is *no* web server running on the host. It starts a *temporary standalone web server* to talk to Let’s Encrypt. Therefore, it does not verify web server.

Recall that `--webroot` challenge domain onwership by *http-01*, GETting an unique URL. Then how does `--standalone` challenge the domain onwership? By *http-01* too! You must make sure [port 80](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-7.2) is available. You may have to turn down existing web servers to release port 80.

Though ACME protocol supports challenge with [tls-01](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-7.3) by verifying the *temporarily* self-generated certificate, but Certbot only implements HTTP 80. If you have chosen to use *tls-o1*, then you must make sure the domain name is *directly* resolved to the host IP where you apply for certificates *certbot* client. If the domain name is resolved to the host IP, it means you manage the domain. You cannot cover your domain with CDN, otherwise the ACME server would got the CDN's certificate instead of the temporary one by Certbot.

The `--standalone` authenticator is usually used when the host you use to apply for certificates is not the one you would like to host your web server.

## DNS Authenticator for Wildcard Certificate ##

With the *certbot* client, the only way to obtain a wildcard certificate from Let's Encrypt is using DNS Plugins. DNS plugins belong to the *authenticator* type but challenge you by DNS protocol.

For each DNS platform (i.e. [Cloudflare](https://certbot-dns-cloudflare.readthedocs.io/en/stable/)), *certbot* provodes a corresponding DNS plugin. DNS plugins are not installed by default.

Basically, a DNS plugin uses a API token from the DNS platform to first add a TXT record and then remove that record, such that you are proved to own the domain name.

# Manage a Certificate #

No matter which plugins you use, the generated certificates are placed under */etc/letsencrypt/*. Also the arguments used to generate the certificates are stored alongside for [latter renwal](#renew-a-certificate). The directory tree looks like:

```
# tree /etc/letsencrypt/
/etc/letsencrypt/
├── accounts
│   └── acme-staging-v02.api.letsencrypt.org
│       └── directory
│           └── abcdq383ndh08457vnlazc234c
│               ├── meta.json
│               ├── private_key.json
│               └── regr.json
├── renewal
└── renewal-hooks
    ├── deploy
    ├── post
    └── pre
```

You will find a certificate has different version, namely *fullchain.pem*, *chain.pem* and *cert.pem*. Use *fullchain.pem* [whenever possible](https://github.com/v2ray/v2ray-core/issues/509#issuecomment-319321002) as it is a superset of *chain* and *cert.pem*.

First, list existing certificates:

```bash
~ # certbot certificates
~ # certbot certificates --cert-name example.com
```

It's not unusual that multiple certificates containe some of the same domains.

From the output, you will find each certificate has a name. Pass argument `--cert-name` to specify a particular certificate for subcommands like *certonly*, *run*, *certificates*, *renew*, *delete* etc.

When *certonly* is provided the `--cert-name` argument, it will update that particular certificate or create a new one based on that certificate. You can combine `--cert-name` with the `--expand` argument to add domain names into the certificate. Alternatively, you can combine it with the `--force-renewal` argument to create a new copy of the existing certificate.

```bash
~ # certbot certonly --cert-name example.com [--expand | --force-renewal ] ...
```

## Change a Certificate's Domains ##

Add new domains to an existing certificate with argument `--expand`.

```bash
~ # certbot certonly --expand --cert-name example.com --webroot -w /var/www/log.example.com -d blog.example.com --dry-run
```

But the it is recommended to abandon the `--expand` option such that you can either add or remove domains by supplying a complete new list of domains to the `-d` argument.

```bash
~ # certbot certonly --cert-name example.com --webroot -w /var/www/log.example.com -d blog.example.com --dry-run
```

After the above operation, the certificate *example.com* only contains domain name *blog.example.com*.

## Renew a Certificate ##

Let's Encrypt certificates expire after 90 days and can be renewed if a certificiate expires in less than 30 days. Renewing a certificate is actually to generate a new identical copy of the original certificate, using the same arguments which they are created with. The only difference is the expiration date is updated.

A certificate can be manually renewed or automatically renewed.

### Manual Renewal ###

To manually renew an individual certificate, just re-run the command used to obtain it, as in section [Obtain a Certificate](#obtain-a-certificate).

To interactively renew *all* of your certificates, invoke the *renew* subcommand.

```bash
~ # certbot -h renew
~ # cerbot renew --deploy-hook "systemctl reload nginx.service" --dry-run
~ # cerbot renew --deploy-hook /path/to/hook-script.sh --dry-run
```

The `--deploy-hook` reloads the web server if and only if the certificate is successfully renewed. You can add the hook to its configuration file under */etc/letsencrypt/renewal/*:

```
# /etc/letsencrypt/renewal/example.com.conf
 
[renewalparams]
deploy_hook = systemctl reload nginx.service
```

You can also put the hook command in an *executable* script under */etc/letsencrypt/renewal-hooks/deploy/*:

```bash
#!/usr/bin/env bash

systemctl reload nginx.service
```

Or more generally, you can put the hook in Certbot's configuration file located at */etc/letsencrypt/cli.ini* or *~/.config/letsencrypt/cli.ini*.

Once hooks is configured, just run:


```bash
~ # cerbot renew --dry-run
```

Renewal of [standalone certificates](#standalone-authenticator) requires port 80 to be available. In other words, Nginx should not listen on HTTP 80. If that's the case, you can stop Nginx first. Alternatively, we use `--pre-hook` and `--post-hook` arguments.

```bash
~ # cerbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx" --dry-run
```

Attention please; the two hooks will be executed no matter of sucessful or failed renewal. Similarly, you can put the hooks into configuration files or script files.

### Automatic Renwal ###

We can automate the process by *crontab* or *systemd timer*.

Crontab:

```bash
# /var/spool/cron/root

~ # crontab -u root -e
~ # crontab -u root -l

30 0,12 * * * /usr/bin/certbot renew --quiet
```

Systemd timer:

```bash
~ # systemctl list-unit-files --type timer
~ # systemctl status certbot-renew.timer
~ # systemctl enable certbot-renew.timer
~ # systemctl start certbot-renew.timer
~ # systemctl list-timers -all
```

## Backup the Account and Certificates ##

Your account credentials have been saved in your Certbot configuration directory at */etc/letsencrypt/*. You should make a secure backup of this folder now. This configuration directory will also contain certificates and private keys obtained by Certbot so making regular backups of this folder is ideal.

# Install a Certificate

You can update web server's SSL vhost configuration accordingly once the certificate is ready. Here is a template:

```
server {
    listen       443 ssl http2 default_server;
    listen       [::]:443 ssl http2 default_server;
    server_name  www.example.com;
    root         /usr/share/nginx/html;

    ssl_certificate "/etc/letsencrypt/live/www.example.com/fullchain.pem";
    ssl_certificate_key "/etc/letsencrypt/live/www.example.com/privkey.pem";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

Then reload the web server:

```bash
~ # nginx -t
~ # systemctl reload nginx
```

Additionaly, you can redirect all HTTP traffic to HTTPS:

```
return 301 https://$server_name$request_uri;
```

This does not influence certificate renewal but may hinder new certificate application.

If everything goes as expected, turn back on CDN coverage.

# ECDSA Certificate and RSA Certificate #

When it comes to certificates and Signature Algorithms, we categorize them into Elliptic Curve Digital Signature Algorithm (ECDSA) certificate and RSA certificate. ECDSA certificate is also named Elliptic Curve Cryptography (ECC) cerficiate.

ECDSA method outweighs RSA method in two ways:

1. Under certain level of security (in bits), ECDSA has much shorter key length.
2. Less CPU computation and networking load.

Therefore, it is always recommended to obtain a ECDSA certificate as it vastly reduce the time taken to perform TLS handshake and load web pages faster.

For a long, Let's Encrypt does not support ECDSA certificate. However, the situation changed recdently (2020-09).

To obtain a certificate from Let's Encrypt, we have to resort to other ACME clients [other than Certbot](https://community.letsencrypt.org/t/certbot-support-for-ecdsa-certificates/132857). I choose [acme.sh](https://github.com/acmesh-official/acme.sh).

When managing certificates with *acme.sh*, please switch to *root* instead of the *sudo*, *su* method.

Firstly, we install *acme.sh*:

```bash
~ # curl https://get.acme.sh | sh

~ # ll ~/.acme.sh/
~ # crontab -u root -l
```

Then, obtain an ECDSA certificate by [Webroot Authenticator](#webroot-authenticator):

```bash
~ # acme.sh --issue -w /usr/share/nginx/html/ -d blog.example.com --keylength ec-256
```

The parameter `--keylength ec-256` specifies the length of ECDSA key.

Check if the certificate is ECDSA and make sure the value of "Public Key Algorithm" is *id-ecPublicKey* as follows:

```bash
~ # openssl x509 -inform pem -noout -text -fingerprint -md5 < ~/.acme.sh/blog.example.com_ecc/blog.example.com.cer
```

By default, everthing about *acme.sh* is placed under *~/.acme.sh/* by default. This directory is only for internal usage. To make use of the certificate, we should install the cert to Apache/Nginx etc.

The following Nginx configuration is error-prone:

```
ssl_certificate /root/.acme.sh/blog.example.com_cc/fullchain.cer;
ssl_certificate_key /root/.acme.sh/blog.example.com_cc/blog.example.com.key;
```

Nginx will report *permission* error:

```
~ # systemctl reload nginx

[emerg] BIO_new_file ... (SSL: error:0200100D:system library:fopen:Permission denied:fopen('/root/.acme.sh/blog.example.com_ecc/fullchain.cer','r')
```

Instead, execute *acme.sh --install-cert <domain>* to copy the certificate and key into another directory, like:

```bash
~ # mkdir -p /etc/acme.sh/blog.example.com_ecc

~ # acme.sh --install-cert --ecc -d blog.example.com --cert-file /etc/acme.sh/blog.example.com_ecc/cert.pem --key-file /etc/acme.sh/blog.example.com_ecc/key.pem --fullchain-file /etc/acme.sh/blog.example.com_ecc/fullchain.pem --reloadcmd "systemctl reload nginx.service"
```

The destination can be anywhare sensible but the `-d` demands domain. Attention please; the `--ecc` option tells *acme.sh* to copy ECC certificate instead of RSA certificate.

The `--reloadcmd` is critical to tell Nginx reload renewed certificates. Check *~/.acme.sh/en.zhstar.win_ecc/en.zhstar.win.conf* , we will find the reload command is encoded by base64:

```
Le_ReloadCmd='__ACME_BASE64__START_c3lzdGVtY3RsIHJlbG9hZCBuZ2lueC5zZXJ2aWNl__ACME_BASE64__END_'
```

Finally, update Nginx to make use of the copy of the certificate and key under */etc/acme.sh/blog.example.com/*.
