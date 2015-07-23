---
layout: post
title: Linux Distributions
---
1. Linux kernel is the core of system management. We still need distributions to provide users a slightly high level interface. A Linux distribution is basically a wrapper of Linux kernel, supporting GUI, package management, etc.

    You never notice Windows kernel thing since Windows is a complete system.
2. Common Linux distributions differ in `package management`, `update method`, `stable` or not, `security`, `free` or not etc. Refer to [如何选择发行版][1]
    1. Debian: `.deb` package; stable.

	    >Ubuntu, Mint.
	2. Fedora: `.rpm` package.
	
	    >RHEL (commercial use), CentOS (Community ENTerprise OS).
	
	    ![Relationship]({{site.baseurl}}assets/fedora.png "distrio diff")

        CentOS derives from reverse engineering of RHEL, which takes away part of RHEL customers.
    3. OpenSUSE: `.rpm` built **specialy** for openSUSE.
	
	
	4. Arch Linux(KISS, binary package), Gentoo (source compiled locally)
	
	    `Rolling release` instead of normal `fix release`. Need time and patience.
		
		Arch Linux Wiki is the best in the world!
	5. BackTrack, Kali
	
	    Based-on Debian now, focusing on security issue.
3. Package management, refer to [What is the difference between dpkg and aptitude/apt-get?][3] and [Comparison of major Linux package management systems][4]
    1. Debian: .deb -> dpkg -> apt-get -> aptitude
	2. Fedora: .rpm -> rpm -> yum
	3. openSUSE: special .rpm -> rpm -> zypper
	4. Arch: AUR binary -> pacman
	5. Gentoo: portage source -> emerge (equery, eix)
	
	They came up with a way to *store* the files of an application in a *package* so that it can be easily installed. So the `.deb` package was born.
	
	They needed a tool to install these `.deb` files, so they came up with the `dpkg` tool. This tool, however, will just install the `.deb` file, but will not install its dependencies because it doesn't have those files and it does not have access to `repositories` to go pull the dependencies from.
	
	Then, they came up with `apt-get`, which automates the problems in the previous point. Underneath the hood, `apt-get` is basically `dpkg` (so `apt-get` is a front-end for `dpkg`), but a *clever* one that will look for the dependencies and install them. It even looks at the currently installed dependencies and determines ones that are not being used by any other packages, and will inform you that you can remove them.
	
	`aptitude` then came along, and it's nothing but a front-end UI for `apt-get`. More difference refer to [what is difference between apt-get and aptitude][2].
