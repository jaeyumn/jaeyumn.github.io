---
title: "java"
layout: archive
permalink: /java
author_profile: true
sidebar:
    nav: "sidebar-category"
---

{% assign posts = site.categories.java %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}