---
layout: default
title: Cheat Sheets
permalink: /cheat-sheets/
tags:
  - docs
  - cheat-sheets
categories:
  - navigation
---

# Cheat Sheets

These pages come from the `_docs/` collection and are intended to be evergreen references.

{% assign sorted_docs = site.docs | sort: 'title' %}
{% for doc in sorted_docs %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
{% endfor %}

## Legacy topic folders

The original topic directories are preserved during transition:

- `Automation/`, `Backup/`, `Docker/`, `Firewall/`, `High Availability/`, `Load Balancing/`, `NTP/`, `SSH/`, `Tunning kernel/`, `Utils/`, `Web Services/`

As documents are modernized, corresponding pages will be moved or aliased into `_docs/`.
