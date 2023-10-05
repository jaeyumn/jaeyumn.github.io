---
title: "Computer Science"
layout: archive
permalink: /computer-science
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.cs %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}