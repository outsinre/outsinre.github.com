---
layout: post
title: Github project page blog
---
1. *blog.example.com* for Github *project page*, mainly hosting Chinese posts.
2. *www.example.com* and *example.com* for Github *user page*, mainly hosting technical posts.

    Top domain *example.com* can ONLY be used as an alternative to subdomain *www*.

# Project on Github

Create a repository *projectname* on Github.

# Project page

Create a branch named exactly as *gh-pages* for repository *projectname*.

# Default branch

In Github repository setting, set *gh-pages* as the default branch instead of the usual *master* one.

Thus every action like pull and push will take *gh-pages* as the default branch.

# Clone

```bash
$ cd ~/workspace
$ git clone https://github.com/username/projectname.git
$ cd projectname
$ cat .git/config
$ git branch
```
# Template

Choose a *Jekyll* template. Here I just use the one for *www.example.com*. Copy the necessary files to *~/workspace/projectname/*.

# Customization

## CNAME *file*

Set the content to be:

>blog.example.com

Do NOT add the prefix *http://*.

## Freenom DNS

We should add a DNS *CNAME record* for *blog.example.com* on DNS control panel.

>BLOG CNAME username.github.io

## Notes

1. If without customization, the *user page* URL would be *username.github.io* while the *project page* URL would be *username.github.io/projectname*.
2. You can see that in the DNS control panel, we have two *CNAME records* to the same destination:

    >BLOG CNAME username.github.io
    >
    >WWW CNAME username.github.io
3. Whether it is *user page* or *project page*, the DNS *CNAME record* points to the same destination *username.github.io*. You should NOT or can NOT set DNS *CNAME record* to *username.github.io/projectname*.

    It Github pages server's job to determine the real (user or project) page location.
4. You can also find two DNS *A record*:

    >␢␢␢␢ A 192.30.252.153
    >
    >␢␢␢␢ A 192.30.252.154
    
    The four `␢` means leaving the first cell blank.
5. The two DNS *A records* are making *example.com* the alternative URL of *www.example.com*.

    Top domain *example.com* can ONLY be the alternative of *www* subdomain.

## *baseurl* and *url*

In *Jekyll* template, we usually need to use variable like *site.url*, *site.baseurl*, *post.title* etc. For a permanent URL:

>http://username.github.io/projectname/2015/09/23/gentoo/

*http://blog.example.com* (without trailing slash) is the *url* variable, while */projectname* (without trailing slash) is the *baseurl* variable.

1. For *url* in *_config.yml*, just set it to *http://blog.example.com*.
2. For *baseurl*,
    1. If use *custom url*, then just set it to `/`;
    2. If not, set it to `/projectname` (without trailing trash). When doing test locally on *localhost:4000*, you should run *jekyll serve --baseurl ''* which sets *baseurl* to empty.
3. Settings in *_config.yml* are *key*: *value* pairs.
    1. Add least a whitespace after colon (:).
    2. The *value* can (or not) be enclosed with quotes depending on your preference.
