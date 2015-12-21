---
layout: post
title: Weechat SASL NickServ
---

> SASL is a method that allows identification to services (NickServ) as the first step in connecting to the network, before anything else happens. To use SASL, one must register with services. To connect to freenode using Tor, SASL is required. In this post, I only focus on SASL for NicServ.

# 1 Basic setup

After starting `$ weechat`:

1. /set irc.server.freenode.nicks "jimgray,jimgray\_,\_jimgray,\_jimgray_"

    If you don't set default nicks, Weechat will take Gentoo username as the default nickname. Check the `server_default` section of `less .weechat/irc.conf`.
2. /set irc.server.freenode.username "Jim Gray"
3. /set irc.server.freenode.realname "Jim Gray"
4. /set irc.server.freenode.autoconnect on

    Autoconnect to *freenode* on startup.
5. /set irc.server.freenode.addresses "chat.freenode.net/7000"
6. /set irc.server.freenode.ssl on

    Connect to *freenode* using SSL.
6. /set irc.server.freenode.command "/mode jimgray +w"

    Add *wallops* to receive important operator announcements.
7. /set irc.network.send\_unknown_commands on

    Some commands (like */ns*) are specific to IRC servers (like *freenode*) which cannot be recoginized by IRC client (like *weechat*). This setting enables such commands.
7. /set irc.look.part_closes_buffer on

    This will close the *buffer* (or called *tab*) immediately when parting a channel.
8. /set irc.server.freenode.autojoin "\#gentoo"

    This is NOT recommended.

    1. You are not always connecting to "\#gentoo" channel. Only connect on demand!
    2. We will setup SASL identification to NickServ which requires authentication to *freenode* server before joining. If *autojoin* is turned on for a channel, Weechat might join that channel before authentication finished.
9. Up to now, all the settings are just for IRC client *weechat* only! The IRC server *freenode* knows nothing about them.

    If this is the first time of Weechat:
    1. /connect freenode
    2. /join #gentoo
    3. /part #gentoo
    4. /exit
10. Read [quick start](https://weechat.org/files/doc/devel/weechat_quickstart.en.html).

# 2 Register NickServ

*most settings below are known to/for freenode server*.

Make sure you are connected to *freenode* with your desired nickname (primary nickname *jimgray* in this post) and now you are in the *server* tab.

**Don't execute the following command in a channel tab**, otherwise sensitive information (i.e. password) might be sniffed by guys there.

1. /msg NickServ REGISTER password youremail@example.com

    In Weechat, the *pasword youremail@example.com* part will be displayed as asterisks. But don't worry as long as you put a space between *password* and *youremail@example.com*.

    After this command, Weechat will show *password* and *youremail@example.com* as plain text.

    In a while, you will receive an email, from which you should copy the verification command to complete the registration process:
2. /msg NickServ VERIFY REGISTER jimgray xxxxxxxx

    This temporary code is only for verification. The *freenode* account name will the very nickname used when registering (*jimgray* here).
3. /msg NickServ SET HIDEMAIL ON

    To keep your email address private, rather than displaying it publicly, mark it as hidden (which is done by default for new accounts).
4. Group more nicknames to *jimgray* account.

    Now, account and nick name *jimgray* is registered on *freenode* server, no one else can use it unless the correct *password* is supplied identify himself.

    An *freenode* account can register/reserve/hold several nicknames. First switch to that nickname and then group it to account as long as that nickname is not owned by others. Grouping nicks in this way gives you the benefit of having all your nicks covered by the same cloak, should you choose to wear a cloak (see *unaffiliated cloak* below).

    ```
    /nick jimgray_
    
    /ns group
    or
    /msg NickServ GROUP
    ```
    Repeat the two commands for other nicknames you desire. We can choose anyone of them when joining channels. If alreay on a channel, use `/nick  [-all] <nickname>` to switch. When joining a channel, the nicknames order to choose is based on IRC client *irc.server.freenode.nicks* setting above.
5. /reconnect freenode

    Reconnect to *freenode*, you will see a message like *jimgray is registered... you must identify yourself...*.
6. Read [freenode nickname setup](https://freenode.net/faq.shtml#nicksetup).

# 3 Identification

## Without SASL

If you don't configure SASL, then you have two choices:

1. Manually identification after connection to server.

    ```
    /connect freenode
    /msg NickServ identify jimgray password
    ```
2. Automation

    ```
    /set irc.server.freenode.command "/msg NickServ identify jimgray password"
    ```
    To run a command **after connection** to server automatically, for example to authenticate with nickserv (only if you don’t use SASL for authentication).

## SASL

You are recommended to use SASL.

1. /set irc.server.freenode.sasl_username "jimgray"

    This step can be ignored. If set, it should be identical to your registered nickname *jimgray*.
2. /secure set freenode password

    *password* **must be the same** as the one to register NickServ.
3. /set irc.server.freenode.sasl_password "${sec.data.freenode}"
4. /set irc.server.freenode.sasl_fail disconnect
5. /reconnect freenode

    You (as *jimgray*) are automatcially identified by SASL once connected to *freenode* server.
5. Read [secured data](https://www.weechat.org/files/doc/stable/weechat_user.en.html#secured_data) and [freenode sasl](https://freenode.net/sasl/).

# 4 unaffilicated cloak

1. *Unaffiliated cloak* is to hide your IP from IRC command like `whois`.
2. To obtain a cloak, just join *freenode* channel, ask for one.

    When channel staff notices your request, you *might* get cloaked. Unaffiliated cloak applies to all channels on *freenode* server.
