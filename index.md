---
layout: default
title: Fez
---

{% for post in site.posts %}
<a href="{{ post.url }}">{{ post.title }}</a> <span class="date">{{ post.date | date_to_string }}</span>
{% endfor %}
