---
title: "Trouble Shooting"
layout: archive
permalink: /trouble-shooting
author_profile: true
sidebar:
  nav: "sidebar-category"
---

{% assign posts = site.categories.trouble %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
