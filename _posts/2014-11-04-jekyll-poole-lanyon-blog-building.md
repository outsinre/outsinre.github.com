---
layout: post
title: Jekyll Poole Lanyon Blog Building
---

<div class="message">
 This is to record the process of build the blog process.
</div>

The most important reference is [How I Created a Beautiful and Minimal Blog Using Jekyll, Github Pages, and poole](http://joshualande.com/jekyll-github-pages-poole/). The original pages is [Lanyon](https://github.com/poole/lanyon) which is based on [Poole](https://github.com/poole) on Github.

If you ever changed some files or folders and found them inappropriate, you can revert/discard the changed by command <strong>checkout</strong> command.

For example, if you changed the file *file.txt*, then you can use command:

    git checkout -- file.txt

If you want to discard all the changes:

    git checkout -- .

Originally the sidebar "Download" and "Github Project" items does not work. I found that it is due to *site.github.repo* is not defined in *_config.yml*.

Just add two lines in *_config.yml* as follows. Pay attention to the indent of the second line.

github:

  repo:	https://github.com/outsinre/outsinre.github.com

This is called **custom variable**. Solved! Futhermore, I removed the "Download" item since it is not necessary.

For site comments, I use DISQUS. Firsly, create a file *comments.html* and add two lines:
{% raw %}
*{% if page.comments %}*

<em>{% endif %}</em>
{% endraw %}

Then paste the DISQUS universal code below between the two lines. The *comments.html* should be placed under *_include* folder.

After that, in *_layout/post.html* file, add a line *&#123;% include comments.html %}* to the end.

## Setup

Some fun facts about the setup of Poole project include:

* Built for [Jekyll](http://jekyllrb.com)
* Developed on GitHub and hosted for free on [GitHub Pages](https://pages.github.com)
* Coded with [Sublime Text 2](http://sublimetext.com), an amazing code editor
* Designed and developed while listening to music like [Blood Bros Trilogy](https://soundcloud.com/maddecent/sets/blood-bros-series)

Have questions or suggestions? [Contact Poole author on Twitter](https://twitter.com/mdo).

Thanks for reading!
