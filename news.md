---
layout: default
title: Fez - News
---

news
===

{% for post in site.posts %}
<span class="news-item"><a href="{{ post.url }}">{{ post.title }}</a> <span class="date">{{ post.date | date_to_string }}</span></span>
{% endfor %}