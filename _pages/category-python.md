---
title: "Python"
layout: archive
permalink: /Python
author_profile: true
sidebar:
    nav: "docs"
---

{% assign posts = site.categories.Python %}
{% for post in posts %} {% include archive-single2.html type=page.entries_layout %} {% endfor %}