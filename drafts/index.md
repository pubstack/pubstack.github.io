---
layout: post
title: Drafts
skip_related: true
skip_bio: true
---

<div id="archive">
<ul>
{% for post in site.posts %}
  {% if post.tags contains "draft" %}
    <li {% if post.favorite %}class="favorite"{% endif %}>
      <a href="{{ post.url }}">{{ post.title }}</a> {% for tag in post.tags %} {{ tag }}{% endfor %}
    </li>
  {% endif %}
{% endfor %}
</ul>
</div>
