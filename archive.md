---
layout: page
title: Archive
---

{% for post in site.posts %}
  {% capture month %}{{ post.date | date: '%m%Y' }}{% endcapture %}
  {% capture nmonth %}{{ post.next.date | date: '%m%Y' }}{% endcapture %}
    {% if month != nmonth %}
*{{ post.date | date: '%B %Y' }}*
    {% endif %}
  <li><small><span class="time">{{ post.date | date: "%d/%m/%Y" }}</span> <a href="{{ post.url }}">{{ post.title }}</a></small></li>
{% endfor %}