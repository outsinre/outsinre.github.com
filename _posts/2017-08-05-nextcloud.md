---
layout: post
title: Nextcloud
---

>Nextcloud is open source platform serving file (private cloud disk), contacts, calendar synchronization ans sharing. By Nextcloud, you control your information, being not subject to privacy leak.

# [Requirements -LNMP](https://docs.nextcloud.com/server/12/admin_manual/installation/source_installation.html)

1. L: CentOS 7 64-bit;
2. N: [Nginx](/2017/04/11/nginx/);
3. M: MariaDB;
4. P: Php-FPM.

# MariaDB

```bash
~ # yum install mariadb mariadb-server
~ # systemctl status mariadb
~ # systemctl start mariadb
~ # systemctl enable mariadb
```

## MariaDB configuration

Create password for MariaDB *root* user:

```
~ # mysql_secure_installation
Enter current password for root (enter for none): # just ENTER
Set root password? [Y/n] Y
New password: 
Re-enter new password: 
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

With the MariaDB root password just set, now we can login to the *mysql* shell to create a database and a user for Nextcloud.

```
~ # mysql -u root -pPassword
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 14
Server version: 5.5.52-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> help
```

Now [creating](https://www.digitalocean.com/community/tutorials/how-to-create-and-manage-databases-in-mysql-and-mariadb-on-a-cloud-server) database and user with SQL within SQL shell:

```
create database nc_db
# or
create database nc_db default character set utf8mb4 collate utf8mb4_unicode_ci;
create user nc_user@localhost identified by 'password';
grant all privileges on nc_db.* to nc_user@localhost identified by 'password';
flush privileges;
show databases;
show tables from nc_db;
select user,host,password,plugin from mysql.user;
show grants for nc_user@localhost;
quit;
```

1. Create a database and user for Nextcloud service. Grant all privileges of the database to the user.
2. If the user account does exist and you provide the IDENTIFIED BY clause, the user's password will be changed.

   So use the same 'password' value in case of losing it.

# Php-fpm

Make sure [webtatic repository](/2017/04/05/centos7/) is enabled and check [webtatic packages](https://webtatic.com/packages/php70/).

```bash
~ # yum --enablerepo=webtatic install php70w-fpm php70w-common
~ # yum --enablerepo=webtatic install php70w-ctype php70w-dom php70w-gd php70w-iconv php70w-json php70w-libxml php70w-mbstring php70w-posix php70w-simplexml php70w-xmlreader php70w-xmlwriter php70w-zip php70w-zlib
~ # yum --enablerepo=webtatic install php70w-mysql php70w-curl php70w-intl php70w-mcrypt php70w-opcache php70w-pecl-apcu
~ # php -v/i
~ # php -m | grep -i apcu
```

1. Some Php modules (i.e. php-70w-curl) does not exist but enclosed by another module (i.e. php70w-common).
2. *opcache* module won't be installed without explicit pointing out, which would cause [APCu memory exhausted issue](https://github.com/nextcloud/server/issues/5249).

## Php-fpm configuration

Edit Php-fpm configuration file:

```
# /etc/php-fpm.d/www.conf

user = nginx
group = nginx

listen = 127.0.0.1:9000

