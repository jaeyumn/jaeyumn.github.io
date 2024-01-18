---
title: "Web"
layout: archive
permalink: /web
author_profile: true
sidebar:
  nav: "sidebar-category"
---

{% assign posts = site.categories.web %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
