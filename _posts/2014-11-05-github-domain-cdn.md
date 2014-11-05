---
layout: post
title: Github Page Domain CDN Acceleration
---

<div class="message">
     Github page supports CDN acceleration service. How to do it?
</div>

1. *Default domain*  
 The github default domain is `username.github.io` which is also a *subdomain*. But usually we need to use our own domain for blog. Personal domain may be top domain or subdomain. Subdomain is like www.example.com while top domain is like example.com. Github names `A`,`Alias`, and `ANAME` all as `apex` domain (record). Here `apex` domain acctually is not a domain but a domain dns record.  
2. *Acceleration*  
The key is to set a `CNAME` record for your github page `username.github.io`. For example if you have a top domain `example.com`, A `CNAME` record is like:

<table>
 <thead>
  <tr>
   <th>Name</th>
   <th>Type</th>
   <th>Target</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>www</td>
   <td>CNAME</td>
   <td>username.github.io</td>
  </tr>
 </tbody>
</table>
This means subdomain address `www.example.com` will be redirected to `username.github.io`.  
3. *CNAME* and *Top domain*  
`CNAME` does not support top domain. The following three lines are all illegal though some DNS provider does not give you alert.

<table>
 <thead>
  <tr>
   <th>Name</th>
   <th>Type</th>
   <th>Target</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>example.com</td>
   <td>CNAME</td>
   <td>username.github.io</td>
  </tr>
  <tr>
   <td>@</td>
   <td>CNAME</td>
   <td>username.github.io</td>
  </tr>
  <tr>
   <td></td>
   <td>CNAME</td>
   <td>username.github.io</td>
  </tr>
 </tbody>
</table>
If you set `CNAME` like this, it will direct all your other subdomains like mail, ftp etc to your github page. Refer to [custom subdomain](https://help.github.com/articles/tips-for-configuring-a-cname-record-with-your-dns-provider/).
>Do not use wildcard DNS records (e.g. *.example.com) with GitHub Pages! A wildcard DNS record will allow anyone to host a GitHub Pages site at one of your subdomains.

Can I use my top domain for acceleration? Yes, but not easy! First, your DNS provider must support `Alias DNS Record` which is not a standard DNS record. Only a few DNS provider support this with fees, like **dnssimple**. Refer to [What is an ALIAS record?](http://support.dnsimple.com/articles/alias-record/) and [Pointing the Domain Apex to Heroku](http://support.dnsimple.com/articles/domain-apex-heroku/).  
4. *Procedures*  

* Add a `CNAME` file at the root directory of github pages. You may find it uncessary at first. If you don't add this file, only the first time you enter the custom domain url works as you wish. If you further click some other links in the blog (i.e. a post), you will find the url changes to original *username.github.io*. This looks ugly. What was worse, The webbrower tab will not show the blog title.  
* In the `CNAME` file, you can put only one line namely *www.example.com* there. Now Github handles everything in your jekyll blog with address *www.example.com* not the original *username.github.io*. Attention! Put bare address. Don't add `http://`.  
* Next need DNS support. I choose **freenom** to register my top domain *example.tk*. Why top domain? Not subdomain? Don't worry! *Freenom* supports top domain, why not get one? Yes, get the top domain. Then we can set a *CNAME* DNS record! Refer to [How to setup A, MX, CNAME...](https://my.freenom.com/knowledgebase.php?action=displayarticle&id=4).  
* The `CNAME` record is to transfer `WWW` subdomain of *example.tk* (namely *www.example.tk*) to *username.github.io*. Now wait for a while, it will be fine. Use `dig www.example.tk` to test your result. Refer to [use the dig command](https://help.github.com/articles/tips-for-configuring-a-cname-record-with-your-dns-provider/#configuring-a-custom-subdomain-with-your-dns-provider).
* But if you want *example.tk* also points to *username.github.io*, what should be done? As mentioned above we cannot use `CNAME`. What is worse, **freenom** does not support `Alias` DNS record like `dnssimple` does. However we can set `A` record for the top domain alghtough it does not support CDN. Please refer to [configuring an A record on Github](https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider/).

<table>
 <thead>
  <tr>
   <th>Name</th>
   <th>Type</th>
   <th>Target</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td></td>
   <td>A</td>
   <td>192.30.252.153</td>
  </tr>
  <tr>
   <td></td>
   <td>A</td>
   <td>192.30.252.154</td>
  </tr>
 </tbody>
</table>
* Now you can access your blog from both subdomain and top domain. Although top domain does not support CDN individually, but if both subdomain CNAME and top domain A records are configured, github itsefl will create redirect between the two based on the **CNAME file** in the github page root directory. Refer to <a href="https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider/#configuring-a-www-subdomain" target="_blank">Configuring a www subdomain</a>. Since the `CNAME` file in github pages are `www.example.com`, then when you input `example.com` in your browser, github will redirects you to `www.example.com`. Although 

<div class="message">
Till now, both subdomain and top domain support CDN!
</div>

*Reference*

1. [Setting up a custom domain with GitHub Pages](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/).
2. [Github Pages 静态网页建站](http://m.blog.csdn.net/blog/chuchus/38964175).
3. [Do I set a DNS A Record for the new GitHub Pages to use their CDN?](http://webmasters.stackexchange.com/questions/56826/do-i-set-a-dns-a-record-for-the-new-github-pages-to-use-their-cdn).
4. [Faster, More Awesome GitHub Pages](https://github.com/blog/1715-faster-more-awesome-github-pages).
