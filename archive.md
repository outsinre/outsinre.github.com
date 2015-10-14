---
layout: page
title: Archive
---

<!---
{% for post in site.posts %}
  {% capture month %}{{ post.date | date: '%m%Y' }}{% endcapture %}
  {% capture nmonth %}{{ post.next.date | date: '%m%Y' }}{% endcapture %}
    {% if month != nmonth %}
{{ post.date | date: '%B %Y' }}
    {% endif %}
  <li><small><span style="color:blue" class="time">{{ post.date | date: "%d/%b" }}</span>&nbsp;&nbsp;<a href="{{ post.url }}">{{ post.title }}</a></small></li>
{% endfor %}
-->

ul {
    margin: 0;
    padding: 0;
    list-style: none;
    width: 500px;
}

li {
    list-style-type: none;             
}

.leftpart {
    width: 100px;
    float:right;
    background-color: yellow;
}

.rightpart {
    width: 400px;
    float:left;
}

<ul>
    {% for post in site.posts %}
    <li><span class="time leftpart">{{ post.date | date_to_string }}</span><span class="rightpart"><a href="{{ post.url }}">{{ post.title }}</a></span></li>
    {% endfor %}
</ul>
