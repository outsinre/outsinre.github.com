---
layout: post
title: symbolic links to config in central location
---
There are all kinds of config files and scripts to take care of during installation or system backup. It is smart practice to put them in a central place, while creating corresponding symbolic links to them.

1. mkdir /gtmisc

    gtmisc is the central place to store those files.
2. mv /etc/fstab /gtmisc
3. ln -sv ../gtmisc/fstab /etc/fstab

    /gitmisc will store files related to root account. Repeat step 2 & 3 for other files.

    For normal user files, you can operate like this in user home directory.
4. List of files

    ```
    /etc/fstab
    /etc/portage/{make.conf,repos.conf/,}
    /usr/local/bin/*
    /usr/local/sbin/*
    /usr/local/portage (local portage, pay attention to portage:portage ownership)
    /etc/wpa_supplicant/wpa_supplicant.conf (pay attention the file attributes)
    etc.
    ```
    gtmisc can be put anywhere you desire. But pay attention to fstab which should be accessible at the early stage of boot. For example, if gtmisc{/fstab,} were put under /home/me/gtmisc, then mount points except / in fstab would not be mounted since the mount of /home depends on /home/me/gtmisc/fstab, which forms a cyclic dependency.
5. Relative VS absolute symbolic link

        ln -sv /path/to/source /path/to/target

    1. 'target' is the symbolic link to created, pointing to 'source' (normal files or directories).
    2. 'ln' command can be executed under nay 'pwd' regardless of 'source' or 'target' directory.
    3. relative link: 'target' will look for 'source' through relative path to its own directory.
    4. absolute link: 'target' will look for 'source through absolute path.
    5. If you move absolute link around, it still points to the correct target, while relative link would be dead.
    3.  Whether 'target' is a relative or absolute link depends on /path/to/source.
        1. If it a relative path (i.e. ln -sv ../gitmisc/fstab /etc/fstab), then target location is relative to '/etc/'. If you move '/etc/fstab' link to '/home/me/fstab', the relative path will be '/home/gitmisc/fstab' which does not exist.
        2. If it a absolute path (ie.e ln -sv /gitmisc/fstab /etc/fstab), then target location is at absolute path '/gitmisc/fstab'.
5. Notes
    1. pvcreate, luksFormat both works on /dev/sda disk?
    2. symbolic link relative absolute
    3. mtab link dead
    4. /home/zachary/gtmisc
    5. dd boot
    6. dd if=/dev/sdb1 | xz > boot-image-backup.xz
    7. xzcat image-file.xz | dd of=/dev/sdb1
    8. sdb1 fat32 on windows, mounted automatically
