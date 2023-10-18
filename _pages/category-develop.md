---
title: "Development"
layout: archive
permalink: /develop
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.develop %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}