env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/bin:/usr/bin:/bin
env[TMP] = /tmp
env[TMPDIR] = /tmp
env[TEMP] = /tmp
```

Next, create a new directory for the session path in the */var/lib/* directory, and change the owner to the *nginx* user:

```bash
~ # mkdir -p /var/lib/php/session
~ # chown nginx:nginx -R /var/lib/php/session/
```

Launch *nginx* and *php-fpm*:

```bash
# php-fpm
~ # systemctl status php-fpm
~ # systemctl enable php-fpm
~ # systemctl start php-fpm
# nginx
~ # systemctl status nginx
~ # systemctl enable nginx
~ # systemctl start nginx
```

There existed an error when starting *php-fpm*:

```
Aug 05 05:52:29 localhost php-fpm[7613]: [05-Aug-2017 05:52:29] NOTICE: PHP message: PHP Fatal error:  PHP Startup: apc_shm_create: shmget(0, 67108864, 914) failed: Invalid argument. It is possible that the chosen SHM segment size is higher than the operation system allows. Linux has usually a default limit of 32MB per segment. in Unknown on line 0
Aug 05 05:52:29 localhost php-fpm[7613]: [05-Aug-2017 05:52:29] NOTICE: PHP message: PHP Fatal error:  PHP Startup: apc_shm_attach: shmat failed: in Unknown on line 0
```

That was caused by *shm* size (32M) of CentOS 7 kernel much smaller than that (64M) of Php APC user cache (APCu) setting. Kernel's *shm* value can be verified by

```bash
~ # sysctl -a | grep -i shm
# kernel.shmmax = 33554432
# kernel.shmall = 2097152
# kernel.shmmni = 4096
# or
~ # cat /proc/sys/kernel/shmmax
```

The upper bond here is 33554432 Bytes (32M). However, the value of APCu *shm* size is:

```
# /etc/php.d/apcu.ini
# The size of each shared memory segment with M/G suffix.
apc.shm_size=64M
```

So we can either cut down APCu *shm* size or increase that of kernel:

```bash
~ # sysctl -w kernel.shmmax=67108864
# To be permanent across reboot:
# vi /etc/sysctl.d/15-shmmax.conf
kernel.shmmax = 67108864
~ # sysctl -f /etc/sysctl.d/15-shmmax.conf
```

# Install Nextcloud tarball

```bash
~ # wget https://download.nextcloud.com/server/releases/nextcloud-12.0.1.tar.bz2
~ # yum install bzip2
~ # tar -xjpvf nextcloud-12.0.1.tar.bz2 -C /usr/share/nginx/html/
~ # chown nginx:nginx -R /usr/share/nginx/html/nextcloud/
```

Create *data* folder of Nextcloud *outside* of web root:

```bash
~ # mkdir -p /opt/nextcloud-data
~ # chown nginx:nginx -R /opt/nextcloud-data/
```

# Nginx configuration for Nextcloud

```bash
~ # cd /etc/nginx/conf.d/
~ # vim nextcloud.conf
```

Refer to [Nginx Configuration](https://docs.nextcloud.com/server/12/admin_manual/installation/nginx.html):

1. Check *server_name*, *root*, *ssl_certificate* and *ssl_certificate_key* etc. directives.
2. If the host posesses IPv6 address, add directives `listen [::]:80;` and `listen [::]:443 ssl http2;`.

```
# /etc/nginx/conf.d/nextcloud.conf

upstream php-handler {
    server 127.0.0.1:9000;
    #server unix:/var/run/php5-fpm.sock;
}

