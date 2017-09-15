---
layout: post
comments: true
title: 顶级域名不能设置DNS CNAME记录
---

要用CDN加速，那么custom domain (无论是top还是sub的）必须指向username.github.io，不能直接指向username.github.io的ip地址。这样，由于是指向域名，而不是具体ip地址，CDN动态的根据访问者的位置来返回一个username.github.io的ip地址，这个ip地址和访问者的ip地址很近。如果是是直接指向一个固定的ip地址，那就只能从那个地址访问custom domain了。

那么如何指向username.github.io呢？如果是subdomain，那好办，直接建立一条DNS CNAME记录，例如www CNAME username.github.io。如果是top domain，就稍微复杂些。Top domain是原理上不能设置CNAME记录，所以@ CNAME username.github.io这样的dns CNAME是不合法的。除非DNS provider 支持 CNAME flattening、ALIAS、ANAME，不过这些不是标准DNS协议，以来DNS provider服务。

这里解释一下为何top domain不能有CNAME记录。因为"A CNAME record is not allowed to coexist with any other data."也就是说如果为一个域名建立了一条CNAME记录，那么这个域名就不能有其他的dns记录了。一个域名一旦有CANME记录，说明它只能用作域名转换用，不能再有其他的作用。A CNAME-record cannot exist with any other records for the same name, because a name cannot both be an alias (CNAME) and something else at the same time.这句话里的name就是我指的域名，或者说是dns的记录中左边的那个值。如果你建立一条example.com CNAME username.github.io，那么它会和所有其它的以example.com开头的DNS记录冲突，而事实是这样的记录必定存在！这样的记录有SOA，NS记录，这些记录平常我们是看不到的。当你在域名提供商那里注册域名时，提供商会默默的为你建立SOA NS记录，这些记录就是以example.com开头的DNS记录，具体例子参考[What is a NS Record?](http://support.dnsimple.com/articles/ns-record/)和[What is a SOA Record?](http://support.dnsimple.com/articles/soa-record/)。SOA 和 NS记录，是必须存在的，必不可少，否则域名都无法注册。好了，既然SOA NS这样的记录必须存在，那么以example.com开头的CNAME记录就不能建立了。记住，不能建立CNAME记录，但可以建立A记录，RFC并没有说A记录不能和其他的记录共存。

现在来回顾一下为何以www.example.com开头的CNAME记录可以存在？因为域名提供商不会默默为你建立其它的以www.example.com开头的域名。如果你自己同时建立两条以www.example.com开头的记录，并且其中一条是CNAME记录，那同样也不行！！但这种情况很少。为何会出现要求建立顶级郁闷开头的CNAME记录呢？因为这些需求以个人为主，个人一般来说就是一个博客网站，不会有其他的如mail ftp等需求，都希望顶级域名访问自己的博客。

总之一旦有一个以label (any name)开头的CNAME记录，就不能再有其它的以label开头的任何类型的DNS记录了！此时label只能唯一用作域名转换。CNAME的作用就是把左边的label（别名）换成右边的真正的域名。

另外，前面讨论中多次提到**顶级域名**。**顶级域名**是一个相对概念，譬如example.com相对www.example.com是顶级域名，但是example.com本身也有顶级域名com。所以前面的讨论中**顶级域名**是针对你手中已经有点最高级的域名来说的，譬如我在freenom注册了域名example.tk，那么对我来说，我现有的最高级的域名就是它了。其他的譬如www.example.tk, mail.example.tk, ftp.example.tk都是我的sub domain。假如你的最高级域名是三级的blog.example.com，那么它就是你的顶级域名，photo.blog.example.com就是sub domain，这个情况同样适用于上面的CNAME问题讨论。

参考资料：

1. [Why can't I create a CNAME record for the zone name itself?](http://support.simpledns.com/kb/a94/why-cant-i-create-a-cname-record-for-the-zone-name-itself.aspx)
2. [How to overcome root domain CNAME restrictions?](http://stackoverflow.com/questions/656009/how-to-overcome-root-domain-cname-restrictions)
3. [Is Root domain CNAME to other domain allowed by DNS RFC?](http://stackoverflow.com/questions/655235/is-root-domain-cname-to-other-domain-allowed-by-dns-rfc)
4. [Why can't a CNAME record be used at the apex of a domain?](http://serverfault.com/questions/613829/why-cant-a-cname-record-be-used-at-the-apex-of-a-domain/613830#613830)
