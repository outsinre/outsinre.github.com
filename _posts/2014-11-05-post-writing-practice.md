---
layout: post
title: Post Writing Tips
---

Use **`** to enclose a code snippet. `int main(void){ printf("hello, world!"); return 1; }`. Or you can use **tab** key:

    int main(void){ ...}

Another way for code highlight is `pygments` by default (refer to Jekyll's offical site):  
{% raw %}
    {% hightlight ruby linenos %}
    {% endhight %}
{% endraw %}
You can replace `ruby` with any languages `pygments` supports (refer to [languages](http://pygments.org/languages/)).

An example. Pay attention to add `linenos` to display code line numbers.
{% highlight c linenos %}
int main(void){
    printf("hello, world!)'
    return 1;
}
{% endhighlight %}
How to write about Jekyll itself in Jkeyll?

For example, I want to write a post about Jekyll itself. Then I may need to include some Liquid statement, what can I do?

Refer to [Writing about Jekyll in Jekyll](http://blog.slaks.net/2013-06-09/writing-about-jekyll-in-jekyll/).

Basically:

 1. put your Liquid statement between {% raw %} {% raw %} {% endraw %} and &#123;% endraw %}.
 2. use `&#123;` to replace `{`. HTML recoginzes `&#123;` as `{`, while Liquid does not recognize it. This way is limited in that it only viable for html code.
