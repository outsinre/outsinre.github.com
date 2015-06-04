---
layout: post
title: Filesystem Hierarchy Standard
---
[The Filesystem Hierarchy Standard (FHS)][1] defines the directory structure and directory contents in Unix and Unix-like operating systems, maintained by the Linux Foundation. The current version is 3.0, released on 3 June 2015.

This command will give you a brief description on FHS:
>_$_ man hier

We focus on some directories relevant to binaries.

<table>
  <thead>
    <tr>
      <th>Directory</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>/</td>
      <td><strong>Primary hierarchy</strong> root and root directory of the entire file system hierarchy.</td>
    </tr>
    <tr>
      <td>/bin</td>
      <td>This directory contains executable programs which are needed in single user mode and to bring the system up or repair it. Essential command binaries that need to be available in single user mode; for all users, e.g., cat, ls, cp. For binaries usable before the /usr partition is mounted. This is used for trivial binaries used in the very early boot stage or ones that you need to have available in booting single-user mode.</td>
    </tr>
    <tr>
      <td>/sbin</td>
      <td>Like /bin, this directory holds commands needed to boot the system, but which are usually not executed by normal users. Essential system binaries, e.g., init, ip, mount.</td>
    </tr>
    <tr>
      <td>/usr</td>
      <td>This directory is usually mounted from a separate partition. It should hold only sharable, read-only data, so that it can be mounted by various machines running Linux. <strong>Secondary hierarchy</strong> for read-only user data; contains the majority of (multi-)user utilities and applications.</td>
    </tr>
    <tr>
      <td>/usr/bin</td>
      <td>This is the primary directory for executable programs. Most programs executed by normal users which are not needed for booting or for repairing the system and which are not installed locally should be placed in this directory. Non-essential command binaries (not needed in single user mode); for all users.</td>
    </tr>
    <tr>
      <td>/usr/sbin</td>
      <td>This directory contains program binaries for system administration which are not essential for the boot process, for mounting /usr, or for system repair. Non-essential system binaries, e.g., daemons for various network-services.</td>
    </tr>
    <tr>
      <td>/usr/local</td>
      <td>This is where programs which are local to the site typically go. <strong>Tertiary hierarchy</strong> for local data, specific to this host. Typically has further subdirectories, e.g., bin/, lib/, share.</td>
    </tr>
    <tr>
      <td>/usr/local/bin</td>
      <td>Binaries for programs local to the site. For example, a simple script to run <em>dropbox</em>.</td>
    </tr>
    <tr>
      <td>/usr/local/sbin</td>
      <td>Locally installed programs for system administration. For example, a simple script to run <em>sslocal</em> which needs root privilege.</td>
    </tr>
    <tr>
      <td>/opt[/bin]</td>
      <td>This  directory  should  contain  add-on  packages  that contain static files. Reserved for the installation of add-on application software packages. A package to be installed in /opt must locate its static files in a separate /opt/package or /opt/provider directory tree, where <em>package</em> is a name that describes the software package and <em>provider</em> is the provider's LANANA registered name.</td>
    </tr>
    <tr>
      <td>~/bin</td>
      <td>This is for user-owned private binaries. Can be appended to your $PATH.</td>
  </tbody>
</table>

1. You will see that files and directories that are admin only are gathered in the same directory: the `s` in /sbin and /usr/sbin and /usr/local/sbin stands for system. A normal user can not even start programs that are in there. Files a normal user can start are in /bin, /usr/bin, /usr/local/bin based on where it most logically should reside. But if they are admin only they should go to the `s` version of that directory. There is a famous utility called fuser. You can kill processes with it. If a normal user could use this (s)he would be able to kill your session.
2. The same goes for /home: /home/user1 is property of user1. /home/user2 is property of user2. user2 has no business doing stuff in user1's home (and the other way around is also true: user1 has no business doing stuff in user2's home). If all the files would be in /home with no username underneath it you would have to give permissions to every file and asses if someone is allowed to write/remove those files. A nightmare if you have tens of users.
3. Addition regarding libraries.

    /lib/, /usr/lib/, and /usr/local/lib/ are the original locations, from before multilib systems existed and the exist to prevent breaking things. /usr/lib32, /usr/lib/64, /usr/local/lib32/, /usr/local/lib64/ are 32-/64-bit multilib inventions.
4. If I'm writing my own scripts, where should I add these?

    None of the above. Please use /usr/local/bin or /usr/local/sbin for system-wide (mulitple users) available scripts. The local path means it's not managed by the system packages (this is an error for Debian/Ubuntu packages). For private-owned scripts, use ~/bin in home directory.

    Always keep **multi-user** nature of Linux in mind.

Refer to [2] and [3].

[1]:http://www.linuxfoundation.org/collaborate/workgroups/lsb/fhs-30
[2]:http://askubuntu.com/a/308048
[3]:http://askubuntu.com/a/138551
