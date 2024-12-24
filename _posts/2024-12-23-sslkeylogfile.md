---
layout: post
title: Nginx SSLKEYLOGFILE
---

1. toc
{:toc}

In this post, I will walk you through a step-by-step tutorial on how to decrypt TLS traffic of Nginx and its variations.

# SSLKEYLOGFILE #

Environment variable SSLKEYLOGFILE points to a pathname storing the TLS (Pre)MasterSecret (Session Secret) that can be used to decrypt TLS traffic when combined with the captured TLS handshake information.

Generally speaking, there are three ways to support SSLKEYLOGFILE.

1. Capture TLS traffic on client side.

   Clients like mitmproxy (TLS proxy), `openssl s_client`, cURL, Firefox, Chrome, etc. all supports SSLKEYLOGFILE. So, we just need to set this variable before launching the client and before capturing the TLS traffic.
2. Capture TLS traffic on Nginx side.

   Nginx by default does not provide the ability to dump the TLS (Pre)MasterSecret. We have to hack the OpenSSL library.
   1. Make use of 3rd-party Nginx modules like [nginx-sslkeylog](https://github.com/tiandrey/nginx-sslkeylog). There is another similar patch available at [PATCH SSL: Added SSLKEYLOGFILE key material to debug logging](https://mailman.nginx.org/pipermail/nginx-devel/2024-January/W5CRPNYOC72XXFF45KQSD3VNNMGJ4WMR.html).
   2. Preload [sslkeylog.c](#sslkeylog.c). Compared to method 2.1, it does not require patching and building. It uses [LD_PRELOAD](https://man7.org/linux/man-pages/man8/ld.so.8.html) to preload a new shared library to Nginx.
3. Make use of [GDB to read TLS (Pre)MasterSecret from core dump](https://security.stackexchange.com/a/80174/248863).

# sslkeylog.c #

In this section, I will build the [sslkeylog.c](https://github.com/Lekensteyn/wireshark-notes/tree/master/src) to the Kong Docker image and enable the TLS (Pre)MasterSecret dumping.

## Dockerfile ##

The Dockerfile is available at [sslkeylogfile.Dockerfile](https://gist.github.com/outsinre/bde97c641b1830bb2d4207176ab29969).

In the build stage, the script [sslkeylog.sh](#test-image) is used to verify the built object.

```
./sslkeylog.sh curl -sI https://www.google.com
```

The test output from the [building process](#build-image) is as follows.

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

## Build Image ##

```bash
~ $ BUILDKIT_PROGRESS=plain docker buildx build --no-cache --load -t 'kong-custom:3.9.0.0' -f sslkeylogfile.Dockerfile .

~ $ docker images 'kong-custom'
REPOSITORY    TAG       IMAGE ID       CREATED          SIZE
kong-custom   3.9.0.0   b85219c906ce   34 minutes ago   498MB
```

## Test Image ##

Create a container.

```bash
~ $ docker run --rm \
--name sslkey \
--entrypoint /bin/bash \
-it kong-custom:3.9.0.0

kong@72a118f01128:/$ kong version
Kong Enterprise 3.9.0.0

kong@72a118f01128:/usr/local/kong/lib$ ls -l libsslkeylog.so
-rwxr-xr-x 1 root root 71736 Dec 24 03:25 libsslkeylog.so
```

Import the script [sslkeylog.sh](https://github.com/Lekensteyn/wireshark-notes/blob/master/src/sslkeylog.sh).

```bash
kong@72a118f01128:/usr/local/kong/lib$ pwd
/usr/local/kong/lib

kong@72a118f01128:/usr/local/kong/lib$ nano -w sslkeylog.sh
...

kong@72a118f01128:/usr/local/kong/lib$ chmod +x sslkeylog.sh
```

Ensure the script and the build object reside in the same directory - `/usr/local/kong/lib`. You can optionally provide "SSLKEYLOGFILE" to specify a pathname, otherwise the dumped (Pre)MasterSecret is printed to the stderr.

### Test TLS 1.3 ###

```bash
kong@72a118f01128:/usr/local/kong/lib$ :> /tmp/premaster.txt

kong@72a118f01128:/usr/local/kong/lib$ SSLKEYLOGFILE=/tmp/premaster.txt ./sslkeylog.sh /usr/local/kong-tools/bin/curl -sI --tlsv1.3 https://www.google.com
HTTP/2 200
content-type: text/html; charset=ISO-8859-1
content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-DUWj1U9UPGVMwCY29p_R8Q' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
accept-ch: Sec-CH-Prefers-Color-Scheme
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
date: Tue, 24 Dec 2024 06:30:07 GMT
server: gws
x-xss-protection: 0
x-frame-options: SAMEORIGIN
expires: Tue, 24 Dec 2024 06:30:07 GMT
cache-control: private
set-cookie: AEC=AZ6Zc-Xnia3ReOwqLapj8yFlujpG9zCCT-Rq3eBye07oMwzr-WLr1OnluLA; expires=Sun, 22-Jun-2025 06:30:07 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
set-cookie: NID=520=aAOm9hYIhcaWIbS4Mfxf3juDrMpiQJUc8Zs0TTLliXo1wgOdaqmqfaHsrSWvRU8gWzmrR0gwqxBySnfQHrgt7_aS8lIiu1yYnU06APwBYMcv6kFvmCR6w1H8100azBwocZ4ScXG6-NqPj8T13hkMZNn-1Noi4TCdXRfzIZZqsygShH_hQ50vnjGiNdjOkeE; expires=Wed, 25-Jun-2025 06:30:07 GMT; path=/; domain=.google.com; HttpOnly
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000

kong@72a118f01128:/usr/local/kong/lib$ cat /tmp/premaster.txt
# SSL key logfile generated by sslkeylog.c
SERVER_HANDSHAKE_TRAFFIC_SECRET 6755ae538171e4100113858b989890d9ddbbb80a5f71180f8c45c5d29220d538 bc84ec978a446acf0b0da5a674b236b75d73ba0bcb949ebb8c73edf127987b8fec8af48c24e4995485c4132ae865ad8e
EXPORTER_SECRET 6755ae538171e4100113858b989890d9ddbbb80a5f71180f8c45c5d29220d538 9b14d21410073a8d7f9f1e8b6885d8240b7cfbf723a709fa538382fd4a4a01d5f8c70752817a5a4b7db5626d8b504e6f
SERVER_TRAFFIC_SECRET_0 6755ae538171e4100113858b989890d9ddbbb80a5f71180f8c45c5d29220d538 c5a31a6c65853d4386c31d2916bfb6e8a4bd900c3195ecb7daf41e613346cf24fdf3bbeca05b4e02f8c898c942bf0d9f
CLIENT_HANDSHAKE_TRAFFIC_SECRET 6755ae538171e4100113858b989890d9ddbbb80a5f71180f8c45c5d29220d538 c05914e620691f1020342c45e1177f113f88281a90885c40f5e5e6f5d102774b0b362c399a589be9174699f7cd63d3e5
CLIENT_TRAFFIC_SECRET_0 6755ae538171e4100113858b989890d9ddbbb80a5f71180f8c45c5d29220d538 1848c3e4cd96112141cafd0a89212e839a258be5b409701ca4e3fe6531e08448de2ea9f95b293a4c0eb091e12446050d
kong@72a118f01128:/usr/local/kong/lib$
```

Each TLS session needs 5 secrets to decrypt traffic.

![nginx-sslkeylogfile.png](/assets/nginx-sslkeylogfile.png)

Let import the dumped (Pre)MasterSecret into the PCAP file.

```bash
14:50:44 zachary@Zacharys-MacBook-Pro ~/misc
$ ls -l sslkey.pcap
-rw-r--r--  1 zachary  staff  8938 Dec 24 14:47 sslkey.pcap
14:50:46 zachary@Zacharys-MacBook-Pro ~/misc
$ docker cp sslkey:/tmp/premaster.txt .
Successfully copied 2.56kB to /Users/zachary/misc/.
14:50:56 zachary@Zacharys-MacBook-Pro ~/misc
$ ls -l premaster.txt
-rw-r--r--  1 zachary  staff  981 Dec 24 14:47 premaster.txt
14:51:00 zachary@Zacharys-MacBook-Pro ~/misc
$ editcap --inject-secrets tls,premaster.txt sslkey.pcap sslkey-dsb.pcap
15:05:18 zachary@Zacharys-MacBook-Pro ~/misc
$ ls -l sslkey-dsb.pcap
-rw-r--r--  1 zachary  staff  10472 Dec 24 15:05 sslkey-dsb.pcap
```

Use Wireshark to decrypt and analyze the TLS traffic.

![nginx-sslkey-dsb.png](/assets/nginx-sslkey-dsb.png)

### Test TLS 1.2 ###

```bash
kong@72a118f01128:/usr/local/kong/lib$ :> /tmp/premaster.txt

kong@72a118f01128:/usr/local/kong/lib$ SSLKEYLOGFILE=/tmp/premaster.txt ./sslkeylog.sh /usr/local/kong-tools/bin/curl -sI --tlsv1.2 --tls-max 1.2 https://www.google.com
HTTP/2 200
content-type: text/html; charset=ISO-8859-1
content-security-policy-report-only: object-src 'none';base-uri 'self';script-src 'nonce-zXzn-lr53oUa0iPPjL-jYg' 'strict-dynamic' 'report-sample' 'unsafe-eval' 'unsafe-inline' https: http:;report-uri https://csp.withgoogle.com/csp/gws/other-hp
accept-ch: Sec-CH-Prefers-Color-Scheme
p3p: CP="This is not a P3P policy! See g.co/p3phelp for more info."
date: Tue, 24 Dec 2024 06:35:02 GMT
server: gws
x-xss-protection: 0
x-frame-options: SAMEORIGIN
expires: Tue, 24 Dec 2024 06:35:02 GMT
cache-control: private
set-cookie: AEC=AZ6Zc-XYq2E4RNGSN48i3AiXeBdwUJJsDimdVa_UQSnVCt2Rx6fqu1j1Lg; expires=Sun, 22-Jun-2025 06:35:02 GMT; path=/; domain=.google.com; Secure; HttpOnly; SameSite=lax
set-cookie: NID=520=Vn8bZrsr2eiT_doMDp6KS8Jvo5lXFMX2z-acFRLxWxwYuGfnFWKwdmd8Y8eZV-kgphfnmPqVjW-0689KjGfUmcZVSHsuz-JMFKxN3CIbczHD2VIJqywOBeIZv3ef5IHr0MdcKNvJffcra9mp-X3QFolGZUylIbjkRybTgY14OIUVfZ2pr3tXSAKVKGk2OCE; expires=Wed, 25-Jun-2025 06:35:02 GMT; path=/; domain=.google.com; HttpOnly
alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000

kong@72a118f01128:/usr/local/kong/lib$ cat /tmp/premaster.txt
# SSL key logfile generated by sslkeylog.c
CLIENT_RANDOM 373ad1cfde82cc6ac98ed54f3b5dbfda9eabee9fe49886b1d3c040aa7bef7ddd 6f526e02d325fbaf0e5406ba6f644ba5c8d5a3c8d2fb534913ba47d9ed8f97f5fff36a058f7625cd974d3fea508d2d43
```

Each TLS session just needs 1 secret to decrypt traffic.
