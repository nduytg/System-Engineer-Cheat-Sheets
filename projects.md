---
layout: default
title: Projects
permalink: /projects/
tags: [projects, case-studies]
categories: [navigation]
---

# Platform Engineering Projects

{% assign sorted_projects = site.projects | sort: 'title' %}
{% for project in sorted_projects %}

- [{{ project.title }}]({{ project.url | relative_url }})

{% endfor %}
