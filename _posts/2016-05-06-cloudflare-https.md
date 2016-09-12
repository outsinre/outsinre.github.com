---
layout: post
title: Enforce HTTPS to your site by Cloudflare
---

>By default, GitHub takes HTTPS protocol to user/project pages i.e. *username.github.io*. However, for a custiomized domain GitHub page, HTTPS is unsupported. This post tells us to enforce HTTPS to GitHub page through free Cloudflare services. Mainly, we just need to tune a few settings on Clouflare and you domain registrar.

# Cloudflare

[Cloudflare](www.cloudflare.com) offers us free DNS and CDN services whilist concentrating on customer security. Those who would like to enforce HTTPS to his sites could take advantage of those services.

The detailed free services are listed below. To achieve HTTPS, only the first three is a must.
1. [Free automatic HTTPS](https://blog.cloudflare.com/introducing-universal-ssl/) for your domain - no need to buy a certificate;
2. [Page Rules](https://support.cloudflare.com/hc/en-us/articles/200168306-Is-there-a-tutorial-for-Page-Rules-) - custom settings and redirects for URL patterns;
3. [HTTP Strict Transport Security (HSTS)](https://blog.cloudflare.com/enforce-web-policy-with-hypertext-strict-transport-security-hsts/) - protection from MITM attacks;
4. [DNSSEC](https://www.cloudflare.com/dnssec/universal-dnssec/) - protection from DNS poisoning attacks;
5. [HTTP/2](https://www.cloudflare.com/http2/) - optimised connections for browsers that support it;
6. [CNAME flattening](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/) - so you can use a DNS CNAME at the domain apex;
7. ["Always online"](https://www.cloudflare.com/always-online/) protection - Your cached website will stay up even if the host goes down;
8. [Firewall](https://www.cloudflare.com/features-security/) - intelligent protection against DDOS attacks.

We should register a Cloudflare account and then *add site*. Follow the *add site* procedure, we will finish the HTTPS enforcement. Before we go into details, we should clarify a few points:

1. First, write down your custom GitHub page domain/sub-domain, i.e. *example.com* or *blog.example.com*. But top domain is recommended even you are using a sub-domain.
2. Find your domain registrar management interface, i.e. *www.freenom.com*.
3. In the domain management interface, find the setting of DNS servers and DNS records.

We will basically do two things:

1. Switch to Cloudflare DNS servers. Subsequently, DNS records will be managed on Cloudflare. You'd best delete the old DNS records.
2. Turn off a few security settings on Cloudflare. That' all!

# DNS switch

1. After filling in your site url following *add site* procedure, Cloudflare will analyze and import the site's DNS records (set on other platforms).

   We can add, delete a new DNS records. Even, we can toggle Cloudflare per a record.
2. Then continue, Cloudflare will give us two new Cloudflare DNS servers.
3. Go to our domain registrar management interface, and replace all original DNS servers with the new ones.

   To make use of Cloudflare service, all other platforms' DNS servers must be deleted. Only Cloudflare ones are permitted.
4. Up to now, our domain DNS servers are taken over by Cloudflare.

# SSL settings

Unfortunately GitHub Pages doesn’t yet support SSL on GitHub Pages for custom domains which would ordinarily rule out using HTTP/2. Whilst the HTTP/2 specification (RFC 7540) allows for HTTP/2 over plain-text HTTP/2, all popular browsers require HTTP/2 to run on top of Transport Layer Security; meaning HTTP/2 only being able to run over HTTPS is the de-facto standard.

1. In the Crypto tab of your CloudFlare site you should ensure your SSL mode is set to *Full* but not *Full (strict)*.
2. We can now add a Page Rule to enforce HTTPS, as you add other Page Rules make sure this is the primary Page Rule:

   ```
   http://*fxhu.tk/*

   Always Use HTTPS
   ```

3. We can also create a Page Rule to ensure that non-www is redirected to www securely when using HTTPS:

   ```
   https://fxhu.tk

   Forwarding URL (Status Code: 301 - Permanent Redirect)

   https://www.fxhu.tk
   ```

4. Back to the Crypto tab, enable and set HTTP Strict Transport Security (HSTS) service. HSTS (RFC 6797) is a header which allows a website to specify and enforce security policy in client web browsers.

   The recommended settings are:

   ```
   Status: On
   Max-Age: 6 months (recommended)
   Include subdomains: On
   Preload: On
   No-sniff: On
   ```

   That's all.
5. Test

   Though we should wait for a while (hours maybe) to acccess GitHub pages over HTTPS, we can test above setting by `curl -I example.com`:

   ```
   HTTP/1.1 301 Moved Permanently
   Date: Mon, 12 Sep 2016 11:14:57 GMT
   Connection: keep-alive
   Set-Cookie: __cfduid=afj4n38eahdglvmneuptoq84h; expires=Tue, 12-Sep-17 11:14:57 GMT; path=/; domain=.exmaple.com; HttpOnly
   Location: https://example.com/
   X-Content-Type-Options: nosniff
   Server: cloudflare-nginx
   CF-RAY: 3fien1q9ehpen-HKG
   ```
   
# Cache all the things

CloudFlare has a “Cache Everything” option in Page Rules. For static sites, it allows your HTML to be cached and served directly from CloudFlare's CDN. This will significantly accelerate your site access time.

Add a last rule as:

```
https://*fxhu.tk/*

Cache Level: Cache Everything
```

When deploying your site you can use the Purge Cache option in the Cache tab on CloudFlare to remove the cached version of the static pages.

# DNSSEC

If DNS is the phone book of the Internet, DNSSEC is the protocol that ensures that a number in the phone book actually belongs to the contact listed. DNSSEC uses cryptographic signatures to verify that the DNS records returned for a domain are untampered.

If your domain registrar support DNSSEC, please turn on DNSSEC on Cloudflare DNS tab.

# Refs

1. [Cloudflare blog](https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare/)
2. [robinwinslow](https://robinwinslow.uk/2016/02/13/free-https-custom-hosting/)
3. [sheharyar](https://sheharyar.me/blog/free-ssl-for-github-pages-with-custom-domains/)

