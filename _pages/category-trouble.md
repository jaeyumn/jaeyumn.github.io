---
title: "Trouble"
layout: archive
permalink: /trouble
author_profile: true
sidebar:
  nav: "sidebar-category"
---

{% assign posts = site.categories.trouble %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}
