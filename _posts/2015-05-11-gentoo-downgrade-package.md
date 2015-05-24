---
layout: post
title: Downgrade Package && wpa_supplicant && local overlay
---

> Once for a while after an update, you system might bump into problem due to a new package version or to an unstable package. Thus the trouble-making packages should be downgraded to its old working version.

Recently, `wpa_supplicant` has upgraded from 2.3 to 2.4, resulting in unable to connecting to `HUST_WIRELESS_AUTO`. After days of surfing on the Internet, I found the solution: downgrade `wpa_supplicant` to 2.3. But before that, let's see how to position the problem. That is to say how do I get to know it is due to `wpa_supplicant` update causing no Wifi connection.

At the very beginning, I am in a mess on the Internet without any useful information since this problem is brand new. Even Google cannot search any clues. Af very first, I thought it was due to patches to kernel. So I reinstall the system with 4.0.0 instead of 4.0.0-r3, which turned out be in vain.

I searched on the Internet again for a day with nothing gain. I also read through the Gentoo [wpa_supplicant wiki](https://wiki.gentoo.org/wiki/Wpa_supplicant) since this package is the key of no connection to Wifi. Then I used `dmesg | tail -n 35` or the recommended debug mode `wpa_supplicant -Dnl80211 -iwlp3s0 -C/var/run/wpa_supplicant/ -c/etc/wpa_supplicant/wpa_supplicant.conf -dd`. Found an error message `reason 15: 4 way handshake timeout`, which is not useful at all when searching online.

Finally, on some Internet page, I was directed to an __important link__: [wpa_supplicant Linux documentation page](https://wireless.wiki.kernel.org/en/users/documentation/wpa_supplicant). Great! One step closer to solution.

From that link, you can find many useful information: _development branch ChangeLog_, _stable branch ChangeLog_, and [__New mailing list archives__](http://lists.shmoo.com/pipermail/hostap/) where the newest package related issues usually lies. Mailing list reports immediate package discussion even before Bug report system.

Following the mailing archives, I found [Unable to connect to WPA2-Enterprise since 2.4-r1: WPA\_ALG_PMK bug?](http://lists.shmoo.com/pipermail/hosfftap/2015-April/032685.html) which unfold the mystery on Wifi problem. Everything is there! Again, I searched the subject get another link [Unable to connect to WPA2-Enterprise since 2.4-r1: WPA\_ALG_PMK bug?](https://www.marc.info/?t=143013943600001&r=1&w=4) which is fairly concise: wpa_supplicant >= 2.4 encountering problems with WPA2-Enterprise networks.

Through the discussion there, I found several solutions:

1. Downgrade wpa_supplicant to 2.3 (<2.4) with the help of local overlay.
2. Ask the university to update its FreeRadius server from to 2.2.6 to 2.2.7.
3. Disable TLS v1.2 with phase1="tls\_disable\_tlsv1\_2=1" in `/etc/wpa_supplicant/wpa_supplicant.conf` for `HUST_WIRELESS_AUTO`. That said, this does result in older TLS version being used and that is not really a good long term
solution.
4. Set `key_mgmt_offload=0` to the wpa_supplicant configuration file (at global level, i.e., not within a network block).

Obviously, the 3rd method simple and effective. In reality, I prefer the 3rd method.  But I would like to try the 1st instead since I want to try the process of creating a local overlay. I did not test the 4th option.

1. Search wpa_supplicant version information:

    >_#_ emerge --search wpa_supplicant
    But that returns only trouble versions >=2.4. That is to say, the old working version <2.4 is no longer in the official portage tree.
2. Mask versions that does not work:

    >_#_ echo ">=net-wireless/wpa\_supplicant-2.4" > /etc/portage/package.mask/wpa_supplicant
3. From above two steps, we know there are no working versions in official portage tree. All the trouble versions in official portage are masked. How to handle this?

    > Local overlay! The basic idea is to create local overlay and then download the old version `.ebuild` and corresponding supporting files.
4. Create local overlay, refer to [Overlay/Local overlay](https://wiki.gentoo.org/wiki/Overlay/Local_overlay):
    1. _#_ mkdir -p /usr/local/portage/{metadata,profiles}
    2. _#_ echo 'zhtux' > /usr/local/portage/profiles/repo_name
    3. _#_ echo 'masters = gentoo' > /usr/local/portage/metadata/layout.conf
    4. _#_ chown -R portage:portage /usr/local/portage
    5. _#_ emacs -nw /etc/portage/repos.conf/local.conf:

        >[zhtux]
	
        >location = /usr/local/portage
	
        >masters = gentoo
	
        >auto-sync = no
5. Adding an ebuild to the local overlay. Now we need to locate wpa_supplicant 2.3, 2.3-r1, or 2.3-r2 ebuild file and download it
    1. Go through [/net-wireless/wpa_supplicant](https://sources.gentoo.org/cgi-bin/viewvc.cgi/gentoo-x86/net-wireless/wpa_supplicant/)
    2. You cannot find `wpa_supplicant-2.3*.ebuild` since version 2.3 is out of portage tree: dead.
    3. Click on `Show 78 dead files` from which you will find old ebuild files: wpa_supplicant-2.3.ebuild, wpa_supplicant-2.3-r1.ebuild, and wpa_supplicant-2.3-r2.ebuild. Here I choose `wpa_supplicant-2.3-r2.ebuild`. Download this file from `Links to HEAD: (view) (download) (annotate)`. Then click on `Hide 78 dead files`.
    4. _#_ mkdir -p /usr/local/portage/net-wireless/wpa\_supplicant
    4. _#_ cd /usr/local/portage/net-wireless/wpa_supplicant
    5. _#_ cp /path/to/downloaded/wpa\_supplicant-2.3-r2.ebuild /usr/local/portage/net-wireless/wpa_supplicant/
        1. Some points out that we also need copy the `metadata.xml` file. This file does not change along different versions. If emerge complains lacking this file, copy it from step 1 link or from `/usr/portage/net-wireless/wpa_supplicant/`.
    6. _#_ repoman manifest (pay attention to: the current directory)
        1. Generate the `Manifest` file each time you change files in this overlay.
    7. _#_ chown -R portage:portage /usr/local/portage
        1. This is important each time you create or copied new files to local overlay to make those files belong to portage:portage.
    8. _#_ emerge -av1 wpa_supplicant
        1. This will not install wpa\_supplicant 2.3-r2 since it complains lacking files. If you go back to step 1, you will find a folder `/net-wireless/wpa_supplicant/files`. Items under this directory is supporting files like patches. emerge won't get this file for you automatically since you are installing from local overlay, you need to get those support files by yourself.
        2. On the other hand, emerge will download the source files for wpa_supplicant-2.3-r2. It is specified in the ebuild file:

            >_#_ grep "SRC\_URI=" /usr/local/portage/net-wireless/wpa\_supplicant/wpa_supplicant-2.3-r2.ebuild

            But the downloaded source file is kept under `/usr/portage/distfiles/` where offical portage `gentoo` and layman overlay `gentoo-zh` stores source files.
        3. The 3rd emerge optiono is number one, not character L. This means for one time installation.
    9. _#_ mkdir files
        1. Now based the emerge error message, we go to step 1, and download the missing files.
        2. We can also read through the ebuild files to locate what supporting files needed.
        3. The top of the page in step 1 reminds that we can use `cvs` command to locate the files.
        4. [IMPORTANT] Repeat steps starting from 6.

---
Wpa_supplicant debug:

1. Run wpa_supplicant in debug mode:

    > _#_ killall -TERM wpa_supplicant

    > _#_ /etc/init.d/dhcpcd stop

    > _#_ wpa\_supplicant -Dnl80211 -iwlp3s0 -C/var/run/wpa\_supplicant/ -c/etc/wpa\_supplicant/wpa_supplicant.conf -dd

    For this step, you can also run `wpa_cli` command interactively.
2. Retart wpa_supplicant if need reconnection:

    > _#_ killall -TERM wpa_supplicant

    > _#_ /etc/init.d/dhcpcd restart
3. Enable log: refer to wiki.

---
Reference:

1. https://wireless.wiki.kernel.org/en/users/documentation/wpa_supplicant
2. https://www.marc.info/?t=143013943600001&r=1&w=4
3. http://lists.shmoo.com/pipermail/hostap/2015-April/032685.html
4. https://www.marc.info/?l=hostap&m=143077239911058&w=4

    >phase1="tls\_disable\_tlsv1_2=1"
5. http://blog.csdn.net/zhuyingqingfen/article/details/7830624
6. https://sources.gentoo.org/cgi-bin/viewvc.cgi/gentoo-x86/net-wireless/wpa_supplicant/
7. https://wiki.gentoo.org/wiki/Overlay/Local_overlay