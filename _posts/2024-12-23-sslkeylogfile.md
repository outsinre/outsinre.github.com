---
layout: post
title: Nginx SSLKEYLOGFILE
---

1. toc
{:toc}

In this post, I will walk you through a step-by-step tutorial on how to decrypt TLS traffic of Nginx and its variations.

# SSLKEYLOGFILE #

Environment variable SSLKEYLOGFILE points to a pathname that storing the TLS (Pre)MasterSecret that can be used to decrypt TLS traffic when combined with the captured TLS handshake information.

Generally speaking, there are three ways to support SSLKEYLOGFILE.

1. Capture TLS traffic on client side.

   Clients like mitmproxy (TLS proxy), `openssl s_client`, cURL, Firefox, Chrome, etc. all supports SSLKEYLOGFILE. So, we just need to set this variable before capturing the TLS traffic.
2. Capture TLS traffic on Nginx side.

   Nginx by default does not provide the ability to dump the TLS (Pre)MasterSecret. We have to hack the OpenSSL library.
   1. Make use of 3rd-party Nginx modules [nginx-sslkeylog](https://github.com/tiandrey/nginx-sslkeylog). There is a similar patch available at [PATCH SSL: Added SSLKEYLOGFILE key material to debug logging](https://mailman.nginx.org/pipermail/nginx-devel/2024-January/W5CRPNYOC72XXFF45KQSD3VNNMGJ4WMR.html).
   2. Preload [sslkeylog.c](#sslkeylog.c). Compared to method 2.1, it does not require patching and building. It uses [LD_PRELOAD](https://man7.org/linux/man-pages/man8/ld.so.8.html) to load a new OpenSSL shared object.
3. Make use of [GDB to read TLS (Pre)MasterSecret from core dump](https://security.stackexchange.com/a/80174/248863).

# sslkeylog.c #

In this section, I will build the [sslkeylog.c](https://github.com/Lekensteyn/wireshark-notes/tree/master/src) to the Kong Docker image and enable the TLS (Pre)MasterSecret dumping.

## Dockerfile ##

The Dockerfile is available at [Use sslkeylog.c to Decrypt TLS Traffic](https://gist.github.com/outsinre/bde97c641b1830bb2d4207176ab29969).

```
#7 1.555 + ./sslkeylog.sh curl -sI https://www.google.com
#7 1.757 SERVER_HANDSHAKE_TRAFFIC_SECRET e4d298a7b78a9366948f621f7898b69e09b209144647736892987b7e78f869fe a89651be69b7df70508c77d80f5b518b4a3f337168bafe686710492e91edbca484593fb6fa8d7ad268359d14f9c9fb6c
#7 1.799 EXPORTER_SECRET e4d298a7b78a9366948f621f7898b69e09b209144647736892987b7e78f869fe e37bf351ac25efcbe23d444d9aec1449f16d2aceb69b582ec9623b17343bc3f32161a4f2979694890a141ad11e78cfdf
#7 1.799 SERVER_TRAFFIC_SECRET_0 e4d298a7b78a9366948f621f7898b69e09b209144647736892987b7e78f869fe f8035f7c40f19398f09a0159e4c20d868bb688ef2a176fac4f5312458b7faadf4937c2111450995f2597bff706686fd3
#7 1.799 CLIENT_HANDSHAKE_TRAFFIC_SECRET e4d298a7b78a9366948f621f7898b69e09b209144647736892987b7e78f869fe 5a37d7ae206380701b6d6ab5c12ddfe3a35def0acd7e4497e8dfe926ce19a30ed978d34d3de3530b771efdba403e11c9
#7 1.799 CLIENT_TRAFFIC_SECRET_0 e4d298a7b78a9366948f621f7898b69e09b209144647736892987b7e78f869fe 59c8486ac85d3172b21bf18834b49a606616bfa8a881afd5b25287a171f9fd6a394506cc625172af769fdbd7ff88d6cd
#7 1.899 HTTP/2 200
#7 1.899 content-type: text/html; charset=ISO-8859-1
#7 1.899 content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-1uCcl_kqI4AgRQc8nm8x8A' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
#7 1.899 accept-ch: Sec-CH-Prefers-Color-Scheme
#7 1.899 p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
#7 1.899 date: Tue, 24 Dec 2024 03:25:09 GMT
#7 1.899 server: gws
#7 1.899 x-xss-protection: 0
#7 1.899 x-frame-options: SAMEORIGIN
#7 1.899 expires: Tue, 24 Dec 2024 03:25:09 GMT
#7 1.899 cache-control: private
#7 1.899 set-cookie: AEC=AZ6Zc-VxxkKIvZWzk6Etrr2TaoT-NbBl22ugcfxfOObaAIFDKw3slf3ZTA; expires=Sun, 22-Jun-2025 03:25:09 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
#7 1.899 set-cookie: NID=520=gulgYq_BY9y4P9xdlBnruMrbIl078sk3z4lF_A6bXpsvDoA5F6zzsEQLy1dSQOswGfITfIequq5TiTQ6nud7F4PbPXmDHRsTC5TUIUjbpZaX-33Wn8Ebj0xtT_p5zz-p-TofTwPaFTkfRWz2yOrUZdDIbXw-fxZrCkLPWUf400FskSA7Ef9rLsEi47NCb-FS; expires=Wed, 25-Jun-2025 03:25:09 GMT; path=/; domain=.google.com; HttpOnly
#7 1.899 alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
#7 1.899
```
