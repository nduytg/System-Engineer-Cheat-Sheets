---
layout: default
title: Blog
permalink: /blog/
tags: [blog, posts]
categories: [navigation]
---

# Blog

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%B %-d, %Y" }}
{% endfor %}
