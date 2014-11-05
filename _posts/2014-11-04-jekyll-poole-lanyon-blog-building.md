---
layout: post
title: Jekyll Poole Lanyon Blog Building
---

<div class="message">
 This is to record the process of build the blog process.
</div>

>The most important reference is [How I Created a Beautiful and Minimal Blog Using Jekyll, Github Pages, and poole](http://joshualande.com/jekyll-github-pages-poole/), [Lanyon](https://github.com/poole/lanyon) and [Poole](https://github.com/poole) on Github.

## Procedure

**1** Create user pages  
Create a empty repository called `username.github.com` on Github without `README` file.  
**2** Ubuntu Command lines  
Copy *Lanyon* locally.  
`$ git clone https://github.com/poole/lanyon.git username.github.com`  
Enter the new cloned repository.  
`$ cd username.github.com`  
Set remote repository to sync later on.  
`$ git remote set-url origin https://github.com/username/username.github.com.git`  
Push locall repository to remote.  
`$ git push origin master`  
**3** Visit `username.github.io`  
What you see it not yours, all the contents are from *Lanyon*. We need to modify the contents for our own use. We should firstly validate the modification before push to remote repository. `Jekyll` can do that.  
**4** `Jekyll`  
For installation, refer to [Install Jekyll 2 on Ubuntu 14.04](http://michaelchelen.net/81fa/install-jekyll-2-ubuntu-14-04/) and [Official site](http://jekyllrb.com/docs/installation/). Jekyll comes with a built-in development server that will allow you to preview what the generated site will look like in your browser locally by `http://localhost:4000`. You can try now:

* Run `$ jekyll server` in `username.github.com` directory
* Open browser enter `localhost:4000`.

There you go!  

## Modification
The first is to modify the `_config.yml` configuration file such as contact info, blog title, etc.
The **url** varialble in *_config.yml*, I found it not useful. Remove it.

For **baseurl** setting, refer to [4. Serving it up](https://github.com/poole/poole#usage) section.  

   1. If you're using a custom domain name, leave it as `/`. Then modify the **CNAME** file to point to new domain `www.fangxiang.tk`.
   2. If you're not using a custom domain name, modify the `baseurl` in *_config.yml* to point to your GitHub Pages URL. Example: for a repo at github.com/username/poole, use http://username.github.io/poole/. Be sure to include the trailing slash.

**Sidebar** "Download" and "Github Project" items does not work. Because *custom variable* `site.github.repo` is not defined in *_config.yml*. The *site.github.repo* is used in *_include/sidebar.html*. Just add two lines in *_config.yml* as follows. Pay attention to the indent of the second line.

{% highlight js %}
github:
  repo:	https://github.com/outsinre/outsinre.github.com
{% endhighlight %}

Though the problem was solved, I don't need the two sidebar items. So I removed them from _config.yml. Correspondingly, the lines in *_include/sidebar.html* should also be removed.

**Disqus** comments. Firsly, create a file *comments.html* and add two lines:
{% raw %}
*{% if page.comments %}*

<em>{% endif %}</em>
{% endraw %}

Then paste the DISQUS universal code between the two lines. The *comments.html* should be placed under *_include* folder. After that, in *_layout/post.html* file, add a line *&#123;% include comments.html %}* to the end. Also in the header part add `comments: true`.

**Archive** page added based on the reference at the beginning.

**Subscribe** page added by modifying `_include/sidebar.html`.

## Setup

Some fun facts about the setup of Poole project include:

* Built for [Jekyll](http://jekyllrb.com)
* Developed on GitHub and hosted for free on [GitHub Pages](https://pages.github.com)
* Coded with [Sublime Text 2](http://sublimetext.com), an amazing code editor
* Designed and developed while listening to music like [Blood Bros Trilogy](https://soundcloud.com/maddecent/sets/blood-bros-series)

Have questions or suggestions? [Contact Poole author on Twitter](https://twitter.com/mdo).

Thanks for reading!
