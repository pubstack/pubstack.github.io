---
layout: post
title: Archives
skip_related: true
skip_bio: true
---

{% assign totalwords = 0 %}
{% for post in site.posts %}
  {% assign wordcount = post.content | number_of_words %}
  {% assign totalwords = totalwords | plus: wordcount %}
{% endfor %}

Since {{ site.posts.last.date | date: "%B %d, %Y" }}, I've written {{ totalwords }} words in {{ site.posts | size }} posts, about software engineering, OpenStack,  professional growth and other miscellaneous topics. I hope you've enjoyed reading at least some of those words. My favorite posts are **bolded** below.

<div id="archive">

{% for post in site.posts %}
  {% unless post.tags contains "draft" %}
    {% assign currentdate = post.date | date: "%Y" %}
    {% if currentdate != date %}
      {% unless forloop.first %}</ul>{% endunless %}
      <h2>{{ currentdate }}</h2>
      <ul>
        {% assign date = currentdate %}
    {% endif %}

    <li {% if post.favorite %}class="favorite"{% endif %}>
      <a href="{{ post.url }}">{{ post.title }}</a> {% for tag in post.tags %} {{ tag }}{% endfor %}
    </li>
  {% endunless %}
  {% if forloop.last %}</ul>{% endif %}
{% endfor %}
</div>
