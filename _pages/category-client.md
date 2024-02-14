---
title: "Client"
layout: archive
permalink: /client
author_profile: true
sidebar:
  nav: "sidebar-category"
---

{% assign posts = site.categories.client %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
