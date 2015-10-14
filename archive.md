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
    <ul>
        <li> <span class="time">{{ post.date | date: "%d/%b" }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a> </li>
    </ul>
{% endfor %}
-->

<ul>
    {% for post in site.posts %}
        <li> <span style="display:inline-block;width:100px">{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a> </li>
    {% endfor %}
</ul>
