---
layout: post
title: Picture of the day
skip_related: true
skip_bio: true
---

<div id="archive">
<ul>
{% for post in site.posts %}
  {% if post.tags contains "pod" %}
    <li {% if post.favorite %}class="favorite"{% endif %}>
      {{post.date | date: "%B %d, %Y" }}: <a href="{{ post.url }}">{{ post.title }}</a> {% for tag in post.tags %} {% endfor %}
    </li>
  {% endif %}
{% endfor %}
</ul>
</div>
