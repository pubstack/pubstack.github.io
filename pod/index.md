---
layout: post
title: Picture of the day (With some intermittency)
skip_related: true
skip_bio: true
---

<div id="archive">
<ul>
{% for post in site.posts %}
    {% unless post.tags contains "draft" %}
      {% if post.tags contains "pod" %}
        <li {% if post.favorite %}class="favorite"{% endif %}>
          {{post.date | date: "%B %d, %Y" }}: <a href="{{ post.url }}">{{ post.title }}</a> {% for tag in post.tags %} {% endfor %}
        </li>
      {% endif %}
    {% endunless %}
{% endfor %}
</ul>
</div>