server {
    listen 80;
    listen [::]:80;
    server_name cloud.example.com;
    # enforce https
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name cloud.example.com;

    ssl_certificate "/etc/letsencrypt/live/cloud.example.com/fullchain.pem";
    ssl_certificate_key "/etc/letsencrypt/live/cloud.example.com/privkey.pem";

    # Add headers to serve security related headers
    # Before enabling Strict-Transport-Security headers please read into this
    # topic first.
    # add_header Strict-Transport-Security "max-age=15768000;
    # includeSubDomains; preload;";
    #
    # WARNING: Only add the preload option once you read about
    # the consequences in https://hstspreload.org/. This option
    # will add the domain to a hardcoded list that is shipped
    # in all major browsers and getting removed from this list
    # could take several months.
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header X-Download-Options noopen;
    add_header X-Permitted-Cross-Domain-Policies none;

    # Path to the root of your installation
    root /usr/share/nginx/html/nextcloud/;

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    # The following 2 rules are only needed for the user_webfinger app.
    # Uncomment it if you're planning to use this app.
    #rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
    #rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json
    # last;

    location = /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }
    location = /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }

    # set max upload size
    client_max_body_size 512M;
    fastcgi_buffers 64 4K;

    # Enable gzip but do not remove ETag headers
    gzip on;
    gzip_vary on;
    gzip_comp_level 4;
    gzip_min_length 256;
    gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
    gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

    # Uncomment if your server is build with the ngx_pagespeed module
    # This module is currently not supported.
    #pagespeed off;

    location / {
        rewrite ^ /index.php$uri;
    }

    location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)/ {
        deny all;
    }
    location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console) {
        deny all;
    }

    location ~ ^/(?:index|remote|public|cron|core/ajax/update|status|ocs/v[12]|updater/.+|ocs-provider/.+)\.php(?:$|/) {
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param HTTPS on;
        #Avoid sending the security headers twice
        fastcgi_param modHeadersAvailable true;
        fastcgi_param front_controller_active true;
        fastcgi_pass php-handler;
        fastcgi_intercept_errors on;
        fastcgi_request_buffering off;
    }

    location ~ ^/(?:updater|ocs-provider)(?:$|/) {
        try_files $uri/ =404;
        index index.php;
    }

    # Adding the cache control header for js and css files
    # Make sure it is BELOW the PHP block
    location ~ \.(?:css|js|woff|svg|gif)$ {
        try_files $uri /index.php$uri$is_args$args;
        add_header Cache-Control "public, max-age=15778463";
        # Add headers to serve security related headers (It is intended to
        # have those duplicated to the ones above)
        # Before enabling Strict-Transport-Security headers please read into
        # this topic first.
        # add_header Strict-Transport-Security "max-age=15768000;
        #  includeSubDomains; preload;";
        #
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Robots-Tag none;
        add_header X-Download-Options noopen;
        add_header X-Permitted-Cross-Domain-Policies none;
        # Optional: Don't log access to assets
        access_log off;
    }

    location ~ \.(?:png|html|ttf|ico|jpg|jpeg)$ {
        try_files $uri /index.php$uri$is_args$args;
        # Optional: Don't log access to other assets
        access_log off;
    }
}
```

## [Let's Encrypt Certificate](/2017/10/10/le-certificate)

We should has SSL certificate to domain *cloud.example.com*.

```bash
~ # certbot certonly --cert-name cloud.example.com --webroot -w /usr/share/nginx/html/ -d cloud.example.com --dry-run
```

## Launch Nextcloud

After uploading the configuration to */etc/nginx/conf.d/nextcloud.conf*, check syntax and reload *nginx*:

```bash
~ # nginx -t
~ # systemctl reload nginx
# or
~ # nginx -s reload
```

# Enhancements

>Nextcloud supports loading configuration parameters from multiple files. You can add arbitrary files ending with *\*.config.php* in the *config/* directory, for example you could place your email server configuration in *email.config.php*.

During configuration, some variables deserve special attention: Php *memory_limit*, APCu *shm* size, etc.

1. Reset admin password.

   Nextcloud installation wizard does not ask for password confirmation. If you cannot remember the correct password, please [reset it](https://docs.nextcloud.com/server/12/admin_manual/configuration_user/reset_user_password.html).

   ```bash
   ~ # su -s /bin/bash -c "php /usr/share/nginx/html/nextcloud/occ user:resetpassword admin-bob" nginx
   ```

2. [HTTP Strict Transport Security - HSTS](https://docs.nextcloud.com/server/12/admin_manual/configuration_server/harden_server.html#use-https)

   ```
   # /etc/nginx/conf.d/nextcloud.conf
   # https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/
   add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;
   ```

3. OPcache

   On the admin panel, you might recived message:

   >The PHP Opcache is not properly configured. For better performance we recommend ↗ to use following settings in the php.ini

   First make sure Php *opcache* module is installed by:

   ```bash
   ~ # php -m | grep -i opcache
   ~ # yum install php70w-opcache
   ```

   Then update */etc/php.d/opcache.ini* as instructed:

   ```
   # /etc/php.d/opcache.ini
   
   opcache.enable=1
   opcache.enable_cli=1
   opcache.interned_strings_buffer=8
   opcache.max_accelerated_files=10000
   opcache.memory_consumption=128
   opcache.save_comments=1
   opcache.revalidate_freq=1
   ```

   Updates to Php *\*.ini* configuration file require reloading *php-fpm* service.
4. Php memory limit

   `memory_limit` in default */etc/php.ini* is 128M. However per-site value of Nextcloud in */opt/nextcloud/.user.ini* is 512M that is the whole available amount. To prevent Php from exhausting too much memory, set to 128M instead.

   ```
   # /usr/share/nginx/html/nextcloud/.user.ini
   upload_max_filesize=100M
   post_max_size=100M
   memory_limit=128M
   ```

   1. Per-site Php setting overrides that of global under */etc*.
   2. It does not require *php-fpm* reloading.
   3. Make sure *upload_max_filesize* and *post_max_size* value is smaller than *memory_limit*.
5. Local user (APCu) and Distributed cache (Redis)

   Php APCu module is installed previously, then enable it in *config.php*:
   
   ```
   # /usr/share/nginx/html/nextcloud/config/personal.config.php
   'memcache.local' => '\OC\Memcache\APCu',
   ```

   1. Updates to *\*.php* requires NO service reloading (just refreshing web page) except installing new Php modules.
   2. As I use nextcloud for private useage, *distributed* cache (Memcache/Redis) is not a must.
6. Redis as file locking backend

   Though Redis as distributed cache is not a must, however Redis cache can also act as Transactional File Locking backend. File locking is enabled by default, using the database locking backend. This places a significant load on your database.

   Using memcache.locking relieves the database load and improves performance. 

   ```bash
   ~ # yum install redis php70w-pecl-redis
   ~ # systemctl status redis
   ~ # systemctl enable redis
   ~ # systemctl start redis
   ```

   Add the following to *config.php*:

   ```
   # /usr/share/nginx/html/nextcloud/config/personal.config.php

   'filelocking.enabled' => true,
   'memcache.locking' => '\OC\Memcache\Redis',
   'redis' => [
     'host' => 'localhost', // can also be a unix domain socket: '/tmp/redis.sock'
     'port' => 6379,
     'timeout' => 0.0,
     'password' => 'dunno', // Optional, if not defined no password will be used.
     'dbindex' => 0, // Optional, if undefined SELECT will not run and will use Redis Server's default DB Index.
   ],
   ```

   For enhanced security, please set a password for Redis cache. Reload *php-fpm* and refresh web page.
7. Disable thumbnailer.

   By default, thumbnailer is enabled for images, mp3, etc., which however impacts overhead on system resources. Without previews, Nextcloud runs fairly smoothly.

   ```
   # /usr/share/nginx/html/nextcloud/config/personal.config.php

   'enable_previews' => false,
   ```

   The [side effect](https://github.com/nextcloud/server/issues/6000) of disabling previews is Gallery app no longer show any media files:

   >No media files found

   Why not disable Gallery as well?
8. Email notification.

   >TLS (STARTTLS) with 587, SSL (SSL/TLS) with 465.

   ```
   # /usr/share/nginx/html/nextcloud/config/personal.config.php

   "mail_smtpmode" => "smtp",
   "mail_smtpsecure" => 'tls',
   "mail_smtpport" => 587,
   "mail_smtphost" => "smtp-mail.outlook.com",
   "mail_domain" => "outlook.com",
   "mail_from_address" => "valid-alias",
   "mail_smtpauth" => true,
   "mail_smtpauthtype" => "LOGIN",
   "mail_smtpname" => "email@outlook.com",
   "mail_smtppassword" => "email-password",
   "mail_smtptimeout" => 30,
   ```

   1. Outlook only accept valid `mail_from_address` owned by `mail_smtpname`. Fake `mail_from_address` or `mail_domain` cannot authenticate to the Outlook SMTP server.
   2. If a malware or SPAM scanner is running on the SMTP server it might be necessary that you increase the `mail_smtptimeout` value.
9. Many administration work can be done through tool [*occ* command](https://docs.nextcloud.com/server/12/admin_manual/configuration_server/occ_command.html) - ownCloud Console.

## [4-byte UTF-8 support](https://docs.nextcloud.com/server/12/admin_manual/configuration_database/mysql_4byte_support.html)

>Be careful on [No FILE_FORMAT](https://github.com/nextcloud/documentation/issues/513). This is just a trial.

**backup** database before going on!

1. maintenance mode

   ```bash
   ~ # su -s /bin/bash -c "php ./occ maintenance:mode --on" nginx
   ```

2. InnoDB

   ```
   /etc/my.cnf.d/4-byte.cnf
   [client]
   default-character-set = utf8mb4

   [mysql]
   default-character-set = utf8mb4

   [mysqld]
   innodb_large_prefix = 1
   innodb_file_format = barracuda
   innodb_file_per_table = 1
   character-set-client-handshake = 0
   character-set-server = utf8mb4
   collation-server = utf8mb4_unicode_ci
   ```

3. Restart Mariadb

   ```bash
   ~ # systemctl restart mariadb
   ```

4. 4-byte UTF-8

   ```
   ~ # mysql -u nc_db -pPassword
   ```

   [Modify database](https://stackoverflow.com/a/9531221):

   ```
   ALTER DATABASE nc_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   use nc_db;
   # For each table in nc_db:
   ALTER TABLE <table_name> CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   # For each column in <table_name>:
   ALTER TABLE <table_name> MODIFY <column_name> VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_unicode_ci;
   ```

5. *mysql.utf8mb4* in *config.php*

   ```
   su -s /bin/bash -c "php ./occ config:system:set mysql.utf8mb4 --type boolean --value=true" nginx
   su -s /bin/bash -c "php ./occ maintenance:mode --off" nginx
   su -s /bin/bash -c "php ./occ maintenance:repair" nginx
   su -s /bin/bash -c "php ./occ maintenance:mode --on" nginx
   ```

   Attention: turn off *maintenace* mode before *repair* since it requires apps be loaded.

## Data folder

The previous setup use a separate data directory */opt/nextcloud-data* for files. If you'd like to switch to another location (i.e. */usr/local/data*), use `php ./occ files:scan --all`.

Firstly, move data folder to the new location:

```bash
~ # mv /opt/nextcloud-data /usr/local/data
~ # chown -R nginx:nginx /usr/local/data
```

Afterwards, tell Nextcloud where the new location is:

```
# vi /usr/share/nginx/html/nextcloud/config/personal.config.php
'datadirectory' => '/usr/local/data',
```

Finally, refresh Nextcloud by *occ*:

```bash
~ # su -s /bin/bash -c "php ./occ files:scan --all" nginx
```

**Attention**: with the help of `occ files:scan --all`, we can modify (add, remove files) data directory directly without the interfence of Nextcloud web or clients. For example, you have a big video file on server, and have to fetch the video file to local PC and then synchronize through Nextcloud client. Alternatively, just copy the video file to */opt/nextcloud-data/username/files/Documents* and run the *occ* command above.

# DAV

1. App Contacts 1.5.3 by default does not create default address-book upon installation. Before creating or importing any contacts, an address-book must be created at first.
2. [Importing address book from file works only partially because import breaks process limit in shared hosting environment](https://github.com/nextcloud/contacts/issues/235).

   >Contact could not be created.

   Further checking web server log, I found this:

   >10 worker_connections are not enough while connecting to upstream, client: 192.168.0.100, server: cloud.example.com, request: "PUT /remote.php/dav/addressbooks/users/jim/test-50/12345678-f873-4f3b-861d-6a6b5b78e5a5.vcf HTTP/1.1", upstream: "fastcgi://127.0.0.1:9999", host: "cloud.example.com"

   1. If you have a big contacts *vcard* file to be imported, cut it into smaller blocks (i.e. every 100 contacts) and import one by one.
   2. Increase web server's allowed concurrent connections, take Nginx for example:

      ```
      # /etc/nginx/nginx.conf

      events {
	  worker_connections 1024;
      }
      ```

      Set back to original value once imported.
3. When adding Davdroid account, select:

   >Contact group method: groups are per-contact categories 

4. [Davdroid does not sync at all](https://forums.bitfire.at/topic/1508/zte-nubia-requires-autostart). For synchronization, must turn on [*autostart*](https://davdroid.bitfire.at/faq/entry/miui-no-synchronization/) for Davdroid in system setting.
5. If Davdroid address book or calendar does not show up stock Contacts/Calendar apps, try *True Contacts* and *Etar* instead.
6. Calendar app may remind incompatible *ics* format, then import without specifying an existing calendar and use *New Calendar* instead.
7. Add or subscribe to [中国农历](https://github.com/infinet/lunar-calendar).

   Currently (as of Nextcloud 12), subscribed *iCal* web link [would not](https://github.com/nextcloud/server/issues/1497) be synced to mobile (i.e. Davdroid).

   1. Use [ICSdroid](https://davdroid.bitfire.at/faq/entry/subscribe-ics-file/) to subscribe the original link or to Nextcloud's subscription link.
   2. As [中国农历](https://github.com/infinet/lunar-calendar) writes, use the tool to generate lunar for incoming years, which won't consume too much disk space. Afterwards, import that as a new calendar instead of a subscription.
   3. Wait for Nextcloud 13.

   >Obviously, the 2nd method is favorable.

# Notes

Evernote has now gone too far, leaving its users behind. I want to just abandon it. But by far, neither [Nextcloud official Notes app](https://apps.nextcloud.com/apps/notes) (I use this one) nor [unofficial Nextnotes](https://github.com/janis91/nextnotes) support importing Evernote notes.

However, we have [QOwnNotes](https://github.com/pbek/QOwnNotes) as provisional tool to transmitting Evernotes to Nextloud.

1. Install QOwnNotes and Nextcloud on your system.
2. Install and enable [QOwnNotesAPI](https://github.com/pbek/qownnotesapi) (optional)
3. Add Nextcloud account to QOwnNotes client. (optional)
4. IMPORTANT! Set QOwnNotes *note folder path* to be that of Nextcloud.
5. [Export Evernote](http://www.qownnotes.org/Blog/Evernote-import) notebook to ENEX format *.enex*.
6. Import the *.enex* in QOwnNotes.
7. Sync the notes in Nextcloud client.

>The 2nd and 3rd steps are unneccessary since we won't use QOwnNotes after the transmission. Pay attention to the [role of QOwnNotesAPI](http://www.qownnotes.org/Knowledge-base/Why-isn-t-QOwnNotesAPI-syncing-my-notes). Normal note files are handled by Nextloud while note transhes and versions (on the server side) are handled by QOwnNotesAPI separately.

## Bug

[A bug of Notes app in web interface](https://github.com/nextcloud/notes/issues/133)

By default, the first line of a note will be the filename. Maybe, the Notes app in web interface cannot create the right file name with full shape punctuators or Chinese characters.

The current solution is to:

1. Disable Notes app;
2. Backup and remove the Files - Notes folder;

   Specifically remove the *troublesome* new note file.
3. Re-enable Notes app;
4. Apply [No. 125](https://github.com/nextcloud/notes/pull/125) patch.
5. Move back the Notes folder
6. Do a `occ files:scan --all` and reload Notes app.

To further fix broken activity app, we should remove the troublesome activity records:

```
mysql -u nc_db -pPassword
select activity_id,type,app,subject,subjectparams,file,link from nc_db.oc_activity order by activity_id DESC limit 10;
delete from nextcloud.oc_activity where activity_id>2300;
```

Adjust the *activity_id* range.

# News

After enabling News app, you receives warnings like:

>Ajax or Web cron mode detected! Your feeds will not updated!

By default, [Nexcloud Ajax](https://docs.nextcloud.com/server/12/admin_manual/configuration_server/background_jobs_configuration.html) is used. For news to be updated automatically, we should use system cron service. If you don't care about automatic news fetch, just ignore that.

Firstly, install *cronie* service:

```bash
~ # yum install cronie
~ # systemctl enable crond
~ # systemctl start crond
~ # cat /etc/crontab
```

Then, we add cron job for user *nginx*:

```bash
~ # crontab -u nginx -e
*/15  *  *  *  * php -f /var/www/nextcloud/cron.php
~ # crontab -u nginx -l
~ # cat /var/spool/cron/nginx
```

Finally, select the option Cron in the admin menu for background jobs. If left on AJAX it would execute the AJAX job on every page load. 

As the reference pointed out, we can use *systemd timer* as well. It's all your choice.

## Tips

1. The Android app requires *autostart* and *storage* permissions to sync feeds.

# [Backup](https://docs.nextcloud.com/server/12/admin_manual/maintenance/backup.html)

>Do it before upgrading.

Mainly, we should backup *folders* and *database*.

## folders

1. The config folder, i.e. *config.php*, *personal.config.php* etc.
2. The data folder. The most important part.
3. The theme folder.
3. During upgrading, the Nextcloud installation folder is backed up automatically. If you trust it, that's fine.

```bash
~ # rsync -aAx nextcloud/ nextcloud-dirbkp_`date +"%Y%m%d"`/ 
~ # rsync -aAx nextcloud-data/ nextcloud-data-dirbkp_`date +"%Y%m%d"`/ 
```

Optionally, we can synchroize data to PC just in case.

## database

```
# backup
mysqldump --single-transaction -h localhost -u nc_user -pPassword nc_db > nextcloud-sqlbkp_`date +"%Y%m%d"`.bak
# restore
drop database nc_db
create database nc_db
# or
create database nc_db default character set utf8mb4 collate utf8mb4_unicode_ci";
~ # mysql -h localhost -u nc_user -pPassword nc_db < nextcloud-sqlbkp.bak
```

There is no space between `-p` and Password.

# [Update](https://docs.nextcloud.com/server/12/admin_manual/maintenance/update.html)

1. Firstly, *updater* checks environment.

   *updater* enables *maintenance* mode automatically.
2. Secondly, *upgrade* does the real update.
3. Both tools could be executed through the web interface and/or command line.

   Web-based *updater* would ask you if *Keep maintenance mode active*. Command line *upgrade* and web-based *upgrade* require *active* (enabled) and *inactive* (disabled) maintenance mode respectively.

Here, I use command line for the whole update process.

```
~ # cd /usr/share/nginx/html/nextcloud/updater
~ # su -s /bin/bash -c 'php updater.phar' nginx
# if errors like "The following extra files have been found: .user.ini.bak", remove those files. Re-run:
~ # su -s /bin/bash -c 'php updater.phar' nginx
# Should the "occ upgrade" command be executed? [Y/n] y
# Keep maintenance mode active? [y/N] y
~ # cd ..
~ # su -s /bin/bash -c 'php occ upgrade ' nginx
~ # su -s /bin/bash -c 'php occ status ' nginx
```

Dunno why *occ upgrade* is not ran automatically even I typed 'y'. Maybe should I type 'Y'?

## miantenance mode

*maintenance:mode* locks the sessions of logged-in users and prevents new logins. This is the mode to use for upgrades. Check the mode by:

```bash
# "maintenance" value in 'config.php' is updated accordingly
~ # su -s /bin/bash -c "php ./occ list" nginx
~ # su -s /bin/bash -c "php ./occ maintenance:mode" nginx
~ # su -s /bin/bash -c "php ./occ maintenance:mode --on/off " nginx
```

# Domain switch

Suppose you would moving from *cloud.old.com* to *cloud.new.com*, then remember to modify *trusted_domains* and *overwrite.cli.url* in *config/config.php*.

```
   'trusted_domains' =>
   array (
     0 => '192.168.0.29',
     1 => 'cloud.new.com',
   ),
   'overwrite.cli.url' => 'https://cloud.new.com',
