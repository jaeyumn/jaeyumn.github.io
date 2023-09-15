---
title: "error"
layout: archive
permalink: /error-log
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.error %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}