4. Rumours

    CentOS让RedHat招安了
	
	---
	Scientific Linux不还有吗。从来都不觉得CentOS有啥好的，package太老，连经常被嘲笑的Debian stable都看不下去了。稍微正常点用Ubuntu，文艺点用Arch Linux，CentOS没有了没什么大不了的
	
	---
	scientific linux你用过吗？我以前怀着希望试了一下，感觉用户群小的linux bug多啊。
	
	---
	CentOS更忠于RHEL原版, Scientific Linux自己改的东西多一些
	
	---
	这下估计 CentOS 要没有什么作为了。
	CentOS 的价值在于，提供一个开源并且可以直接预装的RHEL clone. RHEL 是很多企业的选择，因为他们想要超级稳定性，然后想要技术支持，例如，有了 zero day 的bug
	要赶紧补上。成熟的企业除非有特别强的Linux 背景，不是很感用随便的 distro。

	RHEL 一直都在玩一个游戏就是，开源对吧。RHEL 提供src rpm。 但是 RHEL 不提供可以直接预装的 binary,只对商业用户提供。这个并没有违反开源的游戏规则。而且那个 RHEL 里面含有一些程序让别人不能直接复制那个 RHEL 的安装盘放到其他地区下载。同样，升级也需要是付费用户（有免费试用）。

	CentOS 就是提供这样的服务，把 RHEL 的 src rpm，从新编译最后组装出 RHEL 兼容的但是大家可以随便使用的预装光盘。大家觉得不就是编译一下 src rpm 么，简单阿。其实不是那末简单。Redhat 的 src rpm 要编译出和 RHEL一样效果的 binary 需要用到 Redhat 的不开放的编译环境，还有很怪异的编译启动参数。这些参数是从编译环境直接传过来的。也就是说，CentOS提供的服务是逆向工程了RHEL的编译环境和捣鼓出安装光盘。相当与挖了 RH 的商业壁垒，挡人财路。

	被招安之后，RH 当然不会干雇人来挖自己想保护的商业壁垒的蠢事。我猜结果就是，CentOS 估计要被阉割了。肯定不会完全消失，但是要用 CentOS 不爽。但是没有不爽到要有人舍得另起炉灶的地步。

	类似的例子有 ntfs 3G. NTFS 3G 搞了个用户层的文件系统，就是为了保护他们商业版本的高性能 内核 NTFS 文件系统。这个方式很有效果，大多数人都比较慢的 ntfs 也行，所以搞个纯内核的NTFS 意义就小了很多。目前就没有人对着干。嵌入系统使用 NTFS，那个内核版本的可能还是比较合理的选择。
	
	---
	比如说哈，NSF 和 NIH 强调 open source，所以很多全国的大型计算中心在用 CentOS。这下子可就有大变动了。新的计算中心可能就要选 debian 了。
	
	---
	这是好事，centos的包都老掉牙了我觉得一般小地方的管理员就是为了所谓稳定，来个保守的选择centos，痛苦死了我很多东西都是自己local install的

	---
	RHEL/CentOS系只适合大企业内部养老，稍微正常点的公司都不会用。默认系统是稳定，但是很多包要么很老要么根本就没有需要自己找第三方源或者裸装，这样只要有非官方包进
	来，从根本上就破坏了它所谓的稳定，还不如用稍微正常点然后稳定和后续维护也还不错的Ubuntu LTS。激进和文艺一点的直接上Arch也没见出什么事

	---
	大公司用这个有，我个人观察比较大一个因素是推卸责任。人家不在乎没出什么事，那个是理所当然的。在乎的是出了事谁来背黑锅，可以推卸责任。所以比较像样的支持服务还
	是需要的。作为推卸责任的手段。“我们已经联系XXX公司在深入研究这个崩溃问题了，人家已经派最好的工程师在找原因...”。技术支持是推卸黑锅的很好办法，我已经用了技术
	好的公司了，那责任不在我这里，是那些公司不行。你要是自己用冷门的distro，万一了问题就要自己扛，不是开源没，自己动手改啊。自立更生。改不好啊，当初谁负责谁背黑
	锅。有技术实力的是不怕用比较新的系统的。 kernel.org 就是用fedora。人家流量也很大，一样跑得好好的。关键是里面有牛人，有问题自己能消化。

	---	
	唉唉，在法律责任切分严格的行业，RHEL 还是有压倒性的优势的。比如说医疗系统。我们这里搭个 cluster，我说上 debian，IT 
	直接就问：出了问题算谁的？谁来承担责任并且走人？再比如说，我们这里上了一套数据库系统，因为先期投入大，造成去年预算问题，信用等级被降级，CIO 立刻走人。

	---	
	嗯嗯，是啊是啊。我们这里 IT 说他们有雇员有RH的认证，所以有资质使用RHEL。买大些的东西，首先看的就是出了问题后，如何确定责任。
	每个人都只会支持那些让自己的职业比较安全的东西，这是底线。好不好则另说了。

	---	
	企业端不用RHEL/CentOS的只怕不多。

	---
	For those that are saying Red Hat is the main project, yes it is.. But the diagram shows how the flow actually goes. Red Hat are based on stable packages of Fedora, not that Red Hat is Fedora's derivatives. CentOS however, is the stable release of Red Hat for the community. What we don't see here is that Red Hat uses both Fedora and CentOS for its experiments and stability testings to improve Red Hat, which is genius. This contributes for the corporate highly stable release of Linux, which is one of the best OS around.

	---	
	Fedora is the starting point. Those packages eventually make it into RHEL once they are proven to be stable. Then the RHEL Source code is used to create CentOS without the RHEL branding.

	---	
	From a corporate point of view Red Hat is the main project that Red Hat throws the amount of resources. and the one that brings the revenue. However, as others have said, on a technical level, and as the diagram depicts. Fedora flows into Red Hat and Red Hat flows into CentOS
	
[1]:http://program-think.blogspot.com/2013/10/linux-distributions-guide.html
[2]:http://unix.stackexchange.com/q/767
[3]:http://askubuntu.com/a/309121
[4]:http://how-to.linuxcareer.com/comparison-of-major-linux-package-management-systems