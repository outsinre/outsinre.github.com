---
layout: post
title: Post Writing Tips
---

Use **`** to enclose a code snippet. `int main(void){ printf("hello, world!"); return 1; }`

How to write about Jekyll itself in Jkeyll?

For example, I want to write a post about Jekyll itself. Then I may need to include some Liquid statement, what can I do?

Refer to [Writing about Jekyll in Jekyll](http://blog.slaks.net/2013-06-09/writing-about-jekyll-in-jekyll/).

Basically:

 1. put your Liquid statement between {% raw %} {% raw %} {% endraw %} and &#123;% endraw %}.
 2. use `&#123;` to replace `{`. HTML recoginzes `&#123;` as `{`, while Liquid does not recognize it.
