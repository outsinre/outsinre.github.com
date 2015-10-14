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

<ul>
    {% for post in site.posts %}
    <li><span style="color:blue;font-family:monospace" class="time">{{ post.date | date_to_string }}</span> <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
