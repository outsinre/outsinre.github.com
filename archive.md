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
  <li><small><span class="time">{{ post.date | date: "%d/%b" }}</span>&nbsp;&nbsp;<a href="{{ post.url }}">{{ post.title }}</a></small></li>
{% endfor %}
-->

<ul>
    {% for post in site.posts %}
        <li> {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})</li>
    {% endfor %}
</ul>
