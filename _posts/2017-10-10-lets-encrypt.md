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
3. [Domain Control Validation (DCV)](https://support.sectigo.com/Com_KnowledgeDetailPageFaq?Id=kA01N000000brbt). [Challenge ownership](https://docs.digicert.com/manage-certificates/organization-domain-management/managing-domains-cc-guide/domain-pre-validation-domain-control-validation/) of domain or web server.
4. Automatically renew certificates.

Let's Encrypt officially recommends the [Certbot](https://certbot.eff.org/docs/using.html) client.

```bash
~ # dnf --enablerepo ol8_developer_EPEL search certbot
~ # dnf --enablerepo ol8_developer_EPEL install certbot
~ # dnf repoquery -l certbot

~ # certbot -h all
~ # certbot certificates # List existing certificates
```

To run *certbot*, we supply a subcommand like *certonly* (obtain a certificate), *install* (update vhost), *run* (both) etc. Whenever possible, try with `--dry-run` first.

If no subcommand is given, then *run* is assumed. A subcommand accepts different types of plugins, namely *authenticator* plugins and *installer* plugins. The general usage looks like:

```
certbot subcommand --plugin-name ...
```

The *certonly* subcommand and *install* subcommand accept *authenticator* plugins and *installer* plugins respectivelly, while the *run* subcommand accepts plugins belonging to both types.

*authenticator* plugins challenge you whether you are eligible for a certificate by verifying the domain owership. *installer* plugins automatically modify web server's configuration with specified certificate. Most of the time, we firstly use *authenticator* plugins to obtain a certificate and then *manually* update configurations of the web server. We don't use *installer* plugins/

Examples of *authenticator* plugins are `--webroot` and `--standalone`. There does not exist plugins exclusively belonging to the *installer* type. However, `--apache` and `--nginx` belong to both types of *authenticator* and *installer*, and can be used with the *run* subcommand to both obtain and install a certificate.

The following table simply presents their relationship:

| --- | --- | --- |
| subcommand | type | plugin |
| :--- | :--- | :--- |
| certonly | authenticator | webroot, standalone, dns-cloudflare |
| install | installer | n/a |
| run (default) | both | apache, nginx |
| --- | --- | --- |

## Certbot Configuration ##

By default, the configuration is:

```bash
~ $ cat /etc/letsencrypt/cli.ini
preconfigured-renewal = True
```

A few custom configs:

```
# /etc/letsencrypt/cli.ini

# TOS
agree-tos = true

# cert notification
email = alice@example.com

# ECC key by default
key-type = ecdsa
elliptic-curve = secp384r1

# RSA key size
rsa-key-size = 4096

# renewal hook
deploy-hook = systemctl reload nginx.service
```

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

Use the `--webroot` authenticator if you have full control over the *running* web server and the domain. Let's Encrypt's ACME server tells the Certbot client to write *unique* files under the root of your web server (i.e. */usr/share/nginx/html/*). This step is challenging you whether you do own the web server (i.e. write access). The file URL takes the form:

```
http://domain/.well-known/acme-challenge/<file>
```

Afterwards, the ACME server tries to fetch the URL, verifying you own the domain. Therefore, make sure the domain name is finally resolved to the web server IP and the web server is running on HTTP 80 (NOT HTTPS 443). Though, the *webroot* authenticator use HTTP to challenge the ownership, the security of Certbot itself is guranteed by ACME protocol. We call this kind of challenge *http-01*.

You can cover your web server with CDN (i.e. Cloudflare), as the challenge method is to download the unique file. However, if the CDN enables HSTS, then temporarily turn it off by removing the caching capability in DNS settting.

Once downloaded, the ACME server also compares the file hashes of the fetched copy with its local store.

## Standalone Authenticator ##

The `--standalone` authenticator is usually used when the host where you run *certbot* is not the one you would like to host your web server.

```bash
~ # certbot -h standalone
~ # certbot certonly --standalone -d www.example.com,blog.example.com --dry-run
```

It starts a *temporary standalone web server* to talk to Let’s Encrypt. Therefore, it does not verify web server. You must make sure [port 80](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-7.2) is available. You may have to turn down existing web servers to release port 80.

Recall that `--webroot` challenge domain onwership by *http-01*, GETting an unique URL. Then how does `--standalone` challenge the domain onwership? By *http-01* too!

Though ACME protocol supports challenge with [tls-01](https://tools.ietf.org/html/draft-ietf-acme-acme-03#section-7.3) by verifying the *temporarily* self-generated certificate, but Certbot only implements HTTP 80. If you have chosen to use *tls-o1*, then you must make sure the domain name is *directly* resolved to the host IP where you apply for certificates *certbot* client. If the domain name is resolved to the host IP, it means you manage the domain. You cannot cover your domain with CDN, otherwise the ACME server would got the CDN's certificate instead of the temporary one by Certbot.

## DNS Authenticator for Wildcard Certificate ##

With the *certbot* client, the only way to obtain a wildcard certificate from Let's Encrypt is using DNS Plugins. DNS plugins belong to the *authenticator* type but challenge you by DNS protocol.

Basically, a DNS plugin uses a API token from the DNS platform to first add a TXT record and then remove that record, such that you are proved to own the domain name. Therefore, the DNS authenticator does not interfere in web server.

For each DNS platform, *certbot* provodes a corresponding DNS plugin. DNS plugins are not installed by default.

Take [Cloudflare](https://certbot-dns-cloudflare.readthedocs.io/en/stable/) for example. Firstly, let's install DNS plugin:

```bash
~ $ sudo dnf --enablerepo ol8_developer_EPEL install python3-certbot-dns-cloudflare
```

Then create API token for the DNS plugin at [API Token](https://dash.cloudflare.com/?to=/:account/profile/api-tokens). Please configure the IP whitelist. We test the token:

```bash
~ $ curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \
>      -H "Authorization: Bearer xxxxxxxxxxxxxyyyyyyyyyyyyyzzzzzzzzzzzzz" \
>      -H "Content-Type:application/json"

~ $ curl -X GET "https://api.cloudflare.com/client/v4/zones/" \
>      -H "Authorization: Bearer xxxxxxxxxxxxxyyyyyyyyyyyyyzzzzzzzzzzzzz" \
>      -H "Content-Type:application/json"
```

The token is used for both cert creation and renewal. Store the token somewhere:

```bash
~ # mkdir -p ~/root/.secrets/certbot
~ # chmod 700 ~/root/.secrets/certbot

~ # cat > /root/.secrets/certbot/cloudflare.ini <<EOF
> # Cloudflare API token used by Certbot
> dns_cloudflare_api_token = xxxxxxxxxxxxxyyyyyyyyyyyyyzzzzzzzzzzzzz
> EOF

~ # chmod 600 /root/.secrets/certbot/cloudflare.ini
```

Finally create wildcard certificate:

```bash
~ # certbot certonly --dry-run \
--dns-cloudflare \
--dns-cloudflare-credentials /root/.secrets/certbot/cloudflare.ini \
--dns-cloudflare-propagation-seconds 60 \
--domains example.com,*.example.com \
--cert-name example.com \
--key-type ecdsa \
--elliptic-curve secp384r1 ; echo
```

# Manage a Certificate #

No matter which plugins you use, the generated certificates are placed under */etc/letsencrypt/*. Also the arguments used to generate the certificates are stored alongside for [latter renwal](#renew-a-certificate). The directory tree looks like:

```
# tree /etc/letsencrypt/
/etc/letsencrypt/
├── accounts
│   ├── acme-staging-v02.api.letsencrypt.org
│   │   └── directory
│   │       └── 74cfecb0fe45ec846ae6cc76b6ba0b44
│   │           ├── meta.json
│   │           ├── private_key.json
│   │           └── regr.json
│   └── acme-v02.api.letsencrypt.org
│       └── directory
│           └── 34ec5a2656cb495bb9c7b74755d5dfad
│               ├── meta.json
│               ├── private_key.json
│               └── regr.json
├── archive
│   └── example.com
│       ├── cert1.pem
│       ├── chain1.pem
│       ├── fullchain1.pem
│       └── privkey1.pem
├── cli.ini
├── csr
│   └── 0000_csr-certbot.pem
├── keys
│   └── 0000_key-certbot.pem
├── live
│   ├── README
│   └── example.com
│       ├── cert.pem -> ../../archive/example.com/cert1.pem
│       ├── chain.pem -> ../../archive/example.com/chain1.pem
│       ├── fullchain.pem -> ../../archive/example.com/fullchain1.pem
│       ├── privkey.pem -> ../../archive/example.com/privkey1.pem
│       └── README
├── renewal
│   └── example.com.conf
└── renewal-hooks
    ├── deploy
    ├── post
    └── pre

18 directories, 20 files
```

You will find a certificate has different version, namely *fullchain.pem*, *chain.pem* and *cert.pem*. Use *fullchain.pem* [whenever possible](https://github.com/v2ray/v2ray-core/issues/509#issuecomment-319321002) as it is a combination of *chain.pem* and *cert.pem*. Use *chain.pem* for OCSP stapling as that does not require the leaf cert.

First, list existing certificates:

```bash
~ # certbot certificates
~ # certbot certificates --cert-name example.com
```

It's not unusual that different certificates share common FQDNs. From the output, you will find each certificate has a name. By default, it's named afer the first FQDN from option `--domains`. We can explicitly set a certificate name with option `--cert-name`.

## Change a Certificate's Domains ##

Add new domains to an existing certificate with argument `--expand`.

```bash
~ # certbot certonly --expand --cert-name example.com --webroot -w /var/www/log.example.com -d blog.example.com --dry-run
```

But the it is recommended to abandon the `--expand` option such that you can either add or remove domains by supplying a complete new list of domains to the `-d` option.

```bash
~ # certbot certonly --cert-name example.com --webroot -w /var/www/log.example.com -d blog.example.com --dry-run
```

After the above operation, the certificate *example.com* only contains domain name *blog.example.com*. It actually just creates a new certificate with the same name!

## Renew a Certificate ##

Let's Encrypt certificates expire after 90 days and can be renewed if a certificiate expires in less than 30 days. Renewing a certificate is actually to generate a new identical copy of the original certificate, using the same arguments which they are created with. The only difference is the expiration date is updated.

Renewal just create a new certificate with the same name! So we can just re-run the command used to obtain the certificate, as in section [Obtain a Certificate](#obtain-a-certificate).

But sometimes, we need fine control over renewal. Certbot stores a renewal configuration for each certificate:

```bash
~ # realpath example.com.conf
/etc/letsencrypt/renewal/example.com.conf
```

A certificate can be manually renewed or automatically renewed.

### Manual Renewal ###

To interactively renew *all* of your certificates, invoke the *renew* subcommand.

```bash
~ # certbot -h renew
~ # cerbot renew --deploy-hook "systemctl reload nginx.service" --dry-run
~ # cerbot renew --deploy-hook /path/to/hook-script.sh --dry-run
```

The `--deploy-hook` will execute a program if and only if the certificate is successfully renewed. You can add the hook to its renewal configuration file.

```
# /etc/letsencrypt/renewal/example.com.conf
 
[renewalparams]
deploy_hook = systemctl reload nginx.service
```

You can also put the hook into */etc/letsencrypt/renewal-hooks/deploy/*. For example, here is a hook script:

```bash
#!/usr/bin/env bash

systemctl reload nginx.service
```

Or more generally, you can put the hook in [Certbot Configuration](#certbot-configuration) file.

Once hooks is configured, just run:


```bash
~ # cerbot renew --dry-run
```

Renewal of [standalone certificates](#standalone-authenticator) requires port 80 to be available. In other words, Nginx should not listen on HTTP 80. If that's the case, you can stop Nginx first. Alternatively, we use `--pre-hook` and `--post-hook` arguments.

```bash
~ # cerbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx" --dry-run
```

Attention please; unlike the `--deploy-hook` hook, the `--pre-hook` and `--post-hook` will be executed no matter of sucessful or failed renewal. Similarly, you can put the hooks into configuration files or script files.

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

The *certbot-renew.timer* would invoke *certbot-renew.service* automatically.

## Backup the Account and Certificates ##

Your account credentials have been saved in your Certbot configuration directory at */etc/letsencrypt/*. You should make a secure backup of this folder now. This configuration directory will also contain certificates and private keys obtained by Certbot so making regular backups of this folder is ideal.

## Share Certificates ##

Say we have obtained a certificate and configured auto-renewal on server A, but want to use the same certificate on server B. How to [achive this](https://stackoverflow.com/q/37391158)?

Basically, we have two methods.

1. Write a [deploy-hook](#manual-renewal) to *rsync* renewed certificate and key to server B.
2. Start from scratch on server B to obtain new certificate with the same procedure in this post. Don't worry about conflicts as obtaining a new certificate justs create a brand new one.

   The only concern is the number limit of renewal per-certificate. Currently, Let's Encrypte allows 5 renewal per month for an individual certificate.

# Deploy Certificates #

You can update web server's SSL vhost configuration accordingly once the certificate is ready as below. We can follow [Mozilla SSL Config Recommendation](https://ssl-config.mozilla.org/) to get a secure and comptabile template.

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

>As of version 1.10, Certbot supports ECDSA certificates.

When it comes to certificates and Signature Algorithms, we categorize them into Elliptic Curve Digital Signature Algorithm (ECDSA) certificate and RSA certificate. ECDSA certificate is also named Elliptic Curve Cryptography (ECC) cerficiate.

The two kinds of certificate differ in the type of public/private keys. When generating a key pair, we can choose ECDSA algorithm or RSA algorithm. In return, the key algorithm affecrs almost any security operations, like encryption, signature etc.

[ECDSA outweighs RSA](https://hackernoon.com/rsa-and-ecdsa-hybrid-nginx-setup-with-letsencrypt-certificates-ee422695d7d3) in two ways:

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

How do I know if the certificate is ECDSA-based or RSA-based? Just check the "Public Key Algorithm" field, and make sure the field value is *id-ecPublicKey*, as follows:

```bash
~ # openssl x509 -inform pem -noout -text -fingerprint -md5 < ~/.acme.sh/blog.example.com_ecc/blog.example.com.cer
```

You may find that, the public key is just 256-bit long.

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

Instead, execute `--install-cert <domain>` that will *copy* the certificate and key into another directory, like:

```bash
~ # mkdir -p /etc/acme.sh/blog.example.com_ecc

~ # acme.sh --install-cert --ecc -d blog.example.com --cert-file /etc/acme.sh/blog.example.com_ecc/cert.pem --key-file /etc/acme.sh/blog.example.com_ecc/key.pem --fullchain-file /etc/acme.sh/blog.example.com_ecc/fullchain.pem --reloadcmd "systemctl reload nginx.service"
```

The destination can be anywhare but the `-d` demands domain name. Attention please; the `--ecc` option tells *acme.sh* to copy ECC certificate instead of RSA certificate, by looking for a directory with `_ecc` suffix like `blog.example.com_ecc`.

*acme.sh* maintains two copies of a certificate, one for internal usage, one for webserver. On the contrary, *certbot* use symblic!

The `--reloadcmd` is critical to tell Nginx reload renewed certificates. Check *~/.acme.sh/en.example.com_ecc/en.example.com.conf* , we will find the reload command is encoded by base64:

```
Le_ReloadCmd='__ACME_BASE64__START_c3lzdGVtY3RsIHJlbG9hZCBuZ2lueC5zZXJ2aWNl__ACME_BASE64__END_'
```

Finally, update Nginx to make use of the copy of the certificate and key under */etc/acme.sh/blog.example.com/*.
