---
layout: post
title: Jekyll Poole Lanyon Blog Building
---

<div class="message">
 This is to record the process of build the blog process.
</div>

>The most important reference is [How I Created a Beautiful and Minimal Blog Using Jekyll, Github Pages, and poole](http://joshualande.com/jekyll-github-pages-poole/), [Lanyon](https://github.com/poole/lanyon) and [Poole](https://github.com/poole) on Github.

## Procedure

**1** Install git command  
`$ sudo apt-get install git`  
`$ git config --global ...` For configuration commands refer to official website.
**2** Create user pages  
Create a empty repository called `username.github.com` on Github web without `README` file.  
**3** Ubuntu Command lines  
Copy *Lanyon* locally.  
`$ git clone https://github.com/poole/lanyon.git username.github.com`  
Enter the new cloned repository.  
`$ cd username.github.com`  
Set remote repository to sync later on.  
`$ git remote set-url origin https://github.com/username/username.github.com.git`  
Push locall repository to remote.  
`$ git push origin master`  
**4** Visit `username.github.io`  
What you see it not yours, all the contents are from *Lanyon*. We need to modify the contents for our own use. We should firstly validate the modification before push to remote repository. `Jekyll` can do that validation.  
**5** `Jekyll`  
For installation, refer to [Install Jekyll 2 on Ubuntu 14.04](http://michaelchelen.net/81fa/install-jekyll-2-ubuntu-14-04/) and [Official site](http://jekyllrb.com/docs/installation/).

* `sudo apt-get install ruby ruby-dev make`
* `sudo gem install jekyll --no-rdoc --no-ri`
* `sudo apt-get install nodejs` {% raw %} <span style="color:gray">There is a current issue that causes Jekyll to require a JavaScript runtime even if it will not be used. Installing nodejs helps solve the issue</span> {% endraw %}
* `jekyll -v`

Jekyll comes with a built-in development server that will allow you to preview what the generated site will look like in your browser locally by `http://localhost:4000`. You can try now:

* Run `$ jekyll server` in `username.github.com` directory
* Open browser enter `localhost:4000`.

There you go! You can see a local copy of the repository. What you need to do now is to modify the contents locally for personal use.  

## 6 Modification
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

**Site icon** changed referring to [icons](http://modernweb.com/2013/10/28/building-a-blog-with-jekyll/). Check the file `_include/head.html` for where is icon defined.

**read more...*** to only display the first paragraph of a post. Only need to modify the `index.html` file. The original is to display the post content completely:
{% raw %}
    {{ post.content }}
{% endraw %}
but now replaced by:
{% raw %}
    {{ post.excerpt | remove: '<p>' | remove: '</p>' }}

    {% if post.content.size > 500 %}
      <br /><br /><a href="{{ post.url }}">Read more...</a>
    {% endif %}
{% endraw %}

**code line numbers** refer to <a href="http://demisx.github.io/jekyll/2014/01/13/improve-code-highlighting-in-jekyll.html" target="_blank">Improve Code Highlighting in a Jekyll-based Blog Site</a>. Especially pay due attention to its comments. How? Add several lines for `public/css/syntax.css`:

{% highlight c linenos %}
/* display code line number; the line number is displayed aside from the code separated by a vertical line */
.highlight .lineno { color: #ccc; display:inline-block; padding: 0 5px; border-right:1px solid #ccc; }
.highlight pre code { display: block; white-space: pre; overflow-x: auto; word-wrap: normal; }

/* exclude line numbers from copy-paste user operations; if you copy the code, the line numbers are also copied, so this line is to disable line number copy */
.highlight .lineno {-webkit-user-select: none;-moz-user-select: none; -o-user-select: none;}

/* override text selection hightlighting for line numbers; although the above line disables line number copy, but when you select the code, line numbers are also selected which looks ugly; the transparent color will help erase this */
.lineno::-moz-selection {background-color: transparent;} /* Mozilla specific */
.lineno::selection {background-color: transparent;} /* Other major browsers */
{% endhighlight %}

**baseurl** in `_config.yml` file. Please refer to [GitHub Pages](http://jekyllrb.com/docs/github-pages/). Basically, pay attention to the *leading slash* and *trailing slash* issue when writing post. It is better to include the `site.baseurl` when accessing files. Don't use `/`. Because the blog might be migrated to some other places or to a project github pages in the future, which causes compatibility issue. For example, to include an image in post:
{% highlight html linenos %}
{% raw %}
<img src="{{site.baseurl}}assets/hknight.jpg">
{% endraw %}
{% endhighlight %}
If you replace:
`{% raw %}
{{site.baseurl}}
{% endraw %}`
with `/`, it is fine with user github page. But if the blog is migrated to project github page, then it does not work. You have to change `/` to `/project-name/` in your post. If you have many such posts, it is a big trouble. However, `site.baseurl` works fine. The only place you need to change is the `_config.yml` file.

## Setup

Some fun facts about the setup of Poole project include:

* Built for [Jekyll](http://jekyllrb.com)
* Developed on GitHub and hosted for free on [GitHub Pages](https://pages.github.com)
* Coded with [Sublime Text 2](http://sublimetext.com), an amazing code editor
* Designed and developed while listening to music like [Blood Bros Trilogy](https://soundcloud.com/maddecent/sets/blood-bros-series)

Have questions or suggestions? [Contact Poole author on Twitter](https://twitter.com/mdo).

Thanks for reading!
