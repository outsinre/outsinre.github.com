---
layout: post
title: OpenResty
---

1. toc
{:toc}

This post presents the way to manually build [OpenResty](http://openresty.org) on archlinux. For a quick playaround, try with [Docker image](https://hub.docker.com/r/openresty/openresty).

# Grab Sources #

Download sources.

```bash
# source
14:47:22 outsinre@zhtux ~/misc$ curl -O https://openresty.org/download/openresty-1.21.4.1.tar.gz

# signature
14:47:33 outsinre@zhtux ~/misc$ curl -O https://openresty.org/download/openresty-1.21.4.1.tar.gz.asc
```

# Verify Sources #

[Import and sign](https://www.zhstar.win/2016/02/13/gnupg/#importing-keys) the public PGP key of Yichun Zhang A0E98066

```bash
14:47:33 outsinre@zhtux ~/misc$ gpg --recv-key A0E98066

# locally sign the public key
15:02:08 outsinre@zhtux ~/misc$ gpg --edit-key A0E98066
gpg (GnuPG) 2.2.40; Copyright (C) 2022 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa2048/B550E09EA0E98066
     created: 2013-02-18  expires: never       usage: SC
     trust: unknown       validity: unknown
sub  rsa2048/9C86AC14130DD88B
     created: 2013-02-18  expires: never       usage: E
[ unknown] (1). Yichun Zhang (agentzh) <agentzh@gmail.com>

gpg> lsign 1

pub  rsa2048/B550E09EA0E98066
     created: 2013-02-18  expires: never       usage: SC
     trust: unknown       validity: unknown
 Primary key fingerprint: 2545 1EB0 8846 0026 195B  D62C B550 E09E A0E9 8066

     Yichun Zhang (agentzh) <agentzh@gmail.com>

Are you sure that you want to sign this key with your
key "Zhu Cac (gnupg-rsa-4096) <zhucac@outlook.com>" (38DA5BA4AABFC8FA)

The signature will be marked as non-exportable.

Really sign? (y/N) y

gpg> save
```

Verfiy sources.

```bash
15:07:44 outsinre@zhtux ~/misc$ gpg --verify openresty-1.21.4.1.tar.gz.asc openresty-1.21.4.1.tar.gz
gpg: Signature made Tue 17 May 2022 03:29:41 AM CST
gpg:                using RSA key B550E09EA0E98066
gpg: Good signature from "Yichun Zhang (agentzh) <agentzh@gmail.com>" [full]
```

# Build Binary #

Extract sources.

```bash
15:17:29 outsinre@zhtux ~/misc$ tar -xzpvf openresty-1.21.4.1.tar.gz
15:17:41 outsinre@zhtux ~/misc$ cd openresty-1.21.4.1

15:19:31 outsinre@zhtux ~/misc/openresty-1.21.4.1$ ls -la --color=auto
total 116
drwxrwxr-x  5 outsinre outsinre  4096 May 16  2022 .
drwxr-xr-x  6 outsinre outsinre  4096 Jan  2 15:17 ..
-rw-rw-r--  1 outsinre outsinre 22924 May 16  2022 COPYRIGHT
-rw-rw-r--  1 outsinre outsinre  8974 May 16  2022 README-windows.txt
-rw-rw-r--  1 outsinre outsinre  4689 May 16  2022 README.markdown
drwxrwxr-x 46 outsinre outsinre  4096 May 16  2022 bundle
-rwxrwxr-x  1 outsinre outsinre 51290 May 16  2022 configure
drwxrwxr-x  2 outsinre outsinre  4096 May 16  2022 patches
drwxrwxr-x  2 outsinre outsinre  4096 May 16  2022 util
```

Configure the build. We enable the [debugging log](http://nginx.org/en/docs/debugging_log.html) and dry run. [Make sure](http://openresty.org/en/installation.html#prerequisites) `perl 5.6.1+`, `libpcre`, `libssl` is available on the system.

```bash
15:21:21 outsinre@zhtux ~/misc/openresty-1.21.4.1$ ./configure --help
15:54:41 outsinre@zhtux ~/misc/openresty-1.21.4.1$ ./configure --with-debug --with-pcre-jit --dry-run

15:54:41 outsinre@zhtux ~/misc/openresty-1.21.4.1$ ./configure --with-debug --with-pcre-jit
```

Build from sources.

```bash
16:16:43 outsinre@zhtux ~/misc/openresty-1.21.4.1$ make
16:16:43 outsinre@zhtux ~/misc/openresty-1.21.4.1$ sudo make install
```

Set Env. Program *openresty* is a symbolic link to program *nginx*.

```bash
# .bash_profile

export OPENRESTY_PREFIX=/usr/local/openresty
export PATH=$OPENRESTY_PREFIX/bin:$OPENRESTY_PREFIX/nginx/sbin:$PATH
```

# Quickstart #

Check availability.

```bash
16:31:37 outsinre@zhtux ~$ openresty -V
nginx version: openresty/1.21.4.1
built by gcc 12.2.0 (GCC)
built with OpenSSL 3.0.7 1 Nov 2022
TLS SNI support enabled
configure arguments: --prefix=/usr/local/openresty/nginx --with-debug --with-cc-opt='-DNGX_LUA_USE_ASSERT -DNGX_LUA_ABORT_AT_PANIC -O2' --add-module=../ngx_devel_kit-0.3.1 --add-module=../echo-nginx-module-0.62 --add-module=../xss-nginx-module-0.06 --add-module=../ngx_coolkit-0.2 --add-module=../set-misc-nginx-module-0.33 --add-module=../form-input-nginx-module-0.12 --add-module=../encrypted-session-nginx-module-0.09 --add-module=../srcache-nginx-module-0.32 --add-module=../ngx_lua-0.10.21 --add-module=../ngx_lua_upstream-0.07 --add-module=../headers-more-nginx-module-0.33 --add-module=../array-var-nginx-module-0.05 --add-module=../memc-nginx-module-0.19 --add-module=../redis2-nginx-module-0.15 --add-module=../redis-nginx-module-0.3.9 --add-module=../rds-json-nginx-module-0.15 --add-module=../rds-csv-nginx-module-0.09 --add-module=../ngx_stream_lua-0.0.11 --with-ld-opt=-Wl,-rpath,/usr/local/openresty/luajit/lib --with-pcre-jit --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_ssl_module
```

OpenResty ships with program *resty* (written with Perl) that can execute OpenResty Lua script.

```bash
16:32:16 outsinre@zhtux ~$ resty -e 'local var = 1; print(var)'
1
```

To quickly open a simple HTTP server, follow [OpenResty Getting Started](http://openresty.org/en/getting-started.html).

# macOS #

Check [OpenResty brew tap](https://github.com/openresty/homebrew-brew).

Install.

```bash
~ $ brew tap openresty/brew
~ $ brew install openresty-debug

~ $ type openresty
```

Export Env.

```bash
# ~/.bash_profile

export OPENRESTY_PREFIX=$(brew --prefix openresty/brew/openresty-debug)
PATH="$OPENRESTY_PREFIX/nginx/sbin:$OPENRESTY_PREFIX/luajit/bin/:$PATH"
```