```

This can be done by *occ* as well:

```bash
~ # su -s /bin/bash -c "php ./occ config:system:get trusted_domains" nginx
~ # su -s /bin/bash -c "php ./occ config:system:set trusted_domains 1 --value=cloud.new.com" nginx
~ # su -s /bin/bash -c "php ./occ config:system:get overwrite.cli.url" nginx
~ # su -s /bin/bash -c "php ./occ config:system:set overwrite.cli.url --value=https://cloud.new.com" nginx
```

# Refs

1. [How to Install Nextcloud with Nginx and PHP7-FPM on CentOS 7](https://www.howtoforge.com/tutorial/how-to-install-nextcloud-with-nginx-and-php-fpm-on-centos-7/)
2. [在CentOS7上使用Nginx和PHP7-FPM安装Nextcloud](https://www.orgleaf.com/2504.html)
3. [How to install Nextcloud 12 server on CentOS 7](http://www.marksei.com/install-nextcloud-12-centos-7/)
4. [启用 SELinux 的情况下，在 CentOS 上搭建 NextCloud/ownCloud 服务](https://nifume.com/build_nextcloud_service_on_centos_with_selinux.html)
5. [Install Nginx, PHP 7, MariaDB 10 (LEMP) on CentOS 7](https://github.com/terrylinooo/daily/wiki/Install-Nginx,-PHP-7,-MariaDB-10-(LEMP)-on-CentOS-7)
