---
layout: post
title: Weechat SASL NickServ
---

> SASL is a method that simplify identification to services (NickServ) as the first step after connecting to IRC servers, before anything else happens. To connect to freenode using Tor, SASL is required. In this post, I only focus on SASL for NicServ.

# 1 Basic setup

After starting `$ weechat`:

1. /set irc.server.freenode.nicks "jimgray,jimgray\_,\_jimgray,\_jimgray_"

    If you don't set default nicks, Weechat will take Gentoo username as the default nick. Check the `server_default` section of `less .weechat/irc.conf`.
2. /set irc.server.freenode.username "Jim_Gray"
3. /set irc.server.freenode.realname "Jim Gray"
4. /set irc.server.freenode.autoconnect on

    Autoconnect to *freenode* on startup.
5. /set irc.server.freenode.addresses "chat.freenode.net/7000"
6. /set irc.server.freenode.ssl on

    Connect to *freenode* using SSL.
6. /set irc.server.freenode.command "/mode jimgray +w"

    *Optinal*: add *wallops* to receive operator announcements.
7. /set irc.network.send\_unknown_commands on

    Some commands (like */ns*) are specific to IRC servers (like *freenode*) which cannot be recoginized by IRC client (like *weechat*). This setting enables such commands.
7. /set irc.look.part\_closes_buffer on

    This will close the *buffer* (or called *tab*) immediately when parting a channel.
8. /set irc.server.freenode.autojoin "\#gentoo"

    This is *NOT* recommended.

    1. You are not always connecting to "\#gentoo" channel. Only connect on demand!
    2. We will setup SASL identification to NickServ which requires authentication to *freenode* server before joining. If *autojoin* is turned on for a channel, Weechat might join that channel before authentication finished.
9. Up to now, all the settings are just for *local* IRC client *weechat* only! The IRC server *freenode* knows nothing.

    If this is the first time of Weechat: try `/connect freenode`.
10. Read [quick start](https://weechat.org/files/doc/devel/weechat_quickstart.en.html).

# 2 Register NickServ

*NickServ* is to register user information *remotely* on IRC servers (i.e. *freenode*). Settings in this part are stored remotely on *freenode* server. So those settings should not be repeated when changing IRC clients or PC.

Make sure you are connected to *freenode* (`/connect freenode`) with your desired nick (primary nick *jimgray* in this post) and now you are in the *server* tab.

**Don't execute the following command in a channel tab**, otherwise sensitive information (i.e. password) might be disposed and sniffed.

1. /msg NickServ REGISTER password youremail@example.com

    In Weechat, the *pasword youremail@example.com* part will be displayed as asterisks. But don't worry as long as you put a space between *password* and *youremail@example.com*.

    After this command, Weechat will show *password* and *youremail@example.com* as plain text. In this command, you don't offer *account name* since *freenode* uses the current nick (*jimgray* in this post) as the account name. So *jimgray* is both the nick and account name registered on *freenode*. This special nick is called *primary nick*.

    In a while, you will receive an email, from which you should copy the verification command to complete the registration process:
2. /msg NickServ VERIFY REGISTER jimgray xxxxxxxx

    This temporary code is only for account verification.

    Now, account name *jimgray* has just one nick *jimgray* associated on *freenode* server, no one else can use nick *jimgray* unless the correct *password* is identified (talked later on).
3. /msg NickServ SET HIDEMAIL ON

    To keep your email address private, rather than displaying it publicly, mark it as hidden (which is done by default for new accounts).
4. Group more nicks to *jimgray* account.

    A *freenode* account can register/reserve/hold several nicks.

    First switch to that nick and then *group* it to account(as long as that nick is not grouped by others). Grouping nicks in this way gives you the benefit of having all your nicks covered by the same *cloak* (*unaffiliated cloak* below), should you choose to wear a cloak.

    ```
    /nick jimgray_ # switch to nick 'jimgray_'
    
    /ns group # group it to account 
    or
    /msg NickServ GROUP
    ```
    Repeat the two commands for other nicks you desire. Use `/nick  [-all] <nick>` to switch a specific nick. When joining a channel, the default nick to choose is based on IRC client *irc.server.freenode.nicks* order above.
5. `/reconnect freenode` or restart *weechat*.

    Each time on startup or reconnect with an already registered nick (i.e. *jimgray*), it will prompt like *jimgray is registered... you must identify yourself...*. To identify yourself as nick *jimgray*, run `/msg NickServ identify jimgray password`. More see *3 Identification*.
6. Read [freenode nick setup](https://freenode.net/faq.shtml#nicksetup).

# 3 Identification

## Without SASL

If you don't configure SASL, then you have two choices:

1. Manually identification after connection to server.

    ```
    /connect freenode
    or
    /reconnect freenode
    
    /msg NickServ identify jimgray password
    ```
2. freenode.command

    ```
    /set irc.server.freenode.command "/msg NickServ identify jimgray password"
    ```
    To run a command **after connection** to server automatically, for example to authenticate with NickServ.

The above two methods is slow and stupid in that you must input password for each identification. Now let's come to SASL.

## SASL

You are recommended to use SASL. SASL is nearly the same as *manually identification* above. Both use the same password and offer local infomation. The big difference is SASL read password from a file (*sec.conf* of *weechat*).

Settings below are all locally stored.

1. /set irc.server.freenode.sasl_mechanism plain

    WeeChat supports SASL authentication, using different mechanisms of which *plain* is the simplest - just offer the password. More, read [weechat sasl](https://www.weechat.org/files/doc/stable/weechat_user.en.html#irc_sasl_authentication).
1. /set irc.server.freenode.sasl_username "jimgray"

    The username can be any one of your grouped nicks. But usually the primary nick (namely the account name) is used.
2. /secure set freenode password

    *password* **must be the same** as the one used to register NickServ.
3. /set irc.server.freenode.sasl_password "${sec.data.freenode}"

    Tell *weechat* where to get the password.
4. /set irc.server.freenode.sasl_fail disconnect

    If SASL auth fail, disconnect.
5. /reconnect freenode

    You (as *jimgray*) are automatcially identified by SASL once connected to *freenode* server.
6. Read [secured data](https://www.weechat.org/files/doc/stable/weechat_user.en.html#secured_data) and [freenode sasl](https://freenode.net/sasl/).

# 4 unaffilicated cloak

1. *Unaffiliated cloak* is to hide your IP from IRC command like `whois`.
2. To obtain a cloak, just join *freenode* channel, ask for one.

    When channel staff notices your request, you *might* get cloaked. Unaffiliated cloak applies to all channels on *freenode* server.
