---
title: "C"
layout: archive
permalink: /C
author_profile: true
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.C %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}