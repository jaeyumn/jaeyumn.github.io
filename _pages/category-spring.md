---
title: "spring"
layout: archive
permalink: /spring
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.spring %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}