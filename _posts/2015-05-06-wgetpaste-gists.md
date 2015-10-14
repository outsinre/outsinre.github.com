---
layout: post
title: wgetpaste default to gists
---

1. *wgetpaste* is tool to conveniently paste your text and code snippet online around. However, it defaults to *bpaste* which relatively weak compared to *gist*. In this post, I will configure *wgetpaste* to use *gist* instead.
2. $ wgetpaste -h

    This command will show the basic command options.
3. $ wgetpaste -S

    This will list free *service* supported by *wgetpaste*.

    ```
    Services supported: (case sensitive):
       Name:        | Url:
       =============|=================
       *bpaste      | https://bpaste.net/
        ca          | http://pastebin.ca/
        codepad     | http://codepad.org/
        dpaste      | http://dpaste.com/
        lugons      | https://paste.lugons.org/
        poundpython | http://paste.pound-python.org/
        gists       | https://api.github.com/gists
    ```
    The default service is *bpaste* with a *\** ahead.
4. Now we need to change it to *gists*.

    ```bash
    # ect /etc/wgetpaste.d/gists.conf
    DEFAULT_SERVICE="gists"
    ```
    *wgetpaste* supports many other options, `DEFAULT_{NICK,LANGUAGE,EXPIRATION}[_${SERVICE}]` included.

    Run *wgetpaste -S* again to verify configuration.
5. $ wgetpaste ~/Documents/test-file

    Up to now, *wgetpaste* will paste text and code snippet as **anonymous** [gist.github.com](https://gist.github.com).

    *anonymous gist* cannot be searched later on.
6. *wgetpaste* to GitHub *account*.
    1. Go to GitHub *settings* and find *Personal access tokens*.
    2. Choose *Generate new token* and set the token description to *wgetpaste*.
    3. Change token right to *gist* only.
    4. Edit */etc/wgetpaste.d/gists.conf*:

    ```
    # /etc/wgetpaste.d/gists.conf
    HEADER_gists="Authorization: token 1234abc56789..."
    ```
    Test again *wgetpaste ~/Documents/test-file*